---
title: Ratina
date: 2017-01-19 13:31:18
tags: ratina
---

```haskell
main = do
  ratina <- createWorld "Ratina"
  putStrLn "新しい世界はこれからだ"
  forever $ createMiracles `in_` ratina
```
