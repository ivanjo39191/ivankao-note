# AsyncIO, Threading, and Multiprocessing in Python (二)

Created: March 15, 2022 10:33 PM
Property: Ivan Kao
Status: 已公開
Tags: Python

## **一、程序(Process) vs 執行緒(Thread)**

一個程序(Process)產生的執行實體需要一定的CPU與記憶體等資源，不同程序相互獨立不會共用記憶體資源。

一個程序(Process)則是由多個執行緒(Thread)組成，執行緒(Thread)就是執行工作的基本單位。同一個程序的執行緒可以共用記憶體資源。

## **二、多執行緒(Multithreading)**

### 簡介、

Multi-threading 又稱多執行緒或多線程。

多執行緒(Multithreading)，就是在同一個程序(Process)中，建立多個執行緒同時執行任務，藉此來提升效率。

主執行緒(Main Thread)閒置或等待時，CPU就會切換到另一個執行緒(Thread)執行其它的工作，完成後再回到主執行緒(Main Thread)繼續往下執行。

因為多執行緒(Multithreading)能夠在閒置或等待的時候，透過不斷互相切換的動作，來節省執行的時間。

### T**hreading** 使用方式

使用 `threading` 模組，不用特別安裝即可使用，是 Python 標準函式庫裡面的模組。

### ****建立子執行緒****

1.  建立子執行緒執行的 `job`函數
2. 使用 `threading.Thread` 建立一個新的子執行緒，`target` 指定為 job
3. 呼叫執行緒的 `start` 函數，開始執行
4. 主執行緒繼續處理自己的任務
5. 放在 `join` 之後的程式碼就會等到子執行緒執行完成後，才會接著執行

```python
import threading
import time

# 子執行緒的工作函數
def job():
  for i in range(5):
    print("Child thread:", i)
    time.sleep(1)

# 建立一個子執行緒
t = threading.Thread(target = job)

# 執行該子執行緒
t.start()

# 主執行緒繼續執行自己的工作
for i in range(3):
  print("Main thread:", i)
  time.sleep(1)

# 等待 t 這個子執行緒結束
t.join()

print("Done.")

##################################
Child thread: 0
Main thread: 0
Main thread: 1
Child thread: 1
Main thread: 2
Child thread: 2
Child thread: 3
Child thread: 4
Done.
```

### ****多個子執行緒與參數****

1. 設定子執行緒執行的 `job` 函數會接受一個 `num` 參數
2. 呼叫 `threading.Thread` 建立子執行緒時，將要傳入的參數放在 `args` 參數中
3. 使用  `join` 等待子執行緒結束

```python
import threading
import time

# 子執行緒的工作函數
def job(num):
  print("Thread", num)
  time.sleep(1)

# 建立 5 個子執行緒
threads = []
for i in range(5):
  threads.append(threading.Thread(target = job, args = (i,)))
  threads[i].start()

# 主執行緒繼續執行自己的工作
# ...

# 等待所有子執行緒結束
for i in range(5):
  threads[i].join()

print("Done.")

##################################
Thread 0
Thread 1
Thread 2
Thread 3
Thread 4
Done.

```

### 平行任務 concurrent.futures

`concurrent.futures.**Executor**`

抽像類提供異步執行調用方法。要通過它的子類調用，而不是直接調用。

`concurrent.futures.**ThreadPoolExecutor**`(*max_workers=None*, *thread_name_prefix=''*, *initializer=None*, *initargs=()*)

ThreadPoolExecutor是 Executor 的子類，它使用線程池來異步執行調用。

`concurrent.futures.**Executor.submit**`( *fn* , */* , **args* , ***kwargs* )

調度可調用對象 *fn* 以執行 並返回一個表示可調用對象執行的對象。

`concurrent.futures.**as_completed**`(*fs*, *timeout=None*)

返回一個包含 fs 所指定的 Future 實例（可能由不同的 Executor 實例創建）的迭代器，這些實例會在完成時生成 future 對象

```python
import concurrent.futures as cf
import time

# 子執行緒的工作函數
def job(num):
  print(f"Thread {num} started")
  time.sleep(1)
  return f"Thread {num}"

with cf.ThreadPoolExecutor(max_workers=1) as executor:
  future_to_mapping  = {executor.submit(job, i): i for i in range(1, 6)}
  for future in cf.as_completed(future_to_mapping):
      print(f"{future.result()} Done")

print("Done.")
```

## 三、多進程 **Multi-processing**

Multi-threading 又稱多執行緒或多線程。

使用 multiprocessing 模組，不用特別安裝即可使用，是 Python 標準函式庫裡面的模組。

### 檢查 process 數量

先使用 multiprocessing.cpu_count() 或 os.cpu_count() 來獲取當前機器的 CPU 核心數量。

設置 process 時最好等於當前機器的 CPU 核心數量，避免CPU 之間程序處理會切換造成本進而降低處理效率。

```python
# 方法一
import multiprocessing
cpus = multiprocessing.cpu_count()

# 方法二
Import os
cpus = os.cpu_count()
```

### multiprocessing 使用方式

與 threading 的使用方式大同小異

1. 設定子執行緒執行的 `job` 函數會接受一個 `num` 參數
2. 呼叫 `multiprocessing.Process` 建立子執行緒時，將要傳入的參數放在 `args` 參數中
3. 使用  `join` 等待子執行緒結束

```python
import multiprocessing
import time

# 進程的工作函數
def job():
  for i in range(5):
    print("Child Process:", i)
    time.sleep(1)

# 建立一個進程任務
t = multiprocessing.Process(target = job)

# 執行該子執行緒
t.start()

# 主進程繼續執行自己的工作
for i in range(3):
  print("Main Process:", i)
  time.sleep(1)

# 等待 t 這個進程任務結束
t.join()

print("Done.")

########################
Main Process: 0
Child Process: 0
Main Process: 1
Child Process: 1
Child Process: 2
Main Process: 2
Child Process: 3
Child Process: 4
Done.
```

## 四、AsyncIO

AsyncIO 是單線程單進程協作多任務。asyncio 任務獨占 CPU，直到它希望將其交給協調器或事件循環。

### ****AsyncIO 中的並發****

**協程：**與具有單點退出的傳統函數不同，協程可以暫停和恢復其執行。 創建協程就像在聲明函數之前使用 async 關鍵字一樣簡單。

**事件循環或協調器：** 管理其他協程的協程。 您可以將其視為調度程序或主機。

**可等待對象：**協程、任務 和 Future 是可等待的對象。 協程可以等待這些對象。 當協程在等待一個對象時，它的執行會暫時掛起，並在 Future 完成後恢復。

### Awaitables 可等待對像

在使用 asyncio 相關函式時，經常可以在文件中看到 `awaitables` 關鍵字，這個關鍵字就代表著以下 3 種 Python 物件(objects)，也是 `await` 語法適用的對象：

1. Coroutines 
    
    Python 協程是可*等待*的，因此可以從其他協程中等待
    
2. Tasks 
    
    *任務*用於同時調度協*程*。
    
    當一個協程被包裝到一個*任務*中時， `[asyncio.create_task()](https://docs.python.org/3/library/asyncio-task.html#asyncio.create_task)`會自動安排協程。
    
3. Futures
    
    Future是一個 low-level 可等待對象，表示異步操作的最終結果。
    

### 使用 asyncio

`asyncio.run(coro, *, debug=False)`

該函數運行傳遞的協程，負責管理 asyncio 事件循環、完成異步生成器並關閉線程池。

這個函數總是創建一個新的事件循環並在最後關閉它。 它應該用作 asyncio 程序的主要入口點，並且最好只調用一次。

`asyncio.create_task(coro, *, name=None)`

將協程包裝成一個 Task 並安排其執行。返回任務對象。

`asyncio.sleep(delay, result=None)`

sleep() 總是暫停當前任務，允許其他任務運行。

```python
import asyncio

async def job(num):
  print(f"Job {num} started")
  await asyncio.sleep(num)
  print(f"Job {num} Ended")

async def main():
  print('Main Started')
  task_1 = asyncio.create_task(job(1)) 
  task_2 = asyncio.create_task(job(2)) 
  task_3 = asyncio.create_task(job(3)) 
  task_4 = asyncio.create_task(job(4)) 
  task_5 = asyncio.create_task(job(5)) 
  await task_1
  await task_2
  await task_3
  await task_4
  await task_5
  print('Main Ended')

if __name__ == '__main__':
    
    asyncio.run(main())

#########################
Main Started
Job 1 started
Job 2 started
Job 3 started
Job 4 started
Job 5 started
Job 1 Ended
Job 2 Ended
Job 3 Ended
Job 4 Ended
Job 5 Ended
Main Ended
```

### 使用 asyncio.gather

`asyncio.gather(*aws, return_exceptions=False)`

同時運行aws 序列中的等待對象。

*如果aws*中的任何 awaitable是協程，它會自動安排為任務。

如果所有可等待對像都成功完成，則結果是返回值的聚合列表。結果值的順序對應於*aws*中的可等待對象的順序。

```python
import asyncio

# 子執行緒的工作函數
async def job(num):
  print(f"Job {num} started")
  await asyncio.sleep(num)
  print(f"Job {num} Ended")

async def main():
  print('Main Started')
  await asyncio.gather(*[job(i+1) for i in range(5)])
  print('Main Ended')

if __name__ == '__main__':
    
    asyncio.run(main())

#########################
Main Started
Job 1 started
Job 2 started
Job 3 started
Job 4 started
Job 5 started
Job 1 Ended
Job 2 Ended
Job 3 Ended
Job 4 Ended
Job 5 Ended
Main Ended
```

## 五**. Multi-processing、Multi-threading、AsyncIO 的優缺點：**

- Multi-processing (多處理程序/多進程)：
    1. 資料在彼此間傳遞變得更加複雜及花時間，因為一個 process 在作業系統的管理下是無法去存取別的 process 的 memory
    2. 適合需要 CPU 密集，像是迴圈計算
- Multi-threading (多執行緒/多線程)：
    1. 資料彼此傳遞簡單，因為多執行緒的 memory 之間是共用的，但也因此要避免會有 Race Condition (競爭條件)問題，必須使用 Lock 來防止這種情況發生。
    2. 適合需要 I/O 密集的任務，像是爬蟲需要時間等待 request 回覆。
- AsyncIO (單執行緒/單線程)：
    1. 資料彼此傳遞簡單，由於任務可以完全控制何時暫停執行，很少出現 Race Condition (競爭條件)的問題。
    2. 適合需要 I/O 密集的任務。

## 參考資料

****[Python 多執行緒 threading 模組平行化程式設計教學](https://blog.gtwang.org/programming/python-threading-multithreaded-programming-tutorial/)****

****[【Python教學】淺談 Multi-processing & Multi-threading 使用方法](https://www.maxlist.xyz/2020/03/15/python-threading/)****

****[AsyncIO, Threading, and Multiprocessing in Python](https://medium.com/analytics-vidhya/asyncio-threading-and-multiprocessing-in-python-4f5ff6ca75e8)****

****[Python asyncio 從不會到上路](https://myapollo.com.tw/zh-tw/begin-to-asyncio/)****

**[Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html)**