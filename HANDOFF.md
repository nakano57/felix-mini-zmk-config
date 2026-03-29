# Felix Mini ZMK Config — 引き継ぎドキュメント

## プロジェクト概要

- Helix ベースの無線分割キーボード「Felix」の ZMK ファームウェア設定
- 製造元: beekeeb
- コントローラ: nice!nano v2 (nRF52840)
- ディスプレイ: SSD1306 OLED (128×32) または nice!view e-paper
- RGB: WS2812 × 10個/片側（現在無効）
- 元リポジトリ: https://github.com/Eloy98/zmk-for-felix.git
- 新リポジトリ: https://github.com/nakano57/felix-mini-zmk-config.git

## 今回の変更内容

### 目的
物理レイアウトを 5×7×2（64キー）から 4×6+1 per half（50キー）に変更。
最下行（Row 4）を物理的に外したキーボードに合わせてファームウェアを修正。

### 変更方針
Row 4 を削除するだけで 4×6+1 が実現可能と分析で確定。列の変更は不要。
- Row 0–2: 各6列 × 2半分 = 12キー/行
- Row 3: 6列 + 1サムキー × 2半分 = 14キー（サムキーは col 6/7）
- Row 4: 削除（14キー分）

### 変更ファイル（3つ）

| ファイル | 変更内容 |
|---------|---------|
| `boards/shields/felix/felix.dtsi` | `rows = <5>` → `<4>`、Row4 の RC 行削除、`&pro_micro 8` GPIO 削除 |
| `boards/shields/felix/felix_physical.dtsi` | y=400 の14キーエントリ削除、posmap `5row` → `4row` |
| `config/felix.keymap` | 全面書き換え（新レイヤー構成、新ビヘイビア） |

### 変更不要だったファイル

| ファイル | 理由 |
|---------|------|
| `felix_left.overlay` | サム列（col 6）の GPIO は Row 3 で引き続き必要 |
| `felix_right.overlay` | 同上（col 7、col-offset=7 も維持） |
| `build.yaml` | シールド名に依存するだけでキー数に依存しない |
| `.github/workflows/build.yml` | 同上 |
| `config/felix.conf` | 設定値に変更なし |

## キーマップ詳細

### Layer 0: DEFAULT（50キー）

```
左半分:                                              右半分:
ESC     Q      W      E      R      T               Y      U      I      O      P      BKSP
TAB     A      S      D      F      G               H      J      K      L      ;:     '"
SHIFT   Z      X      C      V      B               N      M      ,<     .>     ↑      /?
CTRL    OPT    CMD    FN     ≡/英数  SPC    [{       ]}     SPC    0/かな  ENTER  ←      ↓      →
```

### Layer 1: SYMBOL（0/かな長押しで有効 — 左半分の記号）

```
ESC     !      @      #      $      %               trans  trans  trans  trans  trans  trans
TAB     ^      &      *      (      )               trans  trans  trans  trans  trans  trans
SHIFT   Z      X      C      V      B               trans  trans  trans  trans  trans  trans
CTRL    OPT    CMD    FN     ≡/英数  SPC    [{       trans  trans  trans  trans  trans  trans  trans
```

右半分は全て transparent（Layer 0 がそのまま透過）。

### Layer 2: NUMBER（≡/英数長押しで有効 — 右半分の数字）

```
trans   trans  trans  trans  trans  trans            1      2      3      _      +      BKSP
trans   trans  trans  trans  trans  trans            4      5      6      -      =      "
trans   trans  trans  trans  trans  trans            7      8      9      *      ↑      '
trans   trans  trans  trans  trans  trans  trans     ]      SPC    0      ENTER  ←      ↓      →
```

左半分は全て transparent。
**重要**: 0/かな の位置は Layer 2 時は通常の `0` キーになる。

### Layer 3: SYS（BT/Bootloader — 既存を4行に縮小して維持）

```
BT_CLR  BT0    BT1    BT2    BT3    BT4             CAPS   NUMLK  SCRLK  ---    STUDIO OUT_TOG
---     ---    ---    ---    ---    ---              none   none   none   none   none   none
---     ---    ---    ---    ---    ---              none   none   none   none   none   ---
BOOT    ---    ---    ---    ---    ---    ---       ---    ---    ---    ---    ---    ---    BOOT
```

### Layer 4–7: extra1–4（ZMK Studio 用予約、全キー reserved）

## ビヘイビア定義

### lt_eisuu（≡/英数キー）
- タイプ: `zmk,behavior-hold-tap`
- タップ: `LANG2`（Mac の英数）
- ホールド: `mo NUMBER`（Layer 2 有効化）
- フレーバー: balanced
- タッピングターム: 200ms
- クイックタップ: 125ms

### lt_kana（0/かなキー）
- タイプ: `zmk,behavior-hold-tap`
- タップ: `LANG1`（Mac のかな）
- ホールド: `mo SYMBOL`（Layer 1 有効化）
- フレーバー: balanced
- タッピングターム: 200ms
- クイックタップ: 125ms

### サムキー
- 左サム: `&kp LBKT` → `[`（Shift 時 `{`）
- 右サム: `&kp RBKT` → `]`（Shift 時 `}`）

### レイヤー切替ロジック（クロス切替）
- 左手の ≡/英数 長押し → 右半分の数字レイヤー (Layer 2) 有効化
- 右手の 0/かな 長押し → 左半分の記号レイヤー (Layer 1) 有効化

### 廃止した機能
- ホームロウモッド（`tpmt` ビヘイビア）を完全削除

## マトリクス構造（参考）

### トランスフォーム（変更後）
```
Row 0: RC(0,0)–RC(0,5)  |  RC(0,8)–RC(0,13)     12キー
Row 1: RC(1,0)–RC(1,5)  |  RC(1,8)–RC(1,13)     12キー
Row 2: RC(2,0)–RC(2,5)  |  RC(2,8)–RC(2,13)     12キー
Row 3: RC(3,0)–RC(3,6)  |  RC(3,7)–RC(3,13)     14キー（col 6/7 = サム）
                                                   合計 50キー
```

### GPIO ピン
- Row 0–3: pro_micro 4, 5, 6, 7（pin 8 = Row 4 を削除）
- Col（左）: pro_micro 21, 20, 19, 18, 15, 14, 16（7本、変更なし）
- Col（右）: pro_micro 16, 14, 15, 18, 19, 20, 21（7本、col-offset=7、変更なし）

## ビルド構成

### ビルドターゲット（build.yaml — 変更なし）
| Board | Shield | 備考 |
|-------|--------|------|
| nice_nano_v2 | felix_left nice_oled | ZMK Studio 有効 |
| nice_nano_v2 | felix_right nice_oled | — |
| nice_nano_v2 | settings_reset | 設定リセット用 |
| nice_nano_v2 | felix_left nice_view_adapter nice_epaper | ZMK Studio 有効 |
| nice_nano_v2 | felix_right nice_view_adapter nice_epaper | — |

### 依存関係（west.yml — 変更なし）
- ZMK: `zmkfirmware/zmk` v0.2
- OLED モジュール: `mctechnology17/zmk-nice-oled` (main)

### 設定（config/felix.conf — 変更なし）
- ディスプレイ有効、カスタムステータス画面
- ディープスリープ: 50分後
- BLE TX 出力: +8dBm
- ZMK Studio: 有効（ロック無効化）

## 別 PC での push 手順

```bash
# zip を展開
unzip felix-mini-zmk-config.zip
cd zmk-for-felix

# git 初期化
git init
git branch -M main
git add .
git commit -m "Change layout from 5x7x2 (64 keys) to 4x6+1 (50 keys)

Remove row 4 from matrix transform, physical layout, and kscan GPIO.
Rewrite keymap with new layer structure:
- DEFAULT: QWERTY with thumb keys [ ]
- SYMBOL: shifted symbols on left half (via kana hold)
- NUMBER: numpad on right half (via eisuu hold)
- SYS: BT/bootloader (unchanged)
Replace home row mods with lt_eisuu/lt_kana hold-tap behaviors."

# リモート設定 & force push
git remote add origin git@github.com:nakano57/felix-mini-zmk-config.git
git push origin main --force
```

## ビルド確認

push 後、GitHub Actions が自動でビルドを開始:
https://github.com/nakano57/felix-mini-zmk-config/actions

成功したビルドから `firmware` アーティファクトをダウンロードし、nice!nano v2 に書き込み。

## 今後の調整が必要な箇所

1. **Layer 1 (SYMBOL) の右半分**: 現在 transparent — 必要に応じて記号を追加
2. **Layer 2 (NUMBER) の左半分**: 現在 transparent — 必要に応じてファンクションキー等を追加
3. **Layer 3 (SYS) のアクセス方法**: 現在未定義 — コンボやレイヤー同時押し等で設定が必要
4. **keymap-drawer**: `keymap-drawer/felix.yaml` と `felix.svg` は旧レイアウト — push 後の CI で自動再生成される
5. **felix.json**: QMK 互換の物理レイアウト JSON — keymap-drawer 用に 50 キーに更新が必要かもしれない
6. **シールドデフォルトキーマップ**: `boards/shields/felix/felix.keymap` は旧 64 キーのまま — ZMK Studio 使用時に問題になる可能性あり（config/felix.keymap が優先されるので通常は問題なし）
