![](https://www.lifeofpix.com/wp-content/uploads/2019/08/fabiancatoni_10-1600x1067.jpg)

# 前言

今天我们来学习一下在Kotlin中如何实现一个接口，然后了解下接口的相关知识。主要内容如下：

- 声明接口
- 实现接口
- override修饰符
- super关键字

## 声明接口

在Kotlin中，声明一个接口需要使用interface关键字。并且接口的声明方式与Java的接口声明方式是类似的，然后我们来声明一个HelloKotlin接口，代码如下：

```kotlin
interface HelloKotlin{
    fun setHelloKt()
    }
```

通过代码我们可以看出HelloKotlin接口通过interface来声明，拥有一个setHelloKt()抽象方法。那么HelloKotlin接口的方法能不能有默认实现呢？答案是可以的，代码如下：

```kotlin
interface HelloKotlin{
    fun setHelloKt()
    fun getHelloKt() = println("Hello Kotlin")
    }
```

在HelloKotlin接口中定义一个拥有默认实现的getHelloKt()方法，该方法的默认实现是输出"Hello Kotlin"。

###  小结

在Kotlin中声明一个接口，需要使用interface关键字，然后接口中的方法可以有默认实现。

## 实现接口

前面我们已经声明了一个HelloKotlin接口，接下来我们来实现它，代码如下：

```kotlin
//Kotlin中实现接口使用 :
class Hello : HelloKotlin{
    //重写接口中的方法，需要使用override关键字
    override fun setHelloKt() {
        println("Hello Kotlin")
    }
    //getHelloKt()在接口中有默认实现
    override fun getHelloKt() = println("Hello Kotlin Kotlin")
}
```

此时我们创建一个Hello类，并实现了HelloKotlin接口。从代码中可以看出setHelloKt()和getHelloKt()两个方法都使用了override关键字，在Kotlin中override修饰符用来标注被重写的父类方法或者接口的方法和属性。也就是说当我们实现接口中的方法和属性时，都需要添加override修饰符，并且Kotlin强制要求使用override修饰符，这样可以避免一些问题。

### 接口中的属性

```kotlin
interface HelloKotlin{
	var name: String
    fun setHelloKt()
    }
```

在接口中声明属性，默认是抽象的，并且不能进行初始化，需要在实现该接口的类中进行初始化。

```kotlin
class Hello : HelloKotlin{
	override var name : String = "HelloKt"
    //重写接口中的方法，需要使用override关键字
    override fun setHelloKt() {
        println("Hello Kotlin")
    }
}
```

从代码中可以看出属性同样需要使用override修饰符。

### 多个接口具有相同方法

接下来我们来看一下如果两个接口具有一个相同的方法，实现类应该如何分别调用。

```kotlin
interface HelloKt{
    fun setHelloKt()
    fun getHello() {
        println("Hello Kt")
    }
}
interface HelloWorld{
    fun setWorld()
    fun getHello()= println("Hello World")
}
```

从代码中可以看出HelloKt和HelloWorld两个接口都拥有getHello()方法，并且它们的实现不同。那么如何正确的调用它们呢？

```kotlin
 class Hello : HelloKt,HelloWorld {
        //重写接口中的方法，需要使用override关键字
        override fun setHelloKt() {
            println("我是Hello啊啊啊")
        }

        override fun getHello() {
          super<HelloWorld>.getHello()
            super<HelloKt>.getHello()
        }

        override fun setWorld() {
            println("set World")
        }
    }
```

在代码通过super关键字来分别调用两个接口中的getHello()方法，语法格式为super<接口名>.方法名。

# 总结

今天学习了接口的声明、属性、实现、override修饰符以及super关键字，有兴趣的小伙伴可以自己实践一下哦。

# 参考资料

- 《Kotlin实战》

