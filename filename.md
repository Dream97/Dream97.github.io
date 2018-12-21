---
title: Dagger2使用笔记
tags: 控制反转,依赖注入,解耦
grammar_cjkRuby: true
---

[toc]

## 1. 优点
> 文章《详解Dagger2》中这样说

- 依赖的注入和配置，是独立于组件之外。

- 因为对象是在一个独立、不耦合的地方初始化，所以当注入抽象方法的时候，我们只需要修改对象的实现方法，而不用大改代码库。

- 依赖可以注入到一个组件中：我们可以注入这些依赖的模拟实现，这样使得测试更加简单。

也就是说使用Dagger2依赖注入的对象，它的初始化独立于将要使用它的对象，这样做到扥更大程度两个类之间的解耦。另外，也同样避免了空指针空指针的出现，而且注入的对像的生命周期也以之相对应的被注入的对象保持一致。

**举个栗子**
在MVP模式中，Persenter往往与View绑定在一起，所以我们可以将Persenter注入到View中，这样有利于两者间的解耦，同时两者的生命周期保持一致，避免其他问题

## 2. 如何使用
既然Dagger2在项目中用处不少，那我们就应该撸起袖子开始干吧。

### 2.1 了解注解

|注解名称|注解意义|
|:-----|:----------|
|@Inject|通常是注入的类的构造函数前和需要注入的位置|
|@Module|使用@Module注解的一个类，通常用于Dagger在注入某个实例时通过在该类中找到实例的方法。也就是某实例的构造函数无需使用@Inject注解，在@Module中提供方法获得实例|
|@Provide|在使用@Module注解的类中配合使用，意思为提供某实例的获取方法|
|@Component|意思为桥接，也有注入器的意思，|
|@Scope|通过自定义注解限定作用域，控制生命周期与View一致也是它的作用|
|@Qualifier|类型的区分，如我使用不同域名获取RetrofitService，可以@BaiduService和@AlipayService很好的区分Service类型|

在初步了解注解类型后，接下来就用实例来说明如何使用Dagger2，Dagger2的注入方式有好几种，下面以一一分析一下。

> 下面例子很多是从皇叔的《Android进阶之光》中改编过来，顺便安利一下皇叔的书，进阶Android真的很不错
### 2.2 简单成员变量注入
![简单成员变量注入](https://img-blog.csdnimg.cn/20181219194015556.png)
根据流程图，我需要把Watch的实例注入到MainActivity中,可以在Watch的构造函数中使用@Inject注解
```java
public class Watch {
	@Inject
	public Watch () {
	
	}
	
}
```
还需要一个注入器把Watch桥接到MainActivity中
```java
@Component
public interface ActivityComponent {
	void inject(MainActivity activity);
}
```
Builder当前项目，生成DaggerActivityComponent类，调用我们定义的inject方法，注入到目标MainActivity中

```java
public MainActivity extends Activity {
	@Inject
	Watch watch;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		DaggerActivityComponent.create().inject(this);
		//这里就可以使用watch里面的方法了
	}
}
```

### 2.3 使用@Provides实现成员变量注入
![使用@Provides实现成员变量注入](https://img-blog.csdnimg.cn/20181221192707441.png)
和简单成员变量注入对比，多了一个Module类，在依赖对象的构造函数前也不需要@Inject注解，而是通过在module中使用@Provider注解实现。这种情况适用于不方便在成员的构造函数加上注解的情况，如使用第三方类库。
```java
public class Tree{
	public Tree() {
	}
}
```
在Module类中使用@Provides注解
```java
@Module
public class TreeModule{
	@Provides
	public Tree provideTree() {
		return new Tree();
	}
}
```
还需要在告诉Component类，可以从Module类中获取依赖对象
```java
@Component(modules = TreeModule.class) 
public interface ActivityComponent {
	void inject(MainActivity mainActivity);
}
```
接下来就可以获取
```java
public MainActivity extends Activity {
	@Inject
	Tree tree;
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		DaggerActivityComponent.create().inject(this);
		//这里就可以使用tree里面的方法了
	}
}
```
## 参考资料

[1][详解Dagger2](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0519/2892.html)
[2][Android进阶之光](https://me.csdn.net/itachi85)