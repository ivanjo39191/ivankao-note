# Effective Python 讀書筆記 Item 6 - 10

Created: March 5, 2022 12:57 PM
Property: Ivan Kao
Status: 已公開
Tags: Python

# 1. Pythonic Thinking

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

## **Item 7: Prefer enumerate Over range**

優先使用超出範圍的 enumerate(枚舉)

### 使用 iterate 迭代

使用 range 函數迭代一組整數

```python
from random import randint
random_bits = 0
for i in range(32):
    if randint(0, 1):
        random_bits |= 1 << i
print(bin(random_bits))
>>>
0b11101000100100000111000010000001
```

迭代 list

```python
flavor_list = ['vanilla', 'chocolate', 'pecan', 'strawberry']
for flavor in flavor_list:
    print(f'{flavor} is delicious')
>>>
vanilla is delicious
chocolate is delicious
pecan is delicious
strawberry is delicious
```

使用 range 函數迭代 list 的索引

```python
for i in range(len(flavor_list)):
    flavor = flavor_list[i]
    print(f'{i + 1}: {flavor}')

>>>
1: vanilla
2: chocolate
3: pecan
4: strawberry
```

使用以上方法進行迭代，顯得有點笨拙，要先取得 list 的長度，且要使用索引調用，最後變得難以閱讀。

### 使用 Enumerate (枚舉) 內置函數

Python 提供了 enumerate 內置函數來解決這種情況。

enumerate 使用 lazy generator (惰性生成器)包裝迭代器。

enumerate 產生成對的循環索引(index, value)和給定迭代器的下一個值(next)。

範例、

```python
it = enumerate(flavor_list)
print(next(it))
print(next(it))
>>>
(0, 'vanilla')
(1, 'chocolate')
```

使用 for 解包

```python
for i, flavor in enumerate(flavor_list):
    print(f'{i + 1}: {flavor}')
>>>
1: vanilla
2: chocolate
3: pecan
4: strawberry
```

指定 enumerate 應該開始計數的數字

```python
for i, flavor in enumerate(flavor_list, 1):
    print(f'{i}: {flavor}')
```

### 要記住的事情

1. enumerate 提供了簡潔的語法，用於在迭代器上循環並取得索引。
2. 優先使用 enumerate 而不使用 range 取得索引。
3. enumerate 可選項第二個參數來指定從哪個數字開始計數（默認值為零）。

## **Item 8: Use zip to Process Iterators in Parallel**

使用 zip 並行處理迭代器

### List 生成式

使用 list 生成式取得 list

```python
names = ['Cecilia', 'Lise', 'Marie']
counts = [len(n) for n in names]
print(counts)
>>>
[7, 4, 5]
```

通過其索引可取得原始 list 的項目。

並行使用兩個列表，可以同時取得原始 list 的項目名稱與長度。

```python
names = ['Cecilia', 'Lise', 'Marie']
counts = [len(n) for n in names]
longest_name = None
max_count = 0

for i in range(len(names)):
    count = counts[i]
    if count > max_count:
        longest_name = names[i]
        max_count = count
print(longest_name)
>>>
Cecilia
```

整個循環語句看起來很雜亂。 名稱和計數的索引使程式碼難以閱讀。 

通過循環索引 i 對數組進行索引發生兩次。 若使用 enumerate 能改善但仍然不理想。

```python
names = ['Cecilia', 'Lise', 'Marie']
counts = [len(n) for n in names]
longest_name = None
max_count = 0

for i, name in enumerate(names):
    count = counts[i]
    if count > max_count:
        longest_name = name
        max_count = count
```

### Zip 內置函數

zip 使用惰性生成器包裝兩個或多個迭代器。

zip 生成器從每個迭代器生成包含下一個值的元組，這些元組可以直接在 for 語句中解包。

zip 一次包裝一個項目的迭代器，意味它可以與無限長的輸入一起使用，而不會有程序使用過多內存和崩潰的風險。

範例、

```python
names = ['Cecilia', 'Lise', 'Marie']
counts = [len(n) for n in names]
longest_name = None
max_count = 0

for name, count in zip(names, counts):
    if count > max_count:
        longest_name = name
        max_count = count
```

當輸入迭代器的長度不同時，它只會執行到一個被包裝的迭代器用完為止。

```python
names.append('Rosalind')
for name, count in zip(names, counts):
print(name)
>>>
Cecilia
Lise
Marie
```

它的輸出與最短的輸入一樣長。當您知道迭代器的長度相同時，Zip 方法才可以正常工作。

### itertools.zip_longest 函數

許多其他情況下，zip 的截斷行為令人驚訝且很糟糕。如果您不希望傳遞給 zip 的列表長度相等，請考慮使用 itertools 內置模塊中的 zip_longest 函數。

```python
import itertools
for name, count in itertools.zip_longest(names, counts):
    print(f'{name}: {count}')

>>>
Cecilia: 7
Lise: 4
Marie: 5
Rosalind: None
zip_longest replaces missing values—the length of the string 'Rosalind' in this case—with whatever fillvalue is passed to it, which defaults to None.
```

### 要記住的事情

1. zip 內置函數可用於並行迭代多個迭代器。
2. zip 創建了一個生成元組的惰性生成器，可以用於無限長的輸入。
3. 如果提供不同了長度的迭代器，zip 會截斷為最短的迭代器。
4. 如果要在長度不等的迭代器上使用 zip 而不截斷，請使用 itertools 內置模塊中的 zip_longest 函數。

## Item 9: Avoid else Blocks After for and while Loops

避免在 for 和 while 循環之後使用 else 

### 在循環後面使用 else

Python 循環有一個額外的特性，可以在之後立即放置一個 else ，它會在循環結束時執行。

```python
for i in range(3):
    print('Loop', i)
else:
    print('Else block!')
>>>
Loop 0
Loop 1
Loop 2
Else block!
```

在循環中使用 break 語句上會跳過 else 

```python
for i in range(3):
    print('Loop', i)
    if i == 1:
        break
else:
    print('Else block!')
```

如果循環空序列，會立即運行

```python
for x in []:
    print('Never runs')
else:
    print('For Else block!')
>>>
While Else block!
```

循環最初為 False 時，會運行 else

```python
while False:
    print('Never runs')
else:
    print('While Else block!')
>>>
While Else block!
```

這些行為的基本原理是循環後的 else 塊在使用循環搜索某些內容時很有用。

例如，假設我想確定兩個數字是否互質。

1. 遍歷每個可能的公約數並測試數字。
2. 嘗試了每個選項後，循環結束。
3. 若循環沒有遇到中斷，則運行 else 來表示數字互質

```python
def coprime(a, b):
    for i in range(2, min(a, b) + 1):
        if a % i == 0 and b % i == 0:
            return False
   return True
assert coprime(4, 9)
assert not coprime(3, 6)
```

### 使用 help function 取代

盡量不使用 循環後 else 的方式編寫程式碼。而會編寫一個輔助函數來進行計算。

這樣的輔助函數以兩種常見的風格寫成。

1. 當發現正在尋找的情況時儘早返回。如果通過循環會返回默認結果

```python
def coprime(a, b):
    for i in range(2, min(a, b) + 1):
        if a % i == 0 and b % i == 0:
            return False
    return True

assert coprime(4, 9)
assert not coprime(3, 6)
```

1. 在循環設一個結果變量來判斷是否找到了目標。發現時就跳出循環。

```python
def coprime_alternate(a, b):
    is_coprime = True
    for i in range(2, min(a, b) + 1):
        if a % i == 0 and b % i == 0:
            is_coprime = False
            break
    return is_coprime

assert coprime_alternate(4, 9)
assert not coprime_alternate(3, 6)
```

對於不熟悉程式碼的維護者來說，這兩種方法都更加清晰。

而若使用 循環後 else 造成閱讀程式碼的負擔將會超過所想表達的內容。

應該完全避免在循環之後使用 else 。

### 要記住的事情

1. Python 有特殊的語法，允許 else 緊跟在 for 和 while 循環內部塊之後。
2. 循環後的 else 僅在循環體沒有遇到 break 語句時運行。
3. 避免在循環之後使用 else ，因為它們的行為不直觀並且可能令人困惑。

## Item 10: Prevent Repetition with Assignment Expressions

使用賦值表達式防止重複

## 賦值表達式

賦值表達式是 Python 3.8 中引入的一種新語法，用於解決可能導致代碼重複的語言長期存在的問題。

賦值表達式很有用，因為它們使您能夠在不允許賦值語句的地方分配變量，例如在 if 語句的條件表達式中。 賦值表達式的值等於分配給賦值運算符左側標識符的任何值。

### 使用賦值表達式判斷

範例、

假設我有一籃新鮮水果

```python
fresh_fruit = {
    'apple': 10,
    'banana': 8,
    'lemon': 5, 
}
```

使用 if 查找檸檬的數量來判斷是否為非零值

```python
def make_lemonade(count):
    ...
def out_of_stock():
    ...
count = fresh_fruit.get('lemon', 0)
if count:
    make_lemonade(count)
else:
    out_of_stock()
```

這個看似簡單的代碼的問題在於看起來很雜亂。 count 變量僅在 if 語句的第一個塊中使用。

這種獲取一個值，檢查它是否非零，然後使用它的模式在 Python 中非常常見。 許多程序員嘗試使用各種損害可讀性的技巧來繞過多個引用來計數。 現在可以使用賦值表達式來精確地簡化這種類型的代碼。 

```python
if count := fresh_fruit.get('lemon', 0):
    make_lemonade(count)
else:
    out_of_stock()
```

雖然這只是短了一行，但它更具可讀性，因為現在很清楚 count 只與 if 語句的第一個塊相關。

### 使用賦值表達式進行比較

為了確保至少有四個蘋果。 這裡通過從 fruit_basket 字典中獲取計數，然後在 if 語句條件表達式中進行比較

```python
def make_cider(count):
    ...
count = fresh_fruit.get('apple', 0)
if count >= 4:
    make_cider(count)
else:
    out_of_stock()
```

使用賦值表達式進行比較

```python
if (count := fresh_fruit.get('apple', 0)) >= 4:
    make_cider(count)
else:
    out_of_stock()
```

這裡需要注意需使用括號以便與 4 進行比較，與其他表達式一樣，應該盡可能避免用括號括起賦值表達式。

### 在封閉範圍使用賦值表達式

範例、

現在要確保有兩個香蕉的切片，否則會引發 OutOfBananas 異常。 

```python
# 使用兩個香蕉切片
def slice_bananas(count):
    ...
class OutOfBananas(Exception):
    pass
def make_smoothies(count):
    ...
```

以兩種方式來實現這個邏輯。

```python
# 方法一
pieces = 0
count = fresh_fruit.get('banana', 0)
if count >= 2:
    pieces = slice_bananas(count)
try:
    smoothies = make_smoothies(pieces)
except OutOfBananas:
out_of_stock()

# 方法二
count = fresh_fruit.get('banana', 0)
if count >= 2:
    pieces = slice_bananas(count)
else:
    pieces = 0
try:
    smoothies = make_smoothies(pieces)
except OutOfBananas:
    out_of_stock()
```

使用賦值表達式

```python
# 方法一
pieces = 0
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
try:
    smoothies = make_smoothies(pieces)
except OutOfBananas:
out_of_stock()

# 方法二
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
else:
    pieces = 0
try:
    smoothies = make_smoothies(pieces)
except OutOfBananas:
    out_of_stock()

```

使用賦值運算符還提高了在 if 語句的兩個部分中拆分片段定義的可讀性。

### 使用賦值表達式進行多重判斷

不熟悉 Python 的程序員經常遇到的一個挫折是缺乏靈活的 switch/case 語句。 近似此類功能的一般風格是深度嵌套多個 if、elif 和 else 語句。

範例、

```python
count = fresh_fruit.get('banana', 0)
if count >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
else:
    count = fresh_fruit.get('apple', 0)
    if count >= 4:
        to_enjoy = make_cider(count)
    else:
        count = fresh_fruit.get('lemon', 0)
        if count:
            to_enjoy = make_lemonade(count)
        else:
to_enjoy‘= 'Nothing'
```

像這樣的醜陋結構在 Python 代碼中非常常見。 使用賦值運算符提供了一個優雅的解決方案，感覺幾乎與 switch/case 語句的專用語法一樣通用。

```python
if (count := fresh_fruit.get('banana', 0)) >= 2:
    pieces = slice_bananas(count)
    to_enjoy = make_smoothies(pieces)
elif (count := fresh_fruit.get('apple', 0)) >= 4:
    to_enjoy = make_cider(count)
elif count := fresh_fruit.get('lemon', 0):
    to_enjoy = make_lemonade(count)
else:
    to_enjoy = 'Nothing'
```

由於嵌套和縮進的減少，可讀性的提高是巨大的。 如果您在代碼中看到如此醜陋的結構，我建議您盡可能將它們轉移到使用賦值運算符。

### 使用賦值表達式進行 while 迴圈判斷

範例、使用迴圈裝果汁直到沒有水果剩餘

```python
def pick_fruit():
    ...
def make_juice(fruit, count):
    ...
    bottles = []
    fresh_fruit = pick_fruit()
    while fresh_fruit:
        for fruit, count in fresh_fruit.items():
            batch = make_juice(fruit, count)
            bottles.extend(batch)
            fresh_fruit = pick_fruit()
```

這是重複的，因為它需要兩個單獨的 fresh_fruit = pick_fruit()

在這種情況下改進代碼重用的一種策略是使用半循環語法。 

這消除了多餘的行，但它也破壞了 while 循環，使其成為一個糟糕的無限循環。 循環的所有流控制都依賴於條件break語句

```python
bottles = []
while True: # Loop
    fresh_fruit = pick_fruit()
    if not fresh_fruit: # And a half
        break
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)
```

通過賦值運算符允許重新分配 fresh_fruit 變量，然後每次通過 while 循環進行條件判斷。 

這個解決方案不需使用半循環語法，且簡短易讀。

```python
bottles = []
while fresh_fruit := pick_fruit():
    for fruit, count in fresh_fruit.items():
        batch = make_juice(fruit, count)
        bottles.extend(batch)
```

還有許多其他情況可以使用賦值表達式來消除冗餘。

當發現多次重複相同的表達式或賦值時，就該考慮使用賦值表達式以提高可讀性了

### 要記住的事情

1. 賦值表達式使用賦值運算符 := 在單個表達式中分配和計算變量名，從而減少重複。
2. 當賦值表達式是較大表達式的子表達式時，必須用括號括起來。
3. 雖然 switch/case 語句和 do/while 循環在 Python 中不可用，但使用賦值表達式可以更清楚地模擬它們的功能。