---
layout: post
title: "MyBatis中批量插入支持MySQL和ORACLE"
date: 2013-08-22 11:16
comments: true
categories: 
- 技海拾贝
- Java
tags:
- Java
- MyBatis
---

## 问题描述

在项目中，ORM使用的是MyBatis，而数据库则是同时使用MySQL和Oracle  
业务开发中需要进行批量的数据入库，需要有一个通用的方式支持MySQL和Oracle  
现有的操作方式只支持MySQL而不支持Oracle 

<!-- more --> 

## 问题分析

现有的操作方式，是通过在XML中拼接SQL语句的方式解决  
但是拼接出来的SQL语句，只能在MySQL中执行，Oracle无法正常执行  

    <insert id="create" parameterType="java.util.List">
        INSERT INTO GSG_KERNEL_GMSISDN_TAB (STARTMS,ENDMS,AREACODE)
        VALUES 
        <foreach collection="list" item="item" index="index" separator=",">
        (#{startMs}, #{endMs}, #{areaCode})
        </foreach>
    </insert>

原因是上面代码生成的SQL语句，在Oracle中是无效的  
所以这种方式不能支持两个数据库  

针对Oracle，使用UNION ALL修改成一份可以执行的SQL，代码如下：


    <insert id="create" parameterType="java.util.List">
        INSERT INTO GSG_KERNEL_GMSISDN_TAB (STARTMS,ENDMS,AREACODE)
        <foreach collection="list" item="item" index="index" separator=" UNION ALL ">
        SELECT #{item.startMs,jdbcType=VARCHAR},#{item.endMs,jdbcType=VARCHAR},#{item.areaCode,jdbcType=VARCHAR} FROM DUAL
        </foreach>
    </insert>


修改成上面的代码，便可以在Oracle中执行了  

但是，这样来解决批理入库，有两个问题：

> 1. 针对MySQL和Oracle两种数据库，需要写两份SQL语句  
> 2. 性能很差，10万条数据，入库需要大概100秒以上  

要解决这两个问题，光靠使用XML配置，已经无法解决了  

既然无法通过XML配置解决，那就需要使用JAVA API了  
在MyBatis中，执行可以分三种类型ExecutorType：  

> 1. SIMPLE  
> 这个类型不做特殊的事情，它只为每个语句创建一个PreparedStatement  
> 2. REUSE  
> 这种类型将重复使用PreparedStatements  
> 3. BATCH  
> 这个类型进行批量操作  


进行批量操作时，只需要将类型设置为BATCH就可以了  
具体的代码，可以参见下一节，问题解决方法  

## 解决方法

通过Java API的方式去执行批量插入操作，
工具类代码如下：



    /*******************************************************************************
     * @(#)MyBatisBatchSupport.java 2013年8月21日
     *
     *******************************************************************************/
    package com.neusoft.mooko.core.persistence;

    import java.lang.reflect.Method;
    import java.util.List;

    import org.apache.ibatis.session.ExecutorType;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;

    /**
     * MyBatis中批量处理的操作类
     * 目前只实现了批量插入
     * 
     * 由于使用XML来拼接SQL的效率太差
     * 所以使用Java API的方式来解决这个问题
     * 
     * @author <a href="mailto:puras@mooko.net">Puras.He</a>
     * @version $Revision 1.0 $ 2013年8月21日 上午9:56:58
     */
    public class MyBatisBatchSupport {
        
        /**
         * 批量插入
         * 
         * @param sqlSessionFactory
         * @param statement 在Mapper文件中定义的namespce加上Mapper中定义的标识符
         * @param objList 要入库的数据列表
         */
        public static void batchInsert(SqlSessionFactory sqlSessionFactory, String statement, List<?> objList) {
            SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH, false);
            
            try {
                for (Object obj : objList) {
                    session.insert(statement, obj);
                }
                session.flushStatements();
                session.commit();
                session.clearCache();
            } catch (Exception ex) {
                ex.printStackTrace();
                session.rollback();
            } finally {
                session.close();
            }
        }
        
        /**
         * 批量插入
         * @param sqlSessionFactory
         * @param mapperClass 要使用的Mapper的Class
         * @param pojoClass 列表中POJO对象的Class
         * @param methodName 要执行的Mapper类中的方法名
         * @param objList 要入库的数据列表
         */
        @SuppressWarnings({ "rawtypes", "unchecked" })
        public static <T> void batchInsertByMapper(SqlSessionFactory sqlSessionFactory, Class<?> mapperClass, Class pojoClass, String methodName, List<?> objList) {
            SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH, false);
            
            Class[] paramTypes = new Class[1];
            T mapper = (T) session.getMapper(mapperClass);
            try {
                paramTypes[0] = Class.forName(pojoClass.getName());
                Method method = mapperClass.getMethod(methodName, paramTypes);
                for (Object obj : objList) {
                    method.invoke(mapper, obj);
                }
                session.flushStatements();
                session.commit();
                session.clearCache();
            } catch (Exception ex) {
                ex.printStackTrace();
                session.rollback();
            } finally {
                session.close();
            }
        }
    }



调用代码，是一个Controller，可快速查看结果：


    @RequestMapping("batch_insert/{num}")
    public String batchInsert(ModelMap model, @PathVariable Integer num) {
        List<Test> ts = new ArrayList<Test>();
        if (null == num || num == 0) num = 1;
        System.out.println("Number->>>>>" + num);
        for (int i = 0;i < num; i++) {
            Test t = new Test();
            t.setStartMs("" + i);
            t.setEndMs("1");
            t.setAreaCode("1");
            ts.add(t);
        }
        Long start = System.currentTimeMillis();
        // 使用Statement方式
        MyBatisBatchSupport.batchInsert(sqlSessionFactory, "com.neusoft.mooko.auth.persistence.shared.TestDao.create2", ts);
        // 使用Mapper方式
        MyBatisBatchSupport.batchInsertByMapper(sqlSessionFactory, TestDao.class, Test.class, "create2", ts);
        Long end = System.currentTimeMillis();
        
        System.out.println("Time =======> " + (end - start));

        model.addAttribute("useTime", (end - start));
        return "batch_insert";
    }


其中，
* SqlSessionFactory是MyBatis中的Session工厂
> 一般情况，都是在Spring配置文件中定义的
> 直接注入到类中就可以
* "com.neusoft.mooko.auth.persistence.shared.TestDao.create2"
> 这个是你的Mapper中定义的一个操作语句，不写Mapper的文件名
> 而是使用你在Mapper文件中定义的namespce加上Mapper中定义的标识符
> 内容则是普通的插入单条数据的SQL语句，如


    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
     "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

    <mapper namespace="com.neusoft.mooko.auth.persistence.shared.TestDao">
      <cache />
      <insert id="create2" parameterType="Test">
        INSERT INTO GSG_KERNEL_GMSISDN_TAB (STARTMS, ENDMS, AREACODE)
        values
        (#{startMs}, #{endMs}, #{areaCode})
      </insert>
    </mapper>


* 参数
> 要执行批量操作的数据参数

### 执行效率

在工作电脑上执行的数据，每次执行的时间不一样，大致相差不多：

* 100条：用时 22 毫秒
* 1000条：用时 31 毫秒
* 10000条：用时 226 毫秒
* 100000条：用时 2423 毫秒

## 注意事项

通过Batch方式去执行批量操作，有一个问题，  
那就是在Insert操作时，事务提交之前，是没有办法获取到自增的id，  
这在某型情形下是不符合业务要求的。  


由于上面的这个问题，导致在写MyBatis的XML配置时，也需要注意，  
不能在INSERT语句的XML中，使用useGeneratedKeys="true"这个属性  
下面的代码是无法正常使用Batch方式的:


    <insert id="create2" useGeneratedKeys="true" parameterType="Test">
        
        INSERT INTO GSG_KERNEL_GMSISDN_TAB (STARTMS, ENDMS, AREACODE)
        values
        (#{startMs}, #{endMs}, #{areaCode})
    </insert>
