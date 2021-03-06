<!--
author: wngn123
head: head.png
date: 2016-08-24
title: Java源码 Integer
tags: java
category: Java
status: publish
summary: 阅读Java的源码，Integer源码
-->

## Integer继承体系 ##
Integer的签名如下，继承了Number类并实现Comparable接口
```java
public final class Integer extends Number implements Comparable<Integer>
```
Comparable接口强行对实现它的每个类的对象进行整体排序。此排序被称为该类的自然排序，类的 compareTo 方法被称为它的自然比较方法。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序。实现此接口的对象可以用作有序映射表中的键或有序集合中的元素，无需指定比较器。强烈推荐（虽然不是必需的）使自然排序与 equals 一致。所谓与equals一致是指对于类C的每一个e1和e2来说，当且仅当 (e1.compareTo((Object)e2) == 0) 与e1.equals((Object)e2) 具有相同的布尔值时，类C的自然排序才叫做与equals一致 
```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
Number为抽象类，提供了实数基本类型的互相转换方法：int,long,short,byte,float,double
Number的基本实现类有java.lang.Byte，java.lang.Double，java.lang.Float，java.lang.Integer，java.lang.Long，java.lang.Short等
```java
public abstract class Number implements java.io.Serializable {
    public abstract int intValue();
    public abstract long longValue();
    public abstract float floatValue();
    public abstract double doubleValue();
    public byte byteValue() {
        return (byte)intValue();
    }
    public short shortValue() {
        return (short)intValue();
    }
    private static final long serialVersionUID = -8742448824652078965L;
}

```
## Integer基本概念 ##
Integer类包装了int基本类型,所有Integer的实际值是存储在一个int类型的熟悉value上的，int存储占用4字节32位内存，第一位是符号位，所以int的最大值是2的31次幂减1，最小值是负2的31次幂，因此int的最大值是：0111 1111 1111 1111 1111 1111 1111 1111=0x7fffffff,int的最小值为负值，负数的表示为正值二进制的补码：
```
负数：正值的补码
补码：反码加一
源码：绝对值二进制
反码：二进制按位取反
```
负数最小值的正值为2的31次幂，正值的二进制为：1000 0000 0000 0000 00000 0000 0000 0000，二进制取反之后为：0111 1111 1111 1111 1111 1111 1111 1111，则正值的补码为：1000 0000 0000 0000 00000 0000 0000 0000，即int的最小值为：0x80000000

## Integer构造函数 ##
Integer有2个构造函数，可以传递一个int数字或者一个数字字符串，数字字符串会被转换成十进制的int数字。

```java
	private final int value;
    public static final int MIN_VALUE = 0x80000000;
    public static final int MAX_VALUE = 0x7fffffff;
    public static final int SIZE = 32;//int存储位数
    
    public Integer(int value) {
        this.value = value;
    }
    public Integer(String s) throws NumberFormatException {
        this.value = parseInt(s, 10);
    }
```
## Integer实例方法 ##
Integer的实例方法，Integer实现了从Number类和Comparable接口以及Object继承的方法。
```java
	public byte byteValue() {
        return (byte)value;
    }
    public short shortValue() {
        return (short)value;
    }
    public int intValue() {
        return value;
    }
    public long longValue() {
        return (long)value;
    }
    public float floatValue() {
        return (float)value;
    }
    public double doubleValue() {
        return (double)value;
    }
     public String toString() {
        return toString(value);
    }
    public int hashCode() {
        return value;
    }
    //判断数字相等使用基本类型int值比较，直接使用==比较Integer的是地址。
    public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
    //此处调用了类的静态方法比较两个int型数字的大小
    public int compareTo(Integer anotherInteger) {
        return compare(this.value, anotherInteger.value);
    }
    
    public static int compare(int x, int y) {
        return (x < y) ? -1 : ((x == y) ? 0 : 1);
    }
```
## IntegerCache ##
Integer类内部有一个私有的静态内部类IntegerCache，该类缓存了一些常用的int值到缓存中，从而保证这些int可以直接获取而不用构造。最大值可以通过虚拟机参数java.lang.Integer.IntegerCache.high类配置，但是最小的为127.

```java
	private static class IntegerCache {
        static final int low = -128;//最小缓存int值
        static final int high;//最大缓存int值，默认127
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }

```
## bitCount ##
Integer中的类方法：bitCount(int i),返回int数字的二进制原码中1出现的次数
如`Integer.bitCount(10)==2`,10的二进制源码为1010,1出现2次。

###### 原理：二分法，两两一组相加，之后四个四个一组相加，接着八个八个，最后就得到各位之和了。

1.两两一组相加,并且用该对应的两位来存储这个个数：二进制中先分割为2位计算，如108=01101100=0000 0000 0000 0000 0000 0000 0110 1100，第一个2位为00，出现1的个数为0+0=00，最后4个两位为：01,10,11,00,计算后的二进制为：
0+1=01,1+0=01,1+1=10,0+0=00,即0101 1000,最后得到的二进制是：0000 0000 0000 0000 0000 0000 0101 1000

2.四四一组相加，并且用该对应的四位来存储这个个数：对两两一组相加相加的结果四四一组相加，即
0000 0000 0000 0000 0000 0000 0101 1000进行四四一组相加，0101：01+01=0010，1000：10+00=0010,最后结果为：0000 0000 0000 0000 0000 0000 0010 0010

3.八八一组相加，并且用该对应的八位来存储这个个数：对四四一组相加相加的结果八八一组相加，即
0000 0000 0000 0000 0000 0000 0010 0010进行八八一组相加，00100010：0010+0010=00000100最后结果为：0000 0000 0000 0000 0000 0000 0000 0100

4.十六十六一组相加，并且用该对应的十六位来存储这个个数：对八八一组相加相加的结果十六十六一组相加，即0000 0000 0000 0100：0000 0000 + 0000 0100 = 0000 0000 0000 0100
最后结果为：0000 0000 0000 0000 0000 0000 0000 0100
最后相加：0000 0000 0000 0000 + 0000 0000 0000 0100 = 0000 0000 0000 0000 0000 0000 0000 0100 = 4

##### 计算过程：

第一行：i = i - ((i >>> 1) & 0x55555555);
i=0000 0000 0000 0000 0000 0000 0110 1100
0x55555555=0101 0101 0101 0101 0101 0101 0101 0101
二位二进制中:原码-高位值=1出现的个数，
（11-(11>>1)&01）=10=2
（10-(10>>1)&01）=01=1
（01-(01>>1)&01）=01=1
（00-(00>>1)&01）=00=0
i>>>1:原码值右移一位：01 10 11 00-->00 11 01 10
(i >>> 1) & 0x55555555：原码中所有两位二进制高位值：00 01 01 00
i - ((i >>> 1) & 0x55555555)：原码中所有两位二进制1的个数：01 01 10 00
二进制减法的规则是低位不足向高位借位，二位二进制原码-高位值，这样在做减法的时候保证了2位相减不会出现借位。

第二行：i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
i=0000 0000 0000 0000 0000 0000 0101 1000
0x33333333：0011 0011 0011 0011 0011 0011 0011 0011
（i & 0x33333333)：4位中的低二位值：0000 0000 0000 0000 0000 0000 0001 0000
((i >>> 2) & 0x33333333)：4位中的高二位值：0000 0000 0000 0000 0000 0000 0001 0010
四位中的高二位和低二位相加为：0000 0000 0000 0000 0000 0000 0010 0010

第三行：i = (i + (i >>> 4)) & 0x0f0f0f0f;
i=0000 0000 0000 0000 0000 0000 0010 0010
0x0f0f0f0f：0000 1111 0000 1111 0000 1111 0000 1111
i + (i >>> 4)：原码和右移四位相加，44相加，一般值无效，
0000 0000 0000 0000 0000 0000 0010 0010 +
0000 0000 0000 0000 0000 0000 0000 0010 = 
0000（无） 0000（有） 0000（无） 0000（有） 0000（无） 0000（有） 0010（无） 0100（有）
(i + (i >>> 4)) & 0x0f0f0f0f：去除无效值（无效值全部变为0）：00000000 00000000 00000000 00000100

第四行：i = i + (i >>> 8);
i = 00000000 00000000 00000000 00000100
i + (i >>> 8)：原码右移8位与原码相加（错8位相加）
00000000（有） 00000000（有） 00000000（有） 00000100（有）
00000000（无） 00000000（有） 00000000（有） 00000000（有）
00000000（无） 00000000（有） 00000000（无） 00000100（有）
i = i + (i >>> 8)： 00000000 00000000 00000000 00000100

第五行：i = i + (i >>> 16);
i = 00000000（无） 00000000（有） 00000000（无） 00000100（有）
i + (i >>> 16): 原码右移16位与原码相加（错16位相加）
00000000（无） 00000000（有） 00000000（无） 00000100（有）
00000000（无） 00000000（无） 00000000（无） 00000000（有）
00000000（无） 00000000（无） 00000000（无） 00000100（有）

第六行：i & 0x3f;
i= 00000000（无） 00000000（无） 00000000（无） 00000100（有）
0x3f:0011 0000 0000 0000 0000 0011 1111
i & 0x3f:最后计算的结果只有后6位二进制是有效结果：000100=4

#### i=1111 1111 1111 1111 1111 1111 1111 1111 
第一行：i = i - ((i >>> 1) & 0x55555555);
i=11 11 11 11 11 11 11 11 11 11 11 11 11 11 11 11 
0x55555555=01 01 01 01 01 01 01 01 01 01 01 01 01 01 01 01
 i - ((i >>> 1) & 0x55555555):10 10 10 10 10 10 10 10 

第二行：i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
i=1010 1010 1010 1010
0x33333333：0011 0011 0011 0011 0011 0011 0011 0011
（i & 0x33333333)：0010 0010 0010 0010 0010 0010 0010 0010
((i >>> 2) & 0x33333333)：4位中的高二位值：0010 0010 0010 0010 0010 0010 0010 0010
四位中的高二位和低二位相加为：0100 0100 0100 0100 0100 0100 0100 0100 0100

第三行：i = (i + (i >>> 4)) & 0x0f0f0f0f;
i=0100 0100 0100 0100 0100 0100 0100 0100 0100
0x0f0f0f0f：0000 1111 0000 1111 0000 1111 0000 1111
i + (i >>> 4)：原码和右移四位相加，44相加，一般值无效，
0100 0100 0100 0100 0100 0100 0100 0100 0100 +
0000 0100 0100 0100 0100 0100 0100 0100 0100 = 
0000（无效） 1000 1000（无效） 1000 1000（无效） 1000 1000（无效） 1000
(i + (i >>> 4)) & 0x0f0f0f0f：去除无效值（无效值全部变为0）：
00001000 0001000 0001000 0001000

第四行：i = i + (i >>> 8);
i=00001000 0001000 0001000 0001000
00001000（有） 00001000（有） 00001000（有） 00001000（有）
00000000（无） 00001000（有） 00001000（有） 00001000（有）
00000000（无） 00010000（有） 00010000（无） 00010000（有）
i = i + (i >>> 8)：00000000（无） 00010000（有） 00010000（无效） 00010000（有）

第五行：i = i + (i >>> 16);
00000000（无） 00010000（有） 00010000（无） 00010000（有）
00000000（无） 00000000（无） 00000000（无） 00010000 （有）
00000000（无） 00010000（无） 00010000（无） 00100000 （有）

第六行：i & 0x3f;
i=00000000（无） 00010000（无） 00010000（无） 00100000 （有）
0x3f:0011 0000 0000 0000 0000 0011 1111
i & 0x3f:最后计算的结果只有后6位二进制是有效结果：100000=32
 
```java
	public static int bitCount(int i) {
        // HD, Figure 5-2
        i = i - ((i >>> 1) & 0x55555555);
        i = (i & 0x33333333) + ((i >>> 2) & 0x33333333);
        i = (i + (i >>> 4)) & 0x0f0f0f0f;
        i = i + (i >>> 8);
        i = i + (i >>> 16);
        return i & 0x3f;
    }
```
## String转换成Integer ##
Integer的valueOf方法：如果缓存中有值，则读取缓存中的数字

Integer的parseInt方法：将字符串解析成Integer

Integer的decode方法： 字符串解码位数字。

valueOf和parseInt的参数必须是标准数字十进制

decode可以是各种形式的数据

```
System.out.println(parseInt("111"));
System.out.println(parseInt("111",10));
System.out.println(valueOf("111"));
System.out.println(valueOf("111",10));
System.out.println(valueOf(10));
System.out.println(Integer.decode("100"));//十进制
System.out.println(Integer.decode("0xa"));//十六进制
System.out.println(Integer.decode("0Xa"));//十六进制
System.out.println(Integer.decode("#a"));//十六进制
System.out.println(Integer.decode("010"));//八进制
System.out.println(Integer.decode("+100"));//正数
System.out.println(Integer.decode("-100"));//负数
```


```java
	public static int parseInt(String s) throws NumberFormatException {
        return parseInt(s,10);
    }
    
	public static int parseInt(String s, int radix)
                throws NumberFormatException {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */
		//s不能为空，进制数在Character.MIN_RADIX（2）和Character.MAX_RADIX（36）之间
        if (s == null) {
            throw new NumberFormatException("null");
        }
        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        boolean negative = false;//符号
        int i = 0;//偏移值
        int len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;
        if (len > 0) {
            char firstChar = s.charAt(0);
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);
                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            //一个字符一个字符累加
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                digit = Character.digit(s.charAt(i++),radix);
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix;
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;//通过负数减法计算加法
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;
    }
    
	public static Integer valueOf(String s, int radix) throws NumberFormatException {
        return Integer.valueOf(parseInt(s,radix));
    }
    
    public static Integer valueOf(String s) throws NumberFormatException {
        return Integer.valueOf(parseInt(s, 10));
    }
    
    public static Integer valueOf(int i) {
        assert IntegerCache.high >= 127;
        //判断数字是否有缓存
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
	public static Integer decode(String nm) throws NumberFormatException {
        int radix = 10;//进制数
        int index = 0;//偏移数
        boolean negative = false;//是否有符号
        Integer result;//最后结果

        if (nm.length() == 0)
            throw new NumberFormatException("Zero length string");
        char firstChar = nm.charAt(0);
        // Handle sign, if present
        if (firstChar == '-') {
            negative = true;//负数
            index++;
        } else if (firstChar == '+')
            index++;

        // Handle radix specifier, if present
        if (nm.startsWith("0x", index) || nm.startsWith("0X", index)) {
            index += 2;
            radix = 16;//16进制
        }
        else if (nm.startsWith("#", index)) {
            index ++;
            radix = 16;//16进制
        }
        else if (nm.startsWith("0", index) && nm.length() > 1 + index) {
            index ++;
            radix = 8;//8进制
        }

        if (nm.startsWith("-", index) || nm.startsWith("+", index))
            throw new NumberFormatException("Sign character in wrong position");

        try {
            result = Integer.valueOf(nm.substring(index), radix);//字符串转换成Integer
            result = negative ? Integer.valueOf(-result.intValue()) : result;//判断正负值
        } catch (NumberFormatException e) {
            // If number is Integer.MIN_VALUE, we'll end up here. The next line
            // handles this case, and causes any genuine format error to be
            // rethrown.
            String constant = negative ? ("-" + nm.substring(index))
                                       : nm.substring(index);
            result = Integer.valueOf(constant, radix);
        }
        return result;
    }
```

## Integer转换成String ##

toString(int i, int radix) 转换成指定进制的字符串

toString(int i) 转换成十进制的字符串

toHexString(int i) 转换成十六进制的字符串

toOctalString(int i) 转换成八进制的字符串

toBinaryString(int i) 转换成二进制的字符串

toUnsignedString(int i, int shift) 

int转2进制：`toUnsignedString（i,1）`

int转8进制：`toUnsignedString（i,3）`

int转16进制：`toUnsignedString（i,4）`
```java
 	public static String toString(int i, int radix) {
        if (radix < Character.MIN_RADIX || radix > Character.MAX_RADIX)
            radix = 10;
        /* Use the faster version */
        if (radix == 10) {
            return toString(i);
        }
        char buf[] = new char[33];
        boolean negative = (i < 0);
        int charPos = 32;
        if (!negative) {
            i = -i;
        }
        while (i <= -radix) {
            buf[charPos--] = digits[-(i % radix)];
            i = i / radix;
        }
        buf[charPos] = digits[-i];

        if (negative) {
            buf[--charPos] = '-';
        }
        return new String(buf, charPos, (33 - charPos));
    }

 	public static String toString(int i) {
        if (i == Integer.MIN_VALUE)
            return "-2147483648";
        int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
        char[] buf = new char[size];
        getChars(i, size, buf);
        return new String(buf, true);
    }
    
    public static String toHexString(int i) {
        return toUnsignedString(i, 4);
    }
    public static String toOctalString(int i) {
        return toUnsignedString(i, 3);
    }
     public static String toBinaryString(int i) {
        return toUnsignedString(i, 1);
    }
    private static String toUnsignedString(int i, int shift) {
        char[] buf = new char[32];//int型最大为32位
        int charPos = 32;//数组下标位置
        int radix = 1 << shift;//将0001左移，shift为偏移位，如16进制则偏移4位计算，8进制偏移3位计算
        int mask = radix - 1;//计算元数据的计算为，如果转16进制取后4位计算，mask为1111，与的结果为取后四位为有效位
        do {
            buf[--charPos] = digits[i & mask];
            i >>>= shift;//通过右移将计算过的位数舍弃掉，如10010100的后四位计算过了，则右移4位变成1001
        } while (i != 0);
        //将char转换成字符串，并舍弃前面的0
        return new String(buf, charPos, (32 - charPos));
    }
```
Interget.getChars:将数字转换成字符数组，第一个参数为要转换的数字，第二个参数为字符数组的长度，第三个参数为字符数组的引用。

```q = i / 100; r = i - ((q << 6) + (q << 5) + (q << 2))```

此处计算相当于r = i - (q * 100);即求i除以100之后的余数，i%100

`(int * 52429) >>> 19 === (int * 52429)/524288 === int / 10`

       
原理:
```
i - (q * (64 + 32 + 4)) 
==> i - (q * (64 + 32 + 4)) 
==> i - (q * 64 + q * 32 + q * 4)) 
==> i - (q << 6 + q << 5 + q << 2))
```
``` q = (i * 52429) >>> (16+3); r = i - ((q << 3) + (q << 1));  // r = i-(q*10)```

此处计算相当于r = i-(q*10);即求i除以10之后的余数，i%10


```java
    //该方法用于将数字转换成字符数组
	static void getChars(int i, int index, char[] buf) {
        int q, r;
        int charPos = index;
        char sign = 0;

        if (i < 0) {
            sign = '-';
            i = -i;
        }

        // Generate two digits per iteration
        //数字的二进制长度大于16位则每2位转换一次字符，
        while (i >= 65536) {
            q = i / 100; //q为数字去掉右边两位之后的数值
            // really: r = i - (q * 100);
            r = i - ((q << 6) + (q << 5) + (q << 2)); //余数
            i = q;
            buf [--charPos] = DigitOnes[r];
            buf [--charPos] = DigitTens[r];
        }

        // Fall thru to fast mode for smaller numbers
        // assert(i <= 65536, i);
        for (;;) {
            q = (i * 52429) >>> (16+3);
            r = i - ((q << 3) + (q << 1));  // r = i-(q*10)
            buf [--charPos] = digits [r];
            i = q;
            if (i == 0) break;
        }
        if (sign != 0) {
            buf [--charPos] = sign;
        }
    }
    //字符数组
    final static char[] digits = {
            '0' , '1' , '2' , '3' , '4' , '5' ,
            '6' , '7' , '8' , '9' , 'a' , 'b' ,
            'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
            'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
            'o' , 'p' , 'q' , 'r' , 's' , 't' ,
            'u' , 'v' , 'w' , 'x' , 'y' , 'z'
        };
   //数字字符数组。用于通过下标获取数据，该数字获取2位数中十位数值
   //如获取23十位数字为：DigitTens[23]=2
    final static char [] DigitTens = {
            '0', '0', '0', '0', '0', '0', '0', '0', '0', '0',
            '1', '1', '1', '1', '1', '1', '1', '1', '1', '1',
            '2', '2', '2', '2', '2', '2', '2', '2', '2', '2',
            '3', '3', '3', '3', '3', '3', '3', '3', '3', '3',
            '4', '4', '4', '4', '4', '4', '4', '4', '4', '4',
            '5', '5', '5', '5', '5', '5', '5', '5', '5', '5',
            '6', '6', '6', '6', '6', '6', '6', '6', '6', '6',
            '7', '7', '7', '7', '7', '7', '7', '7', '7', '7',
            '8', '8', '8', '8', '8', '8', '8', '8', '8', '8',
            '9', '9', '9', '9', '9', '9', '9', '9', '9', '9',
            } ;
	//数字字符数组。用于通过下标获取数据，该数字获取2位数中个位数值
    //如获取23个位数字为：DigitOnes[23]=3
    final static char [] DigitOnes = {
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
        } ;
```
Integer  highestOneBit和lowestOneBit
数字二进制中1出现的位置（最高位和最低位）
如：20=10100，
最高位：10000 = 16
最低位：100 = 4

按位与： & ：真真为真，其余为假：遇0为0
按位或： | ：假假为假，其余为真：遇1为1
按位非： ~ ：取反码
按位异或： ^ ：真假为真，其余为假：不同为1

highestOneBitd的原理

将二进制数的所有位数都转换为1，
然后用原数减去全为一的数右移一位后的数。
如10010 -- 11111 -- 11111 - 1111 = 10000

i |= (i >>  1);

i = 10100 | 10100 >> 1 = 10100 | 1010 = 11110
i = 11110 | 11110 >> 2 = 11110 | 111 = 11111
i = 11111 | 11111 >> 4 = 11111 | 1 = 11111
i = 11111 | 11111 >> 8 = 11111 | 0 = 11111
i = 11111 | 11111 >> 16 = 11111 | 0 = 11111
i = 11111 - (11111 >>> 1) = 11111 - 1111 = 10000

```java
    public static int highestOneBit(int i) {
        // HD, Figure 3-1
        i |= (i >>  1);//
        i |= (i >>  2);
        i |= (i >>  4);
        i |= (i >>  8);
        i |= (i >> 16);
        return i - (i >>> 1);
    }
    
    public static int lowestOneBit(int i) {
        // HD, Section 2-1
        return i & -i;
    }
```

