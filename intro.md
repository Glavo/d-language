# 1. 简介

1. D 语言是一种通用的系统编程语言。D 程序是一个模块集合，每个模块可以单独编译为本地代码，并由链接器编译以创建本机可执行文件。


## 1.1 编译阶段

1. 编译过程分为多个阶段，每个阶段都不依赖于后续阶段。例如，词法分析器不受语义分析器的应用。通过这种分离，更容易实现 IDE 等语言工具。也可以通过以“标记”的形式压缩存储 D 源代码。

    1. **源文件字符集**

    检查源文件的字符集，从而选择不同的词法分析器。D 编译器接受 ASCII 以及 UTF 编码的源文件。

    2. **脚本行**
    
    如果源文件第一行以 `#!` 开头，那么它将会被忽略。
    
    3. **词法分析**
    
    将源文件分割为标记序列。[特殊标记](https://dlang.org/spec/lex.html#specialtokens)会被替换为其他的标记；特殊标记序列会被处理后移除。
    
    4. **语法分析**
    
    标记序列会被解析成语法树。
    
    5. **语义分析**
    
    编译器会遍历语法树来声明变量、加载符号表、分配类型，并大体确定程序语义。
    
    6. **优化**
    
    这一步是可选的。编译器会试图在保证语义不变的情况下生成运行速度更快的程序。
    
    7. **代码生成**
    
    编译器根据目标平台架构选取对应的指令集，生成表现程序的语义的结果。代码生成的结果通常为一个适应于连接器的文件。
    