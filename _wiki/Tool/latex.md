---
layout: wiki
title: "LaTeX"
date: 2016-12-08 22:22
updated: 2016-12-09 11:15
keywords: LaTeX
categories: Tool
---

# 排版工具
以`XeLaTeX`为主。

# 控制序列
以反斜杠`\`开头，以第一个**空格**或**非字母**的字符结束的一串文字，他们并不被输出，但是他们会影响输出文档的效果。

```
\documentclass{article}
%这里是导言区
\begin{document}
Hello, world!
\end{document}
```

`\documentclass{article}`的控制序列是`documentclass`，它后面紧跟着的`{article}`代表这个控制序列有一个必要的**参数**，该参数的值为`article`。这个控制序列的作用是调用名为`article`的文档类。

注意: `TeX`的控制序列是**大小写敏感**的

控制序列`begin`总是与`end`成对出现的，这两个控制序列以及他们之间的内容被称为`环境`，它们之后的第一个必要参数总是一致的，被称为`环境名`。

注意: 只有在`document`环境中的内容，才会被正常输出到文档中或是作为控制序列对文档产生影响

`\begin{document}`与`\documentclass{article}`之间的部分被称为`导言区`。导言区中的控制序列通常会影响到整个输出文档，如页面大小、页眉页脚样式、章节标题样式等。

# 中文支持
只需要使用几个简单的`宏包`，就能完成中文支持了，所谓宏包，就是一系列控制序列的合集。

宏包的调用方式为`\usepackage{...}`

可以使用`CTeX`或`xeCJK`宏包来处理中英文混排的文档，下面以`xeCJK`为例

```
\documentclass{article}
\usepackage{xeCJK} %调用 xeCJK 宏包
\begin{document}
你好，world!
\end{document}
```

# 标题、作者、日期
为文档定义标题、作者和日期，之后使用控制序列`maketitle`按预定的格式展现

```
\documentclass{article}
\usepackage{xeCJK}

\title{Learn LaTeX}
\author{Ruter}
\date{\today}

\begin{document}
\maketitle
你好，world!
\end{document}
```

# 章节和段落
在文档类`article`中，定义了五个控制序列来调整行文组织结构

- `\section{...}`
- `\subsection{...}`
- `\subsubsection{...}`
- `\paragraph{...}`
- `\subparagraph{...}`

举个例子

```
\documentclass{article}
\usepackage{xeCJK}

\title{Learn LaTeX}
\author{Ruter}
\date{\today}

\begin{document}
\maketitle

\section{你好中国}
中国在East Asia.
\subsection{Hello Beijing}
北京是capital of China.

\subsubsection{Hello Dongcheng District}
\paragraph{Tian'anmen Square}
is in the center of Beijing
\subparagraph{Chairman Mao}
is in the center of 天安门广场。

\subsection{Hello 山东}
\paragraph{山东大学} is one of the best university in 山东。

\end{document}
```

# 插入目录
要插入目录，使用`\tableofcontents`控制序列，置于`\maketitle`前

# 数学公式
使用`AMS-LaTeX`提供的数学功能，需要在导言区加载`amsmath`宏包

```
\usepackage{amsmath}
```

## 模式
LaTeX的数学模式有两种：`行内模式 (inline)`和`行间模式 (display)`。前者在正文的行文中，插入数学公式；后者独立排列单独成行，并自动居中。

使用`$ ... $`可以插入行内公式，使用`\[ ... \]`可以插入行间公式，如果需要对行间公式进行编号，则可以使用`equation`环境

```
\begin{equation}
...
\end{equation}
```

注意：若不需编号，可以在`equation`后加一个`*`，即`equation*`

## 上下标
在数学模式中，需要表示上标，可以使用`^`来实现，下标则是`_`。它默认**只作用于之后的一个字符**，如果想对连续的几个字符起作用，将这些字符用花括号`{}`括起来

```
\documentclass{article}
\usepackage{amsmath}

\begin{document}

Einstein 's $E=mc^2$.

\[ E=mc^2. \]

\begin{equation}
E=mc^2.
\end{equation}

\end{document}
```

注意：行内公式和行间公式对**标点**的要求是不同的：行内公式的标点，应该放在数学模式的**限定符之外**，而行间公式则应该放在数学模式**限定符之内**

## 根式与分式
根式用`\sqrt{...}`来表示，分式用`\frac{...}{...}`来表示（第一个参数为分子，第二个为分母）

```
\documentclass{article}
\usepackage{amsmath}
\begin{document}

$\sqrt{x}$, $\frac{1}{2}$.

\[ \sqrt{x}, \]

\[ \frac{1}{2}. \]

\end{document}
```

注意：在行间公式和行内公式中，分式的**输出效果是有差异的**。如果要强制行内模式的分式显示为行间模式的大小，可以使用`\dfrac`, 反之可以使用`\tfrac`

## 运算符
需要用控制序列生成的运算符有

```
\[ \pm\; \times \; \div\; \cdot\; \cap\; \cup\;
\geq\; \leq\; \neq\; \approx \; \equiv \]
```

分别是`±`, `×`, `÷`, `∙`, `∩`, `∪`, `≥`, `≤`, `≠`, `≈`, `≣`

连加、连乘、极限、积分等大型运算符分别用`\sum`, `\prod`, `\lim`, `\int`生成。他们的上下标在行内公式中被压缩以适应行高。可以用`\limits`和`\nolimits`来强制显式地指定是否压缩这些上下标

```
$ \sum_{i=1}^n i\quad \prod_{i=1}^n $

$ \sum\limits _{i=1}^n i\quad \prod\limits _{i=1}^n $

\[ \lim_{x\to0}x^2 \quad \int_a^b x^2 dx \]

\[ \lim\nolimits _{x\to0}x^2\quad \int\nolimits_a^b x^2 dx \]
```

多重积分可以使用`\iint`, `\iiint`, `\iiiint`, `\idotsint`等命令输入

```
\[ \iint\quad \iiint\quad \iiiint\quad \idotsint \]
```

## 定界符
各种括号用`()`, `[]`, `\{\}`, `\langle\rangle`等命令表示，`amsmath`宏包推荐用`\lvert\rvert`和`\lVert\rVert`表示`|`和`||`

`amsmath`宏包推荐使用`\big`, `\Big`, `\bigg`, `\Bigg`等一系列命令放在上述括号**前面**以调整分隔符的大小

```
\[ \Biggl(\biggl(\Bigl(\bigl((x)\bigr)\Bigr)\biggr)\Biggr) \]

\[ \Biggl[\biggl[\Bigl[\bigl[[x]\bigr]\Bigr]\biggr]\Biggr] \]

\[ \Biggl \{\biggl \{\Bigl \{\bigl \{\{x\}\bigr \}\Bigr \}\biggr \}\Biggr\} \]

\[ \Biggl\langle\biggl\langle\Bigl\langle\bigl\langle\langle x
\rangle\bigr\rangle\Bigr\rangle\biggr\rangle\Biggr\rangle \]

\[ \Biggl\lvert\biggl\lvert\Bigl\lvert\bigl\lvert\lvert x
\rvert\bigr\rvert\Bigr\rvert\biggr\rvert\Biggr\rvert \]

\[ \Biggl\lVert\biggl\lVert\Bigl\lVert\bigl\lVert\lVert x
\rVert\bigr\rVert\Bigr\rVert\biggr\rVert\Biggr\rVert \]
```

## 省略号
省略号用`\dots`, `\cdots`, `\vdots`, `\ddots`等命令表示。`\dots`和`\cdots`的**纵向位置不同**，前者一般用于**有下标的序列**

```
\[ x_1,x_2,\dots ,x_n\quad 1,2,\cdots ,n\quad
\vdots\quad \ddots \]
```

## 矩阵
`amsmath`的`pmatrix`, `bmatrix`, `Bmatrix`, `vmatrix`, `Vmatrix`等环境可以在矩阵两边加上各种分隔符，同一行元素用`&`连接，`\\`表示换行

```
\[ \begin{pmatrix} a&b\\c&d \end{pmatrix} \quad
\begin{bmatrix} a&b\\c&d \end{bmatrix} \quad
\begin{Bmatrix} a&b\\c&d \end{Bmatrix} \quad
\begin{vmatrix} a&b\\c&d \end{vmatrix} \quad
\begin{Vmatrix} a&b\\c&d \end{Vmatrix} \]
```

使用`smallmatrix`环境，可以生成行内公式的小矩阵

```
Marry has a little matrix $ ( \begin{smallmatrix} a&b\\c&d \end{smallmatrix} ) $.
```

## 多行公式
无须对齐的**长公式**可以使用`multline`环境，如不需编号，可用`multline*`代替

```
\begin{multline}
x = a+b+c+{} \\
d+e+f+g
\end{multline}
```

需要对齐的公式，可以使用`aligned`**次环境**来实现，它必须包含在数学环境之内

```
\[
\begin{aligned}
x ={}& a+b+c+{} \\
&d+e+f+g
\end{aligned}
\]
```

注意：对齐的位置为公式内不同行的`&`所标记的位置，上式中为`a`和`d`对齐，等式中的`{}`为空格（**测试结果，未查证**）

## 公式组
无需对齐的公式组可以使用`gather`环境，需要对齐的公式组可以使用`align`环境。这两个环境都带有编号，如果不需要编号可以使用带`*`的版本

```
\begin{gather}
a = b+c+d \\
x = y+z
\end{gather}

\begin{align}
a &= b+c+d \\
x &= y+z
\end{align}
```

## 分段函数
分段函数可以用`cases`次环境来实现，它必须包含在数学环境之内

```
\[ y=
\begin{cases}
-x,\quad x\leq 0 \\
x,\quad x>0
\end{cases} 
\]
```

# 图片和表格

## 图片
在LaTeX中插入图片可以用`graphicx`宏包提供的`\includegraphics`命令

```
\documentclass{article}
\usepackage{graphicx}

\begin{document}
\includegraphics{a.jpg}
\end{document}
```

若需要调整图片大小，可以用`\includegraphics`控制序列的可选参数来控制

```
\includegraphics[width = .8\textwidth]{a.jpg}
```

以上参数表示图片的宽度会被缩放至页面宽度的百分之八十，图片的总高度会按比例缩放

## 表格
`tabular`环境提供了最简单的表格功能。它用`\hline`命令表示横线，在列格式中用`|`表示竖线；用`&`来分列，用`\\`来换行；每列可以采用居左、居中、居右等横向对齐方式，分别用`l`、`c`、`r`来表示

```
\begin{tabular}{|l|c|r|}
 \hline
操作系统& 发行版& 编辑器\\
 \hline
Windows & MikTeX & TexMakerX \\
 \hline
Unix/Linux & teTeX & Kile \\
 \hline
Mac OS & MacTeX & TeXShop \\
 \hline
通用& TeX Live & TeXworks \\
 \hline
\end{tabular}
```

## 浮动体
插图和表格通常需要占据大块空间，所以在使用文字处理软件中经常需要调整他们的位置。`figure`和`table`环境可以自动完成这样的任务。这种自动调整位置的环境称作`浮动体(float)`，以`figure`为例

```
\begin{figure}[htbp]
\centering
\includegraphics{a.jpg}
\caption{有图有真相}
\label{fig:myphoto}
\end{figure}
```

`htbp`选项用来指定插图的理想位置，这几个字母分别代表`here`, `top`, `bottom`, `float page`，也就是就这里、页顶、页尾、浮动页(专门放浮动体的单独页面) 。`\centering`用来使插图居中；`\caption`命令设置插图标题，LaTeX会自动给浮动体的标题加上编号，引用时使用`\ref`自动产生图表对应的编号

注意：`\label`应该放在标题命令之后

# 版面设置
## 页眉页脚
设置页眉页脚，推荐使用`fancyhdr`宏包，如果要在页眉左边写上名字，中间写上今天的日期，右边写上电话；页脚的正中写上页码；页眉和正文之间有一道宽为`0.4pt`的横线分割，可以在导言区加上如下几行

```
\usepackage{fancyhdr}
\pagestyle{fancy}
\lhead{\author}
\chead{\date}
\rhead{131xxxxxxxx}
\lfoot{}
\cfoot{\thepage}
\rfoot{}
\renewcommand{\headrulewidth}{0.4pt}
\renewcommand{\headwidth}{\textwidth}
\renewcommand{\footrulewidth}{0pt}
```

## 首行缩进
在导言区调用`\usepackage{indentfirst}`可以让每一段的段首产生缩进，添加控制序列`\setlength{\parindent}{\ccwd}`可以调整首行缩进的大小，`\ccwd`是当前字号下一个中文汉字的宽度

## 行间距和段间距
可以通过`setspace`宏包提供的命令来调整行间距，在导言区添加如下内容，可以将行距设置为字号的`1.5`倍

```
\usepackage{setspace}
\onehalfspacing
```

可以通过修改`\parskip`的值来调整段间距

```
\addtolength{\parskip}{.4em}
```

表示在原有的基础上，增加段间距`0.4em`。如果需要减小段间距，只需将该数值改为负值