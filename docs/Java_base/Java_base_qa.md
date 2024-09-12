> [!TIP]
>
> - IDEA　IntelliJ IDEA 2024.1.6
> - Java　17

# 数组

## 遍历集合是remove操作bug

**代码如下：**

```java
@Test
    public void iteratorRemoveBug() {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()) {
            Integer val = iterator.next();
            if (val == 3) {
                list.remove(val);
            }
        }
        list.forEach(System.out::println);
    }
```

**报错信息如下：**

```shell
java.util.ConcurrentModificationException
	at java.base/java.util.ArrayList$Itr.checkForComodification(ArrayList.java:1013)
	at java.base/java.util.ArrayList$Itr.next(ArrayList.java:967)
	at com.superluckyhu.top.interview.java.base.JavaBaseTest.iteratorRemoveBug(JavaBaseTest.java:27)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
```

**抛出异常：**<span style="color:red;font-weight:bold">ConcurrentModificationException</span>

**阿里开发手册：**

![image-20240912210227219](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122102295.png)

**探究：**

```java
 public E next() {
     //迭代器在next的时候，会检查数组长度
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
...........
    //如果实际的和期望的长度不一致，就会抛出异常
    final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

**更改：**

```java
 @Test
    public void iteratorRemoveBug() {
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);
        list.add(5);
        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()) {
            Integer val = iterator.next();
            if (val == 3) {
//                list.remove(val);//java.util.ConcurrentModificationException
                iterator.remove();
            }
        }
        list.forEach(System.out::println);
    }
```

## 使用工具类 `Arrays.asList()` 把数组转换成集合bug

**代码如下：**

```java
    @Test
    public void arraysAsListBug() {
        String[] myArray = {"Apple", "Banana", "Orange"};
        List<String> list = Arrays.asList(myArray);
        list.add("Pear");
        list.forEach(System.out::println);

    }
```

**报错信息如下：**

```shell
java.lang.UnsupportedOperationException
	at java.base/java.util.AbstractList.add(AbstractList.java:153)
	at java.base/java.util.AbstractList.add(AbstractList.java:111)
	at com.superluckyhu.top.interview.java.base.JavaBaseTest.arraysAsListBug(JavaBaseTest.java:41)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
```

**抛出异常：**<span style="color:red;font-weight:bold">UnsupportedOperationException</span>

**阿里开发手册：**

![image-20240912212205115](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122122206.png)

**探究：**

```java
  @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
....
    
```

`new ArrayList<>(a);`属于`Arrays`中的内部类，且没有重写父类的add等方法。当传入一个原生数据类型数组时，`Arrays.asList()` 的真正得到的参数就不是数组中的元素，而是数组对象本身！此时 `List` 的唯一元素就是这个数组。

![image-20240912212733669](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122127817.png)

![image-20240912212946512](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122129702.png)

**更改：**

```java
 @Test
    public void arraysAsListBug() {
        String[] myArray = {"Apple", "Banana", "Orange"};
        List<String> list = new ArrayList<>(Arrays.asList(myArray));
        list.add("Pear");
        list.forEach(System.out::println);
//        使用 Java8 的 Stream
        List<String> streamList = Arrays.stream(myArray).collect(Collectors.toList());
        streamList.add("Strawberry");
        streamList.forEach(System.out::println);
    }
```

## List去重方法

```java
  @Test
    public void listRemoveDuplicatesDemo(){

        listRemoveDuplicatesMethod1();
        listRemoveDuplicatesMethod2();
        listRemoveDuplicatesMethod3();
        listRemoveDuplicatesMethod4();
        listRemoveDuplicatesMethod5();
        
        
    }

    private void listRemoveDuplicatesMethod5() {
        Integer[] myArray = {89,73,21,22,21,10,90,89,73};
        List<Integer> list = Arrays.asList(myArray);
        List<Integer> sourceList = new ArrayList<>(list);
        List<Integer> targetList = new ArrayList<>(list);
        for (int i = 0; i < targetList.size()-1; i++) {
            for (int j = targetList.size()-1; j >i; j--) {
                if (targetList.get(j).equals(targetList.get(i))) {
                    targetList.remove(j);
                }
            }
        }
        System.out.println(sourceList);
        System.out.println(targetList);
    }

    /**
     * @description 类似双指针，一个从头找，一个从尾找，如果返回的下标不同，则说明有重复的元素
     * @author Superluckyhu
     * @apiNote       
     * @date 2024/9/12 23:08
     * @return: void
     **/
    private void listRemoveDuplicatesMethod4() {
        Integer[] myArray = {89,73,21,22,21,10,90,89,73};
        List<Integer> list = Arrays.asList(myArray);
        List<Integer> sourceList = new ArrayList<>(list);
        List<Integer> targetList = new ArrayList<>(list);
        for (Integer item : sourceList) {
            if (targetList.indexOf(item) != targetList.lastIndexOf(item)){
                targetList.remove(targetList.lastIndexOf(item));
            }
        }
        System.out.println(sourceList);
        System.out.println(targetList);
        
    }
    /**
     * @description stream流去重
     * @author Superluckyhu
     * @apiNote       
     * @date 2024/9/12 23:02
     * @return: void
     **/
    private void listRemoveDuplicatesMethod3() {
        Integer[] myArray = {89,73,21,22,21,10,90,89,73};
        List<Integer> sourceList = new ArrayList<>(Arrays.asList(myArray));
        List<Integer> targetList = new ArrayList<>();
        targetList = sourceList.stream().distinct().collect(Collectors.toList());
        targetList.forEach(System.out::println);
    }

    /**
     * @description 使用HashSet或LinkedHashSet
     * @author Superluckyhu
     * @apiNote       
     * @date 2024/9/12 22:58
     * @return: void
     **/
    private void listRemoveDuplicatesMethod2() {
//        hashset去重但无序
        Integer[] myArray = {89,73,21,22,21,10,90,89,73};
        List<Integer> sourceList = new ArrayList<>(Arrays.asList(myArray));
        List<Integer> list = new ArrayList<>(new HashSet<>(sourceList));
        list.forEach(System.out::println);
        System.out.println("*******************");
//        linkedhashset去重且有序
        List<Integer> linkedHashSet = new ArrayList<>(new LinkedHashSet<>(sourceList));
        linkedHashSet.forEach(System.out::println);
    }

    /**
     * @description for循环遍历
     * @author Superluckyhu
     * @apiNote       
     * @date 2024/9/12 22:54
     * @return: void
     **/
    private void listRemoveDuplicatesMethod1() {

        Integer[] myArray = {89,73,21,22,21,10,90,89,73};
        List<Integer> sourceList = new ArrayList<>(Arrays.asList(myArray));
        List<Integer> targetList = new ArrayList<>();
        for (int i = 0; i < sourceList.size(); i++) {
            if (!targetList.contains(sourceList.get(i))) {
                targetList.add(sourceList.get(i));
            }
        }
        System.out.println(sourceList);
        System.out.println(targetList);

    }
```



# HashCode冲突案例

```java
 @Test
    public void hashCodeConflictDemo() {
        HashSet<Integer> container = new HashSet<>();
        for (int i = 0; i < 15 * 10000; i++) {
            int objHashCode = new Object().hashCode();
            if (!container.contains(objHashCode)) {
                container.add(objHashCode);
            } else {
                System.out.println("发生了hash冲突了，在第" + i + "次，hash值为" + objHashCode);
            }
        }
        System.out.println(container.size());
    }
```

# Integer比较

```java
 @Test
    public void integerDemo(){
        Integer i1 = Integer.valueOf(600);
        Integer i2 = Integer.valueOf(600);
        int i3 = 600;
        System.out.println(i1 == i2);//false
        System.out.println(i1 == i3);//true
        System.out.println(i1.equals(i2));//true
        System.out.println("*******************");
        Integer j1 = Integer.valueOf("80");
        Integer j2 = Integer.valueOf("80");
        System.out.println(j1 == j2);//true
        System.out.println(j1.equals(j2));//true

    }
```

**阿里开发手册：**

![image-20240912222554436](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122225538.png)

**源码及注释：**

```Java
/**
 * Returns an {@code Integer} instance representing the specified
 * {@code int} value.  If a new {@code Integer} instance is not
 * required, this method should generally be used in preference to
 * the constructor {@link #Integer(int)}, as this method is likely
 * to yield significantly better space and time performance by
 * caching frequently requested values.
 *
 * This method will always cache values in the range -128 to 127,
 * inclusive, and may cache other values outside of this range.
 *
 * @param  i an {@code int} value.
 * @return an {@code Integer} instance representing {@code i}.
 * @since  1.5
 */
@IntrinsicCandidate
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
 private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer[] cache;
        static Integer[] archivedCache;

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    h = Math.max(parseInt(integerCacheHighPropValue), 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(h, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            // Load IntegerCache.archivedCache from archive, if possible
            CDS.initializeFromArchive(IntegerCache.class);
            int size = (high - low) + 1;

            // Use the archived cache if it exists and is large enough
            if (archivedCache == null || size > archivedCache.length) {
                Integer[] c = new Integer[size];
                int j = low;
                for(int i = 0; i < c.length; i++) {
                    c[i] = new Integer(j++);
                }
                archivedCache = c;
            }
            cache = archivedCache;
            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }
```

# BIgdecimal坑

**阿里开发手册：**

![image-20240912223310401](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122233505.png)

![image-20240912223428928](https://docs-pics.oss-cn-chengdu.aliyuncs.com/images/202409122234025.png)









































