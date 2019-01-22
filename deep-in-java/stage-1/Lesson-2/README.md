

# lesson-1

> 摘要：Integer, String, Enum, Interface, UnmodifiableInterface, InnerClass, Snapshot, Cloneable

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
@HotSpotIntrinsicCandidate
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

代码与字节码对比可以看出 `Integer value = 99` 实际上被编译器编译成了字节码 `Integer value = Integer.valueOf(99)`， 从而解释了自动装箱和 valueOf 方法的关系。

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