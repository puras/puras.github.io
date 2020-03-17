title: "Eclipse无法启动之解决办法"
date: 2014-11-27 14:04:23
categories: 
- 技海拾贝
- 工具
tags:
- Eclipse
---

今天在使用Eclipse的时候，启了一下Debug，结果Eclipse就死在那了，强制关掉之后，Eclipse则无法正常启动，卡在启动界面上无响应。

重启，强制关闭，都不行……

<!-- more -->

卡在启动界面，提示："Loading org.eclipse.mylyn.tasks.index.core"，查了一下Eclipse的日志，发现有下面的错误日志：

    ```[shell][错误日志]
    !ENTRY org.eclipse.m2e.logback.configuration 2 0 2014-11-27 13:58:57.386
    !MESSAGE Exception while setting up logging:org.eclipse.osgi.internal.framework.EquinoxConfiguration$1 cannot be cast to java.lang.String
    !STACK 0
    java.lang.ClassCastException: org.eclipse.osgi.internal.framework.EquinoxConfiguration$1 cannot be cast to java.lang.String
        at org.eclipse.m2e.logback.configuration.LogHelper.logJavaProperties(LogHelper.java:26)
        at org.eclipse.m2e.logback.configuration.LogPlugin.loadConfiguration(LogPlugin.java:189)
        at org.eclipse.m2e.logback.configuration.LogPlugin.configureLogback(LogPlugin.java:144)
        at org.eclipse.m2e.logback.configuration.LogPlugin.access$2(LogPlugin.java:107)
        at org.eclipse.m2e.logback.configuration.LogPlugin$1.run(LogPlugin.java:62)
        at java.util.TimerThread.mainLoop(Timer.java:555)
        at java.util.TimerThread.run(Timer.java:505)
    ```

无法确认该日志是否是导致无法正常启动的原因。

Google上查了一下，关键词是"Eclipse start org.eclipse.mylyn.tasks.index.core"，按照搜索结果中的办法试了一下，移除了Eclipse工作目录下.metadata/.plugins/org.eclipse.e4.workbench子目录下的workbench.xmi文件，再重新启动Eclipse，则可以正常启动了，只是工作区的配置都没有了，需要重新配置。

原因：没有查到详细的介绍，猜测是工作区配置信息出错了，导致Eclipse无法正常启动。真正的原因，未知，有了解的朋友，可以告知一下，谢谢。