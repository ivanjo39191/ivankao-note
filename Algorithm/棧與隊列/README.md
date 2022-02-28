# 棧與隊列

Created: February 23, 2022 2:37 PM
Status: 待公開
Tags: Algorithm

### 將 List 作為 Stack（堆疊）使用

List 的操作方法使得它非常簡單可以用來實作 stack（堆疊）。Stack 為一個遵守最後加入元素最先被取回（後進先出，"last-in, first-out"）規則的資料結構。你可以使用方法 `append()` 將一個項目放到堆疊的頂層。而使用方法 `pop()` 且不給定索引值去取得堆疊最上面的項目。舉例而言：

```bash
>>> stack = [3, 4, 5]
>>> stack.append(6)
>>> stack.append(7)
>>> stack
[3, 4, 5, 6, 7]
>>> stack.pop()
7
>>> stack
[3, 4, 5, 6]
>>> stack.pop()
6
>>> stack.pop()
5
>>> stack
[3, 4]
```

### 將 List 作為 Queue（佇列）使用

我們也可以將 list 當作 queue（佇列）使用，即最先加入元素最先被取回（先進先出，"first-in, first-out"）的資料結構。然而，list 在這種使用方式下效率較差。使用 `append` 和 `pop` 來加入和取出尾端的元素較快，而使用 `insert` 和 `pop` 來插入和取出頭端的元素較慢（因為其他元素都需要挪動一格）。

如果要實作 queue，請使用 `[collections.deque](https://docs.python.org/zh-tw/3/library/collections.html#collections.deque)`，其被設計成能快速的從頭尾兩端加入和取出。例如：

```bash
>>> from collections import deque
>>> queue = deque(["Eric", "John", "Michael"])
>>> queue.append("Terry")           # Terry arrives
>>> queue.append("Graham")          # Graham arrives
>>> queue.popleft()                 # The first to arrive now leaves
'Eric'
>>> queue.popleft()                 # The second to arrive now leaves
'John'
>>> queue                           # Remaining queue in order of arrival
deque(['Michael', 'Terry', 'Graham'])
```

## Python List

| 操作 | 操作說明 | 時間複雜度（平均情況） | 時間複雜度（最壞情況） |
| --- | --- | --- | --- |
| index(value) | 查詢list某個元素的索引 | O(1) | O(1) |
| a=index(value) | 索引賦值 | O(1) | O(1) |
| list[:] | 列表複製 | O(n) | O(n) |
| list.append(value) | 隊尾新增 | O(1) | O(1) |
| list.insert(index, value) | 根據索引插入某個元素 | O(n) | O(n) |
| list[index] | 取元素 | O(1) | O(1) |
| list[index]=value | 賦值 | O(1) | O(1) |
| list.pop() | 隊尾刪除 | O(1) | O(1) |
| list.pop(index) | 根據索引刪除某個元素 | O(n) | O(n) |
| [i for i in list] | 遍歷/迭代 | O(n) | O(n) |
| list[m:n] | 取切片 | O(k) | O(k) |
| del list[m:n] | 刪除切片 | O(n) | O(n) |
| list[m:n]=[n] | 更改切片 | O(k+n) | O(k+n) |
| list.extend([]) | 列表擴充套件 | O(k) | O(k) |
| list.sort() | 列表排序 | O(nlogn) | O(nlogn) |
| list*n | 列表乘法 | O(nk) | O(nk) |
| i in list | 列表搜尋 | O(n) |  |
| min(list), max(list) | 取最大和最小值 | O(n) |  |
| len(list) | 計算長度 | O(1) | O(1) |

[https://programmercarl.com/栈与队列理论基础.html](https://programmercarl.com/%E6%A0%88%E4%B8%8E%E9%98%9F%E5%88%97%E7%90%86%E8%AE%BA%E5%9F%BA%E7%A1%80.html)

[https://docs.python.org/zh-tw/3/tutorial/datastructures.html](https://docs.python.org/zh-tw/3/tutorial/datastructures.html)

[https://www.itread01.com/content/1547117713.html](https://www.itread01.com/content/1547117713.html)

[http://www.laurentluce.com/posts/python-list-implementation/](http://www.laurentluce.com/posts/python-list-implementation/)