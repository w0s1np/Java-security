# java反射机制

## 基本概念

Java反射机制是在运行状态时，对于任意一个类，都能够获取到这个类的所有属性和方法，对于任意一个对象，都能够调用它的任意一个方法和属性(包括私有的方法和属性)，这种动态获取的信息以及动态调用对象的方法的功能就称为 java 语言的反射机制。

## 使用方法

在 java 中又是如何实现反射的呢，这就涉及到 java 中的几个类，Class、Constructor、Method、Field，

Class 类抽象出了 java 中类的特征并提供了一些方法，Constructor类 抽象出了 java 中所有的构造函数的特征以及提供一些方法

### 访问字段

对任意的一个`Object`实例，只要我们获取了它的`Class`，就可以获取它的一切信息。

`Class`类提供了以下几个方法来获取字段：

- Field getField(name)：根据字段名获取某个 public 的 field（包括父类）
- Field getDeclaredField(name)：根据字段名获取当前类的某个 field（不包括父类）
- Field[] getFields()：获取所有 public 的 field（包括父类）
- Field[] getDeclaredFields()：获取当前类的所有 field（不包括父类）

例如：

```java
package Lnh;

public class Lnh {

    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public字段"score":
        System.out.println(stdClass.getField("score"));
        // 获取继承的public字段"name":
        System.out.println(stdClass.getField("name"));
        // 获取private字段"grade":
        System.out.println(stdClass.getDeclaredField("grade"));
    }
}

class Student extends Person {
    public int score;
    private int grade;
}

class Person {
    public String name;
}
```

上述代码首先获取`Student`的`Class`实例，然后，分别获取`public`字段、继承的`public`字段以及`private`字段，打印出的`Field`类似：

```java
public int Lnh.Student.score
public java.lang.String Lnh.Person.name
private int Lnh.Student.grade
```


一个`Field`对象包含了一个字段的所有信息：

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

What is a field in java？

```
A field is an attribute. A field may be a class's variable, an object's variable, an object's method's variable, or a parameter of a function.
field 是一种属性。一个field可以是一个类的变量，一个对象的变量，一个对象的方法变量，或者一个函数的参数。
```

这个有什么用呢，其实这里可以获取到一个对象实例化对应该字段的值，比如：

```java
package Lnh;

import java.lang.reflect.Field;

public class Lnh {

    public static void main(String[] args) throws Exception {
        Object p = new Person("w0s1np");
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        Object value = f.get(p);
        System.out.println(value);
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }
}
```

他会抛出一个`IllegalAccessException`，这是因为`name`被定义为一个`private`字段，正常情况下，`Main`类无法访问`Person`类的`private`字段。

要修复错误，可以将`private`改为`public`，或者，在调用`Object value = f.get(p);`前，先写一句：

```java
f.setAccessible(true);
```

![1](./img/227.png)

这样就能获取到一个不知道目标实例任何信息的情况下，获取特定字段的值。这是不是就存在一个隐藏的危害呢，调用到了不允许访问的属性。

但是`setAccessible(true)`可能会失败。如果JVM运行期存在`SecurityManager`，那么它会根据规则进行检查，有可能阻止`setAccessible(true)`。例如，某个`SecurityManager`可能不允许对`java`和`javax`开头的`package`的类调用`setAccessible(true)`，这样可以保证JVM核心库的安全。

而且我们不仅可以获取到对象的字段值，还可以设置字段值：

设置字段值是通过`Field.set(Object, Object)`实现的，

其中第一个`Object`参数是指定的实例，第二个`Object`参数是待修改的值。

例如：

```java
package Lnh;

import java.lang.reflect.Field;

public class Lnh {

    public static void main(String[] args) throws Exception {
        Person p = new Person("w0s1np");
        System.out.println(p.getName());
        Class c = p.getClass();
        Field f = c.getDeclaredField("name");
        f.setAccessible(true);
        f.set(p,"天上星.");
        Object value = f.get(p);
        System.out.println(value);
    }
}

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

运行上述代码，打印的`name`字段从`w0s1np`变成了`天上星.`，说明通过反射可以直接修改字段的值。

同样的，修改非`public`字段，需要首先调用`setAccessible(true)`。

### 访问方法

我们已经能通过`Class`实例获取所有`Field`对象，同样的，可以通过`Class`实例获取所有`Method`信息。

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）

例如：

```java
package Lnh;

import java.lang.reflect.Method;

public class Lnh {

    public static void main(String[] args) throws Exception {
        Class stdClass = Student.class;
        // 获取public方法getScore，参数为String:
        System.out.println(stdClass.getMethod("getScore", String.class));
        // 获取继承的public方法getName，无参数:
        System.out.println(stdClass.getMethod("getName"));
        // 获取private方法getGrade，参数为int:
        System.out.println(stdClass.getDeclaredMethod("getGrade", int.class));
    }
}

class Person {
    public String getName() {
        return "Person";
    }
}

class Student extends Person {
    public int getScore(String type) {
        return 99;
    }
    public int getGrade(int year) {
        return 1;
    }
}
```

输出：

```java
public int Lnh.Student.getScore(java.lang.String)
public java.lang.String Lnh.Person.getName()
public int Lnh.Student.getGrade(int)
```

一个`Method`对象包含一个方法的所有信息：

- `getName()`：返回方法名称，例如：`"getScore"`；
- `getReturnType()`：返回方法返回值类型，也是一个Class实例，例如：`String.class`；
- `getParameterTypes()`：返回方法的参数类型，是一个Class数组，例如：`{String.class, int.class}`；
- `getModifiers()`：返回方法的修饰符，它是一个`int`，不同的bit表示不同的含义。

那如何调用方法呢？

通过`Method`实例中的`invoke`，同样的，如果需要调用非 public 方法，就需要在代码中加入`Method.setAccessible(true)`允许其调用



知道了这些之后，那我们如何实现**调用一个对象的任意方法**呢，分下面三步：

以一个 User 类为例子：

```java
class User{
    private String name;
    private int age;

    @Override
    public String toString(){
        return "User{" + "name=" +name + ", age="+age+"}";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

### 1.获取 Class 类实例

实现反射机制获取**"类"**实例（即 java.lang.Class 对象）有三种方法：

```java
package Lnh;

public class Lnh {

    public static void main(String[] args) throws Exception {
        Class class1 = null;
        Class class2 = null;
        Class class3 = null;
        class1 = Class.forName("Lnh.Lnh");
        //java中任何一个对象都有getClass方法
        class2 = new Lnh().getClass();
        //java中任何一个对象都有class属性
        class3 = Lnh.class;
        System.out.println("类名称: " + class1.getName());
        System.out.println("类名称: " + class2.getName());
        System.out.println("类名称: " + class3.getName());
    }
}
```

forName 有两个函数重载： 

- Class forName(String name) 
- Class forName(String name, **boolean** initialize, ClassLoader loader)

第⼀个就是我们最常⻅的获取 class 的⽅式，其实可以理解为第⼆种⽅式的⼀个封装：

```
Class.forName(className)
// 等于
Class.forName(className, true, currentLoader)
```

默认情况下， forName 的第⼀个参数是类名；第⼆个参数表示是否初始化；第三个参数就 是 ClassLoader 。

### 2.获取类中的方法

通过 Method 这个类，java 中所有的方法都是 Method 类型，所以我们通过反射机制获取到某个对象的方法也是 Method 类型。

通过 Class 对象获取到某个方法：

```java
clz.getMethod(方法名，这个方法的参数类型)

例：
Method method = clz.getMethod("setName", String.class);
```

### 3.调用类中的方法

Method 类中有一个 invoke 方法，就是用来调用特定方法的，用法如下：

![1](./img/242.png)

第一个参数是调用该方法的对象，第二个参数是一个可变长参数，是这个方法的需要传入的参数，

所以最后代码：

```java
package Lnh;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Scanner;

public class Lnh {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
            User user = new User();
            Class clz = user.getClass();
            Method method = clz.getMethod("setName", String.class);
            method.invoke(user,"w0s1np");
            System.out.println(user);
    }
}

class User{
    private String name;
    private int age;

    @Override
    public String toString(){
        return "User{" + "name=" +name + ", age="+age+"}";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

![1](./img/243.png)

成功调用 setName 方法，并设置 name 为"w0s1np"。



那么如何利用反射机制来执行恶意代码呢，又先来看一个东西：

类的初始化：

```java
public class Lnh {
    {
        System.out.printf("Empty block initial %s\n", this.getClass());
    }
    static {
        System.out.printf("Static initial %s\n", Lnh.class);
    }
    public Lnh() {
        System.out.printf("Initial %s\n", this.getClass());
    }
}
```

⾸先调⽤的是 static {} ，其次是 {} ，最后是构造函数。

其中， static {} 就是在"类初始化"的时候调⽤的，⽽ {} 中的代码会放在构造函数的 super() 后⾯， 但在当前构造函数内容的前⾯。

所以说， forName 中的 initialize=true 其实就是告诉Java虚拟机是否执⾏"类初始化"。

我的理解就是 static 是绑定到一个类里面的，在反射中，我们可以通过`forName`的第二个参数`initialize`来控制是否将 static 类型的属性或方法绑定到类中。

所以漏洞就产生了：

我们利用`Class.forname`可以获取到任意类的`Class`类，那如果那个类中也存在恶意代码在 static 方法中，那么就可以实现恶意代码执行

### 加载任意类

前文已经说到我们能获得`Class`，获取到`Class`以后们可以继续使用反射来获取这个类中的属性、方法，也可以实例化这个类，并调用方 法

拿如何实例化`Class`呢，有两种方式，

1. 使用Class对象的 newInstance() 方法来创建 Class 对象对应类的实例。

   class.newInstance() 的作用就是调用这个类的无参构造函数，但当使用的类没有无参构造函数或者使用的类构造函数是私有的那么这种方法就行不通了，例如：

   ```java
   package Lnh;
   
   public class Lnh {
       public static void main(String[] args) throws Exception {
           Class clazz = Class.forName("java.lang.Runtime");
           clazz.getMethod("exec", String.class).invoke(clazz.newInstance(), "id");
       }
   }
   ```

   ![1](./img/244.png)

   因为 Runtime 类就是[单例模式](https://www.runoob.com/design-pattern/singleton-pattern.html)，我们只能通过 Runtime.getRuntime() 来获取到 Runtime 对象。（其实这里就是利用的单例模式里的静态方法来实例化 Class 的）

   ```java
   package Lnh;
   
   public class Lnh {
       public static void main(String[] args) throws Exception {
           Class clazz = Class.forName("java.lang.Runtime");
           clazz.getMethod("exec", String.class).invoke(clazz.getMethod("getRuntime").invoke(clazz), "calc.exe");
       }
   }
   ```

但是这里也可以利用`getDeclared`来获取私有方法，它与的 getMethod 、 getConstructor 区别是：

- getMethod 系列方法获取的是当前类中所有公共方法，包括从父类继承的方法
- getDeclaredMethod 系列方法获取的是当前类中“声明”的方法，是实在写在这个类里的，包括私 有的方法，但从父类里继承来的就不包含了

所以上面代码也可以这样执行：

```java
package Lnh;

import java.lang.reflect.Constructor;

public class Lnh {
    public static void main(String[] args) throws Exception {
        Class clazz = Class.forName("java.lang.Runtime");
        Constructor m = clazz.getDeclaredConstructor();
        m.setAccessible(true);
        clazz.getMethod("exec", String.class).invoke(m.newInstance(), "calc.exe");
    }
}
```

2. 先通过 Class 对象获取指定的 Constructor 对象，再调用 Constructor 对象的 newInstance() 方法来创建实例。这种方法可以用指定的构造器构造类的实例。因为构造函数也支持重载， 所以必须用参数列表类型才能唯一确定一个构造函数。

   ```java
   // 获取String所对应的Class对象
   Class<?> c = String.class;
   // 获取String类带一个String参数的构造器
   Constructor constructor = c.getConstructor(String.class);
   // 根据构造器创建实例
   Object obj = constructor.newInstance("23333");
   System.out.println(obj);
   ```

   简单来说就是先用 getConstructor 方法来获取构造函数，然后使用 newInstance 来实例化 Class

   例如：

   ```java
   package Lnh;
   
   import java.util.List;
   import java.util.Arrays;
   
   public class Lnh {
       public static void main(String[] args) throws Exception {
           Class clazz = Class.forName("java.lang.ProcessBuilder");
           //((ProcessBuilder) clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe"))).start();
           clazz.getMethod("start").invoke(clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe")));
       }
   }
   ```

这里 newInstance 的参数需要根据 ProcessBuilder 的构造函数参数形式改变，或者也可以换一个构造函数：

```java
package Lnh;

import java.util.List;
import java.util.Arrays;

public class Lnh {
    public static void main(String[] args) throws Exception {
        Class clazz = Class.forName("java.lang.ProcessBuilder");

        //public ProcessBuilder(List<String> command)
        //((ProcessBuilder) clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe"))).start();
        //clazz.getMethod("start").invoke(clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe")));

        //public ProcessBuilder(String... command)  可变参数在编译的时候会编译成一个数组，所以下面newInstance的参数是一个数组，ProcessBuilder的也是一个数组，所以下面就有了二维数组
        //((ProcessBuilder) clazz.getConstructor(String[].class).newInstance(new String[][]{{"calc.exe"}})).start();
        clazz.getMethod("start").invoke(clazz.getConstructor(String[].class).newInstance(new String[][]{{"calc.exe"}}));
    }
}
```

