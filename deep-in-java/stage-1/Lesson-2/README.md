

# lesson-1

> 摘要：[Integer](#Integer 类型), [String](#String 类型), Enum, Interface, UnmodifiableInterface, InnerClass, Snapshot, Cloneable

## Integer 类型

Integer 类型是对原生类型 int 类型的包装。原始类型不具有对象的特性，包装类型的诞生解决原始类型不具有对象特性的问题。包装类型的对象特性使得数字也可以拥有 valueOf, compareTo, toString 等方法。

### equals 方法

```java
/**
 * The value of the {@code Integer}.
 *
 * @serial
 */
private final int value;  // [1]

public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue(); // [2]
    }
    return false;
}

/**
 * Returns the value of this {@code Integer} as an
 * {@code int}.
 */
public int intValue() {
    return value;
}
```

[1]:Integer 内部其实是维护了一个 int 类型字段名为 `value` 的字段，这是 Integer 真正的「值」。

[2]:equals 方法的对比实现是将两个 Integer 对象的 `value` 拿出来对比，即对比两个对象的「值」。

### valueOf 方法

```java
@HotSpotIntrinsicCandidate
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high) // [1]
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

[1]:如果 valueOf 方法的参数数值在 -128~127 之间，那么通过缓存获取对象。

 - 即 `Integer.valueOf(1) == Integer.valueOf("1")` 默认情况下是等价的。

**自动装箱和 valueOf 方法**

```java
// 自动装箱代码
Integer value = 99;
Integer value3 = Integer.valueOf(99);
System.out.println("value == value3:" + (value == value3));//true
```

以上语句将打印:`value == value3:true`。

通过  `javap -v`  命令查看字节码来理解 `Integer value = 99` 的真实语义。

```java
// 截图局部字节码信息
public static void main(java.lang.String[]);
   descriptor: ([Ljava/lang/String;)V
   flags: (0x0009) ACC_PUBLIC, ACC_STATIC
   Code:
     stack=4, locals=4, args_size=1
        0: bipush        99
        2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        5: astore_1
        6: new           #3                  // class java/lang/Integer
        9: dup
       10: bipush        99
       12: invokespecial #4                  // Method java/lang/Integer."<init>":(I)V
       15: astore_2
       16: bipush        99
       18: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
```

代码与字节码对比可以看出 `Integer value = 99` 实际上被编译器编译成了字节码 `Integer value = Integer.valueOf(99)`， 从而解释了自动装箱和 valueOf 方法的关系。那为什么`Integer.valueOf(99) == Integer.valueOf(99)` 呢？这就涉及到 **IntegerCache**。

```java
// 自动装箱代码
Integer value = 99;

--------------------------------------------------------
// 字节码
0: bipush        99
2: invokestatic  #2                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
```

**IntegerCache**

```java
 /**
    * Cache to support the object identity semantics of autoboxing for values between
    * -128 and 127 (inclusive) as required by JLS.
    *
    * The cache is initialized on first usage.  The size of the cache
    * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
    * During VM initialization, java.lang.Integer.IntegerCache.high property
    * may be set and saved in the private system properties in the
    * jdk.internal.misc.VM class.
    */
```

- 缓存支持 -128 ~ 127 自动装箱

- IntegerCache 缓存的范围可以通过 `-XX:AutoBoxCacheMax=<size>` 来指定。

  - 在 VM 初始化时，`java.lang.Integer.IntegerCache.high` 通过私有系统属性设置和保存到`jdk.internal.misc.VM`。

  - `-XX:AutoBoxCacheMax=<size>` 的取值如下[1]：

    ```java
    int h = 127;
    String integerCacheHighPropValue =
        VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
    if (integerCacheHighPropValue != null) {
        try {
            int i = parseInt(integerCacheHighPropValue);
            i = Math.max(i, 127);  // [1]
            // Maximum array size is Integer.MAX_VALUE
            h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
        } catch( NumberFormatException nfe) {
            // If the property cannot be parsed into an int, ignore it.
        }
    }
    high = h;
    ```

# String 类型

## 语法特性

```java
// 常量化是原生类型支持，赋值方式
int a = 1; // 常量
String value = "Hello"; // 常量（语法特性） = 对象类型常量化
```

面向对象规则：一切对象需要 new。但是也有一些情况被[隐式创建](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.5)。

A new class instance may be implicitly created in the following situations:

- Loading of a class or interface that contains a `String` literal ([§3.10.5](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.10.5)) may create a new `String` object to represent that literal. (This might not occur if the same `String` has previously been interned ([§3.10.5](https://docs.oracle.com/javase/specs/jls/se8/html/jls-3.html#jls-3.10.5)).)

- Execution of an operation that causes boxing conversion ([§5.1.7](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.7)). Boxing conversion may create a new object of a wrapper class associated with one of the primitive types.

- Execution of a string concatenation operator `+` ([§15.18.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.18.1)) that is not part of a constant expression ([§15.28](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.28)) always creates a new `String` object to represent the result. String concatenation operators may also create temporary wrapper objects for a value of a primitive type.

- Evaluation of a method reference expression ([§15.13.3](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.13.3)) or a lambda expression ([§15.27.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.27.4)) may require that a new instance of a class that implements a functional interface type be created.


## 是否能够被修改？

可以通过反射修改 String 的属性 `value`。

```java
/**
    * The value is used for character storage.
    *
    * @implNote This field is trusted by the VM, and is a subject to
    * constant folding if String instance is constant. Overwriting this
    * field after construction will cause problems.
    *
    * Additionally, it is marked with {@link Stable} to trust the contents
    * of the array. No other facility in JDK provides this functionality (yet).
    * {@link Stable} is safe here, because value is never null.
    */
@Stable
private final byte[] value;
```

## 为什么是 final 的？

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
private final byte[] value;
	...
}
```

- String 内部维护的 value 字段适用的修饰符是 final
- String 类的修饰符也是 final
- String 没有提供任何 API 修改 `value` 的值
- String 是语言级别的类。（若可以继承 String，那么继承的类是否要得到语言/[语法](#语法特性)级别的支持？）

## 比较操作

[Java 语言规定 String 字面量必须引用相同的 String 实例](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-5.html)

The Java programming language requires that identical string literals (that is, literals that contain the same sequence of code points) must refer to the same instance of class `String` (JLS §3.10.5). In addition, if the method `String.intern` is called on any string, the result is a `reference` to the same class instance that would be returned if that string appeared as a literal. Thus, the following expression must have the value `true`:

```
("a" + "b" + "c").intern() == "abc"
```

## hashCode 方法

```java
private final char[] value;
private int hash; 

public String(String original) {
    this.value = original.value;
    this.hash = original.hash;	// [1]
}

public int hashCode() {
    int h = hash;   // [2]
    if (h == 0 && value.length > 0) { //  [3]
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

- String 重写了 Object 的 hashCode 方法

- hashCode 的算法`s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`

## equals 方法

```java
public boolean equals(Object anObject) {
    if (this == anObject) {  // [1]
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {	// [2]
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])	//  [3]
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

- [1] 内存地址
- [2] 先判断长度
  - [3] 遍历并每个字符进行对比

# Enum 类型

## equals or ==

```java
/**
  * Returns true if the specified object is equal to this
  * enum constant.
  *
  * @param other the object to be compared for equality with this object.
  * @return  true if the specified object is equal to this
  *          enum constant.
  */
public final boolean equals(Object other) {
    return this==other;
}
```

**《java 核心技术卷 Ⅰ》-5.6 枚举类**第三段说：『枚举比较最好用 ==，而不是用 equals』。其实效果是一样的。

## 枚举有构造器和无构造器的区别

无构造器的枚举类(如:java.time.DayOfWeek),只需要用成员的名称(如:MONDAY,TUESDAY...)就可以了。成员名称代表一个枚举类，即是类级别。

```java
public enum DayOfWeek implements TemporalAccessor, TemporalAdjuster {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;
    // ...
}
```

有构造器的枚举类(如:org.springframework.http.HttpStatus),除了**类名**见文知意之外,类中还包含了属性和方法。即有构造器是为构造有属性和方法的枚举类。

```java
public enum HttpStatus {
	CONTINUE(100, "Continue"),
	SWITCHING_PROTOCOLS(101, "Switching Protocols"),
	PROCESSING(102, "Processing"),
	CHECKPOINT(103, "Checkpoint"),
	OK(200, "OK"),
    // ...

    /**
	 * Return the integer value of this status code.
	 */
	public int value() {
		return this.value;
	}

	/**
	 * Return the reason phrase of this status code.
	 */
	public String getReasonPhrase() {
		return this.reasonPhrase;
	}
    // ...
}
```

## 打印枚举类为什么打印的是枚举的常量/成员名称

```java
/**
  * Returns the name of this enum constant, as contained in the
  * declaration.  This method may be overridden, though it typically
  * isn't necessary or desirable.  An enum type should override this
  * method when a more "programmer-friendly" string form exists.
  *
  * @return the name of this enum constant
  */
public String toString() {
    return name;
}
```

返回枚举常量的名字。这个方法可以被重写，说明可以按自己的需求来写 toString()。一般不用重写，除非有更好的方式组合 toString() 的值。

## 枚举的逆方法

通过 valueOf() 方法可以得到一个枚举类，如果类型和名称找不到则返回 null。

```java
public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                            String name) {
    T result = enumType.enumConstantDirectory().get(name);
    if (result != null)
        return result;
    if (name == null)
        throw new NullPointerException("Name is null");
    throw new IllegalArgumentException(
        "No enum constant " + enumType.getCanonicalName() + "." + name);
}
```

实例：

```java
 DayOfWeek dayOfWeek = Enum.valueOf(DayOfWeek.class, "MONDAY");
```

## 枚举类的隐式方法

```java
Note that for a particular enum type T, the implicitly declared public static T valueOf(String) method on that enum may be used instead of this method to map from a name to the corresponding enum constant. All the constants of an enum type can be obtained by calling the implicit public static T[] values() method of that type.
    
注意，枚举类型 T 有点特别之处，隐式声明了 public static T valueOf(String) 方法。 valueOf(String) 方法可以替代 valueOf(Class, String) 方法，根据名称映射到对应的枚举常量。所有的枚举常量可以被获取，通过调用隐式方法 public static T[] values[]。
```

- valueOf(String)
  - 得到一个枚举类，可以替代逆方法 valueOf(Class, String)。
- values()
  - 打印所有的成员名称。

## ordinal 方法

```java
/**
  * The ordinal of this enumeration constant (its position
  * in the enum declaration, where the initial constant is assigned
  * an ordinal of zero).
  *
  * Most programmers will have no use for this field.  It is designed
  * for use by sophisticated enum-based data structures, such as
  * {@link java.util.EnumSet} and {@link java.util.EnumMap}.
  */
private final int ordinal;

public final int ordinal() {
    return ordinal;
}
```

获取枚举常量的声明顺序，编号从零开始。大多数程序员都不会用到这个字段，它的设计是为服务于基于枚举的数据结构例如 `java.util.EnumSet` 和 `java.util.EnumMap`。

## compareTo 方法

```java
/**
  * Compares this enum with the specified object for order.  Returns a
  * negative integer, zero, or a positive integer as this object is less
  * than, equal to, or greater than the specified object.
  *
  * Enum constants are only comparable to other enum constants of the
  * same enum type.  The natural order implemented by this
  * method is the order in which the constants are declared.
  */
public final int compareTo(E o) {
    Enum<?> other = (Enum<?>)o;
    Enum<E> self = this;
    if (self.getClass() != other.getClass() && // optimization
        self.getDeclaringClass() != other.getDeclaringClass())
        throw new ClassCastException();
    return self.ordinal - other.ordinal;
}
```

compareTo(E o) 方法比较的是声明顺序，返回负数、0、正数。 