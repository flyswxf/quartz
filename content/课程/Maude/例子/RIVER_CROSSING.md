# 农夫过河问题 (River Crossing)

这是一个经典的逻辑谜题：一个农夫（Shepherd）带着一只狼（Wolf）、一只羊（Lamb）和一颗白菜（Cabbage）过河。

**规则限制：**
1. 只有农夫能划船。
2. 船很小，每次农夫只能带一样东西（或自己走）。
3. 农夫不在场时：
   - 狼会吃羊。
   - 羊会吃白菜。
4. **目标**：将所有人和物安全地从左岸运到右岸。

## Maude 代码实现

以下代码使用重写逻辑（Rewriting Logic）对该问题进行建模。

```maude
mod RIVER-CROSSING is
  sorts Side Group .
  
  --- 1. 定义两岸及切换操作
  ops left right : -> Side [ctor] .
  op change : Side -> Side .
  eq change(left) = right .
  eq change(right) = left .

  --- 2. 定义角色：农夫(s), 狼(w), 羊(l), 白菜(c)
  --- 每个角色都带有位置状态 (Side)
  ops s w l c : Side -> Group [ctor] .
  
  --- 定义群组 (Group)，使用空格连接，满足结合律和交换律
  op __ : Group Group -> Group [ctor assoc comm] .

  vars S S1 S2 S3 : Side .

  --- 3. 定义安全判定函数 isSafe
  op isSafe : Group -> Bool .
  
  --- 只要农夫在场，就是安全的
  eq isSafe(s(S) G:Group) = true .
  
  --- 农夫不在场时的安全检查：
  --- 三者都在：狼和羊不能同岸，羊和白菜不能同岸
  eq isSafe(w(S1) l(S2) c(S3)) = (S1 =/= S2) and (S2 =/= S3) .
  --- 只有狼和羊：不能同岸
  eq isSafe(w(S1) l(S2)) = (S1 =/= S2) .
  --- 只有羊和白菜：不能同岸
  eq isSafe(l(S2) c(S3)) = (S2 =/= S3) .
  --- 只有狼和白菜：总是安全（狼不吃白菜）
  eq isSafe(w(S1) c(S2)) = true .

  --- 4. 状态转移规则 (Rules)
  --- [shepherd]: 农夫独自过河
  crl [shepherd] : s(S) w(S1) l(S2) c(S3) => s(change(S)) w(S1) l(S2) c(S3) 
    if isSafe(w(S1) l(S2) c(S3)) . 

  --- [wolf]: 农夫带狼过河
  crl [wolf] : s(S) w(S) l(S1) c(S2) => s(change(S)) w(change(S)) l(S1) c(S2) 
    if isSafe(l(S1) c(S2)) .

  --- [lamb]: 农夫带羊过河
  --- 羊被带走后，剩下狼和白菜是安全的，因此不需要条件
  rl [lamb] : s(S) l(S) => s(change(S)) l(change(S)) .

  --- [cabbage]: 农夫带白菜过河
  crl [cabbage] : s(S) c(S) w(S1) l(S2) => s(change(S)) c(change(S)) w(S1) l(S2) 
    if isSafe(w(S1) l(S2)) .
endm
```

## 求解与验证

使用 `search` 命令寻找从初始状态（全在左岸）到目标状态（全在右岸）的路径。

```maude
search s(left) w(left) l(left) c(left) =>* s(right) w(right) l(right) c(right) .
```

### 求解结果 (Path 9)

Maude 找到了一条包含 7 步移动的解法：

1. **[lamb]** 农夫带**羊**去右岸。 (左: 狼, 白菜 | 右: 农夫, 羊)
2. **[shepherd]** 农夫**独自**回左岸。 (左: 农夫, 狼, 白菜 | 右: 羊)
3. **[wolf]** 农夫带**狼**去右岸。 (左: 白菜 | 右: 农夫, 狼, 羊)
4. **[lamb]** 农夫带**羊**回左岸。 (左: 农夫, 羊, 白菜 | 右: 狼)
   *注：为了防止狼吃羊，农夫必须把羊带回来。*
5. **[cabbage]** 农夫带**白菜**去右岸。 (左: 羊 | 右: 农夫, 狼, 白菜)
6. **[shepherd]** 农夫**独自**回左岸。 (左: 农夫, 羊 | 右: 狼, 白菜)
7. **[lamb]** 农夫带**羊**去右岸。 (左: - | 右: 农夫, 狼, 白菜, 羊)

**完成！**
