style目录下

```
package io.ck.utils.style;

import java.util.ArrayList;
import java.util.List;

/**
 * @author wpp
 */
public class TableStyle {
    /**
     * 表头背景色
     */
    private List<String> header = new ArrayList<>();
    /**
     * 背景颜色
     */
    private String hexColor;
    /**
     * 数据
     */
    List<List<String>> value = new ArrayList<>();

    public List<String> getHeader() {
        return header;
    }

    public void setHeader(List<String> header) {
        this.header = header;
    }

    public String getHexColor() {
        return hexColor;
    }

    public void setHexColor(String hexColor) {
        this.hexColor = hexColor;
    }

    public List<List<String>> getValue() {
        return value;
    }

    public void setValue(List<List<String>> value) {
        this.value = value;
    }
}

```





```
package io.ck.utils.style;

import org.docx4j.wml.JcEnumeration;

/**
 * @author wpp
 */
public class TextStyle {
    public TextStyle(String value, String style, JcEnumeration jcEnumeration) {
        this.value = value;
        this.style = style;
        this.jcEnumeration = jcEnumeration;
    }

    private String value;
    private String style;
    private JcEnumeration jcEnumeration;

    public String getValue() {
        return value;
    }

    public String getStyle() {
        return style;
    }

    public JcEnumeration getJcEnumeration() {
        return jcEnumeration;
    }
}

```

封装工具

```
package io.ck.utils;

import io.ck.utils.style.TableStyle;
import io.ck.utils.style.TextStyle;
import org.docx4j.jaxb.Context;
import org.docx4j.openpackaging.exceptions.Docx4JException;
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.wml.*;


import java.math.BigInteger;
import java.util.List;

/**
 * @author wpp
 */
public class DocxUtils {

    public static WordprocessingMLPackage addHead(WordprocessingMLPackage wordMLPackage, TextStyle textStyle){
        ObjectFactory factory = new ObjectFactory();
        // 创建段落
        P titleParagraph = factory.createP();
        R titleRun = factory.createR();
        Text titleText = factory.createText();
        titleText.setValue(textStyle.getValue());
        titleRun.getContent().add(titleText);
        titleParagraph.getContent().add(titleRun);
        // 创建段落属性对象（PPr）
        PPr titlePPr = factory.createPPr();
        // 设置段落样式为 Heading 1
        PPrBase.PStyle titlePStyle = new PPrBase.PStyle();
        // 设置样式为 Heading1
        titlePStyle.setVal(textStyle.getStyle());
        titlePPr.setPStyle(titlePStyle);

        // 设置段落对齐方式为居中
        Jc titleAlignment = factory.createJc();
        // 设置居中对齐
        titleAlignment.setVal(textStyle.getJcEnumeration());
        titlePPr.setJc(titleAlignment);

        // 将段落属性应用到段落
        titleParagraph.setPPr(titlePPr);
        // 将段落添加到文档
        wordMLPackage.getMainDocumentPart().addObject(titleParagraph);
        return wordMLPackage;
    }



    public static WordprocessingMLPackage addTable(WordprocessingMLPackage wordMLPackage, TableStyle tableStyle) throws Docx4JException {
        // 创建一个标题
        ObjectFactory factory = Context.getWmlObjectFactory();
        // 创建表格
        Tbl table = factory.createTbl();
        // 设置表格边框
        addBorders(table);

        // 添加表头
        if(tableStyle.getHeader() != null){
            addTableRowWithShading(factory, table, tableStyle.getHeader(), tableStyle.getHexColor());
        }

        // 添加数据行
        for (List<String> value : tableStyle.getValue()){
            addTableRow(factory, table, value);
        }

        // 将表格添加到文档中
        wordMLPackage.getMainDocumentPart().addObject(table);

        return wordMLPackage;
    }
    // 添加带背景的表头行
    private static void addTableRowWithShading(ObjectFactory factory, Tbl table, List<String> rowData, String hexColor) {
        Tr row = factory.createTr();
        for (String cellData : rowData) {
            Tc cell = factory.createTc();
            P cellParagraph = factory.createP();
            R cellRun = factory.createR();
            Text cellText = factory.createText();
            cellText.setValue(cellData);
            cellRun.getContent().add(cellText);
            cellParagraph.getContent().add(cellRun);
            cell.getContent().add(cellParagraph);

            // 设置背景色
            TcPr tcPr = factory.createTcPr();
            CTShd shading = new CTShd();
            shading.setVal(STShd.CLEAR);  // 清除已有阴影
            shading.setColor("auto");
            shading.setFill(hexColor);   // 设置背景颜色 (灰色)
            tcPr.setShd(shading);
            cell.setTcPr(tcPr);

            row.getContent().add(cell);
        }
        table.getContent().add(row);
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
        border.setSpace(new BigInteger("10"));
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

