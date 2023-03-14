# Stream

## 定义

`从支持数据处理操作的源生成的元素序列`

## 流与集合

+ 和迭代器类似，流只能遍历一次

  ```
  遍历完之后，我们就说这个流已经被消费掉了。你可以从原始数据源那里再获得一个新的流来重新遍历一遍
  ```

## 流操作

​		两类操作：

+ ❑ filter、map和limit可以连成一条流水线；

+ ❑ collect触发流水线执行并关闭它。

可以连接起来的流操作称为中间操作，关闭流的操作称为终端操作

操作实例：

```java
/**
 *@author: xiaolanhe
 *@createDate: 2023/3/14 9:58
 */
public class Dish {
    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;


    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name=name;
        this.vegetarian=vegetarian;
        this.calories=calories;
        this.type=type;
    }

........

    @Override
    public String toString() {
        return name;
    }
    public enum Type { MEAT, FISH, OTHER }
}
```

```java
List<Dish> menu= Arrays.asList(
            new Dish("pork", false, 800, Dish.Type.MEAT),
            new Dish("beef", false, 700, Dish.Type.MEAT),
            new Dish("chicken", false, 400, Dish.Type.MEAT),
            new Dish("french fries", true, 530, Dish.Type.OTHER),
            new Dish("rice", true, 350, Dish.Type.OTHER),
            new Dish("season fruit", true, 120, Dish.Type.OTHER),
            new Dish("pizza", true, 550, Dish.Type.OTHER),
            new Dish("prawns", false, 300, Dish.Type.FISH),
            new Dish("salmon", false, 450, Dish.Type.FISH) );
```

### 中间操作

```
诸如filter或sorted等中间操作会返回另一个流。这让多个操作可以连接起来形成一个查询
```

```java
List<String> names =
                menu.stream()
                    .filter(d -> {
                        System.out.println("filtering：" + d.getName());
                        return d.getCalories() > 300;
                    })
                .map(d -> {
                    System.out.println("mapping " + d.getName());
                    return d.getName();
                })
                .limit(3)
                .collect(toList());
        System.out.println(names);
        
        // output

        filtering pork
        mapping pork
        filtering beef
        mapping beef
        filtering chicken
        mapping chicken
        [pork, beef, chicken]
```

### 终端操作

```java
终端操作会从流的流水线生成结果。其结果是任何不是流的值，比如List、Integer，甚至void。
```

## 使用Stream

### 筛选与切片

+ filter

  ```java
  该操作会接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流
  ```

+ distinct

  ```
  去重
  ```

+ limit

  ```
  该方法会返回一个不超过给定长度的流。所需的长度作为参数传递给limit。
  ```

+ skip

  ```java
  返回一个扔掉了前n个元素的流。如果流中元素不足n个，则返回一个空流。请注意，limit(n)和skip(n)是互补的！
  ```

### 映射

+ 对流中每一个元素应用函数 	map()

  ```
  流支持map方法，它会接受一个函数作为参数。这个函数会被应用到每个元素上，并将其映射成一个新的元素
  ```

  ```java
          List<String> dishNames=menu.stream()
                                        .map(Dish::getName)
                                        .collect(toList());
  // 因为getName方法返回一个String，所以map方法输出的流的类型就是Stream<String>。
  ```

  

+ 流的扁平化  flatMap()

  ```
  flatmap方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流
  ```

  给定两个数字列表，如何返回所有的数对呢？例如，给定列表[1, 2, 3]和列表[3, 4]，应该返回[(1, 3), (1, 4), (2, 3), (2, 4), (3, 3), (3, 4)]。为简单起见，你可以用有两个元素的数组来代表数对。

  ```java
  List<Integer> words=Arrays.asList(1, 2, 3);
          List<Integer> words2=Arrays.asList(4,5);
          List<int[]> pairs = words.stream()
                  .flatMap(i -> words2.stream()
                          .map(j -> new int[]{i,j})
                  )
                  .collect(toList());
  ```

### 查找和匹配

`allMatch、anyMatch、noneMatch、findFirst和findAny(返回当前流中的任意元素)`

### 归约（将流归约成一个值）

```
需要将流中所有元素反复结合起来，得到一个值，比如一个Integer。这样的查询可以被归类为归约操作（将流归约成一个值）
```

​		把一个流中的元素组合起来，使用`reduce操作`来表达更复杂的查询

```java
reduce 接受两个参数

	一个初始值
	
	一个BinaryOperator<T>来将两个元素结合起来产生一个新值
    
 reduce还有一个重载的变体，它不接受初始值，但是会返回一个Optional对象：
```

+ 元素求和

  ```java
   int sum=numbers.stream().reduce(0, (a, b)-> a+b);
  ```

+ ```java
          Optional<Integer> sum=numbers.stream().reduce((a, b)-> (a+b));
          
          为什么它返回一个Optional<Integer>呢？考虑流中没有任何元素的情况。reduce操作无法返回其和，因为它没有初始值。这就是为什么结果被包裹在一个Optional对象里，以表明和可能不存在。
  ```

+ 最大值/最小值

  ```java
          Optional<Integer> max=numbers.stream().reduce(Integer::max);
          
          Optional<Integer> min=numbers.stream().reduce(Integer::min);
  ```


## 数值流

### 原始类型流特化

```java
	三个原始类型特化流接口 IntStream、DoubleStream和LongStream，分别将流中的元素特化为int、long和double，从而避免了暗含的装箱成本
```

+ 映射到数值流

  ```java
  	将流转换为特化版本的常用方法是mapToInt、mapToDouble和mapToLong。这些方法和前面说的map方法的工作方式一样，只是它们返回的是一个特化流，而不是Stream<T>
          
       mapToInt会返回一个IntStream（而不是一个Stream<Integer>）
  ```

+ 转换回对象流

  ```java
  一旦有了数值流，你可能会想把它转换回非特化流
  
  要把原始流转换成一般流（每个int都会装箱成一个Integer），可以使用boxed方法
      
      IntStream intStream = menu.stream().mapToInt(Dish::getCalories);
  	Stream<Integer>stream=intStream.boxed();
  ```

+ 默认值OptionalInt

  ```java
  Optional类是一个可以表示值存在或不存在的容器。Optional可以用Integer、String等参考类型来参数化。
      
      
      对于三种原始流特化，也分别有一个Optional原始类型特化版本：OptionalInt、OptionalDouble和OptionalLong。
  ```

  ```java
  例如，要找到IntStream中的最大元素，可以调用max方法，它会返回一个OptionalInt：
  
   OptionalInt maxCalories=menu.stream()
                  .mapToInt(Dish::getCalories)
                  .max();
  
  如果没有最大值的话，你就可以显式处理OptionalInt去定义一个默认值了：
      int max = maxCalories.orElse(1); // 如果没有最大值，显示提供一个默认的最大值
  ```

## 构建流

### 由值创建流

```
使用静态方法Stream.of，通过显式值创建一个流。它可以接受任意数量的参数
```

```java
        Stream<String> stream=Stream.of("Java 8 ", "Lambdas ", "In ", "Action");
        stream.map(String::toUpperCase).forEach(System.out::println);


// 使用empty得到一个空流
        Stream<String> emptyStream=Stream.empty();
```

### 由数组创建流

```
使用静态方法Arrays.stream从数组创建一个流
```

### 由文件创建流

```java
	java.nio.file.Files中的很多静态方法都会返回一个流。例如，一个很有用的方法是Files.lines，它会返回一个由指定文件中的各行构成的字符串流
```

```java
// 统计一个文件中有多少不同的词

long uniqueWords = 0;
        try(Stream<String> lines = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())){
            uniqueWords = lines.flatMap(line -> Arrays.stream(line.split(" ")))
                    .distinct()
                    .count();
        }catch (IOException e){

        }
```

### 由函数生成流：创建无限流

```java
Stream API提供了两个静态方法来从函数生成流：Stream.iterate和Stream.generate
```

#### 迭代

```java
	iterate方法接受一个初始值（在这里是0），还有一个依次应用在每个产生的新值上的Lambda（UnaryOperator<t>类型）。
       
     使用limit方法来显式限制流的大小
```

```java
        Stream.iterate(0, n-> n+2)
              .limit(10)
              .forEach(System.out::println);
```

+ 斐波纳契元组序列与此类似，是数列中数字和其后续数字组成的元组构成的序列：(0, 1), (1, 1), (1, 2), (2, 3), (3, 5), (5, 8), (8, 13), (13, 21) …

  ```java
            Stream.iterate(new int[]{0, 1},
                            t-> new int[]{t[1], t[0]+t[1]})
                  .limit(20)
                  .forEach(t-> System.out.println("("+t[0]+", "+t[1]+")"));
  ```

#### 生成

```java
它接受一个Supplier<T>类型的Lambda提供新的值
```

eg:

```java
        Stream.generate(Math::random)
              .limit(5)
              .forEach(System.out::println);
```

