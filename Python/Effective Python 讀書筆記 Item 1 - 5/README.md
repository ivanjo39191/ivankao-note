# Effective Python 讀書筆記 Item 1 - 5

Created: February 21, 2022 11:30 AM
Property: Ivan Kao
Status: 已公開
Tags: Python

# 1. Pythonic Thinking

## **Item 1: Know Which Version of Python You’re Using**

### 查看版本

Linux 查看 Python 版本

```jsx
python --version
python3 --version
```

Python 中查看版本資訊

```jsx
import sys
print(sys.version_info)
print(sys.version)
```

### 需要記住的事項：

1. Python3 是最新的、支援最完善，應使用它。
2. 確保系統上cmd執行的Python 是自己期望的版本。
3. 避免使用 Python2，因它在2020/01/01後不再維護

## **Item 2: Follow the PEP 8 Style Guide**

遵循PEP 8風格指南，使用一致的樣式使您的程式更易於閱讀。 

PEP 8提供了有關如何編寫清晰的Python程式的大量詳細資訊。 隨著Python語言的發展，它持續更新。 值得線上閱讀整個指南（https://www.python.org/dev/peps/pep-0008/）。 

以下是一些需確保遵守的規則：

### Whitespace 空格

在Python中，空格具有句法意義。以下與空白相關的準則：

1. 使用空格不使用 tab 進行縮排。
2. 使用 4 個空格縮排。
3. 行長度應為79個字元或以下。
4. 長表達式的其他排應使用四個額外空格進行縮排。
5. Function 和 Class 應使用兩行空行分隔。
6. 在 Class 中的 Function 應使用一行空行分隔
7. 在字典中，key, value 之間不留空格。不同相應值在同一行時，相應值前要空一格。
8. 在變數賦值時， = 前後要放置一個空格。
9. 對於型別註釋，變數名稱與冒號間沒有分離，型別資訊前使用一格空格。

### Naming 命名

PEP 8為語言的不同部分提供了獨特的命名風格。 這些慣例使區分哪種型別變得容易

函式、變數和屬性應採用小寫下劃線格式。

■受保護的例項屬性應為_leading_underscore格式。

■私有例項屬性應採用__double_leading_ underscore格式。

■類（包括例外）應採用大駝峰格式。

■模組級常量應為全大寫格式。

■類中的例項方法應使用self，這指的是物件，作為第一個引數的名稱。

■類方法應使用引用類的cls作為第一個引數的名稱。

### 表達式和語句

Python 之禪說：“There should be one—and preferably only one—obvious way to do it.”

PEP 8 試圖在其表達式和語句的指南中編纂這種風格：

■ 使用內聯否定(if a is not b)代替肯定表達式的否定(if not a is b)。

```python
# Correct:
if foo is not None:

# Wrong:
if not foo # 必須清楚 foo 等於None, False, 空字串"", 0, 空列表[], 空字典{}, 空元組()時對你的判斷沒有影響才可使用
if not foo is None:
```

■ 不要通過將長度與零進行比較(if len(somelist) == 0)來檢查空容器或序列（如 [] 或 ''）。使用 if not somelist 並假設空值將隱式評估為 False。

■ 非空容器或序列（如[1] 或'hi'）也是如此。對於非空值，if somelist 的語句隱式為 True。

■ 避免使用單行if 語句、for 和while 循環以及except 複合語句。為了清楚起見，將它們分佈在多行上。

■ 如果您無法在一行中放置一個表達式，請用括號將其括起來並添加換行符和縮進以使其更易於閱讀。

■ 與使用\ 行繼續符相比，更喜歡用括號包圍多行表達式。

### Import

■ 始終將 import 語句（包括 from x import y）放在文件的頂部。

■ 導入模塊時始終使用絕對名稱，而不是相對於當前模塊自身路徑的名稱。 例如，要從 bar 包中導入 foo 模塊，您應該使用 from bar import foo，而不僅僅是 import foo。

■ 如果必須進行相對導入，請使用 . 進口富。

■ 導入應按以下順序分段：標準庫模塊、第三方模塊、您自己的模塊。 每個小節都應按字母順序導入。

### Note

Pylint 工具 ([https://www.pylint.org](https://www.pylint.org/)) 是一種流行的 Python 源代碼靜態分析器。 Pylint 提供 PEP 8 樣式指南的自動執行，並檢測 Python 程序中許多其他類型的常見錯誤。 許多 IDE 和編輯器還包括 linting 工具或支持類似的插件。

### 需要記住的事項：

1. 在編寫 Python 代碼時，請始終遵循 Python Enhancement Proposal #8 (PEP 8) 樣式指南。
2.  與更大的 Python 社區分享共同的風格有助於與他人合作。
3.  使用一致的樣式，以後修改自己的代碼更容易。

## Item 3: Know the Differences Between bytes and str

在 Python 中，有兩種表示字符數據序列的類型：bytes 和 str。 字節實例包含原始的、無符號的 8 位值（通常以 ASCII 編碼顯示）：

```python
a = b'h\x65llo'
print(list(a))
print(a)
>>>
[104, 101, 108, 108, 111]
b'hello'
```

str 的實例包含表示人類語言文本字符的 Unicode 代碼點：

```python
a = 'a\u0300 propos'
print(list(a))
print(a)
>>>
['a', '`', ' ', 'p', 'r', 'o', 'p', 'o', 's']
à propos
```

str 實例沒有關聯的二進制編碼，而 bytes 實例沒有關聯的文本編碼。

要將 Unicode 數據轉換為二進制數據，必須調用 str 的 encode 方法。要將二進制數據轉換為 Unicode 數據，必須調用字 bytes 的 decode 方法。

字符類型之間的拆分導致 Python 代碼中的兩種常見情況：

■ 您希望對包含 UTF-8 編碼字符串（或某些其他編碼）的原始 8 位序列進行操作。

■ 您希望對沒有特定編碼的 Unicode 字符串進行操作。

1. 接受一個 bytes 或 str 實例並且總是返回一個 str

```python
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value  # Instance of str

print(repr(to_str(b'foo')))
print(repr(to_str('bar')))
>>>
'foo'
'bar'
```

1. 接受一個 bytes 或 str 實例並且總是返回一個 bytes

```python
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value  # Instance of bytes
print(repr(to_bytes(b'foo')))
print(repr(to_bytes('bar')))
```

使用 + 運算符可以 bytes + bytes 或 str + str ，但不能混用。

使用二元運算符可以 bytes 與 bytes 或str 與 str 比，但不能混用，結果總是為 False。

% 運算符分別使用每種類型的格式字符串，但不能傳遞 bytes 進入 str 中，因為 Python 不知道要使用哪種二進製文本編碼。

寫入文件，’w’ 為文本模式，‘wb’ 為二進制模式

```python
with open('data.bin', 'w') as f:
	f.write(b'\xf1\xf2\xf3\xf4\xf5')
>>>
Traceback ...

with open('data.bin', 'wb') as f:
    f.write(b'\xf1\xf2\xf3\xf4\xf5')
```

讀取文件，’r’ 為文本模式，‘rb’ 為二進制模式

```python
with open('data.bin', 'r') as f:
    data = f.read()
>>>
Traceback ...

with open('data.bin', 'rb') as f:
    data = f.read()
assert data == b'\xf1\xf2\xf3\xf4\xf5'
```

讀取文件，指定編碼

```python
with open('data.bin', 'r', encoding='cp1252') as f:
    data = f.read()
assert data == 'ñòóôõ'
```

您應該檢查系統上的默認編碼（使用 python3 -c 'import locale; print(locale.getpreferredencoding())'）以了解它與您的期望有何不同。 如有疑問，應以明確傳遞編碼參數的方式打開。

### 需要記住的事項：

1. bytes 包含 8 位值序列，str 包含 Unicode 編碼點序列。
2. 使用輔助函數確保您操作的輸入是您期望的字符序列類型（8 位值、UTF-8 編碼字符串、Unicode 代碼點等）。
3. bytes 和 str 實例不能與運算符（如 >、==、+ 和 %）一起使用。
4. 如果您想從文件中讀取或寫入二進制數據，請始終使用二進制模式（如“rb”或“wb”）打開文件
5. 如果您想在文件中讀取或寫入 Unicode 數據，請注意系統的默認文本編碼。 如果您想避免意外，請以明確傳遞編碼參數的方式打開。

## Item 4: Prefer Interpolated F-Strings Over C-style Format Strings and str.format

格式化是將預定義文本與數據值組合成單個人類可讀消息並存儲為字符串的過程。
Python 有四種不同的格式化字符串的方法 —— C-style、內置函式、str.format、f-strings，

這些方法內置於語言和標準庫中。 除了本項目最後介紹的其中一個之外，所有這些都具有您應該理解和避免的嚴重缺點。

### C-style format strings (C 風格的格式字符串)

格式字符串使用格式說明符（如 %d）作為佔位符，將被格式表達式右側的值替換。 

此方法有 4 個問題：

1. 右側元組更改數據值的類型或順序可能會因類型轉換不兼容而出錯。或是右側元組保留，但更改格式字符串會導致錯誤，為避免此錯誤需要不斷檢查兩側，因為需要手動更改而容易出錯。
2. 當您需要在將值格式化為字符串之前對值進行修改時，會變得難以閱讀。
3. 如果想要格式化字符串中多次使用相同的值，必須在右側重複使用它。
4. 若使用字典可以解決 #1 #3 問題，但也會增加冗長，因為每個鍵必須至少指定兩次——一次在格式說明符中，一次在字典中作為鍵，並且可能再一次用於包含字典值的變量名。

```python
menu = {
 'soup': 'lentil',
 'oyster': 'kumamoto',
 'special': 'schnitzel',
}
template = ('Today\'s soup is %(soup)s, '
 'buy one get two %(oyster)s oysters, '
 'and our special entrée is %(special)s.')
formatted = template % menu
print(formatted)
>>>
Today's soup is lentil, buy one get two kumamoto oysters, and
➥our special entrée is schnitzel.
```

### The format Built-in and str.format (內置格式和 str.format)

Python 3 增加了對高級字符串格式的支持比使用 % 運算符的舊 C 樣式格式字符串更具表現力。

```python
# , 千位分隔符和 ^ 居中
a = 1234.5678
formatted = format(a, ',.2f')
print(formatted)
b = 'my string'
formatted = format(b, '^20s')
print('*', formatted, '*')
>>>
1,234.57
* my string *

```

通過調用 str 類型的新格式方法，可以將多個值一起格式化，默認情況下，格式字符串中的佔位符將替換為按其出現順序傳遞給格式方法的相應位置參數。

```python
key = 'my_var'
value = 1.234
formatted = '{} = {}'.format(key, value)
print(formatted)
>>>
my_var = 1.234
```

在每個佔位符中，您可以選擇提供一個冒號字符，後跟格式說明符，以自定義如何將值轉換為字符串。

```python
formatted = '{:<10} = {:.2f}'.format(key, value)
print(formatted)
>>>
my_var = 1.23
```

在大括號內，您還可以指定傳遞給格式方法以用於替換佔位符的參數的位置索引。(解決 C-style 問題  #1)

相同的位置索引也可以在格式字符串中被多次引用，而無需多次將值傳遞給格式方法。(解決 C-style 問題 #3)

```python
formatted = '{1} = {0}'.format(key, value)
print(formatted)
>>>
1.234 = my_var
```

str.format 方法一起使用的說明符還有更高級的選項，例如在佔位符中使用字典鍵和列表索引的組合，以及將值強制轉換為 Unicode 和 repr 字符串。

但是  str.format 無法解決 C-style 問題 #2 ，在格式化之前對值進行小幅修改時，會使您的代碼難以閱讀。更高級選項也無助於減少  C-style 問題 #4 中重複鍵的冗餘。

 str.format 有許多缺點，例如高級特性只能供 Python 表達市的一小部分，缺乏表現力且有許多限制，以至於破壞了格式化方法的價值。

以上這些缺點和 C-style 表達式的問題仍然存在(#2 #4)，建議避免使用 str.format。

了解格式說明符（冒號後的所有內容）中使用的新迷你語言以及如何使用 format built-in function (格式內置函數)非常重要。 但是 str.format 方法的其餘部分應該被視為歷史文物。

### Interpolated Format Strings (插值格式字符串)

Python 3.6 添加了插值格式字符串（F-strings）來一勞永逸地解決這些問題。

這種新的語言語法要求您使用 f 字符作為格式字符串的前綴，F-strings 將格式字符串的表現力發揮到了極致，通過完全消除提供要格式化的鍵和值的冗餘來解決上面的問題 #4。

它們通過允許您引用當前 Python 範圍內的所有名稱作為格式化表達式的一部分來實現這種簡潔性。

```python
key = 'my_var'
value = 1.234
formatted = f'{key} = {value}'
print(formatted)
>>>
my_var = 1.234
```

新格式內置迷你語言中的所有相同選項都可以在 f 字符串中佔位符中的冒號之後使用，以及將值強制轉換為 Unicode 和 repr 字符串的能力，類似於 str.format 方法

```python
formatted = f'{key!r:<10} = {value:.2f}'
print(formatted)
>>>
'my_var' = 1.23
```

在所有情況下，使用 f 字符串格式化都比使用帶有 % 運算符和 str.format 方法的 C 樣式格式字符串短。

按照最短到最長的順序將所有這些選項一起顯示，並在作業的左側排列，以便您輕鬆比較它們

```python
f_string = f'{key:<10} = {value:.2f}'
c_tuple = '%-10s = %.2f' % (key, value)
str_args = '{:<10} = {:.2f}'.format(key, value)
str_kw = '{key:<10} = {value:.2f}'.format(key=key,
                                          value=value)
c_dict = '%(key)-10s = %(value).2f' % {'key': key,
                                       'value': value}
assert c_tuple == c_dict == f_string
assert str_args == str_kw == f_string
```

F-strings 還使您能夠將完整的 Python 表達式放在佔位符大括號內，通過允許對使用簡潔語法格式化的值進行小的修改來解決上面的問題 #2。

使用 C 樣式格式和 str.format 方法需要多行的內容現在很容易適合單行

```python
f_string = f'#{i+1}: {item.title():<10s} = {round(count)}'
```

Python 表達式也可能出現在格式說明符選項中。

```python
places = 3
number = 1.23456
print(f'My number is {number:.{places}f}')
>>>
My number is 1.235
```

F-strings 提供的表現力、簡潔性和清晰性的結合使它們成為 Python 程序員的最佳內置選項。

每當您發現自己需要將值格式化為字符串時，請選擇 f-strings 而不是替代品。

### 需要記住的事項：

1.  使用 % 運算符的 C-style 格式字符串會遇到各種陷阱和冗長問題。
2. str.format 方法在其格式化說明符 mini 語言中引入了一些有用的概念，但它會重複 C-style 格式字符串的錯誤，應避免。
3. F-strings 是一種用於將值格式化為字符串的新語法，它解決了 C-style 格式字符串的最大問題。
4. F-strings 簡潔而強大，因為它們允許將任意 Python 表達式直接嵌入到格式說明符中。

## Item 5: Write Helper Functions Instead of Complex Expressions

編寫輔助函數來取代複雜表達式

Python 簡潔的語法使編寫實現大量邏輯的單行表達式變得容易。

### or 複雜表達式

當第一個子表達式為 False 時，下面的表達式將計算為 or 運算符之後的子表達式

```python
from urllib.parse import parse_qs

# For query string 'red=5&blue=0&green='
red = my_values.get('red', [''])[0] or 0
green = my_values.get('green', [''])[0] or 0
opacity = my_values.get('opacity', [''])[0] or 0
print(f'Red: {red!r}')
print(f'Green: {green!r}')
print(f'Opacity: {opacity!r}')
>>>
Red: '5'
Green: 0
Opacity: 0

# parse the string as an integer
red = int(my_values.get('red', [''])[0] or 0)
```

這現在非常難以閱讀。程式碼看起來非常不親切，新的讀者將不得不花費太多時間來分解表達式來弄清楚它實際上做了什麼。

儘管保持簡短是件好事，但不值得嘗試將所有內容都放在一條線上。

### if/else 表達式

Python 具有 if/else 條件（或三元）表達式，可以使此類情況更清晰，同時保持程式碼簡短。

```python
red_str = my_values.get('red', [''])
red = int(red_str[0]) if red_str[0] else 0
```

對於不太複雜的情況，if/else 條件表達式可以讓事情變得非常清晰。

但是上面的例子仍然不像多行完整的 if/else 語句那樣清晰

```python
green_str = my_values.get('green', [''])
if green_str[0]:
    green = int(green_str[0])
else:
    green = 0
```

如果你需要重複使用這個邏輯——甚至只是兩到三遍，就像這個例子一樣——那麼編寫一個輔助函數就是要走的路。

### 輔助函數

```python
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
      return int(found[0])
  	return default
```

呼叫用的程式碼比使用 or 的複雜表達式和使用 if/else 表達式的兩行版本要清晰得多。

```python
green = get_first_int(my_values, 'green')
```

一旦表達式變得複雜，就要考慮將它們分成更小的部分並將邏輯移動到輔助函數中。

在可讀性方面獲得的收益總是超過簡潔可能給您帶來的收益，避免讓 Python 用於復雜表達式的簡潔語法讓您陷入這樣的混亂。

遵循 DRY 原則：不要重複自己。

### 需要記住的事項

1. Python 的語法使編寫過於複雜且難以閱讀的單行表達式變得容易。
2. 將複雜的表達式移動到輔助函數中，尤其是當您需要重複使用相同的邏輯時。
3. if/else 表達式為在表達式中使用布爾運算符 or 和 and 提供了一種更易讀的替代方法。

## **Item 6: Prefer Multiple Assignment Unpacking Over Indexing**

優先使用多指派而非索引

### Tuple 簡介

Python 有一個內置的 tuple 類型，可用於創建不可變的、有序的值序列。

在最簡單的情況下，元組是一對兩個值，例如字典中的鍵和值

```python
snack_calories = {
    'chips': 140,
    'popcorn': 80,
    'nuts': 190,
}
items = tuple(snack_calories.items())
print(items)
>>>
(('chips', 140), ('popcorn', 80), ('nuts', 190))
```

tuple 中的值可以通過數字索引訪問

```python
item = ('Peanut butter', 'Jelly')
first = item[0]
second = item[1]
print(first, 'and', second)
>>>
Peanut butter and Jelly
```

創建 tuple 後，您無法通過為索引分配新值來修改它

```python
pair = ('Chocolate', 'Peanut butter')
pair[0] = 'Honey'
>>>
Traceback ...
TypeError: 'tuple' object does not support item assignment
```

### Unpacking （解包）

Python 還具有用於解包的語法，它允許在單個語句中分配多個值。

您在解包分配中指定的模式看起來很像試圖改變 tuple ——這是不允許的——但它們實際上工作方式完全不同。

例如，如果您知道一個元組是一對，而不是使用索引來訪問它的值，您可以將它分配給兩個變量名的元組

```python
item = ('Peanut butter', 'Jelly')
first, second = item  # Unpacking
print(first, 'and', second)
>>>
Peanut butter and Jelly
```

與訪問 tuple 的索引相比，解包具有更少的容易閱讀，並且通常需要更少的行。

在分配給列表、序列和多層級可迭代對象時，也可使用相同方式進行解包。

我不建議在您的代碼中執行以下操作，但重要的是要知道它是可能的以及它是如何工作的：

```python
favorite_snacks = {
    'salty': ('pretzels', 100),
    'sweet': ('cookies', 180),
    'veggie': ('carrots', 20),
}
((type1, (name1, cals1)),
 (type2, (name2, cals2)),
 (type3, (name3, cals3))) = favorite_snacks.items()
print(f'Favorite {type1} is {name1} with {cals1} calories')
print(f'Favorite {type2} is {name2} with {cals2} calories')
print(f'Favorite {type3} is {name3} with {cals3} calories')
>>>
Favorite salty is pretzels with 100 calories
Favorite sweet is cookies with 180 calories
Favorite veggie is carrots with 20 calories
```

解包甚至可以用於在原地交換值，而無需創建臨時變量。

### 使用 Unpacking 進行值互換

在這裡，我使用帶有索引的典型語法來交換列表中兩個位置之間的值，作為升序排序算法的一部分

```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i-1]:
                temp = a[i]
                a[i] = a[i-1]
                a[i-1] = temp
names = ['pretzels', 'carrots', 'arugula', 'bacon']
bubble_sort(names)
print(names)
>>>
['arugula', 'bacon', 'carrots', 'pretzels']
```

使用解包

```python
def bubble_sort(a):
    for _ in range(len(a)):
        for i in range(1, len(a)):
            if a[i] < a[i-1]:
                a[i-1], a[i] = a[i], a[i-1]  # Swap
```

這種交換的工作方式是

1. 賦值的右側 (a[i], a[i-1])，然後將其值放入一個新的臨時 tuple 中（例如 ('carrots '，'pretzels'）在循環的第一次迭代中）。
2. 賦值左側的解包語法 (a[i-1], a[i]) 用於接收該 tuple 值並將其分配給變量名 a[i-1] 和 a[i] ， 這將索引 0 處的“pretzels”替換為“carrots”，索引 1 處的 “carrots” 替換為 ”pretzels”。最後，臨時 tuple 默默地消失了。

### 使用 Unpacking 進行迴圈

解包的另一個有價值的應用是在 for 循環和類似構造的目標列表中，例如推導和生成器表達式。 作為對比的一個例子，這裡我在不使用拆包的情況下迭代了一個零食列表

```python
snacks = [('bacon', 350), ('donut', 240), ('muffin', 190)]
for i in range(len(snacks)):
    item = snacks[i]
    name = item[0]
    calories = item[1]
    print(f'#{i+1}: {name} has {calories} calories')
>>>
#1: bacon has 350 calories
#2: donut has 240 calories
#3: muffin has 190 calories
```

這行得通，但它很雜亂。 為了索引到零食結構的各個級別，需要很多額外的字符。 

通過使用 unpacking 和 enumerate 內置函數來實現相同的輸出

```python
for rank, (name, calories) in enumerate(snacks, 1):
    print(f'#{rank}: {name} has {calories} calories')
>>>
#1: bacon has 350 calories
#2: donut has 240 calories
#3: muffin has 190 calories
```

這是編寫這種循環的 Pythonic 方式； 它簡短易懂。 通常不需要使用索引來訪問任何東西。

Python 為列表構造、函數參數、關鍵字參數、多個返回值等等提供了額外的解包功能。

明智地使用解包將使您能夠盡可能避免索引，從而產生更清晰和更 Pythonic 的代碼。

### 要記住的事情

1. Python 有一種特殊的語法，稱為 Unpacking（解包），用於在單個語句中分配多個值。
2. 解包在 Python 中是通用的，可以應用於任何可迭代對象，包括可迭代對像中的許多級別的可迭代對象。
3. 通過使用解包來減少視覺雜亂並提高程式碼清晰度，以避免顯式索引到序列中。