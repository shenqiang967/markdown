## 

# JDK安装

## windows下安装

[jdk最新版下载地址]: http://www.oracle.com/technetwork/java/javase/downloads/index.html "下载地址"

![image-20210726160837951](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\image-20210726160837951-16272869219609.png)

![image-20210726160923569](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\image-20210726160923569-162728696509210.png)

## 配置环境变量

安装完成后，右击"我的电脑"，点击"属性"，选择"高级系统设置"

![image-20210726155248579](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\image-20210726155248579-16272859700934.png)

选择"高级"选项卡，点击"环境变量"；

![img](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\java-win2.png)

在 "系统变量" 中设置 3 项属性，JAVA_HOME、PATH、CLASSPATH(大小写无所谓),若已存在则点击"编辑"，不存在则点击"新建"。

> 注意：如果使用 1.5 以上版本的 JDK，不用设置 CLASSPATH 环境变量，也可以正常编译和运行 Java 程序。

变量设置参数如下：

+ 变量名：**JAVA_HOME**
+ 变量值：**C:\Program Files (x86)\Java\jdk1.8.0_91**     // 要根据自己的实际路径配置

- 变量名：**CLASSPATH**
- 变量值：**.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;**     //记得前面有个"."

- 变量名：**Path**
- 变量值：**%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;**

下面是具体设置参数的步骤

![1573612234599_JDK安装与配置11.jpg](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\1573612234599_JDK安装与配置11.jpg)

![1573612246673_JDK安装与配置12.jpg](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\1573612246672_JDK安装与配置12.jpg)

![1573612258070_JDK安装与配置13.jpg](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\1573612258070_JDK安装与配置13-16272863394755.jpg)



![1573612284458_JDK安装与配置14.jpg](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\1573612284458_JDK安装与配置14.jpg)



## 测试JDK是否安装成功

win+R 输入cmd命令  java -version

![image-20210726155154410](C:\Users\18055\Desktop\笔记\java基础\jdk安装.assets\image-20210726155154410-16272859201941.png)