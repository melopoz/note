gc循环

minor 了还不够 导致oldgc

oldgc了还不够 导致 minor





wiki 新生代  survivor的阈值50%

> 同一年龄的对象  to big



分配担保机制其实就是 大对象直接分配到houmxxxxx  那个区的内部流程的一部分

新建出来的对象  太大了 s1放不下，要分配到 old。old会根据历史数据（以前这种情况放过来的对象的大小）判断：

> 如果能放 直接放到old
>
> 如果不能放，让新生代再minor gc试试

所以可能





怎么管理堆外内存

冰山对象

零拷贝