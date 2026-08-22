# 数组 Array 常用 API

> 刷题高频，重点覆盖：创建、排序、查找、遍历、转换、以及双指针/二分/前缀和等套路。

## 一、创建与初始化

```java
int[] a = new int[5];            // 长度为5，默认全0
int[] b = {1, 2, 3, 4, 5};       // 直接初始化
int[] c = new int[]{1, 2, 3};    // new + 初始化

// 二维数组
int[][] grid = new int[3][4];           // 3行4列
int[][] grid2 = {{1,2},{3,4}};          // 直接初始化
int[][] grid3 = new int[3][];           // 只定行，列后定（锯齿数组）

// 快速填充
int[] f = new int[10];
Arrays.fill(f, -1);                      // 全填 -1
Arrays.fill(f, 0, 5, 100);               // 只填 [0,5) 区间
Arrays.fill(f, 0, f.length, 1);          // 1D 前缀/DP 常用
```

## 二、增 / 删（数组定长，靠拷贝移动）

数组长度固定，不能直接 add/remove，需要用 `System.arraycopy` 或转成 `ArrayList`：

```java
int[] a = {1, 2, 3, 4, 5};

// 删除下标 index 的元素（整体前移）
int index = 2;
System.arraycopy(a, index + 1, a, index, a.length - index - 1);

// 在 index 位置插入 val（注意：数组长度没变，需新建更大数组或留空位）
int[] bigger = new int[a.length + 1];
System.arraycopy(a, 0, bigger, 0, index);
bigger[index] = 99;
System.arraycopy(a, index, bigger, index + 1, a.length - index);
```

> 刷题结论：**频繁增删用 `ArrayList`**，数组适合定长访问。

## 三、查

| 操作 | 方法 | 复杂度 |
| ---- | ---- | ------ |
| 按下标取值 | `a[i]` | O(1) |
| 数组长度 | `a.length` | O(1) |
| 线性查找 | 手动 for 循环 | O(n) |
| 有序数组二分查找 | `Arrays.binarySearch(a, key)` | O(log n) |

```java
int[] a = {1, 3, 5, 7, 9};
Arrays.binarySearch(a, 5);    // 返回 2（5 的下标）
Arrays.binarySearch(a, 4);    // 不存在，返回 -3（-插入点-1，插入点为 2）
Arrays.binarySearch(a, 1, 5, 7)  // 在 [1,5) 区间内查找
```

## 四、改

```java
int[] a = {1, 2, 3};
a[0] = 99;          // {99,2,3}
```

## 五、排序

```java
int[] a = {3, 1, 4, 1, 5, 9, 2};
Arrays.sort(a);                      // 升序
Arrays.sort(a, 1, 4);                // 只排 [1,4) 区间
// Arrays.parallelSort(a);           // 大数据并行排序

// 降序：对 int[] 需先装箱为 Integer[]
Integer[] b = {3, 1, 4};
Arrays.sort(b, Collections.reverseOrder());

// 二维数组按第0列升序
int[][] p = {{3, 1}, {1, 2}, {2, 3}};
Arrays.sort(p, (x, y) -> x[0] - y[0]);             // 按第0列
Arrays.sort(p, (x, y) -> x[0] - y[0] == 0 ? x[1] - y[1] : x[0] - y[0]);  // 第0列相同再按第1列

// 推荐用 Comparator（避免减法溢出/更清晰）
Arrays.sort(p, Comparator.comparingInt(x -> x[0]));
Arrays.sort(p, Comparator.comparingInt(x -> x[0]).thenComparingInt(x -> x[1]));
```

## 六、遍历

```java
int[] a = {1, 2, 3, 4};

// 1. 下标 for
for (int i = 0; i < a.length; i++) { ... }

// 2. for-each
for (int x : a) { ... }

// 3. Stream（lambda）
Arrays.stream(a).forEach(x -> { ... });
```

## 七、转换

| 场景 | 写法 |
| ---- | ---- |
| 数组 → 字符串 | `Arrays.toString(a)` → `"[1, 2, 3]"` |
| 字符串 → 字符数组 | `s.toCharArray()` |
| 字符串数组 → 字符串 | `String.join(",", strArr)` |
| 数组 → List | `Arrays.asList(arr)`（⚠️ 定长视图） |
| List → 数组 | `list.toArray(new int[0])`（对象数组） |
| 数组拷贝 | `Arrays.copyOf(a, n)` / `a.clone()` |
| int[] → List\<Integer\> | `Arrays.stream(a).boxed().collect(Collectors.toList())` |

```java
int[] a = {1, 2, 3};
List<Integer> list = Arrays.stream(a).boxed().collect(Collectors.toList());
String s = Arrays.toString(a);            // "[1, 2, 3]"
int[] copy = Arrays.copyOf(a, a.length);  // 拷贝
```

## 八、复杂度对照

| 操作 | 复杂度 |
| ---- | ------ |
| 按下标访问/赋值 | O(1) |
| 线性查找 | O(n) |
| 二分查找（有序） | O(log n) |
| 排序 | O(n log n) |
| 插入/删除（需移动元素） | O(n) |
| `System.arraycopy` | O(n) |

## 九、刷题高频套路

### 1. 二分查找（左闭右闭模板）

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

### 2. 双指针（左右指针 / 快慢指针）

```java
// 左右指针：找到满足条件的两数
int i = 0, j = a.length - 1;
while (i < j) {
    int sum = a[i] + a[j];
    if (sum == target) { ... } 
    else if (sum < target) i++;
    else j--;
}

// 快慢指针：原地去重
int slow = 0;
for (int fast = 0; fast < a.length; fast++) {
    if (a[fast] != a[slow]) a[++slow] = a[fast];
}
```

### 3. 前缀和

```java
int[] pre = new int[a.length + 1];   // pre[i] 表示 a[0..i-1] 的和
for (int i = 0; i < a.length; i++) pre[i + 1] = pre[i] + a[i];
// 区间 [l, r] 的和 = pre[r+1] - pre[l]
```

### 4. 差分数组（区间加减）

```java
int[] diff = new int[a.length + 1];
// 区间 [l, r] 加 val
diff[l] += val;
diff[r + 1] -= val;
// 前缀和还原
for (int i = 1; i < diff.length; i++) diff[i] += diff[i - 1];
```

### 5. 旋转 / 矩阵遍历

```java
// 顺时针旋转 90°：转置 + 每行逆序
for (int i = 0; i < n; i++) for (int j = i + 1; j < n; j++) swap(m[i][j], m[j][i]);
for (int[] row : m) reverse(row);
```

## 十、常见坑

1. **数组定长**：不能直接 `add`/`remove`，需 `System.arraycopy` 或转 `ArrayList`。
2. **`Arrays.asList` 返回定长视图**：不能 `add`/`remove`，否则抛 `UnsupportedOperationException`。
3. **`Arrays.asList(int[])` 有坑**：`int[]` 会被当成**单个元素**处理（得到 `List<int[]>`），正确做法是 `Arrays.stream(a).boxed()...`。
4. **`toArray()` 泛型**：`list.toArray(new T[0])`，不要用无参 `toArray()`（返回 Object[]）。
5. **`==` 比的是引用**：数组内容比较用 `Arrays.equals(a, b)`；二维用 `Arrays.deepEquals`。
6. **越界**：下标访问注意 `0 <= i < a.length`，越界抛 `ArrayIndexOutOfBoundsException`。
7. **`Arrays.binarySearch` 找不到**：返回负数 `-(插入点)-1`，要区分"找到但下标为0"和"没找到"。
8. **减法溢出**：自定义 `Comparator` 里 `x[0]-y[0]` 可能溢出，建议 `Integer.compare` 或 `Comparator.comparingInt`。
