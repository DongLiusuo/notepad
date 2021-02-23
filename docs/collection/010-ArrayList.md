

# 1. ArrayList集合底层数据结构

1. **ArrayList集合介绍**

   `List`接口的可调整大小的数组实现

   数组:一旦初始化长度就不可以发生改变

2. **数据结构介绍**

   * 增删慢：每次删除元素，都需要更改数组长度、拷贝以及移动元素位置。
   * 查询快：由于数组在内存中是一块连续空间，因此可以根据地址+索引的方式快速获取对应位置上的元素。

# 2. ArrayList继承关系

## 2.1 Serializable标记性接口

1. **序列化和反序列化**

   类的序列化由实现java.io.Serializable接口的类启用。 不实现此接口的类将不会使任何状态序列化或反
   序列化。 可序列化类的所有子类型都是可序列化的。 序列化接口没有方法或字段，仅用于标识可串行化的语义。

   **序列化**:将对象的数据写入到文件(写对象)

   **反序列化**:将文件中对象的数据读取出来(读对象)

2. **Serializable源码介绍**

   ```java
   public interface Serializable {}
   ```
   
3. **案例**

   ```java
   import java.io.Serializable;
   import java.util.StringJoiner;
   
   public class Student implements Serializable {
       private static final long serialVersionUID = 1014100089306623762L;
       //姓名
       private String name;
       //年龄
       private Integer age;
       public Student() {
       }
       public Student(String name, Integer age) {
           this.name = name;
           this.age = age;
       }
       public String getName() {
           return name;
       }
       public void setName(String name) {
           this.name = name;
       }
       public Integer getAge() {
           return age;
       }
       public void setAge(Integer age) {
           this.age = age;
       }
   
       @Override
       public String toString() {
           return new StringJoiner(", ", Student.class.getSimpleName() + "[", "]")
                   .add("name='" + name + "'")
                   .add("age=" + age)
                   .toString();
       }
   
   }
   
   
   import java.io.FileInputStream;
   import java.io.FileOutputStream;
   import java.io.ObjectInputStream;
   import java.io.ObjectOutputStream;
   import java.util.ArrayList;
   
   public class SeriDemo {
   
       public static void main(String[] args) throws Exception {
           Student s = new Student();
           System.out.println(s);
           //创建对象操作流 --> 序列化
           ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("obj.txt"));
           //创建集合,且添加学生对象
           ArrayList<Student> list = new ArrayList<Student>();
           list.add(new Student("悔创阿里杰克马", 51));
           list.add(new Student("会点一点长几颗", 26));
           list.add(new Student("容颜老去蒋青青", 32));
           list.add(new Student("将里最丑刘一飞", 27));
           //将集合写入到文件
           oos.writeObject(list);
           //创建对象输入流 --> 反序列化
           ObjectInputStream ois = new ObjectInputStream(new FileInputStream("obj.txt"));
           //读取数据
           Object o = ois.readObject();
           //向下转型
           ArrayList<Student> al = (ArrayList<Student>) o;
           //遍历集合
           for (int i = 0; i < al.size(); i++) {
               //根据索引取出集合的每一个元素
               Student stu = al.get(i);
               System.out.println(stu);
           }
       }
   
   }
   
   
   ```

## 2.2 Cloneable 标记性接口

1. 介绍 

  一个类实现Cloneable 接口来指示Object.clone() 方法，该方法对于该类的实例进行字段的复制是合法的。在不实现Cloneable 接口的实例上调用对象的克隆方法会导致异常CloneNotSupportedException 被抛出。简言之:**克隆就是依据已经有的数据，创造一份新的完全一样的数据拷贝**

2. Cloneable源码介绍

   ```java
   public interface Cloneable {}
   ```

3. 克隆的前提条件

   * 被克隆对象所在的类必须实现Cloneable 接口
   * 必须重写clone 方法

4. clone的基本使用

   ```java
   import java.util.ArrayList;
   
   public class CloneDemo {
       public static void main(String[] args) {
           ArrayList<String> list = new ArrayList<String>();
           list.add("人生就是旅途");
           list.add("也许终点和起点会重合");
           list.add("但是一开始就站在起点等待终点");
           list.add("那么其中就没有美丽的沿途风景和令人难忘的过往");
           //调用方法进行克隆
           Object o = list.clone();
           System.out.println(o == list);
           System.out.println(o);
           System.out.println(list);
       }
   }
   ```

5. clone源码分析

   ```java
   // ArrayList类中的clone方法
   public Object clone() {
           try {
               ArrayList<?> v = (ArrayList<?>) super.clone();
               v.elementData = Arrays.copyOf(elementData, size);
               v.modCount = 0;
               return v;
           } catch (CloneNotSupportedException e) {
               // this shouldn't happen, since we are Cloneable
               throw new InternalError(e);
           }
       }
   ```

6. 案例

   案例:已知A 对象的姓名为豹子头林冲，年龄30 。由于项目特殊要求需要将该对象的数据复制另外一个对象
   B 中，并且此后A 和B 两个对象的数据不会相互影响

   * 传统方式
        ```java
        new两个对象，然后手动set值
       此时无论修改那个对象的值，另一个都不会受到影响
       ```
   
   * 克隆方式
   
     * 浅拷贝
     
       1. 重写父类的clone方法（简单的调用父类的clone方法即可），然后访问修饰符修改为public
     
       2. 此时修改其中一个对象的属性值，如果是基本数据类型和他们的包装类型，另一个对象的属性值不会收到影响
     
       3. **浅拷贝的局限性**：基本数据类型可以达到完全复制，引用数据类型则不可以
     
          **原因**：引用数据类型仅仅是拷贝了一份引用
     
     * 深拷贝
     
       1. 深拷贝，不能简单的调用父类的clone方法
       2. 先调用父类的方法，然后向下转型
       3. 然后调用引用类型的属性的clone方法，对属性进行克隆，然后向下转型
       4. 将克隆出来的属性，赋值到2返回的对象
       5. 此时无论修改那个对象的值，另一个都不会受到影响，包括引用类型的属性
     
       
   

## 2.3 RandomAccess标记接口

1. 介绍

   标记接口由`List `实现使用，以**表明它们支持快速（通常为恒定时间）随机访问**。此接口的主要目的是允许通用算法更改其行为，以便在应用于**随机访问列表**或**顺序访问列表**时提供良好的性能。

   用于操纵随机访问列表的最佳算法（例如`ArrayList` ）可以在应用于顺序访问列表时产生二次行为（如
   `LinkedList `）。 鼓励通用列表算法在应用如果将其应用于顺序访问列表之前提供较差性能的算法时，检查
   给定列表是否为`instanceof` ，并在必要时更改其行为以保证可接受的性能。

   人们认识到，随机访问和顺序访问之间的区别通常是模糊的。 例如，一些`List` 实现提供渐近的线性访问时
   间，如果它们在实践中获得巨大但是恒定的访问时间。 这样的一个`List` 实现应该通常实现这个接口。 根据经验，` List` 实现应实现此接口，如果对于类的典型实例，此循环：

   ```java
   // 随机访问
   for (int i=0, n=list.size(); i < n; i++)
       list.get(i);
   ```

   比这个循环运行得更快：

   ```java
   // 顺序访问
   for (Iterator i=list.iterator(); i.hasNext(); )
        i.next();
   ```

2. 案例1：ArrayList

   ```java
   		//创建ArrayList集合
           List<String> list = new ArrayList<>();
           //添加1000 0000条数据
           for (int i = 0; i < 10000000; i++) {
               list.add(i + "a");
           }
           System.out.println("----通过索引(随机访问:)----");
           long startTime = System.currentTimeMillis();
           for (int i = 0; i < list.size(); i++) {
               //仅仅为了演示取出数据的时间,因此不对取出的数据进行打印
               list.get(i);
           }
           long endTime = System.currentTimeMillis();
           System.out.println("随机访问: " + (endTime - startTime));
           System.out.println("----通过迭代器(顺序访问:)----");
           startTime = System.currentTimeMillis();
           Iterator<String> it = list.iterator();
           while (it.hasNext()) {
               //仅仅为了演示取出数据的时间,因此不对取出的数据进行打印
               it.next();
           }
           endTime = System.currentTimeMillis();
           System.out.println("顺序访问: " + (endTime - startTime));
   控制台效果
   ----通过索引(随机访问:)----
   随机访问: 5
   ----通过迭代器(顺序访问:)----
   顺序访问: 6
   ```

3. 案例2：LinkedList

   ```java
   将上述代码第2行修改为 List<String> list = new LinkedList<>();
   控制台效果：随机访问及其慢，顺序访问比随机访问快
   ```

   为什么LinkedList随机访问比顺序访问要慢这么多?

   由于随机访问的时候源码底层每次都需要进行折半的动作，再经过判断是从头还是从尾部一个个寻找。
   而顺序访问只会在获取迭代器的时候进行一次折半的动作,以后每次都是在上一次的基础上获取下一个元素。
   因此顺序访问要比随机访问快得多

   **源码分析**

   随机访问

   ```java
   //每次LinkedList对象调用get方法获取元素,都会执行以下代码
   list.get(i);
   
   public class LinkedList<E> {
   
           public E get(int index) {
               //检验是否有效
               checkElementIndex(index);
               //调用node(index)
               return node(index).item;
           }
   
           //node方法
           Node<E> node(int index) {
               //node方法每次被调用的时候都会根据集合size进行折半动作
               //判断get方法中的index是小于集合长度的一半还是大于
               if (index < (size >> 1)) {
                   //如果小于就从链表的头部一个个的往后找
                   Node<E> x = first;
                   for (int i = 0; i < index; i++)
                       x = x.next;
                   return x;
               } else {
                   Node<E> x = last;
                   //如果大于就从链表的尾部一个个的往前找
                   for (int i = size - 1; i > index; i--)
                       x = x.prev;
                   return x;
               }
           }
   
       }
   ```

   顺序访问

   ```java
   
       //获取迭代器的时候,会执行以下代码  ListIterator是Iterator的子接口
       Iterator<String> it = list.iterator();
   
       //AbstractList为LinkedList父类的父类
       public abstract class AbstractList<E> {
   
           public ListIterator<E> listIterator() {
           //返回一个列表迭代器,且指定参数为0
               return listIterator(0);
           }
   
       }
   
       public class LinkedList<E> {
   
           public ListIterator<E> listIterator(int index) {
               //检查索引位置
               checkPositionIndex(index);
               //返回ListItr对象
               return new ListItr(index);
           }
   
           //LinkedList迭代器实现类
           private class ListItr implements ListIterator<E> {
   
               private Node<E> lastReturned;
   
               private Node<E> next;
   
               private int nextIndex;
   
               //将实际修改集合次数赋值给预期修改次数
               private int expectedModCount = modCount;
   
               ListItr(int index) {
                   //判断 0 == size,实际上就是调用 node(index)方法
                   next = (index == size) ? null : node(index);
                   //将index的值赋值给 nextIndex,便于下次查找
                   nextIndex = index;
               }
   
           }
   
       }
   
       Node<E> node(int index) {
           //在获取迭代器的时候也会进行折半的动作
           //但是在获取迭代器的时候 index 一定是0,因此 if的条件成立
           if (index < (size >> 1)) {
               Node<E> x = first;
               //由于循环条件不成立,不会执行 x.next;
               for (int i = 0; i < index; i++)
                   x = x.next;
               return x; //返回第一个元素
           } else {
               Node<E> x = last;
               for (int i = size - 1; i > index; i--)
                   x = x.prev;
               return x;
           }
       }
   
   }
   
   //迭代器调用 hasNext()方法的时候,会执行以下代码
       private class ListItr implements ListIterator<E> {
           public boolean hasNext() {
               //如果nextIndex < 集合的长度,就说明还有元素,可以进行next
               return nextIndex < size;
           }
       }
   
   //当迭代器调用it.next();方法的时候会执行以下代码
   it.next();
   public class LinkedList<E>{
       private class ListItr implements ListIterator<E> {
           public E next() {
               checkForComodification(); //检查集合实际修改次数和预期次数是否一样
               //再次判断是否有元素
               if (!hasNext())
                   throw new NoSuchElementException();
               //将链表的第一个元素赋值给lastReturned
               lastReturned = next;
               //将下一个元素赋值给next
               next = next.next;
               //nextIndex++
               nextIndex++;
               //返回第一个元素
               return lastReturned.item;
           }
       }
   }
   ```

4. 实际开发场景

   假设从数据库中查询``select * from user;``，用Java的List接收

   那么在遍历集合的结果集之前面临一个问题,使用普通for遍历好 还是使用迭代器(增强for)?

   特别是数据量特别大的时候一定要考虑!

   对返回的集合进行判断,如果返回的集合实现了 RandomAccess 就使用 普通for

   否则使用迭代器(增强for)

   ```java
   if(list instanceof RandomAccess){
       for (int i = 0; i < list.size(); i++) {
           System.out.println(list.get(i));
       }
   }else {
       for (Obj obj : list) {
       	System.out.println(obj);
       }
   }
   ```




## 2.4 AbstractList抽象类

# 3. ArrayList源码分析

## 3.1 构造方法

![1614099125089](010-ArrayList/1614099125089.png)

## 3.2 源码分析

### 空参构造

```java
public class ArrayList<E> {
    //默认空容量的数组,长度为0
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //集合真正存储数据的容器
    transient Object[] elementData;
    //空参构造
    public ArrayList() {
        //赋值
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
}
```

结论：通过空参构造方法创建集合对象并未构造一个初始容量为十的空列表，仅仅将
DEFAULTCAPACITY_EMPTY_ELEMENTDATA 的地址赋值给elementData

### 有参构造1

```java
public class ArrayList<E> {
    private static final Object[] EMPTY_ELEMENTDATA = {};
    //指定容量的构造方法
    public ArrayList(int initialCapacity) {
        //判断容量是否大于0
        if (initialCapacity > 0) {
            //根据构造方法的参数创建指定长度的数据
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //将空数组的地址赋值给elementData
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //制造出异常
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        }
    }
}
```

### 有参构造2

```java

public class ArrayList<E> {
    //长度为0的空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //集合存元素的数组
    Object[] elementData;

    //集合的长度
    private int size;

    public ArrayList(Collection<? extends E> c) {
        //将构造方法中的参数转成数组
        elementData = c.toArray();

        if ((size = elementData.length) != 0) {
            // 再次进行判断
            if (elementData.getClass() != Object[].class)
                // 数组的创建和拷贝
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 就把空数组的地址赋值给集合存元素的数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    //将集合转数组的方法
    public Object[] toArray() {
        //调用数组工具类的方法
        return Arrays.copyOf(elementData, size);
    }
}

class Arrays {
    public static <T> T[] copyOf(T[] original, int newLength) {
        //再次调用方法得到一个数组
        return (T[]) copyOf(original, newLength, original.getClass());
    }

    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        //不管三元运算符的结果如何,都会创建一个新的数组
        //新数组的长度一定是和集合的size一样
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //数组的拷贝
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        //返回新数组
        return copy;
    }
}
```

