# JavaScript 的事件循環是什麼？
## 工欲善其事，必先利其器
通常來講，JavaScipt 必須仰賴執行環境（Runtime environment）以執行，最常見的環境當屬「網頁瀏覽器」。

* 以 Google Chrome 而言，其內建之 JavaScipt 執行環境之**核心**為「V8 引擎」
    * Node.js 亦採用該引擎

* V8 引擎幹了什麼？
    * 分配堆積（Heap）記憶體
        * 一塊無結構且夠大的記憶體
        * 程式需要的各種變數將堆積於此
    * 呼叫堆疊（Stack）
        * 堆疊是一種**先進後出**的資料結構
        * 引擎閱讀了 JavaScript 指令後便將函式堆疊於此
        * 若函式又呼叫函式，被呼叫的函式將被置於頂
        * 執行完成之函式會自堆疊移除，此時引擎會返回上層函式
        * 可說是一張**步驟紀錄**表

* [不是只有 V8 引擎！](https://w.wiki/3PBj)

* 有引擎就好嗎？
    * 引擎僅是核心，其包含於執行環境中
    * 就像一輛汽車，除了引擎之外，尚須輪子、儀表、冷氣、音響系統、<del>加熱座椅</del>等
    * 就網頁瀏覽器來說，**Web API** 便非由引擎提供
        * BOM
        * DOM
        * AJAX
        * setTimeout
        * …
    * 還有將這些串在一起的…
        * 事件循環（Event loop）
        * 回調佇列（Callback queue）

## 一次只做一件事
實作 JavaScript 的引擎的**呼叫堆疊**是「單執行緒」的，這意味著一次只能做一件事，**但事實並非如此**…

* 非阻塞
    * 什麼是**阻塞**？
        * 當**堆疊**被很慢的任務佔住了，就可稱「阻塞」

    * 但 JavaScript 通常是非阻塞的，看看這個例子：

    ```javascript
    console.log("A");

    setTimeout(() => {
        console.log("B");
    }, 3000);

    console.log("C");
    ```

    若 JavaScript 真這麼專心致志，那麼上面這段程式的執行結果應是：
    > A -***漫長的 3000 毫秒***-> B -> C

    但眾所周知，結果是：
    > A -> C -***2xxx 毫秒***-> B

    由此可見，程式並未因等候 ```setTimeout``` 而塞車，它確實執行了設定的逾時，但時間還沒到，後面的函式便緊接著執行，這就是「非阻塞」。但單執行緒模式的 JavaScript 是怎麼辦到這點的？

## 執行緒池（Thread pool）
執行環境其實還開了個「執行緒池」，包含先前提到的 Web API、事件循環、回調佇列等，他們皆在引擎運行的呼叫堆疊的**執行緒之外**。

```javascript
console.log("A");

setTimeout(() => {
    console.log("B");
}, 0);

console.log("C");
```

再舉一次例子，這次將 ```setTimeout``` 設為 0 毫秒，執行的結果如何呢？
> A -> C -> B

都說 0 秒了，B 依舊殿後！？

* setTimeout 是非同步的
    * 當堆疊執行至 ```setTimeout``` 時，堆疊將之丟給 Web API（這本來就是它的工作）
    * 丟完以後，```setTimeout``` 便從堆疊移除，隨後執行 ```console.log("C")```

* 回頭看看 Web API 裡的 setTimeout
    * 按照我們給予的參數「0 毫秒」倒數計時
    * 因為是 0 秒，這個碼錶才剛按開始就要按結束了
    * 總之，Web API 做完了它的工作，將 ```setTimeout``` 中的 ```console.log("B")``` 丟給「回調佇列」

* 「回調佇列」終於登場
    * 其恰與堆疊相反，是一種後進後出的資料結構（讓我想起羽毛球罐）
    * 它就像是堆疊的代辦事項清單
    * 當堆疊一有空閒（空空如也時），便會執行佇列裡的項目
    * 那誰來督促堆疊？

* 「事件循環」姍姍來遲
    * 雖說是姍姍來遲，其實它一直運轉
    * 其不停巴望著佇列
    * 一但佇列有任務，事件循環便伺機想將任務交予堆疊
    * 這個時機即堆疊為空時
    * 終於，```console.log("B")``` 被事件循環丟回堆疊，你我都看到了「B」！

* 執行完畢

## 別阻塞堆疊！
有了上列概念，我們瞭解到藉由**回調佇列**與**事件循環**的幫忙，可讓堆疊鬆了一大口氣。但並所有複雜的事都會被分配在諸如 Web API 等地方執行，比方我們可以寫一個 ```while``` 霸佔堆疊，這會發生什麼事？

* 網頁的彩現（Render）
    * 就 60 FPS 而言，最流暢的畫面理想是每 16.6 毫秒更新一次
    * 這種「彩現」請求根本就是一種回調，且優先於先前提到的回調佇列
    * 但它到底還是要等待堆疊清空才能更新畫面（彩現）
    * 所以「別阻塞堆疊！」
        * 否則整個網頁很可能會卡卡的…

* 如何不阻塞？
    * 讓緩慢的函式以非同步的方式執行

## 後記
前陣子進行期中專題，便深刻體悟到 jQuery 提供的 AJAX 的非同步的特性。當時為了實現即時查詢空房，將無房可用的日期傳送給日期選擇器（使該日反灰），而使用了 AJAX 取得指定條件（房型、人數）下的空房資料。然由於觸發查詢的事件亦同時觸發展開日期選擇器，因此**回傳的資料總是慢選擇器半拍**；雖然拿到資料了，但選擇器哪會回頭讀取？於是日期一直無法變灰。最後是加上 ```async: false``` 才得以解決，但這樣便「阻塞」了堆疊，不是個很好的做法，但也許是必要之惡？「（轉圈圈圖示）正在載入…」大概就是用在此處…

## 參考資料
1. [Philip Roberts, JSConf EU. (2014). 所以說event loop到底是什麼玩意兒？](https://youtu.be/8aGhZQkoFbQ)
2. [MDN contributors. (2021). 並行模型和事件循環](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/EventLoop)
3. [Gbadebo Bello. (2021). How does the Event Loop works in JavaScript?](https://geekflare.com/javascript-event-loops)
4. [Daniel. (2019). The JavaScript runtime environment](http://dolszewski.com/javascript/javascript-runtime-environment)
5. [Sam. (2019). JS 同步、異步、阻塞、非阻塞](https://medium.com/@mts40110/js-%E5%90%8C%E6%AD%A5-%E7%95%B0%E6%AD%A5-%E9%98%BB%E5%A1%9E-%E9%9D%9E%E9%98%BB%E5%A1%9E-29e1e1c0193e)
