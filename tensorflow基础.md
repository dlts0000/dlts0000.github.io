#<center>tensorflow的模型训练Training相关函数
## 1.序言
本文所讲的内容主要为以下列表中相关函数。  
本文参考链接: [https://www.cnblogs.com/wuzhitj/p/6648641.html]()  
函数training()通过梯度下降法为最小化损失函数增加了相关的优化操作，在训练过程中，先实例化一个优化函数，比如 tf.train.GradientDescentOptimizer，并基于一定的学习率进行梯度优化训练：

	optimizer = tf.train.GradientDescentOptimizer(learning_rate)

然后，可以设置 一个用于记录全局训练步骤的单值。以及使用minimize()操作，该操作不仅可以优化更新训练的模型参数，也可以为全局步骤(global step)计数。与其他tensorflow操作类似，这些训练操作都需要在tf.session会话中进行

	global_step = tf.Variable(0, name='global_step', trainable=False)
	train_op = optimizer.minimize(loss, global_step=global_step)
主要操作：


| 操作组	 | 操作 |
| :-: | :-: | 
| Training | Optimizers，Gradient Computation，Gradient Clipping，Distributed execution | 
| Testing | Unit tests，Utilities，Gradient checking | 

## 2.Tensorflow函数
###2.1 训练 (Training)
一个TFRecords 文件为一个字符串序列。这种格式并非随机获取，它比较适合大规模的数据流，而不太适合需要快速分区或其他非序列获取方式

**优化 (Optimizers)**

tf中各种优化类提供了为损失函数计算梯度的方法，其中包含比较经典的优化算法，比如GradientDescent 和Adagrad。

**▶▶class tf.train.Optimizer**

| 操作	 | 描述 |
| :-- | :-- | 
|class `tf.train.Optimizer`	 | 基本的优化类，该类不常常被直接调用，而较多使用其子类，比如`GradientDescentOptimizer`, `AdagradOptimizer` 或者`MomentumOptimizer`|
|`tf.train.Optimizer.__init__(use_locking, name)`| 创建一个新的优化器，该优化器必须被其子类(subclasses)的构造函数调用|
|`tf.train.Optimizer.minimize(loss,`<br> `global_step=None, `<br>`var_list=None,` <br>`gate_gradients=1,`<br>` aggregation_method=None,`<br> `colocate_gradients_with_ops=False,`<br> `name=None, grad_loss=None)`| 添加操作节点，用于最小化loss，并更新var_list该函数是简单的合并了`compute_gradients()`与`apply_gradients()`函数返回为一个优化更新后的`var_list`，如果`global_step`非None，该操作还会为`global_step`做自增操作|
|`tf.train.Optimizer.compute_gradients(loss,`<br>`var_list=None, `<br>`gate_gradients=1,`<br>`aggregation_method=None, `<br>`colocate_gradients_with_ops=False, `<br>`grad_loss=None)`| 对var_list中的变量计算loss的梯度,该函数为函数minimize()的第一部分，返回一个以元组(gradient, variable)组成的列表|
|`tf.train.Optimizer.apply_gradients(grads_and_vars,`<br>` global_step=None,name=None)`|将计算出的梯度应用到变量上，是函数minimize()的第二部分，返回一个应用指定的梯度的操作Operation，对`global_step`做自增操作|
|`tf.train.Optimizer.get_name()`| 获取名称|

**class tf.train.Optimizer**  用法  

	# Create an optimizer with the desired parameters.
	opt = GradientDescentOptimizer(learning_rate=0.1)
	# Add Ops to the graph to minimize a cost by updating a list of variables.
	# "cost" is a Tensor, and the list of variables contains tf.Variable objects.
	opt_op = opt.minimize(cost, var_list=<list of variables>)
	# Execute opt_op to do one step of training:
	opt_op.run()
	
	
**▶▶在使用它们之前处理梯度**  
使用minimize()操作，该操作不仅可以计算出梯度，而且还可以将梯度作用在变量上。如果想在使用它们之前处理梯度，可以按照以下三步骤使用optimizer ：  
1、使用函数`compute_gradients()`计算梯度  
2、按照自己的愿望处理梯度  
3、使用函数`apply_gradients()`应用处理过后的梯度  

例如：

	# 创建一个optimizer.
	opt = GradientDescentOptimizer(learning_rate=0.1)
	
	# 计算<list of variables>相关的梯度
	grads_and_vars = opt.compute_gradients(loss, <list of variables>)
	
	# grads_and_vars为tuples (gradient, variable)组成的列表。
	#对梯度进行想要的处理，比如cap处理
	capped_grads_and_vars = [(MyCapper(gv[0]), gv[1]) for gv in grads_and_vars]
	
	# 令optimizer运用capped的梯度(gradients)
	opt.apply_gradients(capped_grads_and_vars)
	
**Slots**

一些optimizer的之类，比如 MomentumOptimizer 和 AdagradOptimizer 分配和管理着额外的用于训练的变量。这些变量称之为’Slots’，Slots有相应的名称，可以向optimizer访问的slots名称。有助于在log debug一个训练算法以及报告slots状态  

| 操作	 | 描述 |
| :-- | :-- | 
|`tf.train.Optimizer.get_slot_names()`|返回一个由Optimizer所创建的slots的名称列表|
|`tf.train.Optimizer.get_slot(var, name)`|返回一个name所对应的slot，name是由Optimizer为var所创建<br>var为用于传入 minimize() 或 apply_gradients()的变量|
|**class `tf.train.GradientDescentOptimizer`**|使用梯度下降算法的Optimizer|
|`tf.train.GradientDescentOptimizer.__init__(learning_rate,` <br>`use_locking=False, name=’GradientDescent’)`|构建一个新的梯度下降优化器(Optimizer)|
|**`class tf.train.AdadeltaOptimizer`**|使用Adadelta算法的Optimizer|
|`tf.train.AdamOptimizer.__init__(learning_rate=0.001,`<br>`beta1=0.9, beta2=0.999, epsilon=1e-08,`<br>`use_locking=False, name=’Adam’)`|创建Adam优化器|
|class `tf.train.FtrlOptimizer`|使用FTRL 算法的Optimizer|
|`tf.train.FtrlOptimizer.__init__(learning_rate,`<br> `learning_rate_power=-0.5,`<br> `initial_accumulator_value=0.1,`<br> `l1_regularization_strength=0.0,`<br> `l2_regularization_strength=0.0,`<br>`use_locking=False, name=’Ftrl’)`|创建FTRL算法优化器|
|class `tf.train.RMSPropOptimizer`|使用RMSProp算法的Optimizer|
|`tf.train.RMSPropOptimizer.__init__(learning_rate,`<br> `decay=0.9, momentum=0.0, epsilon=1e-10,`<br> `use_locking=False, name=’RMSProp’)`|创建RMSProp算法优化器|


**▷ tf.train.AdamOptimizer**

Adam 的基本运行方式，首先初始化：

`m_0 <- 0 (Initialize initial 1st moment vector)`  
`v_0 <- 0 (Initialize initial 2nd moment vector)`  
`t <- 0 (Initialize timestep)`   
在论文中的 section2 的末尾所描述了更新规则，该规则使用梯度g来更新变量：

`t <- t + 1`  
`lr_t <- learning_rate * sqrt(1 - beta2^t) / (1 - beta1^t)`  

`m_t <- beta1 * m_{t-1} + (1 - beta1) * g`  
`v_t <- beta2 * v_{t-1} + (1 - beta2) * g * g`  
`variable <- variable - lr_t * m_t / (sqrt(v_t) + epsilon)` 

其中`epsilon` 的默认值1e-8可能对于大多数情况都不是一个合适的值。例如，当在ImageNet上训练一个 Inception network时比较好的选择为1.0或者0.1。   
需要注意的是，在稠密数据中即便g为0时， `m_t`, `v_t` 以及variable都将会更新。而在稀疏数据中，`m_t`, `v_t` 以及variable不被更新且值为零。

**梯度计算与截断(Gradient Computation and Clipping)**

TensorFlow 提供了计算给定tf计算图的求导函数，并在图的基础上增加节点。优化器(optimizer )类可以自动的计算网络图的导数，但是优化器中的创建器(creators )或者专业的人员可以通过本节所述的函数调用更底层的方法。

| 操作	 | 描述 |
| :-- | :-- | 
| `tf.gradients(ys, xs, grad_ys=None, `<br>`name=’gradients’,`<br> `colocate_gradients_with_ops=False,`<br>` gate_gradients=False,`<br> `aggregation_method=None)`| 构建一个符号函数，计算ys关于xs中x的偏导的和,返回xs中每个x对应的sum(dy/dx)|
|`tf.stop_gradient(input, name=None)`|停止计算梯度，在EM算法、Boltzmann机等可能会使用到|
|`tf.clip_by_value(t, clip_value_min, `<br>`clip_value_max, name=None)`|基于定义的min与max对tesor数据进行截断操作，目的是为了应对梯度爆发或者梯度消失的情况|
|`tf.clip_by_norm(t, clip_norm,`<br>` axes=None, name=None)`|使用L2范式标准化tensor最大值为`clip_norm`,返回 `t * clip_norm / l2norm(t)`|
|`tf.clip_by_average_norm(t, clip_norm,`<br>` name=None)`|使用平均L2范式规范tensor数据t，并以`clip_norm`为最大值,返回 `t * clip_norm / l2norm_avg(t)`|
|`tf.clip_by_global_norm(t_list,`<br> 
`clip_norm, use_norm=None, name=None)`|返回`t_list[i] * clip_norm / max(global_norm, clip_norm)`其中`global_norm = sqrt(sum([l2norm(t)**2 for t in t_list]))`|
|`tf.global_norm(t_list, name=None)`|返回 `global_norm = sqrt(sum([l2norm(t)**2 for t in t_list]))`|

**退化学习率(Decaying the learning rate)**

| 操作	 | 描述 |
| :-- | :-- | 
|`tf.train.exponential_decay(learning_rate,`<br>`global_step,decay_steps, decay_rate,`<br>` staircase=False, name=None)`|对学习率进行指数衰退|

**▷ tf.train.exponential_decay**

	#该函数返回以下结果
	decayed_learning_rate = learning_rate *
	         decay_rate ^ (global_step / decay_steps)
	##例： 以0.96为基数，每100000 步进行一次学习率的衰退
	global_step = tf.Variable(0, trainable=False)
	starter_learning_rate = 0.1
	learning_rate = tf.train.exponential_decay(starter_learning_rate, global_step,
	                                           100000, 0.96, staircase=True)
	# Passing global_step to minimize() will increment it at each step.
	learning_step = (
	    tf.train.GradientDescentOptimizer(learning_rate)
	    .minimize(...my loss..., global_step=global_step)
	)
	
**移动平均(Moving Averages)**

一些训练优化算法，比如GradientDescent 和Momentum 在优化过程中便可以使用到移动平均方法。使用移动平均常常可以较明显地改善结果。

| 操作	 | 描述 |
| :-- | :-- | 
|class `tf.train.ExponentialMovingAverage`|将指数衰退加入到移动平均中|
|`tf.train.ExponentialMovingAverage.apply(var_list=None)`|对var_list变量保持移动平均|
|`tf.train.ExponentialMovingAverage.average_name(var)`|返回var均值的变量名称|
|`tf.train.ExponentialMovingAverage.average(var)`|返回var均值变量|
|`tf.train.ExponentialMovingAverage.variables_to_restore(`<br>`moving_avg_variables=None)`|返回用于保存的变量名称的映射|

**▷ tf.train.ExponentialMovingAverage**

	# Example usage when creating a training model:
	# Create variables.
	var0 = tf.Variable(...)
	var1 = tf.Variable(...)
	# ... use the variables to build a training model...
	...
	# Create an op that applies the optimizer.  This is what we usually
	# would use as a training op.
	opt_op = opt.minimize(my_loss, [var0, var1])
	
	# Create an ExponentialMovingAverage object
	ema = tf.train.ExponentialMovingAverage(decay=0.9999)
	
	# Create the shadow variables, and add ops to maintain moving averages
	# of var0 and var1.
	maintain_averages_op = ema.apply([var0, var1])
	
	# Create an op that will update the moving averages after each training
	# step.  This is what we will use in place of the usual training op.
	with tf.control_dependencies([opt_op]):
	    training_op = tf.group(maintain_averages_op)
	
	...train the model by running training_op...
	
	#Example of restoring the shadow variable values:
	# Create a Saver that loads variables from their saved shadow values.
	shadow_var0_name = ema.average_name(var0)
	shadow_var1_name = ema.average_name(var1)
	saver = tf.train.Saver({shadow_var0_name: var0, shadow_var1_name: var1})
	saver.restore(...checkpoint filename...)
	# var0 and var1 now hold the moving average values
	
**▷ tf.train.ExponentialMovingAverage.variables_to_restore**

	variables_to_restore = ema.variables_to_restore()
	saver = tf.train.Saver(variables_to_restore)
	
**协调器和队列运行器(Coordinator and QueueRunner)**

查看queue中，queue相关的内容，了解tensorflow中队列的运行方式。

| 操作	 | 描述 |
| :-- | :-- | 
|**`class tf.train.Coordinator`**|线程的协调器|
|`tf.train.Coordinator.clear_stop()`|清除停止标记|
|`tf.train.Coordinator.join(threads=None,`<br> `stop_grace_period_secs=120)`|等待线程终止threads:一个threading.Threads的列表，启动的线程，将额外加入到registered的线程中|
|`tf.train.Coordinator.register_thread(thread)`|Register一个用于join的线程|
|`tf.train.Coordinator.request_stop(ex=None)`|请求线程结束|
|`tf.train.Coordinator.request_stop(ex=None)`|请求线程结束|
|`tf.train.Coordinator.should_stop()`|检查是否被请求停止|
|`tf.train.Coordinator.stop_on_exception()`|上下文管理器，当一个例外出现时请求停止|
|`tf.train.Coordinator.wait_for_stop(timeout=None)`|等待Coordinator提示停止进程|
|`class tf.train.QueueRunner`|持有一个队列的入列操作列表，用于线程中运行<br>queue:一个队列<br>enqueue_ops: 用于线程中运行的入列操作列表|
|`tf.train.QueueRunner.create_threads(sess,`<br> 
`coord=None, daemon=False, start=False)`|创建运行入列操作的线程，返回一个线程列表|
|`tf.train.QueueRunner.from_proto(queue_runner_def)`|返回由`queue_runner_def`创建的QueueRunner对象|
|`tf.train.add_queue_runner(qr, collection=’queue_runners’)`|增加一个QueueRunner到graph的收集器(collection )中|
|`tf.train.start_queue_runners(sess=None, coord=None,`<br>` daemon=True,start=True, collection=’queue_runners’)`|启动所有graph收集到的队列运行器(queue runners)|

**▷ class tf.train.Coordinator**

	#Coordinator的使用，用于多线程的协调
	try:
	  ...
	  coord = Coordinator()
	  # Start a number of threads, passing the coordinator to each of them.
	  ...start thread 1...(coord, ...)
	  ...start thread N...(coord, ...)
	  # Wait for all the threads to terminate, give them 10s grace period
	  coord.join(threads, stop_grace_period_secs=10)
	except RuntimeException:
	  ...one of the threads took more than 10s to stop after request_stop()
	  ...was called.
	except Exception:
	  ...exception that was passed to coord.request_stop()
	  
**▷ tf.train.Coordinator.stop_on_exception()**

	with coord.stop_on_exception():
	  # Any exception raised in the body of the with
	  # clause is reported to the coordinator before terminating
	  # the execution of the body.
	  ...body...
	#等价于
	try:
	  ...body...
	exception Exception as ex:
	  coord.request_stop(ex)
	  
**汇总操作(Summary Operations)**

我们可以在一个session中获取summary操作的输出，并将其传输到SummaryWriter以添加至一个事件记录文件中。

| 操作	 | 描述 |
| :-- | :-- | 
|`tf.scalar_summary(tags, values,`<br>`collections=None, name=None)`|输出一个标量值的summary协议buffertag的shape需要与values的相同，用来做summaries的tags，为字符串|
|`tf.image_summary(tag, tensor,`<br>` max_images=3, collections=None, name=None)`|输出一个图像tensor的summary协议buffer|
|`tf.audio_summary(tag, tensor,`<br> `sample_rate, max_outputs=3,`<br> `collections=None, name=None)`|输出一个音频tensor的summary协议buffer|
|`tf.histogram_summary(tag, values,` <br>`collections=None, name=None)`|输出一个直方图的summary协议buffer|
|`tf.nn.zero_fraction(value, name=None)`|返回0在value中的小数比例|
|`tf.merge_summary(inputs,`<br> 
`collections=None, name=None)`|合并summary|
|`tf.merge_all_summaries(key=’summaries’)`|合并在默认graph中收集的summaries|

**▶▶将记录汇总写入文件中(Adding Summaries to Event Files)**

| 操作	 | 描述 |
| :-- | :-- | 
|`class tf.train.SummaryWriter`|将summary协议buffer写入事件文件中|
|`tf.train.SummaryWriter.__init__(logdir, `<br>`graph=None, max_queue=10,`<br> `flush_secs=120, graph_def=None)`|创建一个SummaryWriter实例以及新建一个事件文件|
|`tf.train.SummaryWriter.add_summary(summary,`<br>` global_step=None)`|将一个summary添加到事件文件中|
|`tf.train.SummaryWriter.add_session_log(session_log,`<br>` global_step=None)`|添加SessionLog到一个事件文件中|
|`tf.train.SummaryWriter.add_event(event)`|添加一个事件到事件文件中|
|`tf.train.SummaryWriter.add_graph(graph,`<br> `global_step=None, graph_def=None)`|添加一个Graph到时间文件中|
|`tf.train.SummaryWriter.add_run_metadata(run_metadata,`<br> `tag, global_step=None)`|为一个单一的session.run()调用添加一个元数据信息|
|`tf.train.SummaryWriter.flush()`|刷新时间文件到硬盘中|
|`tf.train.SummaryWriter.close()`|将事件问价写入硬盘中并关闭该文件|
|`tf.train.summary_iterator(path)`|一个用于从时间文件中读取时间协议buffer的迭代器|

**▷ tf.train.SummaryWriter**

创建一个SummaryWriter 和事件文件。如果我们传递一个Graph进入该构建器中，它将被添加到事件文件当中，这一点与使用add_graph()具有相同功能。   
TensorBoard 将从事件文件中提取该graph，并将其显示。所以我们能直观地看到我们建立的graph。我们通常从我们启动的session中传递graph：

	...create a graph...
	# Launch the graph in a session.
	sess = tf.Session()
	# Create a summary writer, add the 'graph' to the event file.
	writer = tf.train.SummaryWriter(<some-directory>, sess.graph)
	

**▷ tf.train.summary_iterator**

	#打印时间文件中的内容
	for e in tf.train.summary_iterator(path to events file):
	    print(e)
	
	#打印指定的summary值
	# This example supposes that the events file contains summaries with a
	# summary value tag 'loss'.  These could have been added by calling
	# `add_summary()`, passing the output of a scalar summary op created with
	# with: `tf.scalar_summary(['loss'], loss_tensor)`.
	for e in tf.train.summary_iterator(path to events file):
	    for v in e.summary.value:
	        if v.tag == 'loss':
	            print(v.simple_value)
	
	
