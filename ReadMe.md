# TensorFlow-Intel-Interview

## 使用TensorFlow来进行二次函数的拟合

为了初步了解TensorFlow的使用我先根据对tensorflow最基本的了解来写一个Demo，进行函数的拟合。然后再去看看Tensorflow Serving的事情。

我在Mac和Linux上面都进行了TensorFlow的安装。在Linux下还比较正常，但是在Mac下面因为“six”这个库的版本过低导致报了一些错。我升级了six这个模块，好像就好了。

```shell
sudo easy_install -U six
```

现在我们继续运行函数的拟合工作。

首先创建导入包。

```python
# -*- coding: utf-8 -*-
#设计一个直线拟合程序
import tensorflow as tf
import numpy as np
```

然后创造一个线性100位的随机数列矩阵作为x_data，并使用使用一个二次函数通过x_data求出对应的y_data。

```python
#首先生成一个训练集，主要就是100个点。
#生成x。这个是一个1-100的随机数列
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data*x_data + 2*x_data + 3

print x_data
print y_data
```

然后我们就可以打印出这两个线性矩阵。

```she
[ 0.81370246  0.42756972  0.22368494  0.06153482  0.42600977  0.18890364
  0.37369776  0.34966856  0.67164725  0.94311523  0.32742587  0.05987306
  0.67205924  0.88843513  0.34179601  0.06025261  0.77868128  0.67401105
  0.10349388  0.53170758  0.29022989  0.54501963  0.43309814  0.77815181
  0.75175041  0.00977294  0.35255209  0.29034185  0.08576855  0.52460378
  0.23889856  0.72486675  0.69800818  0.04585781  0.61279339  0.50740743
  0.68444997  0.34917989  0.84150833  0.60718584  0.92187607  0.06784791
  0.05042362  0.241505    0.10242676  0.75916296  0.69841975  0.73618746
  0.50816816  0.00163782  0.38006333  0.81526124  0.9681828   0.8107264
  0.20101748  0.21990283  0.09143824  0.11925177  0.1662218   0.12141241
  0.67629176  0.61114168  0.84703767  0.77604914  0.06363823  0.66954446
  0.9632895   0.48910257  0.81503087  0.48234224  0.83315003  0.87492347
  0.2759271   0.49929336  0.20392086  0.36211187  0.81115526  0.62664509
  0.66678047  0.73108047  0.13876791  0.24850345  0.60703003  0.81115836
  0.27531698  0.35072556  0.44352838  0.24446779  0.41860369  0.19653501
  0.90031695  0.73517466  0.41834632  0.36447218  0.32926932  0.46669638
  0.54500341  0.19840696  0.63673937  0.01909828]
[ 5.28951645  4.03795528  3.49740481  3.12685609  4.03350401  3.41349196
  3.88704538  3.82160521  4.79440451  5.77569675  3.76205945  3.12333083
  4.79578209  5.5661869   3.80041647  3.12413549  5.16370678  4.80231285
  3.21769881  4.34612799  3.66469312  4.38708591  4.05377007  5.16182375
  5.06862926  3.0196414   3.8293972   3.66498208  3.17889333  4.32441664
  3.53486967  4.97516537  4.88323164  3.09381866  4.60110235  4.27227688
  4.83737183  3.82028627  5.39115286  4.58304644  5.69360733  3.14029908
  3.10338974  3.54133463  3.21534467  5.09465408  4.88462973  5.01434708
  4.27457142  3.00327826  3.90457487  5.29517365  5.87374353  5.27873039
  3.44244289  3.48816299  3.19123745  3.25272465  3.36007333  3.25756574
  4.80995417  4.59577751  5.41154814  5.15435028  3.1313262   4.78737879
  5.85450554  4.2174263   5.29433727  4.19733858  5.3604393   5.51533794
  3.62799001  4.24788046  3.44942546  3.85534859  5.28028345  4.64597416
  4.77815723  4.99663973  3.29679227  3.55876088  4.58254528  5.28029442
  3.62643337  3.82445955  4.08377409  3.54870009  4.01243639  3.43169594
  5.61120462  5.01083088  4.01170635  3.86178446  3.76695681  4.15119839
  4.38703537  3.43617916  4.67891598  3.03856134]
```

然后我们使用这个训练集完成一个函数拟合的训练，我们这次使用的是虚构的训练集，我想，如果我们可以从外部获取训练集，那我们就可以做任何已知x与y离散点的拟合工作。完整程序：

```python
# -*- coding: utf-8 -*-
#设计一个函数拟合程序
import tensorflow as tf
import numpy as np


#首先生成一个训练集，主要就是100个点。
#生成x。这个是一个1-100的随机数列
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data*x_data + 2*x_data + 3

#print x_data
# print y_data

#开始创建tensorflow结构，A、B、C分别是二次函数的三个参数
#我们设定三个参数的初始值范围，以及维度
A= tf.Variable(tf.random_uniform([1],-2,2))
B= tf.Variable(tf.random_uniform([1],-3,3))
C= tf.Variable(tf.random_uniform([1],-4,4))

#使用上面定义的参数以及x_data组成带参的二次函数，这个y是就是一个y_data的预测值
y = A*x_data*x_data + B*x_data + C

#计算y与y_data相比的误差，reduce_mean计算出平局值，square计算的是y-ydata的平方，应该
loss = tf.reduce_mean(tf.square(y - y_data))

#创建一个优化器，GradientDescentOptimizer拥有一种非常基础的优化策略，梯度下降算法，形参应该是类似于预测步长之类的参数
#我觉得如果这里的学习效率越低应该需要的训练次数就越多，但是训练的就会越精准。如果这个值太大就会出问题的
opt = tf.train.GradientDescentOptimizer(0.1)

#创建一个训练器，使用梯度下降算法减少y_data与y之间的误差
trainer = opt.minimize(loss)

#在神经网络中初始化变量
init = tf.initialize_all_variables()

#这里tensorflow结构就建立完了

#创建一个会话，应该就是一个任务的意思
sess = tf.Session();

#使用初始化器激活这个训练任务
sess.run(init)

#从这里开始训练
for stepNum in range(100000):
    sess.run(trainer)#执行一次训练
    #每10部打印出来一个训练的结果
    if stepNum % 10 == 0:
        #run这个函数的意义应该就是在神经网络中执行一些东西，执行变量就会返回变量名
        print "stepNum =", stepNum ,sess.run(A),sess.run(B),sess.run(C)


```

通过我们的训练，我们就可以发现最后拟合的结果已经非常接近了。三个参数应该是1，2，3.这里已经非常接近，我们训练的步长定为0.1，训练了99990次

```shell
stepNum = 99990 [ 1.00010967] [ 1.99988365] [ 3.00002098]
```

而且我们发现16000次左右这个拟合的值就不变化了，我觉得是`tf.train.GradientDescentOptimizer()`形参值还是太大了导致的





