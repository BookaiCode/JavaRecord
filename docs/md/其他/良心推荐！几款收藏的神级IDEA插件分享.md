本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[TOC]


IDEA 拥有众多优秀的插件，这些插件能够极大地提升我们的开发效率和提供更好的编码体验。正所谓：工欲善其事，必先利其器。借助这些插件，我们能更加高效地进行开发，让编码变得轻松愉快。

在本篇中，我将向大家推荐一些个人收藏的实用 IDEA 插件，并根据使用情况对它们进行评级：

> - 强烈推荐：★★★★★
> - 推荐：★★★★

话不多说，我们正式开始。

## CodeGlance

推荐指数：★★★★

编辑区迷你缩放图插件，鼠标悬停还有放大镜的功能。特别适用于处理大量代码时的快速定位需求，让我们更轻松地浏览和编辑代码。


![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lkrjoWIZJTPU3kQT7QkMEAS2VdFEnbeup6MSib2ibicDVJjB2e8VjuqDMw/640)

## GsonFormat

推荐指数：★★★★★

Json 转 Java 类，该插件可以快速生成类，提高开发效率。

使用方法：先新建一个类，选中类名，右键点击生成，点击  `GsonFormat`

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lfxq7dSwqYibDNtt4Z7nYXGQUvEicDuia2Q3hanDTkL4CDDmLPfFojYTQw/640)

然后输入 JSON，点击OK，即可生成。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lW1pHgoLSWicXY8ic1hcyhTeUOKX4Z6jdccvn97O0ibN0Wxr4libQusGpBw/640?wx_fmt=png&amp;from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lWQAH0QoblXz6sSQl0ZYOke8uPfFKFLfGtIcsTQ4icicuKMnZrB1iaTyvw/640)

## POJO to Json

推荐指数：★★★★★

跟 `GsonFormat` 是两兄弟，`GsonFormat` 是将 JSON 转为 POJO，而 `POJO to Json` 则是将 POJO 转为 JSON。

使用方法：选中类，右击 `Copy JSON` 即可复制。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lXP9ickD1bqAdYZtKUV9cvgu3hCiaHrQMHiagkUaeB4w2A4wicib6ibLbOGiaQ/640)

## Rainbow Brackets

推荐指数：★★★★★

可以将括号用不同颜色标记出来，方便使用者快速识别代码层次，提高开发效率。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4l7kSTr2KCicxOtfyY7PDpIoBwFFlWHHoaPKMugcHtsGV1lbM9juiaqJ6g/640)



## Translation

推荐指数：★★★★★

翻译插件，支持谷歌、有道、百度三种翻译。特别是阅读源码的时候，非常有帮助。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lb8SGl3LiaxoRfFVukicgyLRUox1ibxRSebK37cVRibox51GyjJppJibRGpw/640)



## Lombok

推荐指数：★★★★★

主要用来简化代码，减少 get()、set(）等方法的编写，不过有些公司可能禁止使用 Lombok 插件。

最常用的就是 `@Data` 注解，在类上直接使用即可。使用的时候记得打开注解处理器：`Annotation Processors > Enable annotation processing`。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lG59c22vDz3icaj2m83IlSwBkkftuJXmPbxkKcKSO2oHpqoABR2CcXVw/640)



## Maven Helper

推荐指数：★★★★★

可以解析 Maven 依赖，处理依赖冲突很方便，Java开发必备。

使用方法：安装之后，去到项目的 pom.xml 文件，在 pom.xml  右边下面有个  `Dependency Analyzer`  的Tab选项。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lQmk7eNPsb5vr5TuUOSa7Eks3JQ9rDdqCxKH36blbqhTtZD9LpA8Kpg/640)



## Alibaba Java Code Guidelines

推荐指数：★★★★★

阿里巴巴的代码规范插件，可以帮助规范代码质量，程序员必装！

安装完之后，工具栏会显示这两个图标。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lKQKglCzqkk0zyzA5Rfr34wzKE5W8Ae3V8tG5s6TJTnqng03sdrq4WA/640)





## GenerateAllSetter

推荐指数：★★★★★

针对已有的实体对象的属性生成 `set()` 方法代码，在造假数据测试时非常有用。

选择实例，按 `Alt + Enter`，即可出现选项。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lyx1lRic3kHRV7oFy18DIXdR1icia6SfK7RqKxJO3dyUmsOagELJ8uibN3w/640)



## MybatisX

推荐指数：★★★★★

搭配 `Mybatis-Plus` 使用，这个插件有个最大的优点就是可以快速生成，entity，dao，mapper 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lPWib9tibSrchMypicmCf3icSlSkLx6kz7RA8STMknBxktPoicE9p7VibdsZA/640)

连接数据库之后，  右键对应的表，选择 `MybatiX-Generator` 选项即可生成。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lAL00Cjm8Ch40m1SrOL1ic3YZ3GXZicU38NrFWno6NicbIxFKhPtmHorSA/640)



## Chinese (Simplified) Language Pack / 中文语言包

推荐指数：★★★★★

神！IDEA 官方的中文汉化包，对我来说这款插件绝对不能少，可能有人习惯看英文（英语好的略过）。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lIxbBl8xa9WkhJl2rupbORiad9uymLdn3V3UmIWWE1qLOt0o9wiaAXiaCQ/640)

## Key Promoter X

推荐指数：★★★★

`Key Promoter X` 是一个提示插件，当你在 `IDEA` 里面使用鼠标的时候，如果这个鼠标操作是能够用快捷键替代的，那么`Key Promoter X`会弹出一个提示框，告知你这个鼠标操作可以用什么快捷键替代。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lpCg2YJrG55Yqz6uU68jDXOjo9lGibeF4db1iaGzBXWxmIz9zdAEo2Ruw/640)

## Arthas Idea

推荐指数：★★★★★

可以自动帮我们生成 Arthas命令，选中类或方法右键点击 `Arthas Command` 即可生成。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lK1AHyuUziauDauWxjwQaOx8PqNeRA13rs42LdBk5xNKL75j3zArXo0A/640)



## GitToolBox

推荐指数：★★★★

在自带的 Git 功能之上，新增了查看 Git 状态、自动拉取代码、提交通知等功能。

安装之后可以查看到每一行代码的最近一次提交信息。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lciafx92re4LaxINxgvXMNupqAPoltPnhibLyiby3MsZlgy0hJGBfMRcGw/640)



## VisualGC

推荐指数：★★★★

 JVM 堆栈可视化工具，支持查看本地和远程 JVM 进程。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lnTWwtQW2rhA0qaBtzZhgBxTSaYNRNlqK9e4wVmJtJmeQLdEYrbZYkw/640)

## String Manipulation

推荐指数：★★★★

String Manipulation 插件用来对字符串进行处理，比如：变量名使用驼峰形式、常量需要全部大写，编码解码等等，右击字符串即可使用。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4larl7vsy3HkfYMsnvoCQC0HxYYSAjlibXrmbBZaNqfgMVpK6zrplMBBA/640)

## SequenceDiagram

推荐指数：★★★★

自动生成方法调用时序图，能够帮助快速梳理代码逻辑。免费版对方法层级有限制，日常使用基本也够了。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lQtq5tqQ00hibbslaxycXVssCBxlSGznHyaEfZM1MibRQR0KjunBHqpKQ/640)



## CheckStyle-IDEA

推荐指数：★★★★

帮助 JAVA开发人员遵守某些编码规范的工具。它能够自动化代码规范检查过程，右击选择 `Check Current File` 即可给出 Style 建议。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4l2StO6Gic2HvZ02QJHTDMjKoTFYiaia2G5dadogctEcjicofrDr6w087jjg/640)

## SonarLint

推荐指数：★★★★

帮助开发人员发现和修复代码的错误和漏洞，安装完毕之后下方会有 `SonarLint` 菜单栏。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lNERoUCl3RCmuh7HAYVHCQXlLSMI4n4Xl28C2bkwQcuUgrnAYRGZ25w/640)

## jclasslib Bytecode Viewer

推荐指数：★★★★

字节码查看器，对于字节码学习非常有帮助。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lwjjrpkI5XIyH57b9XVwLHtSGt8PZnNKVoSDyymce4lXOQEN6DyB7LA/640)

安装之后在视图栏就可以直接打开查看。

## Properties to YAML Converter

推荐指数：★★★★

把 Properties 文件的格式转为 YAML 格式。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lk7z8WiaI0BY8lWYyNKFpXQcK89p97hR2ibswDXBB6VForH9TIibibUKJQw/640)

鼠标右击 properties 文件选择 `Convert Properties to YAML` 即可转为 YAML 格式。

## Alibaba Cloud Tookit

推荐指数：★★★★★

Alibaba Cloud Toolkit 可以帮助开发者更高效地部署、测试、开发和诊断应用。帮助开发人员大大简化应用部署到服务器，尤其是阿里云服务器中的操作。还可以通过其内嵌的 Arthas 程序诊断、Terminal Shell 终端和 MySQL 执行器等工具，简化应用开发、测试和诊断的过程。



![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lJFVYwwkY7LaDqvGd2KNjHnFS7st08ymhrqUL1yqeiaPicfMEUhMX5NpQ/640)

更多使用建议参考官方文档。

## One Dark theme

推荐指数：★★★★★

个人最喜欢的主题插件。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4l13M2AicOtZxuEnYJMRbeEEGhicAViaOG0cpeNBNDfeme769usEI6B63Dg/640)

安装之后可以去主题里修改，这里推荐：`One Dark vivid ltalic`。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lZ7rp7DibibEkLWrWPniclSert9QgHibvW2Pwib5QNTNlvonMT2vhG2ibdc3A/640)





## PlantUML Integration

推荐指数：★★★★★

神！开发人员必备插件，平时出技术方案流程图，用例图等全靠它了，关键还免费。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lghc4URyGjzCuWmCutwokpLEcQOF0rQdEtIXtwRfteiaTCX7libH8j7OQ/640)



更多语法参考官网：https://plantuml.com/zh/，官网还支持中文，非常人性化。

## any-rule 

推荐指数：★★★★

这款插件不是特别大众，但是特别实用，可以快速生成正则表达式。

安装之后右击 选择 `AnyRule` 即可使用。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lREQfuNAF2ylTYOFC3mKSEzaLaShRzFFbhTleGKSP2jn1x4CmjqtZicg/640)



## Tabnine 

推荐指数：★★★★

代码智能提示插件。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lWXtSTYZco7bq8U09WLzlg5xE4qHen4Kp3DFGcRGysg6YKicjx6AC15A/640)

编码过程中按 `Tab` 即可采纳建议。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lBzZaOhvYVI6EqJNoIjdeHb6R84FD69MSgwziblsznl9gyicia8ItqdBVA/640)



## TONGYI Lingma

推荐指数：★★★★★

阿里出品的通义灵码，刚发布不久，也是智能AI编码插件。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4lvzL6Ow4Yt0ltQanNQmkoo5O6Xl6cjZxnvVvtNPE5iaZ6e2ricV6jz6aQ/640)

注意要登陆才能使用。

## Git Commit Message Helper

推荐指数：★★★★★

这款插件，知道的人并不多，但是却是我使用频率最高的插件之一。

Git Commit Message Helper 能够帮助开发人员提交出规范的 Git Commit。

使用也非常简单，提交代码的时候点击右边的图标即可使用。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOnKt1yKmAD9Y4EaMjrVD4l7yA9wUlEeTtibJWxMofCN2J1oLlNBvdBl4T3TFU6UfhengA2Kh6u28Q/640)

这里再分享一篇关于  Git Commit 规范的文章：[如何规范你的Git commit？](https://zhuanlan.zhihu.com/p/182553920)



以上这几款 IDEA 插件是我平常开发中经常用到的，如果大家有更好的插件，欢迎分享出来。

> 插件持续更新中。记得收藏！