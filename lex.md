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

  <table>
    <caption>UTF Byte Order Marks</caption>    
    <tbody>
      <tr>
        <th><b>Format</b></th>
        <th><b>BOM</b></th>
      </tr>
	    <tr><td>UTF-8</td>    <td>EF BB BF</td></tr>
	    <tr><td>UTF-16BE</td> <td>FE FF</td></tr>
	    <tr><td>UTF-16LE</td> <td>FF FE</td></tr>
      <tr><td>UTF-32BE</td> <td>00 00 FE FF</td></tr>
	    <tr><td>UTF-32LE</td> <td>FF FE 00 00</td></tr>
	    <tr><td>ASCII</td>    <td>no BOM</td></tr>
    </tbody>
  </table>

3. 如果源文件不是以 BOM 开头，那么第一个字符必须小于等于 U+0000007F。

4. D 里不存在二合字以及三合字。

5. 源文本的源表示会被解码为 Unicode [字符](https://dlang.org/spec/lex.html#Character)序列。这些字符又会被进一步划分为：[空白符](https://dlang.org/spec/lex.html#WhiteSpace)、[行终结符](https://dlang.org/spec/lex.html#WhiteSpace)、[注释](https://dlang.org/spec/lex.html#Comment)、[特殊字符序列](https://dlang.org/spec/lex.html#SpecialTokenSequence)、[标记](https://dlang.org/spec/lex.html#Token)以及[文件终结符](https://dlang.org/spec/lex.html#EndOfFile)。

6. 源文本使用贪心算法分割成标记，即词法分析器总是试图分割出最长的标记。例如 `>>` 是一个右移标记，而不是两个大于标记。这个规则有两个例外：

   * 当一个 `..` 嵌入两个类似浮点数字面量的中间时，它会被当做第一个整数后用一个空格分隔解释。

   * 一个 `1.a` 会被解释为三个符号`1`、`.`和`a`，而 `1. a`会被解释为两个符号 `1.` 和 `a`。

## 2.2 字符集

```pegs
Character:
    任何 Unicode 字符
```

## 2.3 文件终止符

```pegs
EndOfFile:
    物理文件的结尾
    \u0000
    \u001A
```

1. 源文本以第一个遇到的为准终止。

## 2.4 行结束符

```pegs
EndOfLine:
    \u000D
    \u000A
    \u000D \u000A
    \u2028
    \u2029
    EndOfFile

```

1. 不支持通过反斜线拼接行，每行也没有字符数限制。

## 2.5 空白符

```pegs
WhiteSpace:
    Space
    Space WhiteSpace

Space:
    \u0020
    \u0009
    \u000B
    \u000C
```

## 2.6 注释

```pegs
    Comment:
        BlockComment
        LineComment
        NestingBlockComment

    BlockComment:
        /* Characters */

    LineComment:
        // Characters EndOfLine

    NestingBlockComment:
        /+ NestingBlockCommentCharacters +/

    NestingBlockCommentCharacters:
        NestingBlockCommentCharacter
        NestingBlockCommentCharacter NestingBlockCommentCharacters

    NestingBlockCommentCharacter:
        Character
        NestingBlockComment

    Characters:
        Character
        Character Characters
```

1. D 支持三种注释：
    1. 可以跨越多行，但是不能嵌套的块注释。
    2. 在行尾结束的单行注释。
    3. 可跨越多行并且可以嵌套的嵌套块注释。

2. 字符串和注释的内容不会被分析。因此，在字符串内部的注释起始符不会开启一个注释，注释内的字符串分隔符也不会影响对注释结尾以及嵌套注释的识别。除了在嵌套块注释内部的 `/+` 之外，注释内部其他的注释起始符都会被忽略。

    ```d
    a = /+ // +/ 1;    // parses as if 'a = 1;'
    a = /+ "+/" +/ 1"; // parses as if 'a = " +/ 1";'
    a = /+ /* +/ */ 3; // parses as if 'a = */ 3;'
    ```

3. 注释不能够连接两个标记，譬如 `abc/**/def` 是两个标记 `abc` 和 `def`，而不是一个标记 `abcdef`。

## 2.7 标记

```pegs
Token:
    Identifier
    StringLiteral
    CharacterLiteral
    IntegerLiteral
    FloatLiteral
    Keyword
    /
    /=
    .
    ..
    ...
    &
    &=
    &&
    |
    |=
    ||
    -
    -=
    --
    +
    +=
    ++
    <
    <=
    <<
    <<=
    <>
    <>=
    >
    >=
    >>=
    >>>=
    >>
    >>>
    !
    !=
    !<>
    !<>=
    !<
    !<=
    !>
    !>=
    (
    )
    [
    ]
    {
    }
    ?
    ,
    ;
    :
    $
    =
    ==
    *
    *=
    %
    %=
    ^
    ^=
    ^^
    ^^=
    ~
    ~=
    @
    =>
    #
```

## 2.8 标识符

```pegs
Identifier:
    IdentifierStart
    IdentifierStart IdentifierChars

IdentifierChars:
    IdentifierChar
    IdentifierChar IdentifierChars

IdentifierStart:
    _
    Letter
    UniversalAlpha

IdentifierChar:
    IdentifierStart
    0
    NonZeroDigit
```

1. 标识符以一个字符、`_`或者通用字母后接任意个字母、`_`数字或者通用字母组成。通用字母定义于 ISO/IEC 9899:1999(E) 的附录 D中。标识符长度没有限制，并区分大小写。以 `__` （两个下划线）开头的标识符被保留。

## 2.9 字符串字面量

```pegs
StringLiteral:
    WysiwygString
    AlternateWysiwygString
    DoubleQuotedString

    HexString
    DelimitedString
    TokenString

WysiwygString:
    r" WysiwygCharacters " StringPostfixopt

AlternateWysiwygString:
    ` WysiwygCharacters ` StringPostfixopt

WysiwygCharacters:
    WysiwygCharacter
    WysiwygCharacter WysiwygCharacters

WysiwygCharacter:
    Character
    EndOfLine

DoubleQuotedString:
    " DoubleQuotedCharacters " StringPostfixopt

DoubleQuotedCharacters:
    DoubleQuotedCharacter
    DoubleQuotedCharacter DoubleQuotedCharacters

DoubleQuotedCharacter:
    Character
    EscapeSequence
    EndOfLine

EscapeSequence:
    \'
    \"
    \?
    \\
    \0
    \a
    \b
    \f
    \n
    \r
    \t
    \v
    \x HexDigit HexDigit
    \ OctalDigit
    \ OctalDigit OctalDigit
    \ OctalDigit OctalDigit OctalDigit
    \u HexDigit HexDigit HexDigit HexDigit
    \U HexDigit HexDigit HexDigit HexDigit HexDigit HexDigit HexDigit HexDigit
    \ NamedCharacterEntity

HexString:
    x" HexStringChars " StringPostfixopt

HexStringChars:
    HexStringChar
    HexStringChar HexStringChars

HexStringChar:
    HexDigit
    WhiteSpace
    EndOfLine

StringPostfix:
    c
    w
    d

DelimitedString:
    q" Delimiter WysiwygCharacters MatchingDelimiter "

TokenString:
    q{ Tokens }
```

1. 一个字符串字面量可以是双引号字符串、所见即所得字符串、定界字符串、标记字符串或者十六进制字符串。

2. 在所有的字符串字面量中，一个行终结符都被看做是一个 `\n` 字符。

### 2.9.1 所见即所得字符串

1. 所见即所得字符串需要被 `r"` 以及 `"` 括起来，位于 `r"` 与 `"` 之间的所有字符都是字符串的一部分。`r""` 中没有转义序列。

  ```d
  r"hello"
  r"c:\root\foo.exe"
  r"ab\n" // 四个字符 'a'，'b'，'\'，'n'
          // 组成的字符串
  ```

2. 所见即所得字符串还有一个替代形式，就是以反引号 \` 括起字符串内容。并不是所有的键盘上都有 \` 字符，并且有些时候我们在屏幕上很难把它与常见的的 `'` 所区分。但是因为很少使用 \` 这个字符，所以我们常用它来表示一个包含 `"` 字符的字符串。

  ```d
  `hello`
  `c:\root\foo.exe`
  `The "lazy" dog`
  `a"b\n` // 五个字符 'a'，'"'，'b'，'\'，'n'
          // 组成的字符串
  ```

### 2.9.2 双引号字符串

1. 双引号字符串指被 `"` 括起的字符串，可以通过 `\` 标记往双引号字符串内嵌入转义序列。

  ```d
  "hello"
  "c:\\root\\foo.exe"
  "ab\n"  // 三个字符 'a'，'b'
          // 和一个换行符组成的字符串
  "ab
  "     // 三个字符 'a'，'b'
        // 和一个换行符组成的字符串
  ```

### 2.9.3 十六进制字符串

1. 十六进制字符串允许使用十六进制数据组成字符串，十六进制数据不需要形成有效的 UTF 字符。

  ```d
  x"0A"              // 和 "\x0A" 相同
  x"00 FBCD 32FD 0A" // 和
    // "\x00\xFB\xCD\x32\xFD\x0A"相同
  ```

2. 空白符和换行符会被忽略，所有可以很容易的格式化十六进制字符串。十六进制字符串里十六进制字符的数量必须是二的倍数。

3. 相邻的字符串可用 `~` 运算符连接：

  ```d
  "hello " ~ "world" ~ "\n" // 形成字符串
       // 'h','e','l','l','o',' ',
       // 'w','o','r','l','d',linefeed
  ```

4. 下面的字符串全部等价：

  ```d
  "ab" ~ "c"
  r"ab" ~ r"c"
  r"a" ~ "bc"
  "a" ~ "b" ~ "c"
  ```

5. 可选的_字符串后缀_为字符串提供了一个特定的类型，此时字符串的类型不需要从上下文推断。当类型不能被明确推断出来时（例如调用基于字符串类型重载的函数），明确标注类型有很大的用处。后缀与字符串类型有这样的对应关系：

字符串字面量后缀 |
--|--------------------|--------
c | immutable(char)[]  | string
w | immutable(wchar)[] | wstring
d | immutable(dchar)[] | dstring
