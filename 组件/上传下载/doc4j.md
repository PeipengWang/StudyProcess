`docx4j` 是一个用于操作 Word 文档（.docx）的 Java 库，可以帮助你创建、读取、修改和保存 Word 文档。以下是 `docx4j 6.1.2` 版本的使用教程，涵盖了常用的功能，包括创建 Word 文档、添加文本、表格、设置样式等。

### **1. 添加依赖**

首先，如果你使用 Maven 来管理项目，在 `pom.xml` 中添加 `docx4j` 依赖：

```xml
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j</artifactId>
    <version>6.1.2</version>
</dependency>
```

### **2. 创建一个简单的 Word 文档**

#### **创建并保存一个简单的 Word 文档**

```java
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.wml.*;

import java.io.File;

public class Docx4jExample {
    public static void main(String[] args) throws Exception {
        // 创建一个 Word 文档包
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();

        // 创建一个段落
        ObjectFactory factory = new ObjectFactory();
        P paragraph = factory.createP();

        // 创建运行 (run)，即文本内容
        R run = factory.createR();
        Text text = factory.createText();
        text.setValue("Hello, docx4j!");
        run.getContent().add(text);

        // 将运行添加到段落中
        paragraph.getContent().add(run);

        // 将段落添加到文档
        wordMLPackage.getMainDocumentPart().addObject(paragraph);

        // 保存文档
        wordMLPackage.save(new File("example.docx"));
    }
}
```

### **3. 向文档中添加标题**

你可以使用 `PStyle` 设置段落的样式，从而创建标题。

#### **添加标题**

```java
  // 创建 Word 文档
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();
        ObjectFactory factory = new ObjectFactory();

        // 创建段落
        P titleParagraph = factory.createP();
        R titleRun = factory.createR();
        Text titleText = factory.createText();
        titleText.setValue("Document Title");

        titleRun.getContent().add(titleText);
        titleParagraph.getContent().add(titleRun);

        // 创建段落属性对象（PPr）
        PPr titlePPr = factory.createPPr();

        // 设置段落样式为 Heading 1
        PPrBase.PStyle titlePStyle = new PPrBase.PStyle();
        titlePStyle.setVal("Heading1");  // 设置样式为 Heading1
        titlePPr.setPStyle(titlePStyle);

        // 设置段落对齐方式为居中
        Jc titleAlignment = factory.createJc();
        titleAlignment.setVal(JcEnumeration.CENTER);  // 设置居中对齐
        titlePPr.setJc(titleAlignment);

        // 将段落属性应用到段落
        titleParagraph.setPPr(titlePPr);

        // 将段落添加到文档
        wordMLPackage.getMainDocumentPart().addObject(titleParagraph);

        // 保存文件
        wordMLPackage.save(new File("example_with_heading1.docx"));
```

### **4. 创建表格**

`docx4j` 允许你在文档中创建表格。你可以定义表格、行和单元格，并设置每个单元格的内容。

#### **创建表格**

```java
import org.docx4j.jaxb.Context;
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.wml.*;

import java.math.BigInteger;
import java.util.Arrays;
import java.util.List;

public class Docx4jTableExample {
    public static void main(String[] args) throws Exception {
        // 创建 Word 文档
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();

        // 创建一个标题
        ObjectFactory factory = Context.getWmlObjectFactory();
        P title = factory.createP();
        R titleRun = factory.createR();
        Text titleText = factory.createText();
        titleText.setValue("User Information Table");
        titleRun.getContent().add(titleText);
        title.getContent().add(titleRun);
        wordMLPackage.getMainDocumentPart().addObject(title);

        // 创建表格
        Tbl table = factory.createTbl();

        // 设置表格边框
        addBorders(table);

        // 添加表头
        addTableRow(factory, table, Arrays.asList("ID", "Name", "Age"));

        // 添加数据行
        addTableRow(factory, table, Arrays.asList("1", "Alice", "25"));
        addTableRow(factory, table, Arrays.asList("2", "Bob", "30"));
        addTableRow(factory, table, Arrays.asList("3", "Charlie", "28"));

        // 将表格添加到文档中
        wordMLPackage.getMainDocumentPart().addObject(table);

        // 保存文档
        wordMLPackage.save(new java.io.File("UserTable.docx"));
        System.out.println("Word document created: UserTable.docx");
    }

    // 添加表格行
    private static void addTableRow(ObjectFactory factory, Tbl table, List<String> cellValues) {
        Tr row = factory.createTr();
        for (String value : cellValues) {
            Tc cell = factory.createTc();

            // 创建段落并添加文本
            P paragraph = factory.createP();
            R run = factory.createR();
            Text text = factory.createText();
            text.setValue(value);

            run.getContent().add(text);
            paragraph.getContent().add(run);
            cell.getContent().add(paragraph);

            row.getContent().add(cell);
        }
        table.getContent().add(row);
    }

    // 设置表格边框
    private static void addBorders(Tbl table) {
        TblPr tblPr = Context.getWmlObjectFactory().createTblPr();
        TblBorders borders = Context.getWmlObjectFactory().createTblBorders();

        CTBorder border = Context.getWmlObjectFactory().createCTBorder();
        border.setColor("auto");
        border.setSz(new BigInteger("4"));
        border.setSpace(new BigInteger("0"));
        border.setVal(STBorder.SINGLE);

        borders.setTop(border);
        borders.setBottom(border);
        borders.setLeft(border);
        borders.setRight(border);
        borders.setInsideH(border);
        borders.setInsideV(border);

        tblPr.setTblBorders(borders);
        table.setTblPr(tblPr);
    }
}

```

### **5. 设置表格样式（边框、颜色等）**

`docx4j` 允许你自定义表格的样式，包括边框、背景色等。

#### **为表格设置边框和背景色**

```java
import org.docx4j.wml.*;
import java.math.BigInteger;

public class Docx4jExample {
    public static void main(String[] args) throws Exception {
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();
        ObjectFactory factory = new ObjectFactory();

        // 创建表格
        Tbl table = factory.createTbl();
        TblPr tblPr = factory.createTblPr();
        TblBorders borders = factory.createTblBorders();
        
        // 设置边框样式
        CTBorder border = factory.createCTBorder();
        border.setColor("000000");  // 黑色
        border.setSz(BigInteger.valueOf(4));  // 边框宽度
        borders.setTop(border);
        borders.setBottom(border);
        borders.setLeft(border);
        borders.setRight(border);
        borders.setInsideH(border);
        borders.setInsideV(border);
        tblPr.setTblBorders(borders);
        
        table.setTblPr(tblPr);

        // 创建行和单元格
        Tr row = factory.createTr();
        Tc cell = factory.createTc();
        P paragraph = factory.createP();
        R run = factory.createR();
        Text text = factory.createText();
        text.setValue("Cell with Border");
        run.getContent().add(text);
        paragraph.getContent().add(run);
        cell.getContent().add(paragraph);
        row.getContent().add(cell);

        // 将行添加到表格
        table.getContent().add(row);

        // 将表格添加到文档
        wordMLPackage.getMainDocumentPart().addObject(table);

        // 保存文件
        wordMLPackage.save(new File("styled_table.docx"));
    }
}
```

### **6. 设置段落和字体样式**

你可以设置段落的对齐方式、字体样式（加粗、斜体等）以及字体大小等。

#### **设置字体样式和段落对齐**

```java
import org.docx4j.wml.*;

public class Docx4jExample {
    public static void main(String[] args) throws Exception {
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();
        ObjectFactory factory = new ObjectFactory();

        // 创建段落
        P paragraph = factory.createP();
        R run = factory.createR();
        Text text = factory.createText();
        text.setValue("This is a styled paragraph");

        run.getContent().add(text);
        paragraph.getContent().add(run);

        // 设置字体样式：加粗、斜体
        RPr rpr = factory.createRPr();
        BooleanDefaultTrue b = factory.createBooleanDefaultTrue();
        rpr.setB(b);
        BooleanDefaultTrue i = factory.createBooleanDefaultTrue();
        rpr.setI(i);
        run.setRPr(rpr);

        // 设置段落对齐
        PPr paragraphProperties = factory.createPPr();
        Jc alignment = factory.createJc();
        alignment.setVal(JcEnumeration.CENTER);  // 设置居中对齐
        paragraphProperties.setJc(alignment);
        paragraph.setPPr(paragraphProperties);

        // 将段落添加到文档
        wordMLPackage.getMainDocumentPart().addObject(paragraph);

        // 保存文件
        wordMLPackage.save(new File("styled_paragraph.docx"));
    }
}
```

### **7. 使用模板生成文档**

你可以加载一个现有的模板（`.docx` 文件），然后替换其中的占位符。

```java
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.openpackaging.parts.WordprocessingML.MainDocumentPart;

public class Docx4jExample {
    public static void main(String[] args) throws Exception {
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.load(new java.io.File("template.docx"));
        MainDocumentPart documentPart = wordMLPackage.getMainDocumentPart();

        // 在模板中查找并替换占位符
        documentPart.variableReplace(Collections.singletonMap("{{name}}", "John Doe"));

        // 保存生成的文档
        wordMLPackage.save(new java.io.File("generated_document.docx"));
    }
}
```

### **总结**

通过 `docx4j` 6.1.2