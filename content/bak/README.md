### 23.1 Customizing the Banner
启动时打印的banner可以通过向类路径中添加`banner.txt`文件或通过将`banner.location`设置为这样一个文件的位置来更改。
如果文件有不寻常的编码，你可以设置`banner.charset`。字符集(默认为`UTF-8` )。

* ${application.version}：MANIFEST.MF文件中的版本号,例如 `Implementation-Version: 1.0` 打印为 `1.0`.

* ${application.formatted-version}：格式化后的MANIFEST.MF文件中的版本号,(用括号括起来，前缀为v). 例如 `(v1.0)`.

* ${spring-boot.version}：正在使用的SpringBoot版本. 例如 `1.3.7.RELEASE`.

* ${spring-boot.formatted-version}：格式化后的正在使用的SpringBoot版本.,(用括号括起来，前缀为v). 例如 `(v1.3.7.RELEASE)`.

* ${Ansi.NAME} (or ${AnsiColor.NAME}, ${AnsiBackground.NAME}, ${AnsiStyle.NAME})： NAME是一个ANSI 码. 详情看 [AnsiPropertySource](https://github.com/spring-projects/spring-boot/blob/v1.3.7.RELEASE/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java).

* ${application.title}：：MANIFEST.MF文件中的标题. 例如 Implementation-Title: MyApp 展示为 MyApp.


您还可以使用spring.main.banner模式属性来确定是否必须使用配置的日志记录器(log)在System.out (console)上打印横幅，或者根本不打印横幅(off)。


>如果你想关闭banner,同样可以在YAML配置off或者false来关闭。

```
spring:
    main:
        banner-mode: "off"

```
