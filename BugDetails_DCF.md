Inspired by the success stories of fuzz testing, we design a graph-based fuzz testing method to improve the  quality of DL inference engines. Our method has discovered more than 40 different exceptions in three types of undesired behaviors: model conversion failure, inferencefailure,  output  comparison  failure. We detail the DCF bugs as follows. 

The threshold for the comparison of MNN and TensorFlow is that numbers with a relative error greater than 0.1% and less than 0.1% of the total data.
Formally, let _RE(mnn)_ be the ratio of the numbers with the relative error between MNN and TensorFlow over the total output data of an operator. 
% If _RE_ of a result is greater than 99.9%, 
When _RE(mnn) >= 99.9%_, the result is considered as a success.


Output Comparison Failure (Inconsistencies)
==============


***DCF-1 Gather. core dumped and data comparison failure  of Sub on MNN X86CPU and TensorFlow .***
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

When converting some FP32 numbers(from 0 to 1) into INT8 numbers, the cast operator will yield a calculation error. 












