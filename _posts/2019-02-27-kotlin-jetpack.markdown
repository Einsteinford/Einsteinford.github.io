---
layout: post
title: Kotlin 和 JetPack 的项目实战（一）
date: 2019-02-27 17:22:24.000000000 +08:00
---

# 搭建基于 MVVM 的项目框架

---
### 前言

从谷歌在 2017 年的 Google IO 宣布 Kotlin 成为 Android 开发的官方语言开始，已经过去将近 2 年了，Kotlin 越来越被开发者所关注，在 Github 的开源项目中使用这门语言的也呈上升趋势。

虽然批评的声音也不少，说 Kotlin 只不过是语法糖的，拿来跟 Java 8/9/10 对比表示不过如此的，但是针对 Android 开发而言，这门语言是有生产力的，具体我在项目中可能会插入一些个人感受。

### 1. 浅谈 MVP 和 MVVM

- #### MVP

公司大概 1 年半前开始改为用 MVP 模式来开发代码，相比曾经上千行的 Activity 代码，实在进步了不少，V (View) 和 P (Presenter) 之间通过接口来互相访问与操作，一定程度抽象了代码逻辑，确实有利于维护
基本上代码目录类似这个

![MVP目录](http://upload-images.jianshu.io/upload_images/2527601-0873faeb3f212dc0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Model 层用了 Retrofit 和 RxJava 进行网络的或者本地的数据获取，比较稳定，就不进行对比了，因为也没区别

其中的 MainContract 代码可能是这样子写的

```java
public class MainContract {
    interface View extends BaseView{
        void showAd(Adverts adverts);
    }

    interface Presenter extends BasePresenter{
        void loadAd();
        Adverts getAppAdvert() ;
    }
}
```
暂且不管我们在 BaseView 和 BasePresenter 里做了什么操作，大致上看方法就是一个获取广告数据然后把广告 List 传递到 View 进行 UI 操作的功能。

之后让 MainActivity 去实现 View 接口 而 MainPresenter 去实现 Presenter 接口，在初始化时，互相都持有了对方的接口实例。

随着生命周期的变化，可能出现 NPE，或者内存泄露，这确实也是我们上一个项目上线测试后出现的最多 Bug，添加了不少判空条件，更加加深了我去尝试其它设计模式的愿望。

- #### MVVM

时隔一年，谷歌在 2018 年的 Google IO 中发布了 JetPack 支持包，主打众多开发库随意添加使用，互不干扰，还顺便把 v7 和 v4 支持包全改了个包名叫 androidx , 如何迁移到 androidx 可以之后再谈。

[jetpack官方介绍](https://developer.android.google.cn/jetpack/)

为了完成 MVVM 的设计，挑选了其中的 LiveData 和 ViewModel 进行使用。

LiveData 其实跟 RxJava 一样属于观察者模式的第三方库，一定程度上来说是重复的，奈何各有优势，所以在数据处理中继续使用 Retrofit 和 RxJava 这套搭配，而在 UI 操作上添加了 LiveData 用于通知 V 端进行页面的刷新。

> - LiveData 优势和劣势

    优势：
    1. 绑定生命周期，不会内存泄露，放心把数据交给他保管
    2. 默认只在 Activity 和 Fragment 在 started 或 resumed 2 种状态时通知 UI 更新数据
    3. 当 UI 处于started 或 resumed 状态外，但是还没销毁之前，一直会接收更新数据，在 UI 处于可见状态时，只会通知最新的数据到 UI。
    4. 屏幕旋转重建后的 View 仍然能利用之前数据。
    5. 以及其它。

    劣势：
    1. MutableLiveData 只能将完整的新数据作为值覆盖旧数据才会通知观察者，也就是说利用 getValue() 方法对旧数据进行微小修改也没办法触发通知。

毕竟是实战中发现的优势劣势，总结得不完全，其实也并不想长长写一大段干涩的字，请多包涵。

#### 插播一个 kt 语言很有意思的实例构造方法，在 AbsFragment 主要是做了一个为页面添加顶部操作栏的功能

![image](http://upload-images.jianshu.io/upload_images/2527601-ed79eff2ecbc474a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 有兴趣可以看这一部分，不然跳过以下一大段

> - 构建 AbsFragment 基础类

```kotlin
abstract class AbsFragment : Fragment() {
    private var titleBar: TitleBar? = null

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        if (getContentViewLayoutID() != 0) {
            titleBar = getTitleBar()
            return if (titleBar != null) {
            //略，在新建的 RelativeLayout 顶部添加自定义 titleBar，再添加主布局 layout
            } else {
                inflater.inflate(getContentViewLayoutID(), container, false)
            }
        }
        return super.onCreateView(inflater, container, savedInstanceState)
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        if (titleBar != null) {
            titleBar!!.apply {
            //略，从 TitleBar 实例中获取自定义 titleBar 所需要显示的数据,以及默认值
            }
        }
        initView()
    }
    /**默认初始化页面功能方法 */
    protected abstract fun initView()

    /** 返回布局layout*/
    protected abstract fun getContentViewLayoutID(): Int

    /**
     * 是否添加title栏
     * 不添加 返回 null
     * 需要添加 返回 [initTitleBar] 方法
     */
    protected abstract fun getTitleBar(): TitleBar?

    /** 就是这一段比较有趣 */
    protected fun initTitleBar(init: TitleHelper.() -> Unit): TitleBar {
        return initAny(TitleBar(), init)
    }
}
```

可能初入门 kt 的朋友不太了解它的 lambda 怎么写，举个栗子
```kotlin
fun <T> lock(body: () -> T): T {
return body()
}
```
以上方法要求返回泛型 T ，直接返回从参数中得到的 body 函数 "()" 空括号代表函数无参数，"
-> T "代表函数将会返回 泛型 T
对使用函数 lock 的人来说
```kotlin
//大括号内就是所填入的 body 函数
lock<String>(body = { "" })
//kt 约定，只有一个 Lambda 表达式的方法应该将大括号移到小括号外侧，于是变成以下
lock<String>() { "" }
// 其实空的小括号也可以省略，尖括号内的泛型也由于 kt 语言的自动推断功能，会根据大括号内的返回值自动变化，故又可以省略
lock { "" }
```

回到 initTitleBar 这个方法，返回的是一个 kt 的扩展函数
```kotlin
/**
 * 创建类型安全的构建器的方法
 */
fun <T : Any> initAny(any: T, init: T.() -> Unit): T {
    any.init()
    return any
}
```
跟上面的例子很像，但"T.() -> Unit" 是⼀个带接收者的函数类型。这意味着我
们需要向函数传递⼀个 T 类型的实例，并且我们可以在函数内部调⽤该实例的成员。

关于 TitleBar 方法很简单，DslMarker 注解暂时不谈
```kotlin
class TitleBar : TitleHelper {
    var title: String = ""

    override fun title(text: String) {
        title = text
    }
    ...
}

@DslMarker
annotation class TitleBarMarker

//抽象，避免内部实例被直接操作
@TitleBarMarker
interface TitleHelper {

    @TitleBarMarker
    fun title(text: String)
    ...
}
```

所以 AbsFragment 的子类实现类似这样子，只调用想要不同于默认值的部分方法
```kotlin
override fun getTitleBar() = initTitleBar {
    title("分类")
}
```

#### 插播结束

> - 构建 BaseFragment 基础类

我希望在 BaseFragment 中实现一些基础的监听者模式，基本只用到 ViewModel 和 LiveData 2个库来完成

#### 那先从 ViewModel 说起
```kotlin
abstract class BaseViewModel : ViewModel() {
    /** 显示布局里的数据加载view */
    private val _showLoadingView = MutableLiveData<Boolean>()
    /** LiveData只有get方法 */
    val showLoadingView: LiveData<Boolean>
        get() = _showLoadingView
    /** 调用set方法即可决定是否显示布局里的数据加载view */
    fun setLoadingView(loading: Boolean) {
        if (_showLoadingView.value != loading) {
            _showLoadingView.value = loading
        }
    }
    /* 基本上就是在初始化页面需要请求的数据的时候，调用此方法 */
    abstract fun sendRequest()
}
```

略微简化来下代码，就变成如上到代码，MutableLiveData 的公共方法有 setValue() 和 postValue() , 而他的父类 LiveData 的 setValue() 是个 protected 方法 ，可以对外隐藏赋值操作，一定程度上让数据操作完全局限在 ViewModel 中。

#### 再来说说 BaseFragment

```kotlin
abstract class BaseFragment<T : BaseViewModel> : AbsFragment(), BaseViewImp<T> {
    lateinit var viewModel: T

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
        viewModel = if (getModelFactory() == null) {
            ViewModelProviders.of(this).get(getModel())
        } else {
            ViewModelProviders.of(this, getModelFactory()).get(getModel())
        }

        return super.onCreateView(inflater, container, savedInstanceState)
    }

    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        observeLoadingView()
    }

    private fun observeLoadingView() {
        viewModel.showLoadingView.observe(viewLifecycleOwner, Observer { showLoad ->
                getLoadingView().let { loadingView ->
                    loadingView?.visibility = if (isLoading) View.VISIBLE else View.GONE
                    dosth...
                }
            }
        })
    }

    /** 需要时子类返回loading页面 */
    protected open fun getLoadingView(): View? {
        return null
    }
}

interface BaseViewImp<T> {

    /** 返回ViewModel的类*/
    fun getModel(): Class<T>

    /** 当需要给viewModel传参时，返回ViewModel的工厂*/
    fun getModelFactory(): ViewModelProvider.Factory? {
        return null
    }
}
```

几个 kotlin 语法我啰嗦几句，var lateinit 只能说是提示编译器，这个变量不要因为没有初始化就给我报错，我会在使用前择期初始化，但是到运行时忘记初始化了，也只有乖乖接收 NPE 错误的选择了。

let方法是前值非空就执⾏代码的简写

getModel() 返回 BaseViewModel 的子类 Class，而因为 ViewModel 初始化的特殊性，他是由 Fragemnt 或者 Activity 创建并且保管的，传参数需要通过实现 ViewModelProvider.Factory 接口来完成，例如以下这个类：

```kotlin
class DownloadFactory(
        val novelId: String
) : ViewModelProvider.NewInstanceFactory() {

    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return DownloadViewModel(novelId) as T
    }
}
```

参数 novelId 就传递到了类 DownloadViewModel(val novelId : String) 中啦

---

### 以上是一个我在项目中构思的简易 MVVM 框架，为了便于介绍，删除了不少代码，如果按照这些步骤有什么觉得不好的，欢迎交流
