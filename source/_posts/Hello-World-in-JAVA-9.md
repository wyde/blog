---
title: Ubuntu 16.04 上寫 Java 9 的 Hello World
date: 2018-01-03 15:33:18
tags:
- Java
---
<center>Hello World in JAVA 9</center>
===
<br>

新的一年因為工作的關係，要來好好學個 java ，先來個環境設置(私心希望團隊未來能轉 go 啊...)，首先有幾件事要先區分

## 最基礎
- Open JDK 和 Oracle JDK 的差別

- JDK、JRE、JVM 的區別
    - JRE: java runtime，java 程式碼如果要可以動必須安裝
    - JDK: java developement kit，如果要寫 java code 必須安裝

- J2SE、J2EE、J2ME 的概念區別

- Reference
    - [假如时光能够倒流， 我会这么学习Java](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=415513252&idx=1&sn=1c1211e23c507c34f9befbf5282a85c8&scene=21#wechat_redirect)

---

## Java Web 相關
- JSP、Java Servlet
    - jSP: 包含 java 程式碼的 html 文件 *.jsp (如果做前後端分離，而且使用 React 之類的就不用了)
    - java servlet: 在 server 端純粹的 java 程式碼 *.java -> *.class

- JavaBeans、EJB、POJO
    - JavaBeans: 符合某些約定的 public 類，可以說是一種彌補語言不足的規範
        - 所有屬性為 private
        - 提供默認構造方法 (無參構造器)
        - 提供 getter 和 setter 例如: 屬性 name ， get 方法寫成 public String getName(){}
        - 實現可序列化(serializable)接口
    - Reference
        - [Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)

- 如何解耦與開發 web 項目的思路
    - 後端
        - 控制層 (controller/action)
        - 業務層 (service/manage)
        - 持久層 (dao)
    - 前端
        - 以後再說...
    - Reference
        - [JavaWeb项目为什么我们要放弃jsp？为什么要前后端解耦？为什么要前后端分离？2.0版，为分布式架构打基础。(2017-03-23)](http://blog.csdn.net/piantoutongyang/article/details/65446892)

---

## 安裝

以下將筆記安裝 Java 9 在 Ubuntu 16.04 上

```
    $ sudo add-apt-repository ppa:webupd8team/java
    $ sudo apt-get update
    $ sudo apt-get install oracle-java9-installer
```

## 檢查
```
    $ javac --version
    javac 9.0.1

    $ java --version
    java 9.0.1
    Java(TM) SE Runtime Environment (build 9.0.1+11)
    Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)
```

## 設定 java default 的版本

用 `sudo update-alternatives --config <要改變預設的 binary>` 可以設定預設使用的版號
```
    $ sudo update-alternatives --config java
    替代項目 java（提供 /usr/bin/java）有 1 個選擇。

      選項       路徑                               優先權  狀態
    ------------------------------------------------------------
      0            /usr/lib/jvm/java-9-oracle/bin/java   1091      自動模式
    * 1            /usr/lib/jvm/java-9-oracle/bin/java   1091      手動模式
    
    Press <enter> to keep the current choice[*], or type selection number:
```

## JAVA_HOME 環境變數

```
    $ echo 'JAVA_HOME="/usr/lib/jvm/java-9-oracle"' | sudo tee --append /etc/environment > /dev/null

    $ echo $JAVA_HOME
    /usr/lib/jvm/java-9-oracle
```

---

## Hello World snippet

Hello.java ，首字要大寫
```=java
public class Hello {
    public static void main(String[] argv) {
        System.out.print("hello, world!!\n");
    }   
}
```

## compile

```
    $ javac *.java
```
`-verbose` 可以看整個 compile 的流程，編出來是 *.class ，以上述片段而言是 Hello.class

## execute

```
    $ java Hello
    hello, world!!
```

(end)
