---
title: GHC扩展极简说明之一 —— 基本语法扩展
date: 2017-01-22 23:19:28
tags: Haskell,GHC扩展
---

## PostfixOperators

可用版本：GHC所有较新版本

### 含义

可将运算符用作后缀。

### 用法举例

```haskell
-- 未打开扩展时
(4 !)  -- 等同于 \x -> 4 ! x

-- 打开扩展后，!可以为一元运算符
(!) :: Integer -> Integer
(!) = 省略

(4 !) :: Integer
```

---

## TupleSections

可用版本：GHC >= 6.12

### 含义

可使用部分应用的元组。

### 用法举例

```haskell
(1, "Hello", , , "World")
```

---

## PackageImports

可用版本：GHC所有较新版本

### 含义

可显式地从指定包中导入模块

### 用法举例

```haskell
import "package-one" Data.Module.X
import "package-two-0.1.0.1" Data.Module.X
```

---

## OverloadedStrings

可用版本：GHC所有较新版本

### 含义

可将字面字符串转变为多态类型。

### 用法举例

```haskell
-- 未开启OverloadedStrings时
:t "Hello"
"Hello" :: [Char]

-- 开启OverloadedStrings后
:t "Hello"
"Hello" :: Data.String.IsString t => t
-- 于是可以
a :: Text
a = "World"
```

---

## UnicodeSyntax

可用版本：GHC所有较新版本

### 含义

可使用诸多Unicode符号替代标准运算符

### 用法举例

```haskell
{-# LANGUAGE UnicodeSyntax #-}

import Data.List.Unicode ((∪))

print $ [1, 2, 3] ∪ [4, 5, 6]
```

---

## RecursiveDo

可用版本：GHC所有较新版本

### 含义

启用monad上下文递归。

一般情况下，monad不允许递归，比如

```haskell
do x <- return $ fst y
   y <- return $ (3, x)
   return $ snd y
```

报“y不在作用域中”的错误。

使用`Control.Monad.Fix`中的mfix可以解决这一问题：

```haskell
do y <- mfix $ \ y0 -> do x  <- return $ fst y0
                          y1 <- return $ (3, x)
                          return $ y1
   return $ snd y
```

开启`RecursiveDo`之后，可使用下面两种形式：

### 用法举例

```haskell
mdo x <- return $ fst y
    y <- return $ (3, x)
    return $ snd y
    
-- 或者

do rec x <- return $ fst y
       y <- return $ (3, x)
       return $ snd y
```

### 注意事项

`mdo`和`rec`略有不同之处：涉及GHC中的优化策略“segmentation”。`mdo`会进行
“segmentation”操作，而`rec`不会。详情[点击此处](https://www.schoolofhaskell.com/school/to-infinity-and-beyond/pick-of-the-week/guide-to-ghc-extensions/basic-syntax-extensions#recursivedo-and-dorec)

---

## LambdaCase

可用版本：GHC >= 7.6

### 含义

将形如`\x -> case x of ...`的代码简化为`\case ...`。

### 用法举例

```haskell
forM_ [Just 1, Just 2, Nothing] $ \case
  Just v  -> ...
  Nothing -> ...
```

---

## EmptyCase

可用版本：GHC >= 7.8

### 含义

允许形如`\x -> case x of {}`或`\case {}`的代码。
与使用`error`或`undefined`不同之处在于，将来可以得到“条件完整度检查”的支持。

---

## MultiwayIf

可用版本：GHC >= 7.6

### 含义

允许多条件`if`表达式。

### 用法举例

```haskell
fn :: Int -> Int -> String
fn x y = if | x == 1    -> "a"
            | y <  2    -> "b"
            | otherwise -> "c"
```

---

## BinaryLiterals

可用版本：GHC >= 7.10

### 含义

允许二进制字面值（形如`0b10110`或`0B10110`）

### 备注

Haskell默认还支持
八进制字面值（形如`0o137`或`0O137`）
以及
十六进制字面值（形如`0x1B`或`0X1b`）

---

## NegativeLiterals

可用版本：GHC >= 7.8

### 含义

将负号原本含义由：
`-1458 <=> negate (fromInteger 1458)`
变为：
`-1458 <=> fromInteger (-1458)`

在极特殊的边缘情况下可能导致性能提升或行为变化，不过一般情况下没卵用。

---

## NumDecimals

可用版本：GHC >= 7.8

### 含义

对于小数字面值，若小数部分仅由0组成,如`1.00`或`1.4790e3`，则将Haskell默认给出的`(Fractional t) => t`的类型更改为更泛化的：`(Num a) => a`。

---

## DoAndIfThenElse

可用版本：GHC >= 7.0

### 含义

在do-notation中，允许if, then和else对齐（实现原理为对于if表达式的解析规则更改为：exp → if exp1 [;] then exp2 [;] else exp3）

### 备注

`ghc -make`默认会启用此扩展，而cabal不会。

---

## NondecreasingIndentation

可用版本：GHC >= 7.2

参见[此处](https://prime.haskell.org/wiki/NondecreasingIndentation)

没个卵用。
