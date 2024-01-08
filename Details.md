### HCI Backend

這個項目的目的是要創造出一個構造可視化知識圖譜的網站，可以由用戶自己導入文件來生成對應知識圖譜並根據圖譜來回答問題。知識圖譜的好處在於能快速找出節點與節點之間的關係而不用通過複雜的遞歸搜索，還有高擴展性和符合自然語言表達等。
整個項目是采取前後端分離開發，前端使用Vue; 後端使用Spring boot，CI/CD用Jenkins進行自動部署和測試等操作，還有使用其他工具及服務後續一一介紹。我是負責整個後端開發，包括數據庫設計和連接，及業務邏輯模塊的實現。下面進行詳細的解釋:

整體後端邏輯可分爲Controller(接受和回復前端請求)、BL/BLImpl(業務邏輯模塊，為主要實現邏輯的地方。前者為定義Interface，後者為實現這些接口的方法)、PO/VO/DAO(三種類型的Object: DAO為Data Access Object為實現Database的Object、PO為Persistent Object是位於DAO和VO間的類型、VO則是View Object為表現在前端的類型)、Util(為工具類，主要包含Aliyun OSS服務: Object Storage Service為存儲少量大型的檔案的服務。還有NLP處理的工具類)

#### Backend Implementation

要解釋**後端邏輯**實現需要先説明前端提供哪些功能: 除了最基本的節點CRUD外，還有提供Q&A和Recommend。

因此數據庫設計為Entity, Relation, Position, User等表來操作。主要説明Q&A和推薦功能:

**Q&A分爲五個步驟:**

1. 先將問題進行分詞/關鍵詞提取(使用Java NLP第三方庫)，得到候選的實體或關係
2. 將候選的實體或關係通過同義詞映射為已存在的實體或關係
3. 將候選的實體或關係計算出優先級
4. 根據characters.txt和relations.txt可以得到確定的、已存在的實體或關係列表
5. 將優先級高到低的關係和實體根據**匹配規則**得出結果

匹配規則(匹配規則内也有優先級，數字越低優先越高)：

1. 擁有實體或關係，得出另一個實體
2. 擁有兩個實體，得出其中可能存在的關係
3. 擁有一個實體，得出實體屬性

**推薦功能**則是根據上一次搜索的節點或關係，計算相似度最高的10個節點，大致步驟如下:

1. 計算並得出匹配度最高的節點列表
2. 根據節點列表，交叉計算出匹配度最高的關係列表
3. 最後再通過關係列表得出匹配度最高的10個節點

#### Test

使用Junit添加單步測試和系統測試，也就是在BLImpl和Controller層添加測試用例

#### Jenkins

架設流水綫並分爲四個步驟:

1. clone: 自動從github上提取代碼並放置在服務器中
2. test: 包括執行Junit test和mvn檢查，還有添加jacoco獲得代碼覆蓋率來校驗開發品質(80%)
3. build: 執行maven install指令生成war包(依賴庫)
4. develop: 運行develop.sh脚本，將項目生成好的war包添加進tomcat目錄下后執行/bin/startup.sh

#### Swagger2, Navicat, OSS

使用Swagger2自動生成API，方便前後端校驗；本地開發時仍有使用Postman進行第一步測試；數據庫則使用Navicat做本地測試; 阿里雲OSS服務則是用於存儲分詞字典，由於字典並不會根據項目本身修改而改變，再加上本身字典體積很大，所以考慮用OSS服務來簡化每次Update項目的過程和時間