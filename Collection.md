# Collection

------

[TOC]



------



## List



 **`List<E>`是一种有序链表**

- List内部按照放入元素的先后顺序存放
- 每个元素都可以通过索引确定自己的位置
- 类似数组，但大小可变
- List的元素可以重复
- List的元素可以是`null`



### ArrayList & LinkedList

|                     |  ArrayList   |     LinkedList     |
| :-----------------: | :----------: | :----------------: |
|    获取指定元素     |    速度快    | 需从头开始查找元素 |
|   添加元素到末尾    |    速度快    |       速度快       |
| 在指定位置添加/删除 | 需要移动元素 |   不需要移动元素   |
|      内存占用       |      少      |        较大        |



**在List中查找元素**

- List的实现类通过元素的`equals`方法比较两个元素
- 放入的元素必须正确覆写`equals`方法
  - JDK提供的`String、Integer`等已经覆写了`equals`方法
- 编写`equals`方法可借助`Objects.equals()`来判断

**如果不在List中查找元素则不必覆写equals方法**



#### Code Demo

list to array

```java
import java.util.ArrayList;
import java.util.List;


public class list2Array{
        public static void main(String[] args){
                // 创建一个ArrayList对象
                List<String> list = new ArrayList<>();
                list.add("Apple");
                list.add("Pear");
                list.add("Orange");
                
                // for each方法由编译器自动转换为Iterator方法来遍历该ArrayList对象
                for(String s : list){
                        System.out.println(s);
                }
             	
                 
                // 通过toArray方法来将ArrayList对象转换为Array对象
                String[] ss = list.toArray(new String[list.size()]);

                for(String s : ss){
                        System.out.println(s);
                }

        }
 }

/**
 * result:
 * 
 * Apple
 * Pear
 * Orange
 * Apple
 * Pear
 * Orange
 */
```



```java
import java.util.Objects;

public class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    /**
     * @return the name
     */
    public String getName() {
        return name;
    }

    /**
     * @return the age
     */
    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person: " + this.name + ", " + this.age;
    }

    /**
     * equals
     * 
     * 1. determine if thess objects have the same reference, return true or false
     * 2. determine if the object is a instance of Person, take object to a new Person object, and compare their items' values
     */
    @Override
    public boolean equals(Object obj) {
        if (this == obj){
            return true;
        }
        if (obj instanceof Person){
            Person p = (Person) obj;
            // 借助Objects.equals方法来比较
            return Objects.equals(p.name, this.name) && p.age == this.age;
        }
        
        return false;
    }
}
```



```java
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) {
        
        List<Person> list = new ArrayList<>();
        list.add(new Person("ming", 12));
        list.add(new Person("Hong", 15));
        list.add(new Person("Jing", 17));

        System.out.println(list); // [(Person: Ming, 12), (Person: Hong, 15), (Person: Jing, 17)]

        System.out.println(list.contains(new Person("Jing", 17))); // true
    }
}
```

------



## Map



**`Map<K, V>`是一种键-值（Key-Value）映射表，可通过Key快速查找Value**

- `Map.put(K key, V value)`:  把Key-Value放入Map
- `Map.get(K key)`:  通过Key获取Value
- `boolean containsKey(K key)`:  判断Key是否存在
- 可通过for each遍历`keySet()`
- 可通过for each遍历`entrySet()`
- 需要对Key排序时使用`TreeMap`
- 通常使用`HashMap`



#### Code Demo-Ⅰ 

```java
public class Person {

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    /**
     * @return the name
     */
    public String getName() {
        return name;
    }

    /**
     * @return the age
     */
    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person: " + name + ", " + age;
    }
}
```

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.TreeMap;
import java.util.List;
import java.util.Map;

import javax.security.auth.kerberos.KeyTab;

public class Main {

    public static void main(String[] args) {
        // 创建一个Person类的泛型List
        List<Person> list = Arrays.asList(new Person("Ming", 12), new Person("Hong", 15), new Person("Jing", 17));
        // 创建一个以String为key，Person为value的HashMap
        Map<String, Person> map = new HashMap<>();

        /**
         * 将list中的Person对象都放入HashMap中
         * 并定义Person对象的name为Key，Person对象为Value
         */
        for (Person p : list) {
            map.put(p.getName(), p);
        }

        // 通过keySet()从HashMap中获取Key的set集合
        // 用for each遍历set集合和对应的value
        // 通过map.get(key)获取key对应的value
        for (String key : map.keySet()) {
            System.out.println(key + " --> " + map.get(key));
        }
        
        System.out.println("========================================");
        
        // TreeMap
        // sort by key
        Map<String, Person> map1 = new TreeMap<>();

        for (Person p : list) {
            map1.put(p.getName(), p);
        }

        for (String key : map1.keySet()) {
            System.out.println(key + " --> " + map1.get(key));
        }

        System.out.println("========================================");
        
        /**
         * EntrySet
         * 通过entry同时获取key和value
         * map.entrySet() 获取set集合
         */
        for (Map.Entry<String, Person> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " --> " + entry.getValue());
        }
    }
}

/**
 * result:
 * 
 * Ming --> Person: Ming, 12
 * Hong --> Person: Hong, 15
 * Jing --> Person: Jing, 17
 * ========================================
 * Hong --> Person: Hong, 15
 * Jing --> Person: Jing, 17
 * Ming --> Person: Ming, 12
 * ========================================
 * Ming --> Person: Ming, 12
 * Hong --> Person: Hong, 15
 * Jing --> Person: Jing, 17
 */
```



**正确使用Map必须保证**

- 作为Key的对象必须正确重写`equals()`方法，例如: `String, Integer, Long...`
- 作为Key的对象同时也必须正确重写`hashCode()`方法：
  - 如果两个对象相等，则两个对象的`hashCode()`必须相等
  - 如果两个对象不相等，则两个对象的`hashCode()`不需要相等

 

如果一个对象重写了`equals()`方法，就必须正确重写`hashCode()`方法

- 如果`a.equals(b) == true`, 则`a.hashCode() == b.hashCode()`
- 如果`a.equals(b) == false`, 则a和b的`hashCode()`尽量不要相等



#### Code Demo - Ⅱ

```java
public class Person {

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    /**
     * @return the name
     */
    public String getName() {
        return name;
    }
    /**
     * @return the age
     */
    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person --> " + this.name + ", " + this.age;
    }

    // 重写equals()方法
    // 用Objects.equals()方法来比较对象
    @Override
    public boolean equals(Object obj){
        if (this == obj) {
            return true;
        }
        if (obj instanceof Person) {
            Person p = (Person) obj;
            return Objects.equals(p.name, this.name) && p.age == this.age;
        }
        return false;
    }

    // 重写hashCode()方法，
    // 确保相同名字和年龄的对象必须产生相同的hashcode
    public int hashCode() {
        return Objects.hash(this.name, this.age);
    }
} 
```

```java
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;

/**
 * Main
 */
public class Main {

    public static void main(String[] args) {
        // 创建一个Person类的泛型List
        List<Person> list = Arrays.asList(new Person("Ming", 12), new Person("Hong", 15), new Person("Jing", 17));
        // 创建一个以Person为Key String为Value的hashMap
        HashMap<Person, String> map= new HashMap<>();
        
        // 该hashmap以Person对象为Key，Person对象的name为Value
        for (Person p : list) {
            map.put(p, p.getName());
        }
        // get(Key) --> value
        // 通过get方法用Person对象获取其对象的name
        System.out.println(map.get(new Person("Jing", 17)));
    }
}

/**
 * result:
 *
 * Jing
 *
 * # 若Person类中未重写hashCode()方法，则返回null
 */
```

------



## Set

**Set用于存储不重复的元素集合**

- ﻿`boolean add (E e)`
- `boolean remove(Object o)`
- `boolean contains(Object o)`
- `int size()`

 

- **Set可视为不存储Value的Map**
- **Set用于去除重复元素**
- **放入Set的元素要正确实现`equals()`和`hashCode()`方法**

 

**Set不保证有序：**

- `HashSet`是无序的
- `TreeSet`是有序的
- 实现了`SortedSet`接口的是有序Set





#### Code Demo

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.HashSet;
import java.util.List;
import java.util.Set;
import java.util.TreeSet;

public class Main {

    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "orange", "grape", "peal", "banana", "apple");

        List<String> list_ = removeDuplicate(list);

        System.out.println(list_);
    }

    // 去除重复元素的方法
    private static List<String> removeDuplicate(List<String> list) {
        // 使用Set 和 TreeSet实现元素正序
        // 借助Comparator倒序输出元素
        Set<String> set = new TreeSet<>(new Comparator<String>() {

            @Override
            // 倒序
            public int compare(String o1, String o2) {
                return - o1.compareTo(o2);
            }
        });
        set.addAll(list);
        return new ArrayList<String>(set);
    }
}

/**
 * result:
 * 
 * [peal, orange, grape, banana, apple]
 * 
 * # 若无Comparator，则输出
 * [apple, banana, grape, orange, peal]
 */
```

------



## Queue



**`Queue<E>`实现了一个先进先出（FIFO）的队列**

- `LinkedList`实现了`Queue<E>`接口

 

- 获取队列长度：`size()`
- 添加元素到队尾：`boolean queue.add(Element e) / boolean queue.offer(Element e)`
- 获取队列==头部==元素并删除：` queue.remove() /  queue.poll()`
- 获取队列==头部==元素但不删除： `queue.element() / queue.peek()`
- ==避免把`null`添加进队列中==



#### Code Demo - Ⅰ 

```java
public class Person {

    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    /**
     * @return the name
     */
    public String getName() {
        return name;
    }

    /**
     * @return the age
     */
    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person - (" + name + ", " + age + ")";
    }
}
```

```java
import java.util.LinkedList;
import java.util.Queue;

public class Main {

    public static void main(String[] args) {
        Queue<Person> queue = new LinkedList<>();
        queue.offer(new Person("Ming", 12));
        queue.offer(new Person("Hong", 15));
        queue.offer(new Person("Jing", 17));

        // for (Person person : queue) {
        //     System.out.println(person);
        // }

        while (!queue.isEmpty()) {
            System.out.println(queue.poll());
        }
    }
}

/**
 * result:
 *
 * Person - (Ming, 12)
 * Person - (Hong, 15)
 * Person - (Jing, 17)
 */
```



### PriorityQueue

- `PriorityQueue<E>`实现一个优先队列
- 从队首获取元素时，总是获取优先级最高的元素
- 默认按元素比较的顺序排序（必须实现`comparable`接口）
- 可以通过Comparator自定义排序算法（不必实现`comparable`接口）



#### Code Demo-Ⅱ

```java
System.out.println("===========Priority===========");

Queue<Person> queue2 = new PriorityQueue<>(new Comparator<Person>() {
// 根据Person对象的name倒序排列
	@Override
	public int compare(Person o1, Person o2) {
		return - o1.getName().compareTo(o2.getName());
	}
});
queue2.offer(new Person("Ming", 12));
queue2.offer(new Person("Hong", 15));
queue2.offer(new Person("Jing", 17));

while (!queue2.isEmpty()) {
    System.out.println(queue2.poll());
}

/**
 * result:
 * 
 * ===========Priority===========
 * Person - (Ming, 12)
 * Person - (Jing, 17)
 * Person - (Hong, 15)
 */
```

------





## Deque

**Deque 实现一个双端队列（Double Ended Queue）**

- 既可以添加到队尾，也可以添加到队首
- 既可以从队首获取，也可以从队尾获取



```java
public interface Deque<E> extends Queue<E>
```

|                  |              Queue              |                  Deque                  |
| :--------------: | :-----------------------------: | :-------------------------------------: |
|  添加元素到队尾  | `add(element) / offer(element)` | `addLast(element) / offerLast(element)` |
| 去队首元素并删除 |       `remove() / poll()`       |      `removeFirst() / pollFirst()`      |
| 去队首元素不删除 |      `element() / peek()`       |       `getFirst() / peekFirst()`        |
|  添加元素到队首  |                                 |       `addFirst() / offerFirst()`       |
| 去队尾元素并删除 |                                 |       `removeLast() / pollLast()`       |
| 去队尾元素不删除 |                                 |        `getLast() / peekLast()`         |



#### Code Demo

```java
import java.util.Deque;
import java.util.LinkedList;

public class Main {

    public static void main(String[] args) {
        
        Deque<String> deque = new LinkedList<>();
        deque.offerFirst("apple");
        deque.addLast("orange");
        deque.offerLast("pear");
        deque.addFirst("lemon");

        // for (String string : deque) {
        //     System.out.println(string);
        // }

        // remove from tail of deque
        while (!deque.isEmpty()) {
            System.out.println(deque.pollLast());
        }
    }
}

/**
 * result:
 * 
 * pear
 * orange
 * apple
 * lemon
 */
```

------



## Stack

 

**栈（Stack）是一种后进先出（LIFO）的数据机构**

- 操作栈的元素的方法：
  - `push()`：压栈
  - `pop()`：出栈
  - `peek()`：取栈顶元素但不出栈
- Java使用Deque实现栈的功能，注意只调用push / pop / peek，避免调用Deque的其他方法
- 不要使用遗留类Stack



#### Code Demo

```java
import java.util.Deque;
import java.util.LinkedList;

public class toHex {

    public static void main(String[] args) {
        // toHex()为转十六进制函数
        String hex = toHex(12500);

        System.out.println(hex);

        if (hex.equalsIgnoreCase("30D4")) {
            System.out.println("Passed");
        } else {
            System.out.println("Failed");
        }
    }

    private static String toHex(int i) {
        Deque<String> stack = new LinkedList<>();
        
        int purpose = i;
        // 将指定的purpose转换为十六进制
        // 同时压入stack中
        while (purpose % 16 != purpose) {
            int res = purpose % 16;
            String str = "";
            if (res < 10) {
                str = String.valueOf(res);
            } else if (res >= 10 && res < 16) {
                switch (res) {
                    case 10:
                        str = "a";
                        break;
                    case 11:
                        str = "b";
                        break;
                    case 12:
                        str = "c";
                        break;
                    case 13:
                        str = "d";
                        break;
                    case 14:
                        str = "e";
                        break;
                    case 15:
                        str = "f";
                    default:
                        break;
                }
            }
            
            stack.push(str);
            purpose /= 16;
        }
        stack.push(String.valueOf(purpose));

        String target  = "";
        // for (String string : stack) {
        //     target += string;
        // }
        
        // Stack中的元素依次出栈存入target
        while (!stack.isEmpty()) {
            target += stack.pop();
        }

        return target;
    }
}


/**
 * result:
 * 
 * 30d4
 * Passed
 */

```

