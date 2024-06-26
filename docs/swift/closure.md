# 闭包

<iframe style="border:none" width="100%" height="450" src="https://whimsical.com/embed/5sCTbd6Ny8inpxbCkbA7Qb"></iframe>

闭包是自包含的函数代码块，允许你创建**没有具体名称的函数结构**，可以在代码中被传递和使用。


::: info 闭包的一个核心特性是它能够捕获周围上下文中的常量和变量
如果你在闭包内部使用了外部的变量或常量，即使闭包在其原始环境之外执行，它仍然可以访问并操作这些变量或常量。

Swift 会自动处理闭包中捕获的常量和变量的内存管理问题，你不需要担心内存泄漏或者变量生命周期的问题。

这意味着在使用闭包时，你可以专注于业务逻辑的实现，而不必过分关心底层的内存管理细节。
:::


## 闭包表达式


嵌套函数是定义在另一个函数内部的函数，它可以帮助将复杂的函数分解成更小、更易管理的部分。

闭包表达式是 Swift 中一种用于构建内联闭包的简洁方式，它们通常用在需要将一个函数作为参数传递给另一个函数的场合。

让我们通过 `sorted(by:)` 的例子来看看如何通过多次迭代简化闭包的表达方式。

首先，我们从一个较为详细的闭包开始，然后逐步简化它：

1. **完整的闭包表达式**：
    ```swift
    let numbers = [5, 3, 9, 1, 6]
    let sortedNumbers = numbers.sorted(by: { (s1: Int, s2: Int) -> Bool in
        return s1 < s2
    })
    ```
    这里，闭包完整地指定了参数类型和返回类型，并明确写出了返回语句。

2. **类型推断**：
    因为 Swift 能够推断闭包参数和返回值的类型，我们可以省略它们：
    ```swift
    let sortedNumbers = numbers.sorted(by: { s1, s2 in return s1 < s2 })
    ```
    在这个版本中，我们省略了参数的类型声明，因为 Swift 能从上下文中推断出它们的类型。

3. **单表达式闭包隐式返回**：
    当闭包体只包含一个表达式时，该表达式的结果会自动作为闭包的返回值，所以我们可以省略 `return` 关键字：
    ```swift
    let sortedNumbers = numbers.sorted(by: { s1, s2 in s1 < s2 })
    ```

4. **速记参数名称**：
    Swift 自动为内联闭包提供了速记参数名称，如 `$0`, `$1` 等，这可以进一步简化表达式：
    ```swift
    let sortedNumbers = numbers.sorted(by: { $0 < $1 })
    ```

5. **运算符方法**：
    最后，因为 `<` 本身是一个接受两个 `Int` 类型并返回 `Bool` 类型的函数，所以我们可以直接传递这个运算符：
    ```swift
    let sortedNumbers = numbers.sorted(by: <)
    ```

## 尾随闭包

尾随闭包是 Swift 中一个语法特性，允许你在调用函数时，将一个较长的闭包表达式作为函数的最后一个参数传递 `{}`，而不是将其包含在函数的括号内 `()`。

### 尾随闭包的使用

当函数的最后一个参数是闭包表达式时，闭包表达式直接跟在函数括号 `()` 之后写，不需要写在括号内。如果闭包是函数的唯一参数，则调用时甚至可以**省略空的圆括号**。

考虑一个简单的例子：

```swift{2,7,12,17}
//原始写法
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // 函数体部分
}

// 以下是不使用尾随闭包进行函数调用
someFunctionThatTakesAClosure(closure: {
    // 闭包主体部分
})

// 以下是使用尾随闭包进行函数调用
someFunctionThatTakesAClosure() {
    // 闭包主体部分
}

// 由于闭包是函数的最后一个（也是唯一一个）参数，你甚至可以省略调用时的空括号：
someFunctionThatTakesAClosure{
    // 闭包主体部分
}
```

::: tip 尾随闭包的优势
1. **提高可读性**：尤其当闭包代码量较大时，将闭包作为尾随闭包书写可以使得函数调用看起来更加清晰。
2. **便于编写复杂闭包**：尾随闭包的格式便于编写多行代码的闭包。
3. **代码结构清晰**：帮助区分函数参数和闭包逻辑，特别是在闭包是最后一个或唯一一个参数的情况下。
:::


## 值捕获

想象你有一个小箱子，你可以在里面放一些东西（比如一个数字），然后你把这个箱子交给你的朋友。即使你离开了，你的朋友仍然可以打开箱子看里面的东西，甚至可以改变里面的东西。在 Swift 的闭包中，这个「箱子」就是闭包能够「捕获」变量的能力。

让我们看一个生活中的例子：

```swift
func createMessage() -> () -> String {
    let greeting = "Hello"
    let message = "World"
    
    let sayHello: () -> String = {
        return greeting + " " + message
    }
    
    return sayHello
}

let helloMessage = createMessage()
print(helloMessage())  // 输出 "Hello World"
```

这里，`createMessage` 函数定义了两个变量 `greeting` 和 `message`，然后定义了一个闭包 `sayHello`。

- 这个闭包通过简单地连接这两个字符串来创建一条消息。

- **捕获行为**：尽管 `createMessage` 函数的执行在返回闭包之后就结束了，这两个变量 `greeting` 和 `message` 仍然「活着」，因为闭包 `sayHello` 已经捕获了它们。即使外部函数的作用域已经结束，闭包仍然持有这些变量的访问权和它们的当前值。

::: tip 关键点理解

1. **生命周期延长**：闭包使得变量即使在其定义的函数已经结束执行后，也可以继续存在。
2. **值复制**：在闭包中，捕获的值类型变量是在闭包被创建时复制的。这意味着闭包内的变量和外部的变量虽然初始值相同，但是它们是独立的副本。
3. **引用和影响**：如果闭包捕获的是引用类型的变量，那么闭包内外的修改将会相互影响，因为它们指向的是同一个对象。

:::

## 闭包是引用类型

当你把一个闭包赋值给一个变量或常量，或者将一个闭包作为参数传递给函数时，实际上传递的是对闭包的**引用**，而不是闭包的**拷贝**。


::: info
引用类型的特性意味着如果你将一个闭包赋值给两个不同的变量，这两个变量将指向同一个闭包实例。

因此，如果闭包内部的状态发生变化，这种变化会反映在所有引用了这个闭包的变量上。
:::

让我们通过一个例子来更直观地理解这一点：

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var total = 0
    let incrementer: () -> Int = {
        total += amount
        return total
    }
    return incrementer
}

let incrementByFive = makeIncrementer(forIncrement: 5)
let alsoIncrementByFive = incrementByFive

incrementByFive()        // 返回 5
alsoIncrementByFive()    // 返回 10
incrementByFive()        // 返回 15
```

在这个例子中：

- `incrementByFive` 是一个闭包，它每次被调用时都会将其内部的 `total` 变量增加 $5$。
- `alsoIncrementByFive` 是对同一个闭包的另一个引用。
- 因为闭包是引用类型，所以通过 `alsoIncrementByFive` 调用闭包，会影响到 `incrementByFive`，反之亦然。它们都操作相同的 `total` 变量，因为它们实际上引用的是同一个闭包实例。

::: tip 关键点

- **状态共享**：通过闭包的引用类型特性，多个变量可以共享对同一个闭包实例的引用，从而共享闭包内的状态。
- **内存管理**：Swift 使用引用计数来管理内存，这对于闭包也是适用的。如果一个闭包在其定义的范围之外被引用，Swift 会确保闭包所捕获的所有变量仍然存在，直到闭包本身不再被使用。

:::

## 逃逸闭包

逃逸闭包是指在其被传入的函数返回之后才被调用的闭包。因为闭包逃离了它被定义的作用域，所以称为「逃逸」闭包。

你需要使用 `@escaping` 关键字来明确地标记一个闭包参数是逃逸的。

这告诉 Swift 编译器这个闭包可能在函数返回之后才被执行，因此编译器会做出相应的内存管理处理。

```swift
func loadFromNetwork(completionHandler: @escaping (String) -> Void) {
    // 模拟网络请求
    DispatchQueue.global().async {
        // 假设这里是从服务器获取到的数据
        let fetchedData = "Data from server"
        DispatchQueue.main.async {
            // 回到主线程调用闭包
            completionHandler(fetchedData)
        }
    }
}

loadFromNetwork { data in
    print("Received network data: \(data)")
}
```

在这个例子中，`completionHandler` 是一个逃逸闭包，因为它在 `loadFromNetwork` 函数执行完毕后，由异步网络请求的回调触发。


:::danger 逃逸闭包的影响

- **内存管理**：逃逸闭包可能需要额外的内存管理，因为 Swift 需要确保闭包内使用的所有捕获的资源在闭包执行时依然有效。
- **生命周期**：逃逸闭包的生命周期可能比函数长。因此，当你使用类实例的属性或方法时，需要小心循环引用问题。

:::

## 自动闭包



自动闭包是一个自动创建的闭包，用于封装传递给函数作为参数的表达式。当函数需要这个参数的值时，闭包被执行，表达式的结果被返回。这样做的好处是，只有在需要其值的时候，表达式才会被求值。

::: tip
自动闭包是一个非常有用的功能，它允许你延迟执行某些表达式的计算，直到这个表达式真正需要被计算。这通常用在延迟求值的场景，尤其是在函数参数的处理上非常方便。
:::

你可以通过在参数类型前使用 `@autoclosure` 标志来声明一个自动闭包。

```swift
func logIfTrue(_ predicate: @autoclosure () -> Bool) {
    if predicate() {
        print("条件为真")
    }
}

logIfTrue(2 > 1)
```

- 在这个例子中，`logIfTrue` 函数接受一个 `@autoclosure` 闭包作为参数。
- 当你调用 `logIfTrue(2 > 1)` 时，`2 > 1` 这个表达式被自动转换成闭包。
- 这个闭包在 `if` 语句内被调用，只有在实际需要判断条件时，表达式才被求值。

自动闭包通常用在那些表达式需要被延迟计算的场景中，例如：

- 延迟重计算，直到必要时才计算值。
- 控制语句内，仅当条件满足时才执行某些计算。
- 函数调用时，减少不必要的计算，提高性能。

::: warning 注意事项

- **副作用**：如果闭包内的表达式具有副作用，或者执行成本较高，你需要小心使用自动闭包，因为它可能导致不易察觉的错误或性能问题。
- **逃逸与非逃逸**：自动闭包可以是逃逸的或非逃逸的。如果你打算在函数返回后使用这个闭包，你需要标记为 `@escaping`。

:::