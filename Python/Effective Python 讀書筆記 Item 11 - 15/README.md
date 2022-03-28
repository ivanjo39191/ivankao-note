# Effective Python 讀書筆記 Item 11 - 15

Created: March 17, 2022 10:10 AM  
Property: Ivan Kao  
Tags: Python  

# 1. Lists and Dictionaries

許多程序都是為了自動化重複性任務而編寫的。

在 Python 中，最常見方法是使用存儲在列表類型中的一系列值。

列表非常通用，可用於解決各種問題。

列表的自然補充是 dict 類型，它存儲映射到相應值的查找鍵。

字典為分配和訪問提供恆定的時間效能，是紀錄動態資料的理想選擇。

Python 具有特殊的語法和內置模塊，可增強可讀性並擴展列表和字典的功能。

## **Item 11: Know How to Slice Sequences**

了解如對對序列進行切片

Python 包含用於將序列分割成片段的語法。

切片允許您以最小的努力訪問序列項目的子集。

切片最簡單的用途是內置類型 list、str 和 bytes。

切片可以擴展到任何實現 **getitem** 和 **setitem** 特殊方法的 Python 類。

### 基本切片形式

切片語法的基本形式是 somelist[start:end]，其中包含了 start ，不包含 end 

```python
a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
print('Middle two: ', a[3:5])
print('All but ends:', a[1:7])
>>>
Middle two: ['d', 'e']
All but ends: ['b', 'c', 'd', 'e', 'f', 'g']
```

從列表的開頭切片時，應省略零索引以減少視覺雜亂

```python
assert a[:5] == a[0:5]
```

切片到列表末尾時，應省略多餘的最終索引

```python
assert a[5:] == a[5:len(a)]
```

使用負數進行切片對列表末尾進行偏移，對代碼的新讀者來說會更清楚。

```python
a[:] # ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
a[:5] # ['a', 'b', 'c', 'd', 'e']
a[:-1] # ['a', 'b', 'c', 'd', 'e', 'f', 'g']
a[4:] # ['e', 'f', 'g', 'h']
a[-3:] # ['f', 'g', 'h']
a[2:5] # ['c', 'd', 'e']
a[2:-1] # ['c', 'd', 'e', 'f', 'g']
a[-3:-1] # ['f', 'g']
```

切片通過默認忽略缺失項來處理超出列表邊界的開始和結束索引，可避免超出最大長度的異常。

```python
first_twenty_items = a[:20]
last_twenty_items = a[-20:]
```

若直接訪問同一個索引

```python
a[20]
>>>
Traceback ...
IndexError: list index out of range
```

負數索引的 -0 時會返回原始列表

```python
somelist[-3] # 正常返回 somelist[-3]
somelist[-0] # 等同返回 somelist[:] 並生成原始列表
```

切片列表的結果是一個全新的列表。 保留對原始列表中對象的引用。 修改切片結果不會影響原列表。

```python
b = a[3:]
print('Before: ', b)
b[1] = 99
print('After: ', b)
print('No change:', a)
>>>
Before: ['d', 'e', 'f', 'g', 'h']
After: ['d', 99, 'f', 'g', 'h']
No change: ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
```

在分配中使用時，切片會替換原始列表中的指定範圍。 這裡與 unpacking 的分配不同，切片分配的長度不需要相同。 分配切片之前和之後的值將被保留。

```python
# 列表縮小是因為替換列表比指定的切片短
print('Before ', a)
a[2:7] = [99, 22, 14]
print('After ', a)
>>>
Before ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
After ['a', 'b', 99, 22, 14, 'h']
And here the list grows because the assigned

# 列表變長是因為替換列表比指定的切片長
print('Before ', a)
a[2:3] = [47, 11]
print('After ', a)
>>>
Before ['a', 'b', 99, 22, 14, 'h']
After ['a', 'b', 47, 11, 22, 14, 'h']
```

如果在切片時省略了開始索引和結束索引，則會得到原始列表的副本

```python
b = a[:]
assert b == a and b is not a
```

如果沒有定義開始或結束索引的切片，則將列表的全部內容替換為引用內容的副本

```python
b = a
print('Before a', a)
print('Before b', b)
a[:] = [101, 102, 103]
assert a is b # Still the same list object
print('After a ', a) # Now has different contents
print('After b ', b) # Same list, so same contents as a
>>>
Before a ['a', 'b', 47, 11, 22, 14, 'h']
Before b ['a', 'b', 47, 11, 22, 14, 'h']
After a [101, 102, 103]
After b [101, 102, 103]
```

### 需要記住的事項：

1. 切片時避免冗長：不要為開始索引提供 0 或為結束索引提供序列長度。
2. 切片可以容忍超出範圍的開始或結束索引，可以在序列的前邊界或後邊界上省略（如 a[:20] 或 a[-20:]）。
3. 分配給列表切片會替換原始序列中的範圍

## Item 12: Avoid Striding and Slicing in a Single Expression

避免在單個表達式中進行跨步與切片

### Stride 跨步

除了基本的切片之外，Python 還為切片的步幅提供了特殊的語法

```python
somelist[start:end:stride]
```

這使您可以在對序列進行切片時獲取每第 n 個項目。 例如，stride 可以很容易地按列表中的偶數和奇數索引進行分組

```python
x = ['red', 'orange', 'yellow', 'green', 'blue', 'purple']
odds = x[::2]
evens = x[1::2]
print(odds)
print(evens)
>>>
['red', 'yellow', 'blue']
['orange', 'green', 'purple']
```

跨步語法可能會導致可能引入錯誤的意外行為。 例如，用於反轉字節字符串的常見 Python 技巧是以 -1 的步幅對字符串進行切片

```python
x = b'mongoose'
y = x[::-1]
print(y)
```

適用於 Unicode 字符串

```python
x = '壽司'
y = x[::-1]
print(y)

```

不適用於 utf-8 字符串，當 Unicode 數據被編碼為 UTF-8 字節字符串時它會中斷。

```python
w = '壽司'
x = w.encode('utf-8')
y = x[::-1]
z = y.decode('utf-8')
>>>
Traceback ...
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb8 in 
position 0: invalid start byte
```

2::2 vs  -2::-2 vs -2:2:-2 vs 2:2:-2

```python
x[2::2] # ['c', 'e', 'g']
x[-2::-2] # ['g', 'e', 'c', 'a']
x[-2:2:-2] # ['g', 'e']
x[2:2:-2] # []
```

這些切片與步幅顯得非常混亂，括號中的三個數字由於其密度而難以閱讀。 然後，相對於步幅值，開始和結束索引何時生效並不明顯，尤其是當他的步幅為負值時。

為防止出現問題，建議避免將步幅與開始和結束索引一起使用。

如果必須使用步幅，最好將其設為正值並省略開始和結束索引。

如果必須使用帶開始或結束索引的跨步，請考慮使用一個分配進行跨步，另一個分配用於切片。

```python
y = x[::2] # ['a', 'c', 'e', 'g']
z = y[1:-1] # ['c', 'e']
```

跨步然後切片會創建額外淺表副本 data。

第一個操作應該盡量減少結果切片的大小。

如果您的程序無法承擔兩個步驟所需的時間或內存，請考慮使用 itertools 內置模塊的 islice 方法，該方法更易於閱讀，並且不允許開始、結束或步幅為負值。

### 需要記住的事項：

1. 在切片中指定開始、結束和步幅可能會非常混亂。
2. 盡量在沒有開始或結束索引的切片中使用正步幅值。 盡可能避免負步幅值。
3. 避免在一個切片中同時使用 start、end 和 stride。 如果需要三個參數，請考慮執行兩個分配（一個用於跨步，另一個用於切片）或使用 itertools 內置模塊中的 islice

## Item 13: Prefer Catch-All Unpacking Over Slicing

使用 星號表達式全面解包 優於使用 切片

### 使用星號表達式進行全面解包

基本的 解包 需要先知道序列的長度，若不吻合就會出現錯誤，雖然可行但代碼會看起來很雜亂。

```python
car_ages = [0, 9, 4, 8, 7, 20, 19, 1, 6, 15]
car_ages_descending = sorted(car_ages, reverse=True)
oldest, second_oldest = car_ages_descending
>>>
Traceback ...
ValueError: too many values to unpack (expected 2)

# 使用 slice
oldest = car_ages_descending[0]
second_oldest = car_ages_descending[1]
others = car_ages_descending[2:]
print(oldest, second_oldest, others)
>>>
20 19 [15, 9, 8, 7, 6, 4, 1, 0]
```

使用星號表達式進行全面解包，此語法允許解包分配的一部分，與其他不匹配的所有值。

這段代碼更短，更容易閱讀，並且不再具有邊界索引容易出錯的脆弱性。

```python
oldest, second_oldest, *others = car_ages_descending
print(oldest, second_oldest, others)
>>>
20 19 [15, 9, 8, 7, 6, 4, 1, 0]
```

星號表達式可以放在任何位置，可以在取得一個切片的時候使用全面解包

```python
oldest, *others, youngest = car_ages_descending
print(oldest, youngest, others)
*others, second_youngest, youngest = car_ages_descending
print(youngest, second_youngest, others)
>>>
20 0 [19, 15, 9, 8, 7, 6, 4, 1]
0 1 [20, 19, 15, 9, 8, 7, 6, 4]
```

星號表達式 不能單獨使用，必須至少有一個必需的值，否則您將收到 SyntaxError。 

```python
*others = car_ages_descending
>>>
Traceback ...
SyntaxError: starred assignment target must be in a list or  ̄tuple
```

不能再單個層級中使用多個星號表達式

```python
first, *middle, *second_middle, last = [1, 2, 3, 4]
>>>
Traceback ...
SyntaxError: two starred expressions in assignment
```

可以在不同層級中使用多個星號表達式，但不建議使用

```python
car_inventory = {
    'Downtown': ('Silver Shadow', 'Pinto', 'DMC'),
    'Airport': ('Skyline', 'Viper', 'Gremlin', 'Nova'),
}
((loc1, (best1, *rest1)),
 (loc2, (best2, *rest2))) = car_inventory.items()
print(f'Best at {loc1} is {best1}, {len(rest1)} others')
print(f'Best at {loc2} is {best2}, {len(rest2)} others')
>>>
Best at Downtown is Silver Shadow, 2 others
Best at Airport is Skyline, 3 others
```

星號的表達式都會為 list 實例
如果被解包的序列中沒有剩餘的項目，則會是一個空 list

```python
short_list = [1, 2]
first, second, *rest = short_list
print(first, second, rest)
>>>
1 2 []
```

您還可以使用星號的表達式解包任意迭代器。 但對於基本的多重賦值語句來說，不建議使用，因為直接賦值會更容易。

```python
it = iter(range(1, 3))
first, second = it
print(f'{first} and {second}')
>>>
1 and 2
```

### 使用星號表達式進行全面解包 — 範例

隨著星號表達式的添加，解包迭代器的價值就變得清晰了。

這裡有一個生成器生成訂單的 CSV 文件：

```python
def generate_csv():
    yield ('Date', 'Make' , 'Model', 'Year', 'Price')
    ...
```

使用索引和切片處理這個生成器的結果很好，但代碼多行且看起來很雜亂。

```python
all_csv_rows = list(generate_csv())
header = all_csv_rows[0]
rows = all_csv_rows[1:]
print('CSV Header:', header)
print('Row count: ', len(rows))
>>>
CSV Header: ('Date', 'Make', 'Model', 'Year', 'Price')
Row count:  200
```

使用星號表達式解包可以很容易地處理第一行——標題——與迭代器的其餘內容分開，這更清楚。

```python
it = generate_csv()
header, *rows = it
print('CSV Header:', header)
print('Row count: ', len(rows))
>>>
CSV Header: ('Date', 'Make', 'Model', 'Year', 'Price')
Row count:  200
```

但是請記住，因為加星號的表達式總是變成一個列表，解壓迭代器也有可能會耗盡計算機上的所有內存並導致程序崩潰。
因此，只有當您有充分的理由相信結果數據都適合內存時，您才應該對迭代器使用全面解包。

### 要記住的事情

1. 解包分配可以使用星號表達式將未分配的值分配到一個 list 中。
2. 帶星號的表達式可以出現在任何位置，將成為一個包含接收到的零個或多個值的列表。
3. 將列表劃分為不重疊的部分時，全面解包比切片和索引更不容易出錯。

## Item 14: Sort by Complex Criteria Using the key Parameter

使用關鍵參數按複雜標準排序

### list 內置類型排序

list 對於內置類型提供了一種排序方法，用於根據各種條件對列表實例中的項目進行排序。

默認情況下，將按照項目的自然升序對列表的內容進行排序。 

```python
numbers = [93, 86, 11, 68, 70]
numbers.sort()
print(numbers)
>>>
[11, 68, 70, 86, 93]
```

### 對於非內置類型排序

sort 方法適用於幾乎所有具有自然排序的內置類型（字符串、浮點數等），但非內置類型將不起作用。

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight
    def __repr__(self):
        return f'Tool({self.name!r}, {self.weight})'
tools = [
    Tool('level', 3.5),
    Tool('hammer', 1.25),
    Tool('screwdriver', 0.5),
    Tool('chisel', 0.25),
]

tools.sort()
>>>
Traceback ...
TypeError: '<' not supported between instances of 'Tool' and
'Tool'
```

### 對 sort 傳遞關鍵參數

sort 方法接受一個關鍵參數，該參數應該是一個函數。

向 key 函數傳遞了一個參數，該參數是正在排序的列表中的一個項目。

鍵函數的返回值應該是一個可比較的值（即具有自然排序），以代替用於排序目的的項目。

以下使用 lambda 關鍵字為關鍵參數定義一個函數，按名稱的字母順序對 Tool 對象列表進行排序

```python
print('Unsorted:', repr(tools))
tools.sort(key=lambda x: x.name)
print('\nSorted:  ', tools)
>>>
Unsorted: [Tool('level',       3.5),
           Tool('hammer',      1.25),
           Tool('screwdriver', 0.5),
           Tool('chisel',      0.25)]
Sorted:   [Tool('chisel',      0.25),
           Tool('hammer',      1.25),
           Tool('level',      3.5),
           Tool('screwdriver', 0.5)]
```

很容易的可以換成使用 weight 進行排序

```python
tools.sort(key=lambda x: x.weight)
print('By weight:', tools)
>>>
By weight: [Tool('chisel',      0.25),
            Tool('screwdriver', 0.5),
            Tool('hammer',      1.25),
            Tool('level',       3.5)]
```

在作為關鍵參數傳遞的 lambda 函數中，可以訪問項目的屬性，對項目進行索引（對於序列、元組和字典），或使用任何其他有效的表達式。
對於像字符串這樣的基本類型，您甚至可能希望在排序之前使用鍵函數對值進行轉換。

例如將 lower 方法應用於列表中的每個項目，以確保它們按字母順序排列，忽略任何大小寫（因為在字符串的自然詞彙排序中，大寫字母排在小寫字母之前）

```python
places = ['home', 'work', 'New York', 'Paris']
places.sort()
print('Case sensitive:  ', places)
places.sort(key=lambda x: x.lower())
print('Case insensitive:', places)
>>>
Case sensitive:   ['New York', 'Paris',    'home',  'work']
Case insensitive: ['home',     'New York', 'Paris', 'work']
```

### 多個標準排序

有時需要使用多個標準進行排序，例如先按重量再按名稱排序，以下為一個列表。

```python
power_tools = [
    Tool('drill', 4),
    Tool('circular saw', 5),
    Tool('jackhammer', 40),
    Tool('sander', 4),
]
```

Python 中最簡單的解決方案是使用元組類型。 元組是任意 Python 值的不可變序列。 默認情況下，元組是可比較的，並且具有自然排序。

```python
saw = (5, 'circular saw')
jackhammer = (40, 'jackhammer')
assert not (jackhammer < saw)  # Matches expectations
```

如果正在比較的元組中的第一個位置相等（在這種情況下為權重），則元組比較將移至第二個位置，依此類推。

```python
drill = (4, 'drill')
sander = (4, 'sander')
assert drill[0] == sander[0]  # Same weight
assert drill[1] < sander[1]   # Alphabetically less
assert drill < sander         # Thus, drill comes first
```

可以利用元組比較行為來首先按重量然後按名稱對列表進行排序。 

以下定義了一個鍵函數，它返回一個元組，其中包含我想按優先級排序的兩個屬性

```python
power_tools.sort(key=lambda x: (x.weight, x.name))
print(power_tools)
>>>
[Tool('drill',        4),
 Tool('sander',       4),
 Tool('circular saw', 5),
 Tool('jackhammer',   40)]
```

讓鍵函數返回元組的一個限制是所有條件的排序方向必須相同（全部按升序排列，或全部按降序排列）。 

如果為排序方法提供反向參數，它將以相同的方式影響元組中的兩個條件

```python
power_tools.sort(key=lambda x: (x.weight, x.name),
                 reverse=True)  # Makes all criteria descending
print(power_tools)
>>>
[Tool('jackhammer',   40),
 Tool('circular saw', 5),
 Tool('sander',       4),
 Tool('drill',        4)]
```

對於數值可使用減號運算符來混合排序方向

```python
power_tools.sort(key=lambda x: (-x.weight, x.name))
print(power_tools)
>>>
[Tool('jackhammer',   40),
 Tool('circular saw', 5),
 Tool('drill',        4),
 Tool('sander',       4)]
```

但是，一元否定不可能適用於所有類型。 

這裡可以通過使用 reverse 參數按重量降序排序，然後再以名稱按升序排列來達到相同的結果。

要注意的是，要由最低排序到最高排序的順序進行調用。

```python
power_tools.sort(key=lambda x: x.name)   # Name ascending
power_tools.sort(key=lambda x: x.weight, # Weight descending
                 reverse=True)
print(power_tools)
>>>
[Tool('jackhammer',   40),
 Tool('circular saw', 5),
 Tool('drill',        4),
 Tool('sander',       4)]
```

同樣的方法可用於在任何方向上分別組合盡可能多的不同類型的排序標準。只需要確保按照您希望最終列表包含的相反順序執行排序。

讓 key 函數返回一個元組並使用一元否定來混合排序順序的方法更易於閱讀並且需要更少的代碼。建議僅在必要時才使用多次調用排序。

### 要記住的事情

1. list 類型的 sort 方法可用於通過內置類型（如字符串、整數、元組等）的自然排序來重新排列列表的內容。
2. sort 方法不適用於非內置類型的物件，除非它們使用特殊方法定義自然排序，這並不常見。
3. sort 方法的 key 參數可用於提供一個幫助函數，該函數返回用於排序的值，以代替列表中的每個項目。
4. 從鍵函數返回一個元組允許您將多個排序標準組合在一起。 一元減號運算符可用於反轉允許它的類型的單個排序順序。
5. 對於無法取反的類型，可以通過使用不同的鍵函數和反向值多次調用 sort 方法來將許多排序標準組合在一起，按照最低排序到最高排序的順序進行調用。

## Item 15: Be Cautious When Relying on dict Insertion Ordering

使用字典進行插入排序時要小心

### Python3.5 前的字典排序

在 Python 3.5 及之前的版本中，迭代 dict 將返回鍵任意順序。
迭代順序與插入項目的順序不匹配。

```python
# Python 3.5
baby_names = {
    'cat': 'kitten',
    'dog': 'puppy',
}
print(baby_names)
>>>
{'dog': 'puppy', 'cat': 'kitten'}
```

創建時的順序是 cat, dog ，打印時卻是 dog, cat ，這使測試與調適都變得困難。

### Python3.6 開始的字典排序

從 Python 3.6 開始，字典將保留插入順序。也正式成為 3.7 版 Python 規範的一部分。

此更改對依賴於 dict 類型及其特定實現的其他 Python 功能有很多影響。

字典保存插入順序的方式現在是 Python 語言規範的一部分。 

對於上述語言特性，您可以依賴此行為，甚至可以將其作為您為類和函數設計的 API 的一部分

```python
# Python3.6+
baby_names = {
    'cat': 'kitten',
    'dog': 'puppy',
}
print(baby_names)
>>>
{'cat': 'kitten', 'dog': 'puppy'}
```

### 在 kwargs 中保留插入順序

kwargs 依賴於 dict 類型進行實現，從 python3.6 開始會保留順序。

```python
# Python 3.5
def my_func(**kwargs):
    for key, value in kwargs.items():
        print('%s = %s' % (key, value))
my_func(goose='gosling', kangaroo='joey')
>>>
kangaroo = joey
goose = gosling

# Python3.6+
def my_func(**kwargs):
    for key, value in kwargs.items():
        print(f'{key} = {value}')
my_func(goose='gosling', kangaroo='joey')
>>>
goose = gosling
kangaroo = joey
```

### Class 保留實例字典

class 的 __init__ 是由 dict 類型進行實現，從 python3.6 開始會保留順序。

```python
# Python 3.5
class MyClass:
    def __init__(self):
        self.alligator = 'hatchling'
        self.elephant = 'calf'
a = MyClass()
for key, value in a.__dict__.items():
    print('%s = %s' % (key, value))
>>>
elephant = calf
alligator = hatchling

# Python3.6+
class MyClass:
    def __init__(self):
        self.alligator = 'hatchling'
        self.elephant = 'calf'
a = MyClass()
for key, value in a.__dict__.items():
    print(f'{key} = {value}')
>>>
alligator = hatchling
elephant = calf
```

### OrderedDict 有序字典

collections 內置模塊都有一個 OrderedDict 類來保留插入順序。 

儘管標準 dict 類型的順序性已經調整，但 OrderedDict 的性能特徵卻大不相同。 

如果您需要處理高速率的鍵插入和 popitem 調用（例如，實現最近最少使用的緩存），OrderedDict 可能比標準 Python dict 類型更適合。

### 自定義容器類型

Python 不是靜態類型的，所以大多數代碼對象的行為是它的事實上的類型，而不是嚴格的類層次結構。 這可能會導致令人驚訝的問題。

範例、小動物比賽投票

定義了一個函數來處理這些投票數據並將每個動物名稱的排名保存到提供的空字典中。

```python
votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 863, 
}

def populate_ranks(votes, ranks):
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i
```

需要一個函數來取得贏得比賽者，假設 populate_ranks 將按升序分配排名字典的內容，這意味著第一個鍵必須是獲勝者。

```python
def get_winner(ranks):
    return next(iter(ranks))

ranks = {}
populate_ranks(votes, ranks)
print(ranks)
winner = get_winner(ranks)
print(winner)
>>>
{'otter': 1, 'fox': 2, 'polar bear': 3}
otter
```

如果將按排名順序改由按排名順序，使用 collections.abc 內置模塊來定義一個新的類字典類，該類按字母順序迭代其內容。

```python
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}
    def __getitem__(self, key):
        return self.data[key]
    def __setitem__(self, key, value):
        self.data[key] = value
    def __delitem__(self, key):
        del self.data[key]
    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key
    def __len__(self):
        return len(self.data)
```

使用 SortedDict 實例代替具有之前功能的標準字典，並且不會引發錯誤，因為此類符合標準字典的協議。 但是，結果不正確。

```python
sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(sorted_ranks.data)
winner = get_winner(sorted_ranks)
print(winner)
>>>
{'otter': 1, 'fox': 2, 'polar bear': 3}
fox
```

問題原因是 get_winner 的實現假設字典的迭代是按照插入順序來匹配 populate_ranks。 此代碼使用 SortedDict 而不是 dict，因此該假設不再成立，有三種方法進行處理。

解法一：

只要重新實現 get_winner 函數，不再假設排名字典具有特定的迭代順序，此方法最保守也最穩健。

```python
def get_winner(ranks):
    for name, rank in ranks.items():
        if rank == 1:
            return name
winner = get_winner(sorted_ranks)
print(winner)
>>> otter
```

解法二：

在函數頂部添加一個顯式檢查，以確保排名類型符合我的期望，如果不符合則引發異常。 此解決方案可能比保守穩健有更好的運行性能。

```python
def get_winner(ranks):
    if not isinstance(ranks, dict):
        raise TypeError('must provide a dict instance')
    return next(iter(ranks))
get_winner(sorted_ranks)
>>>
Traceback ...
TypeError: must provide a dict instance
```

解法三：

第三種選擇是使用類型註釋來強制傳遞給 get_winner 的值是一個 dict 實例，而不是一個具有類似字典行為的 MutableMapping。 這裡以嚴格模式運行 mypy 工具。

```python
from typing import Dict, MutableMapping
def populate_ranks(votes: Dict[str, int],
                   ranks: Dict[str, int]) -> None:
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i
def get_winner(ranks: Dict[str, int]) -> str:
    return next(iter(ranks))
class SortedDict(MutableMapping[str, int]):
...
votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 863, 
}
sorted_ranks = SortedDict()
populate_ranks(votes, sorted_ranks)
print(sorted_ranks.data)
winner = get_winner(sorted_ranks)
print(winner)

$ python3 -m mypy --strict example.py
.../example.py:48: error: Argument 2 to "populate_ranks" has  ̄incompatible type "SortedDict"; expected "Dict[str, int]" 
.../example.py:50: error: Argument 1 to "get_winner" has  ̄incompatible type "SortedDict"; expected "Dict[str, int]"
```

這可以正確檢測 dict 和 MutableMapping 類型之間的不匹配，並將不正確的用法標記為錯誤。

該解決方案提供了靜態類型安全性和運行時性能的最佳組合

### 要記住的事情

1. 從 Python 3.7 開始，迭代 dict 實例的內容將按照最初添加鍵的順序。
2. Python 使定義像字典但不是字典實例的對象變得容易。 對於這些類型，您不能假設將保留插入順序。
3. 對於類字典類有三種注意方式：編寫不依賴於插入順序的代碼，在運行時顯式檢查 dict 類型，或使用類型註釋和靜態分析要求 dict 值。