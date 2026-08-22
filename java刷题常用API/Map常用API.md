# Map相关常用方法

> AI生成,人工优化了一些内容
>
> 可能有些地方不太好,后续会慢慢优化~

## 一、创建与初始化

```java
Map<Integer, Integer> map = new HashMap<>();          // 最常用
Map<Integer, Integer> map2 = new HashMap<>(16);       // 预估容量，减少扩容
Map<String, Integer> map3 = new LinkedHashMap<>();    // 保持插入顺序（LRU 用）
Map<Integer, String> map4 = new TreeMap<>();          // key 有序，O(log n)
```

不可变 Map（Java 9+，最多 10 对键值）：

```java
Map<String, Integer> m = Map.of("a", 1, "b", 2);
```

## 二、增 / 改

| 方法                                  | 作用             | 返回值                        |
| ------------------------------------- | ---------------- | ----------------------------- |
| `put(K k, V v)`                       | 添加或覆盖       | 旧值（原来没有则返回 null）   |
| `putIfAbsent(K k, V v)`               | 不存在才放入     | 旧值（原来没有则返回 null）   |
| `merge(K k, V v, BiFunction f)`       | 合并/累加        | 合并后的新值                  |
| `computeIfAbsent(K k, Function f)`    | 不存在才计算放入 | 已存在或刚计算的值            |
| `computeIfPresent(K k, BiFunction f)` | 存在才更新       | 更新后的值（不存在返回 null） |
| `replace(K k, V v)`                   | 存在才替换       | 旧值（不存在返回 null）       |

> `putIfAbsent(K k, V v)`：若 key 不存在（或映射为 null）才放入新值；否则不修改，返回旧值。

- **merge方法示例**

```java
HashMap<String,Integer> map = new HashMap<>();
//第一次，a不存在，存入 1
map.merge("a",1, (oldVal,newVal)-> oldVal+newVal );
//第二次，a存在，旧值1 +新值1 →结果=2
map.merge("a",1, (oldVal,newVal)-> oldVal+newVal );
System.out.println(map); // {a=2}
```

- **computeIfAbsent方法示例**

```java
Map<String,Integer> map=new HashMap<>();
map.put("a",10);

map.computeIfAbsent("a",k->99); //a存在→不执行，a=10
map.computeIfAbsent("b",k->20);//b不存在→存入b=20
```

- **computeIfPresent方法示例**

```java
Map<String,Integer> map=new HashMap<>();
map.put("a",10);

map.computeIfPresent("a",(k,old)->old*2); //a存在 → a=20
map.computeIfPresent("b",(k,old)->99);     //b不存在→不执行
```

## 三、查

| 方法                       | 说明                                   |
| -------------------------- | -------------------------------------- |
| `get(K k)`                 | 不存在返回 null（无法区分"存了 null"） |
| `getOrDefault(K k, V def)` | 不存在返回默认值，常用来计数          |
| `containsKey(K k)`         | 返回boolean,时间复杂度O(1)             |
| `containsValue(V v)`       | 返回boolean,时间复杂度O(n)，刷题少用   |
| `size()` / `isEmpty()`     | 常用                                   |

> `size()`获取 Map 里面键‑值对（entry）的**总个数**
>
> `isEmpty()`判断是否为空

## 四、删

| 方法               | 说明                                  |
| ------------------ | ------------------------------------- |
| `remove(K k)`      | 返回被删的值                          |
| `remove(K k, V v)` | key 和 value 都匹配才删，返回 boolean |
| `clear()`          | 清空                                  |

## 五、遍历

```java
// 1. 遍历 key
for (Integer k : map.keySet()) { ... }

// 2. 只遍历 value
for (Integer v : map.values()) { ... }

// 3. 遍历键值对（最常用）
for (Map.Entry<Integer, Integer> e : map.entrySet()) {
    int k = e.getKey(), v = e.getValue();
}

// 4. lambda（Java 8+）
map.forEach((k, v) -> { ... });
```

> 需要同时用 key 和 value 时，优先用 `entrySet()`（第 3 种），比 `keySet()` + `get()` 少一次查找。

## 六、刷题可用

### 1. 统计出现次数（词频）

```java
// merge
map.merge(x, 1, Integer::sum);                 
map.merge(x, 1, (oldVal, newVal) -> oldVal + newVal);

// put
map.put(x, map.getOrDefault(x, 0) + 1);        
```

### 2. 判断重复 / 是否已存在

```java
if (map.containsKey(x)) { ... }

// 或利用 put 的返回值（仅当 value 不会为 null 时适用）：
if (map.put(x, i) != null) { ... }            // 原来存在时返回旧值
```

### 3. 两数之和：边遍历边查表（HashMap 存索引）

```java
Map<Integer, Integer> idx = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    if (idx.containsKey(target - nums[i])) {
        return new int[]{idx.get(target - nums[i]), i};
    }
    idx.put(nums[i], i);
}
```

### 4. 分组（如字母异位词分组）

```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
```

### 5. 遍历中安全删除

```java
map.entrySet().removeIf(e -> e.getValue() == 0);

// 或迭代器方式：
Iterator<Map.Entry<K, V>> it = map.entrySet().iterator();
while (it.hasNext()) {
    if (...) it.remove();
}
```

## 七、三个实现类怎么选

| 实现类        | 特点         | 复杂度    | 典型场景                                                     |
| ------------- | ------------ | --------- | ------------------------------------------------------------ |
| HashMap       | 无序，最常用 | 均摊 O(1) | 绝大多数题                                                   |
| LinkedHashMap | 保持插入顺序 | O(1)      | LRU 缓存                                                     |
| TreeMap       | key 自然有序 | O(log n)  | 有序/区间查询，`floorKey` / `ceilingKey` / `firstKey` / `lastKey` |

## 八、常见坑

1. **get 返回 null 的两义性**：key 不存在和 value 为 null 都返回 null，需要区分时用 `containsKey`。
2. **自定义对象当 key**：必须正确重写 `hashCode()` 和 `equals()`。
3. **遍历时直接 remove**：在 `for (k : map.keySet())` 循环里调用 `map.remove(k)` 会抛 `ConcurrentModificationException`，改用 `removeIf` 或迭代器。
4. **Integer 比较**：-128~127 之外的对象用 `==` 会出错，比较值请用 `equals`。
