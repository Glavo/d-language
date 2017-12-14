# 3. 语法

## 3.1 词法

1. 请参考页面 [词法](https://glavo.gitbooks.io/d-language/lex.html)。

## 3.2 类型

```pegs
Type:
    TypeCtorkios opt BasicType BasicType2 opt

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
    BasicType2X BasicType2 opt

BasicType2X:
    *
    [ ]
    [ AssignExpression ]
    [ AssignExpression .. AssignExpression ]
    [ Type ]
    delegate Parameters MemberFunctionAttributes opt
    function Parameters FunctionAttributes opt

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
    new AllocatorArguments opt Type
    NewExpressionWithArgs

NewExpressionWithArgs:
    new AllocatorArguments opt Type [ AssignExpression ]
    new AllocatorArguments opt Type ( ArgumentList opt )
    NewAnonClassExpression

AllocatorArguments:
    ( ArgumentList opt )

ArgumentList:
    AssignExpression
    AssignExpression ,
    AssignExpression , ArgumentList
```

```pegs
NewAnonClassExpression:
    new AllocatorArguments opt class ClassArguments opt SuperClass opt Interfaces opt AggregateBody

ClassArguments:
    ( ArgumentList opt )
```

```pegs
DeleteExpression:
    delete UnaryExpression
```

```pegs
CastExpression:
    cast ( Type ) UnaryExpression
    cast ( TypeCtors opt ) UnaryExpression
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
    PostfixExpression ( ArgumentList opt )
    TypeCtors opt BasicType ( ArgumentList opt )
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
    [ ArgumentList opt ]
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
    function Type opt ParameterAttributes opt FunctionLiteralBody
    delegate Type opt ParameterMemberAttributes opt FunctionLiteralBody
    ParameterMemberAttributes FunctionLiteralBody
    FunctionLiteralBody
    Lambda
```

```pegs
ParameterAttributes:
    Parameters FunctionAttributes opt

ParameterMemberAttributes:
    Parameters MemberFunctionAttributes opt

FunctionLiteralBody:
    BlockStatement
    FunctionContracts opt BodyStatement

Lambda:
    function Type opt ParameterAttributes => AssignExpression
    delegate Type opt ParameterMemberAttributes => AssignExpression
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

```pegs
SpecialKeyword:
    __FILE__
    __FILE_FULL_PATH__
    __MODULE__
    __LINE__
    __FUNCTION__
    __PRETTY_FUNCTION__
```

## 3.4 语句

```pegs
Statement:
    ;
    NonEmptyStatement
    ScopeBlockStatement

NoScopeNonEmptyStatement:
    NonEmptyStatement
    BlockStatement

NoScopeStatement:
    ;
    NonEmptyStatement
    BlockStatement

NonEmptyOrScopeBlockStatement:
    NonEmptyStatement
    ScopeBlockStatement

NonEmptyStatement:
    NonEmptyStatementNoCaseNoDefault
    CaseStatement
    CaseRangeStatement
    DefaultStatement

NonEmptyStatementNoCaseNoDefault:
    LabeledStatement
    ExpressionStatement
    DeclarationStatement
    IfStatement
    WhileStatement
    DoStatement
    ForStatement
    ForeachStatement
    SwitchStatement
    FinalSwitchStatement
    ContinueStatement
    BreakStatement
    ReturnStatement
    GotoStatement
    WithStatement
    SynchronizedStatement
    TryStatement
    ScopeGuardStatement
    ThrowStatement
    AsmStatement
    PragmaStatement
    MixinStatement
    ForeachRangeStatement
    ConditionalStatement
    StaticForeachStatement
    StaticAssert
    TemplateMixin
    ImportDeclaration
```

```pegs
ScopeStatement:
    NonEmptyStatement
    BlockStatement
```

```pegs
ScopeBlockStatement:
    BlockStatement
```

```pegs
LabeledStatement:
    Identifier :
    Identifier : NoScopeStatement
    Identifier : Statement
```

```pegs
BlockStatement:
    { }
    { StatementList }

StatementList:
    Statement
    Statement StatementList
```

```pegs
ExpressionStatement:
    Expression ;
```      

```pegs
DeclarationStatement:
    StorageClasses opt Declaration
```

```pegs
IfStatement:
    if ( IfCondition ) ThenStatement
    if ( IfCondition ) ThenStatement else ElseStatement

IfCondition:
    Expression
    auto Identifier = Expression
    TypeCtors Identifier = Expression
    TypeCtorsopt BasicType Declarator = Expression

ThenStatement:
    ScopeStatement

ElseStatement:
    ScopeStatement
```

```pegs
WhileStatement:
    while ( Expression ) ScopeStatement
```

```pegs
DoStatement:
    do ScopeStatement  while ( Expression ) ;
```    

```pegs
ForStatement:
    for ( Initialize Testopt ; Incrementopt ) ScopeStatement

Initialize:
    ;
    NoScopeNonEmptyStatement

Test:
    Expression

Increment:
    Expression
```

```pegs
AggregateForeach:
    Foreach ( ForeachTypeList ; ForeachAggregate )

ForeachStatement:
    AggregateForeach NoScopeNonEmptyStatement

Foreach:
    foreach
    foreach_reverse

ForeachTypeList:
    ForeachType
    ForeachType , ForeachTypeList

ForeachType:
    ForeachTypeAttributesopt BasicType Declarator
    ForeachTypeAttributesopt Identifier
    ForeachTypeAttributesopt alias Identifier

ForeachTypeAttributes
    ForeachTypeAttribute
    ForeachTypeAttribute ForeachTypeAttributesopt

ForeachTypeAttribute:
    ref
    TypeCtor
    enum

ForeachAggregate:
    Expression
```

```pegs
RangeForeach:
    Foreach ( ForeachType ; LwrExpression .. UprExpression )

LwrExpression:
    Expression

UprExpression:
    Expression

ForeachRangeStatement:
    RangeForeach ScopeStatement
```

```pegs
SwitchStatement:
    switch ( Expression ) ScopeStatement

CaseStatement:
    case ArgumentList : ScopeStatementList

CaseRangeStatement:
    case FirstExp : .. case LastExp : ScopeStatementList

FirstExp:
    AssignExpression

LastExp:
    AssignExpression

DefaultStatement:
    default : ScopeStatementList

ScopeStatementList:
    StatementListNoCaseNoDefault

StatementListNoCaseNoDefault:
    StatementNoCaseNoDefault
    StatementNoCaseNoDefault StatementListNoCaseNoDefault

StatementNoCaseNoDefault:
    ;
    NonEmptyStatementNoCaseNoDefault
    ScopeBlockStatement
```
