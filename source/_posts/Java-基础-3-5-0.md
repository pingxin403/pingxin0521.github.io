---
title: Java 特殊集合类--BitSet
date: 2020-2-27 20:20:59
tags:
 - Java
 - 源码
categories:
 - Java
 - 基础
---

java.util.BitSet可以按位存储。

计算机中一个字节（byte）占8位（bit），我们java中数据至少按字节存储的，比如一个int占4个字节。如果遇到大的数据量，这样必然会需要很大存储空间和内存。

如何减少数据占用存储空间和内存可以用算法解决。java.util.BitSet就提供了这样的算法。

bitmap数据结构:<http://hanyunpeng0521.github.io/数据结构-1-2>

<!--more-->

**函数**

```
public void set(int pos): 位置pos的字位设置为true。 
public void set(int bitIndex, boolean value) 将指定索引处的位设置为指定的值。 
public void clear(int pos): 位置pos的字位设置为false。
public void clear() : 将此 BitSet 中的所有位设置为 false。 
public int cardinality() 返回此 BitSet 中设置为 true 的位数。 
public boolean get(int pos): 返回位置是pos的字位值。 
public void and(BitSet other): other同该字位集进行与操作，结果作为该字位集的新值。 
public void or(BitSet other): other同该字位集进行或操作，结果作为该字位集的新值。 
public void xor(BitSet other): other同该字位集进行异或操作，结果作为该字位集的新值。
public void andNot(BitSet set) 清除此 BitSet 中所有的位,set - 用来屏蔽此 BitSet 的 BitSet
public int size(): 返回此 BitSet 表示位值时实际使用空间的位数。
public int length() 返回此 BitSet 的“逻辑大小”：BitSet 中最高设置位的索引加 1。 
public int hashCode(): 返回该集合Hash 码， 这个码同集合中的字位值有关。 
public boolean equals(Object other): 如果other中的字位同集合中的字位相同，返回true。 
public Object clone() 克隆此 BitSet，生成一个与之相等的新 BitSet。 
public String toString() 返回此位 set 的字符串表示形式。
```

**尝试**

通过BitSet来记录26个字母的使用情况，通过后期索引即可轻松得到对应值为1（True）的索引号。

前期字符串转ASCII，改变对应BitSet的值。

最后再ASCII转字符串，将其输出

```
public class BitSet_Text {
    public static void main(String[] args) {
        /*创建一个大小没有被指定的位组*/
        /*新位组中的所有位都被初始化为false*/
        BitSet bitSet=new BitSet();
        
        
        /*检测一个单词用了几个字母*/
        Scanner scanner=new Scanner(System.in);
        String word=scanner.nextLine();
        char[] getchar=word.toCharArray();
        
        for(int i=0;i<word.length();i++){
            int index=getchar[i];
            if(index>=97&&index<=122)
            {
                //小写
                int newindex=index-96;
                bitSet.set(newindex);
            }
            else {
                if(index>=65&&index<=90)
                {
                    //大写
                    int newindex=index-64;
                    bitSet.set(newindex);
                }
                else {
                    System.out.println("存在非法字符");
                }
                
            }
        }
        
        System.out.print("包含的字母有：");
        for(int j=0;j<bitSet.size();j++)
        {
            
            
            boolean cout=bitSet.get(j);
            if(cout==true)
            {
                int y=96+j;
                char x=(char)y;
                System.out.print(x );
                
            }
        }

    }
}
```

