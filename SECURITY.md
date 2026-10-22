# セキュリティメモ

## 構成
- 静的HTML（GitHub Pages）＋ Firebase Auth（Googleログイン）＋ Firestore
- サーバー側コード（PHP / Node.js）は無い
- FirebaseのAPIキーはクライアントに公開される前提の値。データを守るのは Firestore ルール

## 対策一覧
| 対策 | 場所 |
|---|---|
| 本人以外の読み書き禁止・入力値の型/長さ検証 | `firestore.rules`（コンソールで公開が必要） |
| 自由入力（メモ・ペース等）の出力エスケープ | `index.html` の `esc()` |
| 保存前の入力検証（ルールと同条件） | `index.html` の `buildLog()` |
| インラインイベントハンドラの廃止（イベント委譲） | `index.html` の `__appInit` |
| CSP（スクリプト・通信先・フレームの許可リスト） | `index.html` の `<meta http-equiv="Content-Security-Policy">` |
| APIキーのHTTPリファラ制限 | Google Cloud コンソール（下記） |
| 承認済みドメインの限定 | Firebase コンソール（下記） |

## index.html のスクリプトを編集したら
CSP の `script-src 'sha256-...'` は `<script type="module">` の中身のハッシュ。
1文字でも変えたら再計算しないとアプリが動かなくなる。

```
python3 - <<'PY'
import re,hashlib,base64
c=open('index.html',encoding='utf-8').read()
s=re.search(r'<script type="module">(.*?)</script>',c,re.S).group(1)
print("sha256-"+base64.b64encode(hashlib.sha256(s.encode()).digest()).decode())
PY
```

## コンソール側の設定（コードでは設定できない）
1. Firestore ルール：`firestore.rules` の内容を貼り付けて公開
2. APIキー制限：Google Cloud コンソール → 認証情報 → APIキー →
   アプリケーションの制限「ウェブサイト」に
   `https://moomin-pay.github.io/*` と `https://running-tracker-db-2e192.firebaseapp.com/*` を追加
3. Firebase Authentication → 設定 → 承認済みドメイン：
   `localhost`（不要なら削除）・`running-tracker-db-2e192.firebaseapp.com`・`moomin-pay.github.io` 以外を削除
