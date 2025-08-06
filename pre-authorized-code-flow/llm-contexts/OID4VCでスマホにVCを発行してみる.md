-----

search
トレンド
質問
公式イベント
公式コラム[open\_in\_new](https://www.google.com/search?q=open_in_new)
Organization

-----

## はじめに

OpenID4VCIの動き

1)  QRコードを発行
2)  Credential Issuer Metadata の用意
3)  tokenを発行
4)  VCを発行
    簡易サーバで動作確認

<!-- end list -->

1.  index.html, server.js, package.jsを以下のフォルダ構成のように配置する
2.  関連モジュールをインストールする
3.  サーバに割り当てるドメイン名を取得
4.  取得したドメイン名を引数に、サーバを起動する
5.  ブラウザで[https://57dd-180-53-77-xxx.ngrok-free.app/にアクセス](https://57dd-180-53-77-xxx.ngrok-free.app/にアクセス)

Qiita広告表示について[open\_in\_new](https://www.google.com/search?q=open_in_new)
info
この記事は最終更新日から1年以上が経過しています。

@yk839

# OID4VCでスマホにVCを発行してみる

wallet
VerifiableCredentials
oid4vc
最終更新日 2024年03月12日
投稿日 2024年01月08日

[2024/2/17 update]
credo-ts libは、openid-credential-offerに含まれる"credentials":["OpenBadgeCredential"]をもとにissuerのmetadataを検索する。なのでmetadataに含まれる値、すなわち"credentials":["OpenBadgeCredentialJwt"]に変更

## はじめに

最近EUDI walletの話をよく聞くようになってきた。EUDIとは欧州デジタルIDウォレットのことらしい(参考)。IDウォレットではOpenID4VCI/OpenID4VP+SIOPv2を使って、VCの受け渡しをするようである。
図にすると以下のような感じになる(引用元：globalPlatform)
このOpenID4VCIを試す事ができるものがないか探してみると「Sphereon Wallet」を見つけることが出来た。さらに都合のいいことにios app storeで配付されていた([https://github.com/Sphereon-Opensource/ssi-mobile-wallet](https://github.com/Sphereon-Opensource/ssi-mobile-wallet))。

またOpenID4VCI/OpenID4VPをしゃべるdemoサーバも用意されていて、以下のサイトで試す事が出来る。

このプロトコルの中身を見ながら、最終的には簡易的なopenID4VCIをしゃべるサーバをnodejsで作ってみようと思う。

-----

## OpenID4VCIの動き

### 1\) QRコードを発行

demoサーバにブラウザでアクセスし、\*\*[manually fill out details]\*\*を選択して、適当なfirstname/lastname, emailを入力してcontinueをクリック。その後select credentials画面が出るので「University degree」を選択すると、下記のようなQRコードが発行される

QRコードの中身は以下のようになっている。

```
openid-credential-offer://?credential_offer={
  "grants":{
    "urn:ietf:params:oauth:grant-type:pre-authorized_code":{
      "pre-authorized_code":"AZcbrjCMa9StPQxVQRQAf",
      "user_pin_required":false
    }
  },
  "credentials":["OpenBadgeCredential"],
  "credential_issuer":"https://ssi.sphereon.com/pf3"
}
```

### 2\) Credential Issuer Metadata の用意

QRコードを読み取ったスマホは、issuerのMetadata情報(issuerはどのURLでCredentialsを発行していて、どこでtokenをもらえるかなどが書かれたJSON)を取得しにくる。demoサーバの場合、`https://ssi.sphereon.com/pf3/.well-known/openid-credential-issue`がそのURLである。以下のようなjsonが用意されている。

**metadata.json**

```json
{
   "credential_issuer":"https://ssi.sphereon.com/pf3",
   "credential_endpoint": "https://ssi.sphereon.com/pf3/credentials",
   "token_endpoint": "https://ssi.sphereon.com/pf3/token",
   "display":[{name":"Sphereon","description":"Sphereon JFF Plugfest3 Issuer"}],
   "credentials_supported":[
     {..}
     {..}
     {
       "display":[
         {
           "name":"Example University Degree",
           "description":"JFF Plugfest 3 OpenBadge (JWT)",
           "text_color":"#FFFFFF",
           "background_color":"#1763c1",
           "logo":{
             "url":"https://w3c-ccg.github.io/vc-ed/plugfest-1-2022/images/JFF_LogoLockup.png",
             "alt_text":"Red, magenta and yellow vertical lines with 3 black dots and the text JFF, depicting the Jobs For the Future logo."
           }
         },
         {
           "locale":"en-US",
           "name":"Example University Degree",
           "description":"JFF Plugfest 3 OpenBadge (JWT)"",
           "text_color":"#FFFFFF",
           "background_color":"#1763c1",
           "logo":{
             "url":"https://w3c-ccg.github.io/vc-ed/plugfest-1-2022/images/JFF_LogoLockup.png",
             "alt_text":"Red, magenta and yellow vertical lines with 3 black dots and the text JFF, depicting the Jobs For the Future logo."
           }
         }
       ],
       "id":"OpenBadgeCredentialJwt",
       "types":["VerifiableCredential","OpenBadgeCredential"],
       "format":"jwt_vc_json",
       "cryptographic_binding_methods_supported":["did:key"],
       "cryptographic_suites_supported":["EdDSA"]
     }
   ],
   "credential_supplier_config":{
     "templates_base_dir":"templates/sphereon",
     "template_mappings":[
       {..},
       {..},
       {
         "credential_types":["OpenBadgeCredential"],
         "template_path":"OpenBadgeCredential.hbs",
         "format":"jwt_vc_json"
       }
     ]
   }
 }
```

### 3\) tokenを発行

QRコードに書かれた`pre-authorized_code`を用いて、スマホはdemoサーバからtokenを取得する。curlで書くと以下になる

```bash
curl https://ssi.sphereon.com/pf3/token -X POST -H "Content-Type: application/json" -d @data1.txt
```

**data1.txt**

```json
{
 "client_id": "sphereon:ssi-wallet",
 "grant_type": "urn:ietf:params:oauth:grant-type:pre-authorized_code",
 "pre-authorized_code": "AZcbrjCMa9StPQxVQRQAf"
}
```

POSTに成功すると、以下のような値が返ってくる

```json
{
 "access_token":"eyJ0eXAiOiJKV1....6Vg8v4xrWbxA",
 "token_type":"bearer",
 "expires_in":300,
 "c_nonce":"48aa8701-dbec-4e56-a49e-2d9bf83f7ec3",
 "c_nonce_expires_in":300000,
 "authorization_pending":false,
 "interval":300000
 }
```

access\_tokenの中身は、以下のようなJWTになっている。

```json
{"typ":"JWT", "alg":"ES256K"}.
{
  "iat":1704349765484, "exp":300, 
  "iss":"https://ssi.sphereon.com/pf3",
  "preAuthorizedCode":"AZcbrjCMa9StPQxVQRQAf"
}.signature
```

### 4\) VCを発行

スマホは、metadata情報に記載されている`https://ssi.sphereon.com/pf3/credentials`に、3)で取得した`access_token`を用いてアクセスすることでVCを取得することができる。curlで書くと以下になる。

```bash
curl https://ssi.sphereon.com/pf3/credentials -X POST -H "Content-Type: application/json" -H "authorization: Bearer eyJ0eXAiOiJKV1....6Vg8v4xrWbxA" -d @data2.txt
```

**data2.txt**

```json
{
  "types": [ "OpenBadgeCredential" ],
  "format": "jwt_vc_json",
  "proof": {
    "proof_type": "jwt",
    "jwt": "eyJhbGciOiJFUzI1Nksi....7GootXw"
  }
}
```

このdata2.txtに書かれているjwtをbase64デコードすると中身は以下の通りである。token取得時にdemoサーバから発行されたc\_nonceや、issuer endpoint:`https://ssi.sphereon.com/pf3`を用いて生成されている。

```json
{
  "alg":"ES256K",
  "typ":"openid4vci-proof+jwt",
  "kid":"did:key:zQ3shdST2uDCn8eNRhTNYs3Yh6dDx9aYQSV7gYyFfhc9kCxnm"}.
{
  "iat":1704349849,"exp":1704353449,
  "aud":"https://ssi.sphereon.com/pf3",
  "nonce":"48aa8701-dbec-4e56-a49e-2d9bf83f7ec3",
  "iss":"sphereon:ssi-wallet",
  "jti":"C289A6F6-7731-49B6-BC4A-CF8247369D2D"
}.signature
```

POSTが成功すると、VC credentialが入ったJSONが返ってくる。

```json
{
  "credential":"eyJhbGciOiJFUzI1NiIsInR5c....kX8s64CusI6Tz-ZtJkQ",
  "format":"jwt_vc_json",
  "c_nonce":"1fbd4fa8-cd62-4b90-a07f-9201d97f9951",
  "c_nonce_expires_in":300000
}
```

credentialの中身をデコードすると、以下のようになっている。

```json
{"alg":"ES256","typ":"JWT"}.
{
	"exp":1704954740,
	"vc":{
		"@context":["https://www.w3.org/2018/credentials/v1","https://purl.imsglobal.org/spec/ob/v3p0/context.json"],
		"type":["VerifiableCredential","OpenBadgeCredential"],
		"credentialSubject":{
			"type":["AchievementSubject"],
			"achievement":{
				"id":"urn:uuid:ac254bd5-8fad-4bb1-9d29-efd938536926",
				"type":["Achievement"],
				"name":"JFF x vc-edu PlugFest 3 Interoperability",
				"description":".........<snip>.......",
				"criteria":{
					"type":"Criteria",
					"narrative":".........<snip>......."
				},
				"image":{
					"id":"https://w3c-ccg.github.io/vc-ed/plugfest-3-2023/images/JFF-VC-EDU-PLUGFEST3-badge-image.png",
					"type":"Image"
				}
			},
			"id":"did:key:zQ3shdST2uDCn8eNRhTNYs3Yh6dDx9aYQSV7gYyFfhc9kCxnm"
		}
	},
	"@context":["https://www.w3.org/2018/credentials/v1","https://purl.imsglobal.org/spec/ob/v3p0/context.json"],
	"type":["VerifiableCredential","OpenBadgeCredential"],
	"expirationDate":"2024-01-11T06:32:20.016Z",
	"name":"JFF x vc-edu PlugFest 3 Interoperability",
	"issuer":{
		"type":["Profile"],
		"name":"Jobs for the Future (JFF)",
		"url":"https://www.jff.org/",
		"image":"https://w3c-ccg.github.io/vc-ed/plugfest-1-2022/images/JFF_LogoLockup.png",
		"id":"did:jwk:eyJhbGciOiJ....M4In0"
	},
	"credentialSubject":{
		"type":["AchievementSubject"],
		"achievement":{
			"id":"urn:uuid:ac254bd5-8fad-4bb1-9d29-efd938536926",
			"type":["Achievement"],
			"name":"JFF x vc-edu PlugFest 3 Interoperability",
			"description":".........<snip>.......",
			"criteria":{
				"type":"Criteria",
				"narrative":".........<snip>......."
			},
			"image":{
				"id":"https://w3c-ccg.github.io/vc-ed/plugfest-3-2023/images/JFF-VC-EDU-PLUGFEST3-badge-image.png",
				"type":"Image"
			}
		},
		"id":"did:key:zQ3shdST2uDCn8eNRhTNYs3Yh6dDx9aYQSV7gYyFfhc9kCxnm"
	},
	"issuanceDate":"2024-01-04T06:32:20.016Z",
	"sub":"did:key:zQ3shdST2uDCn8eNRhTNYs3Yh6dDx9aYQSV7gYyFfhc9kCxnm",
	"nbf":1704349940,
	"iss":"did:jwk:eyJhbGciOiJ....M4In0"
}.signature
```

-----

## 簡易サーバで動作確認

demoサーバ：`https://ssi.sphereon.com/demo/issuer/`の動きをnodejsで実装し、SSI sphereon walletにVCを発行してみる

### 1\. index.html, server.js, package.jsを以下のフォルダ構成のように配置する

```
├── server.js
├── package.json
└── views/
   ├── index.html
```

index.htmlのコードを見る

server.jsのコードを見る

package.jsonのコードを見る

### 2\. 関連モジュールをインストールする

```bash
> npm install
```

### 3\. サーバに割り当てるドメイン名を取得

```bash
> ngrok http 8888
```

```
Region                   United States (us)
Web Interface            http://127.0.0.1:4040
Forwarding               http://57dd-180-53-77-xxx.ngrok-free.app -> http://localhost:8888
Forwarding               https://57dd-180-53-77-xxx.ngrok-free.app -> http://localhost:8888
```

### 4\. 取得したドメイン名を引数に、サーバを起動する

```bash
> nodejs server.js 57dd-180-53-77-xxx.ngrok-free.app
```

```
 web page start: listening on port 8888...

 this did:web:6f3d-153-219-137-227.ngrok-free.app's key is registered

 did:web:57dd-180-53-77-xxx.ngrok-free.app
 did:key:zQ3shq4taAWbUYmvaf4gNmfR9ZtJ1wN8nauwRxnacLQarYTCp
 did:jwk:eyJhbGciOiJFUzI1NksiLCJ1c2UiOiJzaWciLCJrdHkiOiJFQyIsImNydiI6InNlY3AyNTZrMSIsIngiOiJtdTR4b0JHV0lmd0VvYkoyN3krcitMYjhCSklqaC9kU1pYRmlRaEN6clIwIiwieSI6InJ5Nkh5QmVqNENOVllveGZZYmQrOW1ibVZWU0VySEdDUUQ1NnJ5TFFrenMifQ
```

### 5\. ブラウザで[https://57dd-180-53-77-xxx.ngrok-free.app/にアクセス](https://57dd-180-53-77-xxx.ngrok-free.app/にアクセス)

画面上にある「発行」ボタンを押してQRコードを表示
表示されたQRコードを、スマホでスキャンするとVC発行の手続きが始まる

-----

comment
0

-----

新規登録して、もっと便利にQiitaを使ってみよう

あなたにマッチした記事をお届けします
便利な情報をあとで効率的に読み返せます
ダークテーマを利用できます
ログインすると使える機能について
関連記事 Recommended by

Encrypted Credentials with Symbol
by inatatsu\_csg

OAuth & OpenID Connect 関連仕様まとめ
by TakahikoKawasaki

OpenID Connectを使ったアプリケーションのテストのためにKeycloakを使ってみる
by shibukawa

OpenID Connect Self-issued OP概要 \~OpenID Technight ...
by samiii

「DocuWare」で業務のデジタルシフトを手厚く支援PR 株式会社PFU

「美しい設計とは何か」を追求。DTSインサイトで築いたキャリアとは？PR 株式会社DTSインサイト
コメント
この記事にコメントはありません。
いいね以上の気持ちはコメントで

How developers code is here.
© 2011-2025Qiita Inc.
ガイドとヘルプ
About
利用規約
プライバシーポリシー
ガイドライン
メディアキット
ご意見・ご要望
ヘルプ
広告掲載
コンテンツ
リリースノート
公式イベント
公式コラム
アドベントカレンダー
Qiita Tech Festa
Qiita 表彰プログラム
エンジニア白書
API
公式アカウント

Qiita（キータ）公式

Qiita マイルストーン

Qiita 人気の投稿
Facebook
YouTube
ポッドキャスト
Qiita 関連サービス
Qiita Team
Qiita Zine
Qiita 公式ショップ
運営
運営会社
採用情報
Qiita Blog
ニュースリリース