# Stream

## 1.基本使用

```java
List<String> list = Arrays.asList("a1", "a2", "b1", "c2", "c1");
list.stream()
   	    // 中间操作
        .filter(s -> s.startsWith("c"))
        .map(String::toUpperCase)
        .sorted()
  			// 终端操作
        .forEach(System.out::println);
```

- 通过of直接生成Stream流

```java
Stream.of("a1", 2, "c", 'd').forEach(System.out::println);
```

## 2.特殊(原始数据)类型的流

1. IntStream
2. LongStream
3. DoubleStream

```java
IntStream.range(1,4).forEach(System.out::println);

// 1
// 2
// 3
```

- 原始类型流使用特有的函数式接口，例如用IntFunction代替Funciton，IntPredicate代替Predicate
- 原始类型流支持额外的终端聚合操作,sum()、average()等

```java
Arrays.stream(new int[]{1, 2, 3})
        .map(n -> 2 * n)
        .average()
        .ifPresent(System.out::println);
```

## 3.将原始类型流转换为常规流

```java
// 转int
Stream.of("a1", "a2", "a3")
        .map(s -> s.substring(1))
        .mapToInt(Integer::parseInt)
        .max()
        .ifPresent(System.out::println);

// 转obj
IntStream.of(1, 2, 3)
        .mapToObj(i -> "a" + i)
        .forEach(System.out::println);

// 连续转换
Stream.of(1.0, 2.0, 3.0)
    .mapToInt(Double::intValue) // double 类型转 int
    .mapToObj(i -> "a" + i) // 对值拼接前缀 a
    .forEach(System.out::println); // for 循环打印
```

## 4.Stream 流的处理顺序

- 仅当存在终端操作时，中间操作操作才会被执行

```java
// 没有终端操作 无输出
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return true;
    });
```

- 输出按照垂直链条

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return true;
    })
    .forEach(s -> System.out.println("forEach: " + s));

filter:  d2
forEach: d2
filter:  a2
forEach: a2
filter:  b1
forEach: b1
filter:  b3
forEach: b3
filter:  c
forEach: c
```

- 垂直链条调用可以减少对每个元素的实际操作次数

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase(); // 转大写
    })
    .anyMatch(s -> {
        System.out.println("anyMatch: " + s);
        return s.startsWith("A"); // 过滤出以 A 为前缀的元素
    });

// map:      d2
// anyMatch: D2
// map:      a2
// anyMatch: A2
```

- 将`filter`移动到链头的最开始，就可以大大减少实际的执行次数

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s)
        return s.startsWith("a"); // 过滤出以 a 为前缀的元素
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase(); // 转大写
    })
    .forEach(s -> System.out.println("forEach: " + s)); // for 循环输出
```

- ==判断流操作是否有状态的判断标准，就是看是否需要知道先前的数据历史。前后数据是否有依赖关系来判断==
- `sorted`是水平执行的,会先对集合中的元素组合调用八次

```java
Stream.of("d2", "a2", "b1", "b3", "c")
    
    .sorted((s1, s2) -> {
        System.out.printf("sort: %s; %s\n", s1, s2);
        return s1.compareTo(s2); // 排序
    })
    .filter(s -> {
        System.out.println("filter: " + s);
        return s.startsWith("a"); // 过滤出以 a 为前缀的元素
    })
    .map(s -> {
        System.out.println("map: " + s);
        return s.toUpperCase(); // 转大写
    })
    .forEach(s -> System.out.println("forEach: " + s)); // for 循环输出
    
sort:    a2; d2
sort:    b1; a2
sort:    b1; d2
sort:    b1; a2
sort:    b3; b1
sort:    b3; d2
sort:    c; b3
sort:    c; d2
filter:  a2
map:     a2
forEach: A2
filter:  b1
filter:  b3
filter:  c
filter:  d2
```

