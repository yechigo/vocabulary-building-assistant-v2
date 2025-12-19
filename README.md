# 英語学習アプリ v2 - ロック画面対応版

スマホのロック画面でも音声再生が継続する、進化版の英語学習アプリです。

## 🆕 v2 の新機能

### ✅ ロック画面でも音声再生が継続
- スマホをロックしても音声が止まりません
- Media Session API により、ロック画面に再生コントロールが表示されます
- 通勤・通学中に画面を見なくても学習できます

### ✅ 音声キャッシュ機能
- 一度再生した音声はブラウザにキャッシュされます
- 2回目以降は **API呼び出しなし** で即座に再生
- オフラインでも（キャッシュ済みなら）再生可能

### ✅ 高品質な音声
- Google Cloud Text-to-Speech API を使用
- Standard voices または WaveNet voices から選択可能
- 自然で聞き取りやすい音声

## 📋 初回設定（全体で15分程度）

### ステップ1: Google Cloud TTS API の設定（10分）

#### 1. Google Cloud Console にアクセス
https://console.cloud.google.com/

#### 2. 新しいプロジェクトを作成
1. 画面上部の「プロジェクトの選択」をクリック
2. 「新しいプロジェクト」をクリック
3. プロジェクト名を入力（例: `english-learning-app`）
4. 「作成」をクリック

#### 3. Text-to-Speech API を有効化
1. 左側メニューから「APIとサービス」→「ライブラリ」
2. 検索ボックスに「Text-to-Speech」と入力
3. 「Cloud Text-to-Speech API」をクリック
4. 「有効にする」をクリック

#### 4. APIキーを作成
1. 左側メニューから「APIとサービス」→「認証情報」
2. 「認証情報を作成」→「APIキー」をクリック
3. APIキーが表示されます（例: `AIzaSyD...`）
4. **このキーをコピーして保存してください**

#### 5. APIキーを制限（推奨）
1. 作成したAPIキーの右側にある「編集」をクリック
2. 「APIの制限」セクションで「キーを制限」を選択
3. 「Cloud Text-to-Speech API」のみにチェック
4. 「保存」をクリック

### ステップ2: アプリにAPIキーを設定（1分）

1. `english-learning-app.html` を開く
2. 「⚙️ Google Cloud TTS API 設定」をクリックして展開
3. コピーしたAPIキーを貼り付け
4. 「API キーを保存」をクリック

### ステップ3: Google Apps Script の設定（既存のまま）

v1 で設定済みの Google Apps Script をそのまま使用できます。

## 💰 料金について

### Google Cloud TTS API の無料枠
- **毎月 400万文字まで無料**（Standard voices）
- **毎月 100万文字まで無料**（WaveNet voices）

### 実質的なコスト
**個人利用なら完全無料で使えます:**
- データ100項目（1項目80文字）= 8,000文字
- 毎日10回繰り返しても: 8,000 × 10 × 30日 = 240万文字/月 → **無料範囲内**

### キャッシュによるコスト削減
- 初回のみAPI呼び出し、2回目以降はキャッシュから再生
- データ100項目なら、初回のみ8,000文字使用 → **以降は無料**

## 📱 日常の使い方

### 1. アプリを開く
- PCの場合: `english-learning-app.html` をダブルクリック
- スマホの場合: ホーム画面のアイコンをタップ

### 2. データを読み込み
「最新データを読み込み」ボタンをクリック
（自動的に Google Sheets から最新データを取得）

### 3. 学習開始
「学習を開始」ボタンをクリック

### 4. ロック画面での使用
1. 学習開始後、スマホをロック
2. ロック画面に再生コントロールが表示されます
3. 一時停止・再開はロック画面から操作可能

## 🔧 機能

### 基本機能（v1 から継承）
- ✅ Google Sheets からデータ自動取得
- ✅ 和訳→英文の順に読み上げ
- ✅ 待機時間のカスタマイズ（1〜30秒）
- ✅ 繰り返し回数の設定（1〜10回）
- ✅ シャッフルモード
- ✅ 一時停止・再開

### 新機能（v2）
- ✅ ロック画面での再生継続
- ✅ 音声キャッシュ
- ✅ 高品質な音声合成
- ✅ キャッシュ状態の表示
- ✅ キャッシュのクリア機能

## 🛠️ トラブルシューティング

### 「APIキーが設定されていません」と表示される

**解決方法:**
1. 「⚙️ Google Cloud TTS API 設定」を開く
2. APIキーを入力して「API キーを保存」をクリック

### 「TTS API Error」と表示される

**原因と解決方法:**
1. **APIキーが無効**: Google Cloud Console でAPIキーを確認
2. **API未有効化**: Cloud Text-to-Speech API が有効になっているか確認
3. **無料枠超過**: Google Cloud Console で使用量を確認（まれ）

### ロック画面で音声が止まる

**iOSの場合:**
- Safari を使用してください（Chrome は制限あり）
- 「ホーム画面に追加」でPWAとして使用すると効果的

**Androidの場合:**
- Chrome または Edge を使用してください
- 通知権限を許可してください

### 音声がキャッシュされない

**解決方法:**
1. ブラウザのプライベートモードを使用していないか確認
2. ブラウザのストレージ設定を確認
3. 「キャッシュをクリア」→ 再度学習開始で再キャッシュ

## 📊 v1 との比較

| 機能 | v1（旧版） | v2（新版） |
|------|----------|----------|
| ロック画面での再生 | ❌ 停止する | ✅ 継続する |
| 音声品質 | △ ブラウザ標準 | ✅ Google TTS（高品質） |
| オフライン再生 | ❌ 不可 | ✅ 可能（キャッシュ後） |
| 初回設定 | 簡単 | やや複雑（API設定必要） |
| 費用 | 完全無料 | 実質無料（無料枠内） |
| 再生速度 | ✅ 同じ | ✅ 同じ |
| データ自動更新 | ✅ 対応 | ✅ 対応 |

## ⚙️ 詳細設定

### 音声の種類を変更

`english-learning-app.html` の 435〜437行目を編集:

```javascript
// Standard voices（無料枠が大きい）
const voiceConfig = lang === 'ja-JP'
    ? { languageCode: 'ja-JP', name: 'ja-JP-Standard-A' }
    : { languageCode: 'en-US', name: 'en-US-Standard-C' };

// WaveNet voices（高品質だが無料枠が小さい）
const voiceConfig = lang === 'ja-JP'
    ? { languageCode: 'ja-JP', name: 'ja-JP-Wavenet-A' }
    : { languageCode: 'en-US', name: 'en-US-Wavenet-C' };
```

利用可能な音声一覧:
https://cloud.google.com/text-to-speech/docs/voices

### 読み上げ速度を変更

442行目の `speakingRate` を編集:

```javascript
audioConfig: { audioEncoding: 'MP3', speakingRate: 0.9 }
// 0.25 〜 4.0 の範囲で設定可能（1.0 が標準速度）
```

## 🔒 セキュリティとプライバシー

### APIキーの保存場所
- ブラウザの LocalStorage に保存されます
- デバイスから外部には送信されません

### データの取り扱い
- Google Sheets のデータは Google Cloud TTS API に送信されます
- 音声はブラウザの IndexedDB にローカル保存されます
- 外部サーバーには保存されません

### APIキーの保護
- APIキーは password type の input で非表示
- Google Cloud Console で API を Text-to-Speech のみに制限推奨

## 📞 サポート

問題が解決しない場合:
1. ブラウザのコンソールを開いてエラーメッセージを確認
2. Google Cloud Console の「APIとサービス」→「ダッシュボード」で使用状況を確認
3. キャッシュをクリアして再試行

## ライセンス

このアプリは自由に使用・改変できます。
