# JavaScript Promise 雜記
## 一步一步來
[之前的筆記](https://lzmxya.github.io/js_event_loop_review)提到了 JavaScript 「非同步」的特性，一個非同步的函式，會在別處處理完成後，再扔至佇列等候回呼。與此同時，後續任務仍馬不停蹄地在堆疊上執行，是為「非同步」。但假如今天我們想讓程式的流程依序執行呢？

## 何時需要依序執行？
「從遠端獲取資料」是最常見的例子。比方今天要讓使用者訂購一張電影票，那麼便必須等候戲院回傳可用的空位後，才可讓使用者劃位。要實現此舉有諸多方法，最直截了當的方式是直接將欲接著進行的函式置於先決函式之中，此即「回呼」。

## 回呼深深深幾許——callback hell
直接將函式嵌套在函式內固然可行，但一旦程式愈發龐大，無止盡的巢狀結構將教人難以閱讀，更遑論修改了。

```javascript
fnA() {
    console.log("A done.");
    fnB() {
        console.log("B done.");
        fnC() {
            console.log("C done.");
        };
    };
};
```

上列例子表示了 ```fnA()``` 先呼叫 ```fnB()```，```fnB()``` 再呼叫 ```fnC()``` 的情況。如果今天想更改其呼叫次序，那將是一件如「構造改革」般的大工程。

## Promise！
於是 Promise 應運而生。其為一個「非同步任務之成敗」的物件，透過呼叫它，我們能執行其中之任務，並靜候其執行完畢（settled：解決），而得知其是**成（resolve：實現）**或**敗（reject：拒絕）**。上述的結果，尚可傳遞給後續 Promise 物件，使其值被其他函式利用，如此，程式流程便能按照我們理想的方式進行。

由於其為「非同步任務之成敗」的物件，因此，由 Promise 呼叫的函式仍是「非同步」的。這使得任務能同步般地執行，但依然不阻塞堆疊。

## 將函式包裹進 Promise 裡
由於 Promise 是近年（ES6）才收錄的新方式，一些舊的 API（如：setTimeout）並未原生支援，但好在我們可以輕易地將之封裝進一個 Promise 物件中，如下所示：

```javascript
function finalCountdown(sec) {
    return new Promise((resolve, reject) => {
        if(sec === 0) reject(new Error("請大於 0 秒再來倒數！"));
        setTimeout(() => {
            resolve("倒數完畢！");
        }, sec);
    })
};
```

對於一個 Promise 執行完的後果，「怎樣算是成功？怎樣算是失敗？」是由我們定義的。我們將成功的情形的回傳值包入 ```resolve()``` 中，失敗則是 ```reject()```。

至此，一個 Promise 版的 setTimeout 已經建構完成。我們隨後可透過 ```.then()``` 與 ```.catch()``` 的方法來取得其執行結果。 

> *註：   
成敗之回呼並非一定要取名為 ```resolve``` 與 ```reject```，只需符合首個參數為實現時之回呼，次個參數為拒絕時之回呼即可。*

## and then??
如果今天朋友講了一個冷笑話（即便已經知道他的老梗），待他講完以後，我們就可以~~不以為然地~~問他「then?」，然後，朋友便會悻然地為您解釋方才說的笑話的笑點在哪裡……

這個「講笑話」就好比「建構一個 Promise」，當其建構完成後，若我們不對其詢問「然後呢？」，那麼它就像薛丁格貓的「疊加態」一般，將之直接印出會告訴我們 Promise 的狀態為「```pending```」。

## 揭曉 Promise 的結果
要得知 Promise 的結果，我們需借助以下**方法**：

- ```.then()```
    - 將可獲得 Promise 中 ```resolve()``` 裡的值
    - 也就是執行「成功」的結果
    - 可繼續串接 ```.then()```，以妥善運用回傳值
- ```.catch()```
    - 將可獲得 Promise 中 ```reject()``` 裡的值
    - 也就是執行「失敗」的結果
        - 如：錯誤訊息

無論以 ```.then()``` 串接了多少個 Promise，一旦接收到 ```reject```，都會被 ```.catch()``` 攔截而顯現，並且，之後的 ```.then()``` 將被略過。

## 一失足成千古恨？
由前段可知，若 Promise 鍵結中出了「拒絕」的結果，流程便會直接跳至「```.catch()``` 階段」，即使出錯處後所銜接的 ```.then()``` 是有值的。縱然，我們將 Promise 物件連在一起，便是為了確保「先決任務」完成了，才進行下一步；但若在某些情境下，我們想讓每件任務都被執行，該怎麼做呢？

- ```.catch()``` 後再 ```.then()```
    - 事實上，```.catch()``` 後依然能繼續串接 ```.then()```
    - 同樣地，一旦出現「拒絕」便會直接進入 ```.catch()```
    - 因此，若要讓每個 ```.then()``` 都**一定**被執行，那麼擇需「逢 then 便 catch」
- 在 ```.then()``` 內處理「拒絕」
    - 方法 ```.then()``` 其實尚有第二個參數
    - 第一欄參數，也就是我們平常的用法，是用來承接「實現」的值
    - 若欲承接「拒絕」，則是寫在第二欄參數中

    ```javascript
    finalCountdown(13).then(function(value) {
    // 成功（實現）時之處裡方式
    }, function(reason) {
    // 失敗（拒絕）時之處裡方式
    });
    ```

以上兩種方式，皆可讓「錯誤處理」更為靈活。我們能藉此使接收到的「拒絕」的值被進一步處理。在 MDN Web Docs 上的[〈Promise.prototype.then()〉一文](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)指出：

> 實務上，使用 catch 捕捉被否決的 promise 較理想的，而不建議使用兩個引數 then 語法……（後略）

## 更接近人話——async function
ES7 推出了新的「async function」，其目的是為了將非同步函式的編排寫得像是同步函式般。async function 是基於 Promise 的語法裝飾，因此，要寫一個 async function，必須先建構一個 Promise 物件。而任意的 Promise 都可被改寫為 async function 的形式，使之看起來更簡單。我們將剛剛建立的 Promise「
```finalCountdown()```」改以 async function 來執行：

```javascript
(async function () {
    try{
        let result = await finalCountdown(13);
        console.log(result);
    } catch (error) {
        console.error(error);
    }
}());
```

要使用「```await```」，其必被包裹於 async function 內，而所謂的  async function，就是在開頭冠上一個「```async```」而已。

當某一非同步函式被冠上了 ```await```，執行時便會等候該函式背後的 Promise 物件回傳「值」之後，才會繼續執行，這等同於原本的 ```.then()``` 或 ```.catch()```，因此，改寫成 async function 的 Promise 組合可以完全不使用到 ```.then()``` 方法。

由於 async function 裡沒有像 ```.catch()``` 這樣的方法來專責處理承接到「拒絕」時的狀況，因此，我們可以使用「try...catch 陳述式」來處理錯誤。

## Promise 的方法
前述 ```.then()``` 及 ```.catch()``` 為對 Promise 的原型（prototype）的方法，而 Promise **本身**有多種可用的方法：

- 並行執行 Promise
    - ```.all()```
        - 可將多個 Promise 放入陣列中，再將此陣列作為方法之參數
        - 所有 Promise 將同時開始**並行**
        - 回傳值將以相同順序被存於陣列中
        - 不過，一旦任一 Promise 為「拒絕」，```.all()``` 將立即終止，並呈現之
    - ```.allSettled()```
        - 與 ```.all()``` 之相異為：其將回傳所有執行結果
        - 即使其中有「拒絕」
    - ```.any()```（*草案階段*）
        - 只要有 Promise 被「實現」，便終止並呈現之
        - 如全未能實現，回傳「拒絕」
        - 可說是與「```.all()```」相反
    - ```.race()```
        - 亦可傳入多個 Promise
        - 一旦一有結果，便終止執行，並呈現結果

    > *```.any()``` 與 ```.race()``` 之差別在於：   
「race」無論成敗，誰最快跑完便回傳；「any」則是只要有一被實現之 Promise 便回傳，若皆無，則回傳拒絕。*
    
- 建立靜態 Promise 物件（*而非透過 ```new Promise``` 建構*）
    - ```.reject()```
        - 回傳一個結果為「拒絕」的 Promise
    - ```.resolve()```
        - 回傳一個結果為「成功」的 Promise

## 最後…….finally()
Promise 原型的方法除了 ```.then()``` 與 ```.catch()``` 外，尚有 ```.finally()```（其實還有一個 ```.done()```）。其用途為無論 Promise 之結果為成功或失敗，都必須執行之任務（如：確認 Promise 組合結束，或清理工作）。由於其被設計用於不在乎過去的結果為何的情況，因此 ```.finally()``` 沒有參數可填。

Promise 組合若是以 async function 的形式撰寫，亦可將在「try...catch 陳述式」的尾端加上 ```finally{}```，即可處理善後任務。

## 參考
1. [MDN contributors. (2021). Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
2. [Jake Archibald. (2013). JavaScript Promises: An introduction](https://web.dev/promises/)
2. [王志誠. (2020). JavaScript Promise 全介紹](https://wcc723.github.io/development/2020/02/16/all-new-promise/)
3. [MDN contributors. (2021). async function](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/async_function)
