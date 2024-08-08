## 简述

Spring表达式语言（简称SpEl）是一个支持查询和操作运行时对象导航图功能的强大的表达式语言. 它的语法类似于传统EL，但提供额外的功能，最出色的就是函数调用和简单字符串的模板函数。

## 功能概述

- 文字表达式
- 布尔和关系运算符
- 正则表达式
- 类表达式
- 访问 properties, arrays, lists, maps
- 方法调用
- 关系运算符
- 参数
- 调用构造函数
- Bean引用
- 构造Array
- 内嵌lists
- 内嵌maps
- 三元运算符
- 变量
- 用户定义的函数
- 集合投影
- 集合筛选
- 模板表达式

## 基础
首先准备支持SpEL的Jar包：“org.springframework.expression-3.0.5.RELEASE.jar”将其添加到类路径中。

SpEL在求表达式值时一般分为四步，其中第三步可选：首先构造一个解析器，其次解析器解析字符串表达式，在此构造上下文，最后根据上下文得到表达式运算后的值。

默认接口

```java
public interface ExpressionParser {
    Expression parseExpression(String expressionString) throws ParseException;
    Expression parseExpression(String expressionString, ParserContext context) throws ParseException;
}
```

```java
package com.javacode2018.spel;

import org.junit.Test;
import org.springframework.expression.EvaluationContext;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;

public class SpelTest {
    @Test
    public void test1() {
        ExpressionParser parser = new SpelExpressionParser();
        Expression expression = parser.parseExpression("('Hello' + ' World').concat(#end)");
        EvaluationContext context = new StandardEvaluationContext();
        context.setVariable("end", "!");
        System.out.println(expression.getValue(context));
    }
}
```

## 表达式

### 字面量表达式
|类型|示例|
|----|----|
|字符串|String str1 = parser.parseExpression("'Hello World!'").getValue(String.class);|
|数字类型|int int1 = parser.parseExpression("1").getValue(Integer.class);long long1 = parser.parseExpression("-1L").getValue(long.class);float float1 = parser.parseExpression("1.1").getValue(Float.class);double double1 = parser.parseExpression("1.1E+2").getValue(double.class);int hex1 = parser.parseExpression("0xa").getValue(Integer.class);long hex2 = parser.parseExpression("0xaL").getValue(long.class);|  
|布尔类型|boolean true1 = parser.parseExpression("true").getValue(boolean.class);boolean false1 = parser.parseExpression("false").getValue(boolean.class);|
|null类型|Object null1 = parser.parseExpression("null").getValue(Object.class);|
|加减乘除|int result1 = parser.parseExpression("1+2-3*4/2").getValue(Integer.class);//-3|
|求余|int result2 = parser.parseExpression("4%3").getValue(Integer.class);//1|
|幂运算|int result3 = parser.parseExpression("2^3").getValue(Integer.class);//8|

关系表达式
等于（==）、不等于(!=)、大于(>)、大于等于(>=)、小于(<)、小于等于(<=)，区间（between）运算。

如parser.parseExpression("1>2").getValue(boolean.class);将返回false；

而parser.parseExpression("1 between {1, 2}").getValue(boolean.class);将返回true。

between运算符右边操作数必须是列表类型，且只能包含2个元素。第一个元素为开始，第二个元素为结束，区间运算是包含边界值的，即 xxx>=list.get(0) && xxx<=list.get(1)。

SpEL同样提供了等价的“EQ” 、“NE”、 “GT”、“GE”、 “LT” 、“LE”来表示等于、不等于、大于、大于等于、小于、小于等于，不区分大小写。

逻辑表达式
且（and或者&&）、或(or或者||)、非(!或NOT)。

...

https://itmyhome.com/spring/expressions.html

https://cloud.tencent.com/developer/article/1676200