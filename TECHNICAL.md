# 技術仕様書

英語学習アプリ v2 の技術的な詳細とメンテナンス情報

## 📐 アーキテクチャ

### システム構成

```
┌─────────────────┐
│  ユーザー        │
└────────┬────────┘
         │
    ┌────▼────┐
    │ Browser │ (Safari推奨)
    └────┬────┘
         │
    ┌────▼─────────────────────┐
    │ english-learning-app.html│
    │  (Single Page App)       │
    └───┬──────────────┬───────┘
        │              │
┌───────▼──────┐  ┌───▼──────────────┐
│ Google Apps  │  │ Google Cloud     │
│ Script       │  │ Text-to-Speech   │
│ (Data API)   │  │ API              │
└───────┬──────┘  └───┬──────────────┘
        │              │
┌───────▼──────┐       │
│ Google       │       │
│ Sheets       │       │
└──────────────┘       │
                       │
                ┌──────▼──────┐
                │ IndexedDB   │
                │ (Cache)     │
                └─────────────┘
```

### 技術スタック

- **フロントエンド**: Vanilla JavaScript (ES6+)
- **UI**: HTML5 + CSS3
- **音声API**: Google Cloud Text-to-Speech API
- **データソース**: Google Sheets via Apps Script
- **キャッシュ**: IndexedDB API
- **メディア制御**: Media Session API

## 🔧 主要コンポーネント

### 1. データ管理

#### データ取得
```javascript
async function loadDataFromAppsScript()
```
- Google Apps Script経由でSheetデータを取得
- 列指定（和訳列、英文列）に基づいてデータを抽出
- `originalData`と`learningData`に保存

#### データ構造
```javascript
{
  japanese: "和訳テキスト",
  english: "English text"
}
```

### 2. 音声生成とキャッシュ

#### 音声生成
```javascript
async function generateAudio(text, lang)
```
- Google Cloud TTS APIにリクエスト
- Base64エンコードされたMP3を取得
- Blobに変換して返却

#### キャッシュ管理
```javascript
// IndexedDB スキーマ
{
  dbName: 'AudioCacheDB',
  storeName: 'audioCache',
  key: `${lang}_${text}`,  // 言語とテキストの組み合わせ
  value: Blob               // 音声データ
}
```

**主要関数:**
- `getCachedAudio(text, lang)`: キャッシュから音声取得
- `setCachedAudio(text, lang, audioData)`: キャッシュに保存
- `getOrGenerateAudio(text, lang)`: キャッシュ優先で音声取得

### 3. 再生制御

#### 状態管理
```javascript
let currentPlaybackState = 'stopped';
// 'playing_japanese' | 'waiting' | 'playing_english' | 'stopped'

let isPlaying = false;
let currentIndex = 0;
let currentItem = null;
```

#### 再生フロー
```
playNext()
  → playAudio(japanese)
  → onJapaneseEnded()
  → setTimeout(waitTime)
  → continueAfterWait()
  → playAudio(english)
  → onEnglishEnded()
  → playNext() (次のセンテンス)
```

### 4. Media Session API

```javascript
function updateMediaSession()
```
- ロック画面にメタデータと再生コントロールを表示
- アクションハンドラー:
  - `play`: 再生/再開
  - `pause`: 一時停止
  - `previoustrack`: 前のセンテンス
  - `nexttrack`: 次のセンテンス

### 5. シャッフル機能

```javascript
let originalData = [];  // 元の順序を保持
let learningData = [];  // 実際の再生順序
```

- 再生中の切り替えに対応
- シャッフルON: 残りの未再生データをシャッフル
- シャッフルOFF: 元の順序に戻して現在位置から継続

## 🎨 UI構成

### メイン画面

```
┌──────────────────────────┐
│ 🎧 英語学習アプリ v2      │
├──────────────────────────┤
│  [現在のテキスト表示]     │
│  進捗: X / Y             │
├──────────────────────────┤
│  ⏮  ▶  ⏭               │
├──────────────────────────┤
│  ☑ シャッフル            │
│  ☑ ループ               │
│  [停止して最初に戻る]     │
├──────────────────────────┤
│  ⚙️ 設定を開く ▼        │
│  └─ [設定項目...]       │
└──────────────────────────┘
```

### 設定項目

- Google Apps Script URL
- データ範囲（開始行/終了行）
- 列指定（和訳/英文）
- 待機時間（0〜30秒）
- Google Cloud TTS APIキー
- キャッシュ情報とクリア

## 🔐 セキュリティ

### APIキーの保護

```javascript
// LocalStorageに保存
localStorage.setItem('tts_api_key', apiKey);

// password type inputで非表示
<input type="password" id="apiKey">
```

**推奨事項:**
- Google Cloud ConsoleでAPIキーを制限
- Text-to-Speech APIのみに制限
- HTTPリファラー制限（GitHub Pages URL）

### データプライバシー

- **送信先**: Google Cloud TTS APIのみ
- **保存先**: ブラウザのIndexedDB（ローカル）
- **外部送信**: なし

## 📝 コードの主要部分

### API呼び出し

**場所**: `english-learning-app.html` 435-460行目付近

```javascript
async function generateAudio(text, lang) {
    const apiKey = getApiKey();

    const voiceConfig = lang === 'ja-JP'
        ? { languageCode: 'ja-JP', name: 'ja-JP-Standard-A' }
        : { languageCode: 'en-US', name: 'en-US-Standard-C' };

    const response = await fetch(
        `https://texttospeech.googleapis.com/v1/text:synthesize?key=${apiKey}`,
        {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                input: { text },
                voice: voiceConfig,
                audioConfig: { audioEncoding: 'MP3', speakingRate: 0.9 }
            })
        }
    );
}
```

### 音声の種類変更

**Standard voices（無料枠大）:**
```javascript
const voiceConfig = lang === 'ja-JP'
    ? { languageCode: 'ja-JP', name: 'ja-JP-Standard-A' }
    : { languageCode: 'en-US', name: 'en-US-Standard-C' };
```

**WaveNet voices（高品質）:**
```javascript
const voiceConfig = lang === 'ja-JP'
    ? { languageCode: 'ja-JP', name: 'ja-JP-Wavenet-A' }
    : { languageCode: 'en-US', name: 'en-US-Wavenet-C' };
```

利用可能な音声一覧:
https://cloud.google.com/text-to-speech/docs/voices

### 読み上げ速度変更

**場所**: 同じく `generateAudio` 関数内

```javascript
audioConfig: {
    audioEncoding: 'MP3',
    speakingRate: 0.9  // 0.25〜4.0（1.0が標準）
}
```

## 🐛 既知の制限事項

### ロック画面再生

**動作する環境:**
- iOS Safari（ブラウザモード）
- Android Chrome/Edge

**動作しない/制限がある環境:**
- iOS Chrome（Media Session API制限）
- iOS PWAモード（ホーム画面に追加）
  - 初回再生は可能
  - ロック画面からの再生再開に制限
  - バックグラウンドでのsetTimeout動作が不安定

**理由:**
- iOSのバックグラウンド制約
- PWAモードでのsetTimeoutの遅延/停止
- WebKitの制限

### 回避策

ブラウザモード（Safari）での使用を推奨

## 🔄 メンテナンスガイド

### バージョン管理

GitHubで管理:
- リポジトリ: `vocabulary-building-assistant-v2`
- GitHub Pages: 自動デプロイ
- メインブランチ: `main`

### デプロイ

```bash
git add .
git commit -m "メッセージ"
git push
```

GitHub Pagesに自動反映（1〜2分）

### バックアップ推奨

- `english-learning-app.html`: メインファイル
- Google Apps Script: コードをローカル保存
- Google Sheets: 定期的にエクスポート

## 🧪 テスト項目

### 基本機能
- [ ] データ読み込み
- [ ] 再生/一時停止
- [ ] 前へ/次へ
- [ ] シャッフルON/OFF
- [ ] ループON/OFF
- [ ] 停止して最初に戻る

### 音声機能
- [ ] 日本語音声再生
- [ ] 英語音声再生
- [ ] 待機時間動作
- [ ] 音声キャッシュ保存
- [ ] キャッシュからの再生

### データ連携
- [ ] Google Sheets取得
- [ ] 列指定の変更
- [ ] 範囲指定の変更

### ブラウザ互換性
- [ ] Safari (iOS/macOS)
- [ ] Chrome (Android/Windows/Mac)
- [ ] Edge

## 📊 パフォーマンス

### 最適化ポイント

1. **音声キャッシュ**: 2回目以降の再生は即座
2. **API呼び出し削減**: キャッシュヒット率を高める
3. **遅延読み込み**: 必要な音声のみ生成

### メトリクス

- 音声生成時間: 1〜3秒（ネットワーク依存）
- キャッシュからの再生: ほぼ即座（<100ms）
- データ読み込み: 1〜2秒（Google Apps Script経由）

## 🔮 将来の拡張案

- [ ] オフラインモード（Service Worker）
- [ ] 音声速度調整UI
- [ ] 学習進捗の保存
- [ ] 複数の学習セット管理
- [ ] 統計情報表示

## 📚 参考リンク

- [Google Cloud Text-to-Speech API](https://cloud.google.com/text-to-speech)
- [Media Session API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Session_API)
- [IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
- [Google Apps Script](https://developers.google.com/apps-script)
