# Effective Python 讀書筆記 Item 26-30

Created: July 7, 2022 6:53 PM  
Property: Ivan Kao  
Status: 已公開  
Tags: Python  

![Untitled](images/Untitled.png)

## Item 26: **Define Function Decorators with functools.wraps**

**用 functools.wraps 定義函數裝飾器**

Python 中有一種特殊的寫法，可以用裝飾器（decorator）來封裝某個函數，從而讓程式在執行這個函數之前與執行完這個函數之後，分別運行某些程式碼。
這意味著，調用者傳給函數的參數值、函數返回給調用者的值，以及函數拋出的異常，都可以由裝飾器訪問並修改。
這是個很有用的機制，能夠確保用戶以正確的方式使用函數，也能夠用來除錯程式或實現函數註冊功能，此外還有許多用途。

例如，假設我們要把函數執行時收到的參數與返回的值記錄下來。
這在除錯遞迴函數時是很有用的，因為我們需要知道，這個函數執行每一層遞迴時，輸入的是什麼參數，返回的是什麼值。

下面我們就定義這樣一個裝飾器，在實現這個裝飾器時，用 *args 與 **kwargs 表示受修飾的原函數 func 所收到的參數。

```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f'{func.__name__}({args!r}, {kwargs!r}) '
              f'-> {result!r}')
        return result
return wrapper
```

寫好之後，我們用 @ 符號把裝飾器運用在想要除錯的函數上面。

```python
@trace
def fibonacci(n):
    """Return the n-th Fibonacci number"""
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))
```

這樣寫，相當於先把受修飾的函數傳給裝飾器，然後將裝飾器所返回的值賦給原來那個函數。
這樣的話，如果我們繼續通過原來那個名字調用函數，那麼執行的就是修飾之後的函數。

```python
fibonacci = trace(fibonacci)
```

修飾過的 fibonacci 函數，會在執行自身的程式碼前，先執行 wrapper 裡位於 func(*args, **kwargs) 那一行之前的邏輯；並且在執行完自身的程式碼後，執行 wrapper 裡位於 func(*args, **kwargs) 那一行之後的邏輯。
本例中，它會在執行完自身的程式碼之後，打印出這次執行所用的參數與返回值，這樣就能看到整個遞迴的情況了。

```python
fibonacci(4)
>>>
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((1,), {}) -> 1
fibonacci((0,), {}) -> 0
fibonacci((1,), {}) -> 1
fibonacci((2,), {}) -> 1
fibonacci((3,), {}) -> 2
fibonacci((4,), {}) -> 3
```

這樣寫確實能滿足需求，但是會帶來一個我們不願意看到的副作用。裝飾器返回的那個值，也就是剛才調用的 fibonacci，它的名字並不叫 “fibonacci“。

```python
print(fibonacci)
>>>
<function trace.<locals>.wrapper at 0x108955dc0>
```

這種現象解釋起來並不困難。 trace 函數返回的，是它裡面定義的 wrapper 函數，所以，當我們把這個返回值賦給 fibonacci 之後，fibonacci 這個名稱所表示的自然就是 wrapper 了。
問題在於，這樣可能會干擾那些需要利用 introspection 機制來運作的工具，例如除錯器（debugger）。

例如，如果用內建的 help 函數來查看修飾後的 fibonacci，那麼打印出來的並不是我們想看的幫助文件，它本來應該打印前面定義時寫的那行 'Return the n-th Fibonacci number' 文字才對。

```python
help(fibonacci)
>>>
Help on function wrapper in module __main__:
wrapper(*args, **kwargs)
```

對象序列化器（object serializer）也無法正常運作，因為它不能確定受修飾的那個原始函數的位置。

```python
import pickle
pickle.dumps(fibonacci)
>>>
Traceback ...
AttributeError: Can't pickle local object 'trace.<locals>. wrapper'
```

要想解決這些問題，可以改用 functools 內建模組之中的 wraps 輔助函數來實現。
wraps 本身也是個裝飾器，它可以幫助你編寫自己的裝飾器。把它運用到 wrapper 函數上面，它就會將重要的元數據（metadata）全都從內部函數複製到外部函數。

```python
from functools import wraps
def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        ...
    return wrapper
@trace
def fibonacci(n):
...
```

對象序列化器，現在也正常了。

```python
help(fibonacci)
>>>
Help on function fibonacci in module __main__:
fibonacci(n)
    Return the n-th Fibonacci number
The pickle object serializer also works: print(pickle.dumps(fibonacci))
>>> b'\x80\x04\x95\x1a\x00\x00\x00\x00\x00\x00\x00\x8c\x08__main__\x94\x8c\tfibonacci\x94\x93\x94.'
```

除了這裡講到的幾個方面之外，Python 函數還有很多標準屬性（例如__name__、**module**、**annotations**）也應該在受到封裝時得以保留，這樣才能讓相關的接口正常運作。 wraps 可以幫助保留這些屬性，使程式表現出正確的行為。

### 需要記住的事項：

1. 裝飾器是 Python 中的一種寫法，能夠把一個函數封裝在另一個函數里面，這樣程式在執行原函數之前與執行完畢之後，就有機會執行其他一些邏輯了。
2. 裝飾器可能會讓那些利用 introspection 機制運作的工具（例如除錯器）產生奇怪的行為
3. Python 內建的 functools 模組裡有個叫作 wraps 的裝飾器，可以幫助我們正確定義自己的裝飾器，從而避開相關的問題。

# 4. Comprehensions and Generators

### 綜合運算式**與生成式**

我們經常需要處理串列（list）、含有鍵值對的字典（dict），以及集合（set）等資料結構，並且要以這種處理邏輯為基礎來架構程式。 Python 提供了一種特殊的寫法，叫作綜合運算式（comprehension），可以簡潔地迭代這些結構，並根據迭代結果派生出另一套資料。
這種寫法能夠讓我們用相當清晰的程式碼實現許多常見任務，而且還有一些其他好處。
Python 把這種理念也運用到了函數上面，這就產生了生成器（generator），它可以讓函數每次返回一系列值中的一個。
凡是可以使用迭代器的任務都支持生成器函數（例如循環，帶星號的表達式）等。
生成器可以提升性能並減少內存用量，還可以讓程式碼更容易讀懂。

## Item 27: **Use Comprehensions Instead of map and filter**

**用串列綜合運算取代 map 與 filter**

Python 裡面有一種很精簡的寫法，可以根據某個序列或可迭代對象建立出一份新的串列。用這種寫法寫成的表達式，叫作串列綜合運算（list comprehension）。
假設我們要用串列中每個元素的平方值建構一份新的串列。傳統的寫法是，採用下面這個簡單的 for 循環來實現。

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = []
for x in a:
    squares.append(x**2)
print(squares)
>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

上面那段程式碼可以改用串列綜合運算來寫，這樣可以在迭代串列的過程中，根據表達式算出新串列中的對應元素。

```python
squares = [x**2 for x in a]  # List comprehension
print(squares)
>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

這種功能當然也可以用內建函數 map 實現，它能夠從多個串列中分別取出當前位置上的元素，並把它們當作參數傳給映射函數，以求出新串列在這個位置上的元素值。
如果映射關係比較簡單（例如只是把一個串列映射到另一個串列），那麼用串列綜合運算來寫還是比用 map 簡單一些，因為用 map 的時候，必須先把映射邏輯定義成 lambda 函數，這看上去稍微有點煩瑣。

```python
alt = map(lambda x: x ** 2, a)
```

串列綜合運算還有一個地方比 map 好，就是它能夠方便地過濾原串列，把某些輸入值對應的計算結果從輸出結果中排除。
例如，假設新串列只需要納入原串列中那些偶數的平方值，那我們可以在綜合運算的時候，在 for x in a 這一部分的右側寫一個條件表達式

```python
even_squares = [x**2 for x in a if x % 2 == 0]
print(even_squares)
>>>
[4, 16, 36, 64, 100]
```

這種功能也可以通過內建的 filter 與 map 函數來實現，但是這兩個函數相結合的寫法要比串列綜合表達式難懂一些。

```python
alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
assert even_squares == list(alt)
```

字典與集合也有相應的綜合運算機制，分別叫作字典綜合運算（dictionary comprehension）與集合綜合運算（set comprehension）。
編寫算法時，可以利用這些機制根據原字典與原集合建立新字典與新集合。

```python
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}
print(even_squares_dict)
print(threes_cubed_set)
>>>
{2: 4, 4: 16, 6: 36, 8: 64, 10: 100}
{216, 729, 27}
```

如果改用 map 與 filter 實現，那麼還必須調用相應的建構式（constructor），這會讓程式碼變得很長，需要分成多行才能寫得下。
這樣看起來比較亂，不如使用綜合運算機制的程式碼清晰。

```python
alt_dict = dict(map(lambda x: (x, x**2),
                filter(lambda x: x % 2 == 0, a)))
alt_set = set(map(lambda x: x**3,
              filter(lambda x: x % 3 == 0, a)))
```

### 需要記住的事項：

1. 串列綜合運算要比內建的 map 與 filter 函數清晰，因為它不用另外定義 lambda 表達式。
2. 串列綜合運算可以很容易地跳過原串列中的某些資料，假如改用 map 實現，那麼必須搭配 filter 才能實現。
3. 字典與集合也可以通過綜合運算來建立。

## **Item 28: Avoid More Than Two Control Subexpressions in Comprehensions**

**控制綜合運算邏輯的子表達式不要超過兩個**

除了最基本的用法外，串列綜合運算還支持多層循環。
例如，要把矩陣（一種二維串列，它的每個元素本身也是串列）轉化成普通的一維串列，那麼可以在綜合運算時，
使用兩條 for 子表達式。這些子表達式會按照從左到右的順序解讀。

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)
>>>
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

這樣寫簡單易懂，這也正是多層循環在串列綜合運算之中的合理用法。多層循環還可以用來重製那種兩層深的結構。
例如，如果要根據二維矩陣裡每個元素的平方值建構一個新的二維矩陣，那麼可以採用下面這種寫法。
這看上去有點兒複雜，因為它把小的綜合運算邏輯 [x**2 for x in row] 嵌入到了大的綜合運算邏輯裡面，不過，這行語句總體上不難理解。

```python
squared = [[x**2 for x in row] for row in matrix]
print(squared)
>>>
[[1, 4, 9], [16, 25, 36], [49, 64, 81]]
```

如果綜合運算過程中還要再加一層循環，那麼語句就會變得很長，必須把它分成多行來寫，例如下面是把一個三維矩陣轉化成普通一維串列的程式碼。

```python
my_lists = [
    [[1, 2, 3], [4, 5, 6]],
    ...
]
flat = [x for sublist1 in my_lists
        for sublist2 in sublist1
        for x in sublist2]
```

在這種情況下，採用串列綜合運算來實現，其實並不會比傳統的 for 循環節省多少程式碼。

下面用常見的 for 循環改寫剛才的例子。這次我們通過級別不同的縮進表示每一層循環的深度，這要比剛才那種三層矩陣的串列綜合運算更加清晰。

```python
flat = []
for sublist1 in my_lists:
    for sublist2 in sublist1:
        flat.extend(sublist2)
```

綜合運算的時候，可以使用多個 if 條件。如果這些 if 條件出現在同一層循環內，那麼它們之間默認是 and 關係，也就是必須同時成立。
例如，如果要用原列表中大於 4 且是偶數的值來建構新串列，那麼既可以連用兩個 if，也可以只用一個 if ，下面兩種寫法效果相同。

```python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0]
```

在綜合運算時，每一層的 for 子表達式都可以帶有 if 條件。
例如，要根據原矩陣建構新的矩陣，把其中各元素之和大於等於 10 的那些行選出來，而且只保留其中能夠被 3 整除的那些元素。
這個邏輯用串列綜合運算來寫，並不需要太多的程式碼，但是這些程式碼理解起來會很困難。

```python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
filtered = [[x for x in row if x % 3 == 0]
            for row in matrix if sum(row) >= 10]
print(filtered)
>>>
[[6], [9]]
```

雖然這個例子是專門構造出來的，但在實際中確實會碰到很多類似的需求。
強烈建議不要使用上方複雜的綜合運算新的 list、dict 或 set，因為這樣寫出來的程式碼很難讓初次閱讀這段程式的人看懂。
對於 dict 來說，問題尤其嚴重，因為必須得把鍵與值的綜合運算邏輯都表達出來才行。

總之，在表示綜合運算邏輯時，最多只應該寫兩個子表達式（例如兩個 if 條件、兩個 for 循環，或者一個 if 條件與一個 for 循環）。只要實現的邏輯比這還復雜，那就應該採用普通的 if 與 for 語句來實現，並且可以考慮編寫輔助函數。

### 需要記住的事項：

1. 綜合運算的時候可以使用多層循環，每層循環可以帶有多個條件。
2. 控制綜合運算邏輯的子表達式不要超過兩個，否則程式碼很難讀懂。

## **Item 29: Avoid Repeated Work in Comprehensions by Using Assignment Expressions**

**用賦值表達式消除綜合運算中的重複程式碼**

綜合運算 list 、 dict 與 set 等變體結構時，經常要在多個地方用到同一個計算結果。

例如，我們要給製作緊固件的公司編寫程序以管理訂單。

顧客下單後，我們要判斷當前的庫存能否滿足這份訂單，也就是說，要核查每種產品的數量有沒有達到可以發貨的最低限制（8個為一批，至少要有一批，才能發貨）。

```python
stock = {
    'nails': 125,
    'screws': 35,
    'wingnuts': 8,
    'washers': 24,
}
order = ['screws', 'wingnuts', 'clips']
def get_batches(count, size):
    return count // size
result = {}
for name in order:
  count = stock.get(name, 0)
  batches = get_batches(count, 8)
  if batches:
    result[name] = batches
print(result)
>>>
{'screws': 4, 'wingnuts': 1}
```

這段循環邏輯，如果改用字典綜合運算來寫，會簡單一些。

```python
found = {name: get_batches(stock.get(name, 0), 8)
         for name in order
         if get_batches(stock.get(name, 0), 8)}
print(found)
>>>
{'screws': 4, 'wingnuts': 1}
```

這樣寫雖然比剛才簡短，但問題是，它把 get_batches(stock.get(name,0), 8) 寫了兩遍。

這樣會讓程式碼看起來比較亂，而且實際上，程式也沒有必要把這個結果計算兩遍。

另外，如果這兩個地方忘了同步更新，那麼程式就會出現 bug。

```python
has_bug = {name: get_batches(stock.get(name, 0), 4)
           for name in order
           if get_batches(stock.get(name, 0), 8)}
print('Expected:', found)
print('Found:   ', has_bug)
>>>
Expected: {'screws': 4, 'wingnuts': 1}
Found:    {'screws': 8, 'wingnuts': 2}
```

有個簡單的辦法可以解決這個問題，那就是在綜合運算的過程中使用 Python 3.8 新引入的 := 操作符進行賦值表達。

```python
found = {name: batches for name in order
         if (batches := get_batches(stock.get(name, 0), 8))}
```

這條 batches := get_batches(...) 賦值表達式，能夠從 stock 字典裡查到對應產品一共有幾批，並把這個批數放在 batches 變量裡。

這樣的話，我們綜合運算這個產品所對應批數時，就不用再通過 get_batches 計算了，因為結果已經保存到 batches 裡面了。這種寫法只需要把 get 與 get_batches 調用一次即可，這樣能夠提升效率。

在綜合運算過程中，描述新值的那一部分也可以出現賦值表達式。但如果在其他部分引用了定義在那一部分的變量，那麼程式可能就會在運行時出錯。例如，如果寫成了下面這樣，那麼程式就必須先考慮 for name, count in stock.items() if tenth > 0，而這個時候，其中的 tenth 還沒有得到定義。

```python
result = {name: (tenth := count // 10)
          for name, count in stock.items() if tenth > 0}
>>>
Traceback ...
NameError: name 'tenth' is not defined
```

為了解決這個問題，可以把賦值表達式移動到 if 條件裡面，然後在描述新值的這一部分引用已經定義過的tenth變量。

```python
result = {name: tenth for name, count in stock.items()
          if (tenth := count // 10) > 0}
print(result)
>>>
{'nails': 12, 'screws': 3, 'washers': 2}
```

如果綜合運算邏輯不帶條件，而表示新值的那一部分又使用了 := 操作符，那麼操作符左邊的變量就會洩漏到包含這條綜合運算語句的那個作用域裡。

```python
half = [(last := count // 2) for count in stock.values()]
print(f'Last item of {half} is {last}')
>>>
Last item of [62, 17, 4, 12] is 12
```

這與普通的for循環所用的那個循環變量類似。

```python
for count in stock.values():  # Leaks loop variable
    pass
print(f'Last item of {list(stock.values())} is {count}')
>>>
Last item of [125, 35, 8, 24] is 24
```

然而，綜合運算語句中的for循環所使用的循環變量，是不會像剛才那樣洩漏到外面的。

```python
half = [count // 2 for count in stock.values()]
print(half)   # Works
print(count)  # Exception because loop variable didn't leak
>>>
[62, 17, 4, 12]
Traceback ...
NameError: name 'count' is not defined
```

最好不要洩漏循環變量，所以，建議賦值表達式只出現在綜合運算邏輯的條件之中。

賦值表達式不僅可以用在綜合運算過程中，而且可以用來編寫生成器表達式（generator expression）。

下面這種寫法創建的是迭代器，而不是字典實例，該迭代器每次會給出一對數值，其中第一個元素為產品的名字，第二個元素為這種產品的庫存。

```python
found = ((name, batches) for name in order
         if (batches := get_batches(stock.get(name, 0), 8)))
print(next(found))
print(next(found))
>>>
('screws', 4)
('wingnuts', 1)
```

### 需要記住的事項：

1. 編寫綜合運算式與生成器表達式時，可以在描述條件的那一部分通過賦值表達式定義變量，並在其他部分複用該變量，可使程式簡單易讀。
2. 對於綜合運算式與生成器表達式來說，雖然賦值表達式也可以出現在描述條件的那一部分之外，但最好別這麼寫。

## **Item 30: Consider Generators Instead of Returning Lists**

**不要讓函數直接返回串列，應該讓它逐個生成串列裡的值**

```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result
```

我們把一段範例文字傳給這個函數，它可以返回正確的結果。

```python
address = 'Four score and seven years ago...'
result = index_words(address)
print(result[:10])
>>>
[0, 5, 11, 15, 21, 27, 31, 35, 43, 51]
```

index_words 函數有兩個缺點：
第一個缺點是，它的程式碼看起來有點雜亂。每找到一個新單詞，它都要調用append方法，而調用這個方法時，必須寫上 result.append 這樣一串字符，
這就把我們想要強調的重點，也就是這個新單詞在字符串中的位置（index + 1）淡化了。

另外，函數還必須專門用一行程式碼創建這個保存結果的 result 串列，並且要用一條 return 語句把它返回給調用者。
這樣算下來，雖然函數的主體部分大約有 130 個字符（非空白的），但真正重要的只有 75 個左右。這種函數改用生成器（generator）來實現會比較好。生成器由包含 yield表 達式的函數創建。

下面就定義一個生成器函數，實現與剛才那個函數相同的效果。

```python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1
```

調用生成器函數並不會讓其中的程式碼立刻得到執行，它會返回一個迭代器（iterator）。
把迭代器傳給 Python 內建的 next 函數，就可以將生成器函數推進到它的下一條 yield 表達式。
生成器會把 yield 表達式的值通過迭代器返回給調用者。

```python
it = index_words_iter(address)
print(next(it))
print(next(it))
>>>
0
5
```

這次的 index_words_iter 函數，比剛才那個函數好懂得多，因為它把涉及串列的那些操作全都簡化掉了。
它通過 yield 表達式來傳遞結果，而不像剛才那樣，要把結果追加到串列之中。
如果確實要製作一份串列，那可以把生成器函數返回的迭代器傳給內建的 list 函數。

```python
result = list(index_words_iter(address))
print(result[:10])
>>>
[0, 5, 11, 15, 21, 27, 31, 35, 43, 51]

```

index_words 函數的第二個缺點是，它必須把所有的結果都保存到串列中，然後才能返回串列。
如果輸入的資料特別多，那麼程式可能會因為耗盡內存而崩潰。
相反，用生成器函數來實現，就不會有這個問題了。它可以接受長度任意的輸入訊息，並把內存消耗量壓得比較低。

例如下面這個生成器，只需要把當前這行文字從文件中讀進來就行，每次推進的時候，它都只處理一個單詞，直到把當前這行文字處理完畢，才讀入下一行文字。

```python
def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset
```

該函數運行時所耗的內存，取決於文件中最長的那一行所包含的字符數。
把剛才那份輸入資料存放到 address.txt 文件，讓這個函數去讀取並用它所返回的生成器建立一份串列，可以看到跟原來相同的結果。

```python
with open('address.txt', 'r') as f:
    it = index_file(f)
    results = itertools.islice(it, 0, 10)
    print(list(results))
>>>
[0, 5, 11, 15, 21, 27, 31, 35, 43, 51]

```

定義這種生成器函數的時候，只有一個地方需要注意，就是調用者無法重複使用函數所返回的迭代器，因為這些迭代器是有狀態的。

### 需要記住的事項：

1. 用生成器來實現比讓函數把結果收集合到串列裡再返回，要更加清晰一些。
2. 生成器函數所返回的迭代器可以產生一系列值，每次產生的那個值都是由函數體的下一條 yield 表達式所決定的。
3. 不管輸入的資料量有多大，生成器函數每次都只需要根據其中的一小部分來計算當前這次的輸出值。它不用把整個輸入值全都讀取進來，也不用一次就把所有的輸出值全都算好。