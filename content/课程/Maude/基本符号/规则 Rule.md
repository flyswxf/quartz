[[模块 Modules#系统模块 (System Modules)|系统模块]]中的规则用于描述状态间的转换，属于重写语义的一部分。与[[等式 Equation|等式]]不同，规则不做值计算，而是刻画系统行为的演化。

## 语法

```maude
rl [Label] : Term => Term' .               -- 无条件规则
crl [Label] : Term => Term' if Condition . -- 条件规则
```

**说明：**
- `Label`：规则的名字。
- `Term`、`Term'`：重写左项与右项。
- `Condition`：布尔条件（在等式理论下简化为 `true` 时规则可用）。

## 示例：无条件规则

```maude
mod SWITCH is
  sort S .
  ops on off : -> S .

  rl [toOff] : on => off .   -- 从 on 切换到 off
  rl [toOn]  : off => on .   -- 从 off 切换到 on
endm
```

## 示例：条件规则

```maude
mod TIMER is
  protecting NAT .

  sort Timer .
  op tick : Nat -> Timer .

  rl  [inc]  : tick(N) => tick(N + 1) .          -- 计数加一
  crl [limit] : tick(N) => tick(N) if 10 <= N .  -- 达到阈值后保持不变
endm
```


由于重写规则通常是**非确定性**（Non-deterministic）和**并发**（Concurrent）的，同一个初始状态可能会演化出多种不同的结果。因此，Maude 提供了三种不同的命令来观察和验证这些行为。

1. [[重写命令#rewrite (缩写 rew)|rewrite]]
2. [[重写命令#frewrite (缩写 frew)|frewrite]]
3. [[重写命令#search|search]]