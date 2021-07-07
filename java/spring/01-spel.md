# 01-spel

> ​	SpEL（Spring Expression Language），即Spring表达式语言，是比JSP的EL更强大的一种表达式语言。为什么要总结SpEL，因为它可以在运行时查询和操作数据，尤其是数组列表型数据，因此可以缩减代码量，优化代码结构。个人认为很有用

## 1. Spel能干什么?

表达式语言一般是用最简单的形式完成最主要的工作，减少我们的工作量。

SpEL支持如下表达式：

**一、基本表达式：** 字面量表达式、关系，逻辑与算数运算表达式、字符串连接及截取表达式、三目运算及Elivis表达式、正则表达式、括号优先级表达式；

**二、类相关表达式：** 类类型表达式、类实例化、instanceof表达式、变量定义及引用、赋值表达式、自定义函数、对象属性存取及安全导航表达式、对象方法调用、Bean引用；

**三、集合相关表达式：** 内联List、内联数组、集合，字典访问、列表，字典，数组修改、集合投影、集合选择；不支持多维内联数组初始化；不支持内联字典定义；

**四、其他表达式**：模板表达式。

**注：SpEL表达式中的关键字是不区分大小写的。**

## 2.实现方式

表达式解析：

```java
public interface ExpressionParser {
	Expression parseExpression(String expressionString) throws ParseException;
	Expression parseExpression(String expressionString, ParserContext context) throws ParseException;
}
```

输入：

```java
public interface ParserContext {
    // 是否是模板
    boolean isTemplate();
	// 前缀
	String getExpressionPrefix();
	// 后缀
	String getExpressionSuffix();
}
```

计算出来的表达式

```java
public interface Expression {}
```

eg:

```java
@Test
public void test11() {
    //创建解析器
    SpelExpressionParser parser = new SpelExpressionParser();
    //创建解析器上下文
    ParserContext context = new TemplateParserContext("%{", "}");
    Expression expression = parser.parseExpression("你好:%{#name},我们正在学习:%{#lesson}", context);

    //创建表达式计算上下文
    EvaluationContext evaluationContext = new StandardEvaluationContext();
    evaluationContext.setVariable("name", "路人甲java");
    evaluationContext.setVariable("lesson", "spring高手系列!");
    //获取值
    String value = expression.getValue(evaluationContext, String.class);
    System.out.println(value);
}
你好:路人甲java,我们正在学习:spring高手系列!
```

## 3.用的例子