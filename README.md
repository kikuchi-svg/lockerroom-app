# ロッカールーム管理

人別・日付ベースで「ロッカールーム」（状態目標 / 成果目標 / 行動目標）を管理する静的Webアプリ。

🔗 公開URL: <https://kikuchi-svg.github.io/lockerroom-app/>

## 特徴

- ブラウザだけで動く（サーバー不要 / 静的サイト）
- **共有モード（Firebase）**: 全員のデータがリアルタイム同期 — みんなが保存したものをアプリ上で見れる
- **ローカルモード**: Firebase未設定なら、データは各自のブラウザに保存
- 入力からPowerPoint（.pptx）を自動生成してダウンロード
- 次回ロッカールーム日の **1週間前** に通知（自分 + 相手の両方）
  - **ChatWork** に投稿（API設定済みの場合）
  - 未設定なら **メール下書き（mailto:）** にフォールバック
- JSON エクスポート / インポートでバックアップ・PC間移行

## 使い方

1. 右上「設定」で自分の名前 / メール（任意で ChatWork APIトークン）を保存
2. 「＋ 新規追加」で **人 / 日付 / 期日 / 状態目標 / 成果目標 / 行動目標 / 次回ロッカールーム日** を入力
3. 「保存して pptx ダウンロード」で PowerPoint が生成される
4. 次回日が1週間以内になると 🔔 リマインドボタンが出現 → クリックでChatWork投稿 or メール下書き

ヘッダ右上の接続ピル：
- ☁ 共有モード — Firebaseに接続済み、全員でデータ共有
- 💾 ローカル — Firebase未設定、自分のブラウザ内のみ
- ⚠ 接続失敗 — Firebase設定はあるが接続できない（ローカルにフォールバック）

## Firebase（共有モード）の設定

全員で同じデータを見るには、Firebase Firestore を1回だけセットアップします（無料）。

### 1. Firebase プロジェクトを作る

1. <https://console.firebase.google.com/> にGoogleアカウントでログイン
2. 「**プロジェクトを追加**」をクリック
3. プロジェクト名（例：`lockerroom-app`）を入力 → 続行
4. Google Analyticsは「**今回は有効にしない**」でOK → プロジェクトを作成

### 2. Firestore Database を有効化

1. 左サイドメニューの「**ビルド**」→「**Firestore Database**」
2. 「**データベースの作成**」をクリック
3. ロケーション: `asia-northeast1`（東京）など
4. ルール: **本番モード** を選択（あとで書き換えます）

### 3. Firestore のセキュリティルールを設定

「**ルール**」タブで以下に書き換えて公開：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /entries/{id} {
      allow read, write: if request.auth != null;
    }
  }
}
```

これで「匿名サインインしたユーザー（＝アプリを開いた人）だけが読み書きできる」状態になります。

### 4. 匿名認証を有効化

1. 左サイドメニュー「**ビルド**」→「**Authentication**」
2. 「**始める**」→「**Sign-in method**」タブ
3. 「**匿名**」を選択 → 有効化 → 保存

### 5. Web アプリを登録して config を取得

1. 左上の歯車アイコン → 「**プロジェクトの設定**」
2. 下にスクロール → 「**マイアプリ**」→ Webの `</>` アイコンをクリック
3. アプリのニックネーム（例：`lockerroom-web`）を入力 → 登録
4. 表示される `firebaseConfig` オブジェクトをコピー（apiKey, authDomain, projectId など）

### 6. アプリに貼り付け

`index.html` を開いて、次の部分を埋める：

```js
const FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "....firebaseapp.com",
  projectId: "...",
  storageBucket: "....appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

### 7. デプロイ

```bash
git add index.html
git commit -m "feat: connect Firebase"
git push
```

GitHub Pagesが1〜2分で反映 → リロードすると ☁ 共有モード になります。

> **既存のローカルデータがある場合**: 共有モードに切り替えて最初に開くと、「ローカルに N 件あります。共有データベースにアップロードしますか？」と確認が出ます。

## ChatWork 連携

1. ChatWork → 右上アイコン → サービス連携 → API Token を取得
2. 本アプリの「設定」に貼り付け → 「接続テスト」で確認
3. 各エントリに以下を入力：
   - 相手のChatWorkアカウントID（メンション用）
   - ChatWorkルームID（投稿先）

> ※ APIトークンは各自のブラウザのlocalStorageに保存。共用PCでは要注意。

## ファイル構成

- `index.html` — アプリ本体（pptxテンプレ + JSZip + ロジック）
- `jszip.min.js` — pptx生成ライブラリ
- `template.pptx` — 元テンプレート（フォールバック用）

## セキュリティモデル

- **アプリ本体（HTML/JS）**: GitHub Pages で公開、誰でも閲覧可
- **Firebase 設定（apiKey 等）**: ソースに含まれるため誰でも見える（これはFirebaseの設計上想定された動作）
- **Firestore ルール**: 「匿名サインイン済みのアクセスのみ許可」 — アプリ経由でしかデータに触れない
- **ChatWork APIトークン / 個人設定**: 各自のブラウザのlocalStorageに保存。サーバー側に送信されない
- **データのアクセス制御**: リンクを知っている人なら誰でも読み書き可能 — 社内ツールなどリンクに信頼境界を置く運用想定

## ライセンス

社内利用向け。
