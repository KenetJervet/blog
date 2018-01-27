---
title: GHC扩展极简说明之二 —— 模式和guard类扩展
date: 2017-01-23 23:18:38
tags: 
  - Haskell
  - GHC扩展
---

## BangPatterns

可用版本：GHC所有较新版本

### 含义

允许对模式的匹配求值到弱首范式（WHNF）。

### 用法举例

```haskell
-- 强制惰性
lazyFunc ~() = ()

-- 强制严格
strictFunc !v = ()

-- 将lazyFunc换为strictFunc将报错
main = print $ lazyFunc undefined
```

---

## ViewPatterns

可用版本：GHC所有较新版本

### 含义

提供形如`e -> p`的模式语法，将e作用于待匹配参数，将其结果与p作匹配。

### 用法举例

```haskell
eitherEndIsZero :: [Int] -> Bool
eitherEndIsZero (head -> 0) = True
eitherEndIsZero (last -> 0) = True
eitherEndIsZero          _  = False
```

---

## PatternGuards

可用版本：GHC所有较新版本，Haskell2010已包含

### 含义

提供多个匹配条件（布尔guard或模式guard）的串联。

### 用法举例

```haskell
x :: Int -> Int
x a | a > 5
    , 10 <- a + 3 = 0
    | otherwise = 1
```

---

## PatternSynonyms

可用版本：GHC >= 7.10

### 含义

模式别名，提供对模式的抽象。

分单向和双向两种。单向别名仅用作模式，双向模式可用于表达式。

### 用法举例

```haskell
pattern P1 a b <- (a, _, b)  -- 单向
pattern P2 a b = (a, (b, 3))  -- 双向
pattern P2E a b = (a, _, b)  -- 双向（非法）
```
