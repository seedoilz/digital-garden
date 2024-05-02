---
aliases: 
title: Hadoop序列化
date created: 2024-03-22 15:03:00
date modified: 2024-03-22 15:03:28
tags: [code/big-data]
---
### 为什么不用 Java 的[[序列化]]
Java 的序列化是一个**重量级序列化框架**（Serializable），一个对象被序列化后，会附带很多额外的信息（各种校验信息，Header，继承体系等），不便于在网络中高效传输。所以，[[Hadoop]] 自己开发了一套序列化机制（Writable）。

### Hadoop序列化特点
1. 紧凑 ：高效使用存储空间。
2. 快速：读写数据的额外开销小。
3. 互操作：支持多语言的交互

### 自定义bean 对象实现序列化接口（Writable）
在企业开发中往往常用的基本序列化类型不能满足所有需求，比如在Hadoop 框架内部传递一个bean 对象，那么该对象就需要实现序列化接口。
#### 步骤
1. 必须实现Writable 接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有**空参构造**
3. 重写**序列化**方法
4. 重写**反序列化**方法 
5. *注意反序列化的顺序和序列化的顺序完全一致*
6. 要想把结果显示在文件中，需要**重写 toString()**，可用"\t"分开，方便后续用。
7. 如果需要将自定义的bean 放在 key 中传输，则还需要实现Comparable 接口，因为MapReduce 框中的Shuffle 过程要求对 key 必须能排序。
```java
// 空参构造
public FlowBean() {
	super();
}
// 序列化
@Override
public void write(DataOutput out) throws IOException {
	out.writeLong(upFlow);
	out.writeLong(downFlow);
	out.writeLong(sumFlow);
}
// 反序列化
@Override
public void readFields(DataInput in) throws IOException {
	upFlow = in.readLong();
	downFlow = in.readLong();
	sumFlow = in.readLong();
}
// Comparable接口
@Override
public int compareTo(FlowBean o) {
	// 倒序排列，从大到小
	return this.sumFlow > o.getSumFlow() ? -1 : 1;
}
```
