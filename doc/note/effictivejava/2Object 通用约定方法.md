Object的一些方法比如hashCode()会被覆盖，覆盖着要遵守这些方法本身的约定，否则就会影响使用这些约定的类，比如HashMap
## 覆盖equals
自反，对称，传递，一致，非空
Object以及覆盖不够用的时候才考虑自己覆盖
==来检查是否是自己，用instanceOf检查类型，类型强制转换，比较所有重要的域(基本类型用==，float,double用Float.compare, 数组用Arrays.equals等)
尽量不要考虑太复杂的相等逻辑
## 覆盖equals必须覆盖hashCode()
    hashCode的约束
同一个对象，没有更改equals用到的信息时返回的hashcode应该相等
不同对象equals返回ture，hashCode应该相等
不同对象equals返回false,hashcode应该尽量不相等，可以增加性能
    通常的办法：
设定基本值比如a=7
然后计算所有关键域，boolean f?1:0, 基本类型强转为int.
float,double 用Float.doubleToLongBits(f)
引用则调用引用的.hashCode.
数组Arrays.hashCodes()
循环调用 a = 31*a + c,c是上面计算的结果
最后return a;
    散列码计算开销会比较大，可以在第一次构建的时候缓存起来使用
## 覆盖toString()
    类应该是自描述的
    可以在javadoc中加入toString返回的格式，String.format("name='%s' age=%d", name, age);
## 谨慎的覆盖clone()
    Cloneable接口只是告诉编译器可以使用clone方法，如果没有实现这个接口则会报错，这个是一个不好的例子
是另外一种构造对象的办法，如果想在外面被调用，需要继承clone方法之后把可见性改为public
public TestB clone() throws CloneNotSupportedException {
        return (TestB) super.clone();
    } 
直接使用super的clone方法是有问题的，如果域中有引用存在，克隆对象会与原来对象使用同一个对上的内容。因此会涉及深度拷贝b.date = (Date) date.clone();  调用所有的clone方法。
不建议使用clone方法
## 考虑实现Comparable接口
     很多框架都是使用了Comparable来进行排序等逻辑的。比如Arrays.sort(),并且在treeMap中也使用了这个接口来判断唯一性