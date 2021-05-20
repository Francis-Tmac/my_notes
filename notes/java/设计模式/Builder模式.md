## 创建对象的方式
- 正常创建 new 
- 反射创建 Class 或 java.lang.reflect.Constructor 的 newInstance()方法
- clone 创建。调用现有对象的clone 方法
- 反序列化

### 单一构造器
不能满足，某些选填字段不传入，大对象时构造函数太长容易出错

### 多构造器
选填字段 出现相同类型时，构造函数不能被重载，Java只认变量的类型。

### set 方法创建
每个字段都调用set 方法创建，如果是多线程环境，对象不能被安全的构建，因为它不是不可变对象。一旦对象呗构建，后续还可以更具 set 方法改变对象内部状态。当一个对象正在执行相关业务方法，另外的线程改变了其内部状态。

### Builder 方式
1. 将对象的字段设置 private final 一旦创建完成，内部状态不可变。--线程安全
2. 对象所属类不是为了继承而设计
3. 使用私有方法构造器，确保对象无法从外部创建
4. Builder 内部静态类，必须是对象类的内部类，否则，由于对象类的构造器私有，不能通过new 的方式创建。
5. 必须是静态：由于对象类无法从外部创建，如果不是静态类，外部则无法引用 Builder 对象。
6. Builder 内部成员变量要与对相关类成员变量保持一致。
7. 最后通过 Builder#builde() 方法创建对象类。

``` java
public static class Builder {  
// 姓名(必填)。注意:这里不能是 final 的 private String name;

// 年龄(必填) private int age;

// 身高(选填) private int height;

// 毕业学校(选填) private String school;

// 爱好(选填) private String hobby;

public Builder(String name, int age) { this.name = name;

this.age = age; }

public Builder setHeight(int height) { this.height = height;

return this; }

public Builder setSchool(String school) { this.school = school;  
return this;

}

public Builder setHobby(String hobby) { this.hobby = hobby;  
return this;

}

/**  
\* 构建对象  
\* @return 返回待构建的对象本身 */

public Person build() {  
return new Person(name, age, height, school, hobby);

}

}