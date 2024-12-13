2.基础创建
这是基本的操作，设定一个路径，如果文件不存在，则创建空文档

```
private static String docxPath = "/Users/chenhailong/Desktop/helloworld.docx";

/**
 * 获取文档可操作对象
 * @param docxPath 文档路径
 * @return
 */
static WordprocessingMLPackage getWordprocessingMLPackage(String docxPath) {
	WordprocessingMLPackage wordMLPackage = null;
	if(StringUtils.isEmpty(docxPath)) {
		try {
			wordMLPackage =  WordprocessingMLPackage.createPackage();
		} catch (InvalidFormatException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	File file = new File(docxPath);
	if(file.isFile()) {
		try {
			wordMLPackage = WordprocessingMLPackage.load(file);
		} catch (Docx4JException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	return wordMLPackage;
}


```

3.追加文档内容

```
/**
 * 像文档中追加内容(默认支持中文)
 * 先清空,在生成,防重复
 */
public static void addParagraph() {
    WordprocessingMLPackage wordprocessingMLPackage;
    try {
    	//先加载word文档
        wordprocessingMLPackage = WordprocessingMLPackage.load(new File(docxPath));
//            Docx4J.load(new File(docxPath));
        
        //增加内容
        wordprocessingMLPackage.getMainDocumentPart().addParagraphOfText("你好!");
        wordprocessingMLPackage.getMainDocumentPart().addStyledParagraphOfText("Title", "这是标题!");
        wordprocessingMLPackage.getMainDocumentPart().addStyledParagraphOfText("Subtitle", " 这是二级标题!");
        
        wordprocessingMLPackage.getMainDocumentPart().addStyledParagraphOfText("Subject", "试一试");
        //保存文档
        wordprocessingMLPackage.save(new File(docxPath));
    } catch (Docx4JException e) {
        logger.error("addParagraph to docx error: Docx4JException", e);
    }
}

```

4.读取docx文件(这里不支持doc文件)

```
/**
 * 读取word文件，这里没有区分 word中的样式格式
 */
public static void readParagraph(String path) {
	try {
        WordprocessingMLPackage wordprocessingMLPackage = getWordprocessingMLPackage(path);

        String contentType = wordprocessingMLPackage.getContentType();
        logger.info("contentType:"+contentType);
        MainDocumentPart part = wordprocessingMLPackage.getMainDocumentPart();
        List<Object> list = part.getContent();
        System.out.println("content -> body -> "+part.getContents().getBody().toString());
        for(Object o :list) {
        	logger.info("info:"+o);
        	
        }
	}catch(Exception e) {
		
	}
}

```

1. docx转 html文件
   样式表可以自行修改

   ```
   /**
    * 转xls,在转html
    */
   public static void wordToHtml() {
   	boolean nestLists = true;
   	boolean save = true;
       WordprocessingMLPackage wordprocessingMLPackage = getWordprocessingMLPackage(docxPath);
       
       HTMLSettings html = Docx4J.createHTMLSettings();
       //设置图片的目录地址
       html.setImageDirPath(docxPath + "_files");
       html.setImageTargetUri(docxPath.substring(docxPath.lastIndexOf("/") + 1 ) + "_files");
       html.setWmlPackage(wordprocessingMLPackage);
       String userCSS = null;
       if (nestLists) {
           userCSS = "html, body, div, span, h1, h2, h3, h4, h5, h6, p, a, img,  table, caption, tbody, tfoot, thead, tr, th, td "
                   + "{ margin: 0; padding: 0; border: 0;}" + "body {line-height: 1;} ";
       } else {
           userCSS = "html, body, div, span, h1, h2, h3, h4, h5, h6, p, a, img,  ol, ul, li, table, caption, tbody, tfoot, thead, tr, th, td "
                   + "{ margin: 0; padding: 0; border: 0;}" + "body {line-height: 1;} ";
       }
       html.setUserCSS(userCSS);
       OutputStream os = null;
       try {
       	if (save) {
               os = new FileOutputStream(docxPath + ".html");
           } else {
               os = new ByteArrayOutputStream();
           }
       	//设置输出
           Docx4jProperties.setProperty("docx4j.Convert.Out.HTML.OutputMethodXML", true);
   
           Docx4J.toHTML(html, os, Docx4J.FLAG_EXPORT_PREFER_XSL);
       }catch(Exception e) {
       	
       }
       if (save) {
           System.out.println("Saved: " + docxPath + ".html ");
       } else {
           System.out.println(((ByteArrayOutputStream) os).toString());
       }
       if (wordprocessingMLPackage.getMainDocumentPart().getFontTablePart() != null) {
       	wordprocessingMLPackage.getMainDocumentPart().getFontTablePart().deleteEmbeddedFontTempFiles();
       }
       html = null;
       wordprocessingMLPackage = null;
   }
   
   ```

   

1. 插入图片

```
public static void wordInsertImage() {
    WordprocessingMLPackage wordprocessingMLPackage = getWordprocessingMLPackage(docxPath);
    File file = new File("/Users/chenhailong/Desktop/avatar.png");
    try {
		byte[] bytes = DocImageHandler.convertImageToByteArray(file);
		DocImageHandler.addImageToPackage(wordprocessingMLPackage, bytes);
		wordprocessingMLPackage.save(new File(docxPath));
	} catch (FileNotFoundException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} catch (Exception e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}

```

图片工具类

```
package com.chl.tools;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;

import org.docx4j.wml.ObjectFactory;
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.openpackaging.parts.WordprocessingML.BinaryPartAbstractImage;
import org.docx4j.wml.Drawing;
import org.docx4j.wml.P;
import org.docx4j.wml.R;

/**
 * 插入图片工具类
 * @author chenhailong
 *
 */
public class DocImageHandler {

	/**
	 * 将图片加入到包中
	 * @param wordMLPackage
	 * @param bytes
	 * @throws Exception
	 */
	public static void addImageToPackage(WordprocessingMLPackage wordMLPackage, byte[] bytes) throws Exception {
		BinaryPartAbstractImage imagePart = BinaryPartAbstractImage.createImagePart(wordMLPackage, bytes);

		int docPrId = 1;
		int cNvPrId = 2;
		org.docx4j.dml.wordprocessingDrawing.Inline inline = imagePart.createImageInline("Filename hint",
				"Alternative text", docPrId, cNvPrId, false);

		P paragraph = addInlineImageToParagraph(inline);

		wordMLPackage.getMainDocumentPart().addObject(paragraph);
	}

	/**
	 * 将图片加入到 段落中去
	 * 
	 * @param inline
	 * @return
	 */
	public static P addInlineImageToParagraph(org.docx4j.dml.wordprocessingDrawing.Inline inline) {
		// 添加内联对象到一个段落中
		ObjectFactory factory = new ObjectFactory();
		P paragraph = factory.createP();
		R run = factory.createR();
		paragraph.getContent().add(run);
		Drawing drawing = factory.createDrawing();
		run.getContent().add(drawing);
		drawing.getAnchorOrInline().add(inline);
		return paragraph;
	}
	
	/**
	 * 将图片转换为字节数组
	 * 
	 * @param file 文件流
	 * @return
	 * @throws FileNotFoundException
	 * @throws IOException
	 */
	public static byte[] convertImageToByteArray(File file) throws FileNotFoundException, IOException {
		InputStream is = new FileInputStream(file);
		long length = file.length();
		// 不能使用long类型创建数组, 需要用int类型.
		if (length > Integer.MAX_VALUE) {
			System.out.println("File too large!!");
		}
		byte[] bytes = new byte[(int) length];
		int offset = 0;
		int numRead = 0;
		while (offset < bytes.length && (numRead = is.read(bytes, offset, bytes.length - offset)) >= 0) {
			offset += numRead;
		}
		// 确认所有的字节都没读取
		if (offset < bytes.length) {
			System.out.println("Could not completely read file " + file.getName());
		}
		is.close();
		return bytes;
	}
}


```

6.导入类
项目引用很多jar,要引入正确的类

```
import org.docx4j.Docx4J;
import org.docx4j.Docx4jProperties;
import org.docx4j.convert.out.HTMLSettings;
import org.docx4j.jaxb.Context;
import org.docx4j.openpackaging.exceptions.Docx4JException;
import org.docx4j.openpackaging.exceptions.InvalidFormatException;
import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
import org.docx4j.openpackaging.parts.WordprocessingML.MainDocumentPart;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

```

