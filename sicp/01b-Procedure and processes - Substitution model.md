## 题记

心智的活动，出了尽力产生各种简单的认识之外，主要表现在如下的三个方面：

1. 将若干简单认识组合为一个复合认识，由此产生出各种复杂的认识。
2. 将两个认识放在一起对照，不管他们如何简单或者复杂，在这样做时，并不将他们合而为一。由此得到有关他们相互关系的认识
3. 将有关的知识与那些在实际中和他们同在的所有其他认识隔离开，这就是抽象，所有具有普遍性的认识，都是这样得到的。

-- John Locke 有关人类理解的随笔，1690

kind of expression

先看一段平方和的代码：

```scheme
(define (square x) (* x x))
;; (define square (lambda (x) (* x x)))

;; (square 21)
;; (square (+ 2 5))
;; (square (square 3))

(define (sum-of-squares x y)
  (+ (square x) (square y)))

(sum-of-squares 3 4)

(define (f a)
  (sum-of-squares (+ a 1) (* a 2)))

(f 5)
```

此时，我们可以知道有这些表达式：

- numbers
- Symbols
- lambda expression(talk later)
- definitions(talk later)
- conditional(talk later)
- combination

# substitution rule

to evaluate an application

- evaluate the operator to get procedure
- evaluate the operands to get arguments
- apply the procedure to the arguments
	- copy the body of the procedure
	- substituting the arguments supplied for the formal parameters of the procedure
	- evaluate the resulting new body

所以，对于上面的代码：

```scheme
(define (square x) (* x x))

(define (sum-of-squares x y)
  (+ (square x) (square y)))

(sum-of-squares 3 4)
```

我们逐步实现：

`(sum-of-squares 3 4)` -> 
`(+ (square 3) (square 4))` -> 
`(+ (square 3) (* 4 4))`

还可以继续计算，但是如果探究计算机的抽象层次，你会发现无论在具体的哪一个层次，在其下都还有若干个你不清楚的抽象层次, 但我们必须明白要学会忽略细节。

at any level of detail, if you look inside this machine, you are going to find that  there are multiple levels below that you don'
t know about. but one of the things we have to learn how to do is ignore details.


**理解复杂事物的关键是避免不必要的观察，计算和思考。**

**the key to understanding complicated things is to know what not to look at and what not to compute and what not to think.**

所以，我们对于乘法过程的细节不做深入研究。

so we are going to stop this one here.
 
继续：

`(sum-of-squares 3 4)` -> 
`(+ (square 3) (square 4))` -> 
`(+ (square 3) (* 4 4))` -> 
`(+ (square 3) 16)` -> 
`(+ (* 3 3) 16)` -> 
`(+ 9 16)` -> 
`25`

这里看到了应用代换(substitutions)模型的基本方法


# conditional rule

to evaluate an IF expression

- evaluate the predicate expression
	- if it yields True
		- evaluate the consequent expression
	- otherwise
		- evaluate the alternative expression

所以，可以来一个对于 a和b求和的例子：

```scheme
(define (+ a b)
  (if (= a 0)
	  b
	  (+ (- a 1) (+ b 1))))
```

这里的inc 和dec 分别是增加一和减少一。

`(+ 3 4)` -> 
`(if (= 3 0) 4(+ (- 3 1) (+ 1 4)))` -> 
`(+ (- 3 1) (+ 1 4))` -> 
`( + 2 5)` —>
`(if (= 2 0) 5 ( + (- 2 1) (+ 1 5)))` -> 
...

最终得到结果7

## Peano Arithmetic

2 ways to add whole number:

TODO: 这里有问题，和后面的 recursive 和 iteration 有点搞混了，需要返工。

应用序（Applicative order）：
```scheme
(define (+ a b)
  (if (= a 0)
      b
      (inc (+ (dec a) b))))
```


```scheme
(+ 3 4)
(+ 2 5)
(+ 1 6)
(+ 0 7)
7
```

正则序（normal order）：

```scheme
(define (+ a b)
  (if (= a 0)
      b
      (+ (dec a) (inc b))))
```


```scheme
(+ 3 4)
(inc (inc 2 4))
(inc (inc (inc 1 4)))
(inc (inc (inc (inc 0 4))))
(inc (inc (inc 4)))
(inc (inc 5)
(inc 6)
7
```

注意，这里的 applicable order 和 normal order， 和我们之前看到的递归 和 尾递归很类似，但是他们是不一样的.

再来一个就是计算fib：

```scheme
(define (fib n)
  (if (< n 2)
      n
      (+ (fib (- n 1))
         (fib (- n 2)))))
(fib 10)
```

或者：

```scheme
;; Recursive

(define (fib n)
  (cond ((= n 0) 0)
        ((= n 1) 1)
        (else (+ (fib (- n 1))
                 (fib (- n 2))))))

;; Iterative

(define (fib n)
  (fib-iter 1 0 n))

(define (fib-iter a b count)
  (if (= count 0)
      b
      (fib-iter (+ a b) a (- count 1))))
```

不管用哪种写法，都会导致有重复的计算，比方说 `fib(5) = fib(4) + fib(3)`, 但是`fib(4) = fib(2) + fib(3)`, 所以，`fib(3)`在这里被重复计算了两次，`fib(2)`被重复计算了三次。
![](https://mitp-content-server.mit.edu/books/content/sectbyfn/books_pres_0/6515/sicp.zip/full-text/book/ch1-Z-G-13.gif)


复杂度指数型增长，有大量的重复计算。

 后面的 hanoi 的例子，暂时略过，后续补充。



























