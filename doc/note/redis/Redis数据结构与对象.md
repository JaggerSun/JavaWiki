Redis中的key和value都是由对象构成，
key总是字符串对象，值可能是String, list, hash, set, 有序集合
讨论这些对象的数据结构，及对性能的影响

SDS Simple Dynamic
what 是什么
用来代替C里面的字符串的
where 用在哪儿
    作为key, 值中的字符串类型，以及各种缓冲区
why 为啥用
    因为比c原生的字符串好：
O(1) 获取长度
杜绝缓冲区溢出
减少修改字符串长度时所需的内存重新分配次数
二进制安全
兼容部分C字符串函数
how 怎么做到的
这个对照着Java中的StringBuffer就比较好理解了


