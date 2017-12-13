# 2. 词法

1. 词法分析独立于语法分析和语义分析。词法分析器将源文本分割为标记，词法描述了这些标记的语法。D 语言的词法设计适合于进行高速扫描，并易于为其编写正确的词法分析器。它有最小化的特例情况，并且只用进行一遍分析。熟悉 C 和 C++ 的人很容易识别这些标记。

## 2.1 源文本

1. D 源文本可以使用下面的几种编码之一：

    * ASCII
    * UTF-8
    * UTF-16BE
    * UTF-16LE
    * UTF-32BE
    * UTF-32LE    

2. UTF-8 是传统 7-bit ASCII 的超集。以下 UTF BOM （字节序标记）可以出现在源文本的开头部分：

    UTF 字节序标记|
    ---------|-------
    格式      | BOM
    UTF-8    | EF BB BF
    UTF-16BE | FE FF
    UTF-16LE | FF FE
    UTF-32BE | 00 00 FE FF
    UTF-32LE | FF FE 00 00
    ASCII    | no BOM

3. 如果源文件不是以 BOM 开头，那么第一个字符必须小于等于 U+0000007F。

4. D 里不存在二合字以及三合字。

5. 源文本的源表示会被解码为 Unicode [字符](https://dlang.org/spec/lex.html#Character)序列。这些字符又会被进一步划分为：[空白符](https://dlang.org/spec/lex.html#WhiteSpace)、[行终结符](https://dlang.org/spec/lex.html#WhiteSpace)、[注释](https://dlang.org/spec/lex.html#Comment)、[特殊字符序列](https://dlang.org/spec/lex.html#SpecialTokenSequence)、[标记](https://dlang.org/spec/lex.html#Token)以及[文件终结符](https://dlang.org/spec/lex.html#EndOfFile)。

6. 源文本使用贪心算法分割成标记，即词法分析器总是试图分割出最长的标记。例如 `>>` 是一个右移标记，而不是两个大于标记。