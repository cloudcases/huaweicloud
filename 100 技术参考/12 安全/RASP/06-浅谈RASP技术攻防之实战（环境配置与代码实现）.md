浅谈RASP技术攻防之实战（环境配置与代码实现）

[安百科技 ](https://www.freebuf.com/author/安百科技) 2019-05-17 09:30:39 392166 3

> 之前偶们有就RASP技术攻防基础作过简单介绍，穿越捷径：[请点击](https://www.freebuf.com/articles/web/197823.html)

**今儿接上回，说一说环境配置的事儿，废话不多说，直接进入正题：**

**PS：代码已上传至github，地址：https://github.com/iiiusky/java_rasp_example**

## 初始化项目

首先我们在IDEA中新建一个maven项目

![1.jpg](https://image.3001.net/images/20190506/1557132245_5ccff3d546637.jpg!small)

取名为JavawebAgent

![2.jpg](https://image.3001.net/images/20190506/1557132298_5ccff40ae5215.jpg!small)![3.jpg](https://image.3001.net/images/20190506/1557132298_5ccff40acc3fa.jpg!small)

然后当前的目录结构如下：

![4.jpg](https://image.3001.net/images/20190506/1557132315_5ccff41b55a6c.jpg!small)

删除src目录，然后右键新建Module

![5.jpg](https://image.3001.net/images/20190506/1557132329_5ccff429b32c4.jpg!small)

依然选择Maven项目

![6.jpg](https://image.3001.net/images/20190506/1557132345_5ccff43976d1d.jpg!small)

然后在ArtifactId处填入agent

![7.jpg](https://image.3001.net/images/20190506/1557132362_5ccff44a93259.jpg!small)

然后确定即可

![8.jpg](https://image.3001.net/images/20190506/1557132375_5ccff4577c0e9.jpg!small)

然后再次重复一遍上面的新建Module的操作，将第二小步中的ArtifactId改为test，第三小步中的Module Name 改为test-struts2,如下图所示

![9.jpg](https://image.3001.net/images/20190506/1557132388_5ccff464123e6.jpg!small)![10.jpg](https://image.3001.net/images/20190506/1557132388_5ccff4647431e.jpg!small)

这时候的目录结构如下

![11.jpg](https://image.3001.net/images/20190506/1557132399_5ccff46f7dbe3.jpg!small)

其中agent目录为我们要实现agent的主要代码区域，test-struts2为测试web代码区域。（注：test-struts2不是必选的）

## test-struts2模块基础配置

test-struts2部分的代码这边就不进行复述了，大家可以去本项目的地址中直接下载test-struts2内容。

## agent模块基本配置

### **≡≡ pom.xml包配置** 

agent这个pom包配置的话有坑，这个以后在说，先看pom.xml内容吧。 

```
<dependencies>
        <dependency>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm-all</artifactId>
            <version>5.1</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>agent</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <archive>
                        <manifestFile>src/main/resources/MANIFEST.MF</manifestFile>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <artifactSet>
                                <includes>
                                    <include>commons-io:commons-io:jar:*</include>
                                    <include>org.ow2.asm:asm-all:jar:*</include>
                                </includes>
                            </artifactSet>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.21.0</version>
                <configuration>
                    <skipTests>true</skipTests>
                </configuration>
            </plugin>
        </plugins>
    </build>
```



将上述内容复制到agent模块下的pom.xml中

![12.jpg](https://image.3001.net/images/20190506/1557132418_5ccff482c7994.jpg!small)

≡≡ 创建MAINFEST.NF文件

在resources目录下创建MAINFEST.NF文件，文件内容如下

```
Manifest-Version: 1.0
Can-Retransform-Classes: true
Can-Redefine-Classes: true
Can-Set-Native-Method-Prefix: true
```

### ≡≡ maven自动打包配置

在idea中右上部分找到Add Configurations , 然后点击此按钮

![13.jpg](https://image.3001.net/images/20190506/1557132433_5ccff4918a959.jpg!small)

在弹出的窗口中点左上角的+,选择maven

![14.jpg](https://image.3001.net/images/20190506/1557132445_5ccff49df0f56.jpg!small)

然后点下图①的位置选择工作目录，在②的位置选择agent，在③的位置填入clean install

![15.jpg](https://image.3001.net/images/20190506/1557132457_5ccff4a971e45.jpg!small)

完成以后如下图所示，然后点击OK保存即可

![16.jpg](https://image.3001.net/images/20190506/1557132469_5ccff4b5aa85b.jpg!small)

这时候右上角已经可以看到我们刚刚配置的maven自动打包功能了，agent每改一处都需要重新build，不然无法生效。

![17.jpg](https://image.3001.net/images/20190506/1557132484_5ccff4c43723a.jpg!small)

### ≡≡ 创建Agent主要实现代码包

在agent包下面的java文件夹下右键选择新建package，然后填入你的包名，我这边的包名为

```
cn.org.javaweb.agent
```

![18.jpg](https://image.3001.net/images/20190506/1557132493_5ccff4cdf3d34.jpg!small)

# 简易版RASP实现 

## 创建入口类

在cn.org.javaweb.agent包下新建一个类。

内容如下：

```
/*
 * Copyright sky 2019-04-03 Email:sky@03sec.com.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package cn.org.javaweb.agent;
import java.lang.instrument.Instrumentation;
/**
 * @author sky
 */
public class Agent {
    public static void premain(String agentArgs, Instrumentation inst) {
        inst.addTransformer(new AgentTransform());
    }
}
```

## 创建Transform

然后我们再新建一个AgentTransform类，该类需要实现ClassFileTransformer的方法，内容如下:



```
/*
 * Copyright sky 2019-04-03 Email:sky@03sec.com.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package cn.org.javaweb.agent;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.security.ProtectionDomain;
/**
 * @author sky
 */
public class AgentTransform implements ClassFileTransformer {
    /**
     * @param loader
     * @param className
     * @param classBeingRedefined
     * @param protectionDomain
     * @param classfileBuffer
     * @return
     * @throws IllegalClassFormatException
     */
    @Override
    public byte[] transform(ClassLoader loader, String className,
                            Class<?> classBeingRedefined, ProtectionDomain protectionDomain,
                            byte[] classfileBuffer) throws IllegalClassFormatException {
        className = className.replace("/", ".");
        System.out.println("Load class:" + className);
        return classfileBuffer;
    }
}
```

build Agent配置

点击右上角的agent[clean,intall]进行build。

![2.jpg](https://image.3001.net/images/20190507/1557219314_5cd147f2ab495.jpg!small)

由上图可见我们的包的位置为

```
/Volumes/Data/code/work/JavawebAgent/agent/target/agent.jar
```

将改包的位置记录下来，然后点开tomcat配置(这边没有对idea如何配置tomcat进行讲解，不会的可以自行百度|谷歌)

![3.jpg](https://image.3001.net/images/20190507/1557219298_5cd147e2ba921.jpg!small)

在VM options处填写以下内容：



```
-Dfile.encoding=UTF-8
-noverify
-Xbootclasspath/p:/Volumes/Data/code/work/JavawebAgent/agent/target/agent.jar
-javaagent:/Volumes/Data/code/work/JavawebAgent/agent/target/agent.jar
```



其中/Volumes/Data/code/work/JavawebAgent/agent/target/agent.jar的路径为你在上一步编译出来的agent的路径，注意替换。

这时候我们在启动tomcat，就可以看到我们在AgentTransform中写的打印包名已经生效了，如下图:

![4.jpg](https://image.3001.net/images/20190507/1557219224_5cd147982c914.jpg!small)

上图红框区域为tomcat启动的时候加载的所有类名。然后我们打开浏览器查看web是否正常。

![5.jpg](https://image.3001.net/images/20190507/1557219216_5cd14790c0656.jpg!small)

可以看到web也正常启动了。

## 创建ClassVisitor类

然后我们新建一个TestClassVisitor类，需要继承ClassVisitor类并且实现Opcodes类，代码如下



```
/*
 * Copyright sky 2019-04-03 Email:sky@03sec.com.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package cn.org.javaweb.agent;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;
/**
 * @author sky
 */
public class TestClassVisitor extends ClassVisitor implements Opcodes {
    public TestClassVisitor(ClassVisitor cv) {
        super(Opcodes.ASM5, cv);
    }
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
        System.out.println(name + "方法的描述符是：" + desc);
        return mv;
    }
}
```

## 对ProcessBuilder（命令执行）类进行hook用户执行的命令

### ≡≡ 使用transform对类名进行过滤

然后回到AgentTransform中，对transform方法的内容进行修改，transform方法代码如下：



```
public byte[] transform(ClassLoader loader, String className,
                            Class<?> classBeingRedefined, ProtectionDomain protectionDomain,
                            byte[] classfileBuffer) throws IllegalClassFormatException {
        className = className.replace("/", ".");
        try {
            if (className.contains("ProcessBuilder")) {
                System.out.println("Load class: " + className);
                ClassReader  classReader  = new ClassReader(classfileBuffer);
                ClassWriter  classWriter  = new ClassWriter(classReader, ClassWriter.COMPUTE_MAXS);
                ClassVisitor classVisitor = new TestClassVisitor(classWriter);
                classReader.accept(classVisitor, ClassReader.EXPAND_FRAMES);
                classfileBuffer = classWriter.toByteArray();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return classfileBuffer;
    }
```

简单介绍一下代码块内容

首先判断类名是否包含ProcessBuilder,如果包含则使用ClassReader对字节码进行读取，然后新建一个ClassWriter进行对ClassReader读取的字节码进行拼接，然后在新建一个我们自定义的ClassVisitor对类的触发事件进行hook，在然后调用classReader的accept方法,最后给classfileBuffer重新赋值修改后的字节码。

可能看起来比较绕，但是如果学会使用以后就比较好理解了。

### ≡≡ 创建测试环境

我们在tomcat中新建一个jsp，用来调用命令执行，代码如下：



```
<%@ page import="java.io.InputStream" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<pre>
<%
    Process process = Runtime.getRuntime().exec(request.getParameter("cmd"));
    InputStream in = process.getInputStream();
    int a = 0;
    byte[] b = new byte[1024];
    while ((a = in.read(b)) != -1) {
        out.println(new String(b, 0, a));
    }
    in.close();
%>
</pre>
```



可以看到就是一个简单的执行命令的代码;下面我们对就此更改过的内容进行build，看一下会输出点什么。

![6.jpg](https://image.3001.net/images/20190507/1557219105_5cd1472165410.jpg!small)

biuld完成，启动tomcat。

![7.jpg](https://image.3001.net/images/20190507/1557219094_5cd147168316e.jpg!small)

访问

> http://localhost:8080/cmd.jsp?cmd=whoami

可以看到已经成功执行命令，我们回到idea里面的控制台看一下输出了什么。

![8.jpg](https://image.3001.net/images/20190507/1557219079_5cd147078d7c1.jpg!small)

通过上图可以完整的看到一个执行命令所调用的所有调用链。



```
Load class: java.lang.ProcessBuilder
<init>方法的描述符是：(Ljava/util/List;)V
<init>方法的描述符是：([Ljava/lang/String;)V
command方法的描述符是：(Ljava/util/List;)Ljava/lang/ProcessBuilder;
command方法的描述符是：([Ljava/lang/String;)Ljava/lang/ProcessBuilder;
command方法的描述符是：()Ljava/util/List;
environment方法的描述符是：()Ljava/util/Map;
environment方法的描述符是：([Ljava/lang/String;)Ljava/lang/ProcessBuilder;
directory方法的描述符是：()Ljava/io/File;
directory方法的描述符是：(Ljava/io/File;)Ljava/lang/ProcessBuilder;
redirects方法的描述符是：()[Ljava/lang/ProcessBuilder$Redirect;
redirectInput方法的描述符是：(Ljava/lang/ProcessBuilder$Redirect;)Ljava/lang/ProcessBuilder;
redirectOutput方法的描述符是：(Ljava/lang/ProcessBuilder$Redirect;)Ljava/lang/ProcessBuilder;
redirectError方法的描述符是：(Ljava/lang/ProcessBuilder$Redirect;)Ljava/lang/ProcessBuilder;
redirectInput方法的描述符是：(Ljava/io/File;)Ljava/lang/ProcessBuilder;
redirectOutput方法的描述符是：(Ljava/io/File;)Ljava/lang/ProcessBuilder;
redirectError方法的描述符是：(Ljava/io/File;)Ljava/lang/ProcessBuilder;
redirectInput方法的描述符是：()Ljava/lang/ProcessBuilder$Redirect;
redirectOutput方法的描述符是：()Ljava/lang/ProcessBuilder$Redirect;
redirectError方法的描述符是：()Ljava/lang/ProcessBuilder$Redirect;
inheritIO方法的描述符是：()Ljava/lang/ProcessBuilder;
redirectErrorStream方法的描述符是：()Z
redirectErrorStream方法的描述符是：(Z)Ljava/lang/ProcessBuilder;
start方法的描述符是：()Ljava/lang/Process;
<clinit>方法的描述符是：()V
Load class: java.lang.ProcessBuilder$NullInputStream
<init>方法的描述符是：()V
read方法的描述符是：()I
available方法的描述符是：()I
<clinit>方法的描述符是：()V
Load class: java.lang.ProcessBuilder$NullOutputStream
<init>方法的描述符是：()V
write方法的描述符是：(I)V
<clinit>方法的描述符是：()V
```



### ≡≡ 拿到用户所执行的命令

接下来我们看看尝试一下能否拿到所执行的命令

新建一个名为ProcessBuilderHook的类，然后在类中新建一个名字为start的静态方法，完整代码如下：

```
/*
 * Copyright sky 2019-04-04 Email:sky@03sec.com.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package cn.org.javaweb.agent;
import java.util.Arrays;
import java.util.List;
/**
 * @author sky
 */
public class ProcessBuilderHook {
    public static void start(List<String> commands) {
        String[] commandArr = commands.toArray(new String[commands.size()]);
        System.out.println(Arrays.toString(commandArr));
    }
}
```



这个方法干啥用的我们一会在说，先看下面。

### ≡≡ 复写visitMethod方法

打开TestClassVisitor，对visitMethod方法进行更改。具体代码如下：



```
@Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
        if ("start".equals(name) && "()Ljava/lang/Process;".equals(desc)) {
            System.out.println(name + "方法的描述符是：" + desc);
            return new AdviceAdapter(Opcodes.ASM5, mv, access, name, desc) {
                @Override
                public void visitCode() {
                    mv.visitVarInsn(ALOAD, 0);
                    mv.visitFieldInsn(GETFIELD, "java/lang/ProcessBuilder", "command", "Ljava/util/List;");
                    mv.visitMethodInsn(INVOKESTATIC, "cn/org/javaweb/agent/ProcessBuilderHook", "start", "(Ljava/util/List;)V", false);
                    super.visitCode();
                }
            };
        }
        return mv;
    }
```



给大家解释下新增加的代码，从if判断开始

判断传入进来的方法名是否为start以及方法描述符是否为()Ljava/lang/Process;,如果是的话就新建一个AdviceAdapter方法，并且复写visitCode方法，对其字节码进行修改，

```
mv.visitVarInsn(ALOAD, 0);
```

拿到栈顶上的this

```
mv.visitFieldInsn(GETFIELD, "java/lang/ProcessBuilder", "command", "Ljava/util/List;");
```

拿到this里面的command

```
mv.visitMethodInsn(INVOKESTATIC, "cn/org/javaweb/agent/ProcessBuilderHook", "start", "(Ljava/util/List;)V", false);
```

然后调用我们上面新建的ProcessBuilderHook类中的start方法,将上面拿到的this.command压入我们方法。

ProcessBuilderHook类的作用就是让这部分进行调用，然后转移就可以转入到我们的逻辑代码了。

我们再次编译一下，然后启动tomcat，访问cmd.jsp看看.

### ≡≡ 测试hook用户执行的命令参数是否拿到

访问

> [http://localhost:8080/cmd.jsp?cmd=ls%20-la](http://localhost:8080/cmd.jsp?cmd=ls -la)

![9.jpg](https://image.3001.net/images/20190507/1557218955_5cd1468b1d9d6.jpg!small)

可以看到已经将当前目录下的内容打印了出来。

我们到idea中看看控制台输出了什么。

![10.jpg](https://image.3001.net/images/20190507/1557218943_5cd1467fe0548.jpg!small)

可以看到我们输入的命令

```
[whoami]
```

已经输出出来了，到此为止，我们拿到了要执行的命令.

## 总结

对于拿到要执行的命令以后怎么做，是需要拦截还是替换还是告警，这边就需要大家自己去实现了。当然，如果要实现拦截功能，还需要注意要获取当前请求中的的response，不然无法对response进行复写，也无法对其进行拦截。这边给大家提供一个思路，对应拦截功能，大家可以去hook请求相关的类，然后在危险hook点结合http请求上下文进行拦截请求。

对于其他攻击点的拦截，可以参考百度开源的[OpenRasp](https://rasp.baidu.com/doc/hacking/architect/hook.html#java-server)进行编写hook点。

如需在Java中实现RASP技术，笔者建议好好了解一下ASM，这样对以后JAVA的运行机制也会有一定的了解，方便以后调试以及写代码。

## 参考

> https://rasp.baidu.com/doc/hacking/architect/hook.html#java-server
>
> https://github.com/anbai-inc/javaweb-codereview
>
> https://static.javadoc.io/org.ow2.asm/asm/5.2/org/objectweb/asm/ClassReader.html
>
> http://www.blogjava.net/vanadies10/archive/2011/02/23/344899.html
>
> http://www.blogjava.net/DLevin/archive/2014/06/25/414292.html

***本文作者：安百科技，转载请注明来自FreeBuf.COM**



[浅谈RASP技术攻防之实战（环境配置与代码实现） - FreeBuf网络安全行业门户](https://www.freebuf.com/articles/web/202806.html)