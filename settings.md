# 英検1級 熟語マスター ― プロジェクト設定 & 仕様メモ

別プロジェクトでも似たような単語学習アプリを作るときの参考用メモ。

## 概要

- **目的**: 英検1級 熟語編 (No.2101–2400 / 300語) を、クイズ・暗記カード・検索・復習で覚えるためのオフラインWebアプリ
- **形式**: 単一の HTML ファイル（外部依存ゼロ・file:// で動く）
- **デプロイ**: ブラウザでファイルをダブルクリックするだけ。サーバー不要

## ファイル構成

```
英検一級　熟語/
├── 英検1級_熟語マスター.html       ← 本体（クイズ＋暗記カード統合版）
├── 英検1級_熟語_2101-2400.md       ← 熟語一覧表（Notion 用 md table）
├── eiken.png                      ← 英検ロゴ画像（favicon元）
├── settings.md                    ← この文書
├── (旧版) 英検1級_熟語クイズ.html
└── (旧版) 英検1級_暗記カード.html
```

## 主要機能

### 🎯 クイズタブ
- **100問×3章** (2101–2200 / 2201–2300 / 2301–2400)
- 4択。正解以外はランダムで他の熟語から選出
- 解答後にフィードバック表示：
  - 例文（英・和訳併記）
  - 解説（コアイメージ・ニュアンス）
  - 類義語の使い分け
  - 構成単語の Weblio 風解説（例: brim over with の brim）
- 結果画面：スコア・コメント（メダル）・全20問のレビュー
- **間違えた熟語を localStorage に蓄積** → 復習モード
- **「自信ない」マーク** (解答前にトグル可) → 別途復習可能
- **クイズ履歴**（日時・章・スコア）最大200件
- **バックアップ／復元**（JSON エクスポート・インポート）
- 前後ナビゲーション（解答済みの状態は復元される）

### 📚 暗記カードタブ
- **100語×3章** で章割もクイズと統一
- 表：熟語 → 裏：訳・例文・解説・類義語・単語解説
- 「習得済み」マークで進捗管理
- **未習得のみ表示モード**（章単位 / 全章まとめ）
- **検索機能**（部分一致：熟語・訳・例文・解説・類義語・例文和訳すべて対象）
- マッチした単語をハイライト表示

### キーボードショートカット
クイズ：
| キー | 動作 |
|---|---|
| `1`–`4` | 選択肢選択 |
| `←` | 前の問題 |
| `→` / `Enter` | 次の問題 |
| `0` | 自信ないトグル |

暗記カード：
| キー | 動作 |
|---|---|
| `←` `→` | カード移動 |
| `Space` / `Enter` | 裏返す |
| `0` | 習得済みトグル |
| `Esc` | 検索クリア |

## データ構造

`full_data.json` に300エントリ。各エントリ：

```json
{
  "id": 2101,
  "idiom": "abide by",
  "meaning": "～に従う（= live by）",
  "ex_en": "The boy was warned that...",
  "ex_ja": "その男の子は学校の規則に従わなければ...",
  "core": "規則・契約・決定など公的なルールを「自分から進んで守る」改まった言い方。",
  "syn": "follow（一般的）／ comply with（要求）／ observe（伝統）...",
  "word_key": "abide",
  "word_def": "(動) (古風) 留まる/我慢する。abide by の形で『〜に従う』。【類】tolerate, endure"
}
```

## localStorage キー（同一 origin で共有）

| キー | 内容 | 形式 |
|---|---|---|
| `eiken1_wrong` | 間違えた熟語ID | `[2101, 2150, ...]` |
| `eiken1_uncertain` | 自信ない熟語ID | `[2101, ...]` |
| `eiken1_known` | 習得済み熟語ID（カード共有） | `[2101, ...]` |
| `eiken1_history` | クイズ履歴（最大200件） | `[{date, label, score, total}, ...]` |
| `eiken1_tab` | 最後に開いていたタブ | `"quiz"` or `"flash"` |

## ビルド構成（開発側）

ソースは複数ファイルに分け、Python スクリプトで結合して単一 HTML を生成：

```
outputs/
├── full_data.json       ← 300語のマスターデータ
├── common.css           ← 共通CSS（カラーパレット・ボタンなど）
├── quiz_extra.css       ← クイズ専用CSS
├── flash_extra.css      ← 暗記カード専用CSS
├── quiz_body.html       ← クイズ画面のHTML（5画面）
├── flash_body.html      ← 暗記カード画面のHTML（2画面）
├── quiz.js              ← クイズロジック
├── flash.js             ← 暗記カードロジック
├── flash_search.js      ← 検索機能
├── eiken_logo.png       ← ロゴ画像
├── build_quiz.py        ← クイズ単独ビルド（旧版）
├── build_flash.py       ← 暗記カード単独ビルド（旧版）
└── build_combined.py    ← 統合版ビルド（現役）
```

`build_combined.py` の流れ：
1. 各部品を読み込み
2. `__DATA__`, `__LOGO__` を実データで置換
3. CSS / JS / HTML を結合し1つの HTML を出力
4. PNG は base64 で埋め込み (`data:image/png;base64,...`)

## 設計上の工夫

### 単一HTML化のために
- すべての CSS / JS / 画像（base64）を1ファイルに埋め込み
- 外部依存 (CDN, fetch) は使わない → file:// でも完全動作
- Tab 切替で2アプリを統合
- 衝突する識別子は flash 側に `flash` プレフィックス、`flashState`, `flashRenderChapters` 等

### 名前空間の整理
- `state` (quiz) ⇄ `flashState` (flash)
- `CHAPTERS` ⇄ `FLASH_CHAPTERS`
- `renderChapters` ⇄ `flashRenderChapters`
- 共通DOM ID（`start-screen` 等）は flash 側を `flash-start-screen` にリネーム

### UX 微調整
- スクロールエリアは `max-height: calc(100vh - 280px)` で画面いっぱい伸びる
- `0` キーは右手の小指から近く、矢印キーと併用しやすい
- 「自信ない」は解答前から押せる（解答後だけ表示にしない）
- 検索：日本語と英語両対応（meaning, ex_ja, core を raw（大文字小文字保持）でも検索）

## カラーパレット

```css
--bg: #0f172a            /* 暗い青 */
--card: #1e293b          /* カード背景 */
--card2: #334155         /* やや明るいカード */
--primary: #6366f1       /* 紫（メインボタン） */
--eiken-red: #cc0000     /* 英検赤（強調用） */
--success: #10b981       /* 緑（正解） */
--danger: #ef4444        /* 赤（不正解） */
--gold: #fbbf24          /* 金（強調・例文ラベル） */
--text: #f1f5f9
--text-dim: #94a3b8
```

背景は `linear-gradient(135deg, #0f172a 0%, #1e1b4b 100%)`。

## レスポンシブ

- 横幅 max 720px のコンテナ
- 480px 以下でフォントサイズ・パディング縮小
- スマホで開いてもサクサク

## 別プロジェクトに転用するときのチェックリスト

1. **データ構造**: 上記 JSON の `id, idiom, meaning, ex_en, ex_ja, core, syn, word_key, word_def` を変えるだけで他の単語集にも応用可
2. **章割り**: `CHAPTERS` / `FLASH_CHAPTERS` の配列だけ変更
3. **localStorage キー**: 別プロジェクトでは `eiken1_*` を別 prefix に変える（同じファイル名で衝突しないため）
4. **ロゴ**: `eiken.png` を差し替え + `build_combined.py` の base64 化部分を再実行
5. **章数の変更**: クイズ・カード両方の `CHAPTERS` を合わせる
6. **キーボードショートカット**: 学習スタイルに応じて `0` の位置などを調整

## OCR 履歴ノート

- LINE_ALBUM の写真21枚から OCR 処理。傾き・小さい文字に苦戦
- Tesseract に日本語フォントなく Vision モデル使用
- 結果は `英検1級_熟語_2101-2400.md` に手動転記・整形
- 各熟語に `core`, `syn`, `word_def` を手動付与（Weblio 風解説）

## 既知の小さな注意点

- localStorage は file:// origin で動くが、ファイル名／パスを変えると消えることがある
  → 必ず**バックアップ機能（💾）で JSON 出力**してから移動
- 一部熟語の訳語は古めの日本語（例: 「とりなす」）。現代訳を追加修正する場合は `full_data.json` の `meaning` を直接編集 → `python3 build_combined.py` で再ビルド
