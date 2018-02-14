---
category: Android
date: 2013-08-01
layout: post
title:  Custom Class Loading in Dalvik (译)
---

[原文链接](http://android-developers.blogspot.com/2011/07/custom-class-loading-in-dalvik.html)

Dalvik虚拟机为开发者执行自定类加载提供了便利。除了从本地加载Dalvik可执行文件("dex"),应用程序可以从备选地点如内部存储或者网络上加载它们。

这种技术不是适用每个应用程序的，事实上，大部分没有使用这种技术的也工作的很好。然而，有些情况下自定义类加载会得心应手。这里有一对情景：

巨大的应用程序包含超过64k个方法引用，超出了在一个dex文件里支持的最大值。为了绕过这个限制，开发者可以将这个程序分成多个二级dex文件，然后在运行时加载它们。

开发者可以将它们的框架设计成执行逻辑具有很好的可扩展性，然后通过在运行时动态加载代码。

我们已经创建了一个[应用](https://code.google.com/p/android-custom-class-loading-sample/)来演示分割的dex文件和运行时类加载。（请注意，在下面讨论的原因，应用不能用ADT Eclipse plug-in 构建，而使用包含其中的Ant构建脚本,详细查看Readme.txt）

App有一个简单的Activity来调用一个类库组件以显示一个Toast.Activity和它的资源包含在默认的dex中，而类库代码是被存储在二级dex中并绑定在APK。这需要一个修改的构建过程，也就是接下来将详细展示的。

在类方法可以被调用前，App首先必须明确地加载二级dex文件。让我们看看这些相关的moving parts

##代码组织
---

应用包含3个类

com.example.dex.MainActivity: UI component from which the library is invoked

com.example.dex.LibraryInterface: Interface definition for the library

com.example.dex.lib.LibraryProvider: Implementation of the library

类库被打包在一个二级dex中，而其余类包含在默认（主要的）dex文件中，“Build Process”章节说明了如何完成它。当然，打包策略依赖于特殊场景--开发者正要处理的问题。

##类加载和方法调用
---

二级dex文件--含有LibraryProvider,存储为App的asset。首先，它必须拷贝到一个路径能被类加载器访问到的存储位置。示例App使用了App的私有内部存储空间做到了这点。（从技术上来说，外部存储也可以，但要考虑到将App二进制文件保存在那里的安全性问题）

下面片段来自MainActivity--使用标准文件I/O完成拷贝工作

	// Before the secondary dex file can be processed by the DexClassLoader,
  	// it has to be first copied from asset resource to a storage location.
  	File dexInternalStoragePath = new File(getDir("dex", Context.MODE_PRIVATE),
          SECONDARY_DEX_NAME);
  	...
  	BufferedInputStream bis = null;
  	OutputStream dexWriter = null；
  	static final int BUF_SIZE = 8 * 1024;
  	try {
      bis = new BufferedInputStream(getAssets().open(SECONDARY_DEX_NAME));
      dexWriter = new BufferedOutputStream(
          new FileOutputStream(dexInternalStoragePath));
      byte[] buf = new byte[BUF_SIZE];
      int len;
      while((len = bis.read(buf, 0, BUF_SIZE)) > 0) {
          dexWriter.write(buf, 0, len);
      }
      dexWriter.close();
      bis.close();
  	} catch (. . .) {...}

接着，DexClassLoader被实例化去加载来自提取出来的二级dex文件的类库。有两种方式去调用加载了的类方法以这种方式。在示例中，类实例转换成接口，然后直接调用方法。

另一个途径是使用反射API调用方法。使用反射的优点是它不要求二级dex文件去实现任何特殊的接口，然而你应该察觉到反射很啰嗦而且慢。


  	// Internal storage where the DexClassLoader writes the optimized dex file to
  	final File optimizedDexOutputPath = getDir("outdex", Context.MODE_PRIVATE);

  	DexClassLoader cl = new DexClassLoader(dexInternalStoragePath.getAbsolutePath(),
                                         optimizedDexOutputPath.getAbsolutePath(),
                                         null,
                                         getClassLoader());
  	Class libProviderClazz = null;
  	try {
      	// Load the library.
      	libProviderClazz =
          cl.loadClass("com.example.dex.lib.LibraryProvider");
      	// Cast the return object to the library interface so that the
      	// caller can directly invoke methods in the interface.
      	// Alternatively, the caller can invoke methods through reflection,
      	// which is more verbose.
      	LibraryInterface lib = (LibraryInterface) libProviderClazz.newInstance();
      	lib.showAwesomeToast(this, "hello");
  	} catch (Exception e) { ... }

##构建过程
---

为了churn out 两个分开的dex文件，我们需要tweak标准的构建过程。完成这个戏法，我们只要简单地修改"-dex"目标，在项目的Ant build.xml中。

修改了的"-dex"目标执行以下操作：

创建两个临时目录去存储.class文件，用来被转换为默认的dex和二级dex。

有选择性的从Project_Root/bin/classes拷贝.class文件到这两个临时目录。

      <!-- Primary dex to include everything but the concrete library
                 implementation. -->
            <copy todir="${out.classes.absolute.dir}.1" >
                <fileset dir="${out.classes.absolute.dir}" >
                        <exclude name="com/example/dex/lib/**" />
                </fileset>
            </copy>
            <!-- Secondary dex to include the concrete library implementation. -->
            <copy todir="${out.classes.absolute.dir}.2" >
                <fileset dir="${out.classes.absolute.dir}" >
                        <include name="com/example/dex/lib/**" />
                </fileset>
            </copy>     

将两个临时目录的.class文件转换成两个分开的dex文件

添加二级dex文件到一个jar文件--DexClassLoader的预期输入格式。最后，存储jar文件到项目的assets目录下。


    <!-- Package the output in the assets directory of the apk. -->
            <jar destfile="${asset.absolute.dir}/secondary_dex.jar"
                   basedir="${out.absolute.dir}/secondary_dex_dir"
                   includes="classes.dex" />


To kick-off the build, you execute ant debug (or release) from the project root directory.

That’s it! In the right situations, dynamic class loading can be quite useful.
