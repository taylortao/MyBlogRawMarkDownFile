---
title: Java cheatsheet
date: 2021-12-02 19:37:25
categories:
 - IT Technology
tags:
 - Java
 - Cheatsheet
---

### 1-dimension array

```java
int[] arr = new int[]{ 1, 2, 3 };
int[] arr2 = {1,2,3,4,5};
int len = arr.length; // 3
int len2 = arr2.length; // 5
```

### 2-dimension array

```java
int[][] matrix = {{1, 2}, {3, 4}};
```


<!-- more -->


### Linklist object

```java
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

### String

```java
String str = "hello";
int len = str.length(); // 5
Char c = str.charAt(0); // 'h'

// sub string
"String".substring(int beginIndexInclusive, int endIndexExclusive);

"T".repeat(3); // TTT
```

### Number

```java
Integer.toBinaryString(15) // "1111" 
Integer.parseInt("1111", 2) // 15

BigInteger n = BigInteger.ONE;
n.multiply(BigInteger.valueOf(123));
n.divide(BigInteger.valueOf(123));
```

Angle could be measured in degrees or radians. 
`atan` stands for "arc tangent." It is the inverse function of tangent - this means it undoes the tangent function. So `atan(tan(30)) = 30`.

`atan(delta-y/delta-x) == atan2(delta-y, delta-x)` // be mindful delta x could be zero
`Math.atan(1) == Math.PI / 4.0 == Math.atan2(1, 1)`


### StringBuilder

```java
// initialize with String
StringBuilder sb = new StringBuilder("String");

// append
sb.append("String"); // StringString

// insert
sb.insert(0, Integer.toString(123)); // 123StringString
sb.insert(3, "s"); // 123sStringString

// delete
sb.deleteCharAt(4); // 123stringString

// replace
sb.setCharAt(9, 's'); // 123stringstring

sb.reverse(); // reverse all chars

sb.toString();
```

#### Two ways to copy string 

```java
String original = "Original";
String copiedString1 = String.valueOf(original);
String copiedString2 = new StringBuilder(original).toString();
```

####

```java
String sentence = "Hello world hello again";
List<String> splits = Arrays.asList(sentence.split("\\s+"));
Collections.reverse(splits);
String.join(" ", splits); // again hello world Hello
```

### Containers
![Java container 1](java_container_1.png)
![Java container 2](java_container_2.png)
![Complexity](container_complexity.png)


#### List

```java
List<Integer> list = new LinkedList<>();
list.add(1);
list.size(); // 1

List<Integer> list2 = new LinkedList<>(List.of(1, 3, 5));
list2.contains(1); // true
list2.get(2); // 5
list2.remove(1); // {1, 5}

List<Integer> list3 = new ArrayList<>(Arrays.asList(1, 5, 9, 11));
list3.isEmpty(); // false
list3.remove(0); // 5, 9, 11
```

Vectors are synchronized, ArrayLists are not.

#### Array / List conversion

```java
List<String> list = Arrays.asList("C", "C++", "Java");
String[] array = list.toArray();
```

#### Copy LinkedList

```java
LinkedList<String> list = new LinkedList<>();
list.add("test");

List<Integer> clonedList = new LinkedList<Integer>();
clonedList = (LinkedList) list.clone();
```


#### Stack

```java
Stack<Integer> st = new Stack<>();
st.push(1); st.push(2); st.push(3);
Integer top = st.pop(); // 3
Integer peekTop = st.peek(); // 2
st.isEmpty(); // false

Integer pos = st.search(1); // 2
Integer pos2 = st.search(2); // 1
Integer pos3 = st.search(3); // -1
```

#### Queue

```java
Queue<Integer> q = new LinkedList<>();
q.add(1); q.add(2); q.add(3);
q.isEmpty(); // false
Integer top = q.poll(); // 1
Integer peekTop = q.peek(); // 2

```

#### Deque

```java
Deque<Integer> deque = new LinkedList<>();
deque.add(1); deque.addLast(2); // [1 2]
deque.addFirst(3); // [3 1 2]

deque.peekFirst(); // 3
deque.peekLast(); // 2
deque.size(); // 3
deque.removeFirst();
deque.removeLast(); // [1]
```


#### Heap / Priority queue

```java
PriorityQueue<Integer> pq = new PriorityQueue();
pq.addAll(List.of(3,5,7,9,11,1));
int top = pq.poll(); // 1
int peekTop = pq.peek(); // 3
pq.isEmpty(); // false
pq.remove(7); // [3, 5, 11, 9]
int size = pq.size(); // 4

PriorityQueue<Integer> pq = new PriorityQueue(Comparator.reverseOrder());
pq.addAll(List.of(3,5,7,9,11,1));
int top = pq.poll(); // 11
int peekTop = pq.peek(); // 9
pq.isEmpty(); // false
pq.remove(7); // [9, 3, 5, 1]
int size = pq.size(); // 4

PriorityQueue<String> pq = new PriorityQueue<String>(Collections.reverseOrder(Comparator.comparing(String::length));
```

#### HashSet HashMap HashTable
- HashTables are synchronized, HashMaps are not.
- HashSet is unordered and unsorted Set. LinkedHashSet is the ordered version of HashSet, LinkedHashSet maintains the insertion order. When we iterate through a HashSet, the order is unpredictable while it is predictable in case of LinkedHashSet.

HashSet:

```java
HashSet<Integer> set = new HashSet<>();
set.add(1); set.add(3); set.add(9);
set.isEmpty(); // false
set.contains(9); // true

for (Object num : set) {}

new ArrayList(set); // convert to arraylist
```

HashMap:

```java
HashMap<Integer, Integer> map = new HashMap<>();
map.put(1,2); map.put(3,4); map.put(5,6); map.put(7,8);
map.containsKey(1); // true
map.containsKey(2); // false
map.put(1, 10);
map.get(1); // 10
map.get(10); // null

map.computeIfAbsent(9, k -> new LinkedList<>()); // if the key is not already associated with a value, associates the key with the computed value.
map.getOrDefault(9, new LinkedList<>()); // if key not exist return the default value
```


#### TreeSet

```java
TreeSet<Integer> set = new TreeSet<>();
set.add(1); set.add(3); set.add(5); set.add(7);
set.ceiling(2); // 3
set.ceiling(3); // 3
set.floor(2); // 1
set.floor(3); // 3
set.lower(2); // 1
set.lower(3); // 1
set.higher(2); // 3
set.higher(3); // 5
```

#### TreeMap

Under the hood, it is implemented by a red-black tree structure.

```java
TreeMap<Integer, Integer> map = new TreeMap();
map.put(1, 11); map.put(2, 22); map.put(5, 55); map.put(3, 33);
Integer v = map.get(3); // 33
Integer v2 = map.get(8); // null
boolean c1 = map.containsKey(3); // true
boolean c2 = map.containsValue(22); // true

Integer first = map.firstKey(); // 1
Integer last = map.lastKey(); // 5
map.remove(1);
Integer first2 = map.firstKey(); // 2
map.ceilingKey(4); // 5
map.ceilingKey(5); // 5
map.floorKey(4); // 3
map.higherKey(4); // 5
map.higherKey(5); // null
map.lowerKey(4); // 3

map.get(1); // 11
map.get(8); // null
```

### Sort

#### Sort array

```java
Integer[] arr = { 13, 7, 6, 45, 21};
Arrays.sort(arr); // [6, 7, 13, 21, 45]
Arrays.sort(arr, Collections.reverseOrder()); // [45, 21, 13, 7, 6]
```

#### Sort collection

```java
List<Integer> list = new LinkedList<>(List.of(3, 1, 2, 8, 7));
Collections.sort(list); // [1, 2, 3, 7, 8]
Collections.sort(list, Collections.reverseOrder()); // [8, 7, 3, 2, 1]
```

#### Custom comparator

```java
class TestClass {
    int sortKey;
    String otherProps;
    public TestClass(int k, String v) {
        sortKey = k;
        otherProps = v;
    }
}

class SortByKey implements Comparator<TestClass> {
    @Override
    public int compare(TestClass t1, TestClass t2) {
        if (t1.sortKey < t2.sortKey) {
            return -1;
        } else if (t1.sortKey == t2.sortKey) {
            return 0;
        } else {
            return 1;
        }
    }
}

List<TestClass> list = new LinkedList<>(List.of(
        new TestClass( 1, "a"),
        new TestClass( 8, "ddd"),
        new TestClass( 3, "aaa"),
        new TestClass( 11, "cc"),
        new TestClass( 6, "b")
));

Collections.sort(list, new SortByKey()); // [1, 2, 3, 7, 8]
// [1 -> a, 3 -> aaa, 6 -> b, 8 -> ddd, 11 -> cc]
```

Another way to implement, which might be easier

```java
class TestClass implements Comparable<TestClass> {
    int sortKey;
    String otherProps;
    public TestClass(int k, String v) {
        sortKey = k;
        otherProps = v;
    }

    @Override
    public int compareTo(TestClass o) {
        return Integer.compare(sortKey, o.sortKey);
    }
}
```

### Java stream

```java
List<String> fromList = Arrays.asList("zza", "bb", "ccc", "dddd");
List<String> result = fromList.stream()
    .filter((item) -> item.length() > 0) // [zza, bb, ccc, dddd]
    .sorted((s1, s2) -> s1.charAt(0) - s2.charAt(0)) // [bb, ccc, dddd, zza]
    .map(s -> s.replace("a", "A")) //[bb, ccc, dddd, zzA]
    .collect(Collectors.toList());

Optional<String> reducedResult = result.stream()
    .reduce((s1, s2) -> String.format("%s:%s", s1, s2)); // bb:ccc:dddd:zzA

String[] fromArray = {"a", "b", "c"};
long count = Arrays.stream(fromArray).count(); // 3
```

### Java IO

Byte Streams, file content: test

```java
try (
    InputStream fis = new FileInputStream(filePathInput);
    OutputStream fos = new FileOutputStream(filePathOutput)
) {
    int content;
    while ((content = fis.read()) != -1) {
        System.out.print((char) content);
        System.out.print(","); // t,e,s,t,
    }
    fos.write("hello".getBytes());
} catch (IOException e) {
}
```

I/O operations are very performance-intensive. Buffered streams load data into a buffer and read/write multiple bytes at once, thereby avoiding frequent I/O operations and improving the efficiency of stream transmission. 

Byte buffered streams use the decorator pattern to enhance the functionality of the InputStream and OutputStream

```java
try (BufferedReader reader = new BufferedReader(new FileReader(filePathInput));
     BufferedWriter writer = new BufferedWriter(new FileWriter(filePathOutput))) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine();
    }
} catch (IOException e) {
}
```

Character streams use Unicode encoding by default, but we can customize the encoding through the constructor. 
- UTF-8: English characters: 1 byte, Chinese characters: 3 bytes. 
- Unicode: Any character: 2 bytes. 
- GBK: English characters: 1 byte, Chinese characters: 2 bytes.

```java
try (FileReader fileReader = new FileReader(filePathInput, StandardCharsets.UTF_8);) {
    int content;
    while ((content = fileReader.read()) != -1) {
        System.out.print((char) content);
    }
} catch (IOException e) {
}
```

