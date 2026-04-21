# inventory-qr2

# QR棚卸アプリ セットアップ手順

## ファイル構成

| ファイル     | 配置先         | 役割                           |
|------------|--------------|-------------------------------|
| index.html | GitHub Pages | フロントエンド（カメラ・DB・UI） |
| Code.gs    | GAS           | バックエンド（データ受信・スプシ書込） |

---

## STEP 1: スプレッドシートを準備する

1. Google スプレッドシートを新規作成
2. URLの `/d/` と `/edit` の間の文字列がスプレッドシートID
   - 例: `https://docs.google.com/spreadsheets/d/【ここ】/edit`
3. このIDをメモしておく（Code.gs に設定します）

---

## STEP 2: GAS をセットアップする

1. Google ドライブで「新規」→「Google Apps Script」
2. **Code.gs の中身を全コピーして貼り付ける**
3. `const SHEET_ID = 'YOUR_SPREADSHEET_ID_HERE';` の部分を
   STEP1 でメモしたIDに書き換える

   例:
   ```javascript
   const SHEET_ID = '1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms';
   ```

4. 「デプロイ」→「新しいデプロイ」をクリック
5. 以下の設定でデプロイ:
   - 種類: **ウェブアプリ**
   - 次のユーザーとして実行: **自分**
   - アクセスできるユーザー: **全員**
6. 「デプロイ」→権限を承認
7. 発行された URL をコピーしておく（後でアプリに設定）

   URL 例:
   ```
   https://script.google.com/macros/s/AKfycby.../exec
   ```

> ⚠️ コードを修正したら「デプロイ」→「デプロイを管理」→「鉛筆アイコン」→
> バージョン「新しいバージョン」→「デプロイ」で更新してください

---

## STEP 3: GitHub Pages にアップする

1. GitHub にログイン（なければ無料アカウント作成）
2. 「New repository」でリポジトリ作成
   - 名前例: `qr-inventory`
   - Public を選択
3. 「uploading an existing file」から **index.html をアップロード**
4. 「Settings」→「Pages」→
   - Source: **Deploy from a branch**
   - Branch: **main** / **(root)**
   - 「Save」をクリック
5. 数分後、以下のURLでアクセス可能になる:
   ```
   https://【GitHubユーザー名】.github.io/qr-inventory/
   ```

---

## STEP 4: iPhoneでアプリとして登録する

1. **iPhoneのSafariで** STEP3のURLを開く
   （ChromeではなくSafariを使うこと）
2. 画面下部の「共有」ボタン（四角から矢印が出たアイコン）をタップ
3. 「ホーム画面に追加」をタップ
4. 「追加」→ ホーム画面にアイコンが追加される
5. アイコンから起動 → フルスクリーンアプリとして動作

---

## STEP 5: アプリの初期設定

1. アプリを起動 →「設定」タブを開く
2. 「GAS Web App URL」に STEP2 でコピーした URL を貼り付ける
3. 「現場コード」に現場識別コードを入力（例: PLANT-01）
4. 「マスタ」タブ → 商品マスタを登録
   - 手動追加 / CSV一括インポート / サンプル読込 から選択

---

## 送信されるデータ形式（GASへのPOST）

```json
{
  "action": "inventory_upload",
  "location": "PLANT-01",
  "sent_at": "2026-04-21T10:00:00.000Z",
  "count": 3,
  "items": [
    {
      "product_code": "BOLT-M6-20",
      "product_name": "ボルト M6×20",
      "unit": "本",
      "quantity": 15,
      "scanned_at": "2026-04-21T09:45:00.000Z"
    }
  ]
}
```

---

## スプレッドシートのシート構成

| シート名    | 用途                    |
|-----------|------------------------|
| 棚卸データ  | 受信した棚卸結果（自動作成） |
| 商品マスタ  | マスタ管理（手動作成可）    |
| 送信ログ   | 送信履歴（自動作成）       |

---

## ASTERIA Warp 連携（次ステップ）

GAS の `receiveInventory()` 関数内で
スプシ書込後に ASTERIA の API を呼ぶか、
スプシをASTERIAのトリガー元として使用します。

```javascript
// receiveInventory() の末尾に追加
callAsteria(rows);

function callAsteria(rows) {
  const url = 'https://your-asteria-endpoint.example.com/trigger';
  UrlFetchApp.fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json',
               'Authorization': 'Bearer YOUR_TOKEN' },
    payload: JSON.stringify({ rows: rows })
  });
}
```

---

## CSVインポート形式

マスタタブの「CSV」ボタンで一括インポートできます。

```csv
product_code,product_name,size,unit
BOLT-M6-20,ボルト M6×20,M6×20,本
NUT-M6,ナット M6,M6,個
```

- 1行目はヘッダー行（自動スキップ）
- 文字コード: UTF-8（BOM無し推奨）

---

## トラブルシューティング

| 症状 | 原因 | 対処 |
|-----|------|------|
| カメラが起動しない | HTTPSでない | GitHub PagesはHTTPS自動対応 |
| QRが読み取れない | 照明不足 | 明るい環境で試す |
| 送信が失敗する | GAS URLが間違い | 設定タブのURLを確認 |
| データが消えた | プライベートブラウジング | 通常モードで使う |
| マスタ取得エラー | GAS の CORS 設定 | doGet のアクセス権を「全員」に |
