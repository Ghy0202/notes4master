# Finding Missed Compiler Optimizations by Differential Testing阅读笔记

## Abstract

对编译器的随机差分测试在对于编译器的崩溃和沉默的错误编译的发现非常有用。本文探讨了相关技术是否可以提高生成代码的能力：找到某个编译器优化但是另一个编译器错过的优化？

根据感兴趣的优化，可以将该工具配置为比较特性，如总指令数、乘或除指令、函数调用、堆栈访问等。一旦发现了一个有趣的差异，一个标准的测试用例缩减工具就会产生最小的示例。

本文中比较了GCC\Clang\CompCert，在这三者中均发现了错过的算数优化以及寄存器溢出、错过了寄存器的合并机会、死储存、冗余计算和缺少指令选择模式等个别情况。

## Movtivation





## Contribution

- 一个用于找寻缺失优化的C的差分测试方法
- 一套用于自动化该方法的可配置工具
- 对该方法和工具的实验评估以及以前没有报告过的遗漏优化的例子