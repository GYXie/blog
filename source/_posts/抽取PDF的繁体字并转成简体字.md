title: 抽取PDF的繁体字并转成简体字
date: 2016-09-23 03:05:50
categories: 自动化
tags:
  - Java
  - Python
  - 自动化
---
*本文为作者原创文章，转载请注明出处。*

## 场景
手头上有一个PDF，里面的文字是繁体字，现在需要使用这些文字，但是必须是简体字。

## 解决方案
1. 首先抽取PDF中文字
    使用Apache的[Pdfbox](https://pdfbox.apache.org/)包。这个包是进行PDF处理的很好的解决方案，目前这个包还在不断更新中。
    ```java
    import org.apache.pdfbox.pdfparser.PDFParser;
    import org.apache.pdfbox.pdmodel.PDDocument;
    import org.apache.pdfbox.util.PDFTextStripper;

    import java.io.*;

    public class PdfboxUtil {

        /**
        * @param args
        */
        public static void main(String[] args) {
            String pdfPath = "/Users/xxx/Downloads/xxx.pdf";
            String txtfilePath = "/Users/xxx/Downloads/xxx.txt";
            PdfboxUtil pdfutil = new PdfboxUtil();
            try {
                String content = pdfutil.getTextFromPdf(pdfPath);
                pdfutil.toTextFile(content, txtfilePath);
                System.out.println("Finished !");
            } catch (Exception e) {
                e.printStackTrace();
            }

        }

        /**
        * 读取PDF文件的文字内容
        * @param pdfPath
        * @throws Exception
        */
        public String getTextFromPdf(String pdfPath) throws Exception {
            // 是否排序
            boolean sort = false;
            // 开始提取页数
            int startPage = 1;
            // 结束提取页数
            int endPage = Integer.MAX_VALUE;

            String content = null;
            InputStream input = null;
            File pdfFile = new File(pdfPath);
            PDDocument document = null;
            try {
                input = new FileInputStream(pdfFile);
                // 加载 pdf 文档
                PDFParser parser = new PDFParser(input);
                parser.parse();
                document = parser.getPDDocument();
                // 获取内容信息
                PDFTextStripper pts = new PDFTextStripper();
                pts.setSortByPosition(sort);

                endPage = document.getNumberOfPages();
                System.out.println("Total Page: " + endPage);
                StringBuilder sb = new StringBuilder();
                for(int i = 1; i<= endPage; i++){
                    pts.setStartPage(i);
                    pts.setEndPage(i);
                    try {
                        String page = pts.getText(document);
                        sb.append("\n\n###############################第"+ i +"页###############################\n\n");
                        sb.append(page);
                    } catch (Exception e) {
                        throw e;
                    }
                }
                content = sb.toString();
            } catch (Exception e) {
                throw e;
            } finally {
                if (null != input)
                    input.close();
                if (null != document)
                    document.close();
            }

            return content;
        }

        /**
        * 把PDF文件内容写入到txt文件中
        * @param pdfContent
        * @param filePath
        */
        public void toTextFile(String pdfContent,String filePath) {
            try {
                File f = new File(filePath);
                if (!f.exists()) {
                    f.createNewFile();
                }
                System.out.println("Write PDF Content to txt file ...");
                BufferedWriter output = new BufferedWriter(new FileWriter(f));
                output.write(pdfContent);
                output.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    ```
    *注意这段代码使用的1.8.10版本的pdfbox，使用2.x以上版本有兼容问题*
    贴出maven依赖，省得大家找了。
    ```
    <dependency>
        <groupId>org.apache.pdfbox</groupId>
        <artifactId>pdfbox</artifactId>
        <version>1.8.10</version>
    </dependency>
    ```

2. 繁体字转简体
[OpenCC](https://github.com/BYVoid/OpenCC)是一个开源python包，可以实现简繁体字互转。
```python
# -*- encoding=utf-8 -*-
from opencc import OpenCC

openCC = OpenCC('t2s')

input_file = open("xxx.txt", "r")
output_file = open("xxx_sim.txt", "w")
for l in input_file.readlines():
	# l = l.strip()
	print(l)
	converted = openCC.convert(l)
	output_file.write(converted)
	print(converted)

input_file.close()
output_file.close()
```

## 工具化
上面的方案采用了java+python两种编程语言，代码也写死了。如果经常有这个需要，可以考虑做一个工具，使用统一的语言。比如，将pdf抽取文字的工作也用python实现（可以用pdfminer包）。

