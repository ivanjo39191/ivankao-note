# AsyncIO, Threading, and Multiprocessing in Python

Created: March 2, 2022 8:26 PM
Property: Ivan Kao
Status: 已公開
Tags: Python

## 簡介、

在 Python 中通常有三個 library 的選項來實現並發—— multiprocessing, threading, 和 Asyncio。

首先要先了解 CPU密集型 與 I/O密集型，Concurrency , Parallelism 的差別，再根據效能與需求來分析當下要使用哪個 library。

三者比較之後，最後則是進行簡易的實作範例。

## CPU 密集型 與 I/O 密集型

### CPU 密集型

CPU 密集型是指它完成任務的時間主要取決於中央處理器的速度。

擁有的 CPU 時脈頻率越快，程序的性能就越高。

大多數單核心處理器電腦的程序都受 CPU 限制。

### I/O 密集型

I/O 密集型是指完成計算所需的時間主要取決於等待輸入/輸出操作完成所花費的時間。

這與受 CPU 限制的任務相反，提高 CPU 時脈頻率不會顯著提高程序的性能。

但如果我們有更快的 I/O，例如更快的內存 I/O、硬碟 I/O、網絡 I/O 等，程序的性能就會得到提升。

大多數的 web service 是使用 I/O 密集型。

## ****Concurrency 與 Parallelism 的差異****

Concurrency 併發：指有同時處理多個任務的能力，也是廣義的並行。

Parallelism 並行：在同一時間執行多個操作，Parallelism 屬於 Concurrency 的一種。

Threading, Async IO 屬於 Concurrency 併發

Multi-processing 屬於 Parallelism 並行

## ****Process VS Thread in Python****

### Process (進程) in Python

global interpreter lock (GIL) 是計算機語言解釋器中的一種機制，用於同步線程的執行，一次只能執行一個本地線程( CPU 內核中的線程數)。

因為 Python 的解釋器中使用 GIL，因此單進程的 Python 程序在執行期間只能使用一個本機線程，也就是說無論 Python 程序是單進程單線程 或是 單進程多線程 CPU 使用率都不能超過 100%。

相反若是 C/C++ 沒有解釋器的語言就不會用到解釋器，單進程多線程時 CPU 使用率就可能會超過 100%。

因此，對於 Python 中的 CPU 密集型 任務，須編寫多進程 Python 程序以最大限度地提高其性能。

### Thread (線程) in Python

因為單進程 Python 只能使用一個 CPU 原生線程。 無論使用多少線程，單進程多線程 Python 程序最多只能達到 100% 的 CPU 使用率，因此，對於 Python 中的 CPU 密集型任務，單進程多線程 Python 程序不會提高性能。 

但對於 Python 中的 I/O 密集型任務，可以使用多線程來提高程序性能。

## **Multiprocessing VS Threading VS AsyncIO in Python**

### Multiprocessing (多進程)

使用 Python Multiprocessing，能夠使用多個進程運行 Python。多進程 Python 程序可以通過在許多本地線程上創建多個 Python 解釋器來充分利用所有可用的 CPU 內核和本地線程。 因為所有的進程都是相互獨立的，並且它們不共享內存。 

要使用 Multiprocessing 在 Python 中執行協作任務，需用 OS 提供的 API，會耗費稍大的成本。

對於 Python 中的 CPU 密集型任務，多處理將是一個完美的 library ，可用於最大限度地提高性能。

### Threading (線程)

使用 Python threading (線程)，我們能夠更好地利用等待 I/O 時處於空閒狀態的 CPU。 通過重疊請求的等待時間，我們能夠提高性能。 

此外，由於所有 Python 線程共享相同的內存，要在 Python 中使用 threading 執行協作任務，我們必須小心並在必要時使用鎖。 鎖定和解鎖確保一次只有一個線程可以寫入內存，但這也會帶來一些成本。 

對於 Python 中的 I/O 密集型任務，線程可能是一個很好的 library 候選者，可用於最大限度地提高性能。

需注意的是，所有線程都在一個 pool 中，並且有一個來自 OS 的執行器來管理線程，決定誰運行以及何時運行。 這可能是線程的一個缺點，因為 OS 實際上知道每個線程並且可以隨時中斷它以開始運行不同的線程。 這稱為搶占式多任務處理，因為 OS 可以搶占線程以進行切換。

### AsyncIO (異步IO)

鑑於線程在 Python 中使用多線程來最大化 I/O 綁定任務的性能，我們想知道是否有必要使用多線程。答案是否定的，如果您知道何時切換任務。例如，對於使用線程的 Python 程序中的每個線程，在發送請求和返回結果之間，它確實會保持空閒狀態。如果某個線程可以知道發送 I/O 請求的時間，它可以切換到執行另一項任務，而不會保持空閒狀態，並且一個線程應該足以管理所有這些任務。如果沒有線程管理開銷，I/O 綁定任務的執行應該更快。顯然，線程無法做到這一點，但我們有 asyncio。

使用 Python asyncio，我們還能夠更好地利用等待 I/O 時處於空閒狀態的 CPU。與線程不同的是，asyncio 是單進程單線程的。 asyncio 中有一個事件循環，用於定期測量任務的進度。如果事件循環測量了任何進度，它會安排另一個任務執行，從而最大限度地減少等待 I/O 所花費的時間。這也稱為協作多任務。這些任務必須通過宣布它們何時準備好被關閉來進行合作。

asyncio 的缺點是，如果我們不告訴它，事件循環將不知道進度是什麼。 當我們使用 asyncio 編寫程序時，這需要一些額外的努力。

## **Multiprocessing VS Threading VS AsyncIO in Python 比較表**

| 協程類型 | 功能 | 使用情境 | Metaphor |
| --- | --- | --- | --- |
| Multiprocessing | 多進程，CPU 利用率高 | CPU 密集 | 十個廚房，十個廚師，十道菜要煮。 |
| Threading | 單進程、多線程
搶先式多任務，OS 決定任務切換。 | 快速 I/O  | 一個廚房，十個廚師，十道菜要煮。 十位大廚齊聚一堂。 |
| AsyncIO | 單進程，單線程
協同多任務，任務協同決定切換。 | 慢速 I/O | 一個廚房，一個廚師，十道菜要煮。 |
- CPU 密集 => Multiprocessing
- I/O 密集, 快速 I/O, 連接數量有限 => Threading
- I/O 密集, 慢速 I/O, 連接數量多     => Asyncio

## 參考資料

****[Multiprocessing VS Threading VS AsyncIO in Python](https://leimao.github.io/blog/Python-Concurrency-High-Level/)****

****[【Python教學】淺談 Concurrency Programming](https://www.maxlist.xyz/2020/04/09/concurrency-programming/)****