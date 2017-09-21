## Eclipse NDK 入门及常见问题处理
### 1.环境搭建
- 1.安装JDK，配置环境变量
- 2.安装Eclipse [下载地址](https://www.eclipse.org/downloads/eclipse-packages/)
- 3.安装ADT插件 [下载地址](https://pan.baidu.com/s/1hs4AsZi)
- 4.下载解压android sdk </br>
    打开Eclipse ``windows->preference->android`` 设置SDK目录
- 5.安装NDK </br>
  根据操作系统选择下载[地址](http://mirrors.zzu.edu.cn/android/repository/)</br>
    打开Eclipse ``windows->preference->android->NDK`` 设置NDK目录
    如果出现提示路径Invalid，可以将NDK->build 下的``ndk-build`` 文件复制到NDK根目录
### 2.Hello NDK
* 新建一个`android` 工程
* 在工程上点击鼠标右键 `Android Tools->Add Native Support...`</br>
  定义要生成的so库名称，然后`finish`</br>
  `example:HelloNDK`
* 点开自动生成的`jni`文件夹->打开生成的`HelloNDK.cpp`文件,会发现
  ```cpp
  #include <jni.h>
  ```
  <font color=red>标黄线，原因是:`Unresolved inclusion: <jni.h>`</font></br>
  执行以下操作</br>
  右键工程选`Properties->C/C++ General->Paths and Symbols`</br>
  `Add`三个`Directory`</br>
  1)`...\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\lib\gcc\arm-linux-androideabi\4.9.x\include`</br>
  2)`...\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\lib\gcc\arm-linux-androideabi\4.9.x\include-fixed`</br>
  3)`...\platforms\android-24\arch-arm\usr\include`</br>
  `注意:[...]表示你的NDK文件夹根目录`</br>
  重新`build`工程
#####
* 创建Native方法
#####
   1.在MainAcitivy创建Native方法</br>
    `public native String getStringForJni();`</br>
    2.cd到src目录下，执行`javah com.example.hellondk.MainActivity`</br>
    3.将生成的.h文件移动到`jni`目录</br>
    4.编辑`HelloNDK.cpp`
```cpp
#include "com_example_hellondk_MainActivity.h"
JNIEXPORT jstring JNICALL Java_com_example_hellondk_MainActivity_getStringForJni
  (JNIEnv *env, jobject obj){
	return env->NewStringUTF("Hello String From C++");
}
```
5.编辑`MainActivity`
```java
public class MainActivity extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		final TextView tv=(TextView) findViewById(R.id.tv);
		tv.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				tv.setText(getStringForJni());
			}
		});
	}
	
	public native String getStringForJni();
	static {
		System.loadLibrary("HelloNDK");
	}
}

```
**一定不要忘了`loadLibrary`**</br>
运行即可成功
    
