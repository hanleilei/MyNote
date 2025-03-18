## 题记

现在到了数学抽象中最关键的一步：让我们忘记这些符号所表示的对象。数学家不应在这里停止脚步，有许多操作可以应用这些符号，而根本不必考虑他们到底代表着什么东西。

-- hermann weyl，思维的数学方式

我们将构造整体的任务划分为了实现部件的任务。

首先是对于计算分数的加减乘除运算:

```scheme
(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))

(define (div-rat x y)
  (make-rat (* (numer x) (denom y))
            (* (denom x) (numer y))))

(define (sub-rat x y)
  (make-rat (- (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (equal-rat? x y)
  (= (* (numer x) (denom y))
     (* (numer y) (denom x))))
```



lisp中有一个pair的概念，两个元素组成：

`(cons x y)`

 表示构建一个pair，他的第一个元素是x，第二个元素是y

`(car p)`

选择pair p 的第一个元素

`(cdr p)`

选择 pair p的第二个元素

for any x and y:

`(car (cons x y))` is x
`(cdr (cons x y))` is y

下面是完整的代码：

```scheme
#lang sicp

(define (make-rat n d) (cons n d))

(define (numer x) (car x))
(define (denom x) (cdr x))

(define (add-rat x y)
  (make-rat (+ (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define (mul-rat x y)
  (make-rat (* (numer x) (numer y))
            (* (denom x) (denom y))))

(define (div-rat x y)
  (make-rat (* (numer x) (denom y))
            (* (denom x) (numer y))))

(define (sub-rat x y)
  (make-rat (- (* (numer x) (denom y))
               (* (numer y) (denom x)))
            (* (denom x) (denom y))))

(define x (make-rat 1 2))
(define y (make-rat 1 4))

(define ans (add-rat x y))
(numer ans)
(denom ans)
```

结果是 6 / 8

这时候，我们就需要做一些修改：

```scheme
(define (gcd a b)
  (if (= b 0)
      a
      (gcd b (remainder a b))))

(define (numer x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (car x) g)))

(define (denom x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (cdr x) g)))

```

完整代码：

```scheme
**第一段代码**在创建有理数时（`make-rat`）就进行约分，确保所有操作（加减乘除）都作用于已经约分的分数。
```

这里，我们可以看到scheme的语法 pair 和 add-rat/sub-rat/mul-rat/div-rat 之间，有一个抽象层，确切地说是, 构造函数和选择函数就是抽象层: make-rat/numer/denom

这一思想的优点：
1. 易于维护和修改。
2. 选择何时计算gcd，这样可以最小化修改。

上面的代码有点混乱，把所有的可能情况放在一起了，何时计算gcd呢？

```scheme
(define (make-rat n d) (cons n d))

(define (numer x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (car x) g)))

(define (denom x)
  (let ((g (gcd (car x) (cdr x))))
    (/ (cdr x) g)))
```

显然这里不是好的选择，哪次运行都要计算一下gcd。

或许可以修改成：

```scheme
(define (make-rat n d)
  (let ((g (gcd n d)))
    (cons (/ n g)
          (/ d g))))

(define (numer x) (car x))

(define (denom x) (cdr x))
```

这样，每次在创建有理数时（`make-rat`）就进行约分，确保所有操作（加减乘除）都作用于已经约分的分数。

什么是数据：

In general, we can think of data as defined by some collection of selectors and constructors, together with specified conditions that these procedures must fulfill in order to be a valid representation.

一般而言，我们总可以将**数据**定义为：一组适当的选择函数和构造函数，以及为使这些过程成为一套合法表示，他们就必须满足的一组特定条件。

data abstraction is sort of the programming methodology of setting up data objects by postulating constructors and selectors to isolate use from representation.

数据抽象是一种通过假定的构造函数和选择函数将数据对象与他的表示分隔开来的编程方法学。

问答环节：
1. 问：通过抽象层做决定，与编码前做完成设计的信条相比，哪个更好？
答：这只是不经常设计大型计算机系统的家伙的信条，计算机科学有时候很像是宗教，这就不好了。通常来说，我认为那些相信在编码前就能把系统设计完美，通常都是一些没有设计过大型系统的人。

我们的方法，厉害之处就在于可以假设我们已经得到结果了。然后再讨论到底是哪个是对的，或者你应该得到怎样的结果。当你学会了这招，你会发现这个是最棒的一招。

这段很牛逼，别做过早优化，你不可能设计的完美，做出来一个能用的，然后再想办法跑起来。

2. 问：let和define的差别。
答：
let 用于创建**局部变量**，其作用范围仅限于 let 表达式的内部。
define 通常用于创建**全局变量**或函数。在某些 Lisp 方言（如 Scheme）中，它也可以用在函数体内部来创建局部变量。

数据抽象作为在大型系统中控制复杂度的一个方法，就像过程定义中一样，以及我们谈论的所有的复杂度的控制方法中，这些东西的真正厉害之处不是体现在他们本身，而是怎么用他们构建大型的系统。

下面是一个二维平面的例子。(这个例子，似乎书中没有)

```scheme
; representing vectors in the plane

(define (make-vecotr x y) (cons x y)) ;; 构造函数
(define (xcor p) (car p)) ;; 选择函数
(define (ycor p) (cdr p))
```

```scheme
;; representing line segments

(define (make-seg p q) (cons p q))
(define (seg-start s) (car s))
(define (seg-end s) (cdr s))

```

当我们完成了这个系统的构建，就可以在上面加上一些操作，比方说：加上midpoint。

```scheme
(define (midpoint s)
  (let ((a (seg-start s))
		(b(seg-end s)))
	(make-vecotr
		(average (xcor a) (xcor b))
		(average (ycor a) (ycor b)))))
```

再加上一个求两个点之间的距离，完整的程序如下：

```scheme
#lang sicp

(define (make-vecotr x y) (cons x y)) ;; 构造函数
(define (xcor p) (car p)) ;; 选择函数
(define (ycor p) (cdr p))

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(define (make-seg p q) (cons p q))
(define (seg-start s) (car s))
(define (seg-end s) (cdr s))

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(define (average a b) (/ (+ a b) 2))

(define (midpoint s)
  (let ((a (seg-start s))
		(b(seg-end s)))
	(make-vector
		(average (xcor a) (xcor b))
		(average (ycor a) (ycor b)))))

(define (square x) ( * x x))

(define (length s)
  (let
      ((dx ( - (xcor (seg-end s))
               (xcor (seg-start s))))
       (dy ( - (ycor (seg-end s))
               (ycor (seg-start s)))))
   (sqrt ( + (square dx)
             (square dy)))))
```

然后，就可以看到这个层次：

```
Segments
-------------------------------------------------
make-segments / seg-start / seg-end
-------------------------------------------------
Vectors
-------------------------------------------------
make-vectors / xcor / ycor
-------------------------------------------------
pairs
```

segments(线段) is pair of points, points is a pair of numbers.

这里就是一个嵌套的数据结构：list of list。

---------------

视频在这里将这种数据结构叫做闭包（closure）： the means of combination in your system are such that when you put things together using them. （但是这个概念在现代的高级语言中，是基本概念：词法作用域）

----------------------------

可以看到在有了层次系统之后，如果不使用数据抽象，系统复杂度会有失控的隐患。因此，我们可以重写上面的length函数：

```scheme
(define (length s)
  (let
      ((dx ( - (car (car s))
               (car (cdr s))))
       (dy ( - (cdr (car s))
               (cdr (cdr s)))))
   (sqrt ( + (square dx)
             (square dy)))))
```

这将是一场灾难。选择就在这里，我们没有必要暴露全部的细节。我们对于这个系统的细节了解的很清楚，所以就可以控制他们。

-------------

Q:对于那些无法使用pairs来表示的情况中，会如何呢？例如，一个pair不能表示三维空间的坐标。
A: 一旦使用了pair，就可以任意扩展，比方说，如果是三维的数据，可以用一个pair，其中的第一个元素是1，另一个元素是一个pair。

----

axiom: 公理

```scheme
#lang sicp

(define (cons a b)
  (lambda (pick)
    (cond (( = pick 1) a)
          (( = pick 2) b))))

(define (car x) (x 1))
(define (cdr x) (x 2))

(define x (cons 3 4))
(car x)
(cdr x)
```

这段视频内容有点没看懂，像是介绍message passing，有点抽象，视频是从1:13:00开始。提问的人水准都是很高，

每次调用cons，就会把相应的数值传递过去。





























