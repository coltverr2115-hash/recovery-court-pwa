# RECOVERY COURT (PWA版) セットアップ手順

このアプリは「ホーム画面に追加できるWebアプリ(PWA)」です。ビルドツール不要の素のHTML/CSS/JSなので、Firebaseの設定さえ終われば、そのままVercelに置くだけで動きます。

## 構成
```
recovery-court-pwa/
  index.html          アプリ本体
  manifest.json        PWA設定(アイコン・アプリ名など)
  service-worker.js    オフライン時の簡易キャッシュ
  firebase-config.js   ← ここにあなたのFirebaseプロジェクトの情報を入れる
  icons/                アプリアイコン(自動生成済み)
```

## 手順1: Firebaseプロジェクトを作る(無料)
1. https://console.firebase.google.com/ にアクセスし、Googleアカウントでログイン
2. 「プロジェクトを作成」→ 好きな名前(例: recovery-court)を入力して作成
3. 左メニューの「構築」→「Firestore Database」→「データベースの作成」
   - ロケーションは `asia-northeast1`(東京)がおすすめ
   - 最初は「テストモードで開始」でOK(下の手順4で必ずルールを調整してください)
4. 左メニューの「プロジェクトの概要」の横にある歯車 →「プロジェクトの設定」
5. 下にスクロールし「マイアプリ」→ `</>`(ウェブ)のアイコンをクリックしてアプリを登録
6. 表示される `firebaseConfig` の中身(apiKey, authDomain, projectId など)をコピー

## 手順2: firebase-config.js を編集
`firebase-config.js` を開き、`YOUR_API_KEY` などの部分を、手順1でコピーした値に置き換えてください。

## 手順3: Firestoreのセキュリティルールを設定
Firebaseコンソールの「Firestore Database」→「ルール」タブで、以下のように設定してください(テストモードのままだと30日で自動的に書き込み禁止になり、また誰でも読み書きできてしまいます)。

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

**重要な注意点**: このルールは「URLを知っている人なら誰でもデータを読み書きできる」設定です。アプリ内のPIN機能はあくまで画面上の簡易的なロックであり、Firestoreのデータ自体を技術的に保護するものではありません。選手の記録を本当に監督/トレーナーしか見られないようにしたい場合は、Firebase Authentication(ログイン機能)を追加し、ルール側で `request.auth` をチェックする形に変更する必要があります。必要であれば、その対応も追加で作成できます。

## 手順4: 動作確認(ローカル)
ブラウザのセキュリティ制限で `index.html` を直接ダブルクリックしただけでは正しく動かないことがあるため、簡易サーバーを立てて確認してください。

```bash
cd recovery-court-pwa
npx serve .
```
表示されたURL(例: http://localhost:3000)をブラウザで開いて確認します。

## 手順5: Vercelにデプロイ
1. このフォルダをGitHubリポジトリにアップロード(keiba-pwaと同じ流れでOK)
2. https://vercel.com/ で「New Project」→ そのリポジトリを選択
3. Framework Presetは「Other」のままでOK(ビルドコマンド不要の静的サイトです)
4. Deployをクリック

デプロイが終わると、選手・監督ともに同じURLにアクセスすればOKです。スマホでURLを開き、下に出てくる「ホーム画面に追加」バナー(またはブラウザの共有メニュー→「ホーム画面に追加」)からアプリのように使えます。

## アイコンについて
`icons/` フォルダに、このアプリ用のシンプルなアイコン(192px, 512px, マスク対応512px)を用意しています。デザインを変更したい場合は同じファイル名で差し替えてください。
