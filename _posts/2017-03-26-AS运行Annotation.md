## AS 运行 Annotation

注解分为运行时注解和编译时注解

### 运行时注解

原理就是代理，使用 Proxy 




### 编译时注解

通过 apt(Annotation Processing Tools) 技术，原理是在某些代码元素上（如类型、函数、字段等）添加注解，在编译时编译器会检查 AbstractProcessor 的子类，并且调用该类型的 process 函数，然后将添加了注解的所有元素都传递到 process 函数中，使得开发人员可以在编译器进行相应的处理，


#### 步骤

1. 使用 AS 创建一个 Android 项目，然后创建一个 module，一定要选择 java library


2. 设置 app 的 build.gradle，在 android 下添加
```gradle
	compileOptions {
	   sourceCompatibility JavaVersion.VERSION_1_7
	   targetCompatibility JavaVersion.VERSION_1_7
	}
```
在 java library 的 build.gradle 添加

sourceCompatibility = 1.7
targetCompatibility = 1.7

3. 在 java library 创建注解类

```java
	package com.major;

	public @interface CustomAnnotation {
	}
```

4. 创建注解处理器
```java
	package com.major;

	import java.util.Set;

	import javax.annotation.processing.AbstractProcessor;
	import javax.annotation.processing.RoundEnvironment;
	import javax.annotation.processing.SupportedAnnotationTypes;
	import javax.annotation.processing.SupportedSourceVersion;
	import javax.lang.model.SourceVersion;
	import javax.lang.model.element.TypeElement;

	@SupportedAnnotationTypes("com.major.CustomAnnotation")
	@SupportedSourceVersion(SourceVersion.RELEASE_7)
	public class CustomAnnotationProcessor extends AbstractProcessor {

		@Override
		public boolean process(Set<? extends TypeElement> set, RoundEnvironment roundEnv) {
			// doSomething
			return true;
		}
	}
```

5. 创建 resources 告诉编译器在编译的时候使用哪个注解处理器
    * 在 java library 的 main 目录下创建文件 resources/META-INF/services/javax.annotation.processing.Processor
	* 在文件中填写处理器路径
```
	com.major.CustomAnnotationProcessor
```
6. 在 project 的 build.gradle 的 dependencies 添加 apt 插件
```gradle
	classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
```
7. 编辑 app build.gradle 

* 添加 插件
```gradle
	apply plugin: 'com.neenbedankt.android-apt'
```
* 在 dependencies 添加依赖
```java
	compile files('libs/processor.jar')
```

* 创建 task 将把生成的 jar 文件复制到 app/libs 目录中
```gradle
	task processorTask(type: Copy) {
		from('../processor/build/libs/processor.jar')
		into('libs/')
	}

	processorTask.dependsOn(':processor:build')
	preBuild.dependsOn(processorTask)
```

8. rebuild，然后可以在 app/libs 下看到生成的 lib 包


9. 在 app 中调用注解


注：
 * 如何获取 java library 的 jar 包
 使用 app 依赖 java library 的模块，然后可在 build/libs/ 下找到 jar 包


### AnnotationProcessor 中调试代码
1. 设置gradle daemon端口和JVM参数。把下面两行加入到你的gradle.properties文件。
```
	org.gradle.daemon=true
	org.gradle.jvmargs=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```
2. 运行命令 gradle daemon 来启动守护线程。

gradle --daemon

3. 在 process 方法中打断点

4. 建立 Remote Debugger 
 Edit Configurations --> + --> Remote --> 填写 host， port --> finish --> 点击 debug 按钮

 
 
 
 
 
 
 
 
 
 
 
 
 
 
 ## 参考
 
 * http://blog.csdn.net/industriously/article/details/53932425