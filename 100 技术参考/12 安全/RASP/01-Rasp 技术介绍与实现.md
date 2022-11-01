# Rasp 技术介绍与实现



 2017年06月21日
 [经验心得](https://paper.seebug.org/category/experience/)

## 目录

- 
- 

- [0x01 什么是RASP](https://paper.seebug.org/330/#0x01-rasp)
- [0x02 RASP 能做什么](https://paper.seebug.org/330/#0x02-rasp)
- [0x03 WAF VS RASP](https://paper.seebug.org/330/#0x03-waf-vs-rasp)
- [0x04 实现基本思路](https://paper.seebug.org/330/#0x04)
- [0x05 利用 Java Instrumentation 在 class 加载前插入修改机会](https://paper.seebug.org/330/#0x05-java-instrumentation-class)
- [0x06 利用 ClassTransformer 进行探针织入](https://paper.seebug.org/330/#0x06-classtransformer)
- [0x07 结尾](https://paper.seebug.org/330/#0x07)



作者: **xbear**

#### 0x01 什么是RASP

RASP（Runtime application self-protection）运行时应用自我保护。Gartner 在2014年应用安全报告里将 RASP 列为应用安全领域的关键趋势，并将其定义为:

> Applications should not be delegating most of their runtime protection to the external devices. Applications should be capable of self-protection (i.e., have protection features built into the application runtime environment).

<img src="https://images.seebug.org/content/images/2017/06/1-6.png-w331s" alt="img" style="zoom:67%;" />

**RSAP 将自身注入到应用程序中，与应用程序融为一体，实时监测、阻断攻击，使程序自身拥有自保护的能力**。并且应用程序无需在编码时进行任何的修改，只需进行简单的配置即可。

#### 0x02 RASP 能做什么

RASP不但能够对应用进行基础安全防护，由于一些攻击造成的应用程序调用栈调用栈具有相似性，还能够对0day进行一定的防护。

除此之外，利用 RASP 也能够对应用打虚拟补丁，修复官方未修复的漏洞。或者对应用的运行状态进行监控，进行日志采集。

#### 0x03 WAF VS RASP

传统的 **WAF 主要通过分析流量中的特征过滤攻击请求，并拦截携带有攻击特征的请求**。WAF 虽然可以有效个过滤出绝大多数恶意请求，但是**不知道应用运行时的上下文，必然会造成一定程度的误报**。并且 **WAF 严重依赖于特征库，各种花式绕过，导致特征编写很难以不变应万变**。

RASP 的不同就在于**运行在应用之中，与应用融为一体，可以获取到应用运行时的上下文，根据运行时上下文或者敏感操作，对攻击进行精准的识别或拦截**。于此同时，由于 RASP 运行在应用之中，只要检测点选取合理，获取到的 payload 已经是解码过的真实 payload ，可以减少由于 WAF 规则的不完善导致的漏报。

虽然 RASP 拥有 WAF 所不具有的一些优势，但是否能够代替 WAF 还有待商榷。毕竟 WAF 是成熟、快速、可以大规模部署的安全产品。**两者相互补充，将 WAF 作为应用外围的防线，RASP 作为应用自身的安全防护，确保对攻击的有效拦截**。

#### 0x04 实现基本思路

这里以 Java Rasp 的实现原理为例。**Rasp 想要将自己注入到被保护的应用中，基本思路类似于 Java 中的 AOP 技术，将 RASP 的探针代码注入到需要进行检测的地方**。

Java 的 AOP 主要可以从几个层面来实现：

- 编译期
- 字节码加载前
- 字节码加载后

在编译期进行 AOP 织入，一般需要编写静态代理，导致灵活性差，对原有的应用代码有修改。

**在字节码加载后进行 AOP 织入，一般使用动态代理，为接口动态生成代理类**。动态代理虽然灵活性高，但仍然需要使用相关的类库，进行动态代理的配置，并融合到应用的源代码中，不是理想的解决方案。

最后只剩下了在字节码加载前进行 AOP 织入。**在字节码加载前进行织入，一般有两种方法，重写 ClassLoader 或利用 Instrumentation** 。如果重写 ClassLoader ，仍然对现有代码进行了修改，不能做到对应用无侵入。所以只有利用 Java 的 Instrumentation 。

#### 0x05 利用 Java Instrumentation 在 class 加载前插入修改机会

“java.lang.instrument”包的具体实现依赖于 JVMTI 。JVMTI（Java Virtual Machine Tool Interface）是一套由 Java 虚拟机提供的，为 JVM 相关的工具提供的本地编程接口集合。

在 Instrumentation 的实现当中，存在一个 JVMTI 的代理程序，通过调用 JVMTI 当中 Java 类相关的函数来完成 Java 类的动态操作。

我们可以 Instrumentation 的代理，并让其在 main 函数之前运行，这里需要实现的主要是 premain 函数。

```java
public static void premain(String agentArgs, Instrumentation inst)
    throws ClassNotFoundException, UnmodifiableClassException {
    Console.log("init");
    init();
    inst.addTransformer(new ClassTransformer());
}

private static boolean init() {
Config.initConfig();
return true;
}
```

在 premain 函数中，我们将类转换器添加到了 Instrumentation ，这样在类加载前，我们便有机会对字节码进行操作，织入 Rasp 的安全探针。

若想使用带有 Instrumentation 代理的程序，需要在 JVM 的启动参数中添加 -javaagent 启动参数。

```
-javaagent:[编译好的agent jar文件路径]XXX.jar
```

#### 0x06 利用 ClassTransformer 进行探针织入

在运行了 Instrumentation 代理的 Java 程序中，字节码的加载会经过我们自定义的 ClassTransformer ，在这里我们可以过滤出我们关注的类，并对其字节码进行相关的修改

```java
public class ClassTransformer implements ClassFileTransformer {
public byte[] transform(ClassLoader loader, String className, Class<?>
classBeingRedefined,
ProtectionDomain protectionDomain, byte[] classfileBuffer) throws
IllegalClassFormatException {
byte[] transformeredByteCode = classfileBuffer;

if (Config.moudleMap.containsKey(className)) {
try {
ClassReader reader = new ClassReader(classfileBuffer);
ClassWriter writer = new ClassWriter(ClassWriter.COMPUTE_MAXS);
ClassVisitor visitor =
Reflections.createVisitorIns((String)Config.moudleMap.get(className).get("loadClass"),
writer, className);
reader.accept(visitor, ClassReader.EXPAND_FRAMES);
transformeredByteCode = writer.toByteArray();
} catch (ClassNotFoundException e) {
e.printStackTrace();
} catch (NoSuchMethodException e) {
e.printStackTrace();
} catch (InstantiationException e) {
e.printStackTrace();
} catch (IllegalAccessException e) {
e.printStackTrace();
} catch (InvocationTargetException e) {
e.printStackTrace();
} catch (Exception e) {
e.printStackTrace();
}
}
return transformeredByteCode;
}
}
```

实现中使用了使用了 Map 将关注的类进行保存，一旦命中我们关心的类，便利用反射生成 asm 的ClassVisitor ，使用 asm 操作字节码，进行探针织入，最终返回修改后的字节码。

这里的 ClassVisitor 以 Struts 2 的 Ognl 表达式执行漏洞为例：

```java
public class OgnlVisitor extends ClassVisitor {
public String className;

public OgnlVisitor(ClassVisitor cv, String className) {
super(Opcodes.ASM5, cv);
this.className = className;
}
@Override
    public MethodVisitor visitMethod(int access, String name, String desc,
            String signature, String[] exceptions) {
MethodVisitor mv = super.visitMethod(access, name, desc, signature, exceptions);
if ("parseExpression".equals(name) &&
"(Ljava/lang/String;)Ljava/lang/Object;".equals(desc)) {
mv = new OgnlVisitorAdapter(mv, access, name, desc);
}
return mv;
}
}
```

在这个 ClassVisitor 中，我们关心的是 ognl.Ognl 类中的 parseExpression 方法。只要在 Ognl 的 parseExpression 执行之前对 Ognl 表达式中的恶意参数进行过滤，可以对 Struts 2 的 Ognl 表达式执行漏洞进行有效的防护。具体的字节码操作封装在了 OgnlVisitorAdapter 中。

```java
public class OgnlVisitorAdapter extends AdviceAdapter {
public OgnlVisitorAdapter(MethodVisitor mv, int access, String name,
String desc) {
super(Opcodes.ASM5, mv, access, name, desc);
}
@Override
protected void onMethodEnter() {
        Label l30 = new Label();
        mv.visitLabel(l30);
mv.visitVarInsn(ALOAD, 0);
mv.visitMethodInsn(INVOKESTATIC,
"xbear/javaopenrasp/filters/rce/OgnlFilter", "staticFilter",
"(Ljava/lang/Object;)Z", false);
Label l31 = new Label();
mv.visitJumpInsn(IFNE, l31);
Label l32 = new Label();
mv.visitLabel(l32);
mv.visitTypeInsn(NEW, "ognl/OgnlException");
mv.visitInsn(DUP);
mv.visitLdcInsn("invalid class in ognl expression because of security");
mv.visitMethodInsn(INVOKESPECIAL, "ognl/OgnlException", "<init>",
"(Ljava/lang/String;)V", false);
mv.visitInsn(ATHROW);
mv.visitLabel(l31);
}
@Override
public void visitMaxs(int maxStack, int maxLocals) {
super.visitMaxs(maxStack, maxLocals);
}
}
```

在 ognl.Ognl 类中的 parseExpression 方法执行前，Hook 了方法的执行，跳转至执行自定义的 OgnlFilter 。 OgnlFilter 中定义了如何对 Ognl 表达式进行过滤。如果出现了威胁的表达式，将进行log记录并抛出异常，若正常将放过，继续进行 parseExpression 。

#### 0x07 结尾

这里介绍了 Java Rasp 实现的基本原理，除了 ognl.Ognl 类中的 parseExpression 方法加探针外，还有很多的地方可以加探针，比如：`java/io/ObjectInputStream`、`java/lang/ProcessBuilder`、`com/mysql/jdbc/StatementImpl` 等等。重点关注数据的关键流转节点加入 Rasp 探针，进行安全过滤。

如果探针部署的足够充分，可以有效的防御 XSS、CSRF、RCE、SQL 注入等 Web 攻击。如果 Rasp 与云端结合，不但能够采集应用的安全日志，也能够对发现的漏洞进行迅速的修补，甚至抵御 0Day 攻击。

Demo: https://github.com/xbeark/javaopenrasp

这里只实现了使用了 Instrumentation 的 premain 进行代理，其实还可以使用 agentmain 进行虚拟机启动后的动态 instrument ，具体就不在这里研究啦~



[Rasp 技术介绍与实现 (seebug.org)](https://paper.seebug.org/330/)