# CSS Grid

Created: Jun 5, 2021 5:54 PM  
Property: ivan kao  
Tags: CSS  

## 簡介

Grid 應用於二維布局，同時處理行、列

## 定義

定義一個 grid 容器，為內容創建一個 grid 環境

```css
.container {
  display: grid | inline-grid;
}
```

## 基本知識

分為幾大項目:


基本知識
名稱           |   中文名稱|說明  
---------------|---------:|-----------------
Grid Container | 網格容器  | 所有網格元素的直接父級
Grid Item      | 網格子項  | 網格容器直接子元素
Grid Line      | 網格線    | 網格結構線，可垂直水平
Grid Track     | 網格軌跡  | 兩相鄰網格線之間的空間
Grid Cell      | 網格單元  | 兩相鄰行與兩相鄰列之間的空間
Grid Area      | 網格區域  | 四根網格線所包含的區域
Gutters        | 網格間距  | 兩個網格軌道之間的間隙，這裡不可以放置任何內容
Grid axis      | 網格軸    | 有兩條軸分別用於塊方向的對齊（列軸）和文本方向的對齊（行軸）


## 常用函數

1.  repeat()

    repeat表示網格軌道的重複部分，當你在定義軌道時，有多個重複的配置屬性，這可以使用這個函數；

    ```css
    repeat([number | auto-fill | auto-fit], <軌道配置(網格線 長度)>)
    ```

    number 表示重複具體的次數；

    auto-fill 表示根據第二個參數自動生成不溢出容器的軌道數；

    auto-fit 與 auto-fill 與行為一致，唯一不同的是當軌道內容為空時，將不會生成軌道。

2. minmax()

    定義大小範圍的屬性，大於等於min值，並且小於等於max值

3. fit-content()

    定義大小表示最大不超過傳入參數的長度大小，當小於這個值時則根據內容自動伸縮。

4. max-content

    表示以網格項的最大的內容來佔據網格軌道的關鍵字，定義網格軌道時可用。

    ```css
    ...
    <div class="item-3">T hr ee</div>
    ...

    grid-template-columns: repeat(3, max-content);
    ```

5. min-content

    表示以網格項的最小的內容來佔據網格軌道的關鍵字，定義網格軌道時可用。

## Grid 布局宣告

將某個元素的display設置為grid，就可以創建grid上下文，使用grid佈局的特性，此時這個元素的所有直系子元素都將成為網格元素。

1. 創建 grid 網格上下文

    ```css
    display: grid | inline-grid | subgrid
    ```

## 網格容器屬性

1. 網格的行數列數，內容所占區域

    ```css
    grid-template-columns
    grid-template-rows
    grid-template-areas
    grid-template
    ```

    1. grid-template-columns

        定義網格的列軌道的尺寸和命名列網格線。每條軌道或網格線的名字之間用空格分開

        ```css

        grid-template-columns: <line-name> <length> ... | <length> ... | none | inherit | initial | unset

        .grid-1 {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
        }

        ```

        line-name 用於設置和定位網格區域

        length可用現有的任何長度單位(px、%、rem等)，grid佈局還新增了一個新的單位fr：代表網格容器中可用空間的一等份。

    2. grid-template-rows

        用於定義網格的行軌道的尺寸和命名行網格線。屬性值同 grid-template-columns

        ```css

        grid-template-rows: <line-name> <length> ... | <length> ... | none | inherit | initial | unset

        .grid-1{
            display: grid;
            grid-template-rows: 100px 100px;
        }
        ```

    3. grid-template-areas

        用於定義網格模板，將已命名的網格子元素放在對應的位置。

        ```css
        grid-template-areas: <string> | none

        .grid-1 {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-areas: "item1 item1 item1"
                                 "item2 item2 ."
        }
        .item-1{
            grid-area: item1;
        }
        .item-2{
            grid-area: item2;
        }
        ```

        string的內容是子元素屬性grid-area的值或' . '

        提供了一種可視化的語法，即設置了grid-area的子元素會自動放置在對應的位置上。' . '代表一個空的單元格，兩個空單元格之間要用空格分割，多個相連的點只代表一個單元格。

        需要注意的是，當你命名了某個網格子元素後，包圍著這個網格子元素的網格線會被自動命名為”名稱-start“和”名稱-end“。

    4. grid-template

        grid-template是grid-template-rows、grid-template-columns和grid-template-areas的縮寫。

        ```css
        grid-template: grid-template-rows / grid-template-columns | [<line-names>? <string> <track-size>? <line-names>?] | none

        .grid-1 {
            display: grid;
            grid-template: 100px / repeat(3, 1fr);
        }

        等同

        .grid-1 {
            display: grid;
            grid-template-rows: 100px;
            grid-template-columns: repeat(3, 1fr);
        }
        ```

        grid-template 可用於定義 grid-template-columns, grid-template-rows 中間用 '/' 分割

2. 網格線

    ```css
    grid-column-gap
    grid-row-gap
    grid-gap
    ```

    1.  grid-column-gap

        定義列方向子元素之間的間距

        ```css
        grid-column-gap: <length>

        .grid-1 {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-column-gap: 10px;
        }
        ```

    2. grid-row-gap

        定義行方向子元素之間的間距；

        ```css
        grid-row-gap: <length>

        .grid-1 {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-row-gap: 10px;
        }
        ```

    3. grid-gap

        grid-gap 為 grid-column-gap, grid-row-gap 的縮寫，如果只有一個 grid-gap  值，代表 grid-column-gap, grid-row-gap  設定了相同的屬性

        ```css
        grid-row-gap: <grid-row-gap> <grid-column-gap>?

        .grid-1 {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-gap: 10px 20px;
        }
        ```

3. 網格元素的對齊方式

    ```css
    justify-items
    align-items
    place-items
    ```

    1.  justify-items

        定義了所有網格子元素沿行軸(通常是水平方向)的對齊方式。默認佔滿整個網格區域（stretch）

        ```css
        justify-items: start | end | center | stretch;

        .grid-1 {
            justify-items: start;
        }

        .grid-1 {
            justify-items: end;
        }

        .grid-1 {
            justify-items: center;
        }

        .grid-1 {
            justify-items: stretch;
        }
        ```

        - start 表示沿行軸起始方向對齊
        - end 表示沿行軸結束方向對齊
        - center 表示沿行軸居中對齊
        - stretch 表示沿行軸拉伸至佔滿整個網格區域
    2. align-items

        定義了所有網格子元素沿列軸(通常是垂直方向)的對齊方式。默認佔滿整個網格區域（stretch）。

        ```css
        align-items: start | end | center | stretch;

        .grid-1 {
            align-items: start;
        }

        .grid-1 {
            align-items: end;
        }

        .grid-1 {
            align-items: center;
        }

        .grid-1 {
            align-items: stretch;
        }
        ```

        - start 表示沿列軸起始方向對齊
        - end 表示沿列軸結束方向對齊
        - center 表示沿列軸居中對齊
        - stretch 表示沿列軸拉伸至佔滿整個網格區域
    3. place-items
4. 網格容器的對齊方式

    ```css
    justify-content
    align-content
    place-content
    ```

    1. justify-content

        定義當網格子元素組成的整體行軸方向的大小小於網格容器時，所有網格子元素組成的整體在網格容器內在行軸方向的對齊方式

        ```css
        justify-content: start | end | center | stretch | space-around | space-between | space-evenly;

        .grid-1 {
            grid-template-columns: repeat(3, 100px); 
            // 在此不能用fr作為單位，因為fr會自動拉伸至整個網格容器。即網格子元素組成的整體的大小等於網格容器的大小。
            justify-content: start;
        }

        .grid-1 {
            justify-content: start;
        }

        .grid-1 {
            justify-content: end;
        }

        .grid-1 {
            justify-content: center;
        }

        .grid-1 {
            justify-content: stretch;
        }

        .grid-1 {
            grid-template-columns: repeat(3, 100px);
            justify-content: space-around;
        }

        .grid-1 {
            grid-template-columns: repeat(3, 100px);
            justify-content: space-between;
        }

        .grid-1 {
            grid-template-columns: repeat(3, 100px);
            justify-content: space-evenly;
        }
        ```

        - start 於網格容器沿行軸起始方向對齊
        - end 於網格容器沿行軸結束方向對齊
        - center 於網格容器沿行軸居中對齊
        - stretch 拉伸網格子元素使之佔滿整個網格容器
        - space-around 每列設置等距空白間隙，邊緣間隙是中間間隙的一半
        - space-between 每列設置等距空白間隙，邊緣無間隙
        - space-evenly 每列設置等距空白間隙，邊緣間隙與中間間隙相同
    2. align-content

        用於定義當網格子元素組成的整體列軸方向的大小小於網格容器時，所有網格子元素組成的整體在網格容器內在列軸方向的對齊方式。

        ```css
        align-content: start | end | center | stretch | space-around | space-between | space-evenly;

        .grid-1 {
            height: 200px;
            grid-template-columns: repeat(3, 1fr);
        }

        .grid-1 {
            align-content: start;
        }

        .grid-1 {
            align-content: end;
        }

        .grid-1 {
            align-content: center;
        }

        .grid-1 {
            align-content: stretch;
        }

        .grid-1 {
            align-content: space-around;
        }

        .grid-1 {
            align-content: space-between;
        }

        .grid-1 {
            align-content: space-evenly;
        }
        ```

        - start 於網格容器沿列軸起始方向對齊
        - end 於網格容器沿列軸結束方向對齊
        - center 於網格容器沿列軸居中對齊
        - stretch 拉伸網格子元素使之佔滿整個網格容器
        - space-around 每行設置等距空白間隙，邊緣間隙是中間間隙的一半
        - space-between 每行設置等距空白間隙，邊緣無間隙
        - space-evenly 每行設置等距空白間隙，邊緣間隙與中間間隙相同
    3. place-content
5. 網格軌跡

    ```css
    grid-auto-columns
    grid-auto-rows
    grid-auto-flow
    ```

    1. grid-auto-columns

        當你對網格子元素進行定位且定位的元素在列方向上位於顯式創建的網格外時，grid-auto-columns 會隱式創建一個列軌道，其他網格子元素會根據自動定位算法放置在某個位置。

        ```css
        grid-auto-columns: <length>

        .grid-1 {
            grid-template-columns: repeat(3, 1fr);
            grid-auto-columns: 100px;
        }
        .item-1 {
            grid-column: 4 / 5;
            grid-row: 2 / 3;
        }
        ```

        此屬性值可使用任何長度單位，也可以使用fr

    2. grid-auto-rows

        類似於 grid-auto-columns，只是隱式創建是在行方向上完成的。

        ```css
        grid-auto-rows: <length>

        .grid-1 {
            grid-template-rows: 50px;
            grid-auto-rows: 100px;
        }
        .item-1 {
            grid-column: 4 / 5;
            grid-row: 2 / 3;
        }
        ```

    3. grid-auto-flow

        控制著自動佈局算法的運作方式，精確指定在網格中被自動佈局的元素（即未進行定位的網格子元素）怎樣排列。

        ```css
        grid-auto-flow: row | column | dense | row dense | column dense

        .grid-1 {
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 1fr);
            grid-auto-flow: row;
        }
        .item-1{
            grid-column: 1 / 3;
        }
        .item-2{
            grid-column: 2 / 4;
        }
        ```

        - row（默認值）指定自動佈局算法按照通過逐行填充來排列元素，在必要時增加新行。
        - column 指定自動佈局算法通過逐列填充來排列元素，在必要時增加新列。
        - dense指定自動佈局算法使用一種“稠密”堆積算法，即如果後面出現了稍小的元素，則會試圖去填充網格中前面留下的空白。**這樣做會填上稍大元素留下的空白，但同時也可能導致原來出現的次序被打亂。**
        - row dense 自動進行“稠密”排序，優先填充行
        - column dense 自動進行“稠密”排序，優先填充列
    4. 簡寫

        ```css
        grid
        ```

        grid 是屬性 grid-template-rows、grid-template-columns、grid-template-areas、grid-auto-rows、grid-auto-columns、grid-auto-flow、grid-column-gap 和 grid-row-gap 的簡寫

        參考資料:[https://developer.mozilla.org/en-US/docs/Web/CSS/grid](https://developer.mozilla.org/en-US/docs/Web/CSS/grid)

## 網格子項屬性

1. 網格的行數列數，內容所占區域

    ```css
    grid-column-start
    grid-column-end
    grid-row-start
    grid-row-end
    grid-column
    grid-row
    grid-area
    ```

    1.  grid-column-start/grid-column-end/grid-row-start/grid-row-end

        透過命名後的網格線給網格子元素定位。當沒有給網格線顯式命名時，可以根據網格線的編號（從1開始沿行軸和列軸方向編號逐一累加）進行定位。

        ```css
        grid-column-start: auto | <line-index> | <grid-line-name> | span <number> | span <grid-line-name>

        .grid-1 {
            grid-template-columns: [col-1] 1fr [col-2] 1fr [col-3] 1fr [col-4];
            grid-template-rows: [row-1] 1fr [row-2] 1fr [row-3] 1fr [row-4];
        }
        .item-1{
            grid-column-start: 1;
            grid-column-end: 3;
            grid-row-start: 1;
            grid-row-end: 3;
        }
        .item-2{
            grid-column-start: col-1;
            grid-column-end: col-4;
            grid-row-start: row-3;
            grid-row-end: row-4;
        }
        ```

        - auto 表示自動定位，且子元素跨度為1；
        - line-index 表示將指定編號的網格線作為該網格子元素的列起始線/列結束線/行起始線/行結束線
        - grid-line-name 表示將已命名的網格線作為該網格子元素的列起始線/列結束線/行起始線/行結束線
        - span + < number > 表示從網格線的初始網格列線/行線開始跨越< number >格
        - span + < grid-line-name > 表示從網格線的初始網格列線/行線開始直至碰到名為< grid-line-name >的網格線（實測無效，應該是規範未被實現）
    2. grid-column/grid-row

        grid-column/grid-row分別是grid-column-start+ grid-column-end，和grid-row-start+grid-row-end的簡寫。語法同上，start和end之間用' / '分隔。當只有一個值時表示只設置了grid-column-start/ grid-row-start。

        ```css
        grid-column: <grid-column-start> / <grid-column-end>?grid-row: <grid-row-start> / <grid-row-end>?

        .item-1{
            grid-column: 1 / 3;
            grid-row: 1 / 3;
        }
        .item-2{
            grid-column: col-1 / col-4;
            grid-row: row-3 / row-4;
        }

        等同

        .item-1{
            grid-column-start: 1;
            grid-column-end: 3;
            grid-row-start: 1;
            grid-row-end: 3;
        }
        .item-2{
            grid-column-start: col-1;
            grid-column-end: col-4;
            grid-row-start: row-3;
            grid-row-end: row-4;
        }
        ```

    3.  grid-area

        grid-area是grid-row-start+ grid-column-start+ grid-row-end+grid-column-end的簡寫。每個參數的語法與grid-column-start相同，網格線之間用' / '分隔。也可以是一個區域名字，與grid-template-areas屬性值對應。

        ```css
        grid-area: <area-name> | <grid-row-start-line> / <grid-column-start-line>? / <grid-row-end-line>? / <grid-column-end-line>?

        .grid-1 {
            grid-template-areas: "header header header"
                                 ". . ."
        }
        .item-1 {
            grid-area: header;
        }

        .item-1 {
            grid-area: 1 / 3 / 3 / 4;
        }

        .item-1 {
            grid-area: 1 / 3 / 3;
        }

        .item-1 {
            grid-area: 1 / 3;
        }

        .item-1 {
            grid-area: 3;
        }
        ```

        area-name當你為某個網格子元素設置grid-area的名字後，就可以使網格子元素定位到grid-template-areas設置的網格模板的相應的位置了。此時，會自動為包圍著網格子元素的4條網格線命名為：行起始線和列起始線< area-name > + '-start'、行結束線和列結束線< area- name > + '-end'

        grid-line如果有4個值，則與grid-row-start、grid-column-start、grid-row-end、grid-column-end依次對應；如果有3個值，則與grid-row-start、grid-column-start、grid-row-end依次對應；如果有2個值，則與grid-row-start、grid-column-start依次對應；如果有1個值，則與grid-row-start對應

2. 對齊方式

    ```css
    justify-self
    align-self
    place-self
    ```

    1. justify-self

        定義了某個子元素沿行軸(通常是水平方向)的對齊方式。

        ```css
        justify-self: start | end | center | stretch;

        .grid-1 {
            grid-template-columns: repeat(3, 1fr)
        }
        .item-1{
            justify-self: start;
        }

        .item-1{
            justify-self: end;
        }

        .item-1{
            justify-self: center;
        }

        .item-1{
            justify-self: stretch;
        }
        ```

        - start 表示沿行軸起始方向對齊
        - end 表示沿行軸結束方向對齊
        - center 表示沿行軸居中對齊
        - stretch 表示沿行軸拉伸至佔滿整個網格區域
    2. align-self

        定義了某個子元素沿列軸(通常是垂直方向)的對齊方式。

        ```css
        align-self: start | end | center | stretch;

        .grid-1 {
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(3, 100px);
        }
        .item-1{
            align-self: start;
        }

        .item-1{
            align-self: end;
        }

        .item-1{
            align-self: center;
        }

        .item-1{
            align-self: stretch;
        }
        ```

        - start 表示沿列軸起始方向對齊
        - end 表示沿列軸結束方向對齊
        - center 表示沿列軸居中對齊
        - stretch 表示沿列軸拉伸至佔滿整個網格區域
    3. place-self

## 範例

1. 在一個4*4的表格內，放一個小格子在第2行第3列的位置；

    ```css
    <div class="grid-1">
        <div class="item-1">One</div>
    </div>

    .grid-1 {
        width: 750px;
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(4, 1fr);
    }
    .item-1{
        grid-area: 2 / 3;
    }
    ```

2. 實現聖杯佈局

    ```css
    <div class="grid">
        <div class="header">1</div>
        <div class="nav">2</div>
        <div class="main">3</div>
        <div class="side">4</div>
        <div class="footer">5</div>
    </div>
    .grid {
        width: 750px;
        display: grid;
        grid: auto / 150px 1fr 180px;
    }
    .header,.footer {
        grid-column: span 3;
    }

    等同

    .grid {
        display: grid;
        grid-template-columns :150px 1fr 180px;
        grid-template-areas: "header header header"
                            "nav main side"
                            "footer footer footer"
    }
    .header {
        grid-area: header
    }
    .nav {
        grid-area: nav
    }
    .main {
        grid-area: main
    }
    .side {
        grid-area: side
    }
    .footer {
        grid-area: footer
    }
    ```

3.  利用grid實現瀑布流佈局

    ```css
    <div class="grid">
            <div class="item-1"><div>One</div></div>
            <div class="item-2"><div>Two</div></div>
            <div class="item-3"><div>Three</div></div>
            <div class="item-4"><div>Four</div></div>
            <div class="item-5"><div>Five</div></div>
            <div>
                <p>1</p>
                <p>2</p>
                <p>3</p>
                <p>4</p>
            </div>
        </div>

        <style>
            .grid {
                display: grid;
                // 自定义列数和列数的宽度
                grid-template-columns: repeat(3, 1fr);
                // 自定义行轨道宽度
                grid-auto-rows: 1px;
                grid-auto-flow: row dense;
                align-items: start;
            }
            .item-1 div{
                height: 150px
            }
            .item-2 div{
                height: 400px
            }
            .item-3 div{
                height: 200px
            }
            .item-4 div{
                height: 250px
            }
            .item-5 div{
                height: 300px
            }
        </style>

        <script>
            var gridItems = document.getElementsByClassName('grid')[0].children;
            for(var i = 0, l = gridItems.length; i< l; i++) {
                gridItems[i].style.gridRow = "span " + gridItems[i].offsetHeight;
            }
        </script>
    ```

## References

[https://developer.mozilla.org/en-US/docs/Web/CSS/grid](https://developer.mozilla.org/en-US/docs/Web/CSS/grid)

[https://zhuanlan.zhihu.com/p/46754464](https://zhuanlan.zhihu.com/p/46754464)

[https://scrimba.com/learn/cssgrid](https://scrimba.com/learn/cssgrid)

[https://css-tricks.com/snippets/css/complete-guide-grid/](https://css-tricks.com/snippets/css/complete-guide-grid/)

[https://css-tricks.com/getting-started-css-grid/](https://css-tricks.com/getting-started-css-grid/)

[https://css-tricks.com/snippets/css/css-grid-starter-layouts/](https://css-tricks.com/snippets/css/css-grid-starter-layouts/)

[https://zhuanlan.zhihu.com/p/51082686](https://zhuanlan.zhihu.com/p/51082686)