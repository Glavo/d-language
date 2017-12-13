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

  <table>
    <caption>字符串字面量后缀 </caption>
    <tbody>
      <tr>
        <th><b>后缀</b></th>
        <th><b>类型</b></th>
        <th><b>别名</b></th>
      </tr>
      <tr>
        <td>c</td>
        <td>immutable(char)[]</td>
        <td>string</td>
      </tr>
      <tr>
        <td>w</td>
        <td>immutable(wchar)[]</td>
        <td>wstring</td>
      </tr>
      <tr>
        <td>d</td>
        <td>immutable(dchar)[]</td>
        <td>dstring</td>
      </tr>
    </tbody>
  </table>

  ```d
  "hello"c  // string
  "hello"w  // wstring
  "hello"d  // dstring
  ```

6. 字符串字面量默认会转为 UTF-8 字符数组，后缀根据需要转换其至 wchar 或者 dchar。

7. 字符串字面量是只读的。向字符串字面量写入数据不一定每次都被检测到，但是会产生未定义的行为。

### 2.9.4 分隔字符串

1. 分隔符字符串使用各种形式的分隔符来创建字符串。无论是字符还是标识符，分隔符都需要紧跟在起始的 `"` 符号后，之间不能有任何字符；分隔终止符必须紧贴在结束字符串的 `"` 前。分隔符可以嵌套，嵌套的分隔符是下列字符之一：

  <table>
    <caption>嵌套分隔符</caption>
    <tbody>
      <tr>
        <th><b>分隔符</b></th>
        <th><b>配对分隔符</b></th>
      </tr>
      <tr>
        <td>[</td>
        <td>]</td>
      </tr>
      <tr>
        <td>(</td>
        <td>)</td>
      </tr>
      <tr>
        <td><</td>
        <td>></td>
      </tr>
      <tr>
        <td>{</td>
        <td>}</td>
      </tr>
    </tbody>
  </table>

  ```d
  q"(foo(xxx))"   // "foo(xxx)"
  q"[foo{]"       // "foo{"
  ```

2. 如果分隔符是标识符，则分隔符后必须紧跟着一个换行符，而所匹配的分隔符则是一个处于行首的相同标识符：

  ```d
  writeln(q"EOS
  This
  is a multi-line
  heredoc string
  EOS"
  );
  ```

3. 起始标识符后的换行符不是字符串的一部分，但是终止标识符前的最后一个换行符是字符串的一部分。终止标识符必须放在最左边的一列。

4. 其他情况下，起始分隔符和配对分隔符要相同：
  ```d
  q"/foo]/"          // "foo]"
  // q"/abc/def/"    // 错误！
  ```

### 2.9.5 标记字符串

1. 标记字符串以 `q{` 开始，以 `}` 结束，其中的内容必须是有效的 D 标记。`{` 和 `}` 标记可以嵌套。该字符串的内容是标记字符串的起始与结束部分之间的所有字符，包括注释。

  ```d
  q{foo}              // "foo"
  q{/*}*/ }           // "/*}*/ "
  q{ foo(q{hello}); } // " foo(q{hello}); "
  q{ __TIME__ }       // " __TIME__ "，它不会被替换为时间
  // q{ __EOF__ }     // 错误， __EOF__ 不是一个标记，而是文件终止符
  ```

### 2.9.6 转义序列

1. 下表解释了 `EscapeSequence` 中所列出的转义序列的含义：

  <table>
    <caption>转义序列</caption>
    <tbody>
      <tr>
        <th><b>序列</b></th>
        <th><b>含义</b></th>
      </tr>
      <tr>
        <td><b>\'</b></td>
        <td>单引号字面量：'</td>
      </tr>
      <tr>
        <td><b>\"</b></td>
        <td>双引号字面量："</td>
      </tr>
      <tr>
        <td><b>\?</b></td>
        <td>问号字面量：?</td>
      </tr>
      <tr>
        <td><b>\\</b></td>
        <td>反斜线字面量：\</td>
      </tr>
      <tr>
        <td><b>\0</b></td>
        <td>二进制 0（NUL， U+0000）</td>
      </tr>
      <tr>
        <td><b>\a</b></td>
        <td>BEL（警报）字符（U+0007）</td>
      </tr>
      <tr>
        <td>\b</b></td>
        <td>退格（U+0008）</td>
      </tr>
      <tr>
        <td><b>\f</b></td>
        <td>换页（FF）（U+000C）</td>
      </tr>
      <tr>
        <td><b>\n</b></td>
        <td>换行（U+000A）</td>
      </tr>
      <tr>
        <td><b>\r</b></td>
        <td>回车（U+000D）</td>
      </tr>
      <tr>
        <td><b>\t</b></td>
        <td>水平制表（U+0009）</td>
      </tr>
      <tr>
        <td><b>\v</b></td>
        <td>垂直制表（U+000B）</td>
      </tr>
      <tr>
        <td><b>\x</b>nn</td>
        <td>以十六进制表示的字节值，其中 nn 被指定为两个十六进制数字。<br>例如：<b>\xFF</b> 表示值为 255 的字符。</td>
      </tr>
      <tr>
        <td>\n<br>\nn<br>\nnn</td>
        <td>以八进制表示的字节值。<br>例如：<b>\101</b> 表示值为 65 的字符（‘A’）。类似于十六进制字符，其最大的字节值为 <b>\377</b>（= <b>\xFF</b> 或 255）</td>
      </tr>
      <tr>
        <td><b>\u</b>nnnn</td>
        <td>Unicode 字符 U+nnnn，其中 nnnn 是四个十六进制数字。<br>例如：<b>\u042F</b> 表示 Unicode 字符 Я（U+42F）。</td>
      </tr>
      <tr>
        <td><b>\U</b>nnnnnnnn</td>
        <td>Unicode 字符 U+nnnnnnnn，其中 nnnnnnnn 是八个十六进制数字<br>例如：<b>\U0001F603</b> 表示 Unicode 字符 U+1F603。</td>
      </tr>
      <tr>
        <td>\名称</td>
        <td>HTML5 规范中的命名字符实体。相信信息参阅 NamedCharacterEntity。</td>
      </tr>
      </tbody>
  </table>

## 2.10 字符字面量

```pegs
CharacterLiteral:
    ' SingleQuotedCharacter '

SingleQuotedCharacter:
    Character
    EscapeSequence
```

1. 字符字面量是由单引号 `'` 括起来的单个字符或者转义序列。

## 2.11 整数字面量

```pegs
IntegerLiteral:
    Integer
    Integer IntegerSuffix

Integer:
    DecimalInteger
    BinaryInteger
    HexadecimalInteger

IntegerSuffix:
    L
    u
    U
    Lu
    LU
    uL
    UL

DecimalInteger:
    0
    NonZeroDigit
    NonZeroDigit DecimalDigitsUS

BinaryInteger:
    BinPrefix BinaryDigitsUS

BinPrefix:
    0b
    0B

HexadecimalInteger:
    HexPrefix HexDigitsNoSingleUS

NonZeroDigit:
    1
    2
    3
    4
    5
    6
    7
    8
    9

DecimalDigits:
    DecimalDigit
    DecimalDigit DecimalDigits

DecimalDigitsUS:
    DecimalDigitUS
    DecimalDigitUS DecimalDigitsUS

DecimalDigitsNoSingleUS:
    DecimalDigit
    DecimalDigit DecimalDigitsUS
    DecimalDigitsUS DecimalDigit

DecimalDigitsNoStartingUS:
    DecimalDigit
    DecimalDigit DecimalDigitsUS

DecimalDigit:
    0
    NonZeroDigit

DecimalDigitUS:
    DecimalDigit
    _

BinaryDigitsUS:
    BinaryDigitUS
    BinaryDigitUS BinaryDigitsUS

BinaryDigit:
    0
    1

BinaryDigitUS:
    BinaryDigit
    _

OctalDigit:
    0
    1
    2
    3
    4
    5
    6
    7

HexDigits:
    HexDigit
    HexDigit HexDigits

HexDigitsUS:
    HexDigitUS
    HexDigitUS HexDigitsUS

HexDigitsNoSingleUS:
    HexDigit
    HexDigit HexDigitsUS
    HexDigitsUS HexDigit

HexDigitsNoStartingUS:
    HexDigit
    HexDigit HexDigitsUS

HexDigit:
    DecimalDigit
    HexLetter

HexDigitUS:
    HexDigit
    _

HexLetter:
    a
    b
    c
    d
    e
    f
    A
    B
    C
    D
    E
    F
```

1. 整数的值可以用十进制、二进制或者十六进制指定。

2. 十进制整数使一串十进制数字。

3. 二进制整数是以 `0b` 或者 `0B` 开头的二进制数字序列。

4. C 风格的八进制整数字面量被认为太容易与十进制数字混淆，所以只在字符串字面量内得到支持。D 语言通过 [std.conv.octal](https://dlang.org/phobos/std_conv.html#.octal) 模板编译时转换十进制数字到八进制数字，例如 `octal!167`。

5. 十六进制整数是以 `0x` 或者 `0X` 开头的十六进制数字序列。

6. 整数字面量里可以嵌入 `_` 字符，它们会被忽略。嵌入的 `_` 对格式化长字面量很有用，譬如用来做千位分隔符：

  ```d
  123_456       // 123456
  1_2_3_4_5_6_  // 123456
  ```
7. 整数字面量后可以紧跟一个 `L` 或者 `U`（或 `u`）或者两者介有。注意，`l` 不被支持。

8. 整数类型按照下述规则进行判断：

  <table>
    <caption>整数字面量类型</caption>
    <tbody>
    <tr>
      <th><b>字面量范围</b></th> <th><b>类型</b></th>
    <tr>
    <tr>
      <td><b>一般十进制字面量</b></td>
      <td></td>
    </tr>
    <tr>
      <td>0 .. 2_147_483_647</td>
      <td>int</td>
    </tr>
    <tr>
      <td>2_147_483_648 .. 9_223_372_036_854_775_807</td>
      <td>long</td>
    </tr>
    <tr>
      <td><b>带显式后缀的十进制字面量</b></td>
      <td></td>
    </tr>
    <tr>
      <td>0L .. 9_223_372_036_854_775_807L</td>
      <td>long</td>
    </tr>
    <tr>
      <td>0U .. 4_294_967_295U</td>
      <td>uint</td>
    </tr>
    <tr>
      <td>4_294_967_296U .. 18_446_744_073_709_551_615U</td>
      <td>ulong</td>
    </tr>
    <tr>
      <td>0UL .. 18_446_744_073_709_551_615UL</td>
      <td>ulong</td>
    </tr>
    <tr>
      <td><b>十六进制字面量</b></td>
      <td></td>
    </tr>
    <tr>
      <td>0x0 .. 0x7FFF_FFFF</td>
      <td>int</td>
    </tr>
    <tr>
      <td>0x8000_0000 .. 0xFFFF_FFFF</td>
      <td>uint</td>
    </tr>
    <tr>
      <td>0x1_0000_0000 .. 0x7FFF_FFFF_FFFF_FFFF</td>
      <td>long</td>
    </tr>
    <tr>
      <td>0x8000_0000_0000_0000 .. 0xFFFF_FFFF_FFFF_FFFF</td>
      <td>ulong</td>
    </tr>
    <tr>
      <td><b>带显式后缀的十六进制字面量</b></td>
      <td></td>
    </tr>
    <tr>
      <td>0x0L .. 0x7FFF_FFFF_FFFF_FFFFL</td>
      <td>long</td>
    </tr>
    <tr>
      <td>0x8000_0000_0000_0000L .. 0xFFFF_FFFF_FFFF_FFFFL</td>
      <td>ulong</td>
    </tr>
    <tr>
      <td>0x0U .. 0xFFFF_FFFFU</td>
      <td>uint</td>
    </tr>
    <tr>
      <td>0x1_0000_0000U .. 0xFFFF_FFFF_FFFF_FFFFU</td>
      <td>ulong</td>
    </tr>
    <tr>
      <td>0x0UL .. 0xFFFF_FFFF_FFFF_FFFFUL</td>
      <td>ulong</td>
    </tr>
    </tbody>
  <table>

## 2.12 浮点字面量

```pegs
FloatLiteral:
    Float
    Float Suffix
    Integer FloatSuffix
    Integer ImaginarySuffix
    Integer FloatSuffix ImaginarySuffix
    Integer RealSuffix ImaginarySuffix

Float:
    DecimalFloat
    HexFloat

DecimalFloat:
    LeadingDecimal .
    LeadingDecimal . DecimalDigits
    DecimalDigits . DecimalDigitsNoStartingUS DecimalExponent
    . DecimalInteger
    . DecimalInteger DecimalExponent
    LeadingDecimal DecimalExponent

DecimalExponent
    DecimalExponentStart DecimalDigitsNoSingleUS

DecimalExponentStart
    e
    E
    e+
    E+
    e-
    E-

HexFloat:
    HexPrefix HexDigitsNoSingleUS . HexDigitsNoStartingUS HexExponent
    HexPrefix . HexDigitsNoStartingUS HexExponent
    HexPrefix HexDigitsNoSingleUS HexExponent

HexPrefix:
    0x
    0X

HexExponent:
    HexExponentStart DecimalDigitsNoSingleUS

HexExponentStart:
    p
    P
    p+
    P+
    p-
    P-


Suffix:
    FloatSuffix
    RealSuffix
    ImaginarySuffix
    FloatSuffix ImaginarySuffix
    RealSuffix ImaginarySuffix

FloatSuffix:
    f
    F

RealSuffix:
    L

ImaginarySuffix:
    i

LeadingDecimal:
    DecimalInteger
    0 DecimalDigitsNoSingleUS
```

1. 浮点数字面量可用十进制或十六进制表示。

2. 十六进制浮点字面量前有 `0x` 或者 `0X` 前缀；指数部分是一个 `p` 或者 `P` 后跟着一个十进制数，以它做以 2 为底的指数。

3. 浮点字面量里可以嵌入 `_` 字符，它们会被忽略。嵌入的 `_` 对格式化长字面量很有用，譬如用来做千位分隔符：

  ```d
  123_456.567_8         // 123456.5678
  1_2_3_4_5_6_.5_6_7_8  // 123456.5678
  1_2_3_4_5_6_.5e-6_    // 123456.5e-6
  ```

4. 没有后置的浮点字面量是 `double` 类型的。浮点数可以跟随一个 `f`、`F` 或者 `L` 后缀。`f` 和 `F` 后缀表示它使 `float` 类型的，而 `L` 后缀表示它是 `real` 类型的。

5. 如果浮点字面量后跟有后缀 `i`，说明它是 `ireal` 类型（虚数类型）的。

6. 例子：

  ```d
  0x1.FFFFFFFFFFFFFp1023 // double.max
  0x1p-52                // double.epsilon
  1.175494351e-38F       // float.min
  6.3i                   // idouble 6.3
  6.3fi                  // ifloat 6.3
  6.3Li                  // ireal 6.3
  ```

7. 如果该字面量超过了该类型的表示范围会被视为错误。如果该字面量四舍五入后可用适应该类型的有效数字，则不是错误。

8. 如果一个浮点字面量有 `.` 以及类型后缀，那么在 `.` 与后缀之间至少要有一个数字：
  ```d
  1f;   // 正确
  1.f;  // 禁止
  1.;   // 正确，double 类型
  ```

9. 复数字面量不是标记，而是语义分析时由虚数和实数表达式构成的。

  ```d
  4.5 + 6.2i  // complex number (phased out)
  ```

## 2.13 关键字

关键字是被保留的标识符。

参见：全局符号。

```pegs
Keyword:
    abstract
    alias
    align
    asm
    assert
    auto

    body
    bool
    break
    byte

    case
    cast
    catch
    cdouble
    cent
    cfloat
    char
    class
    const
    continue
    creal

    dchar
    debug
    default
    delegate
    delete (deprecated)
    deprecated
    do
    double

    else
    enum
    export
    extern

    false
    final
    finally
    float
    for
    foreach
    foreach_reverse
    function

    goto

    idouble
    if
    ifloat
    immutable
    import
    in
    inout
    int
    interface
    invariant
    ireal
    is

    lazy
    long

    macro (unused)
    mixin
    module

    new
    nothrow
    null

    out
    override

    package
    pragma
    private
    protected
    public
    pure

    real
    ref
    return

    scope
    shared
    short
    static
    struct
    super
    switch
    synchronized

    template
    this
    throw
    true
    try
    typedef (deprecated)
    typeid
    typeof

    ubyte
    ucent
    uint
    ulong
    union
    unittest
    ushort

    version
    void
    volatile (deprecated)

    wchar
    while
    with

    __FILE__
    __FILE_FULL_PATH__
    __MODULE__
    __LINE__
    __FUNCTION__
    __PRETTY_FUNCTION__

    __gshared
    __traits
    __vector
    __parameters
```

# 2.14 全局符号

1. 这些符号是在 object_.d 中定义的，它们被默认自动导入。

```pegs
Symbols:
    string (alias to immutable(char)[])
    wstring (alias to immutable(wchar)[])
    dstring (alias to immutable(dchar)[])

    size_t
    ptrdiff_t
```

# 2.15 特殊标记

1. 根据下表，这些标记会被替换为其他标记：

  <table>
    <caption>特殊标记</caption>
    <tr>
      <th><b>特殊的标记</b></th>
      <th><b>被替换为</b></th>
    </tr>
    <tr>
      <td>__DATE__</td>
      <td>用字符串字面量表示的编译时日期 "mmm dd yyyy"</td>
    </tr>
    <tr>
      <td>__EOF__</td>
      <td>将词法分析器状态设置为扫描结束此文件</td>
    </tr>
    <tr>
      <td>__TIME__</td>
      <td>用字符串字面量表示的编译时的时间 "hh:mm:ss"</td>
    </tr>
    <tr>
      <td>__TIMESTAMP__</td>
      <td>用字符串字面量表示的编译时的日期与时间 "www mmm dd hh:mm:ss yyyy"</td>
    </tr>
    <tr>
      <td>__VENDOR__</td>
      <td>编译器供应商的字符串表示，例如 "Digital Mars D"</td>
    </tr>
    <tr>
      <td>__VERSION__</td>
      <td>编译器版本的整数表示，例如 2001</td>
    </tr>
  </table>
