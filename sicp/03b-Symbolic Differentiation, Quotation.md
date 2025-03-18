
符号求导数：

```scheme
#lang sicp

(define (deriv exp var)
  (cond ((number? exp) 0)
;;        ((constant? exp var) 0)
        ((variable? exp)
         (if (same-variable? exp var) 1 0))
        ((sum? exp)
         (make-sum (deriv (addend exp) var)
                   (deriv (augend exp) var)))
        ((product? exp)
         (make-sum
           (make-product (multiplier exp)
                         (deriv (multiplicand exp) var))
           (make-product (deriv (multiplier exp) var)
                         (multiplicand exp))))
        (else
         (error "unknown expression type -- DERIV" exp))))

;; representing algebraic expressions

(define (atom? x)
  (and (not (pair? x)) (not (null? x))))

(define (constant exp var)
  (and (atom? exp)
       (not (eq? exp var))))

(define (variable? x) (symbol? x))

(define (same-variable? v1 v2)
  (and (variable? v1) (variable? v2) (eq? v1 v2)))

;; 视频这里很有趣。
(define (same-var? exp var)
  (and (atom? exp)
       (eq? exp var)))

(define (make-sum a1 a2) (list '+ a1 a2))

(define (make-product m1 m2) (list '* m1 m2))

;; 视频这里总是用atom
(define (sum-video? exp)
  (and (not (atom? exp))
       (eq? (car exp) 'exp)))

(define (sum? x)
  (and (pair? x) (eq? (car x) '+)))

(define (addend s) (cadr s))

(define (augend s) (caddr s))

;; 视频这里总是用atom
(define (product-video? exp)
  (and (not (atom? exp)
            (eq? (car exp) '*))))

 
(define m1 cadr)
(define m2 caddr)

;;;;;;;;;;;; --------------

(define (product? x)
  (and (pair? x) (eq? (car x) '*)))

(define (multiplier p) (cadr p))

(define (multiplicand p) (caddr p))


;;; 开始计算

(deriv '(+ x 3) 'x)

(deriv '(* x y) 'x)
(deriv '(* (* x y) (+ x 3)) 'x)

(deriv '(+ (* a ( * x x)) (* b x) c) 'x)
```

得到的结果：

```scheme
(+ 1 0)
(+ (* x 0) (* 1 y))
(+ (* (* x y) (+ 1 0)) (* (+ (* x 0) (* 1 y)) (+ x 3)))
(+ (+ (* a (+ (* x 1) (* 1 x))) (* 0 (* x x))) (+ (* b 1) (* 0 x)))
```

化简一下，可以把这些函数替换：

```scheme
(define (make-sum a1 a2)
  (cond ((=number? a1 0) a2)
        ((=number? a2 0) a1)
        ((and (number? a1) (number? a2)) (+ a1 a2))
        (else (list '+ a1 a2))))

(define (=number? exp num)
  (and (number? exp) (= exp num)))

(define (make-product m1 m2)
  (cond ((or (=number? m1 0) (=number? m2 0)) 0)
        ((=number? m1 1) m2)
        ((=number? m2 1) m1)
        ((and (number? m1) (number? m2)) (* m1 m2))
        (else (list '* m1 m2))))

```

这时候结果为：

```scheme
1
y
(+ (* x y) (* y (+ x 3)))
(+ (* a (+ x x)) b)
```


所以，这样我们就可以得到一个三层的结构：

```text
  derivative rules
--------------------------------------------
  constant?       sum?      a1
  same-variable?  make-sum  a2 ...
--------------------------------------------
  representation of algebraic expressions

```













































