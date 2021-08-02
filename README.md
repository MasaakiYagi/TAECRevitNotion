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
- [zenn-マッシュアッパーのためのwebAPI入門](https://aaa) 今回のために勉強かねてまとめました

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
|Tab|Name|Key|Value|  
|:---|:---|:---|:---|  
|General|メソッド||POST|  
|General|URL||https://api.notion.com/v1/databases/|  
|Authorization|Token|Bearer|Integrationのトークン|  
|Headers|Header|Notion-Version|2021-05-13|  
|Headers|Header|Content-Type|application/json|  
  - ボディ  
```json
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
### 2 Revitの情報をDynamo介してNotionのテーブルに突っ込む
### 3 Notionのテーブルで変更した情報をDynamo介してRevitに反映する

## 結言
### ▽これ何に使えそう？
