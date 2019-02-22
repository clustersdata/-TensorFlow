# -TensorFlow
从源码编译安装TensorFlow
普通安装方法是pip官方的包，但这种大街货往往没有针对本地环境做优化。比如调用时会警告说你的机器支持一些可加速运算的指令，但编译时没有启用，让你心痒难耐。

2017-06-26 10:34:11.820609: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
2017-06-26 10:34:11.820621: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
2017-06-26 10:34:11.820624: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
2017-06-26 10:34:11.820629: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
你可以选择把头埋进沙子里：

import os
os.environ['TF_CPP_MIN_LOG_LEVEL']='2'
import tensorflow as tf
也可以选择花20分钟编译一个健全的TensorFlow。本来新MBP没有N卡已经够慢了，再被阉割这些加速指令，那就更残废了。

编译环境
以OSX Python2.7 CPU为例。

安装bazel
brew install bazel
查看是否安装成功

bazel version 
Build label: 0.5.1-homebrew
Build target: bazel-out/darwin_x86_64-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Tue Jun 6 12:34:07 2017 (1496752447)
Build timestamp: 1496752447
Build timestamp as int: 1496752447
如果出现

command not found: bazel
则

brew link bazel
如果出现

Error: Could not symlink bin/bazel
/usr/local/bin is not writable.
则

sudo chown -R hankcs:admin /usr/local/bin
configure
然后正式configuration：

cd tensorflow
./configure
一路回车，放心吧macOS都不支持。

No XLA support will be enabled for TensorFlow
Do you wish to build TensorFlow with VERBS support? [y/N] 
No VERBS support will be enabled for TensorFlow
Do you wish to build TensorFlow with OpenCL support? [y/N] 
No OpenCL support will be enabled for TensorFlow
Do you wish to build TensorFlow with CUDA support? [y/N] 
No CUDA support will be enabled for TensorFlow
.........
INFO: Starting clean (this may take a while). Consider using --async if the clean takes more than several minutes.
Configuration finished
编译
然后正式开始编译

bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
然后烧烧开水，大概20分钟

Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package
INFO: Elapsed time: 1411.419s, Critical Path: 81.12s
编译完成后在临时文件夹中生成了一堆binary，当前目录下也有软连接，可以直接调脚本生成wheel文件：

bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
成功得到

Mon Jun 26 11:42:01 CST 2017 : === Output wheel file is in: /tmp/tensorflow_pkg
安装
接下来安装这个wheel。如果之前pip装过tf的话，最好先卸载掉：

pip uninstall tensorflow
Proceed (y/n)? y
  Successfully uninstalled tensorflow-1.2.0
再安装

pip install /tmp/tensorflow_pkg/tensorflow-1.2.0-cp27-cp27m-macosx_10_12_x86_64.whl
安装成功

Installing collected packages: tensorflow
Successfully installed tensorflow-1.2.0
验证安装
跑个demo试试

cd ~
python
Python 2.7.13 (default, Dec 18 2016, 07:03:39) 
[GCC 4.2.1 Compatible Apple LLVM 8.0.0 (clang-800.0.42.1)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow as tf
>>> hello = tf.constant('Hello, TensorFlow built from source on hankcs\' macOS!')
>>> sess = tf.Session()
>>> print(sess.run(hello))
Hello, TensorFlow built from source on hankcs' macOS!
这说明新编译的TF工作正常。

如果遇到

Failed to load the native TensorFlow runtime
莫慌，不要在编译TensorFlow的目录中运行Python，cd到其他路径即可。

这时写代码调用TensorFlow再也不会出现警告了。

TensorFlow.png
