---
layout: post
title: 'java8-Streams 学习及应用'
subtitle: 'Streams 学习应用'
date: 2018-07-20
categories: 技术
tags: java 
---

## 1. Streams API 简介

Stream 相当于一个高级版本的迭代器，但与迭代器不同的是，Stream 可以进行并行化操作，迭代器只能命令式地 串行化操作。使用Stream 可以更加高效地处理数据。

> 流的构成

获取一个数据源（source）→ 数据转换→执行操作获取想要的结果


>  **生成流的方式**

1. 从 Collection 和数组

* Collection.stream()
* Collection.parallelStream()
* Arrays.stream(T array) or Stream.of()
* java.io.BufferedReader.lines()

2. 静态工厂

* java.util.stream.IntStream.range()
* java.nio.file.Files.walk()

3. 自己构建

* java.util.Spliterator

4. 其他

* Random.ints()
* BitSet.stream()
* Pattern.splitAsStream(java.lang.CharSequence)
* JarFile.stream()

> **流的操作类型**

* Intermediate

一个流可以后面跟随零个或多个 intermediate 操作。其目的主要是打开流，做出某种程度的数据映射/过滤，然后返回一个新的流，交给下一个操作使用。这类操作都是惰性化的（lazy），就是说，仅仅调用到这类方法，并没有真正开始流的遍历。

* Terminal

一个流只能有一个 terminal 操作，当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。Terminal 操作的执行，才会真正开始流的遍历，并且会生成一个结果，或者一个 side effect。

* short-circuiting

 对于一个 intermediate 操作，如果它接受的是一个无限大（infinite/unbounded）的 Stream，但返回一个有限的新 Stream。

 对于一个 terminal 操作，如果它接受的是一个无限大的 Stream，但能在有限的时间计算出结果。

当操作一个无限大的 Stream，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

## 2. Streams API 应用详解

当把一个数据结构包装成 Stream 后，就要开始对里面的元素进行各类操作了。常见的操作可以归类如下。

*   Intermediate：

map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

*   Terminal：

forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

*   Short-circuiting：

anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 limit

我们下面看一下 Stream 的比较典型用法。

**map/flatMap**

我们先来看 map。如果你熟悉 scala 这类函数式语言，对这个方法应该很了解，它的作用就是把 input Stream 的每一个元素，映射成 output Stream 的另外一个元素。

'''java
`List<``String``> output = wordList.stream().`

`map(String::toUpperCase).`

`collect(Collectors.toList());`
'''

这段代码把所有的单词转换为大写。


`List<``Integer``> nums = Arrays.asList(1, 2, 3, 4);`

`List<``Integer``> squareNums = nums.stream().`

`map(n -> n * n).`

`collect(Collectors.toList());`


这段代码生成一个整数 list 的平方数 {1, 4, 9, 16}。

从上面例子可以看出，map 生成的是个 1:1 映射，每个输入元素，都按照规则转换成为另外一个元素。还有一些场景，是一对多映射关系的，这时需要 flatMap。

`Stream<``List``<Integer>> inputStream = Stream.of(`

`Arrays.asList(1),`

`Arrays.asList(2, 3),`

`Arrays.asList(4, 5, 6)`

`);`

`Stream<``Integer``> outputStream = inputStream.`

`flatMap((childList) -> childList.stream());`

flatMap 把 input Stream 中的层级结构扁平化，就是将最底层元素抽出来放到一起，最终 output 的新 Stream 里面已经没有 List 了，都是直接的数字。

**filter**

filter 对原始 Stream 进行某项测试，通过测试的元素被留下来生成一个新 Stream。

`Integer[] sixNums = {1, 2, 3, 4, 5, 6};`

`Integer[] evens =`

`Stream.of(sixNums).filter(n -> n%2 == 0).toArray(Integer[]::new);`

经过条件“被 2 整除”的 filter，剩下的数字为 {2, 4, 6}。

`List<``String``> output = reader.lines().`

`flatMap(line -> Stream.of(line.split(REGEXP))).`

`filter(word -> word.length() > 0).`

`collect(Collectors.toList());`

这段代码首先把每行的单词用 flatMap 整理到新的 Stream，然后保留长度不为 0 的，就是整篇文章中的全部单词了。

**forEach**

forEach 方法接收一个 Lambda 表达式，然后在 Stream 的每一个元素上执行该表达式。

打印姓名（forEach 和 pre-java8 的对比）



`// Java 8`

`roster.stream()`

`.filter(p -> p.getGender() == Person.Sex.MALE)`

`.forEach(p -> System.out.println(p.getName()));`

`// Pre-Java 8`

`for (Person p : roster) {`

`if (p.getGender() == Person.Sex.MALE) {`

`System.out.println(p.getName());`

`}`

`}`

对一个人员集合遍历，找出男性并打印姓名。可以看出来，forEach 是为 Lambda 而设计的，保持了最紧凑的风格。而且 Lambda 表达式本身是可以重用的，非常方便。当需要为多核系统优化时，可以 parallelStream().forEach()，只是此时原有元素的次序没法保证，并行的情况下将改变串行时操作的行为，此时 forEach 本身的实现不需要调整，而 Java8 以前的 for 循环 code 可能需要加入额外的多线程逻辑。

但一般认为，forEach 和常规 for 循环的差异不涉及到性能，它们仅仅是函数式风格与传统 Java 风格的差别。

另外一点需要注意，forEach 是 terminal 操作，因此它执行后，Stream 的元素就被“消费”掉了，你无法对一个 Stream 进行两次 terminal 运算。下面的代码是错误的：



`stream.forEach(element -> doOneThing(element));`

`stream.forEach(element -> doAnotherThing(element));`


相反，具有相似功能的 intermediate 操作 peek 可以达到上述目的。如下是出现在该 api javadoc 上的一个示例。

peek 对每个元素执行操作并返回一个新的 Stream

`Stream.of("one", "two", "three", "four")`

`.filter(e -> e.length() > 3)`

`.peek(e -> System.out.println("Filtered value: " + e))`

`.map(String::toUpperCase)`

`.peek(e -> System.out.println("Mapped value: " + e))`

`.collect(Collectors.toList());`

forEach 不能修改自己包含的本地变量值，也不能用 break/return 之类的关键字提前结束循环。

**findFirst**

这是一个 termimal 兼 short-circuiting 操作，它总是返回 Stream 的第一个元素，或者空。

这里比较重点的是它的返回值类型：Optional。这也是一个模仿 Scala 语言中的概念，作为一个容器，它可能含有某值，或者不包含。使用它的目的是尽可能避免 NullPointerException。

Optional 的两个用例

`String strA = " abcd ", strB = null;`

`print(strA);`

`print("");`

`print(strB);`

`getLength(strA);`

`getLength("");`

`getLength(strB);`

`public static void print(String text) {`

`// Java 8`

`Optional.ofNullable(text).ifPresent(System.out::println);`

`// Pre-Java 8`

`if (text != null) {`

`System.out.println(text);`

`}`

`}`

`public static int getLength(String text) {`

`// Java 8`

`return Optional.ofNullable(text).map(String::length).orElse(-1);`

`// Pre-Java 8`

`// return if (text != null) ? text.length() : -1;`

`};`

在更复杂的 if (xx != null) 的情况中，使用 Optional 代码的可读性更好，而且它提供的是编译时检查，能极大的降低 NPE 这种 Runtime Exception 对程序的影响，或者迫使程序员更早的在编码阶段处理空值问题，而不是留到运行时再发现和调试。

Stream 中的 findAny、max/min、reduce 等方法等返回 Optional 值。还有例如 IntStream.average() 返回 OptionalDouble 等等。

**reduce**

这个方法的主要作用是把 Stream 元素组合起来。它提供一个起始值（种子），然后依照运算规则（BinaryOperator），和前面 Stream 的第一个、第二个、第 n 个元素组合。从这个意义上说，字符串拼接、数值的 sum、min、max、average 都是特殊的 reduce。例如 Stream 的 sum 就相当于

Integer sum = integers.reduce(0, (a, b) -> a+b); 或

Integer sum = integers.reduce(0, Integer::sum);

也有没有起始值的情况，这时会把 Stream 的前面两个元素组合起来，返回的是 Optional。

 reduce 的用例

`// 字符串连接，concat = "ABCD"`

`String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat);`

`// 求最小值，minValue = -3.0`

`double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min);`

`// 求和，sumValue = 10, 有起始值`

`int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);`

`// 求和，sumValue = 10, 无起始值`

`sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();`

`// 过滤，字符串连接，concat = "ace"`

`concat = Stream.of("a", "B", "c", "D", "e", "F").`

`filter(x -> x.compareTo("Z") > 0).`

`reduce("", String::concat);`

上面代码例如第一个示例的 reduce()，第一个参数（空白字符）即为起始值，第二个参数（String::concat）为 BinaryOperator。这类有起始值的 reduce() 都返回具体的对象。而对于第四个示例没有起始值的 reduce()，由于可能没有足够的元素，返回的是 Optional，请留意这个区别。

**limit/skip**

limit 返回 Stream 的前面 n 个元素；skip 则是扔掉前 n 个元素（它是由一个叫 subStream 的方法改名而来）。

limit 和 skip 对运行次数的影响

`public void testLimitAndSkip() {`

`List<``Person``> persons = new ArrayList();`

`for (int i = 1; i <= 10000; i++) {`

`Person person = new Person(i, "name" + i);`

`persons.add(person);`

`}`

`List<``String``> personList2 = persons.stream().`

`map(Person::getName).limit(10).skip(3).collect(Collectors.toList());`

`System.out.println(personList2);`

`}`

`private class Person {`

`public int no;`

`private String name;`

`public Person (int no, String name) {`

`this.no = no;`

`this.name = name;`

`}`

`public String getName() {`

`System.out.println(name);`

`return name;`

`}`

`}`

输出结果为：

`name1`

`name2`

`name3`

`name4`

`name5`

`name6`

`name7`

`name8`

`name9`

`name10`

`[name4, name5, name6, name7, name8, name9, name10]`


这是一个有 10，000 个元素的 Stream，但在 short-circuiting 操作 limit 和 skip 的作用下，管道中 map 操作指定的 getName() 方法的执行次数为 limit 所限定的 10 次，而最终返回结果在跳过前 3 个元素后只有后面 7 个返回。

有一种情况是 limit/skip 无法达到 short-circuiting 目的的，就是把它们放在 Stream 的排序操作后，原因跟 sorted 这个 intermediate 操作有关：此时系统并不知道 Stream 排序后的次序如何，所以 sorted 中的操作看上去就像完全没有被 limit 或者 skip 一样。

limit 和 skip 对 sorted 后的运行次数无影响

`List<``Person``> persons = new ArrayList();`

`for (int i = 1; i <= 5; i++) {`

`Person person = new Person(i, "name" + i);`

`persons.add(person);`

`}`

`List<``Person``> personList2 = persons.stream().sorted((p1, p2) ->`

`p1.getName().compareTo(p2.getName())).limit(2).collect(Collectors.toList());`

`System.out.println(personList2);`

上面的示例对清单 13 做了微调，首先对 5 个元素的 Stream 排序，然后进行 limit 操作。输出结果为：

`name2`

`name1`

`name3`

`name2`

`name4`

`name3`

`name5`

`name4`

`[stream.StreamDW$Person@816f27d, stream.StreamDW$Person@87aac27]`

即虽然最后的返回元素数量是 2，但整个管道中的 sorted 表达式执行次数没有像前面例子相应减少。

最后有一点需要注意的是，对一个 parallel 的 Steam 管道来说，如果其元素是有序的，那么 limit 操作的成本会比较大，因为它的返回对象必须是前 n 个也有一样次序的元素。取而代之的策略是取消元素间的次序，或者不要用 parallel Stream。

**sorted**

对 Stream 的排序通过 sorted 进行，它比数组的排序更强之处在于你可以首先对 Stream 进行各类 map、filter、limit、skip 甚至 distinct 来减少元素数量后，再排序，这能帮助程序明显缩短执行时间。我们对清单 14 进行优化：

##### 优化：排序前进行 limit 和 skip

`List<``Person``> persons = new ArrayList();`

`for (int i = 1; i <= 5; i++) {`

`Person person = new Person(i, "name" + i);`

`persons.add(person);`

`}`

`List<``Person``> personList2 = persons.stream().limit(2).sorted((p1, p2) -> p1.getName().compareTo(p2.getName())).collect(Collectors.toList());`

`System.out.println(personList2);`

结果会简单很多：

`name2`

`name1`

`[stream.StreamDW$Person@6ce253f1, stream.StreamDW$Person@53d8d10a]`

当然，这种优化是有 business logic 上的局限性的：即不要求排序后再取值。

**min/max/distinct**

min 和 max 的功能也可以通过对 Stream 元素先排序，再 findFirst 来实现，但前者的性能会更好，为 O(n)，而 sorted 的成本是 O(n log n)。同时它们作为特殊的 reduce 方法被独立出来也是因为求最大最小值是很常见的操作。

##### 找出最长一行的长度

`BufferedReader br = new BufferedReader(new FileReader("c:\\SUService.log"));`

`int longest = br.lines().`

`mapToInt(String::length).`

`max().`

`getAsInt();`

`br.close();`

`System.out.println(longest);`


下面的例子则使用 distinct 来找出不重复的单词。

#####  找出全文的单词，转小写，并排序

`List<``String``> words = br.lines().`

`flatMap(line -> Stream.of(line.split(" "))).`

`filter(word -> word.length() > 0).`

`map(String::toLowerCase).`

`distinct().`

`sorted().`

`collect(Collectors.toList());`

`br.close();`

`System.out.println(words);`


**Match**

Stream 有三个 match 方法，从语义上说：

*   allMatch：Stream 中全部元素符合传入的 predicate，返回 true
*   anyMatch：Stream 中只要有一个元素符合传入的 predicate，返回 true
*   noneMatch：Stream 中没有一个元素符合传入的 predicate，返回 true

它们都不是要遍历全部元素才能返回结果。例如 allMatch 只要一个元素不满足条件，就 skip 剩下的所有元素，返回 false。对清单 13 中的 Person 类稍做修改，加入一个 age 属性和 getAge 方法。

#####  使用 Match

`List<``Person``> persons = new ArrayList();`

`persons.add(new Person(1, "name" + 1, 10));`

`persons.add(new Person(2, "name" + 2, 21));`

`persons.add(new Person(3, "name" + 3, 34));`

`persons.add(new Person(4, "name" + 4, 6));`

`persons.add(new Person(5, "name" + 5, 55));`

`boolean isAllAdult = persons.stream().`

`allMatch(p -> p.getAge() > 18);`

`System.out.println("All are adult? " + isAllAdult);`

`boolean isThereAnyChild = persons.stream().`

`anyMatch(p -> p.getAge() < 12);`

`System.out.println("Any child? " + isThereAnyChild);`


输出结果：

`All are adult? false`

`Any child? true`


### 进阶：自己生成流

**Stream.generate**

通过实现 Supplier 接口，你可以自己来控制流的生成。这种情形通常用于随机数、常量的 Stream，或者需要前后元素间维持着某种状态信息的 Stream。把 Supplier 实例传递给 Stream.generate() 生成的 Stream，默认是串行（相对 parallel 而言）但无序的（相对 ordered 而言）。由于它是无限的，在管道中，必须利用 limit 之类的操作限制 Stream 大小。

##### 生成 10 个随机整数

`Random seed = new Random();`

`Supplier<``Integer``> random = seed::nextInt;`

`Stream.generate(random).limit(10).forEach(System.out::println);`

`//Another way`

`IntStream.generate(() -> (int) (System.nanoTime() % 100)).`

`limit(10).forEach(System.out::println);`


Stream.generate() 还接受自己实现的 Supplier。例如在构造海量测试数据的时候，用某种自动的规则给每一个变量赋值；或者依据公式计算 Stream 的每个元素值。这些都是维持状态信息的情形。

#####  自实现 Supplier

`Stream.generate(new PersonSupplier()).`

`limit(10).`

`forEach(p -> System.out.println(p.getName() + ", " + p.getAge()));`

`private class PersonSupplier implements Supplier<``Person``> {`

`private int index = 0;`

`private Random random = new Random();`

`@Override`

`public Person get() {`

`return new Person(index++, "StormTestUser" + index, random.nextInt(100));`

`}`

`}`


输出结果：

`StormTestUser1, 9`

`StormTestUser2, 12`

`StormTestUser3, 88`

`StormTestUser4, 51`

`StormTestUser5, 22`

`StormTestUser6, 28`

`StormTestUser7, 81`

`StormTestUser8, 51`

`StormTestUser9, 4`

`StormTestUser10, 76`

**Stream.iterate**

iterate 跟 reduce 操作很像，接受一个种子值，和一个 UnaryOperator（例如 f）。然后种子值成为 Stream 的第一个元素，f(seed) 为第二个，f(f(seed)) 第三个，以此类推。

**生成一个等差数列**

`Stream.iterate(0, n -> n + 3).limit(10). forEach(x -> System.out.print(x + " "));.`

输出结果：

`0 3 6 9 12 15 18 21 24 27`

与 Stream.generate 相仿，在 iterate 时候管道必须有 limit 这样的操作来限制 Stream 大小。

### 进阶：用 Collectors 来进行 reduction 操作

java.util.stream.Collectors 类的主要作用就是辅助进行各类有用的 reduction 操作，例如转变输出为 Collection，把 Stream 元素进行归组。

**groupingBy/partitioningBy**

##### 按照年龄归组

`Map<``Integer``, List<Person>> personGroups = Stream.generate(new PersonSupplier()).`

`limit(100).`

`collect(Collectors.groupingBy(Person::getAge));`

`Iterator it = personGroups.entrySet().iterator();`

`while (it.hasNext()) {`

`Map.Entry<``Integer``, List<Person>> persons = (Map.Entry) it.next();`

`System.out.println("Age " + persons.getKey() + " = " + persons.getValue().size());`

`}`

上面的 code，首先生成 100 人的信息，然后按照年龄归组，相同年龄的人放到同一个 list 中，可以看到如下的输出：

`Age 0 = 2`

`Age 1 = 2`

`Age 5 = 2`

`Age 8 = 1`

`Age 9 = 1`

`Age 11 = 2`

`……`
 

#####  按照未成年人和成年人归组
 

`Map<``Boolean``, List<Person>> children = Stream.generate(new PersonSupplier()).`

`limit(100).`

`collect(Collectors.partitioningBy(p -> p.getAge() < 18));`

`System.out.println("Children number: " + children.get(true).size());`

`System.out.println("Adult number: " + children.get(false).size());`
 

输出结果：

`Children number: 23`

`Adult number: 77`

在使用条件“年龄小于 18”进行分组后可以看到，不到 18 岁的未成年人是一组，成年人是另外一组。partitioningBy 其实是一种特殊的 groupingBy，它依照条件测试的是否两种结果来构造返回的数据结构，get(true) 和 get(false) 能即为全部的元素对象。


总之，Stream 的特性可以归纳为：

*   不是数据结构

*   它没有内部存储，它只是用操作管道从 source（数据结构、数组、generator function、IO channel）抓取数据。
*   它也绝不修改自己所封装的底层数据结构的数据。例如 Stream 的 filter 操作会产生一个不包含被过滤元素的新 Stream，而不是从 source 删除那些元素。

*   所有 Stream 的操作必须以 lambda 表达式为参数
*   不支持索引访问

*   你可以请求第一个元素，但无法请求第二个，第三个，或最后一个。不过请参阅下一项。

*   很容易生成数组或者 List
*   惰性化

*   很多 Stream 操作是向后延迟的，一直到它弄清楚了最后需要多少数据才会开始。
*   Intermediate 操作永远是惰性化的。

*   并行能力

*   当一个 Stream 是并行化的，就不需要再写多线程代码，所有对它的操作会自动并行进行的。

*   可以是无限的
    *   集合有固定大小，Stream 则不必。limit(n) 和 findFirst() 这类的 short-circuiting 操作可以对无限的 Stream 进行运算并很快完成。

参考：[https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/)