Inspired by the success stories of fuzz testing, we design a graph-based fuzz testing method to improve the  quality of DL inference engines. Our method has discovered more than 40 different exceptions in three types of undesired behaviors: model conversion failure, inferencefailure,  output  comparison  failure. We detail the DCF bugs as follows. 

The threshold for the comparison of MNN and TensorFlow is that numbers with a relative error greater than 0.1% and less than 0.1% of the total data.
Formally, let _RE(mnn)_ be the ratio of the numbers with the relative error between MNN and TensorFlow over the total output data of an operator. 
If _RE_ of a result is greater than 99.9%, when _RE(mnn) >= 99.9%_, the result is considered as a success.


Output Comparison Failure (Inconsistencies)
==============


***DCF-1 Gather. core dumped and data comparison failure on MNN X86CPU and TensorFlow .***
---------------
The model structure is shown in Fig. . _RE(mnn_X86CPU)_ of Sub1 is 52.08%, _RE(mnn_X86CPU)_ of Sub2 is 76.04\%.
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

When converting some FP32 numbers(from 0 to 1) into INT8 numbers, the cast operator will yield a calculation error. The model contains two operators connected in sequence. The first Cast operator is configured as fp32 to int8, and the second is configured as int8 to fp32.

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
pic
The results of dividing by zero (coming from Pad ) in Realdiv are inf in TensorFlow and nan in MNN respectively. 



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

The output tensor of  mnnX86 CPU: 0.301035 0.41749 0.562585 0.490874 0.432062 0.419105 0.491333 0.485923 0.322798 0.449112 0.498304 0.360348

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





 
