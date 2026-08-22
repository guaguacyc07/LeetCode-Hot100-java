# 数组 Array 常用 API

> 数组是刷题最常用的结构：长度固定、按下标访问 O(1)。
>
> 本文覆盖创建、增删、查、改、排序、遍历、转换、复杂度、高频套路、常见坑。
>
> AI生成+人工优化

## 一、创建与初始化

```java
int[] a = new int[5];            // 长度5，默认全 0
int[] b = {1, 2, 3, 4, 5};       // 直接初始化
int[] c = new int[]{1, 2, 3};    // new + 初始化（可用于方法传参）

// 二维数组
int[][] g1 = new int[3][4];      // 3 行 4 列，默认全 0
int[][] g2 = {{1, 2}, {3, 4}};   // 直接初始化
int[][] g3 = new int[3][];       // 只定行，每行列长可不同（锯齿数组）

// 快速填充
int[] f = new int[10];
Arrays.fill(f, -1);              // 全部填 -1
Arrays.fill(f, 0, 5, 100);       // 只填 [0,5)，即下标 0~4
```

> 注意 `Arrays.fill(f, from, to, val)` 的 `[from, to)` 是**左闭右开**：含 from，不含 to。

## 二、增 / 删

数组长度固定，**不能直接 add/remove**。

> 刷题结论：需要增删时直接用 `ArrayList`（见 List 笔记），数组适合"长度确定、只按下标访问"的场景。

## 三、查

| 操作 | 写法 | 复杂度 |
| ---- | ---- | ------ |
| 按下标取值 | `a[i]` | O(1) |
| 数组长度 | `a.length`（属性，**没有括号**） | O(1) |
| 线性查找 | `for (int i = 0; i < a.length; i++)` | O(n) |
| 有序数组二分查找 | `Arrays.binarySearch(a, key)` | O(log n) |
| 求最大/最小 | 见下 | O(n) |

**查数组长度（一维 / 二维 / 锯齿数组）**

```java
// 一维数组
int[] a = {1, 2, 3};
int aLen = a.length;      // aLen = 3

// 二维数组
int[][] b = {{1, 2, 3}, {4, 5, 6}};
int bRow = b.length;      // bRow = 2（行数）
int bCol = b[0].length;   // bCol = 3（第 0 行的列数）

// 注意：Java 二维数组每行的列数可以不同（锯齿数组）
int[][] c = {{1, 2, 3}, {4}};
int cRow = c.length;      // cRow = 2
int cCol1 = c[0].length;  // cCol1 = 3
int cCol2 = c[1].length;  // cCol2 = 1
```

**二分查找（数组必须已有序）**

```java
int[] a = {1, 3, 5, 7, 9};
Arrays.binarySearch(a, 5);   // 2   —— 找到，返回下标
Arrays.binarySearch(a, 4);   // -3  —— 没找到，返回负数
Arrays.binarySearch(a, 1, 4, 7);  // 只在 [1,4) 下标范围内找，返回 3
```

> **找不到时的负数**：返回 `-(插入点) - 1`。插入点 = 第一个比 key 大的位置。
> key=4 应插在下标 2（在 3 和 5 之间），所以返回 `-(2)-1 = -3`。
> 想还原插入点：`int pos = -Arrays.binarySearch(a, 4) - 1;` → 2。

**求最大/最小值**

```java
int max = Arrays.stream(a).max().getAsInt();   // 最大值
int min = Arrays.stream(a).min().getAsInt();   // 最小值
// 或手动（省去 Stream 开销）：
int max = a[0];
for (int x : a) max = Math.max(max, x);
```

## 四、改

```java
int[] a = {1, 2, 3};
a[0] = 99;          // {99, 2, 3}
```

## 五、排序

```java
int[] a = {3, 1, 4, 1, 5, 9, 2};
Arrays.sort(a);                 // 升序
Arrays.sort(a, 1, 4);           // 只排 [1,4)，即下标 1~3

// 降序：int[] 必须先装箱成 Integer[]（包装类型）
Integer[] b = {3, 1, 4};
Arrays.sort(b, Collections.reverseOrder());
```

**自定义排序（二维数组按某一列）——推荐用 `Comparator`**

```java
int[][] p = {{3, 1}, {1, 2}, {2, 3}};

// 按第 0 列升序
Arrays.sort(p, Comparator.comparingInt(x -> x[0]));
// 按第 0 列升序，第 0 列相同再按第 1 列升序
Arrays.sort(p, Comparator.comparingInt((int[] x) -> x[0]).thenComparingInt(x -> x[1]));
```

> 也常见 `(x, y) -> x[0] - y[0]` 这种减法写法，更短但**大数会溢出**，用 `Comparator.comparingInt` 最安全。

## 六、遍历

```java
int[] a = {1, 2, 3, 4};

// 1. 下标 for：能拿到下标，也能修改元素
for (int i = 0; i < a.length; i++) { ... }

// 2. for-each：最简洁，但拿不到下标，且不能修改元素值（基本类型是副本）
for (int x : a) { ... }

// 3. Stream
Arrays.stream(a).forEach(x -> { ... });
```

## 七、转换

| 场景 | 写法 |
| ---- | ---- |
| 数组 → 字符串 | `Arrays.toString(a)` → `"[1, 2, 3]"` |
| 字符串 → 字符数组 | `s.toCharArray()` |
| 字符数组 → 字符串 | `new String(charArr)` |
| 字符串数组 → 字符串 | `String.join(",", strArr)` |
| 对象数组 → List | `Arrays.asList(arr)`（⚠️ 定长视图） |
| List → 数组 | `list.toArray(new Integer[0])` |
| 数组拷贝 | `Arrays.copyOf(a, len)` / `Arrays.copyOfRange(a, from, to)` / `a.clone()` |
| int[] → List\<Integer\> | `Arrays.stream(a).boxed().collect(Collectors.toList())` |

```java
int[] a = {1, 2, 3};
List<Integer> list = Arrays.stream(a).boxed().collect(Collectors.toList()); // 需 import Collectors
String s = Arrays.toString(a);              // "[1, 2, 3]"
char[] cs = s.toCharArray();                // 字符数组
String back = new String(cs);               // 还原字符串
int[] copy = Arrays.copyOf(a, a.length);    // 拷贝
int[] part = Arrays.copyOfRange(a, 1, 3);   // [1,3) → {2, 3}
```

> `Arrays.asList` 返回的 List **由原数组支撑**（改数组会反映到 List，反之亦然），且**不能 add/remove**。`int[]` 不能直接 `asList`（会被当成单个元素，见"常见坑"）。

## 八、复杂度对照

| 操作 | 复杂度 |
| ---- | ------ |
| 按下标访问 / 赋值 | O(1) |
| 线性查找 | O(n) |
| 二分查找（有序） | O(log n) |
| 排序 | O(n log n) |
| 插入 / 删除（需移动元素） | O(n) |

## 九、刷题可用

### 1. 二分查找（左闭右闭模板）

> 区间是 `[left, right]` 双闭；`mid` 用 `left + (right - left) / 2` 防止 `(left+right)` 溢出。

```java
int left = 0, right = a.length - 1;
while (left <= right) {
    int mid = left + (right - left) / 2;
    if (a[mid] == target) return mid;
    else if (a[mid] < target) left = mid + 1;
    else right = mid - 1;
}
return -1;  // 没找到
```

### 2. 双指针

```java
// 左右指针：数组有序时，找两数之和
int i = 0, j = a.length - 1;
while (i < j) {
    int sum = a[i] + a[j];
    if (sum == target) { ... }
    else if (sum < target) i++;
    else j--;
}

// 快慢指针：原地去重（前提：数组已有序）
int slow = 0;
for (int fast = 0; fast < a.length; fast++) {
    if (a[fast] != a[slow]) a[++slow] = a[fast];
}
// 去重后有效长度为 slow + 1
```

### 3. 前缀和（O(1) 求区间和）

```java
// pre[i] = a[0] + ... + a[i-1]，即前 i 个元素的和
int[] pre = new int[a.length + 1];
for (int i = 0; i < a.length; i++) pre[i + 1] = pre[i] + a[i];

// 区间 [l, r] 的和 = pre[r+1] - pre[l]
int sum = pre[r + 1] - pre[l];
```

### 4. 差分数组（区间整体加减）

> 差分数组 `diff` 记录"相邻差值"；对区间 `[l,r]` 统一加 `val` 时，只需改 `diff[l]` 和 `diff[r+1]` 两个点，最后做一次前缀和还原。常用于"多次区间加减、最后统一查询"。

```java
int[] diff = new int[a.length + 1];
diff[l] += val;        // 区间 [l, r] 整体加 val
diff[r + 1] -= val;

// 还原：对 diff 做前缀和，diff[i] 即最终 a[i] 的值
for (int i = 1; i < diff.length; i++) diff[i] += diff[i - 1];
```

### 5. 矩阵顺时针旋转 90°（转置 + 每行逆序）

```java
// n×n 方阵
for (int i = 0; i < n; i++) {
    for (int j = i + 1; j < n; j++) {          // 转置（对角线交换）
        int t = m[i][j]; m[i][j] = m[j][i]; m[j][i] = t;
    }
}
for (int[] row : m) {                           // 每行逆序
    int l = 0, r = row.length - 1;
    while (l < r) { int t = row[l]; row[l] = row[r]; row[r] = t; l++; r--; }
}
```

## 十、常见坑

1. **数组定长**：不能 `add`/`remove`，改用 `ArrayList`。
2. **`Arrays.asList` 返回定长视图**：`add`/`remove` 会抛 `UnsupportedOperationException`。
3. **`Arrays.asList(int[])` 是坑**：`int[]` 会被当成**一个元素**（得到 `List<int[]>`），正确做法是 `Arrays.stream(a).boxed()...`。
4. **`toArray` 泛型**：`list.toArray(new Integer[0])`，别用无参 `toArray()`（返回 `Object[]`），也别写 `new int[0]`（基本类型不匹配）。
5. **`==` 比的是引用**：比较内容用 `Arrays.equals(a, b)`；二维用 `Arrays.deepEquals`。
6. **越界**：下标必须满足 `0 <= i < a.length`，否则 `ArrayIndexOutOfBoundsException`。
7. **`binarySearch` 找不到返回负数**：返回 `-(插入点)-1`，要区分"找到下标 0"和"没找到"。
8. **`a.length` 是属性**：不是方法，不要写成 `a.length()`。
9. **排序减法溢出**：自定义排序优先用 `Comparator.comparingInt`，别用 `x - y`。
