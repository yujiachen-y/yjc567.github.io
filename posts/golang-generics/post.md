

泛型是一种编程范式，允许在编写代码时使用通用的类型，从而提高代码的复用性和灵活性。在许多编程语言（如 Java、C++ 和 C#）中，泛型已经成为一种非常重要的特性，使得开发者能够编写更加通用、灵活且类型安全的代码。然而，Go 语言在最初的设计中并没有包含泛型特性，这导致了一些局限性和不便。随着 Go 语言的发展，为了使其更加完善，并满足开发者在实际工作中的需求，Go 语言社区逐渐引入了泛型特性。通过引入泛型，Go 语言能够实现更加通用的数据结构和函数，从而提高开发效率和代码质量。

举个例子：

```go
import "golang.org/x/exp/constraints"

func GMin[T constraints.Ordered](x, y T) T {
    if x < y {
        return x
    }
    return y
}

```

不过 Go 泛型和其他语言的泛型略有不同，本文从 Go 的类型参数提案出发，结合实现代码和示例，说明 Go 泛型的一些特性，期望可以达到让大家在工作中更好更高效使用泛型的目的。

> 主要参考：[https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md)

# 名词解释

- parameter，也被称作形参，指函数定义时指定需要传入的参数，实际上是一个占位符，没有真实值，例如 `func Add(a, b int) int` 中的 `a` 和 `b` 。下文用类型参数指代，例如 `func Add[T any](a, b T) T` 中的 `T` 就是 type parameter 类型参数。
- argument，也被称作实参，指函数被调用时实际传入的参数，是一个真实值，例如 `sum := Add[int](a, 2)` 中的 `a` 和 `2`，此示例中 `a` 是一个命名变量或常量，2 是一个未命名常量。下文用类型实参，例子中 `Add[int]` 中的 `int` 就是 type argument 类型实参。
- function，指 Golang 中的函数，如 `func Add(a, b int) int`。[https://go.dev/ref/spec#Function_types](https://go.dev/ref/spec#Function_types)
- method，指 Golang 中结构体的方法，如 `func (s *Struct) Get() string`。[https://go.dev/ref/spec#Method_declarations](https://go.dev/ref/spec#Method_declarations)
- operation，可以理解为 Golang 中内置类型支持的运算符（见下），Go spec 中使用 operator 这个名词。[https://go.dev/ref/spec#Operators](https://go.dev/ref/spec#Operators)

```
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .

```

# 总览

Go 类型参数提案归纳了 Go 类型参数的几个要点（[https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#summary）：](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#summary%EF%BC%89%EF%BC%9A)

- 函数和类型可以有类型参数，类型参数由类型约束定义，类型约束的类型为 interface。(注意，这里没有提及方法，也就是说方法不可以有类型参数）
- 类型约束描述了 type argument，即传入的实参需要实现的方法，和允许传入实参的类型。
- 类型约束描述了 type parameter，即定义的形参支持的方法和运算符。
- 当调用支持类型参数的函数时，类型推断有时可以推断出类型实参，此时不需要显示传入类型实参。（即调用 `func Add[T any](a, b T) T`时，我们通常可以直接使用`sum := Add(a, 2)`，而不需要使用`sum := Add[int](a, 2)`。

接下来我们从这几个角度来说明 Go 泛型的设计和使用：

- 类型约束的具体定义
- 类型推断是如何实现的
- 一些其他相关知识点
- 一些类型推断的代码节选

# 类型约束

Go 类型约束可以看作是类型参数的“类型” ，例如下面的例子中，`T`是类型参数，而`Stringer`则是类型约束，即`T`的类型。

```go
func Stringify[T Stringer](s []T) (ret []string) {
        for _, v := range s {
                ret = append(ret, v.String())
        }
        return ret
}

```

类型约束自身的类型为 interface，在 Go 1.8 之前，interface 是方法的集合，如上面的例子中，`Stringer`描述了 T 应该是具有`func (T) String() string`这个方法的类型。但如果 interface 只能作为方法的集合，下面的函数则无法实现：

```go
func Add[T constraints.Integer](a, b T) T {
        return a + b
}

```

试问，`constraints.Integer`作为一个 interface 如何表述其支持`+`这个操作呢？实际上 [constraints](https://pkg.go.dev/golang.org/x/exp/constraints) 包中的定义如下：

```go
type Integer interface {
        Signed | Unsigned
}

type Signed interface {
        ~int | ~int8 | ~int16 | ~int32 | ~int64
}

type Unsigned interface {
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
}

```

可以看到，`Integer` interface 并没有定义任何方法，这里就涉及到接下来说到的 underlying type。

## Underlying type

> 参考：[https://go.dev/ref/spec#Underlying_types](https://go.dev/ref/spec#Underlying_types)

范型为了使类型约束的表达更加宽泛，把 interface 的定义从“方法的集合”扩展到了“类型的集合”，两者主要的区别是内置运算符的不同。在 Go 中，`<`, `==` 等内置运算符是不支持重置的（Go 本身就没有 override 的概念），只有内置类型可以拥有这些运算符，自定义类型是无法拥有和定义这类运算符的。原先 interface 作为“方法的集合”，无法表达出一个类型是否支持比较相等等内置运算（因为内置运算不能被定义）。为了解决这个问题， Go 引入了 underlying type 并规定凡是 underlying type 支持内置运算符的类型，都能复用其 underlying type 的内置运算。interface 也不仅仅可以用来表达方法集合，而是可以用 ~ underlying type 等语法表示对应的类型集合。

一个类型的 underlying type 的判定规则如下：

1. 自身就是underlying type的类型
    1. boolean
    2. numeric
    3. string
    4. type literal
        1. ArrayType
        2. StructType
        3. PointerType
        4. FunctionType
        5. InterfaceType
        6. SliceType
        7. MapType
        8. ChannelType
2. 不符合以上的情况时，一个类型的underlying type是其被创建时所指向的类型的underlying type。比如：

```go
type (
    A1 = string
    A2 = A1
)

```

这里A1符合规则1，其underlying type是string，A2符合规则2，其underlying type是string。

回看上面`constraints.Integer`的定义，其表达的含义是所有 underlying type 为`int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64` 或 `uintptr`的类型集合。而这些类型都支持`+`操作，故 `Add`函数可以被定义出。

## 使用 type parameter 和直接使用 interface 的区别

> 参考：[https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#values-of-type-parameters-are-not-boxed](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#values-of-type-parameters-are-not-boxed)

Go 的泛型由于类型约束的存在，看上去会和 interface 类似，在这里与 interface 的主要不同是相关函数的返回值可以是具体的类型，而不只是 interface 。

用上面的 `Add` 举例子，删除其类型参数则变成：

```go
func Add(a, b interface{}) interface{} {
        n, _ := a.(int)
        m, _ := b.(int)
        return n + m
}

```

如果 `Add`使用上述实现，调用方得到的返回值只能是 `interface` 或某一个特定的类型，而不能根据调用方的需求返回特定类型。如果返回的是`interface`，调用方想得到返回值的具体类型，还需要进行拆箱 (unboxing) 操作，如：

```go
func main() {
        a, b := 1, 2
        c := Add(a, b)
        d, ok := c.(int)
        ...
}

```

同时，回忆一下 Go 中 interface 实例是如何构造的，一个interface里包含了两个值，代表其指向元素的type和value，构造 interface 实例，即装箱 (boxing) ，这会带来额外的内存开销，泛型可以避免这个内存开销。

# 类型推断

并不是每次调用支持泛型的函数都需要传入类型参数的，在一些条件下，类型推断可以推断出缺失的类型参数，这个技术被叫做类型推断。注意类型推断并不验证推断出的类型是否能通过编译，该检查在推断结束后进行。

接下来会介绍类型推断的 3 个知识点，并在最后介绍完整的类型推断流程。

## 类型归一化 ( Type unification )

> 参考：[https://go.dev/ref/spec#Type_unification](https://go.dev/ref/spec#Type_unification)
> 
> 这个名词我瞎翻译的，ChatGPT 建议翻译这个为“类型统一”

### 功能描述

**输入**

1. 映射关系 P -> A，P 代表类型参数，A 代表已知的类型实参。例如 `func Add[T any](a, b T) T`一个可能的映射关系是 T -> int。
2. 两个类型，这两个类型可以带有类型参数，也可以不带。

**输出**

1. 基于已知的映射关系，判断输入的两个类型是否有可能相等。

### 具体操作

1. 不带有类型参数的类型，与比较类型必须等价，否则归一失败。
    1. 两个类型相等自然等价。
    2. 如果两个类型是 channel 类型，忽略 channel 方向后类型相等，也可以判定为等价。
    3. 如果两个类型的 underlying types 是一致的，那也可以判定为等价。
2. 带有类型参数的类型，在不考虑类型参数的前提下，该类型与比较类型在结构上必须相等，否则归一失败。
    1. 例如 `[]map[T1]T2` 和 `[]T3` 在结构上是一致的，`T3` 可以被替换成 `map[T1]T2`，同理 `[]map[T1]bool` 和 `[]map[string]T2` 在结构上也是一致的。
    2. 例如 `[]map[T1]T2` 和 `int`, `struct{}`, `[]struct{}` 等类型在结构上不可能一致。
3. 如果匹配成功，且类型带有类型参数，便知道了新的 P' -> A' 映射关系，将新的映射关系加入到原有的映射关系中。

## 函数实参类型推断 ( Function argument type inference )

> 参考：
> 
> - [https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#function-argument-type-inference](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#function-argument-type-inference)
> - [https://go.dev/ref/spec#Function_argument_type_inference](https://go.dev/ref/spec#Function_argument_type_inference)

### 功能描述

1. 在调用有类型参数的函数时，若调用方没有传入类型参数，则根据实参推断出类型参数。

### 具体实现

1. 根据调用方传入实参得到一组 `(parameter, argument)`，即参数到实参的组合。
2. 先忽略 `argument` 中没有类型的组合，没有类型即为常量，常量有自身的类型推断规则；对有类型的`(parameter, argument)`组合，对其对应的类型进行类型归一化，并不断更新映射关系 P -> A。
3. 接下来处理常量的`(parameter, argument)`组合，如果一个组合中`parameter`对应的类型参数已经在步骤 3 中被推断出来，那么忽略；如果没有，认定常量`argument`的取值为该常量对应的默认类型，进行类型归一化。
4. 当所有`(parameter, argument)`都被处理完成后，推断结束，中途若发生处理失败的情况，则推断失败。

接下来用一个示例描述上述步骤：

```go
func scale[Number ~int64|~float64|~complex128](v []Number, s Number) []Number {
        ...
}

func main() {
        var vector []float64
        scaledVector := scale(vector, 42)
        ...
}

```

函数实参类型推断开始时，得到两个`(parameter, argument)`组合：

1. `(v []Number, vector []float64)`
2. `(s Number, 42)`

先对`(v []Number, vector []float64)`的类型进行类型归一化，得到映射关系`Number -> float64`。

因为上面已经推断出了映射关系`Number -> float64`，`(s Number, 42)`组合就不必进行类型归一化。

如果没有映射关系`Number -> float64`，`(s Number, 42)`中 42 的类型会被认定为默认类型 `int`，那么映射关系则为`Number -> int`。

## 约束类型推断 ( Constraint type inference )

> 参考：[https://go.dev/ref/spec#Constraint_type_inference](https://go.dev/ref/spec#Constraint_type_inference)

### 功能描述

1. 根据定义的类型参数约束，从一个已知类型参数推导出其他暂时未知的类型参数。
2. 例如有一个函数 `func Double[S ~[]E, E constraints.Integer] (s S) S` ，这个函数被这样调用 `Double([]int{1, 2, 3})`，可以从类型约束`S ~[]E`和`S -> []int` ，推断出`E -> int`。

### 具体实现

1. 遍历所有类型参数
    1. 如果类型参数已有对应的实参，对两者的 underlying type 进行归一化，如`Double`的例子中，`S`的 underlying type 是 `[]E`，则对`[]E`和已知实参`[]int`进行归一化，推断出`E -> int`。
    2. 如果类型参数没有对应的实参，但类型参数的类型约束只有一个类型，那么推断该类型参数对应的实参为约束类型。
2. 在已知的映射关系中，检查是否存在一组 P -> A 和 Q -> B 关系，其中 A 中包含了类型参数 Q，用 B 替换 A 中的 Q。例如`func Copy[T any, P *T](value T, dst P)`，已知 `T -> int`，`P -> *T`，那么可以推断出`P -> *int`。
3. 重复步骤 2，直到已知的映射关系中再也找不到一个类型参数 P 被某个类型实参 A 包含。

## 类型推断执行步骤

> 参考：[https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L33](https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L33)

根据代码注释我们可以知道具体执行步骤如下：

1. 利用类型实参进行函数实参类型推断。
2. 再进行一次约束类型推断。
3. 对剩下无类型实参进行函数实参类型推断。
4. 最后进行一次约束类型推断。

举一个例子：

```go
package main

import "fmt"
import "golang.org/x/exp/constraints"

func Multiple[S ~[]E, E, X constraints.Integer](s S, x X) S {
        for i, e := range s {
                s[i] *= x
        }
        return s
}

type IntVector []int

func main() {
        vector := IntVector{0, 1, 2, 3, 4}
        vector = Multiple(vector, 3)
        fmt.Printf("%s\\n", vector)
        // output: [0, 3, 6, 9, 12]
}

```

`Multiple`函数类型推断的步骤如下：

1. 对有类型的函数实参进行类型推断，即对`(s S, vector IntVector)`进行类型推断，得到：`S -> IntVector`。
2. 进行约束类型推断，`S`的约束为`[]E`，`IntVector`的underlying type 为`[]int`，则对`[]E, []int`进行类型归一化，得到`E -> int`。
3. 对无类型函数实参进行类型推断，即对`(x X, 3)`进行类型推断，对常量 `3`取默认值 `int`，得到`X -> int`。
4. 再次进行约束类型推断，但因为所有参数类型已知，提前结束。

# 其他

一些其他相关内容整理

## 泛型窘境

> 参考：[https://research.swtch.com/generic](https://research.swtch.com/generic)

在一个编程语言中添加泛型，势必会增加下列三方中至少一方的复杂度：

- 程序员，C 语言采用了这种方法，即只支持泛型语法，但编译器和运行时都不考虑泛型带来的问题，如有问题程序员自己排查。
- 编译，C++ 采用了这种方法，在编译时推断出类型，运行时泛型已不存在，这会增加编译耗时以及增加编译产物的数据大小。
- 运行时，Java 采用了这种方法，在运行时携带参数的类型信息，这会降低运行时的效率，针对 Java，这也引入了类型擦除问题（有兴趣可自行了解）。

由前文可以看出，Go 采取的是增加编译耗时，编译时就推断出了一个方法需要的全部类型信息。

## 使用 [T] 而不是 `<T>`

相信很多同学对泛型的第一印象都是 C++ 或 Java 中的 `<T>`语法，[https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#why-not-use-the-syntax-like-c_and-java](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#why-not-use-the-syntax-like-c_and-java) 中解释了 `<`和`>`两个符号因为也用做比较符号，区分`<`和`>`是在表示比较还是类型参数会带来额外的负担，故最后选择了 `[T]`

## 为何 Go 泛型不支持 method？

> 本段说明略跳跃和简短，更详细的说明和示例请参考：[https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#no-parameterized-methods](https://go.googlesource.com/proposal/+/HEAD/design/43651-type-parameters.md#no-parameterized-methods)

在 Go 中，结构体可以使用类型参数，但一个结构体的方法是不被允许使用类型参数的，最主要的原因是因为 Go interface 的特性所致。上文有提到，interface 可以表达“方法的集合”，即一个 interface 可以代表所有实现了其定义方法的结构体。假设说 method 支持泛型，那会出现如下的 interface 定义：

```go
type Phone interface {
        Call[N PhoneNumber](n N)
        Download[A App](a A)
}

```

同时考虑到 Go 的 interface 和结构体之间没有显式的关系定义，即不存在类似`struct iPhone extend Phone`这样的语法。因此确定一个 interface 下各个方法的类型参数，以及确定一个类型是否符合一个 interface 会需要大量的类型推断，会给带来非常大的工作量和编译损耗。

## 支持指针方法

当我们定义了一个泛型函数 `F[T C]`，相应类型参数的约束 `C`定义了一些方法，向这个函数传入类型 `c`时，如果`C`中定义的各种方法被定义在了`*c`类型而不是 `c`上，此时代码将无法编译。

举一个具体例子：

```go
type Setter interface {
        Set(string)
}

func FromStrings[T Setter](s []string) []T {
        result := make([]T, len(s))
        for i, v := range s {
                result[i].Set(v)
        }
        return result
}

type Settable int

func (p *Settable) Set(s string) {
        i, _ := strconv.Atoi(s) // real code should not ignore the error
        *p = Settable(i)
}

func F() {
        // INVALID
        nums := FromStrings[Settable]([]string{"1", "2"})
        // Here we want nums to be []Settable{1, 2}.
        ...
}

```

上面的例子中，`result`的类型为`[]Settable`，而`Settable`并不支持 `Set` 方法，支持`Set`方法的是`*Settable`，故`result[i].Set(v)`无法正常调用。

相应的解法如下：

```go
type Setter2[B any] interface {
        Set(string)
        *B // non-interface type constraint element
}

func FromStrings2[T any, PT Setter2[T]](s []string) []T {
        result := make([]T, len(s))
        for i, v := range s {
                // The type of &result[i] is *T which is in the type set
                // of Setter2, so we can convert it to PT.
                p := PT(&result[i])
                // PT has a Set method.
                p.Set(v)
        }
        return result
}

```

即在泛型函数中明确区分出类型`T`和其对应的指针类型 `PT`，并通过类型约束`Setter2[B any]`定义其之间的转化关系，调用`Set`时转换类型，调用成功。

## 在工作中使用泛型

以上，我们从 Go 的类型约束出发，了解了定义泛型函数需要的知识点，并了解了类型推断的关键点，最后我们讨论下在泛型在工作中的应用：

- 容器操作。
- 通用数据结构。
- 通用操作逻辑，自己日常维护的代码中有请求数据网关服务的操作，由于数据网关返回的数据是一个固定的结构体，使用相对比较麻烦，自己用泛型对请求和返回进行了封装，使不同数据源返回的数据可以转化为特定的结构体。

如果你在工作中有其他的泛型使用方法，或者有其他好用的泛型库推荐，欢迎评论补充 : )

# 类型推断代码

## 类型归一化

> 参考：[https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/unify.go](https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/unify.go)

不断递归判断`x, y Type`在映射关系`p * ifacePair`下是否有可能相等，如果发现`x`或`y`为没有推断出的类型参数，则匹配并返回相等。

```go
// nify implements the core unification algorithm which is an
// adapted version of Checker.identical. For changes to that
// code the corresponding changes should be made here.
// Must not be called directly from outside the unifier.
func (u *unifier) nify(x, y Type, p *ifacePair) (result bool) {

        ......

        // Cases where at least one of x or y is a type parameter.
        switch i, j := u.x.index(x), u.y.index(y); {
        case i >= 0 && j >= 0:
                // both x and y are type parameters
                if u.join(i, j) {
                        return true
                }
                // both x and y have an inferred type - they must match
                return u.nifyEq(u.x.at(i), u.y.at(j), p)

        case i >= 0:
                // x is a type parameter, y is not
                if tx := u.x.at(i); tx != nil {
                        return u.nifyEq(tx, y, p)
                }
                // otherwise, infer type from y
                u.x.set(i, y)
                return true

        case j >= 0:

                ......

        }

        ......

        switch x := x.(type) {

        ......

        case *Slice:
                // Two slice types are identical if they have identical element types.
                if y, ok := y.(*Slice); ok {
                        return u.nify(x.elem, y.elem, p)
                }

        case *Struct:
                // Two struct types are identical if they have the same sequence of fields,
                // and if corresponding fields have the same names, and identical types,
                // and identical tags. Two embedded fields are considered to have the same
                // name. Lower-case field names from different packages are always different.
                if y, ok := y.(*Struct); ok {
                        if x.NumFields() == y.NumFields() {
                                for i, f := range x.fields {
                                        g := y.fields[i]
                                        if f.embedded != g.embedded ||
                                                x.Tag(i) != y.Tag(i) ||
                                                !f.sameId(g.pkg, g.name) ||
                                                !u.nify(f.typ, g.typ, p) {
                                                return false
                                        }
                                }
                                return true
                        }
                }

        ......

        default:
                panic(sprintf(nil, true, "u.nify(%s, %s), u.x.tparams = %s", x, y, u.x.tparams))
        }

        return false
}

```

## 函数实参类型推断

### 有类型的实参直接归一化

> 参考：[https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L250](https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L250)

```go
        // indices of the generic parameters with untyped arguments - save for later
        var indices []int
        for i, arg := range args {
                par := params.At(i)
                // If we permit bidirectional unification, this conditional code needs to be
                // executed even if par.typ is not parameterized since the argument may be a
                // generic function (for which we want to infer its type arguments).
                if isParameterized(tparams, par.typ) {
                        if arg.mode == invalid {
                                // An error was reported earlier. Ignore this targ
                                // and continue, we may still be able to infer all
                                // targs resulting in fewer follow-on errors.
                                continue
                        }
                        if targ := arg.typ; isTyped(targ) {
                                // If we permit bidirectional unification, and targ is
                                // a generic function, we need to initialize u.y with
                                // the respective type parameters of targ.
                                if !u.unify(par.typ, targ) {
                                        errorf("type", par.typ, targ, arg)
                                        return nil
                                }
                        } else if _, ok := par.typ.(*TypeParam); ok {
                                // Since default types are all basic (i.e., non-composite) types, an
                                // untyped argument will never match a composite parameter type; the
                                // only parameter type it can possibly match against is a *TypeParam.
                                // Thus, for untyped arguments we only need to look at parameter types
                                // that are single type parameters.
                                indices = append(indices, i)
                        }
                }
        }

```

### 没有类型的实参赋予常量默认值后归一化

> 参考：[https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L297](https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L297)

```go
        // Use any untyped arguments to infer additional type arguments.
        // Some generic parameters with untyped arguments may have been given
        // a type by now, we can ignore them.
        for _, i := range indices {
                tpar := params.At(i).typ.(*TypeParam) // is type parameter by construction of indices
                // Only consider untyped arguments for which the corresponding type
                // parameter doesn't have an inferred type yet.
                if targs[tpar.index] == nil {
                        arg := args[i]
                        targ := Default(arg.typ)
                        // The default type for an untyped nil is untyped nil. We must not
                        // infer an untyped nil type as type parameter type. Ignore untyped
                        // nil by making sure all default argument types are typed.
                        if isTyped(targ) && !u.unify(tpar, targ) {
                                errorf("default type", tpar, targ, arg)
                                return nil
                        }
                }
        }

```

## 约束类型推断

> 参考：[https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L468](https://github.com/golang/go/blob/go1.18/src/cmd/compile/internal/types2/infer.go#L468)

### 类型参数 core type 处理

在约束类型推断的一阶段，引入了一个新概念 core type，本文不做过多介绍，可以理解为类型约束对应约束类型的 underlying type。利用 core type 与已知实参可以完成一些类型推断。

```go
                for i, tpar := range tparams {
                        // If there is a core term (i.e., a core type with tilde information)
                        // unify the type parameter with the core type.
                        if core, single := coreTerm(tpar); core != nil {
                                // A type parameter can be unified with its core type in two cases.
                                tx := u.x.at(i)
                                switch {
                                case tx != nil:

                                        ......

                                        if !u.unify(tx, core.typ) {
                                                // TODO(gri) improve error message by providing the type arguments
                                                //           which we know already
                                                // Don't use term.String() as it always qualifies types, even if they
                                                // are in the current package.
                                                tilde := ""
                                                if core.tilde {
                                                        tilde = "~"
                                                }
                                                check.errorf(pos, "%s does not match %s%s", tpar, tilde, core.typ)
                                                return nil, 0
                                        }

                                case single && !core.tilde:
                                        // The corresponding type argument tx is unknown and there's a single
                                        // specific type and no tilde.
                                        // In this case the type argument must be that single type; set it.
                                        u.x.set(i, core.typ)

                                default:
                                        // Unification is not possible and no progress was made.
                                        continue
                                }

                                ......

                        }
                }

```

### 映射关系化简

约束类型推断的二阶段，不断化简映射关系。

```go
                smap := makeSubstMap(tparams, types)
                n := 0
                for _, index := range dirty {
                        t0 := types[index]
                        if t1 := check.subst(nopos, t0, smap, nil); t1 != t0 {
                                types[index] = t1
                                dirty[n] = index
                                n++
                        }
                }

```
