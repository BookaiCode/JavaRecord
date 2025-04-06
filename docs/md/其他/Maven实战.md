![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfaaUxnw8IDuicNESzxXE0vOCwHptdhCz9B6QQtOWfGdeuNPelQbYqxxg/640?wx_fmt=png&amp;from=appmsg)

# Maven实战

## 一、Maven介绍

### 1.1 现存问题

jar包问题

* jar包需要在本地保存，而且在使用的时候需要将jar复制到项目中，再build才可以生效。
* jar包的体量不小，一个项目中可能需要上百的jar的支持，这样一个项目就太大了。
* 如果jar包的版本需要升级，需要重新去搜集新版本的jar包，重新去build，时间成本太高了。
* 做一些功能时，可能需要因为几个，甚至十几个jar包，才能完成一个功能，都需要自己维护，甚至记住。

项目结构的问题

* 之前开发工具很多，有Eclipse，MyEclipse，IDEA，VSCode等等……不同的开发工具的项目的结构会有一些不同，多人协同开发时，就会造成冲突，甚至还需要统一开发工具。

整体项目的生命流程

* 整个项目从立项开发，到最后的发布上线到生产环境，没有一套统一的流程来控制。

### 1.2 Maven

- Maven可以帮助我们更好地去管理jar包，只需要指定好jar的一些基本的标识，就可以让jar包支持我们的项目。而且Maven可以帮助咱们导入一个jar包后，自动将和它绑定好的其他jar包引入。
- Maven可以提供一个统一的项目结构。
- Maven也对整体项目的生命周期有响应的管理，从开始的编译、测试、打包、部署等操作，都提供了相应的支持。
- Maven还提供了分模块开发的功能。

Maven是apache组织的一个顶级开源项目。  http://maven.apache.org

## 二、Maven安装&环境变量配置

### 2.1 Maven的安装

首先下载Maven，直接去官网即可

在点击Download之后，需要注意看一下对JDK版本的支持。

Maven需要JDK的环境变量支持，一定要看一下自己又没有设置上JAVA_HOME

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfQFcQNzsTx4Ga7xianbEIPEJxnaH8ibj2HDMNwWMcn0cMe5H6NAsGRxnw/640?wx_fmt=png&amp;from=appmsg)

需要根据自己的环境变量，下载对应的压缩包。

Linux、Mac选择.tar.gz的压缩包

Windows选择zip的压缩包

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfUIITxI3eZkQTPlAYXSB2gKsua68braYNnyXwcQVkUva7Lr1Jv9heMg/640?wx_fmt=png&amp;from=appmsg)

下载好之后，得到一个压缩包。

解压的目录最好没有任何的中文和空格等特殊字符。推荐放到磁盘的根目录下即可。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfOWicdcoIEQibfats3LyAvORzE9D35Pb4qaNmHYrJQFlR1tRFUR2sMshg/640?wx_fmt=png&amp;from=appmsg)

> bin：含有mvn运行的脚本。
>
> boot：含有类加载器框架，Maven使用这个框架来加载自己的类库。
>
> conf：含有非常核心的settings.xml文件。
>
> lib：含有Maven运行时需要的一些类库。

### 2.2 Maven的环境变量的配置

首先配置Maven的环境变量前，必须先查看一下JDK环境变量配置。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tftwTSkK9RXff4d6ibDHJVuOicRgWup6E85sOYtUzTOuFkpeicZCEEKq1icA/640?wx_fmt=png&amp;from=appmsg)

其次，查看一下前面说过的JAVA_HOME。

上述两点都ok的话，直接开始配置环境变量

* 配置MAVEN_HOME![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfIapibng6RhD67b8WVboH8vQeQQbvHMtgVx5QwhVhC0ib8vBHTC4NjNJA/640?wx_fmt=png&amp;from=appmsg)
* 配置到path![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfl15jvKDGxlsJIVLrBdJWYZc5jW5RgJ5TQQib3aK7kvtOrFCMEnh2LNA/640?wx_fmt=png&amp;from=appmsg)

**配置完毕后，记得重新打开一下cmd窗口。别直接在之前的cmd窗口测试**。

在cmd窗口执行mvn -v

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tffGpHQds8Jk1ONP8k5pPKPtVqD4TcNGWceC4YF7k1XLW3KlLJl5z1yA/640?wx_fmt=png&amp;from=appmsg)

> Ps：常见错误，没有配置正确的JAVA_HOME

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfto4PQfHW3F3eZPzqqZvwjk5JAt56H8ODN1qOiauoO3Ngcic27yxNibg6w/640?wx_fmt=png&amp;from=appmsg)

## 三、仓库&settings.xml配置（重要）

### 3.1 仓库

Maven可以帮助咱们管理jar文件，但是，jar包是需要从网上下载下来的。

仓库很多，有官方的中央仓库，还有国内公司的仓库，还有公司内部会搭建的私服

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfB4U2Vbqn7CkqalibtOOKC28TBANS4SjqU8hxDfLOefFuiaFAU0lBicOow/640?wx_fmt=png&amp;from=appmsg)

咱们后面需要配置好国内公司的一些仓库。

### 3.2 settings.xml配置（重要）

在MAVEN_HOME目录下，有一个conf目录。在conf目录下就有需要修改的settings.xml文件。

需要修改三点内容

#### 3.2.1 本地仓库地址

默认情况下，本地仓库在C盘。

> Default: ${user.home}/.m2/repository

根据配置文件中的注释，默认是仍在用户目录下的.m2目录下的repository目录中。

这个本地仓库会随着项目越来越多，这个仓库也会越来越大。可能会占用10多个G，甚至更多。

所以推荐放在系统盘之外。（如果就C盘，那就用默认的吧…………）

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfTO2L2yqCOmCnJ97t2HWPJfhR0V7CfaF6vTu2icXSyOcs68C83rgrqbQ/640?wx_fmt=png&amp;from=appmsg)

#### 3.2.2 配置阿里云/华为云……仓库

配置阿里云仓库

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfx5G3xZIOpZFDwTCbiaLIH9k5cJ5WguUceuZKyd1QEKOmu26tBOQ99dg/640?wx_fmt=png&amp;from=appmsg)

```xml
<!-- 配置远程仓库地址 -->
<mirrors>
  <mirror>
    <id>aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>
</mirrors>
```

华为云的仓库地址：`https://repo.huaweicloud.com/repository/maven/`

#### 3.2.3 JDK编译版本配置

Maven默认采用JDK1.5的编译方式去编译项目。
为了让Maven支持现在JDK的编译版本，可以指定一下现在采用JDK1.8

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfLKU1iabIV7BJfNXAFysiap4LfEibrHQIDnp7jA0CxMecSicOQ6UH4eza5g/640?wx_fmt=png&amp;from=appmsg)

```xml
<!-- 配置JDK的编译版本 -->
<profiles>
  <profile>  
    <id>jdk1.8</id>  
    <activation>  
      <activeByDefault>true</activeByDefault>  
      <jdk>1.8</jdk>  
    </activation>  
    <properties>  
      <maven.compiler.source>1.8</maven.compiler.source>  
      <maven.compiler.target>1.8</maven.compiler.target>  
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
    </properties>  
  </profile>
</profiles>
<!-- 开启前面的profile -->
<activeProfiles>
  <activeProfile>jdk1.8</activeProfile>
</activeProfiles>
```

## 四、IDEA配置Maven

**先看老版本的，再看新版本的！！！**

### 4.1 2019.1.3 IDEA配置Maven

打开IDEA的初始窗口

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfMUsm8qlLPkRHIUk3V6H3oDXnXRO6giaRP2LI7Lj8HTSxFZZVykv9o5Q/640?wx_fmt=png&amp;from=appmsg)

右下角的Configure的位置打开settings，点开后，在左上角可以看到是Settings for New Projects

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfWJWokGI1o3ibPBKyMNlpaVVcCbvM5oQ4yzTukDgUtGz1MBZ00D9Aoqg/640?wx_fmt=png&amp;from=appmsg)

因为IDEA版本的原因，对Maven的版本也是有要求的。

比如现在的2019.1.3的IDEA版本，无法支撑3.6.1以上的Maven版本

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfD8wqEoiapGrYObGicPTibwy1JqgwkRiaN3yGycYE9oUhd6UBhkb7CBJ3QA/640?wx_fmt=png&amp;from=appmsg)

一定要记得，点击Apply，然后ok，确认生效。

### 4.2 2024.1 IDEA配置Maven

首先一定要记住，选择Settings for new projects

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf0nZ8wBAOQXjTPdnCk1dEyQQp8noSutzD0BrdvxgPAlFKgjvqW7PFEA/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfXeNQ0sOiaHliaB2BQPsaPVCONZTno4sPgZicsiaSHms3mY3k4tvwpT23yA/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfw6tDNVk4FAVLGN2xTuqLKicQ00yWq4wTXbniciaPr13vfKNRNLhG0aOIA/640?wx_fmt=png&amp;from=appmsg)

## 五、IDEA构建Maven项目

**先看老版本的，再看新版本的！！！**

### 5.1 2019.1.3 IDEA构建Maven项目

点击Create New Project

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf3KbIc02XzR0RBpxKeraYMeGfzDsYvIbLVYDwrg5uX7I5CI8DnjEhPA/640?wx_fmt=png&amp;from=appmsg)

next后，指定当前项目的三围，包名，项目名，版本号

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf98IQwUdg2eBpicQ8eE0LUkwIupWwqx53YjwAlLC707qaj4FsmE9tdNw/640?wx_fmt=png&amp;from=appmsg)

指定好项目名和存放地址。这里对存放地址修改一下就ok。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfXEtGUPwKT9icIOicU9HYOxUTpt5tk51hDgflrjicGaHdoMQ7T50IH09xA/640?wx_fmt=png&amp;from=appmsg)

指定好之后，点击Finish即可。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf7IFvQ5ptxrVLGykDWjYUGruhuaeM3UElPVljCJ6mvdfm6qR6bGtDicQ/640?wx_fmt=png&amp;from=appmsg)

进来后，可以看到右下角的进度条，在下载一些Maven必要的插件

在下载插件时，可能需要一定的时间，等插件下载好，为了确认咱们阿里云私服的配置是否生效，随便复制下面内容到当前位置。  **一定一定一定记得点击右下角的import Changes**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.1.6.RELEASE</version>
    </dependency>
</dependencies>
```

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfRwqpxbWwESricHicic12RZXHmy4Hia7ibF2TmmYiaJveNlPgAptcl70y5Ktw/640?wx_fmt=png&amp;from=appmsg)

快速地点击右下角的进度条，查看下载的链接地址，确认一下是否是阿里云的地址

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfrMF7OXYeOhDTsFrufy9BS15GoqeDMjELyyE2Vef5CCwKeMqSlqTibtA/640?wx_fmt=png&amp;from=appmsg)

再次查看右侧的Maven栏，确认profiles中的JDK1.8编译版本已经生效

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfiaz54Llbic4GyqkXtzJHiaYS8OZiaYnjXl96yVSh1rXnVheVSqCCFwV9AQ/640?wx_fmt=png&amp;from=appmsg)

最后查看完毕后，要对Maven项目的目录结构有个了解

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfU7SRWUK6rtCxWGMzD7fn2alKo6qbDRdOOVoic8uib7n2H3ToNygPXqqg/640?wx_fmt=png&amp;from=appmsg)

### 5.2 2014.1 IDEA构建Maven项目

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfSvM2T5G52mabcS1kNuhkVTGduhVuVPts7QXMFeibsm5nRr5ia6FqibRicg/640?wx_fmt=png&amp;from=appmsg)

### 5.3 IDEA构建Maven的Web项目

这个新老版本是一致的！！！

这里是先构建Maven的基础项目，然后将基础项目修改为Web项目。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf5BbAOXRVfXGGsZiafgRiaPIUQ67VlgicaGr6ysuQOL7bYJibJuyNFvypIw/640?wx_fmt=png&amp;from=appmsg)

正常，构建的基础maven项目，打包的方式是jar文件。需要将当前web项目的打包方式修改为war的形式。

需要修改pom.xml文件指定打包方式。

默认情况下，这个packaging是jar的打包形式。需要指定好war的形式，一定一定一定记得import Changes

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tficicDsIPcrfznuf3LNP4IcI47wfAur5HHg1XsPkual82XUhQk74wmvtg/640?wx_fmt=png&amp;from=appmsg)

然后选中项目，点击左上角的file，选择Project Structure

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfUm0Ngx8UMPjic3eJWsN1DUlBlrL4laW5ViaaicTmpHdpcqicXJxicW1JT2w/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfh1D90QD3Via1Y7HSz4yuYDlkLVdZBw4FAs1vAG4GQ5EEpqibPM6PiavjQ/640?wx_fmt=png&amp;from=appmsg)

选择左侧导航栏中的facets选项，如果你的Facets界面没有这个Web，说明之前的war没配置好！！

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfLznicInlHbZZ42I5jb3xKbeGKVT3XzV7LYHxhowdtTZTk3ryMoAyRxQ/640?wx_fmt=png&amp;from=appmsg)

然后点击右上角的+，追加一个web.xml文件，记得一定要放到webapp资源目录下

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfzCQyRsgTkrchqC3zRtqvoa0AGrBKC83N0UtDBkb7kMIFLSEDiaNRJXg/640?wx_fmt=png&amp;from=appmsg)

点击ok，就会自动生成webapp目录，以及目录下的web.xml文件

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfpLMQ8TicvFCDx3vZdFM2YZMSDls2E1znUhPno9oKQtlUlaWibgGLug1A/640?wx_fmt=png&amp;from=appmsg)

## 六、导入依赖jar（重要）

创建好Maven项目之后，需要导入具体的jar包时，要通过 **坐标** 导入

* 每个jar都需要三个内容形成一个唯一的坐标，需要groupId + artifactId + version导入一个具体的jar。
* 在maven项目中，只需要导入配置的坐标，Maven便会自动地去网上下载jar文件，并且添加到项目中。

当需要使用某个jar时，知道大概的名字，但是不会背下来具体的坐标信息，可以去一个地方搜索

[https://mvnrepository.com/](https://mvnrepository.com/)

可以去这个地址搜索具体的jar包坐标

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tficM7CwAK47BfdqMfiaLg9vr2PemPUJKgEgd4z1JjV7ibjLlebqubkVgFw/640?wx_fmt=png&amp;from=appmsg)

进入具体的依赖内部后，选择对应的版本

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfMQjBz5yWIjZARfHlOh4nicsNKicmB1vgfLFwLicNH4f8xp7Sr5zArkgrA/640?wx_fmt=png&amp;from=appmsg)

找到需要导入的dependency

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tficxAWicBF3mhMPoPupZdyabHw4TrtoqZjlqU0oicj1keQVuL52N4BibN0w/640?wx_fmt=png&amp;from=appmsg)

复制好之后，扔到项目的pom.xml文件中

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfqMPqZpyg3lCq4WlE7QjwubYtz750X0oibf1zz0sBKBNfTYDFQicWP0QQ/640?wx_fmt=png&amp;from=appmsg)

如果本地仓库出现了.lastUpdated后缀的文件，可能有两个情况

* 这个坐标的jar文件不存在
* 因为网络原因下载失败了

**这种.lastUpdated后缀的文件，会导致后续依赖下载失败，记得如果出现了依赖失败，检查坐标都没问题，并且也是走阿里云或者华为云去下载的，依然失败。记得去本地仓库看一下，是不是有.lastUpdated后缀的文件导致无法下载成功**！

## 七、依赖的作用域

所谓的依赖作用域就是当前这个jar文件在什么情况下，项目会使用到。

这个所谓的情况，可以分成三点来聊：

* 编译阶段
* 测试阶段
* 运行阶段

Maven中给予依赖五种作用域：

* compile（默认作用域）：编译，测试，运行都会提供当前依赖的功能
  ```
  <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.11.0</version>
  </dependency>
  ```
* provided：编译，测试会提供当前依赖的功能。 一般Servlet，JSP会涉及。
  ```
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
  </dependency>
  ```
* runtime：测试，运行会提供当前依赖的功能。一般MySQL会涉及。
  ```
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
      <scope>runtime</scope>
  </dependency>
  ```
* test：测试会提供当前依赖的功能。
  ```
  <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
  </dependency>
  ```
* system：不是在什么情况下用，这个比较特殊，是将一些本地仓库没有的jar文件，引入到当前项目。
  ```
  <dependency>
      <groupId>com.oracle.database.jdbc</groupId>
      <artifactId>ojdbc10</artifactId>
      <version>19.21.0.0</version>
      <scope>system</scope>
      <systemPath>D:/ojdbc10-19.21.0.0.jar</systemPath>
  </dependency>
  ```

**system，不推荐用，哪怕一些依赖，本地仓库无法下载，也别用system去引入。这种引入方式会导致后期打包还是更换了环境之后，无法使用。（后面咱们会根据maven的命令，可以将本地的jar包安装到本地仓库）**

```shell
mvn install:install-file  -Dfile=D:/ojdbc10-19.21.0.0.jar -DgroupId=laozheng -DartifactId=laozheng-oracle -Dversion=yeyeye -Dpackaging=jar
```

搞定后，本地仓库可以看到install的jar文件和路径

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfOOyibtIXcQr6aDJzZq82aAXjOxEdCt4JnOSP9ibdoXyZqZnRgbiaXb3jQ/640?wx_fmt=png&amp;from=appmsg)

然后就可以在项目中引用了。

```xml
<dependency>
    <groupId>laozheng</groupId>
    <artifactId>laozheng-oracle</artifactId>
    <version>yeyeye</version>
</dependency>
```

## 八、依赖冲突

首先，咱们要先了解一下Maven依赖的传递特性。

当咱们导入一个jar包后，如果这个jar为了完成一些功能，还需要其他的jar的功能。

比如有A，有B，其中A依赖了B。

咱们只需要导入A包，B会自动被依赖过来。优点大大的：

* 不需要刻意的去记导入A之后，还需要导入什么其他的依赖。
* 关于某个版本的A需要哪个版本的B也不需要关注。

上面是优点，但是也存在着一些问题。

当前项目 -> A -> B（1.0.0）

当前项目 -> C -> B（2.0.0）

此时，当前项目会出现相同的依赖，有两个，但是版本不一样，此时就会产生依赖冲突问题。

一般依赖冲突会在启动或者测试项目时，直接给你甩异常。而且这个依赖不太好处理。需要解决这种依赖冲突。

### 8.1 就近原则

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfAHwAsN3zB1FQiaOOsG2NWVhokiblxMa4lAgRst4RKhPlM8ImYFIqgZTg/640?wx_fmt=png&amp;from=appmsg)

明显，当前项目通过D依赖C的路径最近，基于就近原则，会使用2.0.0的版本![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfgDhQbY8YVicDKMthNSY90LDzpEics5lQLOtiaoFSH63Ck9FTwFnFmTg9Q/640?wx_fmt=png&amp;from=appmsg)

### 8.2 优先声明原则

当出现依赖传递导致相同jar包版本不一致时，此时会根据优先声明原则来决定使用谁。![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfQB2U6TVaSwOFf4Z9t1yBPVjF73oQxky43oDXXW2yvAsHuIRibcLupzg/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf2aiaTnjicKUWmniaYooFClwVPqCQ6K4pDh9XB6SeibeKWOuu8EFVm1Mnicg/640?wx_fmt=png&amp;from=appmsg)

如果是你主动导入的依赖，此时会根据你最后引用的版本决定![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfibgynfqcOeQjqonNwib6BUqD6lv10lS7dTvIouNNHc57oD6bOkuzueqA/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfdrsXGJjWGhsqZ8PZskP6vhtEBFBZB51O27un1YlheypSMTKgLPHl8w/640?wx_fmt=png&amp;from=appmsg)

### 8.3 手动排除依赖

可以手动的形式，在引入A依赖时，将B依赖中A依赖排除掉

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfIkU0N2CJ0FvtTVHiaoKkpqzIEteDZuicHpazb6yVxzq2eibZDiblIoML0A/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfk4PBgj8OkfvAtWAlgmtkuH3ibQPoSiby6CcOwqg9HoXMcJ5ibT9iaDVKOA/640?wx_fmt=png&amp;from=appmsg)

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.12</version>
        <exclusions>
            <exclusion>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
            </exclusion>
            <exclusion>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
</dependencies>

```

### 8.4 声明依赖版本

可以通过dependencyManagement标签，提前声明依赖的版本。

dependencyManagement标签只会声明版本，不会将依赖导入，导入依赖依然需要借助dependencies

配置完下面的内容后，再导入spring-beans、spring-core无论什么方式，都使用dependencyManagement中声明的版本。

```xml
<dependencyManagement>
    <dependencies>
        <!--只管理依赖的版本，不会导入依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.9</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

如果前面已经声明好了依赖的版本。

但是你在pom.xml文件中，直接引入了一个具体的版本的依赖，和dependencyManagement不一致，那么会使用你指定好的版本。这种依赖传递的版本会严格遵循dependencyManagement。

其次，如果基于dependencyManagement声明好了版本，在dependencies中导入依赖时，是可以不写版本号的，可以直接基于dependencyManagement中的版本导入。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfvHu5eC1D5MpTG2hnzloOnv9GGpl1RloK9U6FibzMqq9RoUZcPK2CC2g/640?wx_fmt=png&amp;from=appmsg)

## 九、Maven指令

Maven为整个项目生命周期的各个阶段，提供了各种各样的指令。

先了解常用的几个：

```java
mvn clean：清空target目录。
mvn compile：编译整个项目，生成到target
mvn test：专门针对test目录下的内容做测试
mvn package：会将当前项目打包，jar，war。
mvn install：将当前项目进行编译，测试，打包，并且将jar包安装到本地仓库。
// mvn deploy：私服的位置再讲
```

* compile：这里是将main目录下的内容编译，生成一个target目录，将编译后的内容全部放到target目录下，java和resources都可以称为classpath，因为编译后的内容都是放在classes目录下的。![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfJSpp1PxbT73I5wR208KzY95ZVbO55T1iaAwyLgdpcaa5z9Kz2XmYn1g/640?wx_fmt=png&amp;from=appmsg)
* clean：就是将编译后的内容全部清除掉。
* test：测试会优先进行编译，并且会针对test目录下以Test结尾的类中追加了@Test注解的方法运行测试，如果报错，控制台会有显示。直接Build失败。![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfhfrLlicMGbBxGwuOaCxMfVRU10fjsMmuf1c5A3IlY4hKGS3cBrHbHFw/640?wx_fmt=png&amp;from=appmsg)
* package：将项目进行打包，但是打包会经历compile以及test，并且成功后，才会将项目打包成具体的jar或者是war。打包后的具体文件，会存放在target目录下。项目打包无法跳过编译过程的，但是可以跳过测试的过程，需要自行敲命令![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfZkLaZKNhRXT98KGMqGuJ8DNTsHS5wQq79NVk4EGFCIsSwXJv470lxg/640?wx_fmt=png&amp;from=appmsg)

  ```
  mvn package -DskipTests
  ```

  ![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfL2EIgV737KPeGU072bZR9xO8GrefYxf7Gwwic7WHlbgE0mGDxQjg2hw/640?wx_fmt=png&amp;from=appmsg)
* install：将当前项目做好编译，测试，打包，并且将项目安装到本地仓库。如果安装到本地仓库的是一个jar包，其他项目就可以将这个jar依赖过来使用。!![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf972dwV8t4RDNfIgUZnYuoqHibndQoOpn2icP6r9C4XNF3rmLClFmI71Q/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfmACmKxfv9X2hONWiaWmYD5DzCaHxGfHiapOWpiagbpSbZWOe0kgLpozcg/640?wx_fmt=png&amp;from=appmsg)

## 十、聚合工程

在项目打包的方式中，前面聊过jar，还有war的形式。

除此之外，还有一个打包的形式，叫做pom。pom就是所谓的聚合工程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfRnHAYSvGEHZYBUTcPW6e7V1bRRNxHC7tOAgFqqPF1yMo2hsqkdPNibA/640?wx_fmt=png&amp;from=appmsg)

构建最外层的电商聚合工程，聚合工程不需要写任何的业务代码，它的目的就是管理其他的子工程

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tflxmTadFHbicfkk51foUU6yurOzd8YXu8ys34n30MBVUtvSawWPFibzmA/640?wx_fmt=png&amp;from=appmsg)

构建好聚合工程后，可以再构建子工程，流程如下。![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfic4jDiaRZ4BFcJ9ibBnMN0aTNXnCao7Fw9gvkdgjjWTArGNft6prrlLIQ/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfFZ0wETqJEn3cC9kzJ0tc01wcvwZhvmT2vvmYYKwgj9kzGaOLmn9dpw/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfVTPR6dKwmmib3Dz07nt3BqBlDxjprOsDO1aPV0AsQ6oRlcDUXYwg3eQ/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfdQyW8via4tiaQfI7Svyp1G6czft5vH2QGdPrBwOD4ia8uCYgFCibyBwYOA/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfdYIlI2bZzvicmaR4QSPZIHk1Zc3qv6wnfICL60W0EZLO4D8FATRQCPA/640?wx_fmt=png&amp;from=appmsg)

好处是可以在聚合工程内去管理依赖的版本。同时可以基于聚合工程做统一的多个项目的打包或者其他操作。而且拆分模块去写项目。

## 十一、Maven私服

### 11.1 Maven私服的概念

> * 私服是搭建在局域网的一种特殊的远程仓库，目的是代理远程仓库，让下载依赖的效率更高。
> * 有了私服之后，使用Maven需要下载依赖时，直接请求私服下载依赖，将私服中的依赖下载到本地仓库中。如果私服中没有具体依赖，私服会去外部的远程仓库下载。
> * 私服可以解决在业务做开发时，有一些内部的依赖，是中央仓库没有提供的，是公司开发人员自行封装的一些依赖。可以将公司自研的一些框架和依赖上传到私服中，让公司内部人员可以通过私服将这种依赖下载到本地仓库。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfTyFGbWcz9gw0JQ6zZTOUeREhrsHH3nBNDaAsaDTKcjTvedFGu1MjGA/640?wx_fmt=png&amp;from=appmsg)

> 搭建私服的方式非常多，Apache Archiva，Sonatype Nexus。 一般都会采用后者。

### 11.2 搭建Nexus私服

去官网下载最新的安装包。

http://www.sonatype.com

但是在官网想找到Download挺麻烦的，下载的话，直接进入到下面这个地址

https://help.sonatype.com/en/download.html

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfuoUampibiaLav3TEhfFql6cpDic7mXYIKvicPicvT5CfGbia0uib2sSYiaHLdw/640?wx_fmt=png&amp;from=appmsg)

下载完毕是一个zip的压缩包，最好解压到非系统盘的位置，路径不要带 **中文和空格** ！！！！

解压后，有两个目录

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfRzw5LXg8UNIPmlkDyZX70pC2cNaDhrt8v9kLVN3SXw5F4cHTjYrxoQ/640?wx_fmt=png&amp;from=appmsg)

进入到nexus-3.67.1-01目录下，再进入bin目录下。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfDIMdtPiaqIkBM4MpZsU4j4s7nlG999mc05fXprvSevXRNFsQGPmkSicQ/640?wx_fmt=png&amp;from=appmsg)

启动时，需要基于doc窗口去运行Nexus私服，但是一定要以 **超级管理员** 的身份打开cmd。![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfhibAGud10LuzjcYoC9IwBFhAADBxRJLQJ9wXLot7sazHyAkIPSm9ibIQ/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf6qBP1s729hVllDR2Lks7dE7TdmZntnD60VTXibXnhJ4NPsYGfBkVx3A/640?wx_fmt=png&amp;from=appmsg)

在bin目录下执行指定，访问外网慢的话，可能需要至少9~10分钟左右甚至更多。

```shell
nexus.exe /run
```

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tf9WFwk5eDMukxlFDFCSv1Fe52aPfdo7AujmgOFZXV6jHIJHewZggLCw/640?wx_fmt=png&amp;from=appmsg)

启动成功后，直接访问http://localhost:8081/

进入首页后，需要加载一小会，可以访问到首页，第一个要做的事情是登录

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfzzFh4H7ug7dHcuTMKAfeibfmjOZpujZA0L8juUgecdIb8NS75R0MVkQ/640?wx_fmt=png&amp;from=appmsg)

登录即可，默认用户名是admin，密码在下面图中的文件里

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfrW3vSH1uQu00skVWwOrSSROr9Jw5BUWFlH9BDMNichta7Yfial9MMOsw/640?wx_fmt=png&amp;from=appmsg)

登录成功后，第二步需要重新设置密码

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfQ3wiaephSxia4ic6wsz1tqx84wiazAKud6c0iafHNcIYK7jUIMQBl6HCQcA/640?wx_fmt=png&amp;from=appmsg)

设置私服下载依赖的权限信息

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfNID5IXYE8jZSxskyUTFgZmKkFgSAMCeFEJOsL2wCPILrvIO3la5FzA/640?wx_fmt=png&amp;from=appmsg)

关注前四个即可

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfj7WJYicj5WQQtGGjxKRsJO21g51FtZVXO67cut3lNOyb1agB8d82ecA/640?wx_fmt=png&amp;from=appmsg)

### 11.3 Nexus私服配置&下载依赖

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfaGkmzTvXrF51RT8hWvU6kpeuAziazwqhnRWF28XZpX4cniajnyTKbfsw/640?wx_fmt=png&amp;from=appmsg)

将私服仓库的代理，设置为国内的仓库镜像源

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfYUicOqvGgwSVoCZxeYjCibDMQhsCfUjg1o9hu6ibaJqiaJnw48VOmPaGicw/640?wx_fmt=png&amp;from=appmsg)![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfWsIewf6mmCzsDOYr7QDLzf7jvfNCtARCNmMnOzVn6Ee16OlVHT1x5Q/640?wx_fmt=png&amp;from=appmsg)

配置完，拉到最下面，记得Save保存一下。

接下来配置好私服的地址，让项目基于私服下载依赖

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfSnJJS0IHXBMybeCeSFtXxm9WEbXBdeQ7RETvHlQFmR1kxq5tCvwic3g/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfTAv5edYxXhl9g08jw32pxfic5zbRLeJKp4DDTupBZpAoLb30SfZgDuA/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfSjnnuaYyNibGNSfJBLVEk40dwdaDmBdnlZWLJ90kOYJ8swhEqdLdeOw/640?wx_fmt=png&amp;from=appmsg)

因为初始化Nexus时，选择的是下载依赖不需要认证信息。

如果选择的是需要，要如何配置。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfIP3tT4mFict8wLYaU8fI3GiboR1Gd5lQFGICKH2olgxT8aHzunmunjMA/640?wx_fmt=png&amp;from=appmsg)

### 11.4 上传依赖到私服

首先在Maven私服的位置，找到release和snapshot的仓库地址

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfK4M5hq8zEbdoh0ef1UrfSHu6fbVqrUko7TWFQTuAwbLZ8jvrrtTicFQ/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfk0CF8uke0ES8pwzCPZ2SKiccmgCicKRgG3TFnZ2L6vpSw6CGicYNz03Jg/640?wx_fmt=png&amp;from=appmsg)

然后在pom.xml文件中配置相应的信息

```xml
<distributionManagement>
    <repository>
        <id>zjw</id>
        <url>http://localhost:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>zjw</id>
        <url>http://localhost:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfphaibx8VciaweZF70hcuMqZibFx9vwUrtuzLy2gefcr2xheHTVOp4h9qQ/640?wx_fmt=png&amp;from=appmsg)

准备好之后，直接在项目右侧，点击deploy上传当前项目的jar到私服

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfSsGJueGnEbCBwI81sRBrJcXzibc0ulRhEgvokQ5heRpwjuDKczKbkoQ/640?wx_fmt=png&amp;from=appmsg)

上传成功后，可以在私服中找到

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNMyAQwia9YnRcHk7s9wu3tfW4vfmJcSPYnEVWYr4KGZt9FoD3ry4Gfv4jyEXXia1HFoCkavwPwBFYg/640?wx_fmt=png&amp;from=appmsg)

其他的项目在配置没问题的情况下，就可以使用私服中的各种依赖了。
