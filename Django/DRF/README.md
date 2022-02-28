# DRF

Created: February 1, 2022 5:22 PM
Tags: Django, Python

## RESTful API?

不同的Method就是對同一件事情做不同的操作。

```python
# 如果我們在寫一隻商品的WebAPI，讓工程師隨便寫可能會有以下方式來作interface：
獲得商品資料 GET   /getAllItems
獲得商品資料 GET   /getItem/11
新增商品資料 POST  /createItem
更新商品資料 POST  /updateItem/
刪除商品資料 POST  /deleteItem/

# 若是以使用 RESTful API 開發的話:
獲取商品資料 /GET     /items
獲取商品資料 /GET     /items/1
新增商品資料 /POST    /items
更新商品資料 /PATCH   /items/1 
刪除商品資料 /DELETE  /items/1
```

## 前端需要什麼?

1. 不用登入可以查找到景點相關資訊
    
    ```python
    # 找全部景點全部欄位
    # http://ttime-demo.ivankaoblog.com:8000/api/travel/places/
    [
        {
            "json_d": "C1_315081800H_000071",
            "name": "萊萊秘境",
            "zone": "",
            "toldescribe": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "description": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "add": "新北市貢寮區蓬萊街2號",
            "zipcode": "",
            "region": "新北市",
            "town": "貢寮區",
            "tel": "886-9-19242070",
            "travellinginfo": "1.\t國光客運1740(雙溪火車站-宜蘭轉運站)→萊萊站下車。2.\t國光客運1811(台北火車站-羅東)或1812(台北火車站-南方澳)→萊萊站下車。3.\t新巴士F831(貢寮火車站-鶯歌石)→萊萊站下車。",
            "opentime": "週二及週四 13:00-17:00；週六-週日 11:00-17:00",
            "website": "https://www.facebook.com/pages/category/Cafe/%E8%90%8A%E8%90%8A%E7%A7%98%E5%A2%83-1756086801379317/",
            "picture1": "https://www.necoast-nsa.gov.tw/FileArtPic.ashx?id=680&w=1280&h=960",
            "picdescribe1": "IMG_9210 [原始大小]",
            "picture2": "",
            "picdescribe2": "",
            "picture3": "",
            "picdescribe3": "",
            "gov": "315081800H",
            "px": "121.99421",
            "py": "25.00192",
            "orgpclass": "",
            "pclass1": "08",
            "pclass2": "08",
            "pclass3": "08",
            "map": "",
            "parkinginfo": "",
            "parkinginfo_px": null,
            "parkinginfo_py": null,
            "ticketinfo": "免費參觀，餐飲另計",
            "remarks": "",
            "keyword": "",
            "changetime": "2020-11-12T05:39:58Z"
        },
        {
            "json_d": "C1_315081700H_000161",
            "name": "李子園小舖",
            "zone": "",
            "toldescribe": "東山李子園清光緒年間，先民由大陸搬遷到此定居，將帶來的李子種子種在自家門前。那時家家戶戶皆有李子樹，彷彿一片李子園，因而將此地命名為李子園。隨著時代的變遷與農業結構改變，當地農民改種植香蕉與椪柑等農作物，為何現今李子園會種植咖啡呢？茲因早期日本人發現東山區的氣候與土壤適合栽種「阿拉比卡」咖啡豆，遂而引進咖啡開始種植培育，當時喝過咖啡的人都說「這東西聞起來很香，但是喝起來好苦，不好喝。」昔日那個年代對於咖啡的接受度很低，因此東山咖啡發展也就此中斷。早年被嫌棄的咖啡，現今喝咖啡卻成為一種時尚，一種品味，有些人更視為必需品，每天不喝一杯咖啡就混身不自在；也因為如此，東山在地的農民想起昔日，日本人曾經在此栽種咖啡，十年前開始重新種植咖啡，近年積極參與國際咖啡評鑑，獲得美國C.Q.I咖啡精品協會認證，品質大獲肯定，讓東山咖啡逐漸在國際市場展露頭角，成為台灣咖啡主要重要產地之一。由在地居民齊心打造的「李子園小舖」，是一處充滿藝術風情的溫馨旅遊資訊站。每當有旅人訪時，居民總是熱情邀約「進來做唷，來喝杯咖啡」，為路過旅人煮一杯香濃咖啡，好客心意配上手沖的溫度，只為了讓旅人感受自家烘焙的的好味道。除了品嚐東山咖啡香，李子園小舖也推出咖啡限定小旅行，帶領遊客體驗烘焙咖啡的樂趣，種下屬於自己的咖啡寶寶，還有機會體驗龍眼乾包咖啡豆的在地吃法，感受滿滿的山城人情味。",
            "description": "東山李子園清光緒年間，先民由大陸搬遷到此定居，將帶來的李子種子種在自家門前。那時家家戶戶皆有李子樹，彷彿一片李子園，因而將此地命名為李子園。隨著時代的變遷與農業結構改變，當地農民改種植香蕉與椪柑等農作物，為何現今李子園會種植咖啡呢？茲因早期日本人發現東山區的氣候與土壤適合栽種「阿拉比卡」咖啡豆，遂而引進咖啡開始種植培育，當時喝過咖啡的人都說「這東西聞起來很香，但是喝起來好苦，不好喝。」昔日那個年代對於咖啡的接受度很低，因此東山咖啡發展也就此中斷。早年被嫌棄的咖啡，現今喝咖啡卻成為一種時尚，一種品味，有些人更視為必需品，每天不喝一杯咖啡就混身不自在；也因為如此，東山在地的農民想起昔日，日本人曾經在此栽種咖啡，十年前開始重新種植咖啡，近年積極參與國際咖啡評鑑，獲得美國C.Q.I咖啡精品協會認證，...",
            "add": "臺南市733東山區高原里李子園93號",
            "zipcode": "733",
            "region": "臺南市",
            "town": "東山區",
            "tel": "886-6-6863327",
            "travellinginfo": "鄰近公車站：李子園站（大臺南公車黃11-1、黃13延）",
            "opentime": "08：00~17：00",
            "website": "",
            "picture1": "https://www.siraya-nsa.gov.tw/image/615/640x480",
            "picdescribe1": "李子園小舖的藝術裝潢",
            "picture2": "",
            "picdescribe2": "",
            "picture3": "",
            "picdescribe3": "",
            "gov": "315081700H",
            "px": "120.50625",
            "py": "23.28691",
            "orgpclass": "",
            "pclass1": "01",
            "pclass2": "",
            "pclass3": "",
            "map": "",
            "parkinginfo": "李仔園小舖有設置停車場",
            "parkinginfo_px": null,
            "parkinginfo_py": null,
            "ticketinfo": "無門票費用",
            "remarks": "※咖啡導覽體驗活動，建議出發前洽詢李子園小舖。※夏季山區易有午後雷陣雨，建議攜帶雨具※山區蚊蟲較多，建議攜帶防蚊液或穿著長袖衣物",
            "keyword": "",
            "changetime": "2020-08-07T03:34:48Z"
        }
    ]
    
    # 找全部景點部分欄位
    # http://ttime-demo.ivankaoblog.com:8000/api/travel/places/position/
    [
        {
            "json_d": "C1_315081800H_000071",
            "name": "萊萊秘境",
            "toldescribe": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "description": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "picture1": "https://www.necoast-nsa.gov.tw/FileArtPic.ashx?id=680&w=1280&h=960",
            "px": "121.99421",
            "py": "25.00192"
        },
        {
            "json_d": "C1_315081700H_000161",
            "name": "李子園小舖",
            "toldescribe": "東山李子園清光緒年間，先民由大陸搬遷到此定居，將帶來的李子種子種在自家門前。那時家家戶戶皆有李子樹，彷彿一片李子園，因而將此地命名為李子園。隨著時代的變遷與農業結構改變，當地農民改種植香蕉與椪柑等農作物，為何現今李子園會種植咖啡呢？茲因早期日本人發現東山區的氣候與土壤適合栽種「阿拉比卡」咖啡豆，遂而引進咖啡開始種植培育，當時喝過咖啡的人都說「這東西聞起來很香，但是喝起來好苦，不好喝。」昔日那個年代對於咖啡的接受度很低，因此東山咖啡發展也就此中斷。早年被嫌棄的咖啡，現今喝咖啡卻成為一種時尚，一種品味，有些人更視為必需品，每天不喝一杯咖啡就混身不自在；也因為如此，東山在地的農民想起昔日，日本人曾經在此栽種咖啡，十年前開始重新種植咖啡，近年積極參與國際咖啡評鑑，獲得美國C.Q.I咖啡精品協會認證，品質大獲肯定，讓東山咖啡逐漸在國際市場展露頭角，成為台灣咖啡主要重要產地之一。由在地居民齊心打造的「李子園小舖」，是一處充滿藝術風情的溫馨旅遊資訊站。每當有旅人訪時，居民總是熱情邀約「進來做唷，來喝杯咖啡」，為路過旅人煮一杯香濃咖啡，好客心意配上手沖的溫度，只為了讓旅人感受自家烘焙的的好味道。除了品嚐東山咖啡香，李子園小舖也推出咖啡限定小旅行，帶領遊客體驗烘焙咖啡的樂趣，種下屬於自己的咖啡寶寶，還有機會體驗龍眼乾包咖啡豆的在地吃法，感受滿滿的山城人情味。",
            "description": "東山李子園清光緒年間，先民由大陸搬遷到此定居，將帶來的李子種子種在自家門前。那時家家戶戶皆有李子樹，彷彿一片李子園，因而將此地命名為李子園。隨著時代的變遷與農業結構改變，當地農民改種植香蕉與椪柑等農作物，為何現今李子園會種植咖啡呢？茲因早期日本人發現東山區的氣候與土壤適合栽種「阿拉比卡」咖啡豆，遂而引進咖啡開始種植培育，當時喝過咖啡的人都說「這東西聞起來很香，但是喝起來好苦，不好喝。」昔日那個年代對於咖啡的接受度很低，因此東山咖啡發展也就此中斷。早年被嫌棄的咖啡，現今喝咖啡卻成為一種時尚，一種品味，有些人更視為必需品，每天不喝一杯咖啡就混身不自在；也因為如此，東山在地的農民想起昔日，日本人曾經在此栽種咖啡，十年前開始重新種植咖啡，近年積極參與國際咖啡評鑑，獲得美國C.Q.I咖啡精品協會認證，...",
            "picture1": "https://www.siraya-nsa.gov.tw/image/615/640x480",
            "px": "120.50625",
            "py": "23.28691"
        }
    ]
    # 找部分欄位的景點的 name 欄位檢索
    # http://ttime-demo.ivankaoblog.com:8000/api/travel/places/position/?name=%E8%90%8A
    [
        {
            "json_d": "C1_315081800H_000071",
            "name": "萊萊秘境",
            "toldescribe": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "description": "坐擁隱藏版絕美海景、全台最大火龍岩與壯闊地質景觀，「萊萊秘境」不僅是一處美景之境，還是一個咖啡館！原本是福連國小萊萊分校廢校後遺留的舊建築物，後續經由管理處進行整修，並委託地方熱心人士經營，提供了點心、咖啡、果汁等多樣餐點，而這咖啡小館就位於舊草嶺環狀線自行車步道上，當騎乘自行車經過時，不妨坐下來喝杯咖啡、恢復體力，還可眺望遠方的龜山島，一片遼闊海景非常療癒！ 沿著咖啡館旁的小路往下走，才能通到美麗的萊萊海岸，也就是秘境傳說之地。近距離觀賞火龍岩，外形彷彿潛入海中的龍露出的背脊，屹立在平坦的沈積岩平台中，景觀甚是特殊。火龍岩據說是距今約2,700萬年前，因地殻破裂致海底岩漿噴發海面冷卻後所形成，是臺灣島上唯一成列一條龍的玄武岩層墻，附近還有大小不一的圓石散佈其旁，猶如恐龍蛋一般，也增添了火龍傳說的整體性。",
            "picture1": "https://www.necoast-nsa.gov.tw/FileArtPic.ashx?id=680&w=1280&h=960",
            "px": "121.99421",
            "py": "25.00192"
        }
    ]
    ```
    
2. 使用者登入後才可找到旅程相關資訊
    
    ```python
    # 未登入呼叫 api
    # http://ttime-demo.ivankaoblog.com:8000/api/travel/travels/
    {
        "detail": "Authentication credentials were not provided."
    }
    
    # token
    # http://ttime-demo.ivankaoblog.com:8000/api/token/
    {
        "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoicmVmcmVzaCIsImV4cCI6MTYyMTI3MTk1MywianRpIjoiZjFjYjVhNmQwNjZhNDA3OTllMWMxMWMzNWRlMjVmYWYiLCJ1c2VyX2lkIjoxfQ.55nz6YAlAtvBMHDOoj6NXDNtqW3FB1zY0TMMJXV9GEg",
        "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNjIwNDA3OTUzLCJqdGkiOiJiNDc0M2RiNTBjNDU0YjcxYTE4NWQ3YjgzM2E1NWNiNCIsInVzZXJfaWQiOjF9.fus014tQCh_HiKrsm2jakrDvVVrwT13CsJ93IAoFO90"
    }
    
    # 帶上 access 後呼叫 api
    # 找到每筆旅程的旅程建立者、旅程中的所有行程、行程中的地點
    # http://ttime-demo.ivankaoblog.com:8000/api/travel/travels/
    [
        {
            "id": 58,
            "schedule": [
                {
                    "id": 81,
                    "title": {
                        "json_d": "C1_315081700H_000160",
                        "name": "臺南山上花園水道博物館",
                        "toldescribe": "行經市道178號旁，稍不留意就會錯過坐落在大馬路旁的百年古蹟-原臺南水道，是一處見證大臺南地區水利發展的重要古蹟。退役後卸下光環，今日臺南水道蛻變為充滿歷史氛圍的自來水博物館，巴洛克式廠房保存良好，內部保留完善的機件組，都是難得一見的老古董，使人興起思古幽情。所謂水道就是自來水管道的五處設施，包括取水設施、沈澱池、濾過氣室，送出唧筒室以及淨水池，列為古蹟的歷史價值包括分列各處為建築體、西式鋼筋水泥、紅磚樓房、石砌平屋等等，還有保存完好的機件組，包括14座英國快濾筒及零組件、天井移動起重機，豎軸式電動機組等，這些機具在臺灣自來水產業的生產設備中，都是珍貴的文化遺產。參觀淨水池需登上189階的淨水池步道才能到達，外觀為天然石材與仿石塊兩種組合而成之灰色建物，形制固若碉堡。淨水池兩側各有一座水質檢驗室，頂部覆土與植披，設有59座鑄鐵通氣管柱。淨水池後方淨水井門上山牆中央有一圓形標誌，上有水泥切砌而成「南水」二字圖案，正是「臺南水道」標誌，為前來參觀的民眾，敘述著臺南水道曾經有過的輝煌歷史。《關於臺灣水道之父─濱野彌四郎》被譽為臺灣水道之父的濱野彌四郎（1863～1932年）生於日本千葉縣，東京帝國大學工業工程部畢業。1896年濱野彌四郎以英國人威廉‧巴爾頓的助手身分來臺，擔任臺灣總督府土木部技師。在臺工作的23年間，陸續參與並完成基隆、臺北、臺中、臺南等地的重要水道計畫，對於臺灣水道貢獻良多。",
                        "description": "行經市道178號旁，稍不留意就會錯過坐落在大馬路旁的百年古蹟-原臺南水道，是一處見證大臺南地區水利發展的重要古蹟。退役後卸下光環，今日臺南水道蛻變為充滿歷史氛圍的自來水博物館，巴洛克式廠房保存良好，內部保留完善的機件組，都是難得一見的老古董，使人興起思古幽情。所謂水道就是自來水管道的五處設施，包括取水設施、沈澱池、濾過氣室，送出唧筒室以及淨水池，列為古蹟的歷史價值包括分列各處為建築體、西式鋼筋水泥、紅磚樓房、石砌平屋等等，還有保存完好的機件組，包括14座英國快濾筒及零組件、天井移動起重機，豎軸式電動機組等，這些機具在臺灣自來水產業的生產設備中，都是珍貴的文化遺產。參觀淨水池需登上189階的淨水池步道才能到達，外觀為天然石材與仿石塊兩種組合而成之灰色建物，形制固若碉堡。淨水池兩側各有一座水質檢...",
                        "picture1": "https://www.siraya-nsa.gov.tw/image/625/640x480",
                        "px": "120.35999",
                        "py": "23.09681"
                    },
                    "date": "2021-03-02",
                    "from_time": "2021-03-02 08:00:00",
                    "to_time": "2021-03-02 10:00:00",
                    "created": "2021-03-27T05:38:05.390881Z",
                    "modified": "2021-03-27T05:38:05.391006Z",
                    "day": 1
                }
            ],
            "user": {
                "username": "ivankao"
            },
            "created": "2021-03-27T05:38:05.387585Z",
            "modified": "2021-03-27T05:38:05.387764Z",
            "title": "第二趟旅程",
            "description": "yayaya",
            "url": "",
            "category": "",
            "subcategory": "",
            "author": ""
        },
    ]
    ```
    
    ![images/Untitled.png](images/Untitled.png)
    

## 開始 DRF

[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)

[https://django-rest-framework-simplejwt.readthedocs.io/en/latest/index.html](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/index.html)

1. 設定檔配置
    
    ```python
    
    INSTALLED_APPS = [
        ...
    		'django_filters',
        'rest_framework', 
        'rest_framework.authtoken'
    ]
    
    CORS_ORIGIN_ALLOW_ALL = True
    
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES':
            {'rest_framework.permissions.IsAdminUser'
             'rest_framework.permissions.IsAuthenticated', },
        'DEFAULT_AUTHENTICATION_CLASSES':
            (
                'rest_framework_simplejwt.authentication.JWTAuthentication', # JWT
                'rest_framework.authentication.TokenAuthentication',
                'rest_framework.authentication.SessionAuthentication',
                'rest_framework.authentication.BasicAuthentication',
            ),
        'DEFAULT_FILTER_BACKENDS': ['django_filters.rest_framework.DjangoFilterBackend']
    }
    
    SIMPLE_JWT = {
        'ACCESS_TOKEN_LIFETIME': timedelta(days=10),
        'REFRESH_TOKEN_LIFETIME': timedelta(days=20),
        'ROTATE_REFRESH_TOKENS': False,
        'BLACKLIST_AFTER_ROTATION': True,
        'ALGORITHM': 'HS256',
        'VERIFYING_KEY': None,
        'AUDIENCE': None,
        'ISSUER': None,
        'AUTH_HEADER_TYPES': ('Bearer', ),
        'USER_ID_FIELD': 'id',
        'USER_ID_CLAIM': 'user_id',
        'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken', ),
        'TOKEN_TYPE_CLAIM': 'token_type',
        'JTI_CLAIM': 'jti',
        'TOKEN_USER_CLASS': 'rest_framework_simplejwt.models.TokenUser',
        'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
        'SLIDING_TOKEN_LIFETIME': timedelta(days=10),
        'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=20),
    }
    ```
    
2. 定義 model 欄位
    
    ```python
    # 整趟旅程
    class Travel(TimeStampedModel):
        user = models.ForeignKey(User, on_delete=models.CASCADE) # 關聯 user
        title = models.CharField(max_length=80, blank=True)
        description = models.TextField(max_length=300, blank=True)
        url = models.URLField(blank=True)
        category = models.CharField(max_length=50, blank=True)
        subcategory = models.TextField(max_length=50, blank=True)
        author = models.TextField(max_length=50, blank=True)
        schedule = models.ManyToManyField('travel.Schedule', blank=True, verbose_name='行程', related_name='travel_set')
    
    # 旅程中的每個行程
    class Schedule(TimeStampedModel):
        title = models.ForeignKey(
            'travel.Place', null=True, blank=True, on_delete=models.CASCADE, verbose_name='地點', related_name='schedule_set'
        ) # 關聯 place
        date = models.DateField('日期')
        day = models.IntegerField('天數')
        from_time = models.DateTimeField('時間')
        to_time = models.DateTimeField('時間')
    
        class Meta:
            verbose_name = '行程'
            verbose_name_plural = '行程'
    
        def __str__(self):
            return "%s" % (self.title)
    
    # 地點
    class Place(TimeStampedModel):
        json_d = models.CharField(max_length=80, null=True, blank=True)
        name = models.CharField(max_length=80, null=True, blank=True)
        zone = models.CharField(max_length=80, null=True, blank=True)
        toldescribe = models.TextField(max_length=300, null=True, blank=True)
        description = models.TextField(max_length=300, null=True, blank=True)
        add = models.CharField(max_length=80, null=True, blank=True)
        zipcode = models.CharField(max_length=80, null=True, blank=True)
        region = models.CharField(max_length=80, null=True, blank=True)
        town = models.CharField(max_length=80, null=True, blank=True)
        tel = models.CharField(max_length=80, null=True, blank=True)
        travellinginfo = models.TextField(max_length=300, null=True, blank=True)
        opentime = models.CharField(max_length=300, null=True, blank=True)
        website = models.CharField(max_length=300, null=True, blank=True)
        picture1 = models.URLField(blank=True)
        picdescribe1 = models.TextField(max_length=300, null=True, blank=True)
        picture2 = models.URLField(blank=True)
        picdescribe2 = models.TextField(max_length=300, null=True, blank=True)
        picture3 = models.URLField(blank=True)
        picdescribe3 = models.TextField(max_length=300, null=True, blank=True)
        gov = models.CharField(max_length=80, null=True, blank=True)
        px = models.CharField(max_length=80, null=True, blank=True)
        py = models.CharField(max_length=80, null=True, blank=True)
        orgpclass = models.CharField(max_length=80, null=True, blank=True)
        pclass1 = models.CharField(max_length=80, null=True, blank=True)
        pclass2 = models.CharField(max_length=80, null=True, blank=True)
        pclass3 = models.CharField(max_length=80, null=True, blank=True)
        map = models.CharField(max_length=80, null=True, blank=True)
        parkinginfo = models.CharField(max_length=300, null=True, blank=True)
        parkinginfo_px = models.CharField(max_length=80, null=True, blank=True)
        parkinginfo_py = models.CharField(max_length=80, null=True, blank=True)
        ticketinfo = models.TextField(max_length=300, null=True, blank=True)
        remarks = models.TextField(max_length=300, null=True, blank=True)
        keyword = models.TextField(max_length=300, null=True, blank=True)
        changetime = models.DateTimeField(max_length=80, null=True, blank=True)
    ```
    
3. Using ModelSerializers
    
    [https://www.django-rest-framework.org/tutorial/3-class-based-views/](https://www.django-rest-framework.org/tutorial/3-class-based-views/) 
    
    [https://blog.csdn.net/qq_31742423/article/details/83241461](https://blog.csdn.net/qq_31742423/article/details/83241461)
    
    ```python
    # Place 所有欄位
    class PlaceSerializer(serializers.ModelSerializer):
        class Meta:
            model = Place
            fields = (
                'json_d', 'name', 'zone', 'toldescribe', 'description', 'add', 'zipcode', 'region', 'town', 'tel',
                'travellinginfo', 'opentime', 'website', 'picture1', 'picdescribe1', 'picture2', 'picdescribe2', 'picture3',
                'picdescribe3', 'gov', 'px', 'py', 'orgpclass', 'pclass1', 'pclass2', 'pclass3', 'map', 'parkinginfo',
                'parkinginfo_px', 'parkinginfo_py', 'ticketinfo', 'remarks', 'keyword', 'changetime'
            )
    
    # Place 部分欄位
    class PlacePositionSerializer(serializers.ModelSerializer):
        class Meta:
            model = Place
            fields = ('json_d', 'name', 'toldescribe', 'description', 'picture1', 'px', 'py')
            read_only_fields = ('name', 'toldescribe', 'description', 'picture1', 'px', 'py')
    
    # User 關聯欄位
    class UserRelatedSerializer(serializers.ModelSerializer):
        class Meta:
            model = get_user_model()
            fields = ['username']
            extra_kwargs = {'username': {'validators': [UnicodeUsernameValidator()], }}
    
    # Schedule 關聯欄位
    class ScheduleSerializer(serializers.ModelSerializer):
        title = PlacePositionSerializer()
        date = serializers.DateField(format="%Y-%m-%d")
        from_time = serializers.DateTimeField(format="%Y-%m-%d %H:%M:%S")
        to_time = serializers.DateTimeField(format="%Y-%m-%d %H:%M:%S")
    
        class Meta:
            model = Schedule
            fields = '__all__'
    
    # 旅程
    class TravelSerializer(serializers.ModelSerializer):
        schedule = ScheduleSerializer(many=True)
        user = UserRelatedSerializer()
    
        class Meta:
            model = Travel
            fields = '__all__'
    
        # level3 nested, level12 m2m,level23 fk
        def create(self, validated_data):
            user_data = validated_data.pop('user')
            username = user_data.pop('username')
            user = get_user_model().objects.get(username=username)
    
            # creat schedule_data , and rm schedule in validated_data
            schedule_data = validated_data.pop('schedule')
            # create travel without schedule
            travel = Travel.objects.create(user=user, **validated_data)
            # forloop level2 (schedule)
            for place_data in schedule_data:
                # creat place_data , and rm title in place_data
                places = place_data.pop('title')
                # level3 get  fk place name
                place = Place.objects.get(json_d=places['json_d'])
                # level2 create schedule with fk titile=place
                schedule = Schedule.objects.create(title=place, **place_data)
                # level1 travel add m2m schedule
                travel.schedule.add(schedule)
            return travel
    ```
    
4. Writing regular Django views using our Serializer
    
    ```python
    class PlaceViewSet(viewsets.ModelViewSet):
        queryset = Place.objects.all()
        permission_classes = (IsAuthenticatedOrReadOnly, )
        filterset_class  = PlaceFilter
        lookup_field='json_d'
    
        def get_serializer_class(self): # select action to serializer
            if hasattr(self, 'action') and self.action == 'list':
                return serializers.PlaceSerializer
            if hasattr(self, 'action') and self.action == 'retrieve':
                return serializers.PlacePositionSerializer
    
        def filter_queryset(self, queryset): # select action to query
            if self.action == 'list':
                self.filterset_class = PlaceFilter
            elif self.action == 'position':
                self.filterset_class = PlaceFilter
            return super().filter_queryset(queryset)
    
        @action(detail=False) # list object
        def position(self, request):
            queryset = self.filter_queryset(self.get_queryset())
            serializer = serializers.PlacePositionSerializer(queryset, many=True)
            return Response(serializer.data)
    
    class TravelViewSet(viewsets.ModelViewSet):
        queryset = Travel.objects.all()
        permission_classes = (IsAuthenticated, )
    
        def get_queryset(self):
            queryset = Travel.objects.filter(user=self.request.user)
            return queryset
    
        queryset = Travel.objects.all()
        serializer_class = serializers.TravelSerializer
    ```
    
5. router
    
    ```python
    from django.urls import include, path
    from rest_framework import routers
    from .views import TravelViewSet, PlaceViewSet 
    
    router = routers.DefaultRouter()
    router.register('travels',TravelViewSet)
    router.register('places',PlaceViewSet)
    
    urlpatterns = [
        path('', include(router.urls)),
    ]
    ```
    

reference:

[https://progressbar.tw/posts/53](https://progressbar.tw/posts/53)