---
layout: post
title: Agda 中的证明，从零到一
category: Agda
tags: Agda, Proof
keywords: Agda, Proof
description: Proof in Agda, from 0 to 1
inline_latex: true
---

类型则命题，程序则证明。这句话表达了定理证明的一个很重要的思想。

我一开始就没有搞懂这句话在说什么。
在我自认为搞懂的时候，我把我以前没有搞懂的原因归结为我看的教程太垃圾了。

理解这个问题的同时我也理解了之前一个 Haskell 关于 `IO Monad` 的问题。

## 前置知识

这是一篇面向略懂 dependent type 的人的定理证明教程，然后你要看得懂类 Haskell 的语法。
因为我在学这个的时候就是只会点 Haskell ，然后用过 GADT 和 type family 模拟过 dependent type 。

给出一些参考资料：

+ [一个比较简单的 Haskell 入门教程 *Learn you a Haskell*](http://learnyouahaskell.com/chapters)
+ [虎哥介绍的 GADT](https://www.zhihu.com/question/67043774/answer/249019401)
+ [介绍 GADT 的 CodeWars Kata: Singletons](https://www.codewars.com/kata/singletons)
+ [介绍 GADT 的 CodeWars Kata: Scott Encoding](https://www.codewars.com/kata/scott-encoding)

## 声明在前面

由于 Agda 语言的特殊性，本文将使用 LaTeX 和代码块来共同展示代码。
前者是为了保证字符的正确显示，后者是为了方便读者复制代码。

本文不讲 Agda 基本语法和 Emacs 的使用。可能以后会有另外的文章。

本文主要内容是帮助一个没接触过定理证明但是接触过 dependent type 的人（这就是我接触定理证明之前的状态）理解一个非常非常简单的定理证明的例子。

## 如何理解定理证明

首先，我们已经知道，我们这是要用类型表达命题，类型对应的实现来证明这个命题的正确性。

命题中的基本元素一般是值的类型(而且很多时候都是代数数据类型)，也就是 $ p \Rightarrow q $ 的那个 $ p $ 或者 $ q $ 。
而这个 $ \Rightarrow $ 对应的就是 "函数" 这一概念，它组合了两个类型，表达了 "推出" 这一逻辑概念。

比如，我实现了一个这样的类型的函数：

$$
\DeclareMathOperator{Set}{Set}
\DeclareMathOperator{refl}{refl}
\DeclareMathOperator{proof}{proof}
\DeclareMathOperator{data}{data}
\DeclareMathOperator{where}{where}
\DeclareMathOperator{with}{with}
p \rightarrow q
$$

```agda
p → q
```

那么这个函数的实现就是

> 如果 p 成立，则 q 成立

这个命题的证明。

再比如，我实现了一个这样的类型的函数：

$$
p \rightarrow q \rightarrow r
$$

```agda
p → q → r
```

那么这个函数的实现就是

> 如果 p 和 q 成立，那么 r 成立

这个命题的证明。

这就是 "类型则命题，程序则证明" 的含义。

在 Agda 中，上面的代码应该写成这样：

$$
\proof : \{p\ q\ r : \Set\} \rightarrow p \rightarrow q \rightarrow r
$$

```agda
proof : {p q r : Set} → p → q → r
```

下面我们看一些实例。

## refl 与相等性

之所以我没有再学习 Idris 就是因为那些教程没说 Refl 是啥 (Idris 叫 `Refl` ， Agda 叫 `refl`) 就直接在代码里面用了，我看的时候就一脸蒙蔽，还以为是我智商太低没看懂他 implicit 的东西。
但是好在我看了一坨很友好的 Agda 代码后民白了。

首先，我们可以定义这样一个用来表示相等关系的 GADT ，用 $ \equiv $ 表示他（标准库的定义在
[`Agda.Builtin.Equality`](http://agda.readthedocs.io/en/v2.5.3/language/built-ins.html#equality)
中（不需要完全看懂类型签名，只需要看懂"这个 GADT 只有一个类型构造器"这一事实））：

$$
\begin{align*}
& \data\ \_{\equiv}\_ : \{a\} \{A : \Set a\} (x : A) : A \rightarrow \Set a\where \\
&\ \ \refl : x \equiv x
\end{align*}
$$

```agda
data _≡_ : {a} {A : Set a} (x : A) : A → Set a where
  refl : x ≡ x
```

然后我们可以用它进行一些证明。比如我们来证明相等性的传递性，也就是

> 如果 a $ \equiv $ b 并且 b $ \equiv $ c ，那么 a $ \equiv $ c

。然后我们来看看这个命题对应的类型：

$$
\_{\leftrightarrows}\_ : \{A : \Set\} \{a\ b\ c : A\} \rightarrow a \equiv b \rightarrow b \equiv c \rightarrow a \equiv c
$$

```agda
_⇆_ : {A : Set} {a b c : A} → a ≡ b → b ≡ c → a ≡ c
```

那么我们要怎么实现它，也就是证明它呢？

我一开始写下了这样的东西：

$$
\_{\leftrightarrows}\_\ ab\ bc =\ ?
$$

```agda
_⇆_ ab bc = ?
```

然后我就不知道该怎么办了。

事实上，这个原本就很简单的证明被我想复杂了。
因为这个定理是不证则明的，那么我们要如何表达，如何通过 `ab`, `bc` 这两个模式匹配出来的结果进行变换得到这个不证则明的定理呢？

首先这个模式匹配的参数就不应该这样通配地用 `ab`, `bc` 来表达。
我们应该把这两个相等关系 (他们的本质是 GADT) 给模式匹配出来。

由于直接写 `ab`, `bc` 什么都得不出来，我于是尝试将 `ab` `bc` 用模式匹配消耗掉，然后 Agda 直接在右边给我自动填入了 `refl` ，然后好像就 Q.E.D 了：

$$
\_{\leftrightarrows}\_\ \refl\ \refl =\refl
$$

```agda
_⇆_ refl refl= refl -- 编译通过！
```

这是为什么呢？我们来分别看下这两种写法的含义。

### 使用 ab bc

这样的话实际上是把 $ a \equiv b $ 和 $ b \equiv c $ 两个条件当成了"变量"而不是作为"条件"。
也就是说，当使用 `ab` `bc` 时，右边就需要"通过 $ a \equiv b $ 和 $ b \equiv c $ 这两个条件，再对这两个条件套用一些变换，得出 $ a \equiv c $"。

在这个时候，编译器并没有把 $ a \equiv b $ 和 $ b \equiv c $ 当成既成条件，而是当成了 "变量" 。

这就回到了我们原本的需求，我们原本就是需要写出一个 $ a \equiv b\ \&\&\ b \equiv c \Rightarrow a \equiv c $ 的变换。

如果要变换的话，可以使用 `with` 语句（就是 Agda 的 `case of`）把这两个变量模式匹配出来，然后直接得证。
这里给出一个强行实现的方法。

$$
\begin{eqnarray}
\_{\leftrightarrows}_1\_\ ab\ bc \with ab\ &|&bc \\
... \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ |\ \ \refl\ &|&\refl = \refl
\end{eqnarray}
$$

```agda
_⇆₁_ ab bc with ab   | bc
...           | refl | refl = refl
```

这种方法和下面的做法是等价的。

如果你没有看懂这一坨，可以尝试继续读下去，说不定看完下面那坨你就懂了。

### 使用 refl

由于 $ a \equiv b $ 已经是一个条件了，我们直接把它的值取出来。
这时，右边的代码就 **已经是建立在 $ a \equiv b $ 和 $ b \equiv c $ 这两个既成条件下** 的了，因此这时 Agda 已经认为 `a` `b` `c` 三者相等了。

利用这一点，我们直接使用 `refl` 是没有问题的。

$$
\_{\leftrightarrows}_0\_\refl \refl = \refl
$$

```agda
_⇆₀_ refl refl = refl
```

## 顺带一提

当然我们也可以这样写，这是一个语法糖：

$$
\refl {\leftrightarrows}_0\refl = \refl
$$

```agda
refl ⇆₀ refl = refl
```

之前那个比较 trivial 的模式匹配也可以这样写：

$$
\begin{eqnarray}
ab\ {\leftrightarrows}_1\ ab \with ab\ &|&bc \\
... \ \ \ \ \ \ \ \ \ \ \ \ \ |\ \ \refl\ &|&\refl = \refl
\end{eqnarray}
$$

```agda
ab ⇆₁ bc with ab   | bc
...         | refl | refl = refl
```

### 结束

我说完了。