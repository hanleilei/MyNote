We are about to study the idea of a _computational process_. Computational processes are abstract beings that inhabit computers. As they evolve, processes manipulate other abstract things called _data_. The evolution of a process is directed by a pattern of rules called a _program_.

一个重要的概念就是：**控制系统复杂度**

## lisp

我们使用drracket，如果遇到问题可以在最开头加上：

```scheme
#lang planet neil/sicp
```

就会自动安装，或者

```scheme
#lang sicp
```

具体的需要在DrRacket上安装一个包，具体方法可以看这个汇总的[页面](https://github.com/DeathKing/Learning-SICP?tab=readme-ov-file)

## 黑盒（blackbox）

- primitive objects
  - primitive procedures
  - primitive data
- means of combination
  - procedure composition
  - construction of compound data
- means of abstraction
  - procedure definition
  - simple data abstraction
- capturing common patterns
  - high order procedures
  - data as procedures

## 约定接口（conventional interface）

- generic operations
- large-scale structure and modularity
- object oriented programming
- operations on aggregates(stream)

## 元语言抽象（metalinguistic abstraction)

当学习一个新语言的时候，需要问自己：

- 什么是 Primitive element?
- 什么是 means of combination
- 什么是 means of abstraction 

```scheme
(+ 3 17.4 5) 
```

其中，加号是Operator，另外三个数字，是operand，这就是combination。lisp用的是 prefix notation

```scheme
(define square (lambda (x) (* x x)))

(define (average x y)
  (/ ( + x y) 2))

(define (mean-square x y)
  (average (square x)
           (square y)))


(mean-square 2 3)
```

这里，我们没有办法区别那些是语言中的基本对象(primitive)，那些是内建（buildin）, 所以你构建的东西，看起来就像是语言自带的基本对象，该对象具有同样的能力和灵活性。这也是lisp的关键特性之一。

所以，很多情况下，buildin 和 compound 之间是很难区分的。 

## 牛顿法求平方根

找到sqrt(x)的近似值：

1. make a guess G
2. improve the guess by averaging g and x / g
3. keep improving the guess until it is good enough
4. use 1 as an initial guess

```scheme
(define (square x) (* x x))

(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x)
                 x)))

(define (improve guess x)
  (average guess (/ x guess)))

(define (average x y)
  (/ (+ x y) 2))

(define (good-enough? guess x)
  (< (abs (- (square guess) x)) 0.001))

(define (sqrt x)
  (sqrt-iter 1.0 x))


(sqrt 9)
(sqrt (+ 100 37))
(sqrt (+ (sqrt 2) (sqrt 3)))
(square (sqrt 1000))
```

这个是尾递归，堆栈不会增加。

问题是，上面的代码，根本没办法收敛，比方说是4的开方，得到的是2.00012312332之类的结果。

二分法实现同样的功能：

```scheme
(define (square x)
  (* x x))

(define (mid low high)
  (/ (+ low high) 2))

(define (my-sqrt-iter low high x)
  (let ((mid (mid low high)))
    (if (> (square mid) x)
        (my-sqrt-iter low (- mid 1) x)
        (if (<= low high)
            (my-sqrt-iter (+ mid 1) high x)
            mid))))

(define (my-sqrt x)
  (my-sqrt-iter 0 x x))
```

```python
class Solution:
    def mySqrt(self, x: int) -> int:
        low, high, mid = 0, x, x // 2
        while low <= high:
            if mid * mid > x:
                high = mid - 1
            else:
                low = mid + 1
            mid = (low + high) // 2
        return mid
```

这也是leetcode的69题:sqrt(x)，有点扯远了。

所以，整个过程可以用一个盒子表示：

```text
         |--------------------------------------|
         |    sqrt                              |
   36    |                   sqrt-iter          |
-------> |                   good-enough?       | ------> 6
         |                   imporve            |
         |--------------------------------------|
```

好吧，重构一下这个黑盒子：

```scheme
(define (sqrt x)
  (define (square x)
    (* x x))
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))

  (define (improve guess)
    (average guess (/ x guess)))

  (define (average x y)
    (/ (+ x y) 2))

  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))

  (sqrt-iter 1.0))
```

看看我们当前掌握了什么：

| item                 | procedures                       | data          |
| -------------------- | -------------------------------- | ------------- |
| primitive elements   | + - * /                          | 23 <br>11.738 |
| means of combination | () composition<br><br>cond<br>if |               |
| means of abstraction | define                           |               |

提问：

```scheme
(define (D) (* 4 4))
(define A (+ 4 4))
```

这里的A和D有什么差别？

在DrRacket运行环境中：

```scheme
> (define (D) (* 4 4))
> D
#<procedure:D>
> (D)
16
> A
. . A: undefined;
 cannot reference an identifier before its definition
```

所以可以看到，D 是一个procedure，没有参数的那种
