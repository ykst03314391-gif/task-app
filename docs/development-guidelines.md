# 開発ガイドライン

## 1. 技術方針

- **素の HTML / CSS / JavaScript（ES Modules）** で実装する。フレームワーク・ビルドツール・実行時依存ライブラリは使わない。
- Node.js 20 以上（開発用サーバーとテストのみ）。`npm install` 不要。
- テストは Node 標準の `node:test` と `node:assert` を使う。

## 2. ディレクトリ構成

```
task-app/
├─ CLAUDE.md                     AI アシスタント向けの作業メモ
├─ README.md                     使い方
├─ package.json                  npm scripts のみ
├─ server.js                     開発用静的サーバー（依存なし）
├─ docs/
│  ├─ product-requirements.md    何を・なぜつくるか
│  ├─ functional-design.md       どうつくるか（画面・データ・ロジック）
│  └─ development-guidelines.md  このファイル
├─ src/                          配信されるファイル一式
│  ├─ index.html
│  ├─ styles.css
│  └─ js/
│     ├─ app.js                  DOM 描画・イベント
│     ├─ store.js                永続化
│     ├─ tasks.js                ドメインロジック（純粋関数）
│     └─ dates.js                日付ユーティリティ（純粋関数）
├─ tests/                        node:test のテスト
└─ android/                      Android 版（Android Studio のプロジェクト。8 章）
```

## 3. コーディング規約

- インデント 2 スペース、セミコロンあり、シングルクォート。
- `tasks.js` / `dates.js` は **DOM・localStorage・`Date.now()` に依存しない**。「今日」は引数で受け取る。
- state は直接書き換えない。更新関数は新しいオブジェクトを返す。
- DOM への文字列挿入は `textContent` を使う。ユーザー入力を `innerHTML` に入れない。
- 関数名・変数名は英語。UI 文言はひらがな中心の日本語。

## 4. UI 文言ルール

- 低学年が読めることを優先し、ひらがなで書く（例: 「追加」→「ふやす」、「削除」→「けす」、「保存」→「ほぞん」）。
- 否定や叱る表現を使わない（「まだだよ」「つぎはがんばろう」など前向きに）。
- 文は短く。1ボタン1〜5文字程度。

## 5. UI デザインルール

- タップ領域は最小 56px。主要ボタンは 64px 以上。
- 基本文字サイズ 20px 以上。
- しゅるいごとの色: しゅくだい=青、おてつだい=オレンジ、じゅんび=緑、そのほか=紫。色に加えてアイコン・文字でも区別する。
- アニメーションは `@media (prefers-reduced-motion: reduce)` で無効化する。

## 6. テスト

```bash
npm test
```

- ロジック（`tasks.js` / `dates.js` / `store.js` の検証部分）は必ずテストを書く。
- 日付をまたぐ処理（持ち越し・曜日・月末）は境界値テストを入れる。
- 画面は `npm start` で起動し、ブラウザで手動確認する。

## 7. データ形式の変更

- 保存形式を変えるときは `version` を上げ、`store.js` に旧バージョンからの移行処理を追加する。
- 同時に `docs/functional-design.md` のデータモデルを更新する。
- Android 版も同じ形式を使うので、Android 側の `data` にも同じ移行処理を入れる。

## 8. Android 版

### 8.1 技術方針

- **Kotlin ＋ Jetpack Compose**。開発は Windows の Android Studio で行う。
- 使ってよいライブラリは AndroidX（Compose・Activity など）と Kotlin 標準（`kotlinx.serialization` を含む）まで。それ以外の外部ライブラリは使わない。
- minSdk 28（Android 9）、targetSdk は最新の安定版。
- 配布は APK の直接インストール。Play ストアには出さない。

### 8.2 コーディング規約

- Kotlin 公式のコーディング規約に従う（インデント 4 スペース）。
- `domain` パッケージは Android API・ファイル・現在時刻に依存しない。「いま」は引数で受け取る。
- state は `data class` の不変データとし、更新関数は新しいオブジェクトを返す（`copy`）。
- `DevicePolicyManager` を呼ぶのは `lock` パッケージだけにする。
- UI 文言・デザインのルールは 4・5 章と同じ（タップ領域 56dp 以上、文字 20sp 以上、端末の「アニメーションを削除」設定を尊重）。

### 8.3 テスト

- `domain` は JUnit の単体テストを必ず書く。**Web 版の `tests/` と同じテストケースを移植**し、2つの実装のふるまいがずれないようにする。Web 版にケースを足したら Android 版にも足す。
- `isLocked` / `nextLockChange` は、しめきり・ねるじゅんび開始・夜中0時・一時かいほうの終わりの前後1分を境界値テストする。
- 端末の所有者まわり（F-27〜F-31）は自動テストしにくいので、エミュレーターで `adb shell dpm set-device-owner` を実行して手動確認する。確認項目は `android/README.md` にチェックリストとして置く。
