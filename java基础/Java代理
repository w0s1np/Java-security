# Java 代理

Java代理是一种程序的设计模式，它提供了一个代理对象，我们可以通过代理对象访问目标对象进行相关操作而不去直接修改原对象

通过代理对象我们可以扩展对目标对象的访问功能，简化访问方式

## 静态代理

静态代理**一般需要代理对象和目标对象实现一样的接口**

优点：可以在不修改目标对象的前提下扩展目标对象的功能。

缺点：

1. 冗余。由于代理对象要实现与目标对象一致的接口，会产生过多的代理类。
2. 不易维护。一旦接口增加方法，目标对象与代理对象都要进行修改。

```java
package Lnh;

public class Lnh {
    public static void main(String[] args) throws Exception {
        IUser user = new User();
        // 创建静态代理
        IUser userProxy = new UserProxy(user);
        userProxy.sayName();
    }
}

interface IUser {
    public void sayName();
}

class User implements IUser{

    @Override
    public void sayName() {
        System.out.println("tntaxin");
    }
}

class UserProxy implements IUser {
    private IUser target;
    public UserProxy(IUser obj){
        this.target = obj;
    }

    @Override
    public void sayName() {
        System.out.println("我是他的代理");
        this.target.sayName();
    }
}
```

输出：

```
我是他的代理
tntaxin
```



## 动态代理

动态代理不需要事先目标对象的接口，但是要求目标对象必须是要事先接口的

JDK原生动态代理用到了JDK原生的API(主要是`java.lang.reflect.Proxy`和`java.lang.reflect.InvocationHandler`)，在Java程序运行过程中动态地生成代理对象。

静态代理与动态代理的区别：

- 静态代理在编译时就已经实现，编译完成后代理类是一个实际的 class 文件
- 动态代理是在运行时动态生成的，即编译完成后没有实际的 class 文件，而是在运行时动态生成类字节码，并加载到JVM中

优点

- 减少代理类的数量，更易维护优化

缺点

- 比较消耗系统性能

代理类 Proxy 的主要方法：

```java
public static Object newProxyInstance(ClassLoader loader,	//类加载器
                                      Class<?>[] interfaces,	//目标对象实现的接口的类型
                                      InvocationHandler h)	//调用处理器(目标对象)
```

代理类 InvocationHandler 的主要方法：

```java
Object invoke(Object proxy, Method method, Object[] args)
    //在代理实例上处理方法调用并返回结果
```

示例代码：

```java
package Lnh;

import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.InvocationHandler;

public class Lnh {
    public static void main(String[] args) throws Exception {
        IUser user = new User();

        // 创建动态代理  
        ProxyFactory proxyFactory = new ProxyFactory(user);
        IUser userProxy2 = (IUser)proxyFactory.getProxyInstance();
        // 打印生成的代理对象的类名
        System.out.println(userProxy2.getClass());
        userProxy2.sayName();
    }
}

interface IUser {
    public void sayName();
}

class User implements IUser{

    @Override
    public void sayName() {
        System.out.println("tntaxin");
    }
}

class ProxyFactory {
    private Object target;
    public ProxyFactory(Object target){
        this.target = target;
    }

    public Object getProxyInstance(){
        return Proxy.newProxyInstance(this.target.getClass().getClassLoader(),           this.target.getClass().getInterfaces(),
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("我是他的代理");
                    method.invoke(target, args);
                    return null; }
            });
    }
}
```

它使用到了`Proxy.newProxyInstance`方法：

```java
public static Object newProxyInstance(ClassLoader loader,	//类加载器
                                      Class<?>[] interfaces,	//目标对象实现的接口的类型
                                      InvocationHandler h)	//调用处理器(目标对象)
    
    //返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。
```

因为这个方法中的第二个参数需要目标对象实现的接口的类型，所以动态代理的类必须要实现接口。

其中第三个参数是事件处理器`InvocationHandler`，实现该类必须要实现`invoke`方法

```java
 Object invoke(Object proxy, Method method, Object[] args) 
// 在代理实例上处理方法调用并返回结果。
```

运行程序，会得到如下输出：

```
class $Proxy0
我是他的代理
tntaxin
```

这里输出了动态生成的代理类的类名，我们还可以进一步用一些字节码操作工具把这个代理类的源码给输出到文件中，在代码中将`sun.misc.ProxyGenerator.saveGeneratedFiles`属性值设为 true，就会在项目根目录下生成代理类的文件，改动后的代码如下：

```java
public class Lnh
{  
    public static void main( String[] args )  
    {  
        IUser user = new User();  
  
 		//生成$Proxy0的class文件 
		System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");  
 		// 创建动态代理  
 		ProxyFactory proxyFactory = new ProxyFactory(user);  
		IUser userProxy2 = (IUser)proxyFactory.getProxyInstance();  
 		// 打印生成的代理对象的类名  
 		System.out.println(userProxy2.getClass());  
 		userProxy2.sayName();  
 	}  
}
```



## cglib代理

cglib 是一个第三方代码库，运行时会在内存动态生成一个子类对象从而实现对目标对象功能的扩展。

cglib 代理无需实现接口，通过生成类字节码实现代理，比反射稍快，不存在性能问题，但 cglib 会继承目标对象，需要重写方法，所以目标对象不能为 final 类。

cglib与动态代理最大的**区别**就是

- 使用动态代理的对象必须实现一个或多个接口
- 使用 cglib 代理的对象则无需实现接口，达到代理类无侵入。

使用cglib需要引入[cglib的jar包](https://link.segmentfault.com/?enc=XOUhxK7XrpqWhz07H5QMTQ%3D%3D.Kwo9Fll31atenHUfyhyDwBBw0W5xY9IGilJe5ZUa1HIMeopOt2GheoMYTjC%2F5GmiFmqrUm5H%2FBqh57uv75SFXlINYtJPt1y5FyfIEJH%2BBR0%3D)，如果已经有spring-core的jar包，则无需引入，因为spring中包含了cglib。

同样的，我们来创建demo，先引入cglib包

```java
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.5</version>
</dependency>
```

然后使用 cglib 来创建一个代理工厂类：

```java
package Lnh;

import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.InvocationHandler;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class Lnh {
    public static void main(String[] args) throws Exception {
        IUser user = new User();
        // cglib创建代理类
        IUser userProxy3 = (IUser)new ProxyFactory(user).getProxyInstance();
        System.out.println(userProxy3.getClass());
        userProxy3.sayName();
    }
}

interface IUser {
    public void sayName();
}

class User implements IUser{

    @Override
    public void sayName() {
        System.out.println("tntaxin");
    }
}

class ProxyFactory implements MethodInterceptor {

    private Object target;

    public ProxyFactory(Object target){
        this.target = target;
    }

    //为目标对象生成代理对象
    public Object getProxyInstance() {
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperclass(target.getClass());
        //设置回调函数
        en.setCallback(this);
        //创建子类对象代理
        return en.create();
    }


    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("我是cglib生成的代理");
        method.invoke(target, objects);
        return null; }
}
```

输出：

```
class User$$EnhancerByCGLIB$$59c1fe5d
我是cglib生成的代理
tntaxin
```
