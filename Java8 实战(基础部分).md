# 通过行为参数化传递代码

# Lambda表达式

## 组成

+ 参数列表
+ 箭头：把参数列表与Lambda主体分隔开
+ Lambda主题

## 函数式接口

```
函数式接口就是只定义一个抽象方法的接口

哪怕有很多默认方法，只要接口只定义了一个抽象方法，它就仍然是一个函数式接口。
```

## 例子

### 假设一种需求：

如果需要读取的内容是一直变化的，该如何处理？

```java
public static String processFile() throws IOException{
    try(BufferedReader br= new BufferedReader(new FileReader("data.txt"))) {
		return br.readline();
    }
}
```

### 解决方法：

+ 把`processFile`的行为参数化：把行为传递给`processFile`，以便它可以利用`BufferedReader执`行不同的行为。

+ 创建一个能匹配`BufferedReader-> String`，还可以抛出`IOException`异常的接口

  ```java
  @FunctionalInterface
  public interface BufferedReaderProcessor{
  	String process(BufferedReader b) throws IOException;
  }
  ```

+ 将这个接口作为新的`processFile`方法的参数

  ```java
  public static String processFile(BufferedReaderProcessor p) trows IOException{
      ....
  }
  ```

+ 在`processFile`主体内，对得到的`BufferedReaderProcessor`对象调用`process`方法执行处理

  ```java
  public static String processFile(BufferedReaderProcessor p) throws IOException{
      try (BufferedReader br=new BufferedReader(new FileReader("data.txt"))) {
          return p.process(br);
      }
  } 
  ```

+ 现在就可以通过传递不同的Lambda表达式重用`processFile`方法，实现以不同方式处理文件

  ```java
  // 处理一行
  String oneLine = process( (BufferedReader br) -> br.readLine() );
  ```

  ```java
  // 处理2行
  String oneLine = process( (BufferedReader br) -> br.readLine() + br.readLine() );
  ```

  

## 使用局部变量

```java
Lambda可以没有限制地在其主体中引用实例变量和静态变量，但局部变量必须显式声明为final
```

+ 实例变量和局部变量背后的实现有一个关键不同：

  ```java
  实例变量都存储在堆中，而局部变量则保存在栈上。如果Lambda可以直接访问局部变量，而且Lambda是在一个线程中使用的，则使用Lambda的线程，可能会在分配该变量的线程将这个变量收回之后，去访问该变量。因此，Java在访问自由局部变量时，实际上是在访问它的副本，而不是访问原始变量。如果局部变量仅仅赋值一次那就没有什么区别了——因此就有了这个限制。
  ```

## 方法引用

```java
使用方法引用时，目标引用放在分隔符：：前，方法的名称放在后面。
    
例如，Apple::getWeight就是引用了Apple类中定义的方法getWeight。请记住，不需要括号
    上述就是Lambda表达式(Apple a)-> a.getWeight()的快捷写法
```

### 如何构建方法引用

+ 指向静态方法的方法引用

  ```java
  例如Integer的parseInt方法，写作Integer::parseInt
  ```

+ 指向任意类型实例方法的方法引用

  ```java
  例如String的length方法，写作String::length
      
   就是你在引用一个对象的方法，而这个对象本身是Lambda的一个参数。例如，Lambda表达式(String s)-> s.toUppeCase()可以写作String::toUpperCase
  ```

+ 指向现有对象的实例方法的方法引用

  ```java
  就是你在引用一个对象的方法，而这个对象本身是Lambda的一个参数。例如，Lambda表达式(String s)-> s.toUppeCase()可以写作String::toUpperCase
      
    就是你在引用一个对象的方法，而这个对象本身是Lambda的一个参数。例如，Lambda表达式(String s)-> s.toUppeCase()可以写作String::toUpperCase
  ```

### 构造函数引用

```java
	/*       无参构造   */

        // 构造函数引用指向默认的 Student() 构造函数
        Supplier<Student> supplier= Student::new;

        // 上面等价于
        Supplier<Student> sp2 = () -> new Student();

        // 调用supplier的get方法将产生一个新Student
        Student s = supplier.get();
        System.out.println(s);

        /*  有一个参数的构造方法  */

        // 对于有参构造方法，这种参数写法固定死了
        Supplier<Student> sp3 = () -> new Student("Group");
        Student student2 = sp3.get();
        System.out.println(student2);

        // 指向Student（String name） 的构造函数引用
        Function<String, Student> function = Student::new;
        // 调用该Function函数的apply方法产生一个Student
        Student student3 = function.apply("lisi");
        System.out.println(student3);

        /*   有两个参数的构造方法  */
        BiFunction<String,Integer,Student> biFunction = Student::new;
        Student s3 = biFunction.apply("lisi", 20);
        System.out.println(s3);
```

#### 将三个及以上参数的构造函数转变为构造函数引用

```java
//自己创建一个与构造函数引用的签名匹配的函数式接口

public interface TriFunction<T, U, V, R> {

    R apply(T t, U u, V v);
}
```

```java
TriFunction<String, Integer, String, Student> TriFactory = Student::new;
Student triStudent = TriFactory.apply("zhangsan", 18, "名古屋");
System.out.println(triStudent);
```

##  Lambda和方法引用实战

终极目标为

```java
inventory.sort(comparing(Apple::getWeight));
```

+ 传递代码

  ```java
  Java 8的API已经为你提供了一个List可用的sort方法
  
  void sort(Comparator<? super E> c) // 它需要一个Comparator对象来比较两个Apple！
      
      // 在Java中传递策略的方式：它们必须包裹在一个对象里。将sort的行为被参数化了：传递给它的排序策略不同，其行为也会不同。
  ```

  ```java
  最基本方法：
  public class AppleComparator implements Comparator<Apple> 
  {	public int compare(Apple al, Apple a2){
  		return al.getWeight().compareTo(a2.getWeight());
  }
   
  inventory.sort(new AppleComparator 0) :
  ```

+ 使用匿名类

  ```java
  inventory.sort(new Comparator<Apple>()
  	public int compare(Apple al, Apple a2){
  		return al.getWeight ().compareTo(a2. getWeight ());
  	}
  });
  ```

+ 使用Lambda表达式

  ```java
  inventory.sort((Apple al, Apple a2)-> al.getWeight 0.compareTo(a2. getWeight()));
  
  // Java编译器可以根据Lambda出现的上下文来推断Lambda表达式参数的类型
  inventory.sort((al, a2)-> al.getWeight().compareTo(a2. getWeight()));
  ```

  `Comparator具有一个叫作comparing的静态辅助方法，它可以接受一个Function来提取Comparable键值，并生成一个Comparator对象`

  ```java
  Comparator<Apple> c = Comparator.comparing((Apple a)-> a.getWeight();
  ```

  ```java
  // 进一步简化
  import static java.util.Comparator.comparing;
  
  inventory.sort(comparing((a)-> a. getWeight()));
  ```

+ 使用方法引用

  ```java
  inventory.sort(comparing(Apple::getWeight));
  ```

## 复合Lambda表达式的有用方法

### 比较器复合



### 谓词复合

```
谓词接口包括三个方法：negate、and和or
```

