# RANDOM
## Constant Value Tensors
```
tf.zeros
tf.zeros_like
tf.ones
tf.ones_like
tf.fill
tf.constant
```
## Sequences
```
tf.linspace
tf.range
```

## Random Tensor
Create random tensors with different distributions. The random ops are stateful, and create new random values each time they are
evaluated.
and the seed keyword argment in these function acts in conjunction with the graph level random seed
```
tf.set_random_seed
```
Some conventional distributions,such as normal(Gaussian distribution), truncated_normal, uniform, multinomial, gamma
```
tf.random_normal
tf.truncated_normal
tf.random_uniform
tf.random_shuffle
tf.random_crop                        ---> crop for image input
tf.multinomial
tf.random_gamma
```

### Examples
```
norm = tf.random_normal([2, 3], mean=-1, stddev=4)
sess.run(norm)
Out[135]: 
array([[ 1.04485011, -3.27299333, -1.52553272],
       [-0.24643952, -9.83152676, -0.71526629]], dtype=float32)
       
       
c = tf.constant([[1, 2], [3, 4], [5, 6]])
shuff = tf.random_shuffle(c)
sess.run(shuff)
Out[138]: 
array([[1, 2],
       [5, 6],
       [3, 4]])
sess.run(c)
Out[141]: 
array([[1, 2],
       [3, 4],
       [5, 6]])
       
print(sess.run(norm))
print(sess.run(norm))
[[-0.01499939  0.16252124  2.79371309]
 [-1.38953209  8.84682083 -3.50333261]]
[[-1.14327836 -1.55230355 -1.63681424]
 [-1.5096972   7.47023201 -1.91704535]]
norm = tf.random_normal([2, 3], seed=1234)
sess = tf.Session()
print(sess.run(norm))
print(sess.run(norm))
2018-01-13 16:37:21.795303: I c:\tf_jenkins\home\workspace\release-win\device\gpu\os\windows\tensorflow\core\common_runtime\gpu\gpu_device.cc:977] Creating TensorFlow device (/gpu:0) -> (device: 0, name: GeForce GTX 1080, pci bus id: 0000:06:00.0)
[[ 0.51340485 -0.25581399  0.65199131]
 [ 1.39236379  0.37256798  0.20336303]]
[[ 0.96462417  0.34291974  0.24251089]
 [ 1.05785966  1.65749764  0.82108974]]
 
 
var = tf.Variable(tf.random_uniform([2, 3]), name="var")
sess.run(var)
tensorflow.python.framework.errors_impl.FailedPreconditionError: Attempting to use uninitialized value var
	 [[Node: var/_2 = _Send[T=DT_FLOAT, client_terminated=false, recv_device="/job:localhost/replica:0/task:0/cpu:0", send_device="/job:localhost/replica:0/task:0/gpu:0", send_device_incarnation=1, tensor_name="edge_1_var", _device="/job:localhost/replica:0/task:0/gpu:0"](var)]]
	 [[Node: var/_3 = _Recv[_start_time=0, client_terminated=false, recv_device="/job:localhost/replica:0/task:0/cpu:0", send_device="/job:localhost/replica:0/task:0/gpu:0", send_device_incarnation=1, tensor_name="edge_1_var", tensor_type=DT_FLOAT, _device="/job:localhost/replica:0/task:0/cpu:0"]()]]
init = tf.global_variables_initializer()
sess.run(init)
sess.run(var)
Out[148]: 
array([[ 0.34862542,  0.64267635,  0.15603614],
       [ 0.57963645,  0.43195641,  0.6803081 ]], dtype=float32)


```