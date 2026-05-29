# Java 语法速查参考 (Java 21)

## 1. 基础

### 1.1 程序结构
```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

### 1.2 基本数据类型
```java
byte b = 127;              // 1 字节  -128 ~ 127
short s = 32000;           // 2 字节
int i = 100;               // 4 字节  最常用
long l = 100L;             // 8 字节  字面量要加 L
float f = 3.14f;           // 4 字节  字面量要加 f
double d = 3.14;           // 8 字节  浮点默认
char c = 'A';              // 2 字节  Unicode
boolean flag = true;

// 字面量写法
int hex = 0xFF;            // 十六进制
int bin = 0b1010;          // 二进制
int big = 1_000_000;       // 下划线分隔
```

### 1.3 变量与常量
```java
int x = 10;
final int MAX = 100;       // 常量
var name = "Lilith";       // Java 10+ 类型推断，仅局部变量
```

### 1.4 类型转换
```java
int i = (int) 3.14;        // 强制转换
double d = 10;             // 自动提升
String s = String.valueOf(123);
int n = Integer.parseInt("123");
double dd = Double.parseDouble("3.14");
```

---

## 2. 运算符

```java
// 算术: + - * / %     注意整数除法: 7/2 = 3
// 比较: == != > < >= <=
// 逻辑: && || !       短路求值
// 位运算: & | ^ ~ << >> >>>
// 三元: int max = a > b ? a : b;
// 复合赋值: += -= *= /= %= &= |= ^= <<= >>=

// 引用比较 vs 内容比较
String a = "hi", b = "hi";
a == b              // 引用比较
a.equals(b)         // 内容比较 — 字符串必须用这个
```

---

## 3. 字符串

```java
String s = "hello";
String t = """
        多行
        文本块  (Java 15+)
        """;

s.length();
s.charAt(0);
s.substring(1, 3);
s.indexOf("ll");
s.contains("ell");
s.startsWith("he");
s.replace("l", "L");
s.split(",");
s.trim();  s.strip();        // strip 支持 Unicode 空白
s.toUpperCase();
s.isEmpty();  s.isBlank();
String.join(",", "a", "b");
String.format("x=%d, y=%.2f", 1, 3.14);

// StringBuilder (可变,性能好)
StringBuilder sb = new StringBuilder();
sb.append("a").append("b");
sb.insert(0, "x");
sb.reverse();
String result = sb.toString();
```

---

## 4. 控制流

### 4.1 if / else
```java
if (x > 0) { ... }
else if (x == 0) { ... }
else { ... }
```

### 4.2 switch (传统 + 表达式)
```java
// 传统语句
switch (day) {
    case 1: case 2:
        System.out.println("weekday");
        break;
    default:
        break;
}

// switch 表达式 (Java 14+)
String name = switch (day) {
    case 1, 2, 3, 4, 5 -> "weekday";
    case 6, 7          -> "weekend";
    default            -> throw new IllegalArgumentException();
};

// 带 yield 的代码块
int val = switch (x) {
    case 1 -> 10;
    case 2 -> {
        int y = compute();
        yield y * 2;
    }
    default -> 0;
};

// 模式匹配 switch (Java 21)
String desc = switch (obj) {
    case Integer i -> "int: " + i;
    case String s when s.length() > 5 -> "long string";
    case String s -> "string";
    case null -> "null";
    default -> "other";
};
```

### 4.3 循环
```java
// for
for (int i = 0; i < 10; i++) { ... }

// 增强 for
for (int x : arr) { ... }
for (var entry : map.entrySet()) { ... }

// while / do-while
while (cond) { ... }
do { ... } while (cond);

// break / continue / 带标签
outer:
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        if (...) break outer;
    }
}
```

---

## 5. 数组

```java
int[] a = new int[5];                 // 默认 0
int[] b = {1, 2, 3};
int[][] mat = new int[3][4];
int[][] jagged = {{1,2}, {3,4,5}};

a.length;                              // 长度属性
Arrays.sort(a);
Arrays.fill(a, 0);
int[] copy = Arrays.copyOf(a, 10);
int[] sub  = Arrays.copyOfRange(a, 1, 4);
Arrays.equals(a, b);
Arrays.toString(a);
Arrays.stream(a).sum();
```

---

## 6. 方法

```java
public static int add(int a, int b) { return a + b; }

// 可变参数
public static int sum(int... nums) {
    int s = 0;
    for (int n : nums) s += n;
    return s;
}

// 方法重载: 同名不同参数列表
int max(int a, int b) { ... }
double max(double a, double b) { ... }
```

---

## 7. 类与对象

```java
public class Person {
    // 字段
    private String name;
    private int age;
    private static int count = 0;       // 静态字段

    // 构造器
    public Person() { this("匿名", 0); }
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
        count++;
    }

    // 实例方法
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    // 静态方法
    public static int getCount() { return count; }

    // 初始化块
    { /* 实例初始化块 */ }
    static { /* 静态初始化块 */ }

    @Override
    public String toString() { return name + "(" + age + ")"; }

    @Override
    public boolean equals(Object o) { ... }

    @Override
    public int hashCode() { return Objects.hash(name, age); }
}

// 使用
Person p = new Person("L", 20);
```

### 7.1 访问修饰符
| 修饰符 | 同类 | 同包 | 子类 | 任何处 |
|--------|:---:|:---:|:---:|:---:|
| public | ✓ | ✓ | ✓ | ✓ |
| protected | ✓ | ✓ | ✓ | ✗ |
| (default) | ✓ | ✓ | ✗ | ✗ |
| private | ✓ | ✗ | ✗ | ✗ |

---

## 8. 继承与多态

```java
public class Animal {
    protected String name;
    public Animal(String name) { this.name = name; }
    public void speak() { System.out.println("..."); }
}

public class Dog extends Animal {
    public Dog(String name) { super(name); }

    @Override
    public void speak() { System.out.println("Woof"); }

    public void fetch() { ... }
}

// 多态
Animal a = new Dog("旺财");
a.speak();                      // Woof

// 类型判断 + 强转
if (a instanceof Dog d) {       // Java 16+ 模式匹配
    d.fetch();
}

// final
final class NoExtend { }        // 不可继承
final void noOverride() { }     // 不可重写
```

---

## 9. 抽象类与接口

```java
// 抽象类
public abstract class Shape {
    public abstract double area();           // 抽象方法
    public void print() { System.out.println(area()); }
}

// 接口
public interface Comparable<T> {
    int compareTo(T other);

    default boolean lessThan(T other) {       // 默认方法 (Java 8+)
        return compareTo(other) < 0;
    }

    static Comparable<Integer> natural() {    // 静态方法 (Java 8+)
        return Integer::compare;
    }

    private int helper() { return 0; }        // 私有方法 (Java 9+)
}

// 实现
public class Circle extends Shape implements Comparable<Circle> {
    public double area() { return ...; }
    public int compareTo(Circle o) { return ...; }
}

// 函数式接口 (只有一个抽象方法,可用 lambda)
@FunctionalInterface
interface Calculator { int apply(int a, int b); }

// sealed 接口/类 (Java 17+) — 限制谁能继承
public sealed interface Shape2 permits Circle2, Square2 { }
public final class Circle2 implements Shape2 { }
public final class Square2 implements Shape2 { }
```

---

## 10. 枚举 (enum)

```java
public enum Day {
    MON, TUE, WED, THU, FRI, SAT, SUN
}

// 带字段和方法
public enum Color {
    RED(255, 0, 0),
    GREEN(0, 255, 0);

    private final int r, g, b;
    Color(int r, int g, int b) { this.r = r; this.g = g; this.b = b; }
    public int getR() { return r; }
}

// 使用
Day d = Day.MON;
switch (d) { case MON -> ...; }
Day[] all = Day.values();
Day.valueOf("MON");
d.ordinal();          // 序号
d.name();             // 名字
```

---

## 11. record (Java 16+)

不可变数据类的简写,自动生成构造器/getter/equals/hashCode/toString。

```java
public record Point(int x, int y) { }

// 等价于一个不可变类,带 x()/y()/equals/hashCode/toString
Point p = new Point(1, 2);
p.x();  p.y();

// 紧凑构造器做校验
public record Range(int lo, int hi) {
    public Range {
        if (lo > hi) throw new IllegalArgumentException();
    }
}

// 可以实现接口、加方法、加静态字段
public record Vec(double x, double y) implements Comparable<Vec> {
    public double len() { return Math.hypot(x, y); }
    public int compareTo(Vec o) { return Double.compare(len(), o.len()); }
}
```

---

## 12. 内部类 / 匿名类 / Lambda

```java
// 静态内部类
class Outer {
    static class Inner { }
}

// 非静态内部类
class Outer2 {
    class Inner2 { }
}
Outer2.Inner2 i = new Outer2().new Inner2();

// 局部类 (方法内)
void m() { class Local { } }

// 匿名类
Runnable r = new Runnable() {
    @Override public void run() { ... }
};

// Lambda (替代单方法匿名类)
Runnable r2 = () -> System.out.println("run");
Comparator<Integer> c = (a, b) -> a - b;
Function<Integer, Integer> sq = x -> x * x;

// 方法引用
list.forEach(System.out::println);
list.sort(Integer::compare);
list.stream().map(String::toUpperCase);
Supplier<List<String>> s = ArrayList::new;
```

### 常用函数式接口
```java
Function<T, R>        R apply(T t)
BiFunction<T, U, R>   R apply(T t, U u)
Consumer<T>           void accept(T t)
Supplier<T>           T get()
Predicate<T>          boolean test(T t)
UnaryOperator<T>      T apply(T t)
BinaryOperator<T>     T apply(T a, T b)
```

---

## 13. 泛型

```java
// 泛型类
public class Box<T> {
    private T value;
    public T get() { return value; }
    public void set(T value) { this.value = value; }
}

// 多类型参数
public class Pair<K, V> { ... }

// 泛型方法
public static <T> T first(List<T> list) { return list.get(0); }

// 边界
public class NumBox<T extends Number> { }
public static <T extends Comparable<T>> T max(T a, T b) { ... }

// 通配符
List<?>            // 未知类型
List<? extends Number>   // 上界 — 只读
List<? super Integer>    // 下界 — 只写

// 类型擦除: 运行时无泛型信息
// 不能 new T[],  new ArrayList<T>() 可以
```

---

## 14. 异常处理

```java
try {
    risky();
} catch (IOException | SQLException e) {     // multi-catch
    e.printStackTrace();
} catch (Exception e) {
    throw new RuntimeException(e);
} finally {
    cleanup();
}

// try-with-resources (自动关闭)
try (BufferedReader br = new BufferedReader(new FileReader("f"))) {
    return br.readLine();
}

// 自定义异常
public class MyException extends RuntimeException {
    public MyException(String msg) { super(msg); }
    public MyException(String msg, Throwable cause) { super(msg, cause); }
}

// 抛出
throw new IllegalArgumentException("bad");

// 检查异常 vs 非检查异常:
// Exception 的子类(除 RuntimeException)必须 throws 或 catch
public void m() throws IOException { ... }
```

---

## 15. 集合框架 (Collections)

### 15.1 主要接口
```
Collection
├── List       有序可重复
├── Set        无序不重复
└── Queue/Deque

Map           键值对
```

### 15.2 List
```java
List<Integer> list = new ArrayList<>();    // 数组实现,随机访问快
List<Integer> ll   = new LinkedList<>();   // 链表实现,插入删除快
List<Integer> imm  = List.of(1, 2, 3);     // 不可变 (Java 9+)

list.add(1); list.add(0, 99);
list.get(0); list.set(0, 100);
list.remove(0);            // 按索引
list.remove(Integer.valueOf(99));   // 按值
list.contains(1);
list.indexOf(1);
list.size();
list.isEmpty();
list.subList(0, 2);
Collections.sort(list);
Collections.reverse(list);
```

### 15.3 Set
```java
Set<String> s   = new HashSet<>();         // 哈希,无序,O(1)
Set<String> ls  = new LinkedHashSet<>();   // 保持插入顺序
Set<String> ts  = new TreeSet<>();         // 红黑树,有序,O(log n)

s.add("a"); s.remove("a"); s.contains("a");
```

### 15.4 Map
```java
Map<String, Integer> m   = new HashMap<>();
Map<String, Integer> lm  = new LinkedHashMap<>();   // 保持顺序
Map<String, Integer> tm  = new TreeMap<>();         // 有序

m.put("a", 1);
m.get("a");
m.getOrDefault("a", 0);
m.containsKey("a");
m.remove("a");
m.size();
m.keySet();  m.values();  m.entrySet();

// 遍历
for (var e : m.entrySet()) {
    System.out.println(e.getKey() + "=" + e.getValue());
}

// 实用方法
m.putIfAbsent("a", 1);
m.merge("a", 1, Integer::sum);                       // 累加器常用
m.computeIfAbsent("a", k -> new ArrayList<>()).add(1);
m.forEach((k, v) -> System.out.println(k + v));
```

### 15.5 Queue / Deque / Stack
```java
Queue<Integer> q = new LinkedList<>();
q.offer(1); q.poll(); q.peek();

Deque<Integer> dq = new ArrayDeque<>();
dq.push(1); dq.pop(); dq.peek();                    // 当栈用 (优于 Stack 类)
dq.offerFirst(1); dq.offerLast(2);
dq.pollFirst();    dq.pollLast();

PriorityQueue<Integer> pq = new PriorityQueue<>();           // 小顶堆
PriorityQueue<Integer> pq2 = new PriorityQueue<>(Comparator.reverseOrder()); // 大顶堆
pq.offer(3); pq.poll();
```

### 15.6 排序
```java
List<Integer> list = ...;
list.sort(null);                                    // 自然序
list.sort(Comparator.reverseOrder());
list.sort((a, b) -> a - b);

// 多关键字
people.sort(Comparator
    .comparingInt(Person::getAge)
    .thenComparing(Person::getName)
    .reversed());
```

---

## 16. Stream API (Java 8+)

```java
// 创建
Stream<Integer> s1 = Stream.of(1, 2, 3);
Stream<Integer> s2 = list.stream();
IntStream s3 = IntStream.range(0, 10);      // [0, 10)
IntStream s4 = IntStream.rangeClosed(1, 10);

// 中间操作 (惰性,返回 Stream)
.filter(x -> x > 0)
.map(x -> x * 2)
.flatMap(list -> list.stream())
.distinct()
.sorted()
.sorted(Comparator.reverseOrder())
.limit(10)
.skip(5)
.peek(System.out::println)       // 调试用

// 终止操作 (触发执行)
.forEach(System.out::println)
.count()
.sum()           // 只在 IntStream/LongStream/DoubleStream
.min(Comparator.naturalOrder())
.max(...)
.reduce(0, Integer::sum)
.anyMatch(x -> x > 0)
.allMatch(...)
.noneMatch(...)
.findFirst()
.findAny()
.toList()                        // Java 16+
.collect(Collectors.toList())
.collect(Collectors.toSet())
.collect(Collectors.toMap(Person::getId, p -> p))
.collect(Collectors.groupingBy(Person::getAge))
.collect(Collectors.partitioningBy(p -> p.age > 18))
.collect(Collectors.joining(", ", "[", "]"))
.collect(Collectors.counting())

// 常见示例
int sum = nums.stream().mapToInt(Integer::intValue).sum();
List<String> names = people.stream()
    .filter(p -> p.age > 18)
    .map(Person::getName)
    .sorted()
    .toList();

Map<Integer, List<Person>> byAge = people.stream()
    .collect(Collectors.groupingBy(Person::getAge));
```

---

## 17. Optional (Java 8+)

避免 NullPointerException 的容器。

```java
Optional<String> o = Optional.of("hi");
Optional<String> e = Optional.empty();
Optional<String> n = Optional.ofNullable(maybeNull);

o.isPresent();
o.isEmpty();
o.get();                          // 不推荐,空时抛异常
o.orElse("default");
o.orElseGet(() -> compute());
o.orElseThrow(() -> new MyException());
o.ifPresent(System.out::println);
o.ifPresentOrElse(System.out::println, () -> System.out.println("empty"));
o.map(String::toUpperCase);
o.filter(s -> s.length() > 0);
```

---

## 18. I/O

### 18.1 控制台
```java
Scanner sc = new Scanner(System.in);
int n = sc.nextInt();
String s = sc.nextLine();
sc.close();

System.out.print("x");
System.out.println("x");
System.out.printf("%d %.2f%n", 1, 3.14);
```

### 18.2 文件读写 (现代 NIO)
```java
import java.nio.file.*;

Path p = Path.of("data.txt");

// 一次性读写
String text = Files.readString(p);
List<String> lines = Files.readAllLines(p);
byte[] bytes = Files.readAllBytes(p);

Files.writeString(p, "hello");
Files.write(p, lines);

// 流式
try (BufferedReader br = Files.newBufferedReader(p)) {
    br.lines().forEach(System.out::println);
}

try (BufferedWriter bw = Files.newBufferedWriter(p)) {
    bw.write("line"); bw.newLine();
}

// 文件操作
Files.exists(p);
Files.createDirectories(p.getParent());
Files.copy(src, dst);
Files.move(src, dst);
Files.delete(p);
Files.size(p);
Files.list(dir);          // Stream<Path>
Files.walk(dir);
```

### 18.3 传统 IO (旧但仍常见)
```java
try (BufferedReader br = new BufferedReader(new FileReader("f"))) {
    String line;
    while ((line = br.readLine()) != null) { ... }
}
```

---

## 19. 日期时间 (java.time, Java 8+)

```java
LocalDate     d  = LocalDate.now();           // 2026-05-29
LocalTime     t  = LocalTime.now();
LocalDateTime dt = LocalDateTime.now();
Instant       i  = Instant.now();             // UTC 时间戳
ZonedDateTime z  = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));

LocalDate d2 = LocalDate.of(2026, 5, 29);
LocalDate d3 = LocalDate.parse("2026-05-29");

d.plusDays(7); d.minusMonths(1);
d.getYear(); d.getMonthValue(); d.getDayOfWeek();
d.isAfter(d2); d.isBefore(d2);

Duration dur = Duration.between(t1, t2);
Period   per = Period.between(d1, d2);

DateTimeFormatter f = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
dt.format(f);
LocalDateTime.parse("2026-05-29 12:00:00", f);
```

---

## 20. 并发基础

```java
// 创建线程
Thread t = new Thread(() -> System.out.println("run"));
t.start(); t.join();

// Runnable / Callable
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.submit(() -> doWork());
Future<Integer> f = pool.submit(() -> 42);
int result = f.get();
pool.shutdown();

// CompletableFuture
CompletableFuture<String> cf = CompletableFuture
    .supplyAsync(() -> "data")
    .thenApply(String::toUpperCase)
    .thenAccept(System.out::println);

// 同步
synchronized (lock) { ... }
public synchronized void method() { ... }

ReentrantLock lock = new ReentrantLock();
lock.lock();
try { ... } finally { lock.unlock(); }

// 原子类
AtomicInteger ai = new AtomicInteger(0);
ai.incrementAndGet();
ai.compareAndSet(0, 1);

// 并发容器
ConcurrentHashMap<K, V> m = new ConcurrentHashMap<>();
CopyOnWriteArrayList<T> l = new CopyOnWriteArrayList<>();
BlockingQueue<T> q = new LinkedBlockingQueue<>();

// 虚拟线程 (Java 21)
Thread.startVirtualThread(() -> ...);
try (var es = Executors.newVirtualThreadPerTaskExecutor()) {
    es.submit(...);
}
```

---

## 21. 注解 (Annotation)

```java
@Override
@Deprecated
@SuppressWarnings("unchecked")
@FunctionalInterface
@SafeVarargs

// 自定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnno {
    String value() default "";
    int times() default 1;
}

@MyAnno(value = "x", times = 3)
public void m() { }
```

---

## 22. 模块 (Java 9+, 较少用)

```java
// module-info.java
module com.example.app {
    requires java.sql;
    exports com.example.app.api;
}
```

---

## 23. 常用工具类速查

```java
Math.max/min/abs/pow/sqrt/random
Math.floor/ceil/round

Integer.parseInt/toString/toBinaryString/MAX_VALUE/MIN_VALUE
Integer.bitCount(n)              // popcount
Integer.numberOfLeadingZeros(n)
Long.parseLong

Objects.equals(a, b)             // 空安全
Objects.hash(a, b, c)
Objects.requireNonNull(x)

Arrays.sort / Arrays.binarySearch / Arrays.asList / Arrays.stream
Collections.sort / shuffle / reverse / max / min / emptyList / unmodifiableList

System.arraycopy(src, 0, dst, 0, len);
System.currentTimeMillis();
System.nanoTime();
```

---

## 24. 现代特性总览 (Java 9-21)

| 版本 | 特性 |
|------|------|
| 9 | 接口私有方法,模块 |
| 10 | `var` 局部变量类型推断 |
| 11 | `String.strip/isBlank/lines/repeat`,HttpClient |
| 14 | switch 表达式 |
| 15 | 文本块 `"""..."""` |
| 16 | record,`instanceof` 模式匹配 |
| 17 | sealed 类/接口,LTS |
| 19 | 虚拟线程 (预览) |
| 21 | switch 模式匹配,record 模式,虚拟线程正式版,LTS |

---

## 25. 编码风格速记

- 类名: `PascalCase`
- 方法/变量: `camelCase`
- 常量: `UPPER_SNAKE_CASE`
- 包名: `lowercase.dots`
- 优先用 `List` 接口声明: `List<T> l = new ArrayList<>()`
- 优先 `Deque` 而非过时的 `Stack`
- 字符串拼接循环用 `StringBuilder`
- 比较字符串永远用 `equals`
- 用 `Objects.equals` 处理可能为 null 的比较
- 资源用 try-with-resources
- 集合优先返回不可变 (`List.of`, `Map.copyOf`)
