## Blob

Blob表示原生二进制数据对象。
- Blob
- File
- ArrayBuffer
- DataView

#### 类型化数组(Typed Array)

JavaScript`类型化数组`是一种类似数组的对象，并提供了一种用于访问原始二进制数据的机制。`Typed Array`分成两部分
- 缓冲 ( ArrayBuffer )
- 视图 ( Int8、Unit32、Float64等等 )

**类型化数组的缓冲**

缓冲描述的是一个数据块，用ArrayBuffer定义的是类数组的对象，长度由byteLength(字节长度，其中一字节8bit)

**类型化数组的视图**

缓冲一般不可以操作里面的内容，要操作缓冲中的内容，要用到视图。视图指的是：具有描述性的名字和所有常用的数值类型像Int8、Uint32、Float64等等

![模型图](https://mdn.mozillademos.org/files/8629/typed_arrays.png)

ArrayBuffer是堆栈的内存结构(数据是存在内存栈中，比普通数组更快)且单位是字节(byte)，简单说就是开一固定长度大小的内存区，要存数据的时候就用它（装货区的仓库）。

1字节(byte) = 8位(bit)

Uint8Array的单位是bit 所以一个Unit8元素是一字节(byte)。ArrayBuffer(16)可以存长度为16的Unit8Array。简单理解就是装数据的载体，就是装货物的集装箱。

ArrayBuffer和Unit8Array配合使用就是仓库和集装箱的搭配。数据存于集装箱中，集装箱放在仓库中。当然也可以将所有数据直接放到仓库中，不使用集装箱，但是如果你要把数据从仓库中取出来，就要用到集装箱了，用集装箱收集仓库中的数据。那么一个16bytes的仓库，要用多少个集装箱呢？集装箱有分类型的，比如Uint8Array就是代表容量为1byte的集装箱，那么要全部取出仓库中的数据，就要用8个集装箱。

**DataView**

DataView是一种底层接口，可操作缓冲区中任意数据的读写接口。
