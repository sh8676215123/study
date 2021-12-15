### 考虑用静态工厂方法代替构造器

> *类可以提供一个公有的静态工厂方法(static factory method)，它只是返回类的实例的静态方法*

```java
//传统new实例化
Map<String, String> map1 = new HashMap<>();
//静态工厂方法实例化
Map<String, String> map2 = Maps.newHashMap();
```

- 静态工厂方法与构造器相比-优势

  - **有名称**

    > ​	如果构造器的参数本身没有确切地描述正被返回的对象，那么具有适当名称的静态工厂方法更容易使用，产生的客户端代码更易于阅读

    ```java
    //用此构造方法返回可能为素数
    BigInteger bigInteger = new BigInteger(10, 2, new Random());
    //由该静态工厂方法名字可知返回值可能为素数
    BigInteger probablePrime = BigInteger.probablePrime(10, new Random());
    ```

  - **可以进行对象实例缓存，不必在每次调用他们的时候都创建一个新对象**

    > 使得不可变类可预先构造好实例，进行实例缓存，从而避免创建不必要的重复对象，减少系统开销继而提升性能.常见的应用如单例类、Integer.valueOf(int)、Boolean.valueOf(boolean)等

    ```java
    //已预先对new Boolean(true)和new Boolean(false)两个实例对象做了缓存
    Boolean value = Boolean.valueOf(true);
    //已预先对数值范围在-128到127之间的实例做了缓存
    Integer integer = Integer.valueOf(2);
    ```

  - **可以返回原返回类型的任何子类型的对象(目前不理解)**

- 静态工厂方法与构造器相比-劣势

  - **类如果不含公有的或者受保护的构造器，就不能被子类化(目前不理解)**

  - **它们与其他的静态方法没有区别**

    > 与我们传统的用构造器实例化对象的习惯不同，对于提供了静态工厂方法而不是构造器的类来说，要想查明如何实例化一个类，这是非常困难的。可以用标准的命名习惯来弥补这一劣势

- 静态工厂方法的惯用名称
  - valueof()
    - 例：Integer.valueOf(int),String.valueOf()
  - of()
    - 例：List.of(E1,E2,...)
  - newInstance()
    - 单例类常用命名方式
  - newType()
    - Maps.newHashMap()

### 消除过期对象的引用

> ​	`过期引用`指永远不会被解除的引用，如果一个没用的对象的引用被无意识的保留起来了，那么垃圾回收机制不会处理这个对象，同时也不会处理被这个对象引用的其他对象，继而可能造成`内存泄露`，对性能存在潜在的重大影响

- ​	引子

  > ​	以下是一个简单的栈实现，包含的入栈方法`push(object)`和出栈方法`pop()`

  ```java
  package com.example.springboot.demo;
  
  import java.util.Arrays;
  import java.util.EmptyStackException;
  
  /**
   * @author shihan
   * @description 测试栈
   * @date 2021/11/30 16:35
   */
  public class DemoStack {
      private Object[] elements;
      private int size = 0;
      private final static int DEFAULT_INITIAL_CAPACITY = 10;
  
      public DemoStack() {
          elements = new Object[DEFAULT_INITIAL_CAPACITY];
      }
  
      public void push(Object element) {
          this.ensureCapacity();
          elements[size++] = element;
      }
  
      public Object pop() {
          if (size == 0) {
              throw new EmptyStackException();
          }
          return elements[--size];
      }
  
      /**
       * ensureCapacity
       *
       * @description 当数组大小达到上限时，将数组大小扩容为原先的2倍
       * @param
       * @return
       * @author shihan
       * @date 2021/11/30 16:40
       * @version 1.0.0
       */
      private void ensureCapacity() {
          if (size == elements.length) {
              elements = Arrays.copyOf(elements, size * 2 + 1);
          }
      }
  }
  
  ```

- ​	发现问题

  > 当对栈先进行入栈，再进行出栈，那么从栈中弹出来的对象不会被垃圾回收器回收，即使使用栈的程序不再引用这些对象，它们也不会被回收。这是因为栈内部维护着对这些对象的`过期引用`，则会有造成内存泄露的风险
  >
  > 如先入栈调用push方法，再出栈调用pop方法，此时下标为0的对象elements[0]已为无用对象，但不会被垃圾回收器回收，因为栈内部维护着对elements[0]的引用，为`过期引用`

- 解决问题

  > 消除掉过期对象的引用即可，即在pop方法中弹出对象的同时释放掉相应内存
  >
  > ```java
  > public Object pop() {
  >         if (size == 0) {
  >             throw new EmptyStackException();
  >         }
  >         Object popObj = elements[--size];
  >         elements[size] = null;//释放无效引用内存
  >         return popObj;
  > }
  > ```

- 何时该手动清空引用？

> 注：*清空对象引用应该是一种例外，而不是一种规范行为*
>
> 对于每个对象的引用，一旦程序不再使用它，就把它清空，这种做法没用必要

> ​	对于`DemoStack`类自己管理内存，其元素数据存储在`elements`数组内，其中有些元素是有效引用，有些元素是无效引用，但是垃圾回收器并不知道这一点，此时对于无效引用的元素，就需要手动清空这些无效引用
>
> *结论：只要类是自己管理内存，就应该警惕内存泄漏的问题，一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空*
