# 编译器常见的优化方式

## 1.常量传播

常量传播，就是说在编译期时，能够直接计算出结果（这个结果往往是常量）的变量，将被编译器由直接计算出的结果常量来替换这个变量。

eg:

```cpp
int main(int argc,char **argv){
    int x = 1;
    //原始语句：std::cout<<x<<std::endl;
    //经过常量传播后
    std::cout<<1<<std::endl;
    return 0;
}
```

## 2.常量折叠

常量折叠，就是说在编译期间，如果有可能，多个变量的计算可以最终替换为一个变量的计算，通常是多个变量的多级冗余计算被替换为一个变量的一级计算，而不是在运行的时候计算它们。

eg:

```cpp
int main(int argc,char **argv){
    int a = 1;
    int b = 2;
    int x = a + b;
    std::cout<<x<<std::endl;
    return 0;
}
```

会被转化为：

```cpp
int main(int argc,char **argv){
    int x = 1 + 2;
    std::cout<<x<<std::endl;
    return 0;
}
```

或者更进一步：

```cpp
int main(int argc,char **argv){
    std::cout<<3<<std::endl;
    return 0;
}
```

## 3.复写传播

复写传播，就是编译器用一个变量替换两个或多个相同的变量。

eg:

```cpp
int main(int argc,char **argv){
    int y = 1;
    int x = y;
    std::cout<<x<<std::endl;
    return 0;
}
```

优化后：

```cpp
int main(int argc,char **argv){
    int x = 1;
    std::cout<<x<<std::endl;
    return 0;
}
```

上例有两个变量y和x，但是其实是两个相同的变量，并且其它地方并未区分它们两个，所以它们是重复的，可称为“复写”，编译器可以将其优化，将x“传播”给y，只剩下一个变量x，当然，反过来优化掉x只剩下一个y也是可以的。

## 4.公共子表达式消除

公共子表达式消除是说，如果一个表达式E已经计算过了，并且从先前的计算到现在的E中的变量都没有发生变化，那么E的此次出现就成为了公共子表达式，因此，编译器可判断其不需要再次进行计算浪费性能。

eg:

```cpp
int main(int argc,char **argv){
    int a = 1;
    int b = 2;
    int x = （a+b) * 2 + (b+a) * 6;
    std::cout<<x<<std::endl;
    return 0;
}
```

优化后：

```cpp
int main(int argc,char **argv){
    int a = 1;
    int b = 2;
    int E = a + b;
    int x = E * 2 + E * 6;
    std::cout<<x<<std::endl;
    return 0;
}
```

当然，也有可能会直接变成：

```cpp
int main(int argc,char **argv){
    int a = 1;
    int b = 2;
    int E = a + b;
    int x = E * 8;
    std::cout<<x<<std::endl;
    return 0;
}
```



## 5.无用代码消除

无用代码消除指的是永远不能被执行到的代码或者没有任何意义的代码会被清除掉，比如return之后的语句，变量自己给自己赋值等等。

eg:

```cpp
int main(int argc,char **argv){
    int x = 1;
    int x = x;
    std::cout<<x<<std::endl;
    return 0;
}
```

上例中，x变量自我赋值显然是无用代码，将会被优化掉：

```cpp
int main(int argc,char **argv){
    int x = 1;
    std::cout<<x<<std::endl;
    return 0;
}
```

## 6.数组范围检查消除

如果开发语言是Java这种动态类型安全型的，那在访问数组时比如array[ ]时，Java不会像C/C++那样只是存粹的裸指针访问，而是会在运行时访问数组元素前进行一次是否越界检查，这将会带来许多开销，如果即时编译器能根据数据流分析出变量的取值范围在[0,array.length]之间，那么在循环期间就可以把数组的上下边界检查消除，以减少不必要的性能损耗。

## 7.方法内联

这种优化方法是将比较简短的函数或者方法代码直接粘贴到其调用者中，以减少函数调用时的开销，比较重要且常用，很容易理解，就比如C++的inline关键字一样，只不过inline是开发者的手动方法内联，而编译器在分析代码和数据流之后，也有可能做出自动inline的优化。

## 8.逃逸分析

一个对象如果被其声明的方法之外的一个或多个函数所引用，那就被称为逃逸，可以通俗理解为，该对象逃逸了其原本的命名空间或者作用域，使得声明（或者定义）该对象的方法结束时，该对象不能被销毁。

通常，一个函数里的局部变量其内存空间是在栈上分配的，而对象则是在堆上分配的内存空间，在函数调用结束时，局部变量会随着栈空间销毁而自动销毁，但堆上的空间要么是依赖类似JVM的垃圾内存自动回收机制（GC），要么就得像C/C++那样的依赖开发者本身的记忆力，因此，堆上的内存分配与销毁一般开销会比栈上的大得多。

逃逸分析的基本原理就是分析对象动态作用域。如果确定一个方法不会逃逸出方法之外，那让整个对象在栈上分配内存将会是一个很不错的主意，对象所占用的内存空间就可以随栈帧而销毁。在一般应用中，不会逃逸的局部对象所占用的比例很大，如果能在编译器优化时，为其在栈上分配内存空间，那大量的对象就会随着方法结束而自动销毁了，不用依赖前面讲的GC或者记忆力，系统的压力将会小很多。

# 参考资料

[1] [知乎文章——编译器常用的8种优化方式](https://zhuanlan.zhihu.com/p/381490718)