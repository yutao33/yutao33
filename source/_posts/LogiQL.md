---
title: LogiQL 教程
date: 2019-04-22 17:12:40
mathjax: true
tags: [LogiQL]
---

参考 LogicBlox 官方教程 https://developer.logicblox.com/content/docs4/tutorial/repl/section/split.html

# 开始

官方的教程可以使用一个即时交互的解释器（REPL, Read-Eval-Print Loop）， 每次打开网页后都会解释器会刷新工作区，但是可以选择保存下载一个zip文件，再选择恢复上传备份文件

# 弗雷德的奥德冰淇淋店

教程提供了一个简单的冰淇淋店账目系统作为背景，在系统里需要记录

- 每种冰淇淋的成本 `cost (labor, ingredients)`
- 每种冰淇淋的价格 `price`
- 每周销售额 `sales`

弗雷德希望查询的信息有

- 每周每种冰淇淋的营收 `revenue`
- 每周所有冰淇淋营收 `week revenue`
- 每周每种冰淇淋的利润 `profit`
- 每周所有冰淇淋的利润  `week profit`

# 记录成本、价格和销售额

为了跟踪每种冰淇淋类型的成本，价格和销售情况，我们将定义一些谓词(predicates)。 具体来说，我们将定义功能谓词(functioanl predicates)。 谓词类似于SQL数据库中的表，功能谓词就像一个表，其中除最后一列之外的所有列都形成主键，最后一列表示该值，因此最多一个值可以与一个主键相关联。 我们首先定义一个`cost`谓词来保存某种冰淇淋的生产成本。 因此，有效成本定义了从冰淇淋类型名称到数字的映射。 我们将价格模型化为整数（以美分计）。 用于定义`cost`谓词的LogiQL如下：

```
cost[icecream] = c ->
   string(icecream), int(c).
```

这个定义说：存在一个名为`cost`的函数谓词，它有一个键：`icecream`和一个名为`c`的值。 指向右侧的箭头表示约束，约束表明如果箭头左侧的部分为真，则右侧的部分也必须保持。在这个例子中，这意味着`icecream`的值必须出现在内置`string`谓词中（即`icecream`的值必须是一个字符串），并且`c`必须出现在内置的`int`谓词中（即它必须具有整数值）。

但是，我们不能简单地将此LogiQL代码以此形式输入REPL。 REPL用于定义谓词的命令称为`addblock`。 `addblock`有一个参数：我们需要在引号之间添加的定义的LogiQL代码。

```
addblock 'cost[icecream] = c ->
   string(icecream), int(c).'
```

现在可以通过list命令查看系统里定义过的谓词

```
list
```

![](/images/LogiQL-c3.png)

# 处理事实(facts)

谓词里的数据项在LogiQL里被称为事实`facts`，为了插入一些`facts`，我们可以执行一些增量更新(`delta updates`)

- `insert` : 插入数据(以`+`号表示)
- `remove` : 移除数据(以`-`号表示)
- `upsert` : 更新数据，如果不存在则插入(以`^`表示，仅用于函数谓词)

我们将从插入一些事实开始。 在REPL中运行以下命令以输入柠檬冰棒的成本（23ct）：

```
exec '+cost["Popsicle Lemon"] = 23.'
```

注意，这里使用的是exec命令而不是addblock。 在LogicBlox中，使用addblock将“持久”代码添加到我们的工作区，例如添加到您的模式（适用于所有未来命令的定义），而exec用以执行一次性命令。

使用`print cost`命令查看效果

![](/images/LogiQL-c4-1.png)

可以使用多行的命令插入

```
exec '+cost["Fruit Sundae"] = 120.
+cost["Mango Sorbet"] = 40.
+cost["Cone Chocolate"] = 50.
+cost["Cone Vanilla"] = 44.
+cost["Cone Chili Fries"] = 200.'
```

更新数据

```
exec '^cost["Popsicle Lemon"] = 25.'
```

移除数据

```
exec '-cost["Cone Chili Fries"] = 200.'
```

![](/images/LogiQL-c4-2.png)


# 更多的谓词： 价格和销售量

```
addblock 'price[icecream] = p ->
   string(icecream), int(p).
  
week_sales[icecream, week] = value ->
  string(icecream),
  int(week),
  int(value).'
```

您会注意到`week_sales`谓词有两个关键参数而不是一个：一个用于冰淇淋类型，另一个用于本周。 您可以将其视为二维地图。

```
exec '+price["Popsicle Lemon"] = 50.
+price["Fruit Sundae"] = 200.
+price["Mango Sorbet"] = 70.
+price["Cone Chocolate"] = 80.
+price["Cone Vanilla"] = 70.

+week_sales["Popsicle Lemon", 1] = 122.
+week_sales["Fruit Sundae", 1] = 88.
+week_sales["Mango Sorbet", 1] = 72.
+week_sales["Cone Chocolate", 1] = 4.
+week_sales["Cone Vanilla", 1] = 257.

+week_sales["Popsicle Lemon", 2] = 112.
+week_sales["Fruit Sundae", 2] = 60.
+week_sales["Mango Sorbet", 2] = 44.
+week_sales["Cone Chocolate", 2] = 9.
+week_sales["Cone Vanilla", 2] = 200.'
```

![](/images/LogiQL-c5.png)


# 使用规则推理新的事实

到目前为止，我们一直在使用LogicBlox作为一个键值存储。 现在让我们首先看看如何使用LogicBlox自动计算收入和利润。

首先我们使用规则(`rule`)定义一个`week_revenue`谓词，可以写为

```
addblock 'week_revenue[icecream, week] =
  price[icecream] * week_sales[icecream, week].'
```

可以使用`print week_revenue`查看结果

同理可以定义`profit`和`week_profit`

```
addblock 'profit[icecream] =
  price[icecream] - cost[icecream].
  
week_profit[icecream, week] =
  profit[icecream] * week_sales[icecream, week].'
```


# 聚合操作

现在如果我们希望看到我们每周的收入和利润汇总，为此，我们可以使用LogiQL的聚合功能。 你可以使用各种聚合函数，在我们的例子中我们将使用`total()`来进行求和：

```
addblock 'agg_profit[week] = value <-
   agg<<value=total(p)>> week_profit[_, week] = p.
 
agg_revenue[week] = value <-
   agg<<value=total(r)>> week_revenue[_, week] = r.'
```

`agg_profit`规则表示的是将对于`profit`的数据里`week`值是我们要聚合的值的所有`p`值的和绑定到`value`，忽略ice cream的类型，（`_`为通配符），`agg_revenue`同理。


![](/images/LogiQL-c7.png)

同理，我们可以统计冰淇淋的类型数量，我们可以定义一个规则统计`price`谓词里的数据项数量

```
addblock 'icecream_count[] = value <-
   agg<<value=count()>> price[_] = _.'
```

![](/images/LogiQL-c7-1.png)


# 查询事实(facts)

我们之前定义的`icecream_count`谓词实际上更像是一次性样式查询。 在工作区中不断跟踪此值可能不是很有用。 相反，我们可以为这样的一次性查询做的是使用`query`命令。 以前我们使用`print`命令来查看谓词的值。 `print`是查询的一个非常基本的命令，但我们可以使用`query`命令执行更高级的查询。

从REPL查询`facts`的方法是通过定义一个谓词`_`来填充我们感兴趣的事实。通常我们通过将其定义为规则来做到这一点。 例如，要获得与`print week_sales`命令相同的结果，我们将使用以下LogiQL规则定义：

```
_(icecream, week, value) <-
  week_sales[icecream, week] = value.
```

您可能会注意到，在此规则中，我们使用略有不同的语法。 我们使用`predicate(k1，k2，k3)`，而不是使用函数谓词语法`predicate[k1，k2] = v`。 原因是我们只对`facts`数据项感兴趣。

上面提到了，查询语句使用`query`命令

```
query '_(icecream, week, value) <-
  week_sales[icecream, week] = value.'
```

![](/images/LogiQL-c8.png)


# 约束

让我们做一些奇怪的事情吧，让我们把柠檬冰棒的价格降低到成本价格以下：

```
exec '^price["Popsicle Lemon"] = 20.'
```

现在我们对柠檬冰棒的利润率是负的，我们可以通过以下查询看到：

```
query '_(profit) <- profit["Popsicle Lemon"] = profit.'
```

虽然这可能是某种商业策略，但我们假设我们不像微软那样以低成本销售Xbox以获得市场份额。让我们强制执行我们的利润率永远不会低于0。为此，我们制定以下约束：

```
profit[_] = value -> value >= 0.
```


这个约束表明，对于利润谓词中的每个事实，值应为0或更多。立即添加此约束并观察它是否失败：

```
addblock 'profit[_] = value -> value >= 0.'
```

这失败了！为什么？给出的理由是，现在利润谓词中存在一个事实，它当前并不认为它的值大于或等于0.当然，这是事实。 在添加新约束之前，我们必须解决此问题。 LogicBlox不允许工作空间处于并非所有约束都成立的状态。

```
exec '^price["Popsicle Lemon"] = 40.'
addblock 'profit[_] = value -> value >= 0.'
```

幸运的是，这次我们成功了。

现在，让我们看看如果我们再次将价格更新到低于成本的价格会发生什么：

```
exec '^price["Popsicle Lemon"] = 10.'
```

同样，我们得到一个错误，一个约束失败，这确实是我们所期望的。 任何导致不满足约束的更新都将自动回滚，这是LogicBlox事务性质的一部分。

# Next

可以到[LogicBlox Developer](http://developer.logicblox.com/)网站学习更多关于LogicBlox和LogiQL语言的知识。