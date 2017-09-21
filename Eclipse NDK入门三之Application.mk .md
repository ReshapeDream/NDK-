## Application.mk
### 1.简介
> `Application.mk`是用来描述你的应用程序需要哪些模块，以及这些模块所要具有的一些特性。而相对的`Android.mk`是用来描述要编译某个具体的模块，所需要的一些资源，包括要编译的源码、要链接的库等等。
```
Application.mk所要描述的内容主要包括：
1）你程序正常运行，所需要到模块的具体列表；
2）程序要编译成什么机器指令集的；
3）所有模块要被编译成Release版本还是Debug版本的；
4）传递给C或者C++编译器的编译参数。
```

> `Application.mk`文件一般是放在`$PROJECT/jni/`目录下的（`$PROJECT`代表你所写程序的项目目录），这样`ndk-build`命令可以自动搜索到它。
当然，`Application.mk`文件其实是可选的。默认情况下，如果`ndk-build`命令找不到`Application.mk`文件的话，就会使用如下规则进行编译：</br>
1）会编译全部在`Android.mk`文件中列出的模块</br>
2）对于所有模块，NDK编译系统会根据`armeabi` ABI来生成机器代码，即指令集是ARMv5TE

### 2.Application.mk中变量的定义

* APP_PROJECT_PATH
> 这个变量会告诉编译系统，你应用程序项目的根目录的绝对路径。
一般编译出来的Native模块是要被打包成某个APK应用程序的。而APK打包工具都是在项目根目录下的某个特定子目录中搜索有没有要打包的`.so`文件的，所以指定了项目根目录的绝对路径后，`ndk-build`命令在完成编译后，会将编译好的.so模块文件拷贝到项目的特定子目录下，方便后面打包。
如果你的项目目录的组织形式是类似于`$PROJECT/jni/Application.mk`，则这个变量是可选的。
* APP_MODULES
> 这个变量是可选的。如果`APP_MODULES`变量没有被定义的话，NDK将编译在`Android.mk`文件中定义的所有模块，以及所有其包含的子MakeFile文件。
如果`APP_MODULES`变量被定义了的话，则NDK只会编译明确列出的这几个模块。
注意，所有模块的名字要以空格分割，并且NDK会自动计算每个模块之间的依赖关系。
另外，从NDK r4开始改变了这个变量的行为。再这个版本之前Application.mk中必须定义这个变量（即其是强制的），并且必须明确列出所有模块的名字列表。而在这个版本之后，取消了这个限制。
* APP_OPTIM
> 这个变量是可选的，可以被定义成`release`或`debug`，用来告诉NDK将所有的模块编译成`Release`版本还是`Debug`版本。
如果这个变量被定义成`release`的话，则NDK会将所有模块编译成`Release`版本的，也就是会对代码进行深度优化，并且去除所有调试信息。这样的结果就是，编译出的代码执行效率高且文件比较小，但不利于调试。
如果这个变量被定义成`debug`的话，则NDK会将所有模块编译成`Debug`版本的，和`Release`版本相反，不会对代码进行任何优化，并且会加上很多调试信息。这样的结果就是，编译出的代码执行效率比较低且文件相对较大，但调试起来很方便。
如果没有特别指定这个变量，则NDK会自动查看你的应用程序是否是可调试的（即查看你项目目录下的`AndroidManifest.xml`文件，看是否在`<application>`标签中是否将`android:debuggable`属性设置成了`rue`）来决定到底编译`Release`版本还是`Debug`版本的。
但是，如果你定义了这个变量的话，则以这个变量定义的情况为准。无论你的应用程序是否是可调试的，都不会产生影响。

* APP_CFLAGS
> 如果想在编译所有模块的C或者C\+\+源码时，都需要指定一些特殊的编译选项的话，可以通过定义这个变量实现。
在`Android.mk`文件中，也可以分别为每个模块指定特定于这个模块的编译选项。但是，如果在`Application.mk`文件中定义了这个变量的话，其作用域是所有模块，而不是某个特定的模块。
比如，你要编译的某几个模块，如果都需要在一个特定目录下搜索头文件定义的话，你可以有两种做法。一是修改各个模块的`Android.mk`文件，为每个模块都定义`LOCAL_CFLAGS`变量，包含那个目录；二是只修改应用程序的`Application.mk`文件，定义`APP_CFLAGS`变量，包含那个目录。
注意，在`Android NDK 1.5 r1`版中，这个变量只会对编译C源文件起作用，而对编译C++源文件没有任何影响。这个问题已经在后面的NDK版本中得到了纠正。
* APP_CPPFLAGS
> 和前面说明的`APP_CFLAGS`变量类似，它也是用来指定一些特殊的编译选项，但只对编译C\+\+源文件其作用。
注意，在`Android NDK 1.5 r1`版本中，这个变量不仅对C源文件的编译，而且对C\+\+源文件的编译都会起作用。这个问题已经在后面的NDK版本中得到了纠正。
现在，如果想对编译C和C++源文件都指定相同的编译选项，可以使用前面介绍的`APP_CFLAGS`变量。

* APP_CXXFLAGS
>它是`APP_CPPFLAGS`变量的一个别名，但它已经过时了，在未来的NDK版本中有可能不再使用。因此，建议尽量不要使用它，而是使用`APP_CPPFLAGS`变量。
* APP_BUILD_SCRIPT
> 默认情况下，NDK编译系统会自动在`$(APP_PROJECT_PATH)/jni`目录下寻找名为`Android.mk`文件，作为模块定义文件。
如果你的`Android.mk`文件被放到别的位置的话，或者甚至你的模块定义文件不叫`Android.mk`，则可以通过修改这个变量的值，来让NDK使用你指定的模块定义文件。
注意，一个非绝对路径，总是会被解释为相对于NDK顶层目录的路径。
* APP_ABI
>默认情况下，NDK编译系统会根据`armeabi`</br> ABI来生成机器代码，即一个使用ARMv5TE指令集并且支持软件浮点操作的CPU。</br>
你可以通过定义`APP_ABI`变量来选择一个不同的ABI。</br>
比如，如果你的程序想在使用ARMv7指令集，且支持硬件FPU的设备上运行，可以使用：</br>
```
[plain] view plain copy
APP_ABI := armeabi-v7a  
或者你的程序想支持IA-32指令集，可以使用：
[plain] view plain copy
APP_ABI := x86  
或者你的程序想支持MIPS指令集，可以使用：
[plain] view plain copy
APP_ABI := mips  
或者，想同时支持这四种平台，可以使用：
[plain] view plain copy
APP_ABI := armeabi armeabi-v7a x86 mips  
```
>当然，你也可以使用任意某几个ABI，只要每个ABI之间用空格隔开即可。
或者，如果你使用NDK r7以上版本的话，还可以使用`all`来表示支持所有ABI平台：
```
[plain] view plain copy
APP_ABI := all  
```
* APP_STL</br>
> 默认情况下，NDK编译系统只为最小的C\+\+运行时库（`/system/lib/libstdc++.so`）提供C\+\+头文件。
然而，NDK还带有其它一些可选的C++运行时库的实现，你可以选择在你自己的应用程序中，通过在`Application.mk`中定义`APP_STL`变量，来使用或链接其中某一个。
这个变量可以被设置成如下几个值：
```
1）system
2）gabi++_static
3）gabi++_shared
4）stlport_static
5）stlport_shared
6）gnustl_static
7）gnustl_shared
```    
> 如果不特别定义的话，`system`运行时库是默认的值。除此之外，凡是后面带`_static`的，表示其是一个静态链接的运行时库（运行时库的代码包含在编译后的程序中）；而凡是后面带`_shared`”的，表示其是一个动态链接的运行时库（运行时库在程序运行时被动态加载进来）。如果去除动态或静态链接的因素，则除了默认的`system`运行时库之外，还有所谓的`gabi++`运行时库、`stlport`运行时库和`gunstl`运行时库。这四种运行时库所支持的C\+\+特性各不相同，
总结如下表：

    |       |C++异常 |C++ RTTI|C++标准库|
    |-------|--------|--------|--------|
    |System |不支持  |不支持  |不支持|
    |gabi++ |不支持  |支持    |不支持|
    |stlport|不支持  |支持    |支持|
    |gnustl |支持    |支持    |支持|
    
>可以看出，如果想支持C++异常的话，必须要使用`gunstl`运行时库。


* APP_GNUSTL_FORCE_CPP_FEATURES</br>
> 在NDK r7b之前的版本中，如果指定使用了`gnustl`运行时库，则默认在编译之后的代码中，都会加上对C\+\+异常和C\+\+ RTTI的支持代码。
这样做有可能会造成一定的问题。并且即使你的程序没有使用C\+\+异常和C\+\+ RTTI这些特性，支持它们的代码还是会加到你的模块中去，这样会大大增加模块的大小。这个问题已经在NDK r7b及以后的版本中被修复了。不过，这样也就意味着，如果你真的需要使用C\+\+异常或C\+\+ RTTI特性的话，必须明确的告知NDK编译系统（可以通过在`Application.mk`中定义`APP_CPPFLAGS`变量，或在`Android.mk`中定义`LOCAL_CPPFLAGS`或`LOCAL_CPP_FEATURES`变量来告知）。
但是，这样带来的问题就是兼容性问题，同一个编译脚本要根据不同的NDK版本来做区分操作。
为了让这种过度更加的顺畅，可以在`Application.mk`文件中定义`APP_GNUSTL_CPP_FEATURES`变量来指定到底要支持哪些C\+\+的特性。
这个变量可以被设置成如下两个值：

<table>
<tr>
<td>exceptions</td>
<td>表示编译后的代码要支持C++异常特性</td>
</tr>
<tr>
<td>rtti</td>
<td>表示编译后的代码要支持C++ RTTI特性</td>
</tr>
<table>

> 例如，如果你想让你的程序既获得C\+\+异常的支持，也得到C\+\+ RTTI的支持，可以使用：
```
[plain] view plain copy
APP_GNUSTL_FORCE_CPP_FEATURES := exceptions rtti 
```
