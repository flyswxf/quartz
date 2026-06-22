当程序中出现`if...then...else...end`时有**固定的**翻译成形式化语言的方法

## H $\vdash$ [if C then A else B]Q
翻译成两个子证明:
- H, C $\vdash$ [A]Q
- H, $\neg$ C $\vdash$ [B]Q

