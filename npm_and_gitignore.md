# 套件之管理（以 npm 為例）
## 專案（套件）的包裝紙——package.json
當我們拿起超商架上的麵包，常會翻到背面看看有什麼成分吧？「package.json」就是這樣的東西，它闡明了這個專案（套件）的基本資訊，從開宗明義的「名稱」到鉅細靡遺的「專案貢獻者」皆可被其收錄。package.json 包羅萬象的屬性[可至 Node.js 的文件](http://nodejs.cn/learn/the-package-json-guide)查閱。

我們可以借由**套件管理系統**（以下以 npm 為例）來產生 package.json 檔，而無須手動謄寫。使用指令 ```npm init``` 可以引導生成一個初步的 package.json，而若在專案中安裝了*某套件*，*某套件*亦會被寫在 package.json 中的「相依套件（dependencies）」欄位。事實上，只要是套件，都會附上一個 package.json，而仔細觀察這個欄位，套件們的「裙帶關係」便一覽無疑：套件、套件所仰賴的套件、套件所仰賴的套件所仰賴的套件……真是買一送十。

## 凍結套件們的版本——package-lock.json
如果不小心把專案中存放套件的 node_modules 資料夾刪個精光，別緊張，只要輸入指令 ```npm install``` 便能回復如初！

npm 藉由讀取「package-lock.json」裡所記載的套件版本，一一將其裝回。相較於 package.json 在相依套件欄中使用了[「語意化版本（semver）」](https://semver.org/lang/zh-TW/)來紀錄版本，package-lock.json 只寫下每個套件的確切版本，藉此，我們可以隨時將套件版本回復至初始樣貌（如果更新出了問題的話……）。

> *注意：   
指令 ```npm update``` 會遵照 package.json 的語意化版本更新套件，並同時更新 package-lock.json。*

## 放該放的東西——.gitignore
既然 npm 能幫我們取得所需的套件，資料夾 **node_modules 便無須上傳至 Git 遠端庫**，而應以 package.json 及 package-lock.json 取而代之。

要使 git 略過檔案或資料夾十分簡單，只需在目錄中新增一個檔名為「.gitignore」的檔案，並在其中列明不想被 git 控制的項目即可。比方我們想略過資料夾 node_modules，那麼 .gitignore 應該要長這樣：

```
# Dependency directories
node_modules/
```

其中，「```#```」後的文字為註解，方便讓後人瞭解其排除之原因。常見應被 git 略過的項目如下：
- 暫存檔
- 記錄檔
- 機密檔案
- 相依套件存放目錄

> *那麼「.gitignore」本身呢？   
當然不必上傳至遠端庫便能有效果，但若欲使專案保持一致，還是上傳吧！*

其實，早已有人整理出適用於各式專案的 .gitignore 檔案，我們可以直接參照，並加以增刪以符合自身使用情境。
- [由 GitHub 提供的 .gitignore](https://github.com/github/gitignore)

## 把套件當成免洗餐具——npx
打開手機，Play 商店與 App Store 其實就是套件管理系統！我們會在上面下載並安裝五花八門的應用程式，但平常用到的根本沒幾個。於是，Google 與蘋果兩者不約而同地推出了 Google Play Instant 與 App Clips，讓使用者無須完整安裝便能存取必要功能，必且在使用後自動清除，不占用儲存空間！這麼棒的功能，其實 npm 也有，那就是 npx！

```
npx cowsay MONEY GET AWAY -g
 ________________
< MONEY GET AWAY >
 ----------------
        \   ^__^
         \  ($$)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

（這個牛身下腹部的「w」是什麼？）

使用指令 ```npx```，我們可以讓套件在執行完畢後便刪除，這對於一次性的任務特別有用（如：使用 @vue/cli 建立一個 vue 專案），畢竟這種一年只用幾次的工具，何必 365 天都留在電腦裡？

除了「用後即刪」，npx 尚提供「快速呼叫套件」的功能。如果我們已在本機安裝了某套件，```npx 某套件``` 將可直接執行之，無論該套件安裝於專案中或是全域內。而若未安裝，則如前段所述，npx 將為您下載、執行、移除。