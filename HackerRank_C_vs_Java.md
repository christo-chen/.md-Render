# HackerRank 错题：C vs Java 对比

---

## 题 1：Compare the Triplets

比较两个数组对应元素，统计各自赢的次数。

### C 写法
```c
int* compareTriplets(int a_count, int* a, int b_count, int* b, int* result_count) {
    *result_count = 2;
    int* result = malloc(2 * sizeof(int));
    result[0] = 0;
    result[1] = 0;
    for (int i = 0; i < 3; i++) {
        if (a[i] > b[i]) result[0]++;
        else if (a[i] < b[i]) result[1]++;
    }
    return result;
}
```
**C 的麻烦之处：**
- 必须用 `malloc` 手动分配返回数组
- 必须通过指针 `result_count` 告诉调用者数组长度
- 忘记分配内存就会 segfault

### Java 写法
```java
static List<Integer> compareTriplets(List<Integer> a, List<Integer> b) {
    int alice = 0, bob = 0;
    for (int i = 0; i < 3; i++) {
        if (a.get(i) > b.get(i)) alice++;
        else if (a.get(i) < b.get(i)) bob++;
    }
    return List.of(alice, bob);
}
```
**Java 优势：** 直接 `return List.of(alice, bob)`，不需要手动管理内存。

---

## 题 2：Diagonal Difference

求矩阵主对角线和副对角线元素之和的差的绝对值。

### C 写法
```c
int diagonalDifference(int arr_rows, int arr_columns, int** arr) {
    int primarySum = 0, secondarySum = 0;
    for (int i = 0; i < arr_rows; i++) {
        primarySum += arr[i][i];                    // 主对角线: [0][0], [1][1], [2][2]
        secondarySum += arr[i][arr_rows - 1 - i];   // 副对角线: [0][2], [1][1], [2][0]
    }
    return abs(primarySum - secondarySum);
}
```
**C 的麻烦之处：**
- 二维数组是 `int**`（指针的指针），概念复杂
- 必须额外传 `arr_rows` 和 `arr_columns` 参数

### Java 写法
```java
static int diagonalDifference(List<List<Integer>> arr) {
    int n = arr.size();
    int primarySum = 0, secondarySum = 0;
    for (int i = 0; i < n; i++) {
        primarySum += arr.get(i).get(i);
        secondarySum += arr.get(i).get(n - 1 - i);
    }
    return Math.abs(primarySum - secondarySum);
}
```
**Java 优势：** `arr.size()` 直接拿长度，`Math.abs()` 求绝对值，不需要额外参数。

**核心知识点：** 副对角线索引 = `[i][n - 1 - i]`，记住这个公式。

---

## 题 3：Mini-Max Sum

5 个数中取 4 个求和，找最小和与最大和。

### C 写法（你的代码 + 修复溢出）
```c
void miniMaxSum(int arr_count, int* arr) {
    long sum = 0;          // ← 关键：用 long 而非 int
    int min = arr[0];
    int max = arr[0];
    for (int i = 0; i < arr_count; i++) {
        sum += arr[i];
        if (arr[i] > max) max = arr[i];
        if (arr[i] < min) min = arr[i];
    }
    printf("%ld %ld", sum - max, sum - min);  // ← %ld 输出 long
}
```
**C 的坑：**
- `int` 最大约 21 亿，5 个大数相加会溢出
- 必须手动选择 `long` 类型
- 格式化输出要用 `%ld` 而不是 `%d`

### Java 写法
```java
static void miniMaxSum(List<Integer> arr) {
    long sum = 0;
    int min = Collections.min(arr);
    int max = Collections.max(arr);
    for (int num : arr) {
        sum += num;
    }
    System.out.println((sum - max) + " " + (sum - min));
}
```
**Java 优势：**
- `Collections.min()` / `Collections.max()` 一行搞定
- `long` 的使用和 C 一样，但不需要 `%ld` 格式符
- for-each 循环更简洁

**核心教训：** 看到"求和"就用 `long`，养成习惯。

---

## 题 4：Time Conversion

12 小时制 → 24 小时制（如 "07:05:45PM" → "19:05:45"）

### C 写法
```c
char* timeConversion(char* s) {
    char* result = malloc(9 * sizeof(char));
    int hh, mm, ss;
    sscanf(s, "%d:%d:%d", &hh, &mm, &ss);
    char period = s[8];  // 'A' 或 'P'

    if (period == 'A') {
        if (hh == 12) hh = 0;       // 12:xx:xxAM → 00:xx:xx
    } else {
        if (hh != 12) hh += 12;     // 1-11 PM → 13-23
    }                                // 12:xx:xxPM → 12:xx:xx（不变）

    sprintf(result, "%02d:%02d:%02d", hh, mm, ss);
    return result;
}
```
**C 的麻烦之处：**
- 手动 `malloc` 分配返回字符串
- 用 `sscanf` 解析，需要记住格式符
- 用 `sprintf` 格式化输出，`%02d` 补零

### Java 写法
```java
static String timeConversion(String s) {
    String period = s.substring(8);         // "AM" 或 "PM"
    int hh = Integer.parseInt(s.substring(0, 2));
    String rest = s.substring(2, 8);        // ":05:45"

    if (period.equals("AM")) {
        hh = (hh == 12) ? 0 : hh;
    } else {
        hh = (hh == 12) ? 12 : hh + 12;
    }

    return String.format("%02d%s", hh, rest);
}
```
**Java 优势：**
- `substring()` 直接截取
- `String.format("%02d", hh)` 自动补零
- 不需要手动管理内存

**核心知识点（两个边界必须记住）：**
- 12:xx AM → 00:xx（午夜）
- 12:xx PM → 12:xx（中午，不变）

---

## 总结对比

| 方面 | C | Java |
|------|---|------|
| 返回数组 | malloc + 指针传长度 | 直接 return List |
| 字符串处理 | sscanf/sprintf/手动索引 | substring/split/format |
| 内存管理 | 手动 malloc/free | 自动垃圾回收 |
| 求最大最小 | 手动遍历 | Collections.min/max |
| 溢出处理 | 同样需要 long | 同样需要 long |
| 代码行数 | 多 30-50% | 更少 |

**结论：Stripe 笔试用 Java，省下来的时间用于处理业务逻辑和边界情况。**
