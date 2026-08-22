# List 常用 API（ArrayList）

> 刷题 99% 用 `ArrayList`，`LinkedList` 极少用（见文末复杂度对比）。只讲高频、好记的方法，难一点的都配了小例子。

## 一、创建与初始化

```java
List<Integer> list = new ArrayList<>();          // 最常用
List<Integer> list2 = new ArrayList<>(16);       // 预估容量
List<Integer> list3 = Arrays.asList(1, 2, 3);    // ⚠️ 定长视图，不能 add/remove
List<Integer> list4 = List.of(1, 2, 3);          // Java 9+，不可变
```

> `Arrays.asList` 和 `List.of` 返回的 List **不能增删**。想要可变 List，用 `new ArrayList<>(Arrays.asList(1, 2, 3))` 包一层。

## 二、增

```java
List<Integer> list = new ArrayList<>();
list.add(1);           // 追加到末尾：[1]
list.add(0, 99);       // 在指定位置插入：[99, 1]
```

## 三、删（重点：两个 remove 的区别）

| 方法 | 删除依据 | 返回值 |
| ---- | -------- | ------ |
| `remove(int index)` | 按下标 | 被删的元素 |
| `remove(Object o)` | 按值（删第一个匹配） | boolean（删成功返回 true） |

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));

list.remove(1);                    // 删下标 1 → 删掉 2，结果 [1, 3]
list.remove(Integer.valueOf(3));   // 删值 3 → 结果 [1]
```

> ⚠️ **经典坑**：`remove(0)` 是"删下标 0"，不是"删值 0"。
> 要删整数 0 必须写 `remove(Integer.valueOf(0))`。

## 四、查

```java
list.get(i);        // 按下标取元素，O(1)
list.size();        // 元素个数
list.isEmpty();     // 是否为空
list.contains(x);   // 是否包含，O(n)
list.indexOf(x);    // 第一次出现的下标，不存在返回 -1
```

## 五、改

```java
list.set(0, 100);   // 把下标 0 改成 100，返回旧值
```

## 六、遍历

```java
// 1. for-each
for (int x : list) { ... }

// 2. 下标 for（需要下标时用）
for (int i = 0; i < list.size(); i++) { int x = list.get(i); ... }

// 3. lambda
list.forEach(x -> { ... });

// 4. 迭代器（遍历中要删除时用，见"常见坑"）
Iterator<Integer> it = list.iterator();
while (it.hasNext()) { int x = it.next(); ... }
```

## 七、转换

```java
// List → 数组（对象）
Integer[] arr = list.toArray(new Integer[0]);
// List<Integer> → int[]（拆箱）
int[] arr2 = list.stream().mapToInt(i -> i).toArray();

// 数组 → List（可变）
List<Integer> l = new ArrayList<>(Arrays.asList(arr));

// List<String> → 字符串
String s = String.join(",", strList);
// 字符串 → List<String>
List<String> parts = new ArrayList<>(Arrays.asList("a,b,c".split(",")));

// 两个高频小工具
Collections.reverse(list);          // 原地反转
Collections.swap(list, i, j);       // 交换下标 i 和 j
```

## 八、排序

```java
List<Integer> list = new ArrayList<>(Arrays.asList(3, 1, 2));
Collections.sort(list);                   // 升序
list.sort(Comparator.reverseOrder());     // 降序

// 按字符串长度排序（Comparator 最安全）
List<String> strs = new ArrayList<>(Arrays.asList("aaa", "b", "cc"));
strs.sort(Comparator.comparingInt(String::length));   // b, cc, aaa
```

## 九、subList（返回视图，改它会动原 List）

```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
List<Integer> sub = list.subList(1, 4);    // [1,4) → [2, 3, 4]

sub.set(0, 99);                            // 改 sub
System.out.println(list);                  // [1, 99, 3, 4, 5] ← 原 List 也被改了！

// 只想要子列表的"副本"，要 new 一份：
List<Integer> copy = new ArrayList<>(list.subList(1, 4));
```

## 十、复杂度对照

| 操作 | ArrayList | LinkedList |
| ---- | --------- | ---------- |
| 按下标 get / set | O(1) | O(n) |
| 末尾 add | 均摊 O(1) | O(1) |
| 中间插入 / 删除 | O(n) | O(1)（但需先定位） |
| 查找 contains / indexOf | O(n) | O(n) |

> 结论：**刷题基本都用 ArrayList**。LinkedList 只有"频繁在头部插入/删除"才可能用到，几乎遇不到。

## 十一、刷题高频套路

### 1. 存结果（二维 List）

```java
List<List<Integer>> ans = new ArrayList<>();
ans.add(Arrays.asList(1, 2, 3));
```

### 2. 双指针反转 List

```java
int i = 0, j = list.size() - 1;
while (i < j) {
    Collections.swap(list, i, j);
    i++; j--;
}
```

### 3. 去重（保留原顺序）

```java
List<Integer> res = new ArrayList<>();
Set<Integer> seen = new HashSet<>();
for (int x : list) if (seen.add(x)) res.add(x);
```

## 十二、常见坑

1. **`remove(0)` 删的是下标不是值**：删整数 0 要用 `remove(Integer.valueOf(0))`。
2. **`Arrays.asList` / `List.of` 定长**：不能 add/remove。
3. **`subList` 是视图**：改它会改原 List，要副本就 `new ArrayList<>(subList)`。
4. **遍历中删除会抛 `ConcurrentModificationException`**：改用 `removeIf` 或 `Iterator.remove()`。
5. **`toArray` 泛型**：用 `new Integer[0]`，别用无参 `toArray()`（返回 Object[]）。
