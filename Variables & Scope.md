# Variable
## variable 
```
tf.Variable
```
## variable helper functions
provide  a set of functions to help manage the set of variables collected in the graph
```
tf.global_variables                                  
tf.local_variables      
tf.model_variables
tf.trainable_variables
tf.moving_average_variables
tf.global_variables_initializer
tf.local_variables_initializer
tf.variables_initializer
tf.is_variable_initialized
tf.report_uninitialized_variables
tf.assert_variables_initialized
tf.assign
tf.assign_add
tf.assign_sub   
```
tf.assign update 'ref' by assigning 'value' to it. We must highlight that tf.assign is an operation, used to reset value.
```
assign(
    ref,
    value,
    validate_shape=None,
    use_locking=None,
    name=None
)

```
```
import tensorflow as tf
var = tf.Variable(0.9)
sess = tf.Session()
sess.run(tf.global_variables_initializer())
print(sess.run(var2))
0.9
op = tf.assign(var, 2)
sess.run(op)
print(sess.run(var))
2.0
```

## Saving and Restore variables
```
tf.train.Saver
tf.train.latest_checkpoint
tf.train.get_checkpoint_state
tf.train.update_checkpoint_state
```
## Sharing Variables
Create variables contingent on certain conditions
```
tf.get_variable
tf.get_local_variable
tf.VariableScope
tf.variable_scope
tf.variable_op_scope
tf.get_variable_scope
tf.make_template
tf.no_regularizer
tf.constant_initializer
tf.random_normal_initializer
tf.truncated_normal_initializer
tf.random_uniform_initializer
tf.uniform_unit_scaling_initializer
tf.zeros_initializer
tf.ones_initializer
tf.orthogonal_initializer
```
## Variable Partitioners for Sharding
```
tf.fixed_size_partitioner
tf.variable_axis_size_partitioner
tf.min_max_variable_partitioner
```
## Sparse Variable Updates
```
tf.IndexedSlices
```
## Read-only Lookup Tables
```
tf.initialize_all_tables
tf.tables_initializer
```
## Exporting and Importing Meta Graphs
```
tf.train.export_meta_graph
tf.train.import_meta_graph
```

# tf.Placeholder
```
placeholder(
    dtype,
    shape=None,
    name=None
)

constant(
    value,                    ---> not a tensor, specific value
    dtype=None,
    shape=None,
    name='Const',
    verify_shape=False
)
```
Insert a placeholder for a tensor that will be always fed
```
x = tf.placeholder(tf.float32, shape=(2, 2))
y = tf.matmul(x, x)

with tf.Session() as sess:
  print(sess.run(y))  # ERROR: will fail because x was not fed.
  print(sess.run(y, feed_dict={x: [[1,2],[1,3]]}))  # Will succeed.
[[  3.   8.]
 [  4.  11.]]

```
return a tensor that may be used as a handle for feeding a value, but not evaluated directly

# tf.Variable
a variable maintains state in the graph across calls to run()
the Variable constructor requires an initial value for the variable, After construction the type and shape are fixed
,The value can be changed using one of the assign methods.
```
tf.Variable(<initial-value>, name=<optional-name>)
```
variable can be used as inputs of other Ops in graph
```
import tensorflow as tf
# Create a variable.
w = tf.Variable(tf.random_normal((2,3)), name='W')
x = tf.constant([[1,2],[3,4],[5,6]])
z = tf.matmul(w,x)

TypeError: Input 'b' of 'MatMul' Op has type int32 that does not match type float32 of argument 'a'.

x = tf.constant([[1,2],[3,4],[5,6]],dtype=tf.float32) # add data type information

# Use the variable in the graph like any Tensor.
z = tf.matmul(w,x)

# Assign a new value to the variable with `assign()` or a related method.
w.assign(w+1.0)
Out[161]: 
<tf.Tensor 'Assign:0' shape=(2, 3) dtype=float32_ref>

```



Variables **have to be explicitly initialized** before you run Ops that use their value by:
* running it initializer Op
* restoring the variable from a save file
* simply running an assign Op
```
# Launch the graph in a session.
with tf.Session() as sess:
    # Run the variable initializer.
    sess.run(w.initializer)
    print(sess.run(w))
    # ...you now can run ops that use the value of 'w'...
[[ 0.4880335   0.07506625  2.16401887]
 [ 0.32055953 -0.2380942  -0.56767261]]
```

The most common initialization pattern is to use the convenience function global_variables_initializer() 
that initializes all the variables.
```
# Add an Op to initialize global variables.
init_op = tf.global_variables_initializer()

# Launch the graph in a session.
with tf.Session() as sess:
    # Run the Op that initializes global variables.
    sess.run(init_op)
    # ...you can now run any Op that uses variable values...
    print(sess.run(w))
[[-1.39853573  1.05518651  0.23997593]
 [ 0.98568857 -0.71756804 -1.31912315]]

```
All variables are automatically collected in the graph. The graph collection *GraphKeys.GLOBAL_VARIABLES*.
the convenience function *global_variables* return the contents of the collection

Method
```

__init__(
    initial_value=None,
    trainable=True,
    collections=None,
    validate_shape=True,
    caching_device=None,
    name=None,
    variable_def=None,
    dtype=None,
    expected_shape=None,
    import_scope=None
)
```
## tf.Variable Vs tf.get_variable 
the difference between the above two method is that *tf.variable* is the default operation for making a variable and *get_variable*
is mainly used for weight sharing.  
tf.Variable.\__init\__: Creates a new variable with initial_value.
```
W = tf.Variable(<initial-value>, name=<optional-name>)
```
tf.get_variable: Gets an existing variable with these parameters or create a new one. You can also use initializer.
```
W = tf.get_variable(name, shape=None, dtype=tf.float32, initializer=None,
       regularizer=None, trainable=True, collections=None)
```
It's very useful to use initializers such as xavier_initializer:
```
W = tf.get_variable("W", shape=[784, 256],
       initializer=tf.contrib.layers.xavier_initializer())
```
There is a limit in the initialization. you can not use the higher lever operation, for example,xavier. but you can just create it.
and there is no more ExeptionError raised.
```
with tf.variable_scope('op'):
    conv1_biases = tf.Variable(tf.zeros([32]), name="conv1_biases")
    conv2_biases = tf.Variable(tf.zeros([32]), name="conv1_biases")
    
conv1_biases
Out[13]: 
<tf.Variable 'op_1/conv1_biases:0' shape=(32,) dtype=float32_ref>
conv2_biases
Out[14]: 
<tf.Variable 'op_1/conv1_biases_1:0' shape=(32,) dtype=float32_ref>
```
We face two choice for the tf.get_variable: weight share if reuse = true or raise a value error if reuse = False.
So how to change the situation. First same scope different variable name or different scope same name.
```
with tf.variable_scope('op'):
    conv1_biases = tf.get_variable(name="conv1_biases", shape=[32],
                                   initializer=tf.contrib.layers.variance_scaling_initializer())
    conv2_biases = tf.get_variable(name="conv1_biases", shape=[32],
                                   initializer=tf.contrib.layers.variance_scaling_initializer())
ValueError: Variable op/conv1_biases already exists, disallowed. Did you mean to set reuse=True in VarScope? Originally defined at:

```


# Sharing variables and variable_scope
--------------------------------------------------------------------------------------------
Today, we will talk something about sharing variables . It can be a very complex problem,however ,it is actually neccessary to know that,for when building complex models , you ofen need to share large of variables and you might want to initialize all of them in one place
so this tutorial show how this can be done using the following functions 
```
tf.variable_scope                         ---> Carry a name that will be used as a prefix for variables names and a reuse flag to         
                                               Distinguish the function of get_variable
tf.get_variable                           ---> Create new variables or reusing variables
tf.reset_default_graph()                  ---> Clears the default graph stack and resets the global default graph
tf.get_variable_scope()                   ---> Retrieve the current scope
tf.get_variable_scope.reuse_variables()   ---> Set reuse flag to be true
```
## tf.get_variable
Gets an existing variable with these parameters
or create a new one

```
get_variable(
    name,
    shape=None,
    dtype=None,
    initializer=None,
    regularizer=None,
    trainable=True,
    collections=None,
    caching_device=None,
    partitioner=None,
    validate_shape=True,
    use_resource=None,
    custom_getter=None
)

```
The function prefixes the name with current variable scope and perform reuse checks
```
with tf.variable_scope("foo"):
    v = tf.get_variable("v", [1])  # v.name == "foo/v:0"
    w = tf.get_variable("w", [1])  # w.name == "foo/w:0"
with tf.variable_scope("foo", reuse=True):
    v1 = tf.get_variable("v")  # The same as v above.
```
```
v1
Out[181]: 
<tf.Variable 'foo/v:0' shape=(1,) dtype=float32_ref>
```

## Necessity
```
import tensorflow as tf
def my_image_filter(input_images):
    conv1_weights = tf.Variable(tf.random_normal([5, 5, 32, 32]),
        name="conv1_weights")
    conv1_biases = tf.Variable(tf.zeros([32]), name="conv1_biases")
    conv1 = tf.nn.conv2d(input_images, conv1_weights,
        strides=[1, 1, 1, 1], padding='SAME')
    relu1 = tf.nn.relu(conv1 + conv1_biases)

    conv2_weights = tf.Variable(tf.random_normal([5, 5, 32, 32]),
        name="conv2_weights")
    conv2_biases = tf.Variable(tf.zeros([32]), name="conv2_biases")
    conv2 = tf.nn.conv2d(relu1, conv2_weights,
        strides=[1, 1, 1, 1], padding='SAME')
    return tf.nn.relu(conv2 + conv2_biases)
    
img1 = tf.placeholder(tf.float32,shape=(100,32,32,32))
result1 = my_image_filter(img1)
result2 = my_image_filter(img1)
```
```
tf.trainable_variables()
Out[10]: 
[# group1
<tf.Variable 'conv1_weights:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv1_biases:0' shape=(32,) dtype=float32_ref>,
 <tf.Variable 'conv2_weights:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv2_biases:0' shape=(32,) dtype=float32_ref>,
 # group2
 <tf.Variable 'conv1_weights_1:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv1_biases_1:0' shape=(32,) dtype=float32_ref>,
 <tf.Variable 'conv2_weights_1:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv2_biases_1:0' shape=(32,) dtype=float32_ref>]
```
Here, we would like to know how tensorflow name tensors, the general rules is descriped as following:
```
1. the name of the operation that produce it
2. the colon(:), and
3. the index of that tensor in outputs of operation that produced it.
```
All tensors are derived from some operation, and operations are either given a name in the constructor, or given the default name for a particular kind of operation. If the name is not unique, TensorFlow automatically handles this by appending "_1", "_2" etc. An operation with n tensor outputs name these tensors "op_name:0", "op_name:1", ..., "op_name:n-1"

**Problem** is that you call my_image_filter twice,but this will create two sets of variables,4 variable in each one, for a total of 8 variables,there is a solution, tedious but efficent ,by using variable_dict.
```
import tensorflow as tf
variables_dict = {
    "conv1_weights": tf.Variable(tf.random_normal([5, 5, 32, 32]),name="conv1_weights"),
    "conv1_biases": tf.Variable(tf.zeros([32]), name="conv1_biases"),
    "conv2_weights": tf.Variable(tf.random_normal([5, 5, 32, 32]),name="conv2_weights"),
    "conv2_biases": tf.Variable(tf.zeros([32]), name="conv2_biases")}
    
def my_image_filter(input_images, variables_dict):
    conv1 = tf.nn.conv2d(input_images, variables_dict["conv1_weights"],
        strides=[1, 1, 1, 1], padding='SAME')
    relu1 = tf.nn.relu(conv1 + variables_dict["conv1_biases"])

    conv2 = tf.nn.conv2d(relu1, variables_dict["conv2_weights"],
        strides=[1, 1, 1, 1], padding='SAME')
    return tf.nn.relu(conv2 + variables_dict['conv2_biases'])
    
# Both calls to my_image_filter() now use the same variables
img1 = tf.placeholder(tf.float32,shape=(100,32,32,32))
result1 = my_image_filter(img1, variables_dict)
result2 = my_image_filter(img1, variables_dict)
```
```
tf.trainable_variables()
Out[3]: 
[<tf.Variable 'conv1_weights:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv1_biases:0' shape=(32,) dtype=float32_ref>,
 <tf.Variable 'conv2_weights:0' shape=(5, 5, 32, 32) dtype=float32_ref>,
 <tf.Variable 'conv2_biases:0' shape=(32,) dtype=float32_ref>]

```
With convenient ,creating varibles like above ,outside of the code,breaks encapsulation
* the code builds graph must document the names ,shapes of variable to create.
* when the code changes ,the callers may have to create more , less or different varibles

So More general solution have been proposed by using Variable Scope mechanism that allows to easily share named variables while constructing a graph.
```
    def conv_relu(input, kernel_shape, bias_shape):
        # Create variable named "weights".
        weights = tf.get_variable("weights", kernel_shape,
            initializer=tf.random_normal_initializer())
        # Create variable named "biases".
        biases = tf.get_variable("biases", bias_shape,
            initializer=tf.constant_initializer(0.0))
        conv = tf.nn.conv2d(input, weights,
            strides=[1, 1, 1, 1], padding='SAME')
        return tf.nn.relu(conv + biases)

    def my_image_filter(input_images):
        with tf.variable_scope("conv1"):
            # Variables created here will be named "conv1/weights", "conv1/biases".
            relu1 = conv_relu(input_images, [5, 5, 32, 32], [32])
        with tf.variable_scope("conv2"):
            # Variables created here will be named "conv2/weights", "conv2/biases".
            return conv_relu(relu1, [5, 5, 32, 32], [32])

    img1 = tf.placeholder(tf.float32,shape=(100,32,32,32))
    result1 = my_image_filter(img1)
    result2 = my_image_filter(img1) # it will raise a error that 
             (Variable conv1/weights already exists, disallowed...)
```
```
ValueError: Variable conv1/weights already exists, disallowed. Did you mean to set reuse=True in VarScope? 
```
```
# this will be ok
with tf.variable_scope("image_filters") as scope:
    result1 = my_image_filter(img1)
    scope.reuse_variables()
    result2 = my_image_filter(img1)
```
## Variable_scope
Variable_scope determinate the function of tf.get_variable
* case 1 : The scope is set for creating new variables ,as evidenced by tf.get_variable_scope.reuse() = false
```
with tf.variable_scope("foo"):
    v = tf.get_variable("v", [1])
assert v.name == "foo/v:0"
```
```
v
Out[10]: 
<tf.Variable 'foo/v:0' shape=(1,) dtype=float32_ref>

```
* case 2 : The scope is set for reusing variables as evidenced by tf.get_variable_scope.reuse()=True
```
with tf.variable_scope("foo", reuse=True):
    v1 = tf.get_variable("v", [1])
assert v1 is v
```
```
v1
Out[12]: 
<tf.Variable 'foo/v:0' shape=(1,) dtype=float32_ref>

```
The primary function of variable scope is to carry a name that will be used as **prefix** for variables names and a **reuse-flag** to distinguish the two cases discribed above. 
nesting variable scopes append their names in a way analogous to how directories work

the current scope can be **recieved** by using tf.get_variable_scope 
and a reuse flag can be set to **True** by tf.get_variable_scope.reuse_variables()
```
with tf.variable_scope("foo"):
    with tf.variable_scope("bar"):
        v = tf.get_variable("v", [1])
assert v.name == "foo/bar/v:0"
v
Out[14]: 
<tf.Variable 'foo/bar/v:0' shape=(1,) dtype=float32_ref>

with tf.variable_scope("foo"):
    v = tf.get_variable("v", [1])
    tf.get_variable_scope().reuse_variables()
    v1 = tf.get_variable("v", [1])
assert v1 is v
```
Obviously ,you cannot set the reuse flag to false. but you can **enter a reusing variable scope** and then exit it, then exist it ,going back to a non_reusing one 
* using a paramster reuse = True when opening a variable scope
* the reusing parameter is inherited
```
with tf.variable_scope("root"):
    # At start, the scope is not reusing.
    assert tf.get_variable_scope().reuse == False
    with tf.variable_scope("foo"):
        # Opened a sub-scope, still not reusing.
        assert tf.get_variable_scope().reuse == False
    with tf.variable_scope("foo", reuse=True):
        # Explicitly opened a reusing scope.
        assert tf.get_variable_scope().reuse == True
        with tf.variable_scope("bar"):
            # Now sub-scope inherits the reuse flag.
            assert tf.get_variable_scope().reuse == True
    # Exited the reusing scope, back to a non-reusing one.
    assert tf.get_variable_scope().reuse == False
```
**Error** arise when:
* case 1: when variable_scope is set reuse=True  ,cannot get exist variable.
* case 2: when variable_scope is set reuse=False ,the new variable have been exist.

## Name_scope vs Variable_scope
when we do with tf.variable_scope('name') ,this implictly opens a tf.name_scope
* variable_scope to govern names of variable.
* name_scope only affect the the names of ops, but not of variables. 
```
tf.reset_default_graph()
with tf.variable_scope("foo"):
    x = 1.0 + tf.get_variable("v", [1])
assert x.op.name == "foo/add"
```
```
x
Out[22]: 
<tf.Tensor 'foo/add:0' shape=(1,) dtype=float32>
x.op.name
Out[23]: 
'foo/add'
`
```
Variable_scope implicitly opening name_scope. Creating a variable in a name_scope automatically expands its name with the scope name as well.
Now you see that tf.variable_scope() adds a prefix to the names of all variables (no matter how you create them), ops, constants. On the other hand tf.name_scope() ignores variables created with tf.get_variable(). 
```
import tensorflow as tf
def scoping(fn, scope1, scope2, vals):
    with fn(scope1):
        a = tf.Variable(vals[0], name='a')
        b = tf.get_variable('b', initializer=vals[1])
        c = tf.constant(vals[2], name='c')
        with fn(scope2):
            d = tf.add(a * b, c, name='res')
        print('\n  '.join([scope1, a.name, b.name, c.name, d.name]), '\n')
    return d
d1 = scoping(tf.variable_scope, 'scope_vars', 'res', [1, 2, 3])
d2 = scoping(tf.name_scope,     'scope_name', 'res', [1, 2, 3])
with tf.Session() as sess:
    writer = tf.summary.FileWriter('logs', sess.graph)
    sess.run(tf.global_variables_initializer())
    print(sess.run([d1, d2]))
    writer.close()
```
```
scope_vars
  scope_vars/a:0
  scope_vars/b:0
  scope_vars/c:0
  scope_vars/res/res:0 
scope_name
  scope_name/a:0
  b:0                                 ---> get_variables is not in the name_scope.
  scope_name/c:0
  scope_name/res/res:0 
```

# How do we just build the graph and automatically retrieve the parameters?

Building graph is discussed above and the following is for optimizer
## Necessity
As talking above, it is good to use variable_dict but not the best.In more complex model,such as four convolutional layer and three dense layers, it has to build every variable for each layer, it is a hard work. So how do we just build the graph and automatically retrieve the parameters? here is a good example.
```
with tf.variable_scope('name') as scope:
  build your graph
 
prameters = tf.get_collection(tf.GraphKeys.TRAINABLE_VARIABLES,scope='name')
```

## Graph collection
Graph collection is used for collect parameters from specific graph
```
tf.add_to_collection(name, value)
tf.get_collection(key, scope=None)   
tf.get_collection_ref(key)
```
About Key is the GraphKeys class which contains many standard names for collections. You can use various preset names to collect and retrive values with a graph.Standard keys are defined as following:
```
key = tf.GraphKeys.TRAINABLE_VARIABLES
```
```
GLOBAL_VARIABLES                   --->  Shared across distributed environment
LOCAL_VARIABLES                    --->  Variable objects that are local to each machine 
MODEL_VARIABLES                    --->  Model for inference
----------------------------------------------------------------------------------------------
Above use tf.contrib.framework to add to this collection
----------------------------------------------------------------------------------------------
TRAINABLE_VARIABLES                --->  Variable that will be trained by an optimzer
SUMMARIES                          --->  Attach to Graph
QUEUE_RUNNERS: 
MOVING_AVERAGE_VARIABLES: 
REGULARIZATION_LOSSES: 
WEIGHTS:
BIASES:
ACTIVATIONS: 
```
There are two ways to debug your code:
```
tf.trainable_variables()                                                ---> List all variables
tf.get_collection(tf.GraphKeys.TRAINABLE_VARIABLES, scope='NAME')       ---> List the variables which belong to particular scope
```



