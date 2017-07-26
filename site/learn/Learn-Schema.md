---
title: 视图与类型(Schemas and Types)
layout: ../_core/DocsLayout
category: Learn
permalink: /learn/schema/
next: /learn/validation/
sublinks: Type System,Type Language,Object Types and Fields,Arguments,The Query and Mutation Types,Scalar Types,Enumeration Types,Lists and Non-Null,Interfaces,Union Types,Input Types
---

通过本章，您将了解有关GraphQL类型系统的所有知识，以及如何描述可查询的数据。由于GraphQL可以与任何后端框架或编程语言一起使用，因此我们将不涉及具体的实现细节，只讨论概念。


### 类型系统(Type system)

如果之前接触过 GraphQL，您就会发现 GraphQL 查询语言是基于对象的字段查询。例如，以下的查询：

```graphql
# { "graphiql": true }
{
  hero {
    name
    appearsIn
  }
}
```

1. 以一个特殊的"root"对象开始
2. 选择该对象的 `hero` 字段
3. 在 `hero` 子对象上，选择 `name` 和 `appearsIn` 字段

因为 GraphQL 查询结构与结果是精确匹配的，所以不必过多了解服务端，你就可以预测查询结果。但是，这就要求我们准确的描述需要查询的数据 - 哪些字段可以查询？它们将返回哪种类型的对象？在子对象中有哪些字段可用？这也就是视图（schema）所在。

每个 GraphQL 服务都定义了一组能够完整描述可查询数据的类型。然后，当开始查询，它们将根据视图（schema）去校验和执行。

### 类型语言（Type language）

GraphQL 服务兼容任何语言。因为我们不能依赖于任何特定编程语言（如JavaScript）的语法去描述 GraphQL 视图（schema），所以我们定义了自己的简单语言 - “GraphQL 视图语言”，它类似于查询语言，并允许我们以语言无关的方式去描述GraphQL视图。

### 对象类型与字段（Object types and fields）

GraphQL 视图最基本的组件是对象类型(Object types)，它表示你可以从服务中查询的一类对象，以及包含的字段。在 GraphQL视图语言中，可以用如下代码来描述：

```graphql
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

该语言具有高度可读性，让我一起看看，以便我们了解通用名词：

- `Character` 是一个 _GraphQL对象类型_ ，表示包含多个字段的一种类型。视图中大多数类型都是对象类型。
- `name` 和 `appearsIn` 是 `Character` 类型的 _字段_ 。那就意味着 `name` 和 `appearsIn` 是任何基于 `Character` 类型的 GraphQL 查询和操作的唯二字段。
- `String` 是内置的标量类型之一 —— 这些类型可以解析为单个标量对象，在查询中不能有子查询。稍后我们会讲标量对象。
- `String!` 表示字段 _不可为空_ ，也就是说当你查询该字段时，GraphQL 服务总会返回一个值。在类型语言中，我们用感叹号标识。
- `[Episode]!` 表示是一组 `Episode` 对象。因为它也是 _不可为空_ 的，当你查询  `appearsIn` 字段时，总是会返回一个数组（[]或者[item1, item2...]）。

现在，你已经了解 GraphQL 对象类型，以及GraphQL类型语言的基础知识。

### 参数（Arguments）

每个 GraphQL 对象类型的字段都可以有零个或多个参数，例如下面的 `length` 字段：

```graphql
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

所有的参数都是被命名的。不像 JavaScript 和 Python 中函数定义有序参数列表，GraphQL 中所有参数都是按照特定的名称传递的。也就是说，`length` 字段有一个定义的参数 `unit`。

参数可以定义为必需或者可选。当一个参数是为可选的，我们可以定义一个 _默认值_ —— 当参数 `unit` 没有传递，它将被默认设置为 `METER`。

### 查询与突变类型（The Query and Mutation types）

虽然视图中大多数类型都为普通对象类型，但是视图中依然存在两种特殊类型：

```graphql
schema {
  query: Query
  mutation: Mutation
}
```

每个 GraphQL 服务都有一个 `查询` 类型，可能有也可能没有 `突变` 类型。虽然它们和普通的对象类型相类，但它们特殊之处在定义了每个 GraphQL 查询的 _入口点_。所以你会看到类似于这样的查询：

```graphql
# { "graphiql": true }
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

这意味着 GraphQL 服务需要有一个包含 `hero` 和 `droid` 字段的 `查询` 类型：

```graphql
type Query {
  hero(episode: Episode): Character
  droid(id: ID!): Droid
}
```

`突变` 类型与之类似。当你定义一个字段为 `突变` 类型，这些字段可以在你的查询中作为根 突变字段来调用。

重要的是要记住，除了作为视图的“入口点”之外，`查询` 和 `突变` 类型与任何其他GraphQL对象类型相同，字段使用也无不同。

### 标量类型（Scalar types）

每个 GraphQL 对象类型有一个名称和若干字段，但与此同时，这些字段也可以再解析为具体的数据。标量类型就是由此而来：它们代表查询的叶子结点。

在下面的查询中，`name` 和 `appearsIn` 将被解析为标量类型：

```graphql
# { "graphiql": true }
{
  hero {
    name
    appearsIn
  }
}
```

我们得出这样的结论是因为这些字段没有任何子字段，即它们是查询的叶子结点。

GraphQL 另外还定义了一组默认标量类型：

- `Int`: 有符号的32位整数。
- `Float`: 有符号的双精度浮点值。
- `String`: UTF‐8 字符序列。
- `Boolean`: `true` 或 `false`。
- `ID`: ID标量类型表示唯一标识符, 通常用于重写对象或缓存的主键. ID 类型与 String 以同样的方式序列化；然而，将其定义为 `ID` 是表示它不是人类可读（human-readable）的.

在大多的 GraphQL 服务实现中，还有一种方法来自定义标量类型。例如，我们可以定义一个 `日期` 类型：

```graphql
scalar Date
```

然后由 `实现` 来定义该类型应该如何序列化，反序列化以及验证。例如，可以指定 `日期` 类型总是被序列化成一个整数型时间戳，并且客户端中任何日期字段都期待这种格式。

### 枚举类型（Enumeration types）

也称 _枚举_ ，枚举类型是一种特殊的标量，它只限于一组列举值。这允许您：

1. 验证此类型的任何参数只能是列举值的一个
2. 在类型系统中，该字段总是一组有限的枚举值的一个

在 GraphQL 视图语言中，一个枚举类型的定义可能像这样：

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

这意味着，无论在 `视图` 中什么地方使用 `Episode` 类型，我们都期待它是 `NEWHOPE`、`EMPIRE` 或 `JEDI` 的一种。

请注意，各种语言的GraphQL服务实现会以具有自己的语言特定方式来处理枚举。支持枚举作为一等公民的语言，实现上可能会利用这点；而像 JavaScript 这种不支持枚举的语言，这些值可能在内部映射为一组整数。但是，这些细节不会泄露到客户端，客户端完全可以按照枚举值的字符串名称进行操作。

### 列表与非空（Lists and Non-Null）

对象类型，标量，和枚举是你在GraphQL中唯一可以定义的类型。但是如果你在视图中其他部分或者在查询变量声明中使用类型，可以添加 _类型修饰符_ 校验这些值。我们来看一个例子：

```graphql
type Character {
  name: String!
  appearsIn: [Episode]!
}
```

在这里，我们使用 `String` 类型并且在类型名后添加感叹号来标识它是 _非空_ 。这意味着服务端总是为这个字段返回一个非空的值，如果它最终获得一个空值，将会触发 QraphQL 执行错误，通知客户端出错了。

当定义字段的参数时，也可以使用非空类型修饰符，无论是在GraphQL字符串还是变量中，如果将空值作为该参数传递，则GraphQL服务器将返回验证错误。

```graphql
# { "graphiql": true, "variables": { "id": null } }
query DroidById($id: ID!) {
  droid(id: $id) {
    name
  }
}
```

列表与之类似：我们可以使用类型修饰符将一个类型标记为 `列表`，这表示此字段将返回该类型的数组。在视图语言中，通过将类型包裹在方括号`[` 和 `]`中来表示。对于参数也一样，验证步骤将期望该值是一个数组。

非空和列表修饰符可以组合使用。例如，您可以定义非空字符串的列表：

```graphql
myField: [String!]
```

这意味着 _列表本身_ 可以为空，但不能有任何空的成员。例如，在JSON中：

```js
myField: null // valid
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // error
```

现在，我们定义一个字符串的非空列表：

```graphql
myField: [String]!
```

这意味着列表本身不能为空，但它可以包含空值：

```js
myField: null // error
myField: [] // valid
myField: ['a', 'b'] // valid
myField: ['a', null, 'b'] // valid
```

您可以根据需要随意嵌套任何数量的非空和列表修饰符。

### 接口（Interfaces）

像许多类型的系统一样，GraphQL 也支持接口。_接口_ 是一个抽象的类型，包含一组特定的包含接口实现的字段。

例如，您定义一个 `Character` 接口，代表星球大战三部曲的任何角色：

```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

这意味着任何实现 `Character` 的类型都需要具有这些字段，带有这些参数和返回类型。

例如，以下是 `Character` 的些许实现：

```graphql
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

您可以看到这两种类型都具有接口 `Character` 的所有字段，但也包含额外的字段，`totalCredits`, `starships` 和 `primaryFunction`，这些是特定角色类型所特有的。

如果您想要返回一个对象或一组对象，但这些对象可能有多种不同的类型时，接口会很有用。

例如，请注意以下查询会产生错误：

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}
```

`hero` 字段返回 `Character` 类型，这意味着取决于 `episode` 参数，它可能是一个 `Human` 或一个 `Droid`。在上面的查询中，您只能请求接口 `Character` 中存在的字段，这并不包括 `primaryFunction`。

要请求特定对象类型的字段，您需要使用内联碎片（inline fragment）:

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI" } }
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

可以查询指南中的[内联碎片（inline fragments）](/learn/queries/#inline-fragments)部分了解更多信息。

### 复合类型（Union types）

复合类型和接口非常相似，但是它们不能在类型之间指定任何公共字段。

```graphql
union SearchResult = Human | Droid | Starship
```

无论我们在模式中任何地方返回一个 `SearchResult` 类型，我们都将会得到 `Human`、`Droid`或 `Starship`。 注意，复合类型的成员需要是具体的对象类型；你不能用接口或者其他复合类型创建一个复合类型。

既然如此，如果你查询一个返回 `SearchResult` 复合类型的字段，则需要使用条件碎片才能查询字段：

```graphql
# { "graphiql": true}
{
  search(text: "an") {
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

### 输入类型（Input types）

到目前为止，我们只谈到将标量值（例如枚举或字符串）作为参数传递给一个字段。但您也可以轻松地传递复杂的对象。这在突变（mutations）, 您可能希望传入要创建的整个对象时，特别有用。在 GraphQL 视图语言中，输入类型与常规对象类型完全相同，仅是使用关键字input而不是type：

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}
```

以下是如何在数据突变（mutation）中使用输入对象类型：

```graphql
# { "graphiql": true, "variables": { "ep": "JEDI", "review": { "stars": 5, "commentary": "This is a great movie!" } } }
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

输入对象类型的字段本身也可以引用输入对象类型，但您不能在视图中混合使用输入和输出类型。输入对象类型也不能在其字段上有参数。
