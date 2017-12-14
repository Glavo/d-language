# 3. 语法

## 3.1 词法

1. 请参考页面 [词法](https://glavo.gitbooks.io/d-language/lex.html)。

## 3.2 类型

```pegs
Type:
    TypeCtorsopt BasicType BasicType2opt

TypeCtors:
    TypeCtor
    TypeCtor TypeCtors

TypeCtor:
    const
    immutable
    inout
    shared

BasicType:
    BasicTypeX
    . IdentifierList
    IdentifierList
    Typeof
    Typeof . IdentifierList
    TypeCtor ( Type )
    TypeVector

BasicTypeX:
    bool
    byte
    ubyte
    short
    ushort
    int
    uint
    long
    ulong
    char
    wchar
    dchar
    float
    double
    real
    ifloat
    idouble
    ireal
    cfloat
    cdouble
    creal
    void

BasicType2:
    BasicType2X BasicType2opt

BasicType2X:
    *
    [ ]
    [ AssignExpression ]
    [ AssignExpression .. AssignExpression ]
    [ Type ]
    delegate Parameters MemberFunctionAttributesopt
    function Parameters FunctionAttributesopt

IdentifierList:
    Identifier
    Identifier . IdentifierList
    TemplateInstance
    TemplateInstance . IdentifierList
    Identifier [ AssignExpression ]. IdentifierList

Typeof:
    typeof ( Expression )
    typeof ( return )

TypeVector:
    __vector ( Type )
```

## 3.2 表达式

```pegs
Expression:
    CommaExpression

CommaExpression:
    AssignExpression
    AssignExpression , CommaExpression
```

```pegs
AssignExpression:
    ConditionalExpression
    ConditionalExpression = AssignExpression
    ConditionalExpression += AssignExpression
    ConditionalExpression -= AssignExpression
    ConditionalExpression *= AssignExpression
    ConditionalExpression /= AssignExpression
    ConditionalExpression %= AssignExpression
    ConditionalExpression &= AssignExpression
    ConditionalExpression |= AssignExpression
    ConditionalExpression ^= AssignExpression
    ConditionalExpression ~= AssignExpression
    ConditionalExpression <<= AssignExpression
    ConditionalExpression >>= AssignExpression
    ConditionalExpression >>>= AssignExpression
    ConditionalExpression ^^= AssignExpression
```

```pegs
ConditionalExpression:
    OrOrExpression
    OrOrExpression ? Expression : ConditionalExpression
```

```pegs
OrOrExpression:
    AndAndExpression
    OrOrExpression || AndAndExpression
```

```pegs
AndAndExpression:
    OrExpression
    AndAndExpression && OrExpression
    CmpExpression
    AndAndExpression && CmpExpression
```

```pegs
OrExpression:
    XorExpression
    OrExpression | XorExpression
```

```pegs
XorExpression:
    AndExpression
    XorExpression ^ AndExpression
```

```pegs
AndExpression:
    ShiftExpression
    AndExpression & ShiftExpression
```

```pegs
CmpExpression:
    ShiftExpression
    EqualExpression
    IdentityExpression
    RelExpression
    InExpression
```

```pegs
EqualExpression:
    ShiftExpression == ShiftExpression
    ShiftExpression != ShiftExpression
```

```pegs
IdentityExpression:
    ShiftExpression is ShiftExpression
    ShiftExpression !is ShiftExpression
```

```pegs
RelExpression:
    ShiftExpression < ShiftExpression
    ShiftExpression <= ShiftExpression
    ShiftExpression > ShiftExpression
    ShiftExpression >= ShiftExpression
    ShiftExpression !<>= ShiftExpression
    ShiftExpression !<> ShiftExpression
    ShiftExpression <> ShiftExpression
    ShiftExpression <>= ShiftExpression
    ShiftExpression !> ShiftExpression
    ShiftExpression !>= ShiftExpression
    ShiftExpression !< ShiftExpression
    ShiftExpression !<= ShiftExpression
```

```pegs
InExpression:
    ShiftExpression in ShiftExpression
    ShiftExpression !in ShiftExpression
```

```pegs
ShiftExpression:
    AddExpression
    ShiftExpression << AddExpression
    ShiftExpression >> AddExpression
    ShiftExpression >>> AddExpression
```

```pegs
AddExpression:
    MulExpression
    AddExpression + MulExpression
    AddExpression - MulExpression
    CatExpression
```

```pegs
CatExpression:
    AddExpression ~ MulExpression
```

```pegs
MulExpression:
    UnaryExpression
    MulExpression * UnaryExpression
    MulExpression / UnaryExpression
    MulExpression % UnaryExpression
```

```pegs
UnaryExpression:
    & UnaryExpression
    ++ UnaryExpression
    -- UnaryExpression
    * UnaryExpression
    - UnaryExpression
    + UnaryExpression
    ! UnaryExpression
    ComplementExpression
    ( Type ) . Identifier
    ( Type ) . TemplateInstance
    DeleteExpression
    CastExpression
    PowExpression
```

```pegs
ComplementExpression:
    ~ UnaryExpression
```

```pegs
NewExpression:
    new AllocatorArgumentsopt Type
    NewExpressionWithArgs

NewExpressionWithArgs:
    new AllocatorArgumentsopt Type [ AssignExpression ]
    new AllocatorArgumentsopt Type ( ArgumentListopt )
    NewAnonClassExpression

AllocatorArguments:
    ( ArgumentListopt )

ArgumentList:
    AssignExpression
    AssignExpression ,
    AssignExpression , ArgumentList
```

```pegs
NewAnonClassExpression:
    new AllocatorArgumentsopt class ClassArgumentsopt SuperClassopt Interfacesopt AggregateBody

ClassArguments:
    ( ArgumentListopt )
```

```pegs
DeleteExpression:
    delete UnaryExpression
```

```pegs
CastExpression:
    cast ( Type ) UnaryExpression
    cast ( TypeCtorsopt ) UnaryExpression
```

```pegs
PowExpression:
    PostfixExpression
    PostfixExpression ^^ UnaryExpression
```

```pegs
PostfixExpression:
    PrimaryExpression
    PostfixExpression . Identifier
    PostfixExpression . TemplateInstance
    PostfixExpression . NewExpression
    PostfixExpression ++
    PostfixExpression --
    PostfixExpression ( ArgumentListopt )
    TypeCtorsopt BasicType ( ArgumentListopt )
    IndexExpression
    SliceExpression
```

```pegs
IndexExpression:
    PostfixExpression [ ArgumentList ]
```

```pegs
SliceExpression:
    PostfixExpression [ ]
    PostfixExpression [ Slice ,opt ]
Slice:
    AssignExpression
    AssignExpression , Slice
    AssignExpression .. AssignExpression
    AssignExpression .. AssignExpression , Slice
```

```pegs
PrimaryExpression:
    Identifier
    . Identifier
    TemplateInstance
    . TemplateInstance
    this
    super
    null
    true
    false
    $
    IntegerLiteral
    FloatLiteral
    CharacterLiteral
    StringLiterals
    ArrayLiteral
    AssocArrayLiteral
    FunctionLiteral
    AssertExpression
    MixinExpression
    ImportExpression
    NewExpressionWithArgs
    BasicTypeX . Identifier
    Typeof
    TypeidExpression
    IsExpression
    ( Expression )
    TraitsExpression
    SpecialKeyword
```

```pegs
StringLiterals:
    StringLiteral
    StringLiterals StringLiteral
```

```pegs
ArrayLiteral:
    [ ArgumentListopt ]
```

```pegs
AssocArrayLiteral:
    [ KeyValuePairs ]

KeyValuePairs:
    KeyValuePair
    KeyValuePair , KeyValuePairs

KeyValuePair:
    KeyExpression : ValueExpression

KeyExpression:
    AssignExpression

ValueExpression:
    AssignExpression
```

```pegs
FunctionLiteral:
    function Typeopt ParameterAttributes opt FunctionLiteralBody
    delegate Typeopt ParameterMemberAttributes opt FunctionLiteralBody
    ParameterMemberAttributes FunctionLiteralBody
    FunctionLiteralBody
    Lambda
```

```pegs
ParameterAttributes:
    Parameters FunctionAttributesopt

ParameterMemberAttributes:
    Parameters MemberFunctionAttributesopt

FunctionLiteralBody:
    BlockStatement
    FunctionContractsopt BodyStatement

Lambda:
    function Typeopt ParameterAttributes => AssignExpression
    delegate Typeopt ParameterMemberAttributes => AssignExpression
    ParameterMemberAttributes => AssignExpression
    Identifier => AssignExpression
```

```pegs
AssertExpression:
    assert ( AssignExpression ,opt )
    assert ( AssignExpression , AssignExpression ,opt )
```

```pegs
MixinExpression:
    mixin ( AssignExpression )
```

```pegs
ImportExpression:
    import ( AssignExpression )
```

```pegs
TypeidExpression:
    typeid ( Type )
    typeid ( Expression )
```

```pegs
IsExpression:
    is ( Type )
    is ( Type : TypeSpecialization )
    is ( Type == TypeSpecialization )
    is ( Type : TypeSpecialization , TemplateParameterList )
    is ( Type == TypeSpecialization , TemplateParameterList )
    is ( Type Identifier )
    is ( Type Identifier : TypeSpecialization )
    is ( Type Identifier == TypeSpecialization )
    is ( Type Identifier : TypeSpecialization , TemplateParameterList )
    is ( Type Identifier == TypeSpecialization , TemplateParameterList )

TypeSpecialization:
    Type
    struct
    union
    class
    interface
    enum
    function
    delegate
    super
    const
    immutable
    inout
    shared
    return
    __parameters
```

```pegs
TraitsExpression:
    __traits ( TraitsKeyword , TraitsArguments )

TraitsKeyword:
    isAbstractClass
    isArithmetic
    isAssociativeArray
    isFinalClass
    isPOD
    isNested
    isFloating
    isIntegral
    isScalar
    isStaticArray
    isUnsigned
    isVirtualFunction
    isVirtualMethod
    isAbstractFunction
    isFinalFunction
    isStaticFunction
    isOverrideFunction
    isTemplate
    isRef
    isOut
    isLazy
    hasMember
    identifier
    getAliasThis
    getAttributes
    getFunctionAttributes
    getFunctionVariadicStyle
    getLinkage
    getMember
    getOverloads
    getParameterStorageClasses
    getPointerBitmap
    getProtection
    getVirtualFunctions
    getVirtualMethods
    getUnitTests
    parent
    classInstanceSize
    getVirtualIndex
    allMembers
    derivedMembers
    isSame
    compiles

TraitsArguments:
    TraitsArgument
    TraitsArgument , TraitsArguments

TraitsArgument:
    AssignExpression
    Type
```

```pegs
TraitsExpression:
    __traits ( TraitsKeyword , TraitsArguments )

TraitsKeyword:
    isAbstractClass
    isArithmetic
    isAssociativeArray
    isFinalClass
    isPOD
    isNested
    isFloating
    isIntegral
    isScalar
    isStaticArray
    isUnsigned
    isVirtualFunction
    isVirtualMethod
    isAbstractFunction
    isFinalFunction
    isStaticFunction
    isOverrideFunction
    isTemplate
    isRef
    isOut
    isLazy
    hasMember
    identifier
    getAliasThis
    getAttributes
    getFunctionAttributes
    getFunctionVariadicStyle
    getLinkage
    getMember
    getOverloads
    getParameterStorageClasses
    getPointerBitmap
    getProtection
    getVirtualFunctions
    getVirtualMethods
    getUnitTests
    parent
    classInstanceSize
    getVirtualIndex
    allMembers
    derivedMembers
    isSame
    compiles

TraitsArguments:
    TraitsArgument
    TraitsArgument , TraitsArguments

TraitsArgument:
    AssignExpression
    Type
```
