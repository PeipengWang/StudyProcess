## **1. 引入 Aviator 依赖**

在您的 `pom.xml` 文件中添加 Aviator 的依赖：

```
<dependency>
    <groupId>com.googlecode.aviator</groupId>
    <artifactId>aviator</artifactId>
    <version>5.3.1</version>
</dependency>

```

## **2. 使用 Aviator 计算数学函数**

Aviator 支持许多其他数学函数，以下是一些常用的函数：

- `abs(x)`: 计算绝对值
- `ceil(x)`: 向上取整
- `floor(x)`: 向下取整
- `pow(x, y)`: 计算 `x` 的 `y` 次幂
- `max(a, b)`: 返回较大值
- `min(a, b)`: 返回较小值
- `sin(x)`, `cos(x)`, `tan(x)`: 三角函数

## 3、逻辑判断

```
        String expression = "a==5&&b==6";
        Map<String, Object> env = new HashMap<>();
        env.put("a", 5);
        env.put("b", 6);
        Expression compile = AviatorEvaluator.compile(expression);

        Object execute = compile.execute(env);

        Boolean result = (Boolean) execute;
        System.out.println("Result: " + result);  // Output: true
```

## 4、基本运算

AviatorScript 支持常见的类型，如数字、布尔值、字符串等等，同时将大整数、BigDecimal、正则表达式也作为一种基本类型来支持。

首先介绍下数字。

### 数字

数字包括整数和浮点数，AviatorScript 对 java 的类型做了缩减和扩展，同时保持了一致的运算符规则。

整数和算术运算

整数例如 -99、0、1、2、100……等等，对应的类型是 java 中的 long 类型。**AviatorScript 中并没有 byte/short/int 等类型，统一整数类型都为 long**，支持的范围也跟 java 语言一样：-9223372036854774808~9223372036854774807。

整数也可以用十六进制表示，以 `0x` 或者 `0X` 开头的数字，比如 0xFF(255)、0xAB(171) 等等。

整数可以参与所有的算术运算，比如加减乘除和取模等等。

```java
let a = 99;
let b = 0xFF;
let c = -99;

println(a + b);
println(a / b);
println(a- b + c);
println(a + b * c);
println(a- (b - c));
println(a/b * b + a % b);
```

加减乘除对应的运算符就是 `+,-,*,/` 这都比较好理解，取模运算符就是 `%` ，规则和语法和 java 是一样的。

**需要注意，整数相除的结果仍然是整数，比如例子中的**  `**a/b**` **结果就是 0，遵循 java 的整数运算规则。**

运算符之间的优先级如下：

- 单目运算符 `-` 取负数
- `*, /` 
- `+,-` 

整个规则也跟 java 的运算符优先级保持一致。你可以通过括号来强制指定优先级，比如例子中的 `a-(b-c)` 就是通过括号，强制先执行 `b-c` ，再后再被 a 减。

**通常来说，复杂的算术表达式，从代码可读性和稳健角度，都推荐使用括号来强制指定优先级。**

### 大整数(BigInt)

对于超过 long 的整数， AviatorScript 还特别提供了大整数类型的支持，对应 `java.math.BigInteger` 类。任何超过 long 范围的整数**字面量**，会自动提升为 `BigInteger` 对象（以下简称 BigInt)，任何数字以 `N` 字母结尾就自动变 BigInt：

```java
## examples/bigint.av

let a = 10223372036854774807;  ## Literal BigInteger
let b = 1000N;  ## BigInteger
let c = 1000; ## long type

println(a);

println(a + a);

println(a * a);

println(a + b + c);
```

请注意，**默认的 long 类型在计算后如果超过范围溢出后，不会自动提升为 BigInt，但是 BigInt 和 long 一起参与算术运算的时候，结果为 BigInt 类型。关于类型转换的规则，我们后面再详细介绍。**



### 浮点数

数字除了整数之外，AviatorScript 同样支持浮点数，但是仅支持 double 类型，也就是双精度 64 位，符合 IEEE754 规范的浮点数。传入的 java float 也将转换为 double 类型。所有的浮点数都被认为是 double 类型。浮点数的表示形式有两种：

1. 十进制的带小数点的数字，比如 `1.34159265` ， `0.33333` 等等。
2. 科学计数法表示，如 `1e-2` ， `2E3` 等等，大小写字母 `e` 皆可。



看一个简单例子，牛顿法求平方根

```
## examples/square_root.av

let a = 2;
let err = 1e-15;
let root = a;

while math.abs(a - root * root) > err {
  root = (a/root + root) / 2.0;
}

println("square root of 2 is: " + root);
```



## 5、自定义函数

```java
import com.googlecode.aviator.AviatorEvaluator;
import com.googlecode.aviator.runtime.function.AbstractFunction;
import com.googlecode.aviator.runtime.type.AviatorDouble;
import com.googlecode.aviator.runtime.type.AviatorObject;

import java.util.HashMap;
import java.util.Map;

// 定义 ln 函数
class LnFunction extends AbstractFunction {
    @Override
    public AviatorObject call(Map<String, Object> env, AviatorObject arg) {
        Double value = (Double) arg.getValue(env);
        return new AviatorDouble(Math.log(value));  // 使用 Math.log 计算自然对数
    }

    @Override
    public String getName() {
        return "ln";  // 自定义的 ln 函数名
    }
}

// 定义 log 函数
class LogFunction extends AbstractFunction {
    @Override
    public AviatorObject call(Map<String, Object> env, AviatorObject arg) {
        Double value = (Double) arg.getValue(env);
        return new AviatorDouble(Math.log10(value));  // 使用 Math.log10 计算常用对数
    }

    @Override
    public String getName() {
        return "log";  // 自定义的 log 函数名
    }
}

// 定义 sqrt 函数
class SqrtFunction extends AbstractFunction {
    @Override
    public AviatorObject call(Map<String, Object> env, AviatorObject arg) {
        Double value = (Double) arg.getValue(env);
        return new AviatorDouble(Math.sqrt(value));  // 使用 Math.sqrt 计算平方根
    }

    @Override
    public String getName() {
        return "sqrt";  // 自定义的 sqrt 函数名
    }
}
```

计算

```java
public class AviatorMathExample {
    public static void main(String[] args) {
        // 注册自定义的 ln、log 和 sqrt 函数
        AviatorEvaluator.addFunction(new LnFunction());
        AviatorEvaluator.addFunction(new LogFunction());
        AviatorEvaluator.addFunction(new SqrtFunction());

        // 定义要计算的数值
        double n = 10.0;

        // 创建变量环境
        Map<String, Object> env = new HashMap<>();
        env.put("n", n);

        // 计算 ln(n)
        String lnExpression = "ln(n)";
        Object lnResult = AviatorEvaluator.execute(lnExpression, env);
        System.out.println("ln(" + n + ") = " + lnResult);

        // 计算 log(n)
        String logExpression = "log(n)";
        Object logResult = AviatorEvaluator.execute(logExpression, env);
        System.out.println("log(" + n + ") = " + logResult);

        // 计算 sqrt(n)
        String sqrtExpression = "sqrt(n)";
        Object sqrtResult = AviatorEvaluator.execute(sqrtExpression, env);
        System.out.println("sqrt(" + n + ") = " + sqrtResult);

        // 组合计算：ln(n) + log(n) + sqrt(n)
        String combinedExpression = "ln(n) + log(n) + sqrt(n)";
        Object combinedResult = AviatorEvaluator.execute(combinedExpression, env);
        System.out.println("ln(" + n + ") + log(" + n + ") + sqrt(" + n + ") = " + combinedResult);
    }
}

```

输出

```
com.example.aviatordemo.test2.AviatorMathExample
ln(10.0) = 2.302585092994046
log(10.0) = 1.0
sqrt(10.0) = 3.1622776601683795
ln(10.0) + log(10.0) + sqrt(10.0) = 6.464862753162425

Process finished with exit code 0
```

## 系统函数

| 函数名称                              | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| assert(predicate, [msg])              | 断言函数，当 predicate 的结果为 false 的时候抛出 AssertFailed 异常， msg 错误信息可选。 |
| sysdate()                             | 返回当前日期对象 java.util.Date                              |
| rand()                                | 返回一个介于 [0, 1) 的随机数，结果为 double 类型             |
| rand(n)                               | 返回一个介于 [0, n) 的随机数，结果为 long 类型               |
| cmp(x, y)                             | 比较 x 和 y 大小，返回整数，0 表示相等， 1 表达式 x > y，负数则 x < y。 |
| print([out],obj)                      | 打印对象,如果指定 out 输出流，向 out 打印， 默认输出到标准输出 |
| println([out],obj) 或者 p([out], obj) | 与 print 类似,但是在输出后换行                               |
| pst([out], e);                        | 等价于 e.printStackTrace()，打印异常堆栈，out 是可选的输出流，默认是标准错误输出 |
| now()                                 | 返回 System.currentTimeMillis() 调用值                       |
| long(v)                               | 将值转为 long 类型                                           |
| double(v)                             | 将值转为 double 类型                                         |
| boolean(v)                            | 将值的类型转为 boolean，除了 nil 和 false，其他都值都将转为布尔值 true。 |
| str(v)                                | 将值转为 string 类型，如果是 nil（或者 java null），会转成字符串 'null' |
| bigint(x)                             | 将值转为 bigint 类型                                         |
| decimal(x)                            | 将值转为 decimal 类型                                        |
| identity(v)                           | 返回参数 v 自身，用于跟 seq 库的高阶函数配合使用。           |
| type(x)                               | 返回参数 x 的类型，结果为字符串，如 string, long, double, bigint, decimal, function 等。Java 类则返回完整类名。 |
| is_a(x, class)                        | 当 x 是类 class 的一个实例的时候，返回 true，例如 `is_a("a", String)` ，class 是类名。 |
| is_def(x)                             | 返回变量 x 是否已定义（包括定义为 nil），结果为布尔值        |
| undef(x)                              | “遗忘”变量 x，如果变量 x 已经定义，将取消定义。              |
| range(start, end, [step])             | 创建一个范围，start 到 end 之间的整数范围，不包括 end， step 指定递增或者递减步幅。 |
| tuple(x1, x2, ...)                    | 创建一个 Object 数组，元素即为传入的参数列表。               |
| eval(script, [bindings], [cached])    | 对一段脚本文本 script 进行求值，等价于 AviatorEvaluator.execute(script, env, cached) |
| comparator(pred)                      | 将一个谓词（返回布尔值）转化为 java.util.Comparator 对象，通常用于 sort 函数。 |
| max(x1, x2, x3, ...)                  | 取所有参数中的最大值，比较规则遵循逻辑运算符规则。           |
| min(x1, x2, x3, ...)                  | 取所有参数中的最小值，比较规则遵循逻辑运算符规则。           |
| constantly(x)                         | 用于生成一个函数，它对任意（个数）参数的调用结果 x。         |





## 字符串函数

| date_to_string(date,format)               | 将 Date 对象转化化特定格式的字符串,2.1.1 新增                |
| ----------------------------------------- | ------------------------------------------------------------ |
| string_to_date(source,format)             | 将特定格式的字符串转化为 Date 对 象,2.1.1 新增               |
| string.contains(s1,s2)                    | 判断 s1 是否包含 s2,返回 Boolean                             |
| string.length(s)                          | 求字符串长度,返回 Long                                       |
| string.startsWith(s1,s2)                  | s1 是否以 s2 开始,返回 Boolean                               |
| string.endsWith(s1,s2)                    | s1 是否以 s2 结尾,返回 Boolean                               |
| string.substring(s,begin[,end])           | 截取字符串 s,从 begin 到 end,如果忽略 end 的话,将从 begin 到结尾,与 java.util.String.substring 一样。 |
| string.indexOf(s1,s2)                     | java 中的 s1.indexOf(s2),求 s2 在 s1 中 的起始索引位置,如果不存在为-1 |
| string.split(target,regex,[limit])        | Java 里的 String.split 方法一致,2.1.1 新增函数               |
| string.join(seq,seperator)                | 将集合 seq 里的元素以 seperator 为间隔 连接起来形成字符串,2.1.1 新增函数 |
| string.replace_first(s,regex,replacement) | Java 里的 String.replaceFirst 方法, 2.1.1 新增               |
| string.replace_all(s,regex,replacement)   | Java 里的 String.replaceAll 方法 , 2.1.1 新增                |



## 数学函数

| math.abs(d)     | 求 d 的绝对值         |
| --------------- | --------------------- |
| math.round(d)   | 四舍五入              |
| math.floor(d)   | 向下取整              |
| math.ceil(d)    | 向上取整              |
| math.sqrt(d)    | 求 d 的平方根         |
| math.pow(d1,d2) | 求 d1 的 d2 次方      |
| math.log(d)     | 求 d 的自然对数       |
| math.log10(d)   | 求 d 以 10 为底的对数 |
| math.sin(d)     | 正弦函数              |
| math.cos(d)     | 余弦函数              |
| math.tan(d)     | 正切函数              |
| math.atan(d)    | 反正切函数            |
| math.acos(d)    | 反余弦函数            |
| math.asin(d)    | 反正弦函数            |



## Sequence 函数（集合处理）

| 函数名称                                     | 说明                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| repeat(n, x)                                 | 返回一个 List，将元素 x 重复 n 次组合而成。                  |
| repeatedly(n, f)                             | 返回一个 List，将函数 f 重复调用 n 次的结果组合而成。        |
| seq.array(clazz, e1, e2,e3, ...)             | 创建一个指定 clazz 类型的数组，并添加参数 e1,e2,e3 ...到这个数组并返回。 clazz 可以是类似 java.lang.String 的类型，也可以是原生类型，如 int/long/float 等 |
| seq.array_of(clazz, size1, size2, ...sizes)  | 创建 clazz 类型的一维或多维数组，维度大小为 sizes 指定。clazz 同 seq.array 定义。 |
| seq.list(p1, p2, p3, ...)                    | 创建一个 java.util.ArrayList 实例，添加参数到这个集合并返回。 |
| seq.set(p1, p2, p3, ...)                     | 创建一个 java.util.HashSet 实例，添加参数到这个集合并返回。  |
| seq.map(k1, v1, k2, v2, ...)                 | 创建一个 java.util.HashMap 实例，参数要求偶数个，类似 k1,v1 这样成对作为 key-value 存入 map，返回集合。 |
| seq.entry(key, value)                        | 创建 Map.Entry 对象，用于 map, filter 等函数                 |
| seq.keys(m)                                  | 返回 map 的 key 集合                                         |
| seq.vals(m)                                  | 返回 map 的 value 集合                                       |
| into(to_seq, from_seq)                       | 用于 sequence 转换，将 from sequence 的元素使用 `seq.add` 函数逐一添加到了 to sequence 并返回最终的 to_seq |
| seq.contains_key(map, key)                   | 当 map 中存在 key 的时候（可能为 null），返回 true。对于数组和链表，key 可以是 index，当 index 在有效范围[0..len-1]，返回 true，否则返回 false |
| seq.add(coll, element)seq.add(m, key, value) | 往集合 coll 添加元素，集合可以是 java.util.Collection，也可以是 java.util.Map（三参数版本） |
| seq.add_all(seq1, seq2)                      | 将集合 seq2 的元素全部添加到 seq1， 5.3.3 版本新增函数       |
| seq.put(coll, key, value)                    | 类似 List.set(i, v)。用于设置 seq 在 key 位置的值为 value，seq 可以是 map ，数组或者 List。 map 就是键值对， 数组或者 List 的时候， key 为索引位置整数，value 即为想要放入该索引位置的值。 |
| seq.remove(coll, element)                    | 从集合或者 hash map 中删除元素或者 key                       |
| seq.get(coll, element)                       | 从 list、数组或者 hash-map 获取对应的元素值，对于 list 和数组， element 为元素的索引位置（从 0 开始），对于 hash map 来说， element 为 key。 |
| map(seq,fun)                                 | 将函数 fun 作用到集合 seq 每个元素上, 返回新元素组成的集合   |
| filter(seq,predicate)                        | 将谓词 predicate 作用在集合的每个元素 上,返回谓词为 true 的元素组成的集合 |
| count(seq)                                   | 返回集合大小，seq 可以是数组，字符串，range ，List 等等      |
| is_empty(seq)                                | 等价于 count(seq) == 0，当集合为空或者 nil，返回 true        |
| distinct(seq)                                | 返回 seq 去重后的结果集合。                                  |
| is_distinct(seq)                             | 当 seq 没有重复元素的时候，返回 true，否则返回 false         |
| concat(seq1, seq2)                           | 将 seq1 和 seq2 “连接”，返回连接后的结果，复杂度 O(m+n)， m 和 n 分别是两个集合的长度。 |
| include(seq,element)                         | 判断 element 是否在集合 seq 中,返回 boolean 值，对于 java.uitl.Set 是 O(1) 时间复杂度，其他为 O(n) |
| sort(seq, [comparator])                      | 排序集合,仅对数组和 List 有效,返回排序后的新集合，comparator 是一个 java.util.Comparator 实例，可选排序方式。 |
| reverse(seq)                                 | 将集合元素逆序，返回新的集合。                               |
| reduce(seq,fun,init)                         | fun 接收两个参数,第一个是集合元素, 第二个是累积的函数,本函数用于将 fun 作用在结果值（初始值为 init 指定)和集合的每个元素上面，返回新的结果值；函数返回最终的结果值 |
| take_while(seq, pred)                        | 遍历集合 seq，对每个元素调用 pred(x)，返回 true则加入结果集合，最终返回收集的结果集合。也就是说从集合 seq 收集 pred 调用为 true 的元素。 |
| drop_while(seq, pred)                        | 与 take_while 相反，丢弃任何 pred(x) 为 true 的元素并返回最终的结果集合。 |
| group_by(seq, keyfn)                         | 对集合 seq 的元素按照 keyfn(x) 的调用结果做分类，返回最终映射 map。具体使用见[文档](https://www.yuque.com/boyan-avfmj/aviatorscript/yc4l93#uCW1U)。 |
| zipmap(keys, values)                         | 返回一个 HashMap，其中按照 keys 和 values 两个集合的顺序映射键值对。具体使用见[文档](https://www.yuque.com/boyan-avfmj/aviatorscript/yc4l93#6UFhw)。 |
| seq.every(seq, fun)                          | fun 接收集合的每个元素作为唯一参数，返回 true 或 false。当集合里的每个元素调用 fun 后都返回 true 的时候，整个调用结果为 true，否则为 false。 |
| seq.not_any(seq, fun)                        | fun 接收集合的每个元素作为唯一参数，返回 true 或 false。当集合里的每个元素调用 fun 后都返回 false 的时候，整个调用结果为 true，否则为 false。 |
| seq.some(seq, fun)                           | fun 接收集合的每个元素作为唯一参数，返回 true 或 false。当集合里的只要有一个元素调用 fun 后返回 true 的时候，整个调用结果立即为该元素，否则为 nil。 |
| seq.eq(value)                                | 返回一个谓词,用来判断传入的参数是否跟 value 相等,用于 filter 函数,如filter(seq,seq.eq(3)) 过滤返回等于3 的元素组成的集合 |
| seq.neq(value)                               | 与 seq.eq 类似,返回判断不等于的谓词                          |
| seq.gt(value)                                | 返回判断大于 value 的谓词                                    |
| seq.ge(value)                                | 返回判断大于等于 value 的谓词                                |
| seq.lt(value)                                | 返回判断小于 value 的谓词                                    |
| seq.le(value)                                | 返回判断小于等于 value 的谓词                                |
| seq.nil()                                    | 返回判断是否为 nil 的谓词                                    |
| seq.exists()                                 | 返回判断不为 nil 的谓词                                      |
| seq.and(p1, p2, p3, ...)                     | 组合多个谓词函数，返回一个新的谓词函数，当今仅当 p1、p2、p3 ...等所有函数都返回 true 的时候，新函数返回 true |
| seq.or(p1, p2, p3, ...)                      | 组合多个谓词函数，返回一个新的谓词函数，当 p1, p2, p3... 其中一个返回 true 的时候，新函数立即返回 true，否则返回 false。 |
| seq.min(coll)                                | 返回集合中的最小元素，要求集合元素可比较（实现 Comprable 接口），比较规则遵循 aviator 规则。 |
| seq.max(coll)                                | 返回集合中的最大元素，要求集合元素可比较（实现 Comprable 接口），比较规则遵循 aviator 规则。 |



## 模块加载



| load(path)    | 加载 path 路径指定的模块，每次都将重新编译该模块文件并返回 exports |
| ------------- | ------------------------------------------------------------ |
| require(path) | 加载 path 路径指定的模块，如果已经有缓存，将直接返回结果，否则将编译并缓存 exports |