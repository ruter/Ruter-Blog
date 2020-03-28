---
title: 初识Auto Layout的VFL
date: 2016-04-21 22:35:26
toc: true
categories: iOS
keywords: iOS, Swift, VFL, Auto Layout
description: Auto Layout 是个好东西
---

最近在学习`Swift`的过程中涉及到了自动布局方面的内容，在之前用`Objective-C`的时候也涉及过这方面的内容，不过基本都是在`storyboard`里拖线做约束，有时候遇到要修改多个约束的情况，简直就是噩梦！好在苹果自家有个`VFL`(Visual Format Language)可以比在SB里直接拖线设置约束方便修改一些(数量多的时候)，不过说实话，VFL其实也没有特别好到哪儿去;)

这里就通过设置4个`UILabel`的约束来进行演示，以了解VFL的一些概念以及使用方法。

# 简介
`VFL`全称`Visual Format Language`，是一种用于定义视图自动布局约束(`Auto Layout constraints`)的说明性语言。

Auto Layout的约束有很多属性，例如垂直布局、间隔、大小等等，这些都可以通过VFL来实现。它以一个字符串作为参数传递给`NSLayoutConstraint`这个类的方法`constraintsWithVisualFormat:options:metrics:views:`以及`constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:`。

在需要使用纯代码构建UI的项目里，VFL就可以作为一种替代storyboard的选择了 :)

# 基础
先创建一个`Single View Application`项目，然后创建3个`UILabel`并设置它们的颜色以及文本内容:
```
override func viewDidLoad() {
    super.viewDidLoad()

    let label1 = UILabel()
    label1.translatesAutoresizingMaskIntoConstraints = false
    label1.backgroundColor = UIColor.greenColor()
    label1.text = "HELLO"

    let label2 = UILabel()
    label2.translatesAutoresizingMaskIntoConstraints = false
    label2.backgroundColor = UIColor.cyanColor()
    label2.text = "AUTO LAYOUT"

    let label3 = UILabel()
    label3.translatesAutoresizingMaskIntoConstraints = false
    label3.backgroundColor = UIColor.orangeColor()
    label3.text = "VFL"

    view.addSubview(label1)
    view.addSubview(label2)
    view.addSubview(label3)
}
```

这里我们将`translatesAutoresizingMaskIntoConstraints`设置成`false`是因为我们要手动设置Label的约束而不希望系统自动产生约束。

这时候可以试试把app跑起来看下。Oops! 所有的label都重叠在一起了，我们只能看到，这是因为我们还没有设置任何约束，这些label被放在了默认的位置（屏幕的左上角），大小则因它们的内容而定。
![叠在一起的label们](/images/VFL/P1.png)

OK! 是时候请出我们今天的主角`VFL`来给这3个label设置约束了，我们先来点简单的，让每个label都从父视图的左边开始到右边结束。

在那之前，我们需要先创建一个label的`字典`(`Dictionary`)，这个我们之后会用到:
```
let viewsDictionary = ["label1": label1, "label2": label2, "label3": label3]
```

把它放在`viewDidLoad()`的底部，然后接着把下面的代码添加到它的后面:
```
for label in viewsDictionary.keys {
    view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("H:|[\(label)]|", options: [], metrics: nil, views: viewsDictionary))
}
```

来看看这段代码里都有些什么。首先是`view.addConstraints()`，这个方法会往当前视图添加一组约束；然后是前面已经提到过的`NSLayoutConstraint.constraintsWithVisualFormat()`方法，用来将VFL转换成一组约束，其中第一个和最后一个参数是最重要的，而options和metrics这里暂时不讨论。

再来看看这段代码的灵魂所在：`H:|[\(label)]|`，这里的`\(label)`会被替换成`viewDictionary`里的`Key`。开头的`H:`表示定义一个水平布局(Horizontal layout)；而管道符号`|`则表示视图的边缘；最后看到`[label1]`，表示把`label1`放在这个位置，我们可以把中括号看成是这个label1视图的边。

现在我们可以很清楚地描述`H:|[label1]|`所表达的意思：我想要把这个`label1`放在视图里，并且让它的左右边缘紧贴视图的左右边缘(即与视图同宽)。那么问题来了，这个label1是从哪里蹦出来的？这里就轮到我们前面提到的最后一个参数views了。

我们定义了一个`viewsDictionary`以键值对的形式存放了三个`UILabel`，然后作为最后一个参数传递给我们的方法。所以这个`label1`会作为key在我们提供的字典里寻找对应的值(这里指`UILabel`)产生自动布局的约束。

如果现在运行程序，我们应该会看到label们边对边贴着主视图了，但是我们只看到了橙色的写着"VFL"的label，其他的叠在一起被挡住了！不要忘了，我们只设置了水平方向的约束，还有垂直方向的没有设置嗷~
![只能看到橙色label](/images/VFL/P2.png)

我们只需要一行代码就可以把垂直方向的约束搞定:
```
view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|[label1]-[label2]-[label3]", options: [], metrics: nil, views: viewsDictionary))
```

这里和前面那段代码，除了VFL部分外其他全都一样，我们这里用了`V:`，表示要进行垂直方向的约束。这里还多了个新东西，一个减号`-`，表示这个符号所在的位置要有间隙，默认值为10 `point`s。

机智的你肯定留意到了我们的label3后面没有跟着管道符`|`，这就表示label3的底边不需要贴着主视图的底边，所以底部会留下一大片空白。

还在等什么，让程序跑起来！之后我们就会看到与主视图同宽，三个label以相同的间隔隔开。Awesome! 到这里我们已经对VFL有了初步的认识了，跟在storyboard里拖线设置约束相比，是不是好了那么一丢丢呢~
![设置了垂直约束后](/images/VFL/P3.png)

# 进阶
嗯哼，就这么简单就结束了吗？当然不是啦！我们只做了很基础的一些东西，看下运行后的label，高度看起来是不是很不顺眼呢。还有就是我们没有对底边做约束，如果最后一个label很高，那它超出主视图的部分就会被遮盖掉，这并不是我们想要看到的，所以我们需要进行一些改动，为每一个label都设置一个高度，并且对最后一个label的底边进行约束。

要完成这些非常简单，把下面这行代码替换掉前面写的垂直约束那行:
```
view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|[label1(==88)]-[label2(==88)]-[label3(==88)]-(>=10)-|", options: [], metrics: nil, views: viewsDictionary))
```

在这里我们添加了两个新东西，一个是在label里面的`(==88)`，还有一个是在label到底边之间的`-(>=10)-`。前者指定了对应label的高度，`==`表示等于，这里我们指定了每个label的高度等于88 points；后者表示最后一个label到视图底边之间的空隙应该大于等于10 points。
![为label设置了高度](/images/VFL/P4.png)

到这里你肯定发现了，关于数值的约束，我们一般会用`()`包裹起来。运行起来看看吧，感觉似乎好多了。等等，要是老板说88 points太高了，要你改成80 points，于是你一个个把`(==88)`改成了`(==80)`，可是设计师不乐意了，说80 points太傻逼了坚决不可以用这个高度，于是你又要着手把它改掉，一来二去特别蛋疼啊有木有！要是再多几个不同的view多很多约束怎么改得过来啊！

好的好的，不用抓狂先，还记得我们`constraintsWithVisualFormat()`这个方法里的第三个参数metrics吗，我们之前一直都是传了个`nil`给它，现在我们可以把它改掉了，是的，我们要用它减轻我们修改约束数据的工作量。首先要做的就是创建一个字典:
```
let metrics = ["labelHeight": 88]
```

然后把它传递给参数metrics，再对`==88`进行一点点的修改，变成上面创建的字典的键`labelHeight`，修改后是这样的:
```
view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|[label1(labelHeight)]-[label2(labelHeight)]-[label3(labelHeight)]-(>=10)-|", options: [], metrics: metrics, views: viewsDictionary))
```

就是这么简单，当遇到需要修改约束数值的时候，只需要修改字典里对应的值就可以一次解决啦XD再次运行一下，结果和之前是不是一样呢，看起来我们的约束可以正常工作，可是如果我们把模拟器设置成横屏模式的话，是不是也还能继续正常工作呢？很好，看起来也能完美地实现自动布局。
![横屏时的布局](/images/VFL/P5.png)

那接下来我们试着添加多一个label吧，添加下面的代码到`viewDidLoad()`里:
```
let label4 = UILabel()
label4.translatesAutoresizingMaskIntoConstraints = false
label4.backgroundColor = UIColor.orangeColor()
label4.text = "NEW LABEL"

view.addSubview(label4)
```

并且把新的label添加到`viewsDictionary`里，以及修改垂直方向的约束:
```
let viewsDictionary = ["label1": label1, "label2": label2, "label3": label3, "label4": label4]

view.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat("V:|[label1(labelHeight)]-[label2(labelHeight)]-[label3(labelHeight)]-[label4(labelHeight)]-(>=10)-|", options: [], metrics: metrics, views: viewsDictionary))
```

现在运行看看，在竖屏的时候一切看起来都很平静很美好对不对，试试旋转屏幕变成横屏看看吧，我们会发现橙色的label被红色的那个给遮住了很大一部分，并且`Xcode`会有一大串的信息打印出来，在最开始会看到这样一句"Unable to simultaneously satisfy constraints."简单理解就是我们把屏幕横向之后，不够位置放置4个label，那要怎么办才好啊？莫慌，抱紧我，噢不，抱紧`priority`(优先级)。
![4个label时横屏显示](/images/VFL/P7.png)

`priority`是VFL里一个很重要的概念，它的取值在1到1000之间，其中1000(默认值)表示最高优先级，不管怎么样都必须满足它，而其他级别的优先级都是可选的。

在动手写代码解决问题之前，我们需要先了解优先级的工作方式。当我们没有指定优先级，它会默认使用1000，所以当我们横屏显示的时候没有足够空间满足label的约束，自动布局就会找不到解决方法，我们设置的约束就会失效。如果我们设置label的高度为可选的优先级，例如设置成999，当空间不足的情况下，自动布局就会找办法来尽量满足我们设置的约束：压缩label的高度以适应当前的约束。

在我们这个例子中，设置了每一个的label为88 points，如果我们将每个label的高度的优先级设置为可选的(1000以下的任意数)，当我们横屏显示的时候，自动布局就会想方设法去满足我们的约束，可能是把每个label的高都压缩成78或者别的更小的值，它会尽可能将label的高设置为最接近的88的值。

所以我们需要把label高度的优先级修改为999，并且还要额外修改一个地方，让每个label都与`label1`同高。这是为了避免自动布局为了满足其他label的高能达到88而把其中一个label的高压缩到很小很小。VFL部分的代码修改成下面这样:
```
"V:|[label1(labelHeight@999)]-[label2(label1)]-[label3(label1)]-[label4(label1)]-(>=10)-|"
```

我们把`@999`加到label1的`labelHeight`后面，表示对它的高度约束设置优先级999，然后让所有label的高都跟随label1变化。

到这里为止，我们的设置已经完成了！快运行一下看看效果如何，是不是很棒！现在不管是竖屏还是横屏，都能很好地展示我们的label啦~来，给自己一点掌声(啪啪啪啪啪啪啪啪啪):)
![设置优先级后](/images/VFL/P6.png)

# 小结
使用storyboard进行布局，可以很方便直观地看到各个约束以及效果，并且会有提示帮助我们修复一些约束的冲突。如你所见，我们使用代码来设置约束同样显得很简单，在团队协作开发的时候，使用代码进行界面的布局约束显得尤为重要。这里无意比较二者孰优孰劣，将这两种方法根据不同场景混合使用，反而能更大地发挥各自的优势。

这篇博客只是简单粗暴地用了个简单粗暴的例子来对VFL的使用进行了一些讲解，还有很多没有涉及到的方面希望大家可以通过苹果官方的文档进行学习，以及对于本文出现的错误，恳请大家指正:)

# 参考
[iOS Developer Library - Auto Layout Guide](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html)

---

**本文采用** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 许可协议。转载或引用时请遵守协议内容！

**本文地址** http://blog.ruterly.com/2016/04/21/An-introduction-to-VFL/
