---
layout: post
title: "使用POI和iText将Excel转换成PDF"
date: 2013-09-16 15:28
comments: true
categories: 
- 技海拾贝
- Java
tags: 
- Java 
- POI
- iText
- Excel 
- PDF
---

前几天，同事问有没有什么方案能将Excel转换成PDF，说了几个技术都有些问题。 
我给的建议是使用POI和iText，但是这两个技术只是了解过，没有真正的使用过。 
写了一些简单的程序，来验证是否可行。 

<!-- more -->

由于项目使用的Maven，所以要加载poi和iText这两个包，只需要添加两段依赖便可： 

    <dependency>
        <groupId>org.apache.poi</groupId>
        <artifactId>poi</artifactId>
        <version>3.9</version>
    </dependency>
    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itextpdf</artifactId>
        <version>5.4.3</version>
    </dependency>

注意：使用iText5.0以后的版本。 

下面介绍一些基础的内容，从打开文件到获取Workbook、Sheet，再到读取行和列的值，一步步的解析一个Excel文件，复杂的情况，这里并未涉及。 

* 打开Excel文件 

        InputStream is = ExcelToPdf.class.getResourceAsStream("test.xls");
        POIFSFileSystem fs = new POIFSFileSystem(is);

* 获取Workbook对象 

        HSSFWorkbook book = new HSSFWorkbook(fs);

* 按索引读取Sheet 

        book.getSheetAt(0);

* 遍历所有的Sheet 

        for (int i = 0; i < book.getNumberOfSheets(); i++) {
            sheet = book.getSheetAt(i);
            System.out.println('SheetName: ' + sheet.getSheetName());
        }

* 按索引读取行 

        HSSFRow row = sheet.getRow(0);

* 遍历所有的行 

        for (int j = 0; j <= sheet.getPhysicalNumberOfRows(); j++) {
            HSSFRow row = sheet.getRow(j);
        }

* 按索引读取列 

        row.getCell(0);

* 遍历一行中所有的列 

        for (int k = 0; k <= row.getPhysicalNumberOfCells(); k++) {
            row.getCell(k);
        }

读取Excel非常简单，同时，写一个PDF文件，也比较简单。 

* 创建一个PDF文档的实例 

        Document doc = new Document();
        doc.setPageSize(PageSize.A4);
        PdfWriter.getInstance(doc, new FileOutputStream("test.pdf"));

文件名为test.pdf，同时设置页面大小为A4。 

* 打开PDF文档，以便写入内容 

        doc.open();

* 向PDF写入内容 

写入内容比较简单，这里只测试了文本，未测试图片等其他的内容。 

    doc.add(new Paragraph("Hello World!"));

* 向PDF写入表格 

向PDF中写入表格也非常简单，首先定义一个表格对象，需指定列数；之后向里面增加单元格，这里就不像POI一样可以按行列的插入了，只能插入单元格，按定义表格对象时传入的列数来自动换行。没有找到更有效的方式，有知道请告知。 

    // 定义表格对象，指定列数，也可传入数组，指定每列的宽度
    PdfPTable table = new PdfPTable(maxNum);
    // 设置表格的宽度
    table.setWidthPercentage(100);
    // 设置表格的对齐方式
    table.setHorizontalAlignment(PdfPTable.ALIGN_LEFT);
    // 定义单元格
    PdfPCell cell = new PdfPCell();
    // 设置单元格的对齐方式
    cell.setHorizontalAlignment(PdfPCell.ALIGN_CENTER);
    // 将单元格写入表格
    table.addCell(cell);
    // 将表格写入PDF文档
    doc.add(table);

* 设置字体 

如果使用上面的代码来向PDF中写入内容，中文的话，会出现乱码，可以通过指定字体来解决。 
指定字体有几种方式，一种是使用系统中自带的字体，一种是将字体文件放入classpath，还有一种是使用iTextAsian来解决。 
这里只介绍使用iTextAsian的方式。 

首先是引入iTextAsian的包： 

    <dependency>
        <groupId>com.itextpdf</groupId>
        <artifactId>itext-asian</artifactId>
        <version>5.2.0</version>
    </dependency>

之后在程序中定义字体： 

    BaseFont bf = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
    Font headFont = new Font(bf, 12, Font.BOLD);
    Font font = new Font(bf, 10, Font.NORMAL);

定义字体之后，便可以在写入内容时设置相应的字体了： 

    doc.add(new Paragraph("Hello World", headFont));
    doc.add(new Paragraph("大家好", font));

* 关闭文档 

写完内容后不要忘关闭PDF文档 

    doc.close();

通过上面这些，便可以将一个Excel简单的转换成PDF文档了，为了生成的内容更统一，可以在遍历Excel的内容时，获取每个单元格的样式，再设置到PDF文档中的表格的单元格中。这样生成的PDF文档才能和Excel更相近。细节的调整，这里就没有进一步的测试了。 

同时，在读取Excel文件内容时，需要注意的是Excel中的单元格的内容类型不，需要根据业务需求做进一步的处理。 

下面给出测试的完整代码： 

    public class ExcelToPdf {        
        public static void main(String[] args) {
            
            try {
                int maxNum = 1;
                InputStream is1 = ExcelToPdf.class.getResourceAsStream("test.xls");
                POIFSFileSystem fs1 = new POIFSFileSystem(is1);
                HSSFWorkbook book1 = new HSSFWorkbook(fs1);
                HSSFSheet sheet1;
                for (int i = 0; i < book1.getNumberOfSheets(); i++) {
                    sheet1 = book1.getSheetAt(i);
                    for (int j = 0; j <= sheet1.getPhysicalNumberOfRows(); j++) {
                        HSSFRow row = sheet1.getRow(j);
                        if (null != row) {
                            if (maxNum < row.getPhysicalNumberOfCells()) {
                                maxNum = row.getPhysicalNumberOfCells();
                            }
                        }
                    }
                }
                maxNum += 1;
                InputStream is = ExcelToPdf.class.getResourceAsStream("test.xls");
                Document doc = new Document();
                doc.setPageSize(PageSize.A4);
                PdfWriter.getInstance(doc, new FileOutputStream("test.pdf"));
                
                BaseFont bf = BaseFont.createFont("STSong-Light", "UniGB-UCS2-H", BaseFont.NOT_EMBEDDED);
                Font headFont = new Font(bf, 12, Font.BOLD);
                Font font = new Font(bf, 10, Font.NORMAL);
                doc.open();
                
                POIFSFileSystem fs = new POIFSFileSystem(is);
                HSSFWorkbook book = new HSSFWorkbook(fs);
                HSSFSheet sheet;
                for (int i = 0; i < book.getNumberOfSheets(); i++) {
                    sheet = book.getSheetAt(i);
                    doc.add(new Paragraph(sheet.getSheetName(), headFont));
                    PdfPTable table = new PdfPTable(maxNum);
                    table.setWidthPercentage(100);
                    table.setHorizontalAlignment(PdfPTable.ALIGN_LEFT);
                    for (int j = 0; j <= sheet.getPhysicalNumberOfRows(); j++) {
                        PdfPCell cell = new PdfPCell();
                        cell.setHorizontalAlignment(PdfPCell.ALIGN_CENTER);
                        HSSFRow row = sheet.getRow(j);
                        if (null != row) {
                            int currentNum = row.getPhysicalNumberOfCells();
                            for (int k = 0; k < maxNum; k++) {
                                if (currentNum < maxNum) {
                                    String value = getCellFormatValue(row.getCell(k));
                                    cell.setPhrase(new Paragraph(value, font));
                                } else {
                                    cell.setPhrase(new Paragraph(" ", font));
                                }
                                table.addCell(cell);
                            }
                        }
                    }
                    doc.add(table);
                }
                doc.close();
            } catch (Exception e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        
        private static String getCellFormatValue(HSSFCell cell) {
            String cellValue = "";
            if (null != cell) {
                switch(cell.getCellType()) {
                case HSSFCell.CELL_TYPE_NUMERIC:
                case HSSFCell.CELL_TYPE_FORMULA:
                    if (HSSFDateUtil.isCellDateFormatted(cell)) {
                        Date date = cell.getDateCellValue();
                        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
                        cellValue = sdf.format(date);
                    } else {
                        DecimalFormat df = new DecimalFormat("########");
                        cellValue = df.format(cell.getNumericCellValue());
                    }
                    break;
                case HSSFCell.CELL_TYPE_STRING:
                    cellValue = cell.getRichStringCellValue().getString();
                    break;
                default:
                    cellValue = " ";
                }
            } else {
                cellValue = "";
            }
            return cellValue;
        }
    }

*注意：在Ubuntu系统下，使用了iTextxxx包之后，仍然无法正常的输出中文，查阅资料后，安装了poppler-data后，便可正常输出。*