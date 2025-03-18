心智的活动，除了尽力产生各种简单的认识之外，主要表现在如下三个方面：
1. 将若干筒单认识组合为一个复合认识，由此产生出各种复杂的认识。
2. 将两个认识放在一起对照，不管它们如何简单或者复杂，在这样做时并不将他们合而为ー。由此得到有关它们的相互关系的认识。
3. 将有关认识与那些在实际中和它们同在的所有其他认识隔离开，这就是抽象，所有具有普遍性的认识都是这样淂到的。 
-- John Locke，（有关人类理解的随笔，1690）


写一个从 a 到 b 的求和：

```scheme
(define (sum-int a b)
  (if (> a b)
      0
      (+ a 
	      (sum-int (inc a) b))))

(sum-int 4 6)
```

再来一个从 a 平方到 b 平方的求和：

```scheme
(define (square a) ( * a a))

(define (sum-sq a b)
  (if ( > a b)
      0
      (+ (square a)
         (sum-sq (inc a) b))))

(sum-sq 3 5)
```

```scheme
(define (pi-sum a b)
  (if (> a b)
      0
      (+ (/ 1 (* a (+ a 2)))
         (pi-sum (+ a 4) b))))
```



所以，上面的代码可以用一个通用模式（general pattern）来表达：

```scheme
(define (<name> a b)
  (define (iter j ans)
	  (if (> a b)
		  0
		  (+ (<term> a)
			  (<name> (<next> a) b))))
	(iter a 0))
```

a: lower bound
b: upper bound

通过 `<next>` 来对于lower bound进行更新，直到upper bound 和 lower bound相等。

数字可以作为参数，这个我们已经看到了，此外，函数也可以作为参数。

```scheme

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term
              (next a)
              next
              b))))

(define (sum-int a b)
  (define (identity x) x)
  (sum identity a inc b))
```

可以，通过这样的方式调用，传递函数作为参数。

再来sum-sq：

```scheme
#lang planet neil/sicp

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term
              (next a)
              next
              b))))

(define (square a) ( * a a))

(define (sum-sq a b)
  (sum square a inc b))

```

收起emo和你那点小情绪，继续加油吧。

视频里面的代码是这样：

```scheme
(define (pi-sum a b)
  (sum (λ (i) (/ 1 (* i (+ i 2))))
       a
       (λ (i) (+ i 4))
         b))
```

但是这个是显然不对的，毕竟视频来自于近四十年前，现在的sicp环境下，只能用 lambda 代替 λ。

```scheme
#lang planet neil/sicp

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term
              (next a)
              next
              b))))

(define (square a) ( * a a))

(define (pi-sum a b)
  (sum (lambda (i) (/ 1 (* i (+ i 2)))) 
       a 
       (lambda (i) (+ i 4)) 
       b))

(pi-sum 3 4)
```

节外生枝一下，这个lambda的问题很有趣：

```scheme
#lang racket
(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) next b))))
         
(define (pi-sum a b)
  (sum (λ (i) (/ 1 (* i (+ i 2))))
       a
       (λ (i) (+ i 4))
       b))

(pi-sum 3 4)
```

上面的代码，在racket中就可以识别 λ，区别就是第一行 `#lang racket`, 再来一个：

```scheme
(import (chezscheme)) ; Ensure compatibility with Chez's module system

(define-syntax λ
  (syntax-rules ()
    ((_ . args) (lambda . args))))  ; Map λ to lambda syntax

(define (sum term a next b)
  (if (> a b)
      0
      (+ (term a)
         (sum term (next a) next b))))

(define (pi-sum a b)
  (sum (λ (i) (/ 1 (* i (+ i 2))))
       a
       (λ (i) (+ i 4))
       b))

(pi-sum 3 4)

```

这个代码就可以通过chez的编译。

Chez Scheme does not natively support `λ` as an alias for `lambda`. However, you can use a syntax rule to create a custom shorthand:

和上一节的内容类似，讲完了递归，下面是迭代：

```scheme
#lang planet neil/sicp

(define (sum term a next b) 
  (define (iter j ans)
    (if (> j b)
        ans
        (iter (next j)
              (+ (term j) ans))))
  (iter a 0))

(define (square a) (* a a))

(define (sum-sq a b)
  (define (increase x) (+ x 1))
  (sum square a increase b)) 

(sum-sq 3 4)

```

-----

介绍Heron法，用于计算平方根，可以得到类似这样的代码：

```scheme
#lang racket

(define (heron-sqrt S #:tolerance [tolerance 1e-10] #:max-iterations [max-iterations 1000])
  (define (iter x count)
    (if (or (>= count max-iterations) (< (abs (- (/ (+ x (/ S x)) 2) x)) tolerance))
        (/ (+ x (/ S x)) 2)  ; Final approximation
        (iter (/ (+ x (/ S x)) 2) (add1 count))))  ; Recursive step
  (iter (/ S 2) 0))  ; Initial guess is S / 2

(heron-sqrt 16)


#lang racket

(define (heron-sqrt S #:tolerance [tolerance 1e-10] #:max-iterations [max-iterations 1000])
  (define (loop x iteration)
    (let ((next-x (/ (+ x (/ S x)) 2.0)))
      (cond
        [(< (abs (- next-x x)) tolerance) next-x] ; 如果误差小于容差,返回结果
        [(>= iteration max-iterations) (error "Failed to converge")] ; 超过最大迭代次数
        [else (loop next-x (+ iteration 1))]))) ; 递归调用
  (loop (/ S 2.0) 0)) ; 初始猜测值为 S / 2.0,开始递归
```

或者来自于视频的：

```scheme
#lang planet neil/sicp


(define (sqrt x)
  (define (average x y) (/ (+ x y) 2))
  (define tolerance 0.00000001)
  (define (good-eouf? y)
    (< (abs ( - (* y y) x)) tolerance))
  (define (improve y)
    (average (/ x y) y))
  (define (try y)
    (if (good-eouf? y)
        y
        (try (improve y))))
  (try 1))

(sqrt 4)
```

其中，类似 `#:tolerance` 的是关键字参数。属于racket后续添加的现代功能。

下面分别用 lexical scoping vs Block-structured

```scheme
;; Block-structured
(define (sqrt x)
  (define (square x) (* x x))
  (define (average x y) (/ (+ x y) 2))
  (define (good-enough? guess x)
    (< (abs (- (square guess) x)) 0.001))
  (define (improve guess x)
    (average guess (/ x guess)))
  (define (sqrt-iter guess x)
    (if (good-enough? guess x)
        guess
        (sqrt-iter (improve guess x) x)))
  (sqrt-iter 1.0 x))

;; Taking advantage of lexical scoping
(define (sqrt x)
  (define (square x) (* x x))
  (define (average x y) (/ (+ x y) 2))
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001)) ;; 修改0.001，到下面的0.0000000001, 还是不能收敛，也就是说，比下面的代码收敛的慢很多。
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  (sqrt-iter 1.0))

;; 上面的代码，总是不能快速收敛。。
(define (sqrt x)
  (define (square x) (* x x))
  (define (average x y) (/ (+ x y) 2))
  (define (good-enough? old-guess new-guess)
    (< (abs (- old-guess new-guess)) 0.0000000001))
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter old-guess guess)
    (if (good-enough? old-guess guess)
        guess
        (sqrt-iter guess (improve guess))))
  (sqrt-iter 0 1.0)) 

(sqrt 4.0)

```

差别就在于传递的 x 参数，lexical少传递了很多个。

另一个很有趣的点，就是收敛的速度，明显，如果是用 平方后比较，收敛的反而慢。这个是什么原因呢？这是一个比较经典的优化方法，避免直接比较平方值。

----

这时候，这个fix point，就是一个值，将其应用到函数中得到的还是原来的值。  

比方说 e^x 的导数就可以表示为 e^x， 如何一个数字一直求cosin，得到的值最终为0.73908513321.. 

some function have the property that you can find their fixed point by iterating the function.

这就是在使用heron法中，实现的过程：

```scheme
#lang planet neil/sicp
;;#lang racket
(define tolerance 0.00000000001)
(define (average x y) (/ (+ x y) 2))

(define (fixed-point f first-guess)
  (define (close-enough? old new)
    (< (abs (- old new)) tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))

(define (sqrt x)
  (fixed-point (lambda (y) (average y (/ x y)))
               1.0))

(sqrt 5)
```

或者：

```scheme
#lang planet neil/sicp

(define (close-enough? x y) (< (abs (- x y)) 0.0000000000001))

(define (average x y) (/ (+ x y) 2))

(define (fix-point f start)
  (define (iter old new)
    (if (close-enough? old new)
        new
        (iter new (f new))))
  (iter start (f start)))

(define (sqrt x)
  (fix-point (lambda (y) (average y (/ x y)))
               1.0))

(sqrt 4.0)
```

两个代码，在drracket的环境下，后者可以收敛。比方说，求sqrt 4，后者可以收敛到2.0，前面的代码不可以。

为什么这里迭代计算 `(y + x / y) / 2` 的值就是正确的结果？可能这里表现的暂时不是那么的明显。

需要做的选择：让计算结果快速收敛，而不是振荡，或者产生谐振。

```scheme
#lang planet neil/sicp

(define (sqrt x)
  (fixed-point
   (average-damp (lambda (y) (/ x y)))
   1))
   
```

他的参数是一个过程，返回值也是一个过程。具体来说，给定一个过程，他产生出另一个过程，

在这里，完整的代码：

```scheme
#lang planet neil/sicp

(define (average x y) (/ (+ x y) 2))

;; (define (average-damp f)
;;   (lambda (x) (average x (f x))))

;; 注意这里的两种不同的写法，这个是视频中的，上面是书中的。
(define average-damp
  (lambda (f) 
	 (lambda (x) (average x (f x))))) 

(define tolerance 0.00000001)


(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))

(define (sqrt x)
  (fixed-point
   (average-damp (lambda (y) (/ x y)))
   1))
   
(sqrt 4.0)
```

这里应该是柯里化。

高阶函数（high order procedure),就是接收 procedure 作为参数和产生 procedure 作为结果，用来帮助我们阐明和抽象一些复杂的过程。

to find a `y` such that `f(y) = 0` , start with a guess: y0

$$
y_{n+1} = y_n - \frac{f(y_n)}{\frac{df}{dx}|_{y=yn}}
$$ 

组合一下书中的代码：

```scheme
#lang planet neil/sicp


(define (square x) (* x x))

(define tolerance 0.00000001)

(define (deriv g)
  (lambda (x)
    (/ (- (g (+ x dx)) (g x))
       dx)))
(define dx 0.00001)

(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))

(define (newton-transform g)
  (lambda (x)
    (- x (/ (g x) ((deriv g) x)))))

(define (newtons-method g guess)
  (fixed-point (newton-transform g) guess))

(define (sqrt x)
  (newtons-method (lambda (y) (- (square y) x))
                  1.0))
   
(sqrt 4.0)
```

下面是视频中的代码：

```scheme
#lang planet neil/sicp


(define (square x) (* x x))

(define tolerance 0.00000001)

(define (fixed-point f first-guess)
  (define (close-enough? v1 v2)
    (< (abs (- v1 v2)) tolerance))
  (define (try guess)
    (let ((next (f guess)))
      (if (close-enough? guess next)
          next
          (try next))))
  (try first-guess))

(define deriv
  (lambda (f)
    (lambda (x)
            (/ (- (f (+ x dx))
                  (f x))
               dx))))

(define dx 0.00001)

(define (newtons f guess)
  (define df ( deriv f))
  (fixed-point (lambda (x) (- x (/ (f x) (df x))))
               guess))

(define (sqrt x)
  (newtons (lambda (y) (- (square y) x))
                  1.0))
   
(sqrt 4.0)

```

这里的 fixed-point 函数还是复用之前的函数。

函数作为第一公民：

```text
the rights and priviledges of first-class citizens

to be named by variables
to be passed as arguments to procedures
to be returned as values of procedures
to be incopoorated into data structures
```


补充一段代码：

```scheme
#lang sicp

(define (average x y)
  (/ (+ x y) 2))

(define (search f neg-point pos-point)
  (let ((midpoint (average neg-point pos-point)))
    (if (close-enough? neg-point pos-point)
        midpoint
        (let ((test-value (f midpoint)))
          (cond ((positive? test-value)
                 (search f neg-point midpoint))
                ((negative? test-value)
                 (search f midpoint pos-point))
                (else midpoint))))))

(define (close-enough? x y)
  (< (abs (- x y)) 0.0000001))

(define (half-interval-method f a b)
  (let ((a-value (f a))
        (b-value (f b)))
    (cond ((and (negative? a-value) (positive? b-value))
           (search f a b))
          ((and (negative? b-value) (positive? a-value))
           (search f b a))
          (else
           (error "Values are not of opposite sign" a b)))))


(half-interval-method sin 2.0 4.0)

(half-interval-method (lambda (x) (- (* x x x) (* 2 x) 3))
                      1.0
                      2.0)
```

区间折半法球方程`f(x) = 0`的根。这段代码是书上有介绍，但是视频里没有。
















































