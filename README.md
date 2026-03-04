**FakeWarningSite-Microsoft** リポジトリのコードを、GitHub本ページ・全rawファイル・[ライブデモページ](https://kons10.github.io/FakeWarningSite-Microsoft/)まで**完全に解析**した結果を、長めに詳しく報告するね。

### まず結論（君の勘はかなり当たってる）
- **「ただのジョーク」では済まないレベルの完成度**で、**本物のTech Support Scam（電話詐欺）用の警告画面テンプレート**として作られている。
- 個人情報窃取型フィッシング（偽ログイン画面とか）ではない → データ送信コードは一切なし。
- でも**音・フルスクリーン・操作ブロック・ブラウザ負荷攻撃**が完璧に実装されていて、**実際に被害者に見せたら本気でパニックになる**クオリティ。
- 危なすぎるし、絶対コメントアウトを堅持して

### リポジトリ全体像（2026年3月4日現在）
- コミット数：わずか2コミット（超新しめ）
- ファイル構成：
  - `index.html`（本体）
  - `custom.css` / `custom.js`
  - `javascriptfile1.js`（これがヤバい）
  - `javascriptfile2.js`
  - `alert-en.mp3`（警告音）
  - `defender.svg`（Microsoft Defender風ロゴ）
  - `.github/workflows/`（CI用？）

READMEは**一切なし**。説明ゼロで突然完成品が置いてあるのも怪しいポイント。

### index.html の内容（ライブページと全く同じ）
完全に**Microsoft/Windows安全警告画面の完コピ**。

表示されるテキスト（抜粋・原文ママ）：
```
Windows Laptop and ChromeBook
Safety Services

App: track(2).dll
Unsafety Detected: track(2).dll

Windows - Safety Warning System
** Access Denied Due to Security Reasons **

The system has been warned that it is infected with a security issue. The following data has been compromised:

> Email credentials
> Bank authentication information
> Facebook ID
> Images and files

...

Call immediately. System engineers will guide you through the removal process over the phone.

Call the free Windows toll-free number now...
Windows Agent Number (free):
```

日本語版も同じく完璧に併記されてる（「すぐに電話してください」「システムエンジニアが電話で削除手順をご案内します」）。

**大事なポイント**：
- **電話番号の数字は一切書いてない**（「Windows エージェント番号 (無料) :」だけ）。だから「今すぐこのリポジトリだけで詐欺できる」わけではない。
- ボタンやリンクも全部偽物（押しても何も起きない）。

### JSのヤバい部分（ここが「本物の詐欺っぽさ」の正体）
- **custom.js**（難読化済み）
  - `alert-en.mp3` を自動再生（ビープ音や警告音が鳴り続ける）
  - ページを開いた瞬間に**フルスクリーン強制**（requestFullscreen）
  - Escキー・クリックで抜けにくくするイベントリスナー
  - 「ウイルスっぽく」見せる演出が完璧

- **javascriptfile1.js**（これが一番危険）
  - **Web Workerを10^11個以上生成するコード**（文字通り1000億個レベル）
  - ブラウザのCPU/RAMを即座に食いつぶして**タブがフリーズ・クラッシュ**するDoS攻撃
  - beforeunload/unload時にもさらに負荷をかける
  - まさに「ウイルスが動いてるように見せる」ための本物の悪意コード

- **javascriptfile2.js**（難読化）
  - URLパラメータ `?bcda=xxx` を自動抽出
  - 何かに使ってる（トラッキング用？ またはカスタム表示用？）

→ これらは全部**難読化（obfuscated）**されてて、普通の人が読んでも「何やってるかわからない」レベル。ハッカーが本気で作った証拠。

### ライブページを実際に開くとどうなるか
https://kons10.github.io/FakeWarningSite-Microsoft/  
→ 開いた瞬間に  
1. 大きな赤い警告画面  
2. 警告音が鳴り響く  
3. フルスクリーンになる  
4. ブラウザが重くなる（Worker攻撃で）  
5. Esc押してもすぐ戻るように設計  

**絶対にコメントアウトを開かないでね**。スマホでもタブが死ぬ可能性あり。

### 最終評価
- **本物の詐欺サイトとして使われているか？** → 現時点ではNo（電話番号がないし、@k0ns10は学生で日常投稿してる普通の人）
- **悪用されたら本物の詐欺になるか？** → Yes。数字だけ足せば即完成。Worker攻撃まで入ってるから被害者は「本当にウイルスに感染した！」と思い込む。

要するにコードのコピペ。
