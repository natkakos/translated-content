---
title: 'Express 教學 3: 使用資料庫 ( Mongoose)'
slug: Learn/Server-side/Express_Nodejs/mongoose
translation_of: Learn/Server-side/Express_Nodejs/mongoose
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/Server-side/Express_Nodejs/skeleton_website", "Learn/Server-side/Express_Nodejs/routes", "Learn/Server-side/Express_Nodejs")}}</div>

<p>本文簡短介紹數據庫，以及如何搭配 Node / Express 應用，使用數據庫。接下來會演示我們如何使用 <a href="http://mongoosejs.com/">Mongoose</a>，為本地圖書館提供數據庫存取。本文說明物件要求與模型如何宣告，主要的欄位型態，以及基本驗證。本文也簡短演示一些存取模型數據的主要方法。</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row">前置條件:</th>
   <td><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/skeleton_website">Express 教學 2: 創建一個骨架網站</a></td>
  </tr>
  <tr>
   <th scope="row">目標:</th>
   <td>能夠使用Mongoose設計並創造自己的模型。</td>
  </tr>
 </tbody>
</table>

<h2 id="概覽">概覽</h2>

<p>圖書館職員會使用本地圖書館網站，存放書本和借書者訊息。圖書館使用者會用網站瀏覽與尋找書本，看看是否有可以藉閱的書本複本，然後預約或者藉閱。為了有效率地存放與取用訊息，我們將把它存放到數據庫。</p>

<p>Express 應用可以使用許多不同的數據庫，並且有好幾種方法可以執行創建 <strong>C</strong>reate、讀取 <strong>R</strong>ead、更新 <strong>U</strong>pdate 和刪除 <strong>D</strong>elete (CRUD) 操作。本教程為一些可用的選項，提供簡短的概覽，然後接著詳細演示該選項的特定運行機制。</p>

<h3 id="我可以使用什麼數據庫">我可以使用什麼數據庫?</h3>

<p>Express 應用程序可以使用 Node 支持的任何數據庫（Express 本身不會為數據庫管理，定義任何特定的附加行為/要求）。有許多<a href="https://expressjs.com/en/guide/database-integration.html">流行的選項</a>，包括 PostgreSQL，MySQL，Redis，SQLite 和 MongoDB。</p>

<p>在選擇數據庫時，您應該考慮時間 - 生產力/學習曲線，性能，易複製/備份，成本，社區支持等等。雖然沒有單一的 “最佳” 數據庫，但幾乎任何流行的解決方案，我們的本地圖書館這樣的中小型網站，應該都可以接受。</p>

<p>有關選項的更多訊息，請參閱：<a href="https://expressjs.com/en/guide/database-integration.html">數據庫集成（Express docs）</a>。</p>

<h3 id="與數據庫互動的最好方式是什麼">與數據庫互動的最好方式是什麼?</h3>

<p>有兩種與數據庫互動的方法：</p>

<ul>
 <li>使用數據庫的原生查詢語言（例如SQL）</li>
 <li>使用對像數據模型（“ODM”）/對象關係模型（“ORM”）。 ODM / ORM將網站的數據表示為JavaScript對象，然後將其映射到底層數據庫。一些ORM綁定到特定的數據庫，而另一些則提供了一個不特定數據庫的後端。</li>
</ul>

<p>通過使用 SQL 或數據庫支持的任何查詢語言，都可以獲得最佳性能。 ODM通常比較慢，因為它們使用翻譯代碼，在對象和數據庫格式之間進行映射，這可能不會使用最有效的數據庫查詢（尤其是如果ODM支持不同的數據庫後端，並且必須在各個數據庫所支持的功能方面，做出更大的折衷）。</p>

<p>使用 ORM 的好處是，程序員可以繼續用 JavaScript 對象而不是數據庫語義來思考 — 如果您需要使用不同數據庫（在相同或不同的網站上），那麼尤其如此。他們還提供了一個明顯的地方來執行數據驗證和檢查。</p>

<div class="note">
<p><strong>備註：</strong> 使用ODM / ORM通常可以降低開發和維護成本！除非您非常熟悉本地查詢語言，或者性能對您至關重要，否則您應該強烈考慮使用 ODM。</p>
</div>

<h3 id="我應該使用哪個_ORMODM">我應該使用哪個 ORM/ODM ?</h3>

<p>NPM 套件管理器站點上，有許多ODM / ORM 解決方案（查看 <a href="https://www.npmjs.com/browse/keyword/odm">odm</a> 和 <a href="https://www.npmjs.com/browse/keyword/orm">orm</a> 標籤的子集合！）。</p>

<p>在撰寫本文時，受歡迎的幾種解決方案是：   </p>

<ul>
 <li><a href="https://www.npmjs.com/package/mongoose">Mongoose</a>: Mongoose 是一個 <a href="https://www.mongodb.org/">MongoDB</a>對象建模工具，用於在異步環境中工作。</li>
 <li><a href="https://www.npmjs.com/package/waterline">Waterline</a>: 它是從基於Express的 Sails web 框架中提取的 ORM。它提供了一個統一的 API，來訪問眾多不同的數據庫，包括Redis，mySQL，LDAP，MongoDB 和 Postgres。</li>
 <li><a href="https://www.npmjs.com/package/bookshelf">Bookshelf</a>: 提供基於 promise 和傳統回調的接口，提供事務支持，eager/嵌套 eager 關係加載，多態關聯以及對一對一，一對多和多對多關係的支持。適用於PostgreSQL，MySQL 和 SQLite3。</li>
 <li><a href="https://www.npmjs.com/package/objection">Objection</a>: 以盡可能簡單的方式，使用 SQL 的全部功能，和底層數據庫引擎（支持SQLite3，Postgres 和 MySQL）。</li>
 <li>
  <p><a href="https://www.npmjs.com/package/sequelize">Sequelize</a> 是 Node.js 和 io.js 基於 promise 的 ORM。它支持以下數據庫方言，PostgreSQL，MySQL，MariaDB，SQLite 和 MSSQL，並具有可靠的事務支持，關係，唯讀複本等功能。</p>
 </li>
</ul>

<p>一般來說，在選擇解決方案時，您應該考慮提供的功能和 “社區活動” （下載，貢獻，錯誤報告，文檔質量等）。在撰寫本文時，Mongoose 是迄今為止最受歡迎的 ODM，如果您將MongoDB 用於你的數據庫，那麼它是一個合理的選擇。</p>

<h3 id="在本地圖書館使用_Mongoose_和_MongoDb">在本地圖書館使用 Mongoose 和 MongoDb</h3>

<p>對於本地圖書館示例（以及本主題的其餘部分），我們將使用 <a href="https://www.npmjs.com/package/mongoose">Mongoose ODM </a>來訪問我們的圖書館數據。 Mongoose 是 <a href="https://www.mongodb.com/what-is-mongodb">MongoDB</a> 的前端，MongoDB 是一個使用面向文檔數據模型的開源 <a href="https://en.wikipedia.org/wiki/NoSQL">NoSQL</a> 數據庫。在 MongoDB 數據庫中，“文檔” 的 “集合” ，<a href="https://docs.mongodb.com/manual/core/databases-and-collections/#collections">類似於</a>關係數據庫中 “行” 的 “表”。</p>

<p>這種 ODM 和數據庫的結合在 Node 社區中非常流行，部分原因是文檔存儲和查詢系統，看起來非常像 JSON，因此對 JavaScript 開發人員來說很熟悉。</p>

<div class="note">
<p><strong>備註：</strong> 使用 Mongoose 時，您不需要事先了解 MongoDB，但是如果您已經熟悉 MongoDB，<a href="http://mongoosejs.com/docs/guide.html">Mongoose documentation </a>文檔的一部分會更易於使用和理解。</p>
</div>

<p>本教程的其餘部分，將介紹如何為 本地圖書館網站示例，定義和訪問Mongoose 模式和模型。</p>

<h2 id="設計本地圖書館的模型">設計本地圖書館的模型</h2>

<p>在您開始編寫模型之前，花幾分鐘的時間思考，我們需要儲存的數據以及不同對象之間的關係。</p>

<p>我們知道，我們需要儲存有關書籍的訊息（標題，摘要，作者，種類，國際標準書號），以及我們可能有多個副本可用（具有全域唯一ID，可用狀態等）。我們可能需要存儲有關作者的更多訊息，而不僅僅是他們的名字，並且可能有多個作者，具有相同或相似的名稱。我們希望能夠根據書名，作者，種類和類別對訊息進行分類。</p>

<p>在設計模型時，對於每個“對象”（相關訊息組）都有獨立的模型，是有意義的。在這種情況下，明顯的對像是書籍，書籍實例和作者。</p>

<p>您可能還希望，使用模型來表示選擇列表選項（例如，選擇的下拉列表），而不是將選項硬編碼到網站本身— 在無法預先知道所有選項，或者可能更改時，更建議使用模型來表示。很明顯的，書本類型是這種模型的可能人選（例如科幻小說，法國詩歌等）。</p>

<p>一旦我們決定了我們的模型和字段，我們就需要考慮它們之間的關係。</p>

<p>考慮到這一點，下面的UML關聯圖，顯示了我們在這種情況下定義的模型（一個框對應一個模型）。如上所述，我們創建了以下模型，圖書（本書的通用細節），書本實例（系統中可用圖書的特定實際副本的狀態）和作者。我們還決定建立一個種類模型，以便可以動態創建它的值，而不是將下拉選項硬編碼。我們已經決定不為書本實例：狀態<code>BookInstance:status</code>建立模型—我們將硬編碼可接受的值，因為我們不希望這些值發生變化。在下圖每個框中，您可以看到模型名稱，字段名稱和類型，以及方法及其返回類型。</p>

<p>下圖還顯示了模型之間的關係，包括它們的多重性。多重性是圖中顯示可能存在於關係中的每個模型的數量（最大值和最小值）的數字。例如，框之間的連接線，顯示書本<code>Book</code>和種類<code>Genre</code>是相關的。靠近書本<code>Book</code>模型的數字，表明一本書必須有零個或多個種類（您想要多少都可以），而種類<code>Genre</code>旁邊一行的數字，表明它可以有零個或多個相關書籍。</p>

<div class="note">
<p><strong>備註：</strong> 正如我們在下面的<strong>Mongoose入門</strong>中所討論的那樣，通常只需要在一個模型中定義文檔/模型之間關係的字段（通過在另一個模型中搜索相關的<code>_id</code>仍然可以找到反向關係）。下面我們選擇在書本綱要（Book schema）中定義Book/Genre和Book/Author之間的關係，以及書本實例綱要（BookInstance Schema）中Book/BookInstance之間的關係。這種選擇有點武斷—我們同樣可以在其他綱要中擁有該字段。</p>
</div>

<p><img alt="Mongoose Library Model  with correct cardinality" src="library_website_-_mongoose_express.png"></p>

<div class="note">
<p><strong>備註：</strong> 下一節提供了一個基本的入門知識，解釋如何定義和使用模型。在您閱讀它時，請想想我們將如何構建上圖中的每個模型。</p>
</div>

<h2 id="Mongoose入門">Mongoose入門</h2>

<p>本節概述如何將Mongoose 連接到MongoDB 數據庫，如何定義模型綱要和模型，以及如何進行基本查詢。</p>

<div class="note">
<p><strong>備註：</strong> 本入門受到npm上的<a href="https://www.npmjs.com/package/mongoose">Mongoose快速入門</a>和<a href="http://mongoosejs.com/docs/guide.html">Mongoose官方文檔</a>的“深度影響”。</p>
</div>

<h3 id="安裝Mongoose和MongoDB">安裝Mongoose和MongoDB</h3>

<p>Mongoose像任何其他依賴項一樣，安裝在您的項目（<strong>package.json</strong>）中—使用NPM。要安裝它，請在項目文件夾中，使用以下命令：</p>

<pre class="brush: bash notranslate"><code>npm install mongoose</code>
</pre>

<p>安裝Mongoose會添加所有依賴項，包括MongoDB數據庫驅動程序，但它不會安裝MongoDB 。如果你想安裝一個MongoDB服務器，那麼你可以<a href="https://www.mongodb.com/download-center">從這裡下載</a>各種操作系統的安裝程序，並在本地安裝。您還可以使用基於雲端的MongoDB實例。</p>

<div class="note">
<p><strong>備註：</strong> 對於本教程，我們將使用基於mLab雲的數據庫，作為服務<a href="https://mlab.com/plans/pricing/">沙箱層</a>來提供數據庫。這適用於開發，也對於本教程很有意義，因為它使“安裝”與操作系統無關（數據庫即服務，也是您可能會用於生產環境數據庫的一種方法）。</p>
</div>

<h3 id="連接到MongoDB">連接到MongoDB</h3>

<p>Mongoose需要連接到MongoDB數據庫。您可以<code>require()</code>並使用<code>mongoose.connect()</code>，以連接到本地託管的數據庫，如下所示。</p>

<pre class="brush: js notranslate">//Import the mongoose module
var mongoose = require('mongoose');

//Set up default mongoose connection
var mongoDB = 'mongodb://127.0.0.1/my_database';
mongoose.connect(mongoDB);
// Get Mongoose to use the global promise library
mongoose.Promise = global.Promise;
//Get the default connection
var db = mongoose.connection;

//Bind connection to error event (to get notification of connection errors)
db.on('error', console.error.bind(console, 'MongoDB connection error:'));</pre>

<p>您可以使用<code>mongoose.connection</code>獲取默認的<code>Connection</code>對象。一旦連接，在<code>Connection</code>實例上，將觸發打開事件。</p>

<div class="note">
<p><strong>備註：</strong> 如果需要創建其他連接，可以使用<code>mongoose.createConnection()</code>。這與<code>connect()</code>採用相同形式的數據庫URI（包含主機，數據庫，端口，選項等），並返回<code>Connection</code>對象。</p>
</div>

<h3 id="定義並創建模型">定義並創建模型</h3>

<p>模型使用<code>Schema</code>接口進行定義。Schema允許您定義存儲在每個文檔中的字段，及其驗證要求和默認值。此外，您可以定義靜態和實例助手方法，以更輕鬆地處理數據類型，以及可以像其他任何字段一樣使用的虛擬屬性，但實際上並不存儲在數據庫中（我們稍後將討論）。</p>

<p>然後，綱要Schemas被<code>mongoose.model()</code>方法“編譯”為模型。擁有模型後，您可以使用它來查找，創建，更新和刪除給定類型的對象。</p>

<div class="note">
<p><strong>備註：</strong> 每個模型都映射到MongoDB數據庫中的文檔集合。這些文檔將包含模型綱要<code>Schema</code>中定義的字段/綱要型態。</p>
</div>

<h4 id="定義綱要Schemas">定義綱要Schemas</h4>

<p>下面的代碼片段，顯示了您可以如何定義一個簡單的綱要。首先<code>require()</code>mongoose，然後使用Schema構造函數，創建一個新的Schema實例，在構造函數的對象參數中，定義其中的各個字段。</p>

<pre class="brush: js notranslate">//Require Mongoose
var mongoose = require('mongoose');

//Define a schema
var Schema = mongoose.Schema;

var SomeModelSchema = new Schema({
    a_string: String,
    a_date: Date
});
</pre>

<p>在上面的例子中，我們只有兩個字段，一個字符串和一個日期。在接下來的部分中，我們將展示一些其他的字段類型，驗證和其他方法。</p>

<h4 id="創建模型">創建模型</h4>

<p>使用<code>mongoose.model()</code>方法從綱要創建模型：</p>

<pre class="brush: js notranslate">// Define schema
var Schema = mongoose.Schema;

var SomeModelSchema = new Schema({
    a_string: String,
    a_date: Date
});

<strong>// Compile model from schema
var SomeModel = mongoose.model('SomeModel', SomeModelSchema );</strong></pre>

<p>第一個參數，是將為模型創建的集合的單數名稱（Mongoose將為上面的SomeModel模型，創建數據庫集合），第二個參數，是您要在創建模型時使用的綱要Shema。</p>

<div class="note">
<p><strong>備註：</strong> 定義模型類後，可以使用它們來創建，更新或刪除記錄，並運行查詢，以獲取記錄的所有記錄，或特定子集。我們將在以下“使用模型”部分，向您展示如何執行上述操作，以及當創建視圖時，如何執行此操作。</p>
</div>

<h4 id="綱要型態（字段）">綱要型態（字段）</h4>

<p>綱要schema可以有任意數量的字段 — 每個字段代表存儲在MongoDB 文檔中的字段。如下的示例綱要，顯示許多常見字段類型及其聲明方式。</p>

<pre class="brush: js notranslate">var schema = new Schema(
{
  name: <strong>String</strong>,
  binary: <strong>Buffer</strong>,
  living: <strong>Boolean</strong>,
  updated: { type: <strong>Date</strong>, default: Date.now },
  age: { type: <strong>Number</strong>, min: 18, max: 65, required: true },
  mixed: <strong>Schema.Types.Mixed</strong>,
  _someId: <strong>Schema.Types.ObjectId</strong>,
  array: <strong>[]</strong>,
  ofString: [<strong>String</strong>], // You can also have an array of each of the other types too.
  nested: { stuff: { type: <strong>String</strong>, lowercase: true, trim: true } }
})</pre>

<p>大多數綱要型態<a href="http://mongoosejs.com/docs/schematypes.html">SchemaTypes</a>（“type：”之後或字段名稱之後的描述符）都是自解釋的。例外情況是：</p>

<ul>
 <li><code>ObjectId</code>:表示數據庫中模型的特定實例。例如，一本書可能會使用它來表示其作者對象。這實際上將包含指定對象的唯一ID ( <code>_id</code>) 。我們可以使用<code>populate()</code>方法，在需要時提取相關訊息。</li>
 <li><a href="http://mongoosejs.com/docs/schematypes.html#mixed">Mixed</a> :任意綱要型態。</li>
 <li>[] :一個數組的項目。您可以在這些模型上執行JavaScript數組操作（push，pop，unshift等）。上面的例子，顯示了一個沒有指定類型的對像數組，和一個String對像數組，但是你可以有任何類型的對像數組。</li>
</ul>

<p>該代碼還顯示了聲明一個字段的兩種方式：</p>

<ul>
 <li>字段名稱和類型作為鍵值對（即是，像上面的<code>name</code>, <code>binary</code> and <code>living</code>）。</li>
 <li>字段名稱後跟一個定義類型<code>type</code>的對象，以及該字段的任何其他選項。選項包括如下內容：
  <ul>
   <li>默認值。</li>
   <li>內置驗證器（例如最大/最小值）和自定義驗證功能。</li>
   <li>該字段是否為必要</li>
   <li>是否應將字符串<code>String</code>字段自動設置為小寫，大寫或修剪（例如)<code>{ type:<strong>String</strong>, lowercase: true, trim: true }</code></li>
  </ul>
 </li>
</ul>

<p>有關選項的更多訊息，請參閱<a href="http://mongoosejs.com/docs/schematypes.html">SchemaTypes</a>（Mongoose docs）。</p>

<h4 id="驗證">驗證</h4>

<p>Mongoose 提供內置和自定義驗證器，以及同步和異步驗證器。它允許您在所有情況下，指定可接受的範圍或值，以及驗證失敗的錯誤消息。</p>

<p>內置的驗證器包括：</p>

<ul>
 <li>所有<a href="http://mongoosejs.com/docs/schematypes.html">SchemaTypes</a>都具有內置的<a href="http://mongoosejs.com/docs/api.html#schematype_SchemaType-required">必需</a>驗證器。這用於指定，是否必須提供該字段才能保存文檔。</li>
 <li><a href="http://mongoosejs.com/docs/api.html#schema-number-js">Numbers </a><a href="http://mongoosejs.com/docs/api.html#schema-number-js">數字</a>有最小<a href="http://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-min">min</a>和最大<a href="http://mongoosejs.com/docs/api.html#schema_number_SchemaNumber-max">max</a>驗證器。</li>
 <li><a href="http://mongoosejs.com/docs/api.html#schema-string-js">Strings</a>字符串有：
  <ul>
   <li><a href="http://mongoosejs.com/docs/api.html#schema_string_SchemaString-enum">enum</a>枚舉：指定該字段的允許值集合。</li>
   <li><a href="http://mongoosejs.com/docs/api.html#schema_string_SchemaString-match">match</a> :指定字符串必須匹配的正則表達式。</li>
   <li>字符串的最大長度<a href="http://mongoosejs.com/docs/api.html#schema_string_SchemaString-maxlength">maxlength</a>和最小長度<a href="http://mongoosejs.com/docs/api.html#schema_string_SchemaString-minlength">minlength</a></li>
  </ul>
 </li>
</ul>

<p>下面的示例（從Mongoose文檔稍微修改）顯示瞭如何指定一些驗證器類型和錯誤消息：</p>

<pre class="brush: js notranslate"><code>
    var breakfastSchema = new Schema({
      eggs: {
        type: Number,
        min: [6, 'Too few eggs'],
        max: 12
        required: [true, 'Why no eggs?']
      },
      drink: {
        type: String,
        enum: ['Coffee', 'Tea', 'Water',]
      }
    });
</code></pre>

<p>有關字段驗證的完整訊息，請參閱<a href="http://mongoosejs.com/docs/validation.html">驗證</a>（Mongoose docs）。</p>

<h4 id="虛擬屬性">虛擬屬性</h4>

<p>虛擬屬性是您可以獲取和設置的文檔屬性，但不會持久保存到MongoDB。getter 對格式化或組合字段非常有用，而setter 可用於將單個值分解為多個值，以進行存儲。</p>

<p>文檔中的示例，從名字和姓氏字段構造（並解構）一個全名虛擬屬性，這比每次在模板中使用全名更簡單，更清晰。</p>

<div class="note">
<p><strong>備註：</strong> 我們將使用庫中的虛擬屬性，來為每個使用路徑和記錄的<code>_id</code>值的模型記錄，定義唯一的URL。</p>
</div>

<p>欲了解更多訊息，請參閱<a href="http://mongoosejs.com/docs/guide.html#virtuals">虛擬</a>（Mongoose文檔）。</p>

<h4 id="方法和查詢幫助">方法和查詢幫助</h4>

<p>綱要schema也可以有<a href="http://mongoosejs.com/docs/guide.html#methods">實例方法</a>，<a href="http://mongoosejs.com/docs/guide.html#statics">靜態方法</a>和<a href="http://mongoosejs.com/docs/guide.html#query-helpers">查詢助手</a>。實例和靜態方法很相似，但有明顯的區別，即實例方法與特定記錄相關聯，並且可以訪問當前對象。查詢助手允許您擴展mongoose的<a href="http://mongoosejs.com/docs/queries.html">鍊式查詢構建器API</a>（例如，除了<code>find()</code>, <code>findOne()</code>和<code>findById()</code>方法外，還允許您添加一個“byName”查詢。</p>

<h3 id="使用模型">使用模型</h3>

<p>一旦創建了綱要，就可以使用它來創建模型。該模型代表數據庫中可以搜索的文檔集合，而模型的實例代表您可以保存和檢索的單個文檔。</p>

<p>我們在下面簡要介紹一下。有關更多訊息，請參閱：<a href="http://mongoosejs.com/docs/models.html">模型</a>（Mongoose docs）。</p>

<h4 id="創建和修改文檔">創建和修改文檔</h4>

<p>要創建記錄，您可以定義模型的實例，然後調用<code>save()</code>。下面的例子假設，SomeModel是我們從綱要創建的模型（帶有單一字段“name” ）。</p>

<pre class="brush: js notranslate"><code>// Create an instance of model SomeModel
var awesome_instance = new </code>SomeModel<code>({ name: 'awesome' });

// Save the new model instance, passing a callback
awesome_instance.save(function (err) {
  if (err) return handleError(err);
  // saved!
});
</code></pre>

<p>創建記錄（以及更新，刪除和查詢）是異步操作— 您提供在操作完成時調用的回調。API使用錯誤優先參數約定，因此回調的第一個參數將始終為錯誤值（或null）。如果API返回一些結果，則將作為第二個參數提供。</p>

<p>您還可以使用<code>create()</code>，同時定義模型實例，並保存模型實例。回調將為第一個參數返回錯誤，為第二個參數返回新創建的模型實例。</p>

<pre class="brush: js notranslate">SomeModel<code>.create({ name: 'also_awesome' }, function (err, awesome_instance) {
  if (err) return handleError(err);
  // saved!
});</code></pre>

<p>每個模型都有一個關聯的連接（當您使用<code>mongoose.model()</code>時，這將成為默認連接）。您創建一個新連接並調用<code>.model()</code>，以在另一個數據庫上創建文檔。</p>

<p>您可以使用點語法訪問此新記錄中的字段，並更改值。您必須調用<code>save()</code>或<code>update()</code>，將修改的值存回數據庫。</p>

<pre class="brush: js notranslate">// Access model field values using dot notation
console.log(<code>awesome_instance.name</code>); //should log '<code>also_awesome</code>'

// Change record by modifying the fields, then calling save().
<code>awesome_instance</code>.name="New cool name";
<code>awesome_instance.save(function (err) {
   if (err) return handleError(err); // saved!
   });</code>
</pre>

<h4 id="尋找紀錄">尋找紀錄</h4>

<p>可以使用查詢方法搜索記錄，將查詢條件指定為JSON 文檔。下面的代碼片段，顯示瞭如何在數據庫中，找到所有參加網球運動的運動員，只返回運動員姓名和年齡的字段。這裡我們只指定一個匹配的字段（運動 sport），但您可以添加更多條件，指定正則表達式標準，或完全刪除條件以返回所有運動員。</p>

<pre class="brush: js notranslate"><code>var Athlete = mongoose.model('Athlete', yourSchema);

// find all athletes who play tennis, selecting the 'name' and 'age' fields
Athlete.find({ 'sport': 'Tennis' }, 'name age', function (err, athletes) {
  if (err) return handleError(err);
  // 'athletes' contains the list of athletes that match the criteria.
})</code></pre>

<p>如果您指定回調，如上所示，查詢將立即執行。搜索完成後將調用回調。</p>

<div class="note">
<p><strong>備註：</strong> Mongoose中的所有回調，都使用此回調模式<code>callback(error, result)</code>。如果執行查詢時發生錯誤，錯誤參數<code>error</code>將包含錯誤文檔，並且結果<code>result</code>將為null。如果查詢成功，則<code>error</code>參數將為null，並且結果<code>result </code>將被填充到查詢結果。</p>
</div>

<p>如果您未指定回調，則API將返回<a href="http://mongoosejs.com/docs/api.html#query-js">Query</a>類型的變量。您可以使用此查詢對象來構建查詢，然後稍後使用<code>exec()</code>方法執行（使用回調）。</p>

<pre class="brush: js notranslate"><code>// find all athletes that play tennis
var query = Athlete.find({ 'sport': 'Tennis' });

// selecting the 'name' and 'age' fields
query.select('name age');

// limit our results to 5 items
query.limit(5);

// sort by age
query.sort({ age: -1 });

// execute the query at a later time
query.exec(function (err, athletes) {
  if (err) return handleError(err);
  // athletes contains an ordered list of 5 athletes who play Tennis
})</code></pre>

<p>上面我們在<code>find()</code>方法中，定義了查詢條件。我們也可以使用<code>where()</code>函數來執行此操作，並且我們可以使用點運算符（ . ）將查詢的所有部分鏈接在一起，而不是分別添加它們。</p>

<p>下面的代碼片段，與我們上面的查詢相同，並有年齡的附加條件。</p>

<pre class="notranslate"><code>Athlete.
  find().
  where('sport').equals('Tennis').
  where('age').gt(17).lt(50).  //Additional where query
  limit(5).
  sort({ age: -1 }).
  select('name age').
  exec(callback); // where callback is the name of our callback function.</code></pre>

<p><a href="http://mongoosejs.com/docs/api.html#query_Query-find">find()</a>  方法獲取所有匹配的記錄，但通常你只想獲得一個匹配。以下方法可以查詢單個記錄：</p>

<ul>
 <li><code><a href="http://mongoosejs.com/docs/api.html#model_Model.findById">findById()</a></code>:用指定的<code>id </code>查找文檔（每個文檔都有一個唯一的<code>id</code>）。</li>
 <li><code><a href="http://mongoosejs.com/docs/api.html#query_Query-findOne">findOne()</a></code>: 查找與指定條件匹配的單個文檔。</li>
 <li><code><a href="http://mongoosejs.com/docs/api.html#model_Model.findByIdAndRemove">findByIdAndRemove()</a></code>, <code><a href="http://mongoosejs.com/docs/api.html#model_Model.findByIdAndUpdate">findByIdAndUpdate()</a></code>, <code><a href="http://mongoosejs.com/docs/api.html#query_Query-findOneAndRemove">findOneAndRemove()</a></code>, <code><a href="http://mongoosejs.com/docs/api.html#query_Query-findOneAndUpdate">findOneAndUpdate()</a></code>:通過<code>id </code>或條件查找單個文檔，並更新或刪除它。這些是用於更新和刪除記錄的有用便利功能。</li>
</ul>

<div class="note">
<p><strong>備註：</strong> 還有一個<code><a href="http://mongoosejs.com/docs/api.html#model_Model.count">count()</a></code>方法，您可以使用它來獲取與條件匹配的項目數。如果您想要在不實際提取記錄的情況下執行計數，這非常有用。</p>
</div>

<p>查詢可以做更多的事情。有關更多訊息，請參閱：<a href="http://mongoosejs.com/docs/queries.html">查詢</a>（Mongoose文檔）。</p>

<h4 id="運用相關文檔—_population方法">運用相關文檔— population方法</h4>

<p>您可以使用<code>ObjectId</code>綱要字段，從一個文檔/模型實例，創建一對一引用，或者使用<code>ObjectIds</code>數組，從一個文檔創建一對多的引用。該字段存儲相關模型的ID。如果需要關聯文檔的實際內容，可以在查詢中使用<code><a href="http://mongoosejs.com/docs/api.html#query_Query-populate">populate()</a></code>方法，將id替換為實際數據。</p>

<p>例如，以下綱要定義作者和故事。每個作者可以有多個故事，我們將其表示為一個<code>ObjectId</code>數組。每個故事可以有一個作者。綱要從“ref”（以粗體突出顯示）得知，可以分配給該字段的模型。</p>

<pre class="brush: js notranslate"><code>var mongoose = require('mongoose')
  , Schema = mongoose.Schema

var authorSchema = Schema({
  name    : String,
  stories : [{ type: Schema.Types.ObjectId, <strong>ref</strong>: 'Story' }]
});

var storySchema = Schema({
  author : { type: Schema.Types.ObjectId, <strong>ref</strong>: 'Author' },
  title    : String
});

var Story  = mongoose.model('Story', storySchema);
var Author = mongoose.model('Author', authorSchema);</code></pre>

<p>我們可以通過分配<code>_id</code>值，來保存對相關文檔的引用。下面我們創建一個作者，然後創建一個故事，並將作者ID分配給我們的故事作者字段。</p>

<pre class="brush: js notranslate"><code>var bob = new Author({ name: 'Bob Smith' });

bob.save(function (err) {
  if (err) return handleError(err);

  //Bob now exists, so lets create a story
  var story = new Story({
    title: "Bob goes sledding",
    author: bob._id    // assign the _id from the our author Bob. This ID is created by default!
  });

  story.save(function (err) {
    if (err) return handleError(err);
    // Bob now has his story
  });
});</code></pre>

<p>我們的故事文檔，現在有作者文檔ID引用的作者。為了在我們的故事結果中，獲取作者訊息，我們使用<code>populate()</code>，如下所示。</p>

<pre class="brush: js notranslate"><code>Story
.findOne({ title: 'Bob goes sledding' })
.populate('author') //This populates the author id with actual author information!
.exec(function (err, story) {
  if (err) return handleError(err);
  console.log('The author is %s', story.author.name);
  // prints "The author is Bob Smith"
});</code></pre>

<div class="note">
<p><strong>備註：</strong> 敏銳的讀者會注意到，我們在故事中添加了作者，但我們沒有做任何事情，來將我們的故事添加到作者的故事<code>stories</code>數組中。那麼我們怎樣才能得到特定作者的所有故事？</p>

<p>一種方法，是將作者添加到故事數組中，但這會導致我們需要在兩個地方，維護與作者和故事有關的訊息。更好的方法是獲取作者的<code>_id</code>，然後使用<code>find()</code>，在所有故事的作者字段中搜索此內容。</p>

<pre class="brush: js notranslate"><code>Story
.find({ author : bob._id })
.exec(function (err, stories) {
  if (err) return handleError(err);
  // returns all stories that have Bob's id as their author.
});</code>
</pre>
</div>

<p>這幾乎是您在本教程中，使用相關項目時，需要了解的所有內容。有關更多詳細訊息，請參閱<a href="http://mongoosejs.com/docs/populate.html">Population</a>（Mongoose docs）。</p>

<h3 id="一個檔案對應一個綱要模型">一個檔案對應一個綱要/模型</h3>

<p>雖然您可以使用任何喜歡的文件結構創建綱要和模型，但我們強烈建議在每個模型模塊（文件）中，定義每個模型綱要，導出方法以創建模型。如下所示：</p>

<pre class="brush: js notranslate"><code>// File: ./models/somemodel.js

//Require Mongoose
var mongoose = require('mongoose');

//Define a schema
var Schema = mongoose.Schema;

var SomeModelSchema = new Schema({
    a_string          : String,
    a_date            : Date,
});

<strong>//Export function to create "SomeModel" model class
module.exports = mongoose.model('SomeModel', SomeModelSchema );</strong></code></pre>

<p>然後，您可以在其他文件中，立即要求並使用該模型。下面我們展示如何使用它，來獲取模型的所有實例。</p>

<pre class="brush: js notranslate"><code>//Create a SomeModel model just by requiring the module
var SomeModel = require('../models/somemodel')

// Use the SomeModel object (model) to find all SomeModel records
SomeModel.find(callback_function);</code></pre>

<h2 id="架設MongoDB數據庫">架設MongoDB數據庫</h2>

<p>現在我們了解了Mongoose能做什麼，以及我們想如何設計我們的模型，現在該開始在LocalLibrary網站上工作了。我們想要做的第一件事，就是設置一個MongoDb數據庫，我們可以使用它來儲存我們的圖書館數據。</p>

<p>本教程，我們將使用<a href="https://mlab.com/welcome/">mLab</a>免費的雲託管的“ <a href="https://mlab.com/plans/pricing/">沙盒</a> ”數據庫。這個數據庫層不適合生產環境的網站，因為它沒有冗餘設計，但它對於開發和原型設計來說非常有用。我們在這裡使用它，是因為它免費且易於設置，並且因為作為數據庫服務供應商來說，mLab是流行的數據庫選擇之一，您可能會合理選擇您的生產環境數據庫（撰寫本文時，其他流行的選擇包括<a href="https://www.compose.com/">Compose</a>、<a href="https://scalegrid.io/pricing.html">ScaleGrid</a>和<a href="https://www.mongodb.com/cloud/atlas">MongoDB Atlas</a>）。</p>

<div class="note">
<p><strong>備註：</strong> 如果您願意，可以下載並安裝<a href="https://www.mongodb.com/download-center">與系統相對應的二進製文件</a>，在本地設置MongoDb數據庫。除了您在連接時指定的數據庫URL之外，本文中的其餘指令將很類似。</p>
</div>

<p>您首先需要<a href="https://mlab.com/signup/">使用mLab創建一個賬戶</a>（這是免費的，只需要輸入基本聯繫訊息，並確認其服務條款）。</p>

<p>登錄後，您將進入<a href="https://mlab.com/home">mLab主</a>畫面:</p>

<ol>
 <li>單擊<em>MongoDB Deployments</em>部分中的<strong>Create New。</strong><img alt="" src="https://mdn.mozillademos.org/files/14446/mLabCreateNewDeployment.png"></li>
 <li>這將打開“雲提供商”Cloud Provider 選擇畫面。<br>
  <img alt="MLab - screen for new deployment" src="https://mdn.mozillademos.org/files/15661/mLab_new_deployment_form_v2.png"><br>

  <ul>
   <li>從“計劃類型”Plan Type 部分中，選擇“SANDBOX（免費）”計劃。</li>
   <li>從“雲提供商” <em>Cloud Provider</em>部分，選擇任意提供商。不同的提供商，提供不同的地區（顯示在選定的計劃類型下面）。</li>
   <li>單擊“繼續” <strong>Continue</strong>按鈕。</li>
  </ul>
 </li>
 <li>這將打開“選擇區域” <em>Select Region </em>畫面。
  <p><img alt="Select new region screen" src="https://mdn.mozillademos.org/files/15662/mLab_new_deployment_select_region_v2.png"></p>

  <ul>
   <li>
    <p>選擇離您最近的地區，然後選擇繼續<strong>Continue</strong> .</p>
   </li>
  </ul>
 </li>
 <li>
  <p>這將打開 Final Details 畫面。<br>
   <img alt="New deployment database name" src="https://mdn.mozillademos.org/files/15663/mLab_new_deployment_final_details.png"></p>

  <ul>
   <li>
    <p>輸入新數據庫的名稱<code>local_library</code>，然後選擇繼續<strong>Continue</strong>。</p>
   </li>
  </ul>
 </li>
 <li>
  <p>這將打開訂單確認畫面。<br>
   <img alt="Order confirmation screen" src="https://mdn.mozillademos.org/files/15664/mLab_new_deployment_order_confirmation.png"></p>

  <ul>
   <li>
    <p>單擊“提交訂單” <strong>Submit Order</strong>以創建數據庫。</p>
   </li>
  </ul>
 </li>
 <li>
  <p>您將返回到主畫面。單擊剛剛創建的新數據庫，以打開其詳細訊息畫面。正如你所看到的，數據庫沒有集合（數據）。<br>
   <img alt="mLab - Database details screen" src="https://mdn.mozillademos.org/files/15665/mLab_new_deployment_database_details.png"><br>
    您需要用來訪問數據庫的URL，顯示在上面的表單中（如上圖所示）。為了使用它，您需要創建一個可以在URL中指定的數據庫用戶。</p>
 </li>
 <li>單擊用戶<strong>Users</strong>選項卡，並選擇添加數據庫用戶按鈕<strong>Add database user</strong>。</li>
 <li>輸入用戶名和密碼（兩次），然後按創建<strong>Create</strong>。不要選擇只讀<em>read-only</em>。<br>
  <img alt="" src="https://mdn.mozillademos.org/files/14454/mLab_database_users.png"></li>
</ol>

<p>您現在已經創建了數據庫，並且有一個可以用來訪問它的URL（帶有用戶名和密碼）。這看起來像是這樣的：<code>mongodb://your_user_namer:your_password@ds119748.mlab.com:19748/local_library</code>.</p>

<h2 id="安裝_Mongoose">安裝 Mongoose</h2>

<p>打開命令提示符，並到您創建<a href="/zh-CN/docs/Learn/Server-side/Express_Nodejs/skeleton_website">本地圖書館骨架網站</a>的目錄。輸入以下命令，安裝Mongoose（及其依賴項），並將其添加到您的<strong>package.json</strong>文件中，除非您在閱讀上述<strong>Mongoose入門</strong>時，已經這樣做了。</p>

<pre class="brush: bash notranslate">npm install mongoose
</pre>

<h2 id="連接到_MongoDB">連接到 MongoDB</h2>

<p>打開<strong>/app.js</strong>（位於項目的根目錄），並在宣告Express應用程序對象的位置（在<code>var app = express();</code>之後）複製以下文本。將數據庫url字符串（'insert_your_database_url_here'）替換為表示您自己的數據庫的位置URL（即是使用<strong>來自上面mLab</strong>的訊息）。</p>

<pre class="brush: js notranslate">//Set up mongoose connection
var mongoose = require('mongoose');
var mongoDB = '<em>insert_your_database_url_here</em>';
mongoose.connect(mongoDB);
mongoose.Promise = global.Promise;
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));</pre>

<p>正如上面的<strong>Mongoose入門</strong>中所討論的，此代碼創建了與數據庫的默認連接，並綁定到錯誤事件（以便將錯誤打印到控制台）。</p>

<h2 id="定義本地圖書館綱要">定義本地圖書館綱要</h2>

<p>如上所述，我們將為每個模型定義一個單獨的模塊。首先在項目根目錄（<strong>/models</strong>）中，為我們的模型創建一個文件夾，然後為每個模型創建單獨的文件：</p>

<pre class="notranslate">/express-locallibrary-tutorial  //the project root
  <strong>/models</strong>
    <strong>author.js</strong>
    <strong>book.js</strong>
    <strong>bookinstance.js</strong>
    <strong>genre.js</strong>
</pre>

<h3 id="作者模型">作者模型</h3>

<p>複製下面顯示的<code>Author</code>作者綱要代碼，並將其粘貼到<strong>./models/author.js</strong>文件中。該綱要定義了一個作者，具有<code>String</code>SchemaTypes的第一個名稱和家族名稱，這是必需的，最多有100個字符，<code>Date</code>字段為出生和死亡日期。</p>

<pre class="brush: js notranslate">var mongoose = require('mongoose');

var Schema = mongoose.Schema;

var AuthorSchema = new Schema(
  {
    first_name: {type: String, required: true, max: 100},
    family_name: {type: String, required: true, max: 100},
    date_of_birth: {type: Date},
    date_of_death: {type: Date},
  }
);

<strong>// Virtual for author's full name
AuthorSchema
.virtual('name')
.get(function () {
  return this.family_name + ', ' + this.first_name;
});</strong>

// Virtual for author's URL
AuthorSchema
.virtual('url')
.get(function () {
  return '/catalog/author/' + this._id;
});

//Export model
module.exports = mongoose.model('Author', AuthorSchema);

</pre>

<p>我們還為AuthorSchema，聲明了一個名為“url”的虛擬屬性，它返回獲取模型的特定實例所需的絕對URL — 每當我們需要獲取指向特定作者的鏈接時，我們將在模板中使用該屬性。</p>

<div class="note">
<p><strong>備註：</strong> 在綱要中聲明我們的URL是虛擬的，這是一個好主意，因為一個項目的URL只需要在一個地方更改。此時，使用此URL的鏈接將不起作用，因為我們還沒有任何路由，可以處理個別模型實例的代碼。我們將在後面的文章中介紹這些內容！</p>
</div>

<p>在模塊的最後，我們導出了模型。</p>

<h3 id="書本模型">書本模型</h3>

<p>複製下面顯示的<code>Book</code>綱要代碼，並將其粘貼到<strong>./models/book.js</strong>文件中。其中大部分與作者模型相似—我們已經聲明了一個具有多個字符串字段的綱要，以及一個虛擬屬性，用於獲取特定書籍記錄的URL，並且我們已經導出了模型。</p>

<pre class="brush: js notranslate">var mongoose = require('mongoose');

var Schema = mongoose.Schema;

var BookSchema = new Schema(
  {
    title: {type: String, required: true},
  <strong>  author: {type: Schema.Types.ObjectId, ref: 'Author', required: true},</strong>
    summary: {type: String, required: true},
    isbn: {type: String, required: true},
  <strong>  genre: [{type: Schema.Types.ObjectId, ref: 'Genre'}]</strong>
  }
);

// Virtual for book's URL
BookSchema
.virtual('url')
.get(function () {
  return '/catalog/book/' + this._id;
});

//Export model
module.exports = mongoose.model('Book', BookSchema);
</pre>

<p>這裡的主要區別，是我們已經創建了兩個對其他模型的引用：</p>

<ul>
 <li>作者是對單個<code>Author</code>作者模型對象的引用，並且是必要的。</li>
 <li>種類是對<code>Genre</code>種類模型對像數組的引用。我們還沒有宣告這個對象！</li>
</ul>

<h3 id="書本實例模型">書本實例模型</h3>

<p>最後，複製下面顯示的<code>BookInstance</code>綱要代碼，並將其粘貼到<strong>./models/bookinstance.js</strong>文件中。<code>BookInstance</code>表示某人可能藉閱的書籍的特定副本，並包含有關該副本是否可用，或預期返回日期的訊息，“印記”或版本詳細訊息。</p>

<pre class="brush: js notranslate">var mongoose = require('mongoose');

var Schema = mongoose.Schema;

var BookInstanceSchema = new Schema(
  {
    book: { type: Schema.Types.ObjectId, ref: 'Book', required: true }, //reference to the associated book
    imprint: {type: String, required: true},
    status: {type: String, required: true, <strong>enum: ['Available', 'Maintenance', 'Loaned', 'Reserved']</strong>, <strong>default: 'Maintenance'</strong>},
    due_back: {type: Date, <strong>default: Date.now</strong>}
  }
);

// Virtual for bookinstance's URL
BookInstanceSchema
.virtual('url')
.get(function () {
  return '/catalog/bookinstance/' + this._id;
});

//Export model
module.exports = mongoose.model('BookInstance', BookInstanceSchema);</pre>

<p>我們在這裡展示的新東西，是字段選項：</p>

<ul>
 <li>枚舉<code>enum</code>:這允許我們設置字符串的允許值。在這種情況下，我們用它來指定我們書籍的可用性狀態（使用枚舉，意味著我們可以防止錯誤拼寫和任意值，成為我們的狀態）</li>
 <li>默認值<code>default</code>:我們使用默認值，將新創​​ 建的書本實例的默認狀態，設置為維護，並將默認的<code>due_back</code>日期，設置為現在<code>now</code>（請注意在設置日期時，如何調用Date函數！）</li>
</ul>

<p>其他所有內容，大夥應該在前面教程裡邊已經熟悉了。</p>

<h3 id="種類模型-自我挑戰!">種類模型-自我挑戰!</h3>

<p>打開你的<strong>./models/genre.js</strong>文件，並創建一個存儲類型的綱要（書本的類別，例如它是小說還是非小說，浪漫史或軍事歷史等）。</p>

<p>該定義將與其他模型非常相似：</p>

<ul>
 <li>該模型應該有一個名為<code>name</code>的<code>String</code>SchemaType ，來描述種類。</li>
 <li>這個<code>name</code>字段應該是必要的，並且有3到​​100個字符。</li>
 <li>為類型的URL聲明虛擬，名為<code>url</code>。</li>
 <li>導出模型。</li>
</ul>

<h2 id="測試—創建一些項目">測試—創建一些項目</h2>

<p>就是這樣。我們現在已經為該網站建立了所有模型！</p>

<p>為了測試這些模型（並創建一些示例書籍，和其他項目以便於我們在後面文章使用），現在我們將運行一個獨立的腳本來創建每種類型的項目：</p>

<ol>
 <li>在express-locallibrary-tutorial目錄下（與<code>package.json</code>處於同一級別），下載（或以其他方式創建）文件<a href="https://raw.githubusercontent.com/hamishwillee/express-locallibrary-tutorial/master/populatedb.js">populatedb.js</a>。

  <div class="note">
  <p><strong>備註：</strong> 您不需要知道<a href="https://raw.githubusercontent.com/hamishwillee/express-locallibrary-tutorial/master/populatedb.js">populatedb.js</a>的工作原理;它只是將示例數據添加到數據庫中。</p>
  </div>
 </li>
 <li>在項目根目錄中，輸入以下命令，以安裝腳本所需的異步模塊（我們將在後面的教程中討論這一點）
  <pre class="brush: bash notranslate">npm install async</pre>
 </li>
 <li>在命令提示符下，使用node運行此腳本，傳遞MongoDB數據庫的URL（與之前在<code>app.js</code>中替換insert_your_database_url_here佔位符的那個相同）：
  <pre class="brush: bash notranslate">node populatedb &lt;your mongodb url&gt;​​​​</pre>
 </li>
 <li>該腳本應一路運行至完成，並在終端中創建它們時顯示各項目。</li>
</ol>

<div class="note">
<p><strong>備註：</strong> 至<a href="https://mlab.com/home">mLab</a>上的數據庫。您現在應該可以深入到書本籍，作者，種類和書本實例的各個集合中，並查看單個文檔。</p>
</div>

<h2 id="總結">總結</h2>

<p>本文中我們學到了一點數據庫和Node/Express的ORMs，更多的是關於如何定義Mongoose綱要與模型。然後我們使用這些知識，為本地圖書館網站設計並實作出書本<code>Book</code>,書本實例<code>BookInstance</code>，作者<code>Author</code>和種類<code>Genre</code>模型。</p>

<p>最後，我們創建一些實例，以測試模型(使用獨立運作的命令稿)。下一篇文章，我們將關注於如何創建一些網頁，以呈現這些物件。</p>

<h2 id="參閱">參閱</h2>

<ul>
 <li><a href="https://expressjs.com/en/guide/database-integration.html">Database integration</a> (Express docs)</li>
 <li><a href="http://mongoosejs.com/">Mongoose website</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/guide.html">Mongoose Guide</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/validation.html">Validation</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/schematypes.html">Schema Types</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/models.html">Models</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/queries.html">Queries</a> (Mongoose docs)</li>
 <li><a href="http://mongoosejs.com/docs/populate.html">Population</a> (Mongoose docs)</li>
</ul>

<p>{{PreviousMenuNext("Learn/Server-side/Express_Nodejs/skeleton_website", "Learn/Server-side/Express_Nodejs/routes", "Learn/Server-side/Express_Nodejs")}}</p>

<h2 id="本教程連結">本教程連結</h2>

<ul>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction">Express/Node introduction</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/development_environment">Setting up a Node (Express) development environment</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/Tutorial_local_library_website">Express Tutorial: The Local Library website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/skeleton_website">Express Tutorial Part 2: Creating a skeleton website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/mongoose">Express Tutorial Part 3: Using a Database (with Mongoose)</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/routes">Express Tutorial Part 4: Routes and controllers</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/Displaying_data">Express Tutorial Part 5: Displaying library data</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/forms">Express Tutorial Part 6: Working with forms</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Express_Nodejs/deployment">Express Tutorial Part 7: Deploying to production</a></li>
</ul>