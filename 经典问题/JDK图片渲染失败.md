确认操作系统的字体配置是否正确。可以通过以下命令查看系统字体是否可用：

fc-list

检查JDK是否正确配置了字体。可以尝试使用以下命令打印JDK当前配置的字体：

java -jar fontlist.jar

其中fontlist.jar是一个可以从Oracle官网下载的工具，用于打印JDK当前配置的字体。

如果仍然存在问题，请尝试在JDK启动命令中增加如下参数，以强制JDK使用本地系统字体配置：

-Dsun.java2d.fontpath=/usr/share/fonts

如果您的系统字体配置不在/usr/share/fonts下，请修改该路径为正确的字体配置路径。  


可以通过以下步骤检查JDK是否正确配置了字体：    

运行Java程序，如果程序中涉及到字体的显示，观察显示效果是否正确。  
在Java程序中使用java.awt.GraphicsEnvironment类获取系统中可用的字体，并输出字体名称。  
在命令行中运行java -jar <your_jar_file>.jar -Djava.awt.headless=true命令，查看是否会输出错误信息，如找不到字体等。  
在Java程序中使用java.awt.Font.createFont()方法动态加载字体文件，并输出字体名称和字体文件路径。  
通过以上步骤，可以判断JDK是否正确配置了字体。如果出现了字体相关的错误，可以根据错误信息进行相应的调整。  


## 具体检查步骤：
如果没有找到可用的字体，可能是因为字体配置文件未被正确加载。可以尝试以下方法来解决：  
确认系统中是否安装了字体配置工具（如fontconfig）以及相关的字体文件。  
确认字体配置文件的路径是否正确，并且在Java程序中是否正确指定了该路径。  
检查Java程序是否有足够的权限来访问字体配置文件和相关的字体文件。  
如果使用的是非标准的字体文件格式（如TrueType Collection），则需要确保Java程序支持该格式。  
尝试重新安装字体配置工具和相关的字体文件，并重新启动Java程序。  


在命令行中运行java -jar <your_jar_file>.jar -Djava.awt.headless=true命令，查看是否会输出错误信息，如找不到字体等。  
在Java程序中使用java.awt.Font.createFont()方法动态加载字体文件，并输出字体名称和字体文件路径。  
如果以上方法都不能解决问题，可能需要进一步检查系统和Java程序的配置，或者尝试在其他系统上测试同样的程序，以确定问题的具体原因。  

1、检查字体是否正确加载：  

```
import java.awt.GraphicsEnvironment;

public class CheckFontConfig {
    public static void main(String[] args) {
        // 检查系统中可用的字体名称
        String[] fontFamilies = GraphicsEnvironment.getLocalGraphicsEnvironment().getAvailableFontFamilyNames();
        for (String fontFamily : fontFamilies) {
            System.out.println(fontFamily);
        }
        // 尝试动态加载字体文件
        try {
            String fontPath = "/path/to/your/font.ttf"; // 字体文件的路径
            Font customFont = Font.createFont(Font.TRUETYPE_FONT, new File(fontPath));
            GraphicsEnvironment.getLocalGraphicsEnvironment().registerFont(customFont);
            System.out.println("Custom font loaded successfully.");
        } catch (Exception e) {
            System.out.println("Failed to load custom font: " + e.getMessage());
        }
        // 再次检查可用的字体名称，看是否包含自定义字体名称
        fontFamilies = GraphicsEnvironment.getLocalGraphicsEnvironment().getAvailableFontFamilyNames();
        boolean hasCustomFont = false;
        for (String fontFamily : fontFamilies) {
            if (fontFamily.equals("Your Custom Font Name")) {
                hasCustomFont = true;
                break;
            }
        }
        if (hasCustomFont) {
            System.out.println("Custom font is available.");
        } else {
            System.out.println("Custom font is not available.");
        }
    }
}
```
在这个示例程序中，首先使用 GraphicsEnvironment.getLocalGraphicsEnvironment().getAvailableFontFamilyNames() 方法获取系统中可用的字体名称，并输出到控制台上。
然后，尝试动态加载一个字体文件，并注册到 GraphicsEnvironment 中，如果加载成功，则输出提示信息到控制台上。
最后，再次使用 getAvailableFontFamilyNames() 方法检查可用的字体名称，看是否包含自定义字体名称，如果包含，则输出提示信息到控制台上。如果不包含，则可能是字体配置文件未被正确加载，需要根据具体情况进行排查。

2、确认字体配置文件的路径是否正确，并且在Java程序中是否正确指定了该路径。
确认字体配置文件的路径是否正确。字体配置文件通常位于/etc/fonts/fonts.conf或者~/.fonts.conf文件中。可以使用以下命令检查字体配置文件的路径：
fc-config -c
在Java程序中指定字体配置文件的路径。可以使用以下代码将字体配置文件的路径设置到Java程序中：
System.setProperty("sun.awt.fontconfig", "/path/to/fonts.conf");
其中，/path/to/fonts.conf应替换为字体配置文件的实际路径。可以将此代码添加到Java程序的入口点（例如main方法）之前，确保在程序中正确加载字体配置文件。
3、确认系统中是否安装了字体配置工具（如fontconfig）
在Linux系统中，可以使用命令行工具fc-list查看系统中已安装的字体列表。如果系统中已经安装了fontconfig，那么该命令应该可以正常使用。如果该命令无法使用，则可能是系统中未安装fontconfig。


在部分linux环境中，使用zxjdk绘图时（如验证码加载），报出空指针异常：

Exception in thread "main" java.lang.NullPointerException
    at sun.awt.FontConfiguration.getVersion(FontConfiguration.java:1264)

X86可能存在两种情况：

1、该问题在cgslv5中出现，V4并不能复现该问题，经定位，发现机器缺少很多32位的包，即（i686）的包，在该环境中，主要是缺少了fontconfig的包导致。

yum install -y fontconfig.i686

2、在CGSLV6/RedHat8等glibc 2.28版本的linux系统上，zxjdk8u181/zxjdk8u252存在渲染故障，请先替换zxjdk8u212/zxjdk8u272的版本（该情况请跳过下述处理步骤），执行以下语句确认glibc版本号

ldd --version

ARM可能存在以下情况：

1、操作系统使用了高版本的libpng（已知在CGSLV6.06.02P2B5-aarch64版本）。此情况可以libpng15.so.15.13.0 拷贝到/usr/lib64目录下，并执行以下操作：

ln -s /usr/lib64/libpng15.so.15.13.0 /usr/lib64/libpng15.so.15

2、部分ARM系统（如TAG_CGS_MAIN_V6_06_02B5P2B8）需要手动为该文件赋权：

chmod 755 /usr/lib64/libpng15.so.15.13.0

一、如果是CGSLV6/RedHat8等glibc 2.28版本的linux系统，则检查jdk版本，看jdk版本是否为181或者252，如是，则按以下操作：

方式一（推荐）：升级jdk版本；

方式二（临时规避）：

1、在 $JAVA_HOME/lib中增加了fontconfig.properties里面增加

version=1 
sequence.allfonts=default

2、删除 jre/lib/amd64/libfreetype.so.6



二、如果是其他版本，则按照下面步骤检查：

1、确认提供验证码服务所在的服务器（如装在docker上，则需要进入docker环境中）

2、确认操作系统版本

3、确认是否安装了fontconfig

如cgsl/centos/redhat使用以下命令：

yum list installed | grep fontconfig

alpine系统需要自行查询

4、如64位操作系统安装了32位的jdk，还需要确认fontconfig是否安装了32位的，（这种情况下建议换同位的jdk）

5、安装fontconfig

1）、如果能连上外网，各个操作系统对应了自己的yum库，默认情况下直接用yum install fontconfig命令直接安装即可

2）、如果yum库无法连上，则需要自行根据操作系统及版本号，

6、安装完成后，执行fc-cache命令

7、重启应用


如果在alpine操作系统中，除了缺少fontconfig外，还缺少字体库，以3.8内网为例：

1、vi /etc/apk/repositories

增加


2、apk update

3、apk add --update font-adobe-100dpi ttf-dejavu fontconfig

4、fc-cache


