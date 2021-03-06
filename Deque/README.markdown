# Deque
# 双端队列

A double-ended queue. For some reason this is pronounced as "deck".
出于某种原因，双端队列被称为“deck”。

A regular [queue](../Queue/) adds elements to the back and removes from the front. The deque also allows enqueuing at the front and dequeuing from the back, and peeking at both ends.
常规[队列](../Queue/)将元素添加到后面并从前面删除。 双端队列还允许在前面排队并从后面出队，并在两端查看。

Here is a very basic implementation of a deque in Swift:
这是Swift中双端队列的一个非常基本的实现：

```swift
public struct Deque<T> {
  private var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(_ element: T) {
    array.insert(element, atIndex: 0)
  }

  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    return array.first
  }

  public func peekBack() -> T? {
    return array.last
  }
}
```

This uses an array internally. Enqueuing and dequeuing are simply a matter of adding and removing items from the front or back of the array.
这在内部使用数组。 入队和出列只是在数组的前面或后面添加和删除项目。

An example of how to use it in a playground:
如何在 playground 中使用它的示例：

```swift
var deque = Deque<Int>()
deque.enqueue(1)
deque.enqueue(2)
deque.enqueue(3)
deque.enqueue(4)

deque.dequeue()       // 1
deque.dequeueBack()   // 4

deque.enqueueFront(5)
deque.dequeue()       // 5
```

This particular implementation of `Deque` is simple but not very efficient. Several operations are **O(n)**, notably `enqueueFront()` and `dequeue()`. I've included it only to show the principle of what a deque does.
`Deque`的这种特殊实现很简单但效率不高。 几个操作是 **O(n)**，特别是`enqueueFront()`和`dequeue()`。 我把它包括在内只是为了说明deque的作用原理。

## A more efficient version
## 更高效的版本

The reason that `dequeue()` and `enqueueFront()` are **O(n)** is that they work on the front of the array. If you remove an element at the front of an array, what happens is that all the remaining elements need to be shifted in memory.
`dequeue()`和`enqueueFront()`是**O(n)**的原因是它们在数组的前面工作。 如果删除数组前面的元素，那么所有剩余的元素都需要在内存中移位。

Let's say the deque's array contains the following items:
假设deque的数组包含以下项：

	[ 1, 2, 3, 4 ]

Then `dequeue()` will remove `1` from the array and the elements `2`, `3`, and `4`, are shifted one position to the front:
然后`dequeue()`将从数组中删除`1`，元素`2`，`3`和`4`将向前移动一个位置：

	[ 2, 3, 4 ]

This is an **O(n)** operation because all array elements need to be moved by one position in the computer's memory.
这是一个**O(n)**操作，因为所有数组元素都需要移动到计算机内存中的一个位置。

Likewise, inserting an element at the front of the array is expensive because it requires that all other elements must be shifted one position to the back. So `enqueueFront(5)` will change the array to be:
同样，在数组的前面插入一个元素是昂贵的，因为它要求所有其他元素必须向后移动一个位置。 因此`enqueueFront(5)`会将数组更改为：

	[ 5, 2, 3, 4 ]

First, the elements `2`, `3`, and `4` are moved up by one position in the computer's memory, and then the new element `5` is inserted at the position where `2` used to be.
首先，将元素`2`，`3`和`4`向后移动计算机内存中的一个位置，然后将新元素`5`插入到曾经是`2`的位置。

Why is this not an issue at for `enqueue()` and `dequeueBack()`? Well, these operations are performed at the end of the array. The way resizable arrays are implemented in Swift is by reserving a certain amount of free space at the back.
为什么这不是`enqueue()`和`dequeueBack()`的问题？ 好吧，这些操作是在数组末尾执行的。 在Swift中实现可调整大小的数组的方式是在后面保留一定量的可用空间。

Our initial array `[ 1, 2, 3, 4]` actually looks like this in memory:
我们的初始数组`[1,2,3,4]`实际上在内存中看起来像这样：

	[ 1, 2, 3, 4, x, x, x ]

where the `x`s denote additional positions in the array that are not being used yet. Calling `enqueue(6)` simply copies the new item into the next unused spot:
其中`x`s表示数组中尚未使用的其他位置。 调用`enqueue(6)`只是将新项目复制到下一个未使用的位置：

	[ 1, 2, 3, 4, 6, x, x ]

The `dequeueBack()` function uses `array.removeLast()` to delete that item. This does not shrink the array's memory but only decrements `array.count` by one. There are no expensive memory copies involved here. So operations at the back of the array are fast, **O(1)**.
`dequeueBack()`函数使用`array.removeLast()`删除该项。 这不会缩小数组的内存，只会将`array.count`减1。 这里没有涉及昂贵的内存副本。 因此数组后面的操作很快，**O(1)**。

It is possible the array runs out of free spots at the back. In that case, Swift will allocate a new, larger array and copy over all the data. This is an **O(n)** operation but because it only happens once in a while, adding new elements at the end of an array is still **O(1)** on average.
数组可能会用尽后面的自由点。 在这种情况下，Swift将分配一个新的更大的数组并复制所有数据。这是一个**O(n)**操作，但因为它只偶尔发生一次，所以在数组末尾添加新元素的平均值仍然是**O(1)**。

Of course, we can use this same trick at the *beginning* of the array. That will make our deque efficient too for operations at the front of the queue. Our array will look like this:
当然，我们可以在数组的*开头*使用相同的技巧。 这将使我们的deque对队列前面的操作也有效。 我们的数组将如下所示：

	[ x, x, x, 1, 2, 3, 4, x, x, x ]

There is now also a chunk of free space at the start of the array, which allows adding or removing elements at the front of the queue to be **O(1)** as well.
现在在数组的开头还有一大块可用空间，这允许在数组前面添加或删除元素也是**O(1)**。

Here is the new version of `Deque`:
这是`Deque`的新版本：

```swift
public struct Deque<T> {
  private var array: [T?]
  private var head: Int
  private var capacity: Int
  private let originalCapacity:Int

  public init(_ capacity: Int = 10) {
    self.capacity = max(capacity, 1)
    originalCapacity = self.capacity
    array = [T?](repeating: nil, count: capacity)
    head = capacity
  }

  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(_ element: T) {
    // this is explained below
  }

  public mutating func dequeue() -> T? {
    // this is explained below
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }

  public func peekBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.last!
    }
  }  
}
```

It still largely looks the same -- `enqueue()` and `dequeueBack()` haven't changed -- but there are also a few important differences. The array now stores objects of type `T?` instead of just `T` because we need some way to mark array elements as being empty.
它仍然看起来基本相同 —— `enqueue()` 和 `dequeueBack()` 没有改变 —— 但也有一些重要的区别。 数组现在存储类型为`T？`的对象而不仅仅是`T`，因为我们需要某种方法将数组元素标记为空。

The `init` method allocates a new array that contains a certain number of `nil` values. This is the free room we have to work with at the beginning of the array. By default this creates 10 empty spots.
`init`方法分配一个包含一定数量的`nil`值的新数组。 这是我们必须在数组开头处理的免费房间。 默认情况下，这会创建10个空白点。

The `head` variable is the index in the array of the front-most object. Since the queue is currently empty, `head` points at an index beyond the end of the array.
`head`变量是最前面对象数组中的索引。 由于队列当前是空的，`head`指向超出数组末尾的索引。

	[ x, x, x, x, x, x, x, x, x, x ]
	                                 |
	                                 head

To enqueue an object at the front, we move `head` one position to the left and then copy the new object into the array at index `head`. For example, `enqueueFront(5)` gives:
为了将对象排入前面，我们将`head`向左移动一个位置，然后将新对象复制到索引`head`的数组中。 例如，`enqueueFront(5)`给出：

	[ x, x, x, x, x, x, x, x, x, 5 ]
	                             |
	                             head

Followed by `enqueueFront(7)`:

	[ x, x, x, x, x, x, x, x, 7, 5 ]
	                          |
	                          head

And so on... the `head` keeps moving to the left and always points at the first item in the queue. `enqueueFront()` is now **O(1)** because it only involves copying a value into the array, a constant-time operation.
等等......`head`继续向左移动并始终指向队列中的第一个项目。 `enqueueFront()`现在是**O(1)**，因为它只涉及将值复制到数组中，这是一个恒定时间操作。

Here is the code:
代码：

```swift
  public mutating func enqueueFront(element: T) {
    head -= 1
    array[head] = element
  }
```

Appending to the back of the queue has not changed (it's the exact same code as before). For example, `enqueue(1)` gives:
附加到队列的后面没有改变（它与之前的代码完全相同）。 例如，`enqueue(1)`给出：

	[ x, x, x, x, x, x, x, x, 7, 5, 1, x, x, x, x, x, x, x, x, x ]
	                          |
	                          head

Notice how the array has resized itself. There was no room to add the `1`, so Swift decided to make the array larger and add a number of empty spots to the end. If you enqueue another object, it gets added to the next empty spot in the back. For example, `enqueue(2)`:
注意数组如何调整自身大小。 没有空间添加`1`，因此Swift决定将数组放大并在末尾添加一些空白点。 如果您将另一个对象排入队列，它将被添加到后面的下一个空白点。 例如，`enqueue(2)`：

	[ x, x, x, x, x, x, x, x, 7, 5, 1, 2, x, x, x, x, x, x, x, x ]
	                          |
	                          head

> **Note:** You won't see those empty spots at the back of the array when you `print(deque.array)`. This is because Swift hides them from you. Only the ones at the front of the array show up.
> **注意：** 当你打印（deque.array）时，你不会在数组后面看到那些空点。 这是因为Swift会将它们隐藏起来。 只显示数组前面的那些。

The `dequeue()` method does the opposite of `enqueueFront()`, it reads the value at `head`, sets the array element back to `nil`, and then moves `head` one position to the right:
`dequeue()`方法与`enqueueFront()`相反，它读取`head`的值，将数组元素设置回`nil`，然后将`head`移动到右边的一个位置：

```swift
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    return element
  }
```

There is one tiny problem... If you enqueue a lot of objects at the front, you're going to run out of empty spots at the front at some point. When this happens at the back of the array, Swift automatically resizes it. But at the front of the array we have to handle this situation ourselves, with some extra logic in `enqueueFront()`:
有一个很小的问题......如果你在前面排列了很多物体，你会在某些时候用尽前面的空白点。 当这发生在数组的后面时，Swift会自动调整它的大小。 但是在数组的前面我们必须自己处理这种情况，在`enqueueFront()`中有一些额外的逻辑：

```swift
  public mutating func enqueueFront(element: T) {
    if head == 0 {
      capacity *= 2
      let emptySpace = [T?](repeating: nil, count: capacity)
      array.insert(contentsOf: emptySpace, at: 0)
      head = capacity
    }

    head -= 1
    array[head] = element
  }
```

If `head` equals 0, there is no room left at the front. When that happens, we add a whole bunch of new `nil` elements to the array. This is an **O(n)** operation but since this cost gets divided over all the `enqueueFront()`s, each individual call to `enqueueFront()` is still **O(1)** on average.
如果`head`等于0，则前面没有剩余空间。 当发生这种情况时，我们在数组中添加了一大堆新的`nil`元素。 这是一个**O(n)**操作，但由于这个成本在所有`enqueueFront()`中被分开，所以每次对`enqueueFront()`的单独调用仍然是**O(1)**。

> **Note:** We also multiply the capacity by 2 each time this happens, so if your queue grows bigger and bigger, the resizing happens less often. This is also what Swift arrays automatically do at the back.
> **注意：** 每次发生这种情况时，我们也会将容量乘以2，因此如果您的队列越来越大，调整大小的次数就越少。 这也是Swift数组在后面自动执行的操作。

We have to do something similar for `dequeue()`. If you mostly enqueue a lot of elements at the back and mostly dequeue from the front, then you may end up with an array that looks as follows:
我们必须为`dequeue()`做类似的事情。 如果你大部分时间将很多元素排入队列并且大多数从前面出列，那么你最终可能会得到一个如下所示的数组：

	[ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
	                                                              |
	                                                              head

Those empty spots at the front only get used when you call `enqueueFront()`. But if enqueuing objects at the front happens only rarely, this leaves a lot of wasted space. So let's add some code to `dequeue()` to clean this up:
当你调用`enqueueFront()`时，只会使用前面的空白点。 但是如果前面的对象很少发生，那么就会浪费很多浪费的空间。 所以让我们在`dequeue()`中添加一些代码来清理它：

```swift
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    if capacity >= originalCapacity && head >= capacity*2 {
      let amountToRemove = capacity + capacity/2
      array.removeFirst(amountToRemove)
      head -= amountToRemove
      capacity /= 2
    }
    return element
  }
```

Recall that `capacity` is the original number of empty places at the front of the queue. If the `head` has advanced more to the right than twice the capacity, then it's time to trim off a bunch of these empty spots. We reduce it to about 25%.
回想一下`capacity`是队列前面的空位的原始数量。 如果`head`向右前进的次数超过了容量的两倍，那么就该修剪掉这些空洞了。 我们将它降低到约25％。

> **Note:**  The deque will keep at least its original capacity by comparing `capacity` to `originalCapacity`.
> **注意：** 通过将`capacity`与`originalCapacity`进行比较，deque将至少保持其原始容量。

For example, this:

	[ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
	                                |                             |
	                                capacity                      head

becomes after trimming:

	[ x, x, x, x, x, 1, 2, 3 ]
	                 |
	                 head
	                 capacity

This way we can strike a balance between fast enqueuing and dequeuing at the front and keeping the memory requirements reasonable.
通过这种方式，我们可以在前面的快速入队和出队以及保持内存要求合理之间取得平衡。

> **Note:** We don't perform trimming on very small arrays. It's not worth it for saving just a few bytes of memory.
> **注意：** 我们不对非常小的数组执行修剪。 仅保存几个字节的内存是不值得的。

## See also
## 扩展阅读

Other ways to implement deque are by using a [doubly linked list](../Linked%20List/), a [circular buffer](../Ring%20Buffer/), or two [stacks](../Stack/) facing opposite directions.
实现deque的其他方法是使用[双向链表](../Linked％20List/)，[环形缓冲区](../Ring％20Buffer/)，或两个[stacks](../Stack/)面向相反的方向。

[A fully-featured deque implementation in Swift](https://github.com/lorentey/Deque)

*Written for Swift Algorithm Club by Matthijs Hollemans*
*作者：Matthijs Hollemans*  
*翻译：[Andy Ron](https://github.com/andyRon)*
