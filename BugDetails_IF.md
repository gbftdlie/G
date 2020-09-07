Inspired by the success stories of fuzz testing, we design a graph-based fuzz testing method to improve the  quality of DL inference engines. Our method has discovered more than 40 different exceptions in three types of undesired behaviors: model conversion failure, inference failure,  output  comparison  failure. We detail the IF bugs as follows.


Inference Failure
================

 ***IF-1 Reduceprod: The Reduceprod cannot output results. Inference aborted.***
----------------

The Reduceprod cannot output results (shown in Fig.  the model with 26 operators). 

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
