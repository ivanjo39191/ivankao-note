# Effective Python 讀書筆記 Item 21-25

Created: June 18, 2022 1:43 PM  
Property: Ivan Kao  
Status: 已公開  
Tags: Python  

![Untitled](images/Untitled.png)

## Item 21: Know How Closures Interact with Variable Scope

**了解如何在閉包裡面使用外圍作用域中的變量**

有時，我們要給串列中的元素排序，而且要優先把某個群組之中的元素放在其他元素的前面。

例如，渲染用戶界面時，可能就需要這樣做，因為關鍵的消息和特殊的事件應該優先顯示在其他資訊之前。

### 閉包函數原理

實現這種做法的一種常見方案，是把輔助函數通過 key 參數傳給串列的 sort 方法，讓這個方法根據輔助函數所返回的值來決定元素在列表中的先後順序。

輔助函數先判斷當前元素是否處在重要群組裡，如果在，就把返回值的第一項寫成 0 ，讓它能夠排在不屬於這個組的那些元素之前。

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)
```

可以處理簡單的輸入資料

```python
numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = {2, 3, 5, 7}
sort_priority(numbers, group)
print(numbers)
>>>
[2, 3, 5, 7, 1, 4, 6, 8]
```

它為什麼能夠實現這個功能呢？
這要分三個原因來講：Python 支持閉包（closure），這讓定義在大函數里面的小函數也能引用大函數之中的變量。
具體到這個例子，sort_priority函數里面的那個 helper 函數也能夠引用前者的 group 參數。
函數在 Python 裡是頭等對象（first-class object），所以你可以像操作其他對像那樣，直接引用它們、把它們賦給變量、將它們當成參數傳給其他函數，或是在 in 表達式與 if 語句裡面對它做比較，等等。

閉包函數也是函數，所以，同樣可以傳給 sort 方法的 key 參數。 Python 在判斷兩個序列（包括元組）的大小時，有自己的一套規則。它首先比較0號位置的那兩個元素，如果相等，那就比較 1 號位置的那兩個元素；如果還相等，那就比較 2 號位置的那兩個元素；依此類推，直到得出結論為止。所以，我們可以利用這套規則讓helper這個閉包函數返回一個元組，並把關鍵指標寫為元組的首個元素以表示當前排序的值是否屬於重要群組（ 0 表示屬於，1 表示不屬於）。

如果這個 sort_priority 函數還能告訴我們，串列裡面有沒有位於重要群組之中的元素，那就更好了，因為這樣可以讓用戶界面開發者更方便地做出相應處理。

添加這樣一個功能似乎相當簡單，因為閉包函數本身就需要判斷當前值是否處在重要群組之中，既然這樣，那麼不妨讓它在發現這種值時，順便把標誌變量翻轉過來。最後，讓閉包外的大函數（即 sort_priority 函數）返回這個標誌變量，如果閉包函數當時遇到過這樣的值，那麼這個標誌肯定是 True。

### 在閉包函數賦值外圍作用域的變量

下面，試著用最直接的寫法來實現。

```python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        if x in group:
            found = True  # Seems simple
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```

排序結果沒有問題，可以看到：在排過序的 numbers 裡面，重要群組 group 裡的那些元素（2、3、5、7），確實出現在了其他元素前面。既然這樣，那表示函數返回值的 found 變量就應該是True，但我們看到的卻是 False。

這是為什麼？在表達式中引用某個變量時，Python解釋器會按照下面的順序，在各個作用域（scope）裡面查找這個變量，以解析（resolve）這次引用。

1. 當前函數的作用域。
2. 外圍作用域（例如包含當前函數的其他函數所對應的作用域）。
3. 包含當前代碼的那個模塊所對應的作用域（也叫全局作用域，globalscope）。
4. 內置作用域（built-in scope，也就是包含len與str等函數的那個作用域）。

如果這些作用域中都沒有定義名稱相符的變量，那麼程序就拋出NameError異常。

```python
foo = does_not_exist * 5
>>>
Traceback ...
```

剛才講的是怎麼引用變量，現在我們來講怎麼給變量賦值，這要分兩種情況處理。

1. 如果變量已經定義在當前作用域中，那麼直接把新值交給它就行了。
2. 如果當前作用域中不存在這個變量，那麼即便外圍作用域裡有同名的變量，Python 也還是會把這次的賦值操作當成變量的定義來處理，這會產生一個重要的結果：Python 會把包含賦值操作的這個函數當成新定義的這個變量的作用域。

這可以解釋剛才那種寫法錯在何處：

sort_priority2 函數里面的 helper 閉包函數是把 True 賦給了 found 變量。
當前作用域裡沒有這樣一個叫作 found 的變量，所以就算外圍的 sort_priority2 函數里面有found變量，系統也還是會把這次賦值當成定義，也就是會在 helper 裡面定義一個新的 found 變量，而不是把它當成給 sort_priority2 已有的那個 found 變量賦值。

```python
def sort_priority2(numbers, group):
    found = False         # Scope: 'sort_priority2'
    def helper(x):
        if x in group:
            found = True  # Scope: 'helper' -- Bad!
            return (0, x)
return (1, x)
```

這種問題有時也稱作作用域 bug（scoping bug），Python 新手可能認為這樣的賦值規則很奇怪，但實際上 Python 是故意這麼設計的。因為這樣可以防止函數中的局部變量污染外圍模塊。

假如不這樣做，那麼函數里的每條賦值語句都有可能影響全局作用域的變量，這樣不僅混亂，而且會讓全局變量之間彼此交互影響，從而導致很多難以探查的 bug。

### 使用 nonlocal

Python 有一種特殊的寫法，可以把閉包裡面的數據賦給閉包外面的變量。用 nonlocal 語句描述變量，就可以讓系統在處理針對這個變量的賦值操作時，去外圍作用域查找。然而，nonlocal 有個限制，就是不能侵入模塊級別的作用域（以防污染全局作用域）。

下面，用 nonlocal 改寫剛才那個函數。

```python
def sort_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found  # Added
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found
```

nonlocal 語句清楚地表明，我們要把資料賦給閉包之外的變量。

有一種跟它互補的語句，叫作 global，用這種語句描述某個變量後，在給這個變量賦值時，系統會直接把它放到模塊作用域（或者說全局作用域）中。

我們都知道全局變量不應該濫用，其實 nonlocal 也這樣。除比較簡單的函數外，大家盡量不要用這個語句，因為它造成的副作用有時很難發現。
尤其是在那種比較長的函數里，nonlocal 語句與其關聯變量的賦值操作之間可能隔得很遠。

### 使用輔助類來封裝狀態

如果 nonlocal 的用法比較複雜，那最好是改用輔助類來封裝狀態。下面就定義了這樣一個類，用來實現與剛才那種寫法相同的效果。這樣雖然稍微長一點，但看起來更清晰易讀。

```python
class Sorter:
    def __init__(self, group):
        self.group = group
        self.found = False
    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)
```

### 需要記住的事項：

1. 閉包函數可以引用定義它們的那個外圍作用域之中的變量。
2. 按照默認的寫法，在閉包裡面給變量賦值並不會改變外圍作用域中的同名變量。
3. 先用nonlocal語句說明，然後賦值，可以修改外圍作用域中的變量。
4. 除特別簡單的函數外，盡量少用 nonlocal 語句。

## Item 22: **Reduce Visual Noise with Variable Positional
Arguments**

**用數量可變的位置參數給函數設計清晰的參數列表**

讓函數接受數量可變的位置參數（positional argument），可以把函數設計得更加清晰（這些位置參數通常簡稱 varargs ，或者叫作 star args ，因為我們習慣用 *args ）。例如，假設我們要記錄 debug 資料。如果採用參數數量固定的方案來設計，那麼函數應該接受一個表示資料的 message 參數和一個 values 串列（這個串列用於存放需要放入到資料裡的那些值）。

```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')
log('My numbers are', [1, 2])
log('Hi there', [])
>>>
My numbers are: 1, 2
Hi there
```

即便沒有值需要放入到資料裡面，也必須傳一個空白的串列進去，這樣顯得多餘，而且讓程式碼會看起來比較雜亂。

最好是能允許調用者把第二個參數留空。在 Python 裡，可以給最後一個位置參數加前綴 *，這樣調用者就只需要提供不帶星號的那些參數，然後可以不再指其他參數，也可以繼續指定任意數量的位置參數。

```python
def log(message, *values):  # The only difference
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{message}: {values_str}')
log('My numbers are', 1, 2)
log('Hi there')  # Much better
>>>
My numbers are: 1, 2
Hi there
```

這種寫法與拆解資料時用在賦值語句左邊帶星號的 unpacking 操作非常類似。

如果想把已有序列（例如某列表）裡面的元素當成參數傳給像 log 這樣的參數個數可變的函數（variadic function），那麼可以在傳遞序列的時採用 * 操作符。

這會讓 Python 把序列中的元素都當成位置參數傳給這個函數。

```python
favorites = [7, 33, 99]
log('Favorite colors', *favorites)
>>>
Favorite colors: 7, 33, 99
```

令函數接受數量可變的位置參數，可能導致兩個問題。

第一個問題是，程序總是必須先把這些參數轉化成一個元組，然後才能把它們當成可選的位置參數傳給函數。

這意味著，如果調用函數時，把帶 * 操作符的生成器傳了過去，那麼程序必須先把這個生成器裡的所有元素迭代完（以便形成元組），然後才能繼續往下執行。

這個元組包含生成器所給出的每個值，這可能耗費大量內存，甚至會讓程式崩潰。

```python
def my_generator():
    for i in range(10):
        yield i
def my_func(*args):
    print(args)
it = my_generator()
my_func(*it)
>>>
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
```

接受 *args 參數的函數，適合處理輸入值不太多，而且數量可以提前預估的情況。

在調用這種函數時，傳給 *args 這一部分的應該是許多個字面值或變量名才對。這種機制，主要是為了讓程式碼寫起來更方便、讀起來更清晰。

第二個問題是，如果用了 *args 之後，又要給函數添加新的位置參數，那麼原有的調用操作就需要全都更新。

例如給參數列表開頭添加新的位置參數 sequence ，那麼沒有據此更新的那些調用程式碼就會出錯。

```python
def log(sequence, message, *values):
    if not values:
        print(f'{sequence} - {message}')
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f'{sequence} - {message}: {values_str}')
log(1, 'Favorites', 7, 33)      # New with *args OK
log(1, 'Hi there')              # New message only OK
log('Favorite numbers', 7, 33)  # Old usage breaks
>>>
1 - Favorites: 7, 33
1 - Hi there
Favorite numbers - 7: 33
```

問題在於：第三次調用log函數的那個地方並沒有根據新的參數列表傳入 sequence 參數，所以 'Favorite numbers' 就成了 sequence 參數， 7 就成了 message 參數。

這樣的 bug 很難排查，因為程式不會拋出異常，只會採用錯誤的資料繼續運行下去。

為了徹底避免這種漏洞，在給這種 *arg 函數添加參數時，應該使用只能通過關鍵字來指定的參數）。要是想做得更穩一些，可以考慮添加類型註解。

### 需要記住的事項：

1. 用 def 定義函數時，可以通過 *args 的寫法讓函數接受數量可變的位置參數。
2. 調用函數時，可以在序列左邊加上 * 操作符，把其中的元素當成位置參數傳給 *args 所表示的這一部分。
3. 如果 * 操作符加在生成器前，那麼傳遞參數時，程式有可能因為耗盡內存而崩潰。
4. 給接受 *args 的函數添加新位置參數，可能導致難以排查的bug。

## **Item 23: Provide Optional Behavior with Keyword Arguments**

**用關鍵字參數來表示可選的行為**

與大多數其他程式語言一樣，Python 允許在調用函數時，按照位置傳遞參數。

```python
def remainder(number, divisor):
    return number % divisor
assert remainder(20, 7) == 6
```

Python 函數裡面的所有普通參數，除了按位置傳遞外，還可以按關鍵字傳遞。

調用函數時，在調用括號內可以把關鍵字的名稱寫在 = 左邊，把參數值寫在右邊。

這種寫法不在乎參數的順序，只要把必須指定的所有位置參數全都傳過去即可。另外，關鍵字形式與位置形式也可以混用。下面這四種寫法的效果相同：

```python
remainder(20, 7)
remainder(20, divisor=7)
remainder(number=20, divisor=7)
remainder(divisor=7, number=20)
```

如果混用，那麼位置參數必須出現在關鍵字參數之前，否則就會出錯。

```python
remainder(number=20, 7)
>>>
Traceback ...
SyntaxError: positional argument follows keyword argument
```

每個參數只能指定一次，不能既通過位置形式指定，又通過關鍵字形式指定。

```python
remainder(20, number=7)
>>>
Traceback ...
TypeError: remainder() got multiple values for argument  ̄'number'
```

如果有一個字典，而且字典裡面的內容能夠用來調用 remainder 這樣的函數，

那麼可以把 **運算符加在字典前面，這會讓 Python 把字典裡面的鍵值以關鍵字參數的形式傳給函數。

```python
my_kwargs = {
    'number': 20,
    'divisor': 7,
}
assert remainder(**my_kwargs) == 6
```

調用函數時，帶**操作符的參數可以和位置參數或關鍵字參數混用，只要不重複指定就行。

```python
my_kwargs = {
    'divisor': 7,
}
assert remainder(number=20, **my_kwargs) == 6
```

也可以對多個字典分別施加**操作，只要這些字典所提供的參數不重疊就好。

```python
my_kwargs = {
    'number': 20,
}
other_kwargs = {
    'divisor': 7,
}
assert remainder(**my_kwargs, **other_kwargs) == 6
```

定義函數時，如果想讓這個函數接受任意數量的關鍵字參數，那麼可以在參數列表裡寫上萬能形參 **kwargs，它會把調用者傳進來的參數收集合到一個字典裡面稍後處理。

```python
def print_parameters(**kwargs):
    for key, value in kwargs.items():
        print(f'{key} = {value}')
print_parameters(alpha=1.5, beta=9, gamma=4)
>>>
alpha = 1.5
beta = 9
gamma = 4
```

關鍵字參數的靈活用法可以帶來三個好處:
第一個好處是，用關鍵字參數調用函數可以讓初次閱讀程式碼的人更容易看懂。例如，讀到 remainder(20, 7) 這樣的調用程式碼，就不太容易看出誰是被除數 number，誰是除數 divisor，只有去查看 remainder 的具體實現方法才能明白。
但如果改用關鍵字形式來調用，例如 remainder(number=20, divisor=7)，那麼每個參數的含義就相當明了。

關鍵字參數的第二個好處是，它可以帶有默認值，該值是在定義函數時指定的。在大多數情況下，調用者只需要沿用這個值就好，但有時也可以明確指定自己想要傳的值。
這樣能夠減少重複程式碼，讓程式看上去乾淨一些。
例如，我們要計算液體流入容器的速率。如果這個容器帶刻度，那麼可以取前後兩個時間點的刻度差（weight_diff），並把它跟這兩個時間點的時間差（time_diff）相除，就可以算出流速了。

```python
def flow_rate(weight_diff, time_diff):
    return weight_diff / time_diff
weight_diff = 0.5
time_diff = 3
flow = flow_rate(weight_diff, time_diff)
print(f'{flow:.3} kg per second')
>>>
0.167 kg per second
```

一般來說，我們會用每秒的千克數表示流速。但有的時候，我們還想估算更長的時間段（例如幾小時或幾天）內的流速結果。只需給同一個函數加一個period參數來表示那個時間段相當於多少秒即可。

```python
def flow_rate(weight_diff, time_diff, period):
    return (weight_diff / time_diff) * period
```

這樣寫有個問題，就是每次調用函數時，都得明確指定 period 參數。即便我們想計算每秒鐘的流速，也還是得明確指定 period 為 1。

```python
flow_per_second = flow_rate(weight_diff, time_diff, 1)
```

為了簡化這種用法，我們可以給 period 參數設定默認值。

```python
def flow_rate(weight_diff, time_diff, period=1):
    return (weight_diff / time_diff) * period
```

這樣的話，period 就變成可選參數了。

```python
flow_per_second = flow_rate(weight_diff, time_diff)
flow_per_hour = flow_rate(weight_diff, time_diff, period=3600)
```

這個辦法適用於默認值比較簡單的情況。如果默認值本身要根據比較複雜的邏輯來確定，那就得仔細考慮一下了。

關鍵字參數的第三個好處是，我們可以很靈活地擴充函數的參數，而不用擔心會影響原有的函數調用程式碼。
這樣的話，我們就可以通過這些新參數在函數里面實現許多新的功能，同時又無須修改早前寫好的調用程式碼，這也讓程式不容易因此出現 bug。
例如，我們想繼續擴充上述 flow_rate 函數的功能，讓它可以用千克之外的其他重量單位來計算流速。那隻需要再添加一個可選參數，用來表示千克相當於多少個那樣的單位即可。

```python
def flow_rate(weight_diff, time_diff,
              period=1, units_per_kg=1):
    return ((weight_diff * units_per_kg) / time_diff) * period
```

新參數 units_per_kg 的默認值為 1，這表示默認情況下，依然以千克為重量單位來計算。
於是，以前寫好的那些調用代碼就不用修改了。

以後調用 flow_rate時，可以通過關鍵字形式給這個參數指定值，以表示他們想用的那種單位（例如磅，1千克約等於2.2磅）。

```python
pounds_per_hour = flow_rate(weight_diff, time_diff,
                            period=3600, units_per_kg=2.2)
```

可選的關鍵字參數有助於維護向後兼容（backward compatibility）。這是個相當重要的問題，對於接受帶 *args參數的函數，也要注意向後兼容。

像 period 和 units_per_kg 這樣可選的關鍵字參數，只有一個缺點，就是調用者仍然能夠按照位置來指定。

```python
pounds_per_hour = flow_rate(weight_diff, time_diff, 3600, 2.2)
```

通過位置來指定可選參數，可能會讓讀程式碼的人有點兒糊塗，因為他不太清楚 3600 和 2.2 這兩個值分別指哪個量的縮放係數。

所以，最好是能以關鍵字的形式給這些參數傳值，而不要按位置去傳。從設計函數的角度來說，還可以考慮用更加明確的方案以降低出錯概率。

### 需要記住的事項：

1. 函數的參數可以按位置指定，也可以用關鍵字的形式指定。
2. 關鍵字可以讓每個參數的作用更加清楚，因為在調用函數時只按位置指定參數，可能導致這些參數的含義不夠明確。
3. 應該通過帶默認值的關鍵字參數來擴展函數的行為，因為這不會影響原有的函數調用代碼。
4. 可選的關鍵字參數總是應該通過參數名來傳遞，而不應按位置傳遞。

## **Item 24: Use None and Docstrings to Specify Dynamic
Default Arguments**

**用 None 和 docstring 來描述默認值會變的參數**

有時，我們想把那種不能夠提前固定的值，當作關鍵字參數的默認值。
例如，記錄日誌消息時，默認的時間應該是觸發事件的那一刻。
所以，如果調用者沒有明確指定時間，那麼就默認把調用函數的那一刻當成這條日誌的記錄時間。
現在試試下面這種寫法，假定它能讓 when 參數的默認值隨著這個函數每次的執行時間而發生變化。

```python
from time import sleep
from datetime import datetime
def log(message, when=datetime.now()):
    print(f'{when}: {message}')
log('Hi there!')
sleep(0.1)
log('Hello again!')
>>>
2019-07-06 14:06:15.120124: Hi there!
2019-07-06 14:06:15.120124: Hello again!
```

這樣寫不行。因為 datetime.now 只執行了一次，所以每條日誌的時間戳（timestamp）相同。

參數的默認值只會在系統加載這個模塊的時候，計算一遍，而不會在每次執行時都重新計算，這通常意味著這些默認值在程式啟動後，就已經定下來了。
只要包含這段程式碼的那個模塊已經加載進來，那麼 when 參數的默認值就是加載時計算的那個 datetime.now() ，系統不會重新計算。

要想在 Python 裡實現這種效果，慣用的辦法是把參數的默認值設為None，同時在 docstring 文件裡面寫清楚，這個參數為 None 時，函數會怎麼運作）。
給函數寫實現程式碼時，要判斷該參數是不是 None ，如果是，就把它改成相應的默認值。

```python
def log(message, when=None):
    """Log a message with a timestamp.
    Args:
        message: Message to print.
        when: datetime of when the message occurred.
            Defaults to the present time.
    """
    if when is None:
        when = datetime.now()
print(f'{when}: {message}')
```

這次，兩條日誌的時間戳就不同了。

```python
log('Hi there!')
sleep(0.1)
log('Hello again!')
>>>
2019-07-06 14:06:15.222419: Hi there!
2019-07-06 14:06:15.322555: Hello again!
```

參數的默認值寫成 None 還有個重要的意義，就是用來表示那種以後可能由調用者修改內容的默認值（例如某個可變的容器）。

例如，我們要寫一個函數對採用 JSON 格式編碼的資料做解碼。如果無法解碼，那麼就返回調用時所指定的默認結果，假如調用者當時沒有明確指定，那就返回空白的字典。

```python
import json
def decode(data, default={}):
    try:
        return json.loads(data)
    except ValueError:
        return default
```

這樣的寫法與前面 datetime.now 的例子有著同樣的問題。
系統只會計算一次 default 參數（在加載這個模塊的時候），所以每次調用這個函數時，給調用者返回的都是一開始分配的那個字典，這就相當於凡是以默認值調用這個函數的程式碼都共用同一份字典。這會使程式出現很奇怪的效果。

```python
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
>>>
Foo: {'stuff': 5, 'meep': 1}
Bar: {'stuff': 5, 'meep': 1}
```

我們本意是想讓這兩次調用操作得到兩個不同的空白字典，每個字典都可以分別用來存放不同的鍵值。
但實際上，只要修改其中一個字典，另一個字典的內容就會受到影響。
這種錯誤的根源在於，foo 和 bar 實際上是同一個字典，都等於系統一開始給 default 參數確定默認值時所分配的那個字典。它們表示的是同一個字典對象。

```python
assert foo is bar
```

要解決這個問題，可以把默認值設成 None，並且在 docstring 文件裡面說明，函數在這個值為 None 時會怎麼做。

```python
def decode(data, default=None):
    """Load JSON data from a string.
    Args:
        data: JSON data to decode.
        default: Value to return if decoding fails.
            Defaults to an empty dictionary.
    """
    try:
        return json.loads(data)
    except ValueError:
        if default is None:
            default = {}
        return default
```

這樣寫，再運行剛才那段測試程式碼，就可以得出預期的結果了。

```python
foo = decode('bad data')
foo['stuff'] = 5
bar = decode('also bad')
bar['meep'] = 1
print('Foo:', foo)
print('Bar:', bar)
assert foo is not bar
>>>
Foo: {'stuff': 5}
Bar: {'meep': 1}
```

這個思路可以跟類型註解搭配起來。下面這種寫法把 when 參數標註成可選（Optional）值，並限定其類型為 datetime 。於是，它的取值就只有兩種可能，只會是 None，或是 datetime 對象。

```python
from typing import Optional
def log_typed(message: str,
              when: Optional[datetime]=None) -> None:
    """Log a message with a timestamp.
    Args:
        message: Message to print.
        when: datetime of when the message occurred.
            Defaults to the present time.
    """
    if when is None:
        when = datetime.now()
print(f'{when}: {message}')
```

### 需要記住的事項：

1. 參數的默認值只會計算一次，也就是在系統把定義函數的那個模塊加載進來的時候。所以，如果默認值將來可能由調用方修改（例如{}、[]）或者要隨著調用時的情況變化（例如 datetime.now() ），那麼程式就會出現奇怪的效果。
2. 如果關鍵字參數的默認值屬於這種會發生變化的值，那就應該寫成None，並且要在 docstring 裡面描述函數此時的默認行為。
3. 默認值為 None 的關鍵字參數，也可以添加類型註解。

## **Item 25: Enforce Clarity with Keyword-Only and
Positional-Only Arguments**

**用只能以關鍵字指定和只能按位置傳入的參數來設計清晰的參數列表**

按關鍵字傳遞參數是 Python 函數的一項強大特性。這種關鍵字參數特別靈活，在很多情況下，都能讓我們寫出一看就懂的函數程式碼。
例如，計算兩數相除的結果時，可能需要仔細考慮各種特殊情況。
例如，在除數為0的情況下，是拋出 ZeroDivisionError 異常，還是返回無窮（infinity）；在結果溢出的情況下，是拋出 OverflowError 異常，還是返回0。

```python
def safe_division(number, divisor,
                  ignore_overflow,
                  ignore_zero_division):
    try:
        return number / divisor
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
else: raise
```

這個函數用起來很直觀。如果想在結果溢出的情況下，讓它返回 0，那麼可以像下面這樣調用函數。

```python
result = safe_division(1.0, 10**500, True, False)
print(result)
>>> 0
```

如果想在除數是 0 的情況下，讓函數返回無窮，那麼就按下面這樣來寫。

```python
result = safe_division(1.0, 0, False, True)
print(result)
>>> inf
```

表示要不要忽略異常的那兩個參數都是 Boolean 值，所以容易弄錯位置，這會讓程序出現難以查找的 bug。要想讓程式碼看起來更清晰，一種辦法是給這兩個參數都指定默認值。按照默認值，該函數只要遇到特殊情況，就會拋出異常。

```python
def safe_division_b(number, divisor,
                    ignore_overflow=False,        # Changed
                    ignore_zero_division=False):  # Changed
...
```

調用者可以用關鍵字參數指定覆蓋其中某個參數的默認值，以調整函數在遇到那種特殊情況時的處理方式，同時讓另一個參數依然取那個參數自己的默認值。

```python
result = safe_division_b(1.0, 10**500, ignore_overflow=True)
print(result)
result = safe_division_b(1.0, 0, ignore_zero_division=True)
print(result)
>>> 0 inf
```

然而，由於這些關鍵參數是可選的，我們沒辦法要求調用者必須按照關鍵字形式來指定這兩個參數。他們還是可以用傳統的寫法，按位置給這個新定義的 safe_division_b 函數傳遞參數。

```python
assert safe_division_b(1.0, 10**500, True, False) == 0
```

### 使用 keyword-only argument

對於這種參數比較複雜的函數，我們可以聲明只能通過關鍵字指定的參數（keyword-only argument），這樣的話，寫出來的程式就能清楚地反映調用者的想法了。這種參數只能用關鍵字來指定，不能按位置傳遞。

下面就重新定義 safe_division 函數，讓它接受這樣的參數。參數列表裡的*符號把參數分成兩組，左邊是位置參數，右邊是只能用關鍵字指定的參數。

```python
def safe_division_c(number, divisor, *,  # Changed
                    ignore_overflow=False,
                    ignore_zero_division=False):
...
```

如果按位置給只能用關鍵字指定的參數傳值，那麼程式就會出錯。

```python
safe_division_c(1.0, 10**500, True, False)
>>>
Traceback ...
TypeError: safe_division_c() takes 2 positional arguments but 4  ̄were given
```

當然我們還是可以像前面那樣，用關鍵字參數指定覆蓋其中一個參數的默認值（即忽略其中一種特殊情況，並讓函數在遇到另一種特殊情況時拋出異常）。

```python
result = safe_division_c(1.0, 0, ignore_zero_division=True)
assert result == float('inf')
try:
    result = safe_division_c(1.0, 0)
except ZeroDivisionError:
    pass  # Expected
```

這樣改依然有問題，因為在 safe_division_c 版本的函數里面，有兩個參數（也就是 number 和 divisor ）必須由調用者提供。
然而，調用者在提供這兩個參數時，既可以按位置提供，也可以按關鍵字提供，還可以把這兩種方式混起來用。

```python
assert safe_division_c(number=2, divisor=5) == 0.4
assert safe_division_c(divisor=5, number=2) == 0.4
assert safe_division_c(2, divisor=5) == 0.4
```

在未來也許因為擴展函數的需要，甚至是因為程式碼風格的變化，或許要修改這兩個參數的名字

```python
def safe_division_c(numerator, denominator, *,  # Changed
                    ignore_overflow=False,
                    ignore_zero_division=False):
...
```

這看起來只是文字上面的微調，但之前所有通過關鍵字形式來指定這兩個參數的調用代碼，都會出錯。

```python
safe_division_c(number=2, divisor=5)
>>>
Traceback ...
TypeError: safe_division_c() got an unexpected keyword argument  ̄'number'
```

其實最重要的問題在於，我們根本就沒打算把 number 和 divisor 這兩個名稱納入函數的接口；我們只是在編寫函數的實現程式碼時，隨意挑了這兩個比較順口的名稱而已。也並不期望調用者也非得採用這種稱呼來指定這兩個參數。

### 使用 positional-only argument

Python 3.8 引入了一項新特性，可以解決這個問題，這就是只能按位置傳遞的參數（positional-only argument）。

這種參數與剛才的只能通過關鍵字指定的參數（keyword-only argument）相反，它們必須按位置指定，絕不能通過關鍵字形式指定。

下面來重新定義 safe_division 函數，使其前兩個必須由調用者提供的參數只能按位置來提供。參數列表中的 / 符號，表示它左邊的那些參數只能按位置指定。

```python
def safe_division_d(numerator, denominator, /, *,  # Changed
                    ignore_overflow=False,
                    ignore_zero_division=False):
...
```

下面來驗證一下，看看調用者按照位置提供了兩個參數後，能否得到正確結果。

```python
assert safe_division_d(2, 5) == 0.4
```

假如調用者是通過關鍵字形式指定這兩個參數的，那麼程式就會在運行時拋出異常。

```python
safe_division_d(numerator=2, denominator=5)
>>>
Traceback ...
TypeError: safe_division_d() got some positional-only arguments  ̄passed as keyword arguments: 'numerator, denominator'
```

現在我們可以確信：給 safe_division_d 函數的前兩個參數（也就是那兩個必備的參數）所挑選的名稱，已經與調用者的代碼解耦了。
即便以後再次修改這兩個參數的名稱，也不會影響已經寫好的調用程式碼。

在函數的參數列表之中，

/ 符號左側的參數是只能按位置指定的參數， 

* 符號右側的參數則是只能按關鍵字形式指定的參數。
那麼，這兩個符號如果同時出現在參數列表中，會有什麼效果呢？
這是個值得注意的問題。這意味著，這兩個符號之間的參數，既可以按位置提供，又可以用關鍵字形式指定（其實，如果不特別說明Python函數的參數全都屬於這種參數）。

在設計API時，為了體現某些風格或者實現某些需求，可能會允許某些參數既可以按位置傳遞，也可以用關鍵字形式指定，這樣可以讓程式簡單易讀。例如，給下面這個 safe_division 函數的參數列表添加一個可選的 ndigits 參數，允許調用者指定這次除法應該精確到小數點後第幾位。

```python
def safe_division_e(numerator, denominator, /,
                    ndigits=10, *,                 # Changed
                    ignore_overflow=False,
                    ignore_zero_division=False):
    try:
        fraction = numerator / denominator         # Changed
        return round(fraction, ndigits)            # Changed
    except OverflowError:
        if ignore_overflow:
            return 0
        else:
            raise
    except ZeroDivisionError:
        if ignore_zero_division:
            return float('inf')
        else:
            raise

```

下面我們用三種方式來調用這個 safe_division_e 函數。 ndigits 是個帶默認值的普通參數，因此，它既可以按位置傳遞，也可以用關鍵字來指定，還可以直接省略。

### 需要記住的事項：

1. Keyword-only argument 是一種只能通過關鍵字指定而不能通過位置指定的參數。這迫使調用者必須指明，這個值是傳給哪一個參數的。在函數的參數列表中，這種參數位於 * 符號的右側。
2. Positional-only argument 是這樣一種不允許調用者通過關鍵字來指定，而是要求必須按位置傳遞的參數。這可以降低調用程式碼與參數名稱之間的耦合程度。在函數的參數列表中，這些參數位於 / 符號的左側。
3. 在參數列表中，位於 / 與 * 之間的參數，可以按位置指定，也可以用關鍵字來指定。這也是 Python 普通參數的默認指定方式。