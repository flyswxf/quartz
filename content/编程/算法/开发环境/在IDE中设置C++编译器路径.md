## 可解决问题
1. max,min等函数明明include了对应头文件但是IDE提示没有定义
2. vector 用`V={1,2,3,4};`定义报错

## 原因
1. **C++编译器太老了**，比如cl， 解决方法见[[在IDE中设置C++编译器路径#^aaa498|步骤一]]
2. **下载了新的C++编译器**，但是没有告诉IDE它的路径，路径仍然指向旧编译器 解决方法见[[在IDE中设置C++编译器路径#^5985da|步骤二]]
	- 这会导致**IDE中显示错误，但是用命令行进行编译是可以通过的**
## 步骤
1. 下载新的C++编译器 ^aaa498
	1. 访问[MSYS2](https://www.msys2.org/)
	2. 在打开的MSYS2命令行窗口中执行：
	   `pacman -S --needed base-devel mingw-w64-x86_64-toolchain`
	3. 下载完成后，MSYS2会自动将系统环境变量中的路径配置好，继续在MSYS2中输入`g++ --version`可以确定新编译器可以使用
2. 在IDE中设置C++编译器路径 ^5985da
	1. 在设置中选择**Editor设置**
	   ![[assets/编辑器设置界面.png]]
	2. 搜索cpp，找到**Compiler Path**
	   ![[assets/编译器路径设置.png]]
	3. 设置编译器路径
	   `"C_Cpp.default.compilerPath": "D:/coding/msys64/mingw64/bin/g++.exe"`
	   ![[assets/编译器路径配置.png]]

