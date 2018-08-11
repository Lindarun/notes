[TOC]
# TensorFlow基本使用
##### 解决Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2问题
只需要在开头加上这一段代码即可
```
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'
```
## 常量，变量，数据类型
TensorFlow用张量这种数据结构表示所有的数据。可以把一个张量想象成一个n维数组组成的列表，而一个张量有一个静态类型和动态类型的维数，张量可以在图中的节点之间流通   
常量的创建方法
```
hello = tf.constant("Hello, tensorflow!", dtype=tf.string)
a = tf.constant(1)
```
其中**tf.string**是常量类型，平时编写可以忽略。   
变量的创建方法
```
a = tf.Variable(1.0, dtype=tf.float32)
b = tf.Variable(1.0, dtype=tf.float64)
```
编写以下程序
```
input1 = tf.constant(1)
print(input1)
input2 = tf.Variable(2, dtype=tf.int32)
print(input2)
input2 = input1
sess = tf.Session()
print(sess.run(input2))
```
结果如下
```
Tensor("Const:0", shape=(), dtype=int32)
<tf.Variable 'Variable:0' shape=() dtype=int32_ref>
1
```
第一个打印结果是一个张量而不是一个具体的数值。对于input2来说，此时仍旧是read：0的状态，表示虽然被赋予了值，但是并没有发生真实值的改变。而当只有**会话**中执行过，才能够真正发生真实值的改变。   
TensorFlow还存在一种特殊的数据类型**占位符(placeholder)**，因为其特殊的数据计算和处理形式，图进行计算时，可以从外界传入数值，而TensorFlow并不能直接对传入的数据进行处理，因此使用placeholder保留一个空位，之后会话运行时进行赋值。
```
input1 = tf.placeholder(tf.float32)
```
若传入的数据类型不符会报错。下面演示用占位符进行输出的例子
```
input1 = tf.placeholder(tf.int32)
input2 = tf.placeholder(tf.int32)
output = tf.add(input1, input2)
sess = tf.Session()
print(sess.run(output, feed_dict={input1: [1], input2: [2]}))
```
> Feeding_dict函数用于对占位符传递数据，是一种机制。允许你在运行时使用不同的值替换一个或多个tensor值。

一些具体的计算函数看书100页。注意以下一些函数api已被废弃
```
tf.sub -> tf.subtract
tf.mul -> tf.multiply
tf.neg -> tf.negative
```
从上可得，TensorFlow是以图的形式在对话中统一运行，用户建立复杂的表达式来形成计算图，之后将整个计算图的运行过程交给一个Tensorflow对话，此对话可以运行整个计算过程，提高效率。
## 矩阵运算
创建一个常张量矩阵
```
tf.constant([1,2,3], shape=[2,3)
```
输出以下结果
```
[[1,2,3],[3,3,3]
```
此处自动生成了一个符合要求的矩阵，这是一种优化结果。   
生成随机矩阵张量
```
tf.random_normal(shape,mean=0.0,stddev=1.0,dtype=tf.float32,seed=None,name=None)
tf.truncated_normal(shape,mean=0.0,stddev=1.0,dtype=tf.float32,seed=None,name=None)
tf.random_uniform(shape,minval=0,maxval=None,dtype=tf.float32,seed=None,name=None)
```
- random_normal: 正态分布随机数，均值mean，标准差stddev
- truncated_normal: 截断正态分布随机数，只保留[mean-2\*stddev, mean+2*stddev]的随机数
- random_uniform:均匀分布随机数
```
tf.shape(Tensor)
```
返回张量的形状   
tf.reshape(tensor, shape, name=None)用法
- shape=[-1]:表示flatten
- shape=[a, -1, c, d, ...]: 这表示b将自动计算
## TensorFlow数据的生成与读取详解
### 队列
通过先进先出的线性数据结构，一端只负责增加数据，另一端则负责输出和删除。
#### 队列创建
详细函数参考书138页。   
一般而言创建一个队列首先要选定数据出入类型，例如是**FIFOQueue**的先入先出模式，还是**RandomShuffleQueue**这种随机元素出列的方式。
```
q = tf.FIFOQueue(3, "float")
```
函数第一个参数是数据个数，第二个是数据类型。   
记得操作要由会话完成
```
sess = tf.Session()
init = q.enqueue_many(([0.1, 0.2, 0.3],)
sess.run(init)
```
输入以下程序
```
with tf.Session() as sess:
    q = tf.FIFOQueue(3, "float")
    init = q.enqueue_many(([0.1, 0.2, 0.3],))
    init2 = q.dequeue()
    init3 = q.enqueue(1.)

    sess.run(init)
    sess.run(init2)
    sess.run(init3)

    queue_len = sess.run(q.size())
    for i in range(queue_len):
        print(sess.run(q.dequeue()))

```
结果为
```
0.2
0.3
1.0
```
从中可看见队列的操作是在主线程的对话中依次完成，这样做的好处是不易堵塞队列，但这样子速度也会较慢。   
tf提供了QueueRunner函数来解决异步操作的问题。运行以下程序
```
with tf.Session() as sess:
    q = tf.FIFOQueue(1000, "float")
    counter = tf.Variable(0.0)
    add_op = tf.assign_add(counter, tf.constant(1.0))
    enqueue_data_op = q.enqueue(counter)

    qr = tf.train.QueueRunner(q, enqueue_ops=[add_op, enqueue_data_op] * 2)
    sess.run(tf.global_variables_initializer())
    enqueue_threads = qr.create_threads(sess, start=True)  # 启动入队线程

    for i in range(10):
        print(sess.run(q.dequeue()))

```
首先这里创建了一个数据处理函数，add_op的操作是将1叠加到counter上，为了执行这个qr创建了一个队列管理器，其调用了2个线程去完成这个任务，create_threads函数是对线程启动，此时线程开始运行，而在for中主程序同时也对队列不断弹出数据。将得到以下结果
```
25.0
92.0
120.0
190.0
244.0
289.0
340.0
383.0
410.0
ERROR:tensorflow:Exception in QueueRunner: Session has been closed.
ERROR:tensorflow:Exception in QueueRunner: Session has been closed.
ERROR:tensorflow:Exception in QueueRunner: Session has been closed.
ERROR:tensorflow:Exception in QueueRunner: Session has been closed.
458.0
Exception in thread QueueRunnerThread-fifo_queue-fifo_queue_enqueue:
Traceback (most recent call last):
  File "D:\anaconda\lib\threading.py", line 916, in _bootstrap_inner
    self.run()
  File "D:\anaconda\lib\threading.py", line 864, in run
    self._target(*self._args, **self._kwargs)
  File "D:\anaconda\lib\site-packages\tensorflow\python\training\queue_runner_impl.py", line 252, in _run
    enqueue_callable()
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1244, in _single_operation_run
    self._call_tf_sessionrun(None, {}, [], target_list, None)
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1409, in _call_tf_sessionrun
    run_metadata)
tensorflow.python.framework.errors_impl.CancelledError: Session has been closed.
Exception in thread QueueRunnerThread-fifo_queue-fifo_queue_enqueue:
Traceback (most recent call last):
  File "D:\anaconda\lib\threading.py", line 916, in _bootstrap_inner
    self.run()
  File "D:\anaconda\lib\threading.py", line 864, in run
    self._target(*self._args, **self._kwargs)
  File "D:\anaconda\lib\site-packages\tensorflow\python\training\queue_runner_impl.py", line 252, in _run
    enqueue_callable()
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1244, in _single_operation_run
    self._call_tf_sessionrun(None, {}, [], target_list, None)
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1409, in _call_tf_sessionrun
    run_metadata)
tensorflow.python.framework.errors_impl.CancelledError: Session has been closed.
Exception in thread QueueRunnerThread-fifo_queue-AssignAdd:0:
Traceback (most recent call last):
  File "D:\anaconda\lib\threading.py", line 916, in _bootstrap_inner
    self.run()
  File "D:\anaconda\lib\threading.py", line 864, in run
    self._target(*self._args, **self._kwargs)
  File "D:\anaconda\lib\site-packages\tensorflow\python\training\queue_runner_impl.py", line 252, in _run
    enqueue_callable()
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1254, in _single_tensor_run
    results = self._call_tf_sessionrun(None, {}, fetch_list, [], None)
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1409, in _call_tf_sessionrun
    run_metadata)
tensorflow.python.framework.errors_impl.CancelledError: Session has been closed.



Exception in thread QueueRunnerThread-fifo_queue-AssignAdd:0:
Traceback (most recent call last):
  File "D:\anaconda\lib\threading.py", line 916, in _bootstrap_inner
    self.run()
  File "D:\anaconda\lib\threading.py", line 864, in run
    self._target(*self._args, **self._kwargs)
  File "D:\anaconda\lib\site-packages\tensorflow\python\training\queue_runner_impl.py", line 252, in _run
    enqueue_callable()
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1254, in _single_tensor_run
    results = self._call_tf_sessionrun(None, {}, fetch_list, [], None)
  File "D:\anaconda\lib\site-packages\tensorflow\python\client\session.py", line 1409, in _call_tf_sessionrun
    run_metadata)
tensorflow.python.framework.errors_impl.CancelledError: Session has been closed.


Process finished with exit code 0

```
提示为队列管理器企图关闭会话，循环已经结束了，绘画要关闭。如果换一种表达方式
```
q = tf.FIFOQueue(1000, "float")
counter = tf.Variable(0.0)
add_op = tf.assign_add(counter, tf.constant(1.0))
enqueue_data_op = q.enqueue(counter)

sess = tf.Session()
qr = tf.train.QueueRunner(q, enqueue_ops=[add_op, enqueue_data_op] * 2)
sess.run(tf.global_variables_initializer())
enqueue_threads = qr.create_threads(sess, start=True)  # 启动入队线程

for i in range(10):
    print(sess.run(q.dequeue()))

```
可以看到这里并没有报错，但也没有结束。由于入队线程自顾自地执行，在需要的出队操作完成之后，程序没法结束。造成程序被挂起。
#### 线程同步与停止
可以从上看见TensorFlow会话支持多线程，但是这种并行同步会造成某个线程想要关闭对话时，对话被强行关闭而未完成的工作的线程也被强行关闭。   
tf为了解决这个，提供了**Coordinator和QueueRunner**函数来对线程进行控制和协调。在使用上这两个类必须同时工作，共同协作来停止会话中所有的线程，**并向等待所有工作线程终止的程序报告**。

```
q = tf.FIFOQueue(1000, "float")
counter = tf.Variable(0.0)
add_op = tf.assign_add(counter, tf.constant(1.0))
enqueue_data_op = q.enqueue(counter)

sess = tf.Session()
qr = tf.train.QueueRunner(q, enqueue_ops=[add_op, enqueue_data_op] * 2)
sess.run(tf.global_variables_initializer())

coord = tf.train.Coordinator()
enqueue_threads = qr.create_threads(sess, coord=coord, start=True)  # 启动入队线程
for i in range(10):
    print(sess.run(q.dequeue()))

coord.request_stop()
coord.join(enqueue_threads)

```

在函数create_thread被添加了一个新的参数：线程协调器，用于协调线程之间的关系，程序正常运行。
#### 队列中数据的读取
1. 从磁盘读取数据的名称与路径
2. 将文件名堆入队列尾部
3. 从队列头部读取文件名并读取数据
4. Decoder将读取的数据编码
5. 将数据输入样本队列，供后续使用

### csv文件的创建与读取
#### csv文件创建
属于python范畴。
#### csv文件读取
对于tf使用csv，需要特殊的csv读取方法。
### TF文件的创建与读取
pass
