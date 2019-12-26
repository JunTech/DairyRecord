# java操作excel表

## 1.准备工作

准备一个excel表，我这里是（.xslx）,poi依赖，maven依赖如下：

```xml
<!--APACHE POI-->
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.11</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.11</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml-schemas</artifactId>
            <version>3.11</version>
        </dependency>
```

读取的excel表如下：

```sql
name	age	gender
jay	22	man
ryan	22	man

```

## 2.代码

```java
package top.juntech.excel;



import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hssf.usermodel.HSSFSheet;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

import java.io.FileInputStream;


public class TestPoi {
    //先抛出file*异常
    public static void testPOI() throws Exception{
        //1.读取excel表数据
        String c = System.getProperty("user.dir");
        //这里我们使用XSSFWorkBook,因为这是office2007+,如果版本低于2007的，可以使用HSSFWorkBook
//        C:\Users\Ryan\Desktop\jvm\testpoi\src\main\resources\test.xlsx
//        XSSFWorkbook workbook = new XSSFWorkbook(new FileInputStream(c+"\\src\\main\\resources\\test.xlsx"));
        HSSFWorkbook workbook = new HSSFWorkbook(new FileInputStream(c+"\\src\\main\\resources\\test.xlsx"));
        //2.获取要解析的表格
//        XSSFSheet sheet = workbook.getSheetAt(0);
        HSSFSheet sheet = workbook.getSheetAt(0);
        //获取最后一行的行号
        int lastRowNum = sheet.getLastRowNum();
        //遍历每一行的数据
        for(int i=1;i<=lastRowNum;i++){
            //获取要解析的行
//            XSSFRow row = sheet.getRow(i);
            HSSFRow row = sheet.getRow(i);
            //获取该行每一个单元格的数据
            String name = row.getCell(0).getStringCellValue();
            String age = row.getCell(1).getStringCellValue();
            String gender = row.getCell(2).getStringCellValue();
            System.out.println("name:"+name+",age:"+age+"gender:"+gender);
        }
    }

    public static void main(String[] args) throws Exception{
        testPOI();
    }

}

```

在这里，可能会异常，其中有几种常见的，我将列举在下面：

```json
根据 Apache POI快速指南， POIFSFileSystem （或类似地， NPOIFSFileSystem ）仅用于.xls（Excel版本到2003年）文档。
```



### 2.1 office 2007+

报错异常：org.apache.poi.poifs.filesystem.OfficeXmlFileException：提供的数据似乎在Office 2007+ XML。您正在调用处理OLE2 Office文档的POI部分。您需要调用POI的其他部分来处理此数据（例如，XSSF而不是HSSF）

使用上面的XSSFWorkBook，将上面的XSSFWorkBook注释去掉就行了。

### 2.2 office 2003

使用上面的HSSFWorkBook,保持不动就行

### 2.3  java.io.IOException

异常信息：Exception in thread "main" java.io.IOException: Unable to read entire header; 250 bytes read; expected 512 bytes

说明数据量太少了，不支持poi读取

解决方法：“readFirst512()将读取您的前512个字节，Inputstream并在没有足够的字节读取时抛出异常。我认为您的文件不够大，无法被POI阅读。”

可以修改一下excel格式或者把xls改成xlsx都行，

#### 2.3.1 使用workbookFactory

由于Excel版本的关系，文件扩展名分xls和xlsx，

以往的经验都是使用HSSFWorkbook和XSSFWorkbook来分别处理。具体的方式就是先判断文件的类型，然后根据文件扩展名来选择方法

最终整个解析Excel文件的功能才算完成，我们需要实现4个方法readXlsForAllSheets和readXlsxForAllSheets，getXssfCellValue和getHssfCellValue，那么有没有更加简单实用的方法呢？

下面要介绍的是POI jar包提供的WorkbookFactory类。需要加载poi-ooxm-3.15.jar到build path。

只需要两行就可以实例化workbook，而不用管它是xls还是xlsx。

```java
inStream = new FileInputStream(new File(filePath));
Workbook workBook = WorkbookFactory.create(inStream);
```

 后续可以直接操作sheet，Row,Cell,也不用管文件类型。

目前还没有发现这种方法的缺点。