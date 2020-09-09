Inspired by the success stories of fuzz testing, we design a graph-based fuzz testing method to improve the  quality of DL inference engines. Our method has discovered more than 40 different exceptions in three types of undesired behaviors: model conversion failure, inferencefailure,  output  comparison  failure. We detail the DCF bugs as follows. 

The threshold for the comparison of MNN and TensorFlow is that numbers with a relative error greater than 0.1% and less than 0.1% of the total data.
Formally, let _RE(mnn)_ be the ratio of the numbers with the relative error between MNN and TensorFlow over the total output data of an operator. 
If _RE_ of a result is greater than 99.9%, when _RE(mnn) >= 99.9%_, the result is considered as a success.


Output Comparison Failure (Inconsistencies)
==============


***DCF-1 Gather. core dumped and data comparison failure on MNN X86CPU and TensorFlow .***
---------------
![dcfgather1fusion_fp16_randomnet_rn_5_540172_0001](https://user-images.githubusercontent.com/69624583/92605943-f24fc200-f2e4-11ea-86f1-10842e7e15b3.png)

 _RE(mnn_X86CPU)_ of Sub1 is 52.08%, _RE(mnn_X86CPU)_ of Sub2 is 76.04\%.
 MNN log: Error in `python': double free or corruption (!prev): 0x0000000002 4ae8b0. Backtrace: /dist-packages/\_mnncengine.so (\_ZN3MNN15BufferAllocator4NodeD1Ev+0x88) [0x7ff0e81 7a098]. Aborted (core dumped). 

#0  0x00007ff418033438 in __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:54

#1  0x00007ff41803503a in __GI_abort () at abort.c:89

#2  0x00007ff4180757fa in __libc_message (do_abort=do_abort@entry=2, fmt=fmt@entry=0x7ff41818ef98 "*** Error in `%s': %s: 0x%s ***\n") at ../sysdeps/posix/libc_fatal.c:175

#3  0x00007ff41807e38a in malloc_printerr (ar_ptr=<optimized out>, ptr=<optimized out>, str=0x7ff41818f0c8 "double free or corruption (!prev)", action=3) at malloc.c:5020

#4  _int_free (av=<optimized out>, p=<optimized out>, have_lock=0) at malloc.c:3874

#5  0x00007ff41808258c in __GI___libc_free (mem=<optimized out>) at malloc.c:2975

#6  0x00007ff417070098 in MNN::BufferAllocator::Node::~Node() () from /usr/local/lib/python2.7/dist-packages/_mnncengine.so

#7  0x00007ff4170700c2 in std::_Sp_counted_ptr<MNN::BufferAllocator::Node*, (__gnu_cxx::_Lock_policy)2>::_M_dispose() ()
   from /usr/local/lib/python2.7/dist-packages/_mnncengine.so

#8  0x00007ff417070b02 in std::_Rb_tree<unsigned long, std::pair<unsigned long const, std::shared_ptr<MNN::BufferAllocator::Node> >, std::_Select1st<std::pair<unsigned long const, std::shared_ptr<MNN::BufferAllocator::Node> > >, std::less<unsigned long>, std::allocator<std::pair<unsigned long const, std::shared_ptr<MNN::BufferAllocator::Node> > > >::_M_erase(std::_Rb_tree_node<std::pair<unsigned long const, std::shared_ptr<MNN::BufferAllocator::Node> > >*) () from /usr/local/lib/python2.7/dist-packages/_mnncengine.so



***DCF-2 Sigmoid. NaN: data comparison failure on MNN X86CPU and TensorFlow.***
---------------
When the input of a sigmoid operator is nan (coming from Rsqrt), the results are nan in TensorFlow and 1 in MNN X86 CPU respectively.



***DCF-3 Sigmoid. -inf: data comparison failure on MNN X86CPU and TensorFlow.***
---------------
When the input of a sigmoid operator is -inf (coming from Rsqrt), the results are 0 in TensorFlow and 1.64581e-38 in MNN X86 CPU respectively.





***DCF-4 Cast. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------

The model contains two operators connected in sequence. The first Cast operator is configured as fp32 to int8, and the second is configured as int8 to uint8. The cast operator will yield a calculation error.

The input tensor: 0.40432 0.117169 0.962151 0.183967 0.49692 0.0275081 0.909009 0.604157 0.493754 0.897677 0.535287 0.429582 0.266089 0.458383 0.997908 0.868963
0.232218 0.0356807 0.056058 0.476565 0.593033 0.700972 0.781927 0.485121

The output tensor of TF: 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 . The  output tensor of  mnn X86 CPU: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1.

 _RE(mnn_X86CPU)_ of Cast is 0%.




***DCF-5 LRN. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The configuration of the LRN operator is as follows.

alpha=1 beta=0.5 bias = 1  depth_raduis=5

x_data =(np.ones((1,1,1,1))*2).astype(np.float32)

x_data.tofile(name + "_inputdata_10000_0_fp32.bin")

out = tf.nn.local_response_normalization(x,name="lrn")

The output tensor of TF is 0.894427. The  output tensor of  mnn X86 CPU is 1.712698.





***DCF-6 Realdiv. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
![dcffusion_fp16_randomnet_rn_3_832028_0001](https://user-images.githubusercontent.com/69624583/92606037-13b0ae00-f2e5-11ea-9567-db98bcdba823.png)

The results of dividing by zero (coming from Pad ) in Realdiv are inf in TensorFlow and nan in MNN respectively. 


The x of Realdiv: array([ 1.2786534 ,  1.0426922 , -0.11910683,  1.5758498 ,  1.4206867 ,        1.6706531 ,  1.0834675 ,  0.25468162,  0.09891693,  1.4739399 ,        0.32884791,  0.11667726,  1.8873491 ,  0.5553351 ,  1.5132152 ,        1.5450279 ,  0.5409553 ,  1.2378874 ,  1.78609   ,  0.61828995,        1.5263603 ,  0.7822404 ,  0.9433871 ,  1.5502377 ,  0.0717475 ,        1.421492  ,  1.7305765 ,  1.0178283 ,  1.8812672 ,  1.0458821 ,        0.52104974,  1.1909834 ,  0.50230515,  1.6570765 ,  1.3553871 ,        0.9626574 ,  0.25978124,  1.4960558 ,  0.4294046 ,  1.5899268 ,       -0.08192477, -0.22359428,  0.73022735, -0.20688827,  1.5729395 ,        1.7708656 ,  0.5167163 ,  0.17279305, -0.18313001,  0.69207156,        1.8150818 ,  1.3760335 ,  0.63115156,  0.198356  ,  0.80806834,        0.8461137 ,  0.396619  ,  0.95837283,  1.126148  , -0.00151169,        1.55765   ,  0.43355164,  1.1452124 ,  1.7827791 ,  0.06954476,        0.6177971 ,  0.6698139 ,  0.5043217 ,  1.2868638 , -0.21163152,        1.8892969 ,  0.80474234], dtype=float32)

The y of Realdiv: array([1.5803587, 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       1.5677226, 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       1.574414 , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ,       0.       , 0.       , 0.       , 0.       , 0.       , 0.       ],  dtype=float32)

The output tensor of TF:
array([0.8090906 ,  inf, -inf,  inf, inf,  inf,  inf,inf,  inf,       inf, inf,  inf, 1.2038796 ,  inf,       inf, inf,  inf,  inf,  inf,inf,
              inf,  inf,        inf,  inf,0.04557092, inf,  inf,        inf,  inf, inf,   inf,  inf,   inf,  inf,       inf, inf,  inf, inf,  inf, inf,
             -inf, -inf,        inf,  inf,       inf,  inf,  inf,        inf,  inf, inf,   inf,  inf, inf,  inf,       inf,  inf,  inf,  inf,  inf,-inf,
              inf,  inf,        inf,  inf,       inf, inf,  inf,        inf,  inf, -inf,  inf,  inf], dtype=float32)

The output tensor of  mnn X86 CPU:
array([0.8347643 , nan,        nan,  nan,       nan,               nan, nan,        nan,  nan,       nan,              nan, nan, 1.201485  ,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,0.04603206,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan,        nan,  nan,       nan,              nan, nan], dtype=float32)


***DCF-7 Concat. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------

When the number of inputs is more than 2, the Concat operator  will yield a calculation error.

The values of the input and output tensors of the Concat operator are as follows. 

The input1: 3.60285 9.53354

The input2: 8.67775 2.24457

The input3: 1.38196 2.91937

The output tensor of TF: 3.602852 9.533535 8.677751 2.244566 1.381964 2.919371. The output tensor of  mnn X86 CPU: 1.381964 2.919371 8.677751 2.244566 1.381964 2.919371.

No errors are reported in the log of the Concat operator. The reference guide of MNN also does not indicate that the number of inputs of the Concat operator does not support more than 2.




***DCF-8 Deconv. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The shape of the output tensor of Tf and mnn is inconsistent.
The configuration of the Deconv operator is as follows.
 n = 10,  h = 1,  w =3, c = 3
 
 x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder")

 filter_data = np.random.uniform(-1, 1, size=[15,11,3,3]).astype(np.float32) * 0.3  # HWCK
 
 c = tf.nn.conv2d(x, filter_data,strides=[1,2,1,1],padding='SAME',dilations=[1,1,1,1],name="conv2d")
 
 filter_data = np.random.uniform(-1, 1, size=[1,3,3,3]).astype(np.float32) * 0.3  # HWCK
 
 out = tf.nn.conv2d_transpose(c,filter_data, strides=[1,2,2,1],output_shape=[10,2,7,3], padding="VALID",name="deconv")

The shape of the output tensor of Tf and mnn is inconsistent. The output tensor shape of TF: [10,2,7,3](NHWC).  The output tensor shape of MNN X86 CPU: [10,3,1,7].




***DCF-9 Argmax. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The configuration of the Argmax operator is as follows.

n = 3, h = 1, w = 1, c = 1, argmax/dimension = 0

The values of the input and output tensors of the Argmax operator are as follows. 
 
The input tensor: [[[[3.5494123]]] [[[1.2762802]]]  [[[5.062324 ]]]]

The output tensor of TF: 2. The output tensor of  mnn X86 CPU: 1.

 


***DCF-10 Depthwith. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
Data comparison failure on MNN X86CPU and TensorFlow. The configuration of the Depthwith operator is as follows.

n = 2, h = 2, w =2, *c =1* ,padding = 'VALID'

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder")

filter = np.random.uniform(-1, 1, size=[1,1,1,1]).astype(np.float32) #hwcn

out = tf.nn.depthwise_conv2d(x,filter,strides = [1,1,1,1],padding = *'VALID'*,rate = [2,2],name = "depthwise")

The values of the input and output tensors of the Depthwith operator are as follows. 
 
The input tensor: 1.89075 1.01571 1.24118 1.15506 1.58546 1.69911 1.12812 1.8123

The output tensor of TF:
-0.426098
-0.228900
-0.279712
-0.260302
-0.357297
-0.382909
-0.254232
-0.408417
. 

The output tensor of  mnn X86 CPU: 
-0.426098
-0.357297
-0.228900
-0.382909
-0.279712
-0.254232
-0.260302
-0.408417
.
 _RE(mnn_X86CPU)_ of Depthwith is 37.50%.
 
 
 
 
 ***DCF-11 Depthwith. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The shape of the output tensor of Tf and mnn is inconsistent. 
The configuration of the Depthwith operator is as follows.
 
n = 2，h = 2，w =2，c =1

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder")

filter = np.random.uniform(-1, 1, size=[1,1,1,2]).astype(np.float32)#hwcn

out = tf.nn.depthwise_conv2d(x,filter,strides = [1,1,1,1],padding = 'VALID',rate = [2,2],name = "depthwise")

The input tensor: 1.45974 1.36636 1.07671 1.97315 1.79221 1.52585 1.2357 1.98181.
The input tensor shape is [2,2,2,1].

The output tensor of TF: 0.0435238 0.915834 0.0407397 0.85725 0.0321035 0.675526 0.0588319 1.23795 0.0534368 1.12442 0.0454951 0.957314 0.0368439 0.775274 0.0590901 1.24338
The output tensor shape of TF is [2,2,2,2].

The output tensor of  mnn X86 CPU: 0.0435238 0.0534368 0.0407397 0.0454951 0.0321035 0.0368439 0.0588319 0.0590901
The output tensor shape of MNN X86 CPU is [8,1,1,1].

The shape of the output tensor of Tf and mnn is inconsistent. 



 ***DCF-12 Deconv. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The configuration of the Deconv operator is as follows.

n = 1，h = 1，w = 3，c = 3

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")
x_data =(np.ones((n,h,w,c))).astype(np.float32)

filter_data = (np.ones((1,3,3,3))).astype(np.float32) # HWCK
out_2 = tf.nn.conv2d_transpose(x,filter_data, strides=[1,1,1,1],output_shape=[1,1,3,3], padding="SAME",name="deconv")


The input tensor:1 1 1 1 1 1 1 1 1

The output tensor of TF: 6 6 6 9 9 9 6 6 6

The output tensor of  mnn X86 CPU: 3 3 3 6 6 6 9 9 9

The shape of the output tensor of Tf and MNN X86 CPU is inconsistent. The output tensor shape of TF: [1,1,3,3](NHWC).  The output tensor shape of MNN X86 CPU: [1, 3, 1, 3].

 _RE(mnn_X86CPU)_ of Deconv is 0%. 
 
 
 
  ***DCF-13 Reducemin. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
The cause of this inconsistency should be the calculation error of TensorFlow.

The configuration of the Reducemin operator is as follows.

n = 1，h = 2，w =3，c = 4

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="input_1")

x_data = (np.ones((n,h,w,c))*16).astype(np.float32)

a = tf.math.sqrt(x,name="sqrt_1")

res_reducemin_0= tf.reduce_min(a,axis = None,keep_dims = True,name = "redecemin_0")


The output tensor of TF: 16.0

The output tensor of  mnn X86 CPU: 4.0

If an Add operator is added after the Reducemin operator to add 0 to the result of the reducemin operator, then the inference result of tf is correct(4.0).

And thus, the cause of this inconsistency should be the calculation error of TensorFlow.


  ***DCF-14 Batchnorm. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
The configuration of the Batchnorm operator is as follows.

n = 1，h = 2，w = 3，c = 4

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

scale = [0,0.6856415271759033,0.6856415271759033,0.6856415271759033] 

offset = [0.6856415271759033,0.6856415271759033,0.6856415271759033,0.6856415271759033] 

out = tf.nn.fused_batch_norm(x,scale = scale,offset = offset,mean=scale,variance=scale,epsilon=0.001,data_format='NHWC',is_training=False,name="bn")[0]

  
The input tensor:1.80969 1.92234 1.10676 1.50458 1.37146 1.7904 1.26449 1.17676 1.98465 1.04982 1.87448 1.15665 1.45925 1.50845 1.80422 1.95404 1.46966 1.75505 1.13797 1.45105 1.68181 1.70959 1.38402 1.23861

The output tensor of TF: 0.685642 1.70892 1.03409 1.36326 0.685642 1.59976 1.1646 1.09201 0.685642 0.986972 1.66932 1.07537 0.685642 1.36646 1.61119 1.73515 0.685642 1.5705 1.05991 1.31896 0.685642 1.53289 1.2635 1.14318

The output tensor of  mnn ARM CPU: 0.685642 0.685642 0.685642 0.685642 0.685642 0.685642 1.1646 1.09201 1.76048 0.986972 1.66932 1.07537 1.32575 1.36646 1.61119 1.73515
1.33437 1.5705 1.05991 1.31896 1.5099 1.53289 1.2635 1.14318

The output tensor of  mnnX86 CPU: 0.685642 1.70892 1.03409 1.36326 0.685642 1.59976 1.1646 1.09201 0.685642 0.986972 1.66932 1.07537 0.685642 1.36646 1.61119 1.73515
0.685642 1.5705 1.05991 1.31896 0.685642 1.53289 1.2635 1.14318


 _RE(mnn_ARMCPU)_ of Batchnorm is 66.67%. 

 
 
   ***DCF-15 Maxpooling. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
  
  
The configuration of the Maxpooling operator is as follows.

n = 4，h = 2，w = 2，c = 2

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

out_2= tf.nn.max_pool(x, ksize =[1,1,1,1] ,strides=[1,2,1,1],padding="VALID",name="maxpooling")



The input tensor:1.85204 1.14694 1.97292 1.33441 1.53921 1.43205 1.9677 1.68036 1.01731 1.15119 1.28549 1.4035 1.2546 1.54129 1.75193 1.77484
1.58525 1.38116 1.10232 1.92552 1.87033 1.57231 1.842 1.51482 1.09474 1.88493 1.51141 1.08825 1.92578 1.16217 1.98962 1.87622

The output tensor of TF: 1.85204 1.14694 1.97292 1.33441 1.01731 1.15119 1.28549 1.4035 1.58525 1.38116 1.10232 1.92552 1.09474 1.88493 1.51141 1.08825

The output tensor of  mnn ARM CPU: 1.85204 1.14694 1.53921 1.43205 1.01731 1.15119 1.2546 1.54129 1.58525 1.38116 1.87033 1.57231 1.09474 1.88493 1.92578 1.16217

The output tensor of  mnnX86 CPU: 1.85204 1.14694 1.97292 1.33441 1.01731 1.15119 1.28549 1.4035 1.58525 1.38116 1.10232 1.92552 1.09474 1.88493 1.51141 1.08825

 _RE(mnn_ARMCPU)_ of Maxpooling is 50.00%. 



 
   ***DCF-16 Avgpooling. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
The configuration of the Avgpooling operator is as follows.

n = 1，h =3，w =3，c =2

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

out= tf.nn.avg_pool(x, [1,1,1,1],strides=[1, 1, 2, 1],padding='VALID',name="avgpooling")

The input tensor:1.19919 1.64421 1.52825 1.80458 1.21612 1.13802 1.33801 1.79613 1.22788 1.59343 1.22308 1.82913 1.18881 1.68926 1.76186 1.06241
1.3741 1.17902

The output tensor of TF: 1.19919 1.64421 1.21612 1.13802 1.33801 1.79613 1.22308 1.82913 1.18881 1.68926 1.3741 1.17902

The output tensor of  mnn ARM CPU: 1.19919 1.52825 1.80458 1.13802 1.33801 1.22788 1.59343 1.82913 1.18881 1.76186 1.06241 1.17902

The output tensor of  mnnX86 CPU: 1.19919 1.64421 1.21612 1.13802 1.33801 1.79613 1.22308 1.82913 1.18881 1.68926 1.3741 1.17902

 _RE(mnn_ARMCPU)_ of Avgpooling is 50.00%. 


 
   ***DCF-17 Deconv. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
The configuration of the Deconv operator is as follows.

n = 3，h = 2，w = 2，c = 3

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

filter_data = np.random.uniform(-1, 1, size=[1,1,1,3]).astype(np.float32) * 0.3  # HWNC

out_2 = tf.nn.conv2d_transpose(x,filter_data, strides=[1,1,1,1],output_shape=[3,2,2,1], padding="SAME",name="deconv")

The input tensor:1.1954 1.46606 1.21522 1.40054 1.24535 1.60786 1.99609 1.4368 1.92193 1.71225 1.15864 1.6657 1.46595 1.74937 1.86695 1.5089
1.37309 1.5522 1.91413 1.46281 1.57776 1.94598 1.99442 1.76259 1.20255 1.82843 1.52027 1.57966 1.60674 1.7726 1.86414 1.26261
1.58428 1.17788 1.14936 1.46023

The output tensor of TF: 0.301035 0.41749 0.562585 0.490874 0.432062 0.419105 0.491333 0.485923 0.322798 0.449112 0.498304 0.360348

The output tensor of  mnn ARM CPU: 0.428883 0.418508 0.255569 0.412048 0.415169 0.511926 0.513831 0.445365 0.349692 0.389087 0.320603 0.426135

The output tensor of  mnn X86 CPU: 0.301035 0.41749 0.562585 0.490874 0.432062 0.419105 0.491333 0.485923 0.322798 0.449112 0.498304 0.360348

 _RE(mnn_ARMCPU)_ of Deconv is 8.33%.




   ***DCF-18 Resize_bilinear. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
The configuration of the Resize_bilinear operator is as follows.

n = 2，h = 2，w = 2，c = 2
x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")
re = tf.image.resize_bilinear(x,size = [7,2],align_corners=False,name="resize_bilinear")

The input tensor:2.52656 0.141995 5.55546 5.83816 6.57609 1.52823 4.67499 7.81567 6.39217 8.54632 7.56845 4.99505 9.92131 8.55646 8.7727 7.98398

The output tensor of TF: 2.52656 0.141995 5.55546 5.83816 3.68357 0.538063 5.30389 6.40316 4.84057 0.93413 5.05233 6.96816 5.99758 1.3302 4.80077 7.53317
6.57609 1.52823 4.67499 7.81567 6.57609 1.52823 4.67499 7.81567 6.57609 1.52823 4.67499 7.81567 6.39217 8.54632 7.56845 4.99505
7.4005 8.54922 7.91252 5.84903 8.40882 8.55212 8.25659 6.70301 9.41715 8.55501 8.60067 7.55699 9.92131 8.55646 8.7727 7.98398
9.92131 8.55646 8.7727 7.98398 9.92131 8.55646 8.7727 7.98398

The output tensor of  mnn ARM CPU: 2.52656 0.141995 3.39196 1.76947 4.25736 3.39694 5.12276 5.02442 5.55546 5.83816 5.55546 5.83816 5.55546 5.83816 6.57609 1.52823
6.03292 3.32464 5.48975 5.12105 4.94658 6.91746 4.67499 7.81567 4.67499 7.81567 4.67499 7.81567 6.39217 8.54632 6.72825 7.53167
7.06433 6.51703 7.40041 5.50238 7.56845 4.99505 7.56845 4.99505 7.56845 4.99505 9.92131 8.55646 9.59314 8.39289 9.26496 8.22933
8.93679 8.06576 8.7727 7.98398 8.7727 7.98398 8.7727 7.98398

The output tensor of  mnnX86 CPU: 2.52656 0.141995 5.55546 5.83816 3.68357 0.538063 5.30389 6.40316 4.84057 0.93413 5.05233 6.96816 5.99758 1.3302 4.80077 7.53317
6.57609 1.52823 4.67499 7.81567 6.57609 1.52823 4.67499 7.81567 6.57609 1.52823 4.67499 7.81567 6.39217 8.54632 7.56845 4.99505
7.4005 8.54922 7.91252 5.84903 8.40882 8.55212 8.25659 6.70301 9.41715 8.55501 8.60067 7.55699 9.92131 8.55646 8.7727 7.98398
9.92131 8.55646 8.7727 7.98398 9.92131 8.55646 8.7727 7.98398

_RE(mnn_ARMCPU)_ of Resize_bilinear is 21.428571%.



   ***DCF-19 Conv2d. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
The configuration of the Conv2d operator is as follows.

n = 1，h = 2，w = 1，c = 2

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

filter_data = (np.random.rand(1,1,2,1)*10).astype(np.float32)#HWCN

out = tf.nn.conv2d(x, filter_data, strides=[1,1,1,1],padding="SAME",dilations=[1,1,1,1],name="conv")


The input tensor: 3.32932 7.1221 8.35369 3.85031

The output tensor of TF: 76.9141 73.1483

The output tensor of  mnn ARM CPU: 87.4414 67.2161

The output tensor of  mnnX86 CPU: 76.9141 73.1483

_RE(mnn_ARMCPU)_ of Conv2d is 0%.



   ***DCF-20 Deconv2d. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------


The configuration of the Deconv2d operator is as follows.

n = 3，h = 2，w = 2，c = 2
x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")
filter_data = np.random.uniform(-1, 1, size=[1,1,1,2]).astype(np.float32) * 0.3  # HWNC

o_h = int((h + (h - 1) * (strides[1]  - 1) + (kernel[0] - 1) * 2 - kernel[0]) / 1 + 1)

o_w = int((w + (w - 1) * (strides[2]  - 1) + (kernel[2] - 1) * 2 - kernel[2]) / 1 + 1)

out_2 = tf.nn.conv2d_transpose(x,filter_data, strides=[1,1,1,1],output_shape=[3,2,2,1], padding="VALID",name="deconv")


The input tensor: 1.02216 1.56782 1.13286 1.1632 1.2089 1.97479 1.98287 1.29203 1.82363 1.04148 1.4673 1.13671 1.88557 1.50993 1.14716 1.86642
1.86152 1.25844 1.81675 1.32558 1.19287 1.18547 1.63139 1.80021

The output tensor of TF: -0.409532 -0.376534 -0.500582 -0.558878 -0.49423 -0.437887 -0.569335 -0.474004 -0.5308 -0.53115 -0.391181 -0.559082

The output tensor of  mnn ARM CPU: -0.3612 -0.570252 -0.486908 -0.399772 -0.607893 -0.405487 -0.439293 -0.471977 -0.52197 -0.403909 -0.572329 -0.499721

The output tensor of  mnnX86 CPU: -0.409532 -0.376534 -0.500582 -0.558878 -0.49423 -0.437887 -0.569335 -0.474004 -0.5308 -0.53115 -0.391181 -0.559082

_RE(mnn_ARMCPU)_ of Deconv2d is 0%.




   ***DCF-21 SpaceToBatchND. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
  
After analyzing the data of each layer of the model, it is found that the calculation of the first SpaceToBatchND operator is incorrect. The configuration of the SpaceToBatchND operator is as follows.

n = 1，h = 2，w = 1，c = 2

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

filter_data = (np.random.rand(1,1,2,1)*10).astype(np.float32)#HWCN

out = tf.nn.conv2d(x, filter_data, strides=[1,1,1,1],padding="SAME",dilations=[1,1,1,1],name="conv")


The input tensor: 1.81285 1.80182 1.09175 1.34579 1.72089 1.51118 1.35476 1.83301 1.34466 1.81689 1.74305 1.17221 1.89568 1.43885 1.61227 1.67711 1.3753 1.71589 1.17715 1.28493 1.53955 1.91123 1.71185 1.50172 1.90004 1.10641 1.4819 1.40583 1.13994 1.18919 1.84038 1.36894 1.40226 1.32921 1.10263 1.00787 1.30912 1.74669 1.19233 1.14202 1.94152 1.43464 1.74921 1.40845 1.01566 1.70362 1.00647 1.0829 1.6929 1.10756 1.9646 1.42497 1.67611 1.22671 1.94018 1.8546 1.82265 1.48128 1.72428 1.46487 1.31575 1.15607 1.49664 1.84624 1.23121 1.16041 1.41739 1.44333 1.33098 1.24006 1.04439 1.71925 1.14984 1.81575 1.90262 1.10342 1.24691 1.10301 1.77748 1.67415 1.79952 1.42616 1.45274 1.04644 1.48503 1.07376 1.09438 1.4688 1.32712 1.69093 1.92985 1.95699 1.64589 1.42173 1.54272 1.14737 1.86412 1.31165 1.62311 1.93979 1.84567 1.88417 1.16312 1.34032 1.82499 1.23435 1.17927 1.06113 1.98209 1.59234 1.0961 1.74577 1.9278 1.91586 1.75939 1.35022 1.13791 1.80659 1.98983 1.80961 1.22276 1.63678 1.72411 1.87997 1.34435 1.84229 1.10311 1.39226 1.84723 1.82624 1.92291 1.80067 1.15709 1.43695 1.6523 1.35589 1.17243 1.6111 1.90181 1.26177 1.31176 1.64643 1.54058 1.74862 1.14093 1.55403 1.25646 1.2847 1.2333 1.50363 1.78761 1.51293 1.95894 1.50829 1.2121 1.01706 1.61223 1.78511 1.52368 1.57139 1.48224 1.92002 1.43546 1.99533 1.86006 1.40947 1.62721 1.14133

The output tensor of TF: 1.81285 1.80182 1.13994 1.18919 1.82265 1.48128 1.48503 1.07376 1.9278 1.91586 1.31176 1.64643 1.09175 1.34579 1.84038 1.36894 1.72428 1.46487 1.09438 1.4688 1.75939 1.35022 1.54058 1.74862 1.72089 1.51118 1.40226 1.32921 1.31575 1.15607 1.32712 1.69093 1.13791 1.80659 1.14093 1.55403 1.35476 1.83301 1.10263 1.00787 1.49664 1.84624 1.92985 1.95699 1.98983 1.80961 1.25646 1.2847 1.34466 1.81689 1.30912 1.74669 1.23121 1.16041 1.64589 1.42173 1.22276 1.63678 1.2333 1.50363 1.74305 1.17221 1.19233 1.14202 1.41739 1.44333 1.54272 1.14737 1.72411 1.87997 1.78761 1.51293 1.89568 1.43885 1.94152 1.43464 1.33098 1.24006 1.86412 1.31165 1.34435 1.84229 1.95894 1.50829 1.61227 1.67711 1.74921 1.40845 1.04439 1.71925 1.62311 1.93979 1.10311 1.39226 1.2121 1.01706 1.3753 1.71589 1.01566 1.70362 1.14984 1.81575 1.84567 1.88417 1.84723 1.82624 1.61223 1.78511 1.17715 1.28493 1.00647 1.0829 1.90262 1.10342 1.16312 1.34032 1.92291 1.80067 1.52368 1.57139 1.53955 1.91123 1.6929 1.10756 1.24691 1.10301 1.82499 1.23435 1.15709 1.43695 1.48224 1.92002 1.71185 1.50172 1.9646 1.42497 1.77748 1.67415 1.17927 1.06113 1.6523 1.35589 1.43546 1.99533 1.90004 1.10641 1.67611 1.22671 1.79952 1.42616 1.98209 1.59234 1.17243 1.6111 1.86006 1.40947 1.4819 1.40583 1.94018 1.8546 1.45274 1.04644 1.0961 1.74577 1.90181 1.26177 1.62721 1.14133

The output tensor of  mnn ARM CPU: 1.81285 1.61227 1.13994 1.74921 1.82265 1.04439 1.48503 1.62311 1.9278 1.10311 1.31176 1.2121 1.80182 1.67711 1.18919 1.40845 1.48128 1.71925 1.07376 1.93979 1.91586 1.39226 1.64643 1.01706 1.09175 1.3753 1.84038 1.01566 1.72428 1.14984 1.09438 1.84567 1.75939 1.84723 1.54058 1.61223 1.34579 1.71589 1.36894 1.70362 1.46487 1.81575 1.4688 1.88417 1.35022 1.82624 1.74862 1.78511 1.72089 1.17715 1.40226 1.00647 1.31575 1.90262 1.32712 1.16312 1.13791 1.92291 1.14093 1.52368 1.51118 1.28493 1.32921 1.0829 1.15607 1.10342 1.69093 1.34032 1.80659 1.80067 1.55403 1.57139 1.35476 1.53955 1.10263 1.6929 1.49664 1.24691 1.92985 1.82499 1.98983 1.15709 1.25646 1.48224 1.83301 1.91123 1.00787 1.10756 1.84624 1.10301 1.95699 1.23435 1.80961 1.43695 1.2847 1.92002 1.34466 1.71185 1.30912 1.9646 1.23121 1.77748 1.64589 1.17927 1.22276 1.6523 1.2333 1.43546 1.81689 1.50172 1.74669 1.42497 1.16041 1.67415 1.42173 1.06113 1.63678 1.35589 1.50363 1.99533 1.74305 1.90004 1.19233 1.67611 1.41739 1.79952 1.54272 1.98209
1.72411 1.17243 1.78761 1.86006 1.17221 1.10641 1.14202 1.22671 1.44333 1.42616 1.14737 1.59234 1.87997 1.6111 1.51293 1.40947 1.89568 1.4819 1.94152 1.94018 1.33098 1.45274 1.86412 1.0961 1.34435 1.90181 1.95894 1.62721 1.43885 1.40583 1.43464 1.8546 1.24006 1.04644 1.31165 1.74577 1.84229 1.26177 1.50829 1.14133

The output tensor of  mnnX86 CPU: 1.81285 1.80182 1.13994 1.18919 1.82265 1.48128 1.48503 1.07376 1.9278 1.91586 1.31176 1.64643 1.09175 1.34579 1.84038 1.36894 1.72428 1.46487 1.09438 1.4688 1.75939 1.35022 1.54058 1.74862 1.72089 1.51118 1.40226 1.32921 1.31575 1.15607 1.32712 1.69093 1.13791 1.80659 1.14093 1.55403 1.35476 1.83301 1.10263 1.00787 1.49664 1.84624 1.92985 1.95699 1.98983 1.80961 1.25646 1.2847 1.34466 1.81689 1.30912 1.74669 1.23121 1.16041 1.64589 1.42173 1.22276 1.63678 1.2333 1.50363 1.74305 1.17221 1.19233 1.14202 1.41739 1.44333 1.54272 1.14737 1.72411 1.87997 1.78761 1.51293 1.89568 1.43885 1.94152 1.43464 1.33098 1.24006 1.86412 1.31165 1.34435 1.84229 1.95894 1.50829 1.61227 1.67711 1.74921 1.40845 1.04439 1.71925 1.62311 1.93979 1.10311 1.39226 1.2121 1.01706 1.3753 1.71589 1.01566 1.70362 1.14984 1.81575 1.84567 1.88417 1.84723 1.82624 1.61223 1.78511 1.17715 1.28493 1.00647 1.0829 1.90262 1.10342 1.16312 1.34032 1.92291 1.80067 1.52368 1.57139 1.53955 1.91123 1.6929 1.10756 1.24691 1.10301 1.82499 1.23435
1.15709 1.43695 1.48224 1.92002 1.71185 1.50172 1.9646 1.42497 1.77748 1.67415 1.17927 1.06113 1.6523 1.35589 1.43546 1.99533 1.90004 1.10641 1.67611 1.22671 1.79952 1.42616 1.98209 1.59234 1.17243 1.6111 1.86006 1.40947 1.4819 1.40583 1.94018 1.8546 1.45274 1.04644 1.0961 1.74577 1.90181 1.26177 1.62721 1.14133

_RE(mnn_ARMCPU)_ of SpaceToBatchND is 7.14% .


 
 
   ***DCF-22 Reducemax. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
![dcf22_reducemax_rn_3_832029_0001](https://user-images.githubusercontent.com/69624583/92606825-f6301400-f2e5-11ea-9d74-6ac27b614389.png)

After analyzing the data of each layer of the model, it is found that the calculation of the Reducemax operator is incorrect. When comparing nan and -inf(output by sqrt), TensorFlow returns -inf, MNN returns nan. The result of calculation with nan should be nan. And thus, the cause of this inconsistency should be the calculation error of TensorFlow.

The configuration of the Reducemax operator is as follows.

n = 1，h = 2，w = 7，c = 1

nx =np.array(x,dtype=np.float32).reshape(n,h,w,c)

placeholder = tf.placeholder(shape=(n,h,w,c),dtype=tf.float32,name="placeholder_10000")

res_reducemax = tf.reduce_max(placeholder, axis=1, keep_dims=True,  name = "reducemax")


The input tensor： [[[[  1.] [ 2.] [ 3.] [  4.] [ nan] [  5.] [  6.]] [[  7.] [  8.] [  9.] [ inf]  [-inf] [ nan] [ 10.]]]]

The output tensor of TF:           [[[[7.]  [8.]    [9.]   [inf]    [-inf]    [5.]     [10.]]]]

The output tensor of  mnn X86 CPU: [[[[7.]  [8.]    [9.]   [inf]    [nan]      [5]     [10.]]]] 


 
   ***DCF-23 Reducemax. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------
![dcf23_reducemax_ws_2_832024_0001](https://user-images.githubusercontent.com/69624583/92606774-e7496180-f2e5-11ea-81f7-14b0591cff10.png)

After analyzing the data of each layer of the model, it is found that the calculation of the Reducemax operator is incorrect. When comparing nan and -inf(output by sqrt), TensorFlow returns -inf, MNN returns nan. The result of calculation with nan should be nan. And thus, the cause of this inconsistency should be the calculation error of TensorFlow.

The configuration of the Reducemax operator is as follows.

n = 1，h = 2，w = 2，c = 2

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")

x_data = (np.random.rand(n,h,w,c)*10).astype(np.float32)

ceil =tf.math.ceil(x,name="ceil")

out = tf.reduce_max(ceil,axis =[0,1,3], keep_dims=True,name="reducemax")


The input tensor： [[[[8.370346   7.894142  ]    [4.6929383  0.80955863]]   [[4.7769775  8.743156  ]    [2.3229198  9.342096  ]]]]

The output tensor of Ceil operator:[[[[ 9.  8.]    [ 5.  1.]]    [[ 5.  9.]    [ 3. 10.]]]]

The output tensor of TF:  8.74316 9.3421

The output tensor of  mnn X86 CPU: 9 10
 
 
 
   ***DCF-24 operators unsupported in MNN reference guide.***
---------------
  
 We also tried to infer some models that contain operators unsupported in MNN reference guide, such as Addn, Clip and Asin. And these models are successfully inferred by MNN on X86 CPU and ARM CPU.
 
 
   ***DCF-25 Data types unsupported in MNN reference guide.***
---------------
  
 We also tried to infer some models that Data types  unsupported in MNN reference guide, such as uint8 and bool of Cast. And these models are successfully inferred by MNN on X86 CPU and ARM CPU.
 
 
 
 
 
   ***DCF-26 BatchToSpaceND. Data comparison failure on MNN ARMCPU and TensorFlow.***
  ---------------
  ![dcf26_BatchToSpaceND_rn_2_268332_0001](https://user-images.githubusercontent.com/69624583/92606701-cf71dd80-f2e5-11ea-9cd0-e297966183f4.png)
  
After analyzing the data of each layer of the model, it is found that the calculation of the first SpaceToBatchND operator is incorrect. The configuration of the SpaceToBatchND operator is as follows.
n = 2,h = 2,w = 3,c = 4
x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="placeholder_10000")
out = tf.batch_to_space_nd(x,crops=[[0,0],[0,0]], block_shape=[1,2], name="batch")


The input tensor: 1.03884 1.16265 1.76784 1.78022 1.1338 1.21764 1.25461 1.51277 1.48314 1.70554 1.19038 1.47641 1.07206 1.07769 1.47213 1.23621 1.68363 1.49882 1.0184 1.8426 1.07029 1.28709 1.45593 1.01174 1.47935 1.06786 1.02764 1.64876 1.058 1.7861 1.34905 1.58041 1.00863 1.9733 1.76953 1.06869 1.36675 1.32908 1.38327 1.39296 1.1866 1.20684 1.53111 1.92347 1.58694 1.74022 1.87438 1.38115

The output tensor of TF: 1.03884 1.16265 1.76784 1.78022 1.47935 1.06786 1.02764 1.64876 1.1338 1.21764 1.25461 1.51277 1.058 1.7861 1.34905 1.58041
1.48314 1.70554 1.19038 1.47641 1.00863 1.9733 1.76953 1.06869 1.07206 1.07769 1.47213 1.23621 1.36675 1.32908 1.38327 1.39296
1.68363 1.49882 1.0184 1.8426 1.1866 1.20684 1.53111 1.92347 1.07029 1.28709 1.45593 1.01174 1.58694 1.74022 1.87438 1.38115

The output tensor of  mnn ARM CPU: 1.03884 1.47935 1.16265 1.06786 1.76784 1.02764 1.78022 1.64876 1.1338 1.058 1.21764 1.7861 1.25461 1.34905 1.51277 1.58041 1.48314 1.00863 1.70554 1.9733 1.19038 1.76953 1.47641 1.06869 1.07206 1.36675 1.07769 1.32908 1.47213 1.38327 1.23621 1.39296 1.68363 1.1866 1.49882 1.20684 1.0184 1.53111 1.8426 1.92347 1.07029 1.58694 1.28709 1.74022 1.45593 1.87438 1.01174 1.38115

The output tensor of  mnnX86 CPU: 1.03884 1.16265 1.76784 1.78022 1.47935 1.06786 1.02764 1.64876 1.1338 1.21764 1.25461 1.51277 1.058 1.7861 1.34905 1.58041 1.48314 1.70554 1.19038 1.47641 1.00863 1.9733 1.76953 1.06869 1.07206 1.07769 1.47213 1.23621 1.36675 1.32908 1.38327 1.39296 1.68363 1.49882 1.0184 1.8426 1.1866 1.20684 1.53111 1.92347 1.07029 1.28709 1.45593 1.01174 1.58694 1.74022 1.87438 1.38115

_RE(mnn_ARMCPU)_ of SpaceToBatchND is 25.00% .

 
 
 
 
 
   ***DCF-27 Maximum. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------

After analyzing the data of each layer of the model, it is found that the calculation of the Maximum operator is incorrect. When comparing nan and -inf(output by sqrt), TensorFlow returns nan , MNN returns -inf. The result of calculation with nan should be nan. And thus, the cause of this inconsistency should be the calculation error of MNN.

The configuration of the Maximum operator is as follows.

 placeholder1 = tf.placeholder(shape=shape1,dtype=tf.float32,name="placeholder_10000")
 
 placeholder2 = tf.placeholder(shape=shape2,dtype=tf.float32,name="placeholder_10200")
  
 res_maximum = tf.math.minimum(placeholder1, placeholder2,name = "minimum_outputdata_10100")


The input tensor1： nan nan inf inf nan nan inf nan inf inf nan inf inf nan nan nan inf inf inf inf nan inf nan inf

The input tensor2：nan nan -inf nan -inf -inf -inf nan -inf -inf nan nan nan nan nan nan -inf -inf -inf nan -inf -inf nan -inf

The output tensor of TF: nan, nan, inf, inf, nan, nan, inf, nan, inf, inf, nan, inf, inf, nan, nan, nan, inf, inf, inf, inf, nan, inf, nan, inf

The output tensor of  mnn X86 CPU: nan,  nan,  inf,  inf, -inf, -inf,  inf,  nan,  inf,  inf,  nan, inf,  nan,  nan,  nan,  nan,  inf,  inf,  inf,  inf, -inf,  inf,  nan,  inf
 
 
 
  ***DCF-28 Minimum. Data comparison failure on MNN X86CPU and TensorFlow.***
---------------

After analyzing the data of each layer of the model, it is found that the calculation of the Minimum operator is incorrect. When comparing nan and -inf(output by sqrt), TensorFlow returns nan , MNN returns -inf. The result of calculation with nan should be nan. And thus, the cause of this inconsistency should be the calculation error of MNN.

The configuration of the Minimum operator is as follows.

placeholder1 = tf.placeholder(shape=shape1,dtype=tf.float32,name="placeholder_10000")

placeholder2 = tf.placeholder(shape=shape2,dtype=tf.float32,name="placeholder_10200")

res_maximum = tf.math.minimum(placeholder1, placeholder2,name = "minimum_outputdata_10100")


The input tensor1： nan inf nan nan nan nan nan inf inf inf nan nan nan nan inf inf inf inf inf nan nan nan nan nan

The input tensor2：-inf -inf nan -inf -inf nan -inf nan nan -inf nan nan -inf -inf nan nan nan nan -inf -inf nan -inf nan -inf

The output tensor of TF: nan, -inf,  nan,  nan,  nan,  nan,  nan,  inf,  inf, -inf,  nan, nan,  nan,  nan,  inf,  inf,  inf,  inf, -inf,  nan,  nan,  nan,  nan,  nan

The output tensor of  mnn X86 CPU: -inf, -inf,  nan, -inf, -inf,  nan, -inf,  nan,  nan, -inf,  nan, nan, -inf, -inf,  nan,  nan,  nan,  nan, -inf, -inf,  nan, -inf,  nan, -inf
 
 
 
 
 
 
 
 
 
 
 
 
 
 
