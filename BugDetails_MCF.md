We detail these bugs as follows. 



**Model Conversion Failure**
================

 
***MCF-1 TopKV2:  Segmentation fault in TopKV2 conversion.***
----------------

MNN model cannot be generated. Almost all models that include TopKV2 operators cannot be converted and inferred successfully. 

MNN Log: Error for 18. Segmentation fault (core dumped). 

(gdb) bt

#0  0x00007f2cc088c6e5 in MNN::TopKV2SizeComputer::onComputeSize(MNN::Op const*, std::vector<MNN::Tensor*, std::allocator<MNN::Tensor*> > const&, std::vector<MNN::Tensor*,std::allocator<MNN::Tensor*> > const&) const () from /usr/local/lib/python2.7/dist-packages/_tools.so

#1  0x00007f2cc093e84b in MNN::Express::Executor::computeInfo(MNN::Express::Expr*) () from /usr/local/lib/python2.7/dist-packages/_tools.so
 
 
***MCF-2 Deconv： Conversion aborted.***
----------------
![mcf3fusion_fp16_randomnet_ws_2_832384_0001](https://user-images.githubusercontent.com/69624583/92605031-ce3fb100-f2e3-11ea-857e-44e21507ac68.png)

The Deconv API of the pb model is tf.nn.conv2dtranspose. 

MNN log: /converter/source/common/writeFb.cpp:108  Check failed: (notSupportOps.size()) == (0). Not Support: Tensorflow::Conv2DBackpropInput, output\_shape is not consistent with inferred output shape in MNN. (height, width): (38,102) vs (4,21). Convert Tensorflow's Op deconv2d\_outputdata\_10100, type = Conv2DBackpropInput, failed. Aborted (core dumped).

#0  0x00007f9ed925e438 in __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:54

#1  0x00007f9ed926003a in __GI_abort () at abort.c:89

#2  0x00007f9ed7d2884d in __gnu_cxx::__verbose_terminate_handler() () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6

#3  0x00007f9ed7d266b6 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6

#4  0x00007f9ed7d26701 in std::terminate() () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6

#5  0x00007f9ed7d26919 in __cxa_throw () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6

#6  0x00007f9e34fdcf9c in writeFb(std::unique_ptr<MNN::NetT, std::default_delete<MNN::NetT> >&, std::string const&, bool, bool) [clone .cold.981] ()
   from /usr/local/lib/python2.7/dist-packages/_tools.so
 #7  0x00007f9e3500ff59 in PyTool_Converter(_object*, _object*) () at /ruhuan/Projects/AliNNPrivate/pymnn/src/MNNTools.cc:62
 
 

  
 ***MCF-3 A model with 10+ operators： Conversion aborted.***
----------------
![8](https://user-images.githubusercontent.com/69624583/92605134-ef080680-f2e3-11ea-8987-a2da9e408f3a.jpg)

MNN log:

 *** Error in `/usr/bin/python': free(): invalid pointer: 0x0000000001db1940 ***
 ======= Backtrace: =========
 /lib/x86_64-linux-gnu/libc.so.6(+0x777f5)[0x7f0ed0ec27f5]
 /lib/x86_64-linux-gnu/libc.so.6(+0x8038a)[0x7f0ed0ecb38a]
 /lib/x86_64-linux-gnu/libc.so.6(cfree+0x4c)[0x7f0ed0ecf58c]
 /usr/local/lib/python2.7/dist-packages/_tools.so(_ZNSt15_Sp_counted_ptrIPN3MNN7Express4Expr6InsideELN9__gnu_cxx12_Lock_policyE2EE10_M_disposeEv+0x7e)[0x7                               f0e2cd7cdce]
 /usr/local/lib/python2.7/dist-packages/_tools.so(_ZNSt16_Sp_counted_baseILN9__gnu_cxx12_Lock_policyE2EE10_M_releaseEv+0x47)[0x7f0e2cc46347]
 /usr/local/lib/python2.7/dist-packages/_tools.so(_ZN3MNN7Express4ExprD1Ev+0x25)[0x7f0e2cd7ca95]
 /usr/local/lib/python2.7/dist-packages/_tools.so(_ZNSt15_Sp_counted_ptrIPN3MNN7Express4ExprELN9__gnu_cxx12_Lock_policyE2EE10_M_disposeEv+0x12)[0x7f0e2cd7                               cd32]

#0  0x00007fd945390438 in __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:54

#1  0x00007fd94539203a in __GI_abort () at abort.c:89

#2  0x00007fd9453d27fa in __libc_message (do_abort=do_abort@entry=2, fmt=fmt@entry=0x7fd9454ebf98 "*** Error in `%s': %s: 0x%s ***\n") at ../sysdeps/posix/libc_fatal.c:175

#3  0x00007fd9453db38a in malloc_printerr (ar_ptr=<optimized out>, ptr=<optimized out>, str=0x7fd9454e8d6f "free(): invalid pointer", action=3) at malloc.c:5020

#4  _int_free (av=<optimized out>, p=<optimized out>, have_lock=0) at malloc.c:3874

#5  0x00007fd9453df58c in __GI___libc_free (mem=<optimized out>) at malloc.c:2975

#6  0x00007fd8a11454a6 in std::string::_Rep::_M_dispose (__a=..., this=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/ext/atomicity.h:82

#7  std::string::_Rep::_M_dispose (__a=..., this=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/basic_string.h:3250

#8  std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string (this=0x12e9320, __in_chrg=<optimized out>)
    at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/basic_string.h:3640

#9  MNN::AttributeT::~AttributeT (this=0x12e9310, __in_chrg=<optimized out>) at /ruhuan/Projects/AliNNPrivate/schema/current/Tensor_generated.h:378

#10 std::default_delete<MNN::AttributeT>::operator() (this=<optimized out>, __ptr=0x12e9310) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/unique_ptr.h:81

#11 std::default_delete<MNN::AttributeT>::operator() (this=0x12e9648, __ptr=0x12e9310) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/unique_ptr.h:75

#12 std::unique_ptr<MNN::AttributeT, std::default_delete<MNN::AttributeT> >::~unique_ptr (this=0x12e9648, __in_chrg=<optimized out>) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/unique_ptr.h:274

#13 std::_Destroy<std::unique_ptr<MNN::AttributeT, std::default_delete<MNN::AttributeT> > > (__pointer=0x12e9648) at /opt/rh/devtoolset-8/root/usr/include/c++/8/bits/stl_construct.h:98

#14 std::_Destroy_aux<false>::__destroy<std::unique_ptr<MNN::AttributeT, std::default_delete<MNN::AttributeT> >*> (__last=<optimized out>, __first=0x12e9648)


 ***MCF-4 A model with 5 operators： Conversion aborted.***
----------------

![10_mcf](https://user-images.githubusercontent.com/69624583/92605209-08a94e00-f2e4-11ea-9042-4e018fbef622.png)

MNN log:
Start to Optimize the MNN Net...
 [22:08:40] :20: Inputs: placeholder_10000
 [22:08:40] :37: Outputs: reducesum_outputdata_10400, Type = Reduction

#0  _int_malloc (av=av@entry=0x7f1376b8ab20 <main_arena>, bytes=bytes@entry=4097) at malloc.c:3619

#1  0x00007f137684a1d4 in __GI___libc_malloc (bytes=4097) at malloc.c:2920

#2  0x00000000004a41bf in PyImport_ImportModuleLevel ()

#3  0x00000000004a5ce4 in ?? ()

#4  0x00000000004a59e5 in PyObject_CallFunction ()

#5  0x00000000004a3d27 in PyImport_Import ()

#6  0x00007f13757dec8e in importName (symbol=0x7f1375a1777f "Tensor", name=0x7f1375a16a3b "MNN") at /ruhuan/Projects/AliNNPrivate/pymnn/src/MNN.cc:50

#7  0x00007f13757e2912 in PyMNNInterpreter_getSessionOutputAll(PyMNNInterpreter*, _object*) () at /ruhuan/Projects/AliNNPrivate/pymnn/src/MNN.cc:904

#8  0x00000000004bc9ba in PyEval_EvalFrameEx ()


 ***MCF-5： StopGradient： Model Generation Failure In MNN.***
----------------
No error was reported when converting models that include unsupported StopGradient operators. The conversion process did not intercept an error and was aborted, and an empty model with a size of 0 was generated.
MNN log: Model has no oplist.


 ***Concat： Model Generation Failure In TensorFlow.***
----------------
It is worthy of mentioning that some exceptions of TensorFlow are found in model generation. When generating a model containing two Concats whose two inputs come from the same two constants, TensorFlow will get stuck.

The configuration of the Concats is as follows.

x  = tf.placeholder(shape=(n,h,w,c), dtype=tf.float32,name="input_1")

y=tf.constant(y_data,dtype=tf.float32, shape=(n,h,w,c),name="constant_1")

m=tf.constant(m_data,dtype=tf.float32, shape=(n,h,w,c),name="constant_3")

c_1= tf.concat([x,y,m], axis=2, name="concat")

out = tf.concat([c_1,y,m], axis=2, name="concat_1")

