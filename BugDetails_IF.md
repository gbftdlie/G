Inspired by the success stories of fuzz testing, we design a graph-based fuzz testing method to improve the  quality of DL inference engines. Our method has discovered more than 40 different exceptions in three types of undesired behaviors: model conversion failure, inference failure,  output  comparison  failure. We detail the IF bugs as follows.


Inference Failure
================

 ***IF-1 Reduceprod: The Reduceprod cannot output results. Inference aborted.***
----------------

The Reduceprod cannot output results (on the left side of the structure ). This model contains 26 operators.

![image](https://user-images.githubusercontent.com/69624583/92420420-c1b53e80-f1a5-11ea-8c6c-37d64034fa32.png)
![image](https://user-images.githubusercontent.com/69624583/92420438-d98cc280-f1a5-11ea-8513-256922c35553.png)

MNN Log: 
*** Error in `python': free(): invalid next size (fast): 0x0000000001e2cb90 ***
======= Backtrace: =========
/lib/x86_64-linux-gnu/libc.so.6(+0x777f5)[0x7f767300b7f5]
/lib/x86_64-linux-gnu/libc.so.6(+0x8038a)[0x7f767301438a]
/lib/x86_64-linux-gnu/libc.so.6(cfree+0x4c)[0x7f767301858c]
/usr/local/lib/python2.7/dist-packages/_mnncengine.so(_ZN3MNN6TensorD1Ev+0x74)[0x7f7672028654]
/usr/local/lib/python2.7/dist-packages/_mnncengine.so(+0x4f2a0)[0x7f7671fac2a0]

#0  0x00007f0c6e11b438 in __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:54

#1  0x00007f0c6e11d03a in __GI_abort () at abort.c:89

#2  0x00007f0c6e15d7fa in __libc_message (do_abort=do_abort@entry=2, fmt=fmt@entry=0x7f0c6e276f98 "*** Error in `%s': %s: 0x%s ***\n") at ../sysdeps/posix/libc_fatal.c:175

#3  0x00007f0c6e16638a in malloc_printerr (ar_ptr=<optimized out>, ptr=<optimized out>, str=0x7f0c6e277010 "free(): invalid next size (fast)", action=3) at malloc.c:5020

#4  _int_free (av=<optimized out>, p=<optimized out>, have_lock=0) at malloc.c:3874

#5  0x00007f0c6e16a58c in __GI___libc_free (mem=<optimized out>) at malloc.c:2975

#6  0x00007f0c6d17a654 in MNN::Tensor::~Tensor() () from /usr/local/lib/python2.7/dist-packages/_mnncengine.so

#7  0x00007f0c6d0fe2a0 in PyMNNTensor_dealloc (self=0x7f0c6e86ab50) at /ruhuan/Projects/AliNNPrivate/pymnn/src/MNN.cc:1086




 ***IF-2 The shapes of the 4 output tensors of the model are 0 with a certain probability.***
----------------
![if_2_9_443247_0001](https://user-images.githubusercontent.com/69624583/92539865-63f02780-f275-11ea-8f92-6c44e2ba87dc.png)

The shapes of the 4 output tensors(mul_10100(left), mul_10800, reduceprod_10500,squeeze_10600) of the model are 0 with a certain probability.

The shapes of these 4 output tensors in  three inferences are listed as follows.  And the  results of three inferences are not correct.


| Inference  | mul_10100   | mul_10800  | reduceprod_10500  | squeeze_10600 |
| ------------- |:-------------:|:----------:|:----------:| :----------: |
| 1        | (0,0,0,0) | (0,0,0,0)     |(0,0,0,0)       | (0,0,0,0)  |
| 2       | (8,2,11,11)    |  (0,0,0,0)     |(0,0,0,0)       | (0,0,0,0)  |
| 3      | (8,2,11,11)    |   (0,0,0,0)  |   (11,)   | (176, 11) |


MNN log: 
Reshape error: 22 -> 2.
due to the internal logic of MNN, if your MNN model doesn't have input shape, you may ignore this 'Resize error' information:
** Resize error for [Reshape], reshape_10800, code=3 **
it will work after you set the input tensor shape in MNN, and then resize the Session

However, Slice's 'size' is (2,1,1,1). And Reshape's 'shape' is (2,1). That is to say, the number of input and output data of the reshape operator is 2.









