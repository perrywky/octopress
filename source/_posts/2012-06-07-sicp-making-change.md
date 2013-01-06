--- 
categories: 
  - algorithm
  - sicp
comments: true
layout: post
title: SICP换零钱问题的线性迭代解法
---
SICP的1.2.2节里提到了一个换零钱的问题
> 给了半美元、四分之一美元、10美分、5美分和1美分的硬币，将1美元换成零钱，一共有多少种不同方式？

书里给了一个树形递归的解法，思路非常简单，把所有的换法分成两类，包含50美分的和不包含的。包含50美分的换法里，因为它至少包含一张50美分，所以它的换法就相当于用5种硬币兑换剩下的50美分的换法；不包含50美分的，只能用4种硬币兑换1美元。这样用5种硬币兑换1美元就等价于用5种硬币兑换50美分的换法加上用前4种硬币兑换1美元的换法。以次类推，用4种硬币兑换1美元的换法就等价于用4种硬币兑换75美分的换法加上用3种硬币兑换1美元的换法。

<!--more-->

假设用1种硬币求换法数量的函数是f(n)，用2种的是g(n)，3种的是h(n)，4种的是i(n)，5种的是j(n)，那么
    j(100) = j(50) + i(100)
    j(50) = j(0) + i(50)
    j(0) = 1 #有1种兑法兑换0元，那就是一个硬币都没有

    i(100) = i(75) + h(100)
    i(75) = i(50) + h(75)
    i(50) = i(25) + h(50)
    i(25) = i(0) + h(25)
    i(0) = 1
程序如下
``` scheme 
(define (count-change amount)
  (cc amount 5))

(define (cc amount kinds-of-coins)
  (cond ((= amount 0) 1)
        ((or (< amount 0) (= kinds-of-coins 0)) 0)
        (else (+ (cc amount
                     (- kinds-of-coins 1))
                 (cc (- amount
                        (first-denomination kinds-of-coins))
                     kinds-of-coins)))))

(define (first-denomination kinds-of-coins)
  (cond ((= kinds-of-coins 1) 1)
        ((= kinds-of-coins 2) 5)
        ((= kinds-of-coins 3) 10)
        ((= kinds-of-coins 4) 25)
        ((= kinds-of-coins 5) 50)))

(count-change 100)
```
这个算法非常的简单，但是它的效率很低，有大量的重复计算，比如i(50)，它的时间复杂度是指数级的，在我的电脑上(2.2GHz i7)**计算500块就需要15秒了**，根本不实用。书中给读者留了一个挑战，找出线性迭代的解法。这是个难的问题，我在[Stackoverflow](http://stackoverflow.com/questions/1485022/sicp-making-change/)上找到了一点思路，用动态规划的方法从0到100“推”出结果。这个算法的核心思想跟之前的递归其实是一样的，只不过是反过来推，先算出f(1)到f(100)（都是1），将所有的结果保存到一个数组里，再算g(1)到g(100)，保存到另一个数组里，因为计算g(n)所需要的数据g(n-5)和f(n)都已经准备好了，这样就可以避免重复的计算。接着再算h(1)到h(100)，i(1)到i(100)，最后是j(1)到j(100)。程序如下
``` scheme
(define coins (list 1 5 10 25 50))
(define (current-coin coins)
    (car coins))
(define (rest-coins coins)
    (cdr coins))
(define (empty-coin? coins)
    (= (length coins) 0))

(define current-counts (list 1))
(define prev-counts '())
(define (add-count counts new-count)
    (append counts (list new-count)))
(define (get-count counts amount)
    (cond ((< amount 0) 0)
          ((>= amount (length counts)) 0)
          (else (list-ref counts amount))))

(define (cc total coins amount current prev)
    (cond ((empty-coin? coins) (get-count prev total))
          ((<= amount total)
                (let ((last-count (get-count current (- amount (car coins))))
                      (prev-count (get-count prev amount)))
                     (cc total coins (+ amount 1) (add-count current (+ last-count prev-count)) prev)))
          (else (cc total (rest-coins coins) 1 (list 1) current))))

(cc 100 coins 1 current-counts prev-counts)
```
这种解法只需要循环5遍就可以得到结果，所以它的时间复杂度是O(n)，比书中的例子快多了。但是因为它至少需要保存两个长度为n的数组，所以它的空间复杂度也是O(n)，还不能算是线性迭代的解法，因为它要求空间复杂度是O(1)。
我们再仔细观察下这个动态规划的过程，可以发现，当我们从1推到100的过程中，有很多值是没必要存储的。
    j(100)=j(50)+i(100)
    j(50)=j(0)+i(50)
    j(0)=1

    i(100)=i(75)+h(100)
    i(75)=i(50)+h(75)
    i(50)=i(25)+h(50)
    i(25)=i(0)+h(25)
    i(0)=1
对于j(100)，我们只需要存储j(50)，j(25)和j(0)
对于i(100)，我们只需要存储i(75)，i(50)，i(25)和i(0)
对于h(100)，我们只需要存储h(90)，h(80)，h(70)，h(60)，…，h(10)，h(0)，但是为了辅助i(75)，我们还需要多存h(75)，h(65)，…，h(5)
对于g(100)，我们只需要存储g(95)，g(90)直到g(5)，g(0)

如果我们改变下循环的次序，先计算f(0)到j(0)，再计算f(5)到j(5)，接着是f(10)到j(10)，最后f(100)到j(100)，这样就可以节省很多不必要的空间。当我们计算j(100)时，j(25)对我们来说已经没有意义了，我们只需要知道j(50)就够了，计算i(100)时，只用i(75)也就够了，所以对于每一个函数，都只用保存一个值。但一个值其实是不够的，我们可以从h(0)推到h(10)再推到h(100)，但是没法从h(0)推到h(75)，只能从h(5)开始推起，所以对于函数h，我们需要保存两个值，一个用于推出h(100)，一个用于推出h(75)。为什么i不需要保存两个值呢，因为50是25的整数倍，所以当我们推导i的时候，会自动包含h所需要的值，而25并不是10的整数倍，所以需要为其单独保存一个值。这里不管是哪个函数我们都是从0开始推的，因为0是100除以所有硬币的余数，当我们要算99块钱的兑法时，就不能从0开始了，对于j，我们需要知道j(49)，对于i，我们则需要知道i(24)，而对于h，则需要h(9)和h(4)了，h(4)怎么算出来的呢，(99-25)%10。所以对于每一个函数，我们只需要保存最多两个值就够了，一个推出f(n)，一个推出f(n-V)，这里的V是下一种硬币的面值。

我用两个长度为硬币种数的数组来保存计算结果，一个是用来保存f(n)到j(n)的counts，一个是用来保存f(n-Vg)到i(n-Vj)的counts-alt。
``` scheme
(define coins (list 1 5 10 25 50))
(define (coin index)
    (list-ref coins index))

(define (append-count count coinIndex value)
    (define (ac head tail index)
        (cond ((< index coinIndex)
               (cond ((= (length tail) 0)
                           (ac (append head (list 0)) (list 0) (+ index 1)))
                    ((= (length tail) 1)
                         (ac (append head (list (car tail))) (list 0) (+ index 1)))
                    ((> (length tail) 1)
                   (ac (append head (list (car tail))) (cdr tail) (+ index 1)))))
              ((= index coinIndex)
               (if (= (length tail) 0)
                   (append head (list value))
                   (append head (list (+ (car tail) value)) (cdr tail))))))
    (ac '() count 0))

(define (cal-count index amount coinIndex counts counts-alt)
    (if (= (remainder index (coin coinIndex)) (remainder amount (coin coinIndex)))
        (if (= coinIndex 0)
            (if (= index 0)
                 (append-count counts coinIndex 1)
                 counts)
            (cond ((= (remainder index (coin (- coinIndex 1))) (remainder amount (coin (- coinIndex 1))))
                    (append-count counts coinIndex (list-ref counts (- coinIndex 1))))
                  ((= (remainder index (coin (- coinIndex 1))) (remainder (- amount (coin coinIndex)) (coin (- coinIndex 1))))
                    (append-count counts coinIndex (list-ref counts-alt (- coinIndex 1))))))
        counts))

(define (cal-count-alt index amount coinIndex counts counts-alt)
    (if (and (< coinIndex (- (length coins) 1)) (= (remainder index (coin coinIndex)) (remainder (- amount (coin (+ coinIndex 1))) (coin coinIndex))))
        (if (= coinIndex 0)
            (if (= index 0)
                 (append-count counts-alt coinIndex 1)
                 counts-alt)
            (cond ((= (remainder index (coin (- coinIndex 1))) (remainder amount (coin (- coinIndex 1))))
                    (append-count counts-alt coinIndex (list-ref counts (- coinIndex 1))))
                  ((= (remainder index (coin (- coinIndex 1))) (remainder (- amount (coin coinIndex)) (coin (- coinIndex 1))))
                    (append-count counts-alt coinIndex (list-ref counts-alt (- coinIndex 1))))))
        counts-alt))

(define (cc index amount coinIndex counts counts-alt)
    (if (< coinIndex (length coins))
        (cc index amount (+ coinIndex 1) (cal-count index amount coinIndex counts counts-alt) (cal-count-alt index amount coinIndex counts counts-alt))
        (if (= index amount)
            (list-ref counts (- coinIndex 1))
            (cc (+ index 1) amount 0 counts counts-alt))))

(cc 0 100 0 '() '())
```
cc是主函数，有两层循环，外层循环从0到100，内层循环从1美分到50美分，对于每一个index，都计算一次包含当前coinIndex的情况下，有多少种换法，由counts或counts-alt里已计算好的值来得出。

append-count是一个辅助函数，用来更新数组，当index超出数组长度时，自动补零。

cal-count计算f(n)到j(n)的counts，cal-count-alt计算f(n-Vg)到i(n-Vj)的counts-alt。这两个函数的逻辑基本上一样的，只是计算新值的条件不同。在index从0涨到100的过程中，假设coinIndex为3，即10美分，只有当index除以10的余数和100除以10的余数相同时，才会计算新的counts，否则不用计算，因为h(20)=h(10)+g(20)，h(11)到h(19)对我们来说都没意义。每次计算counts里某个coinIndex的值时，都是由现有的值加上一个counts或counts-alt里coinIndex-1的值，是counts还是counts-alt取决于余数的状态。计算counts-alt时，则是考虑当前coinIndex的值是否对coinIndex+1有用，也是通过余数来比较。

这个版本的算法空间上的需求是O(1)，满足线形迭代的要求，在计算较大数目的时候不管是时间还是空间的优势都很明显，但是理解起来就比树形递归那个难多了。
