修改内存前, 复制一份该内存页到硬盘(这份内存页不是脏页), 称为Journal file
如果执行[[Undo]], 只需要用复制的Journal file恢复硬盘即可