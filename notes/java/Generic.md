```java

public class Generic<T\> {  
  
    // 编译错误泛型类中今天太方法和静态变量不可以  
 // 使用泛型类所申明的泛型类型参数 // 泛型类中的泛型参数的实例化是在定义泛型对象的时候指定的。 // 静态变量不需要使用对象来调用。对象没有创建，如何确定泛型参数是何种类型 /\*   static T item;  
 static T show(T one){ return null; }*/  
  
 // 泛型方法，在泛型方法中使用的T是自己再方法中定义的T ,而不是泛型类中定义的T  public static<T\> T show(T one){  
        return null;  
  }  
}

```

11. Java 中 List&lt;Object&gt;和原始类型 List 之间的区别?
原始类型和带参数类型<Object>之间的主要区别是，在编译时编译器不会对原始类型进行类型安全检 查，**却会对带参数的类型进行检查**  ，通过使用 Object 作为类型，可以告知编译器该方法可以接受任何类 型的对象，比如 String 或 Integer。这道题的考察点在于对泛型中原始类型的正确理解。它们之间的第 二点区别是，你可以把任何带参数的泛型类型传递给接受原始类型 List 的方法，但却不能把 List&lt;String&gt;传递给接受 List&lt;Object&gt;的方法，因为会产生编译错误。  

12. Java 中 List&lt;?&gt;和 List&lt;Object&gt;之间的区别是什么? 
这道题跟上一道题看起来很像，实质上却完全不同。List&lt;?&gt; 是一个未知类型的 List，而 List&lt;Object&gt;其实是任意类型的 List。你可以把 List&lt;String&gt;, List&lt;Integer&gt;赋值给 List&lt;?&gt;，却 不能把 List&lt;String&gt;赋值给 List&lt;Object&gt;。