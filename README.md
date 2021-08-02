# TAECRevitNotion
## 🧐前置き
### ▽このリポジトリはなに？
- TAECのRevit⇔Notion連携ハンズオンの資料です  
- NotionAPIを用いてRevit⇒Notion/Notion⇒Revit双方向の接続を試みます  
- meetupTAEC[渡邊さんの回](https://www.meetup.com/ja-JP/Tokyo-AEC-Industry-Dev-Group/events/xfrxvrycckbnb/)をRevitに応用したものです
### ▽具体的にやること全体像
![image](https://user-images.githubusercontent.com/6135252/125967744-410cb46c-8bba-4bcc-a40f-a564e5b6faa8.png)
↑の①と③の部分を作ります
### ▽必要なもの
- Notion(Admin権限Workspace)
- Postman
- Revit2021↑/Dynamo2.5.0↑  
（jsonを使う制約で，厳密にはRevit2020.2↑/Dynamo2.3.0↑ですが環境がなくて動作確認できていません）
### ▽事前にみておくといいです
- [Getting Started with NotionAPI](https://www.notion.so/Getting-Started-with-NotionAPI-8dbc03801bf54d23b4ffded7d7981a73)2章まで
- [zenn-マッシュアッパーのためのwebAPI入門](https://aaa) 今回のために勉強かねてまとめました（まだ編集中，出来たらリンク先更新）

## ☺本題
### 1 NotionAPIでNotionにテーブルを作る
まずは，Revitの拡張DBとしてのテーブルをNotion上に作成します。  
もちろん手動でテーブルを作ることはできますが，せっかくなのでAPIを使ってテーブルを作成します。
#### 1-1 Notionのadmin権限Workspaceを作成する
- 現在開いているWorkspace名をクリック  
![image](https://user-images.githubusercontent.com/6135252/127806725-9ddd133a-7d41-4516-bb66-9c0f7cc09784.png)
- "・・・"をクリック  
![image](https://user-images.githubusercontent.com/6135252/127806919-99b236a7-ad1e-438e-9371-59acfae6a9ba.png)
- "Join or create workspace"を選択  
![image](https://user-images.githubusercontent.com/6135252/127806547-06af1715-2523-4574-ba14-e8955a3d6678.png)
- "For myself"を選択し，"Take me to Notion"をクリック  
![image](https://user-images.githubusercontent.com/6135252/127806590-b2a666e9-93a4-4d54-b4c9-b6794619255e.png)
- "Add a page"をクリックし，DBを作成するpageを作成する  
![image](https://user-images.githubusercontent.com/6135252/127811433-90842cb8-a1dc-44ce-ae33-69bd93e35861.png)

#### 1-2 NotionのIntegration(=API接点)を作成する
Workspaceを作成した状態では，APIによる操作を受け入れる口が設けられておりません。  
これをIntegrationと呼んでおり，手動で設定する必要があります。  
- 「My Integrations」（インテグレーションの管理ページ）にアクセス  
[My Integrations](https://www.notion.so/my-integrations)
- "Create new integration"をクリック  
![image](https://user-images.githubusercontent.com/6135252/127807644-cb490762-cc6f-4f8e-b15c-2dc19ab1f615.png)
- "Name","Logo","Associated workspace"を記入(選択)し，"Submit"  
![image](https://user-images.githubusercontent.com/6135252/127807473-bef44444-42d1-47e7-8854-4179c72609a6.png)
- 作成したIntegrationのトークンを控えておく（後で何度も使います）
![image](https://user-images.githubusercontent.com/6135252/127807847-6a281899-68ef-4744-917b-95d1f5da823c.png)  
```
❕  
トークンはWorkspaceにアクセスするためのキーになります。  
外部に漏れると意図しないユーザーからの操作を受け入れることになってしまうので，必ずセキュアに管理してください。
```
- 1-1で作成したページにて，"Share"をクリック  
![image](https://user-images.githubusercontent.com/6135252/127811735-eaf6bc3c-7c66-487f-aa33-fc37975a6fb9.png)
- "Add people, emails, groups, or integrations"をクリックし，作成したIntegrationをInvite  
![image](https://user-images.githubusercontent.com/6135252/127811796-854e6ee1-1f63-420b-aef2-90dd0162b341.png)
#### 1-3 NotionAPIでテーブルを用意する
Postman（APIを簡単に叩くためのツール）を使って，APIによるテーブル作成を行います。  
今回は下記の如く，dummy(text)/id(number)/x(number)/y(number)の4フィールドのテーブルを作成します。  
![image](https://user-images.githubusercontent.com/6135252/127809724-0a244b45-0fbf-479f-b5f1-985c1e2511e8.png)
- Postmanに新規requestを作成。名前は"Create a table"とでも
- 下記の設定でリクエストをたたく
  - リクエストライン/ヘッダ  

  | Tab | Name | Key | Value |  
  | :--- | :--- | :--- | :--- |  
  | General | メソッド |  | POST |  
  | General | URI |  | https://api.notion.com/v1/databases/ |  
  | Authorization | Token | Bearer | Integrationのトークン |  
  | Headers | Header | Notion-Version | 2021-05-13 |  
  | Headers | Header | Content-Type | application/json |  
  - ボディ  
  ```JSON:body.json
  {
    "parent": {
        "type": "page_id",
        "page_id": "ページIDをペースト"
    },
    "title": [
        {
            "type": "text",
            "text": {
                "content": "Augmented Revit DB",
                "link": null
            }
        }
    ],
    "properties": {
        "dummy": {
            "title": {}
        },
        "id": {
            "number": {}
        },
        "x": {
            "number": {}
        },
        "y": {
            "number": {}
        }
    }
  }
  ```
```
❕  
ページIDは，ページのURLから確認できます。規則は下記の通りです。  
https://www.notion.so/[ページ名]-[ページID]
```
- 成功すればこのように，空白だったページにテーブルオブジェクトが作成されます。  
![image](https://user-images.githubusercontent.com/6135252/127812584-74e053bf-b9eb-4c5e-8233-b4e8007e703f.png)
![image](https://user-images.githubusercontent.com/6135252/127812598-065a9fbd-bc1b-4992-bd81-daa6b743e9e7.png)
- テーブルのIDを控えます（後で使います）
```
❕  
テーブルIDも，テーブルを全画面表示した時のURLから確認できます。規則は下記の通りです。  
https://www.notion.so/[テーブルID]?v=[何かのパラメータ]
```  

### 2 Revitの情報をDynamo介してNotionのテーブルに突っ込む  
リポジトリのテストプロジェクト(taec_testpj.rvt)には，パラメトリックな直方体の一般ファミリ(F1.rfa)のインスタンスが6つ配置されています。  
上記6つのF1のインスタンスのElementID/x座標/y座標を取得し，Notionのテーブルにレコードとして挿入します。  
![image](https://user-images.githubusercontent.com/6135252/127825246-6f28cfdc-6906-4bb9-8e9c-394bac73c2ff.png)
- Dynamo(Send2Notion.dyn)
![Send2Notion_2021-08-02_05-08-41](https://user-images.githubusercontent.com/6135252/127826897-4b54af3d-7f55-47d2-a25e-90872034993b.png)  

Extract data  
```Python:Extractdata.py
import clr

clr.AddReference('RevitAPI')
from Autodesk.Revit.DB import *
from Autodesk.Revit.DB.Structure import *

clr.AddReference('RevitAPIUI')
from Autodesk.Revit.UI import *

clr.AddReference('System')
from System.Collections.Generic import List

clr.AddReference('RevitNodes')
import Revit
clr.ImportExtensions(Revit.GeometryConversion)
clr.ImportExtensions(Revit.Elements)

clr.AddReference('RevitServices')
import RevitServices
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager

doc = DocumentManager.Instance.CurrentDBDocument
uidoc=DocumentManager.Instance.CurrentUIApplication.ActiveUIDocument

# このノードへの入力は、リスト形式で IN 変数に格納されます。
elems = IN[0]

# この行の下にコードを配置します
data = []

for elem in elems:
	dict = {
		"id":elem.Id,
		"x":round(elem.Location.X),
		"y":round(elem.Location.Y)
	}
	data.append(dict)

# 出力を OUT 変数に割り当てます。
OUT = data
```  
Send data to Notion  
```Python:SenddatatoNotion.py
from System.Net import WebRequest
from System.Text import ASCIIEncoding
from System.IO import StreamReader
from System.Net import ServicePointManager
from System.Net import SecurityProtocolType
from System.Net import WebException
import time, json

# Functions
def http(method, url, data_string=None, header={}, retry=3):
    ServicePointManager.SecurityProtocol |= SecurityProtocolType.Tls11 | SecurityProtocolType.Tls12
    request = WebRequest.Create(url)
    request.UseDefaultCredentials = True
    request.Method = method
    request.ContentLength = 0
    
    for k,v in header.items():
        request.Headers.Add(k,v)
    
    if data_string:
        request.ContentType = "application/json"
        encoding = ASCIIEncoding()
        data = encoding.GetBytes(data_string)
        request.ContentLength = data.Length
        stream = request.GetRequestStream()
        stream.Write(data, 0, data.Length)
        stream.Close()

    try:
        response = request.GetResponse()
        print (response.StatusDescription)
        dataStream = response.GetResponseStream()
        reader = StreamReader(dataStream)
    
        responseFromServer = reader.ReadToEnd()
            
        reader.Close()
        dataStream.Close()
        response.Close()
        return responseFromServer

    except WebException as e : 
        if e.Response.StatusDescription == "Not Found" and retry>0:
            time.sleep(10)
            return http(method, url, data_string, header, retry-1)

        print (e.Response.StatusDescription)
        dataStream = e.Response.GetResponseStream()
        reader = StreamReader(dataStream)
    
        responseFromServer = reader.ReadToEnd()
        print (responseFromServer)
    
        reader.Close()
        dataStream.Close()

# このノードへの入力は、リスト形式で IN 変数に格納されます。
dicts = IN[0]

# この行の下にコードを配置します

# URL
url = "https://api.notion.com/v1/pages"
# Token
token = "Bearer 取得したトークン"
# headers
headers = {
            'Authorization': token,
            'Notion-Version': "2021-05-13",
            }

for dict in dicts:
    values = {
        "parent": {"database_id": "テーブルID"},
        "properties": {
            "id": {
                "number":dict["id"]
            },
            "x": {
                "number":dict["x"]
            },
            "y": {
                "number":dict["y"]
            },
        }
        }
    # bodyを文字列化
    data = json.dumps(values)
    # POST
    http("POST",url,data,headers)

# 出力を OUT 変数に割り当てます。
OUT = 0
```  
- 実行結果  
  上記を実行すると，こんな感じでNotionのテーブルにレコードが挿入されます  
![image](https://user-images.githubusercontent.com/6135252/127834096-86519377-f834-4d77-85f9-5f64c8aea901.png)
- 適当に編集  
  x/yはそれぞれプロジェクトの絶対座標[mm]なので，これを適当に書き換えてください。  
  ただしidによってインスタンスとの紐づけを行っているので，idは書き換えないように注意してください。  
  
### 3 Notionのテーブルで変更した情報をDynamo介してRevitに反映する  
Notionのテーブルで書き換えたx/yの値をRevitプロジェクトに反映させます。  
- Dynamo(GetFromNotion.dyn)
![GetFromNotion_2021-08-02_06-20-42](https://user-images.githubusercontent.com/6135252/127837458-f66ead3f-ed98-4250-8982-51ea714aacbc.png)  

Get data from Notion  
```Python:GetDataFromNotion.py
from System.Net import WebRequest
from System.Text import ASCIIEncoding
from System.IO import StreamReader
from System.Net import ServicePointManager
from System.Net import SecurityProtocolType
from System.Net import WebException
import time, json

# Functions
def http(method, url, data_string=None, header={}, retry=3):
    ServicePointManager.SecurityProtocol |= SecurityProtocolType.Tls11 | SecurityProtocolType.Tls12
    request = WebRequest.Create(url)
    request.UseDefaultCredentials = True
    request.Method = method
    request.ContentLength = 0
    
    for k,v in header.items():
        request.Headers.Add(k,v)
    
    if data_string:
        request.ContentType = "application/json"
        encoding = ASCIIEncoding()
        data = encoding.GetBytes(data_string)
        request.ContentLength = data.Length
        stream = request.GetRequestStream()
        stream.Write(data, 0, data.Length)
        stream.Close()

    try:
        response = request.GetResponse()
        print (response.StatusDescription)
        dataStream = response.GetResponseStream()
        reader = StreamReader(dataStream)
    
        responseFromServer = reader.ReadToEnd()
            
        reader.Close()
        dataStream.Close()
        response.Close()
        return responseFromServer

    except WebException as e : 
        if e.Response.StatusDescription == "Not Found" and retry>0:
            time.sleep(10)
            return http(method, url, data_string, header, retry-1)

        print (e.Response.StatusDescription)
        dataStream = e.Response.GetResponseStream()
        reader = StreamReader(dataStream)
    
        responseFromServer = reader.ReadToEnd()
        print (responseFromServer)
    
        reader.Close()
        dataStream.Close()

# この行の下にコードを配置します
# URL
database_id = "テーブルID"
url = "https://api.notion.com/v1/databases/{}/query/".format(database_id)
# Token
token = "Bearer 取得したトークン"
# headers
headers = {
	'Authorization': token,
        'Notion-Version': "2021-05-13"
	}
# body
values = {}
data = json.dumps(values)

#run webAPI
res = http("POST",url,data,headers)
objects = json.loads(res)["results"]

# 出力を OUT 変数に割り当てます。
OUT = res
```

Extract data from json  
```Python:ExtractDataFromJSON.py
import time, json

# このノードへの入力は、リスト形式で IN 変数に格納されます。
res = IN[0]

# この行の下にコードを配置します
objects = json.loads(res)["results"]

data = []
for obj in objects:
	props = obj["properties"]
	record = {
		"id":props["id"]["number"],
		"x":props["x"]["number"],
		"y":props["y"]["number"]
	}
	data.append(record)

# 出力を OUT 変数に割り当てます。
OUT = data
```
Update instances by data
```Python:UpdateInstancesByData.py
import clr
clr.AddReference('RevitAPI')
from Autodesk.Revit.DB import*

clr.AddReference('RevitServices')
from RevitServices.Persistence import DocumentManager
from RevitServices.Transactions import TransactionManager
uiapp = DocumentManager.Instance.CurrentUIApplication
app = uiapp.Application

#input assigned the IN variable
data = IN[0]


#default document set to DocumentManager.Instance.CurrentDBDocument
doc = DocumentManager.Instance.CurrentDBDocument

#core data processing
def update_instance(record):
	#パラメータセット
	TransactionManager.Instance.EnsureInTransaction(doc)
	
	elem = doc.GetElement(ElementId(record["id"]))
	dxyz = XYZ(
		record["x"]/304.8-elem.Location.Point[0],
		record["y"]/304.8-elem.Location.Point[1],
		0
	)
	ElementTransformUtils.MoveElement(doc,elem.Id,dxyz)
	
	TransactionManager.Instance.ForceCloseTransaction()
	
	return elem
	
a = []
for record in data:
	b = update_instance(record)
	a.append(b)

#output assigned the OUT variable
OUT = a
```
- 実行結果  
  上記を実行すると，Notionのテーブルからデータを取得し，各インスタンスのx座標およびy座標を更新します。
- JSONの解析  
  NotionAPIで取得したJSONは[JSONViewer](http://jsonviewer.stack.hu/)などで確認すると，構造がいい感じに可視化されてコードが書きやすいです。  
  ![image](https://user-images.githubusercontent.com/6135252/127849101-f2a149db-ef80-4919-a861-f16f696a257c.png)



## 🥳おわりに
### ▽まとめ
Revitの外部にあるNotionテーブルとRevitのプロジェクトをDynamoでバッチ同期できるようにしたことで，Revitのデータベースが拡張されました。  

### ▽これ何に使えそう？
- 正直あまり思いついてませんが，何かできそうな気はします
- 非RevitユーザーにRevitの特定の内部データを編集させたいケース？
- Revit内に埋め込みたくないファミリのメタデータをNotion側で管理したいケース？

### ▽エクササイズ
F1はインスタンスパラメータL/W/Hを持っており，直方体ジオメトリの3方に連動しています。  
ElementID/x/yに加えてL/W/HもNotionのテーブルフィールドに追加し，Notion側から変更できるようにしてみましょう。  
![image](https://user-images.githubusercontent.com/6135252/127825827-dd3217a8-68cd-4395-83f1-7cb4ddfa3952.png)
