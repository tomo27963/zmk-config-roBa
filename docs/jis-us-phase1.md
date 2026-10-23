# Phase 1: JIS基準の対応表（作業メモ）

引き継ぎ文書 `roBa_JIS-USデュアル配列_Claude引き継ぎ.md` の Phase 1 に対応する調査メモ。
基準は `sync/upstream-2026`（3c6f172）で、ビルド成功を確認済み（Actions run 39984923323 / 39984925719）。

未確認の項目は「要確認」と明記している。実機の確認はまだ行っていない。

## 1. 現行キーマップのレイヤー

| 番号 | 名前 | 呼び出し（主なキー） | センサー（エンコーダ）の割り当て |
|---|---|---|---|
| 0 | default_layer | 常時有効 | `inc_dec_kp PG_UP PAGE_DOWN` |
| 1 | FUNCTION | ENTERのホールド | なし |
| 2 | NUM | SPACEのホールド | なし |
| 3 | ARROW | INT_MUHENKANのホールド（to_layer_0の仕組み） | `inc_dec_kp LC(PAGE_UP) LC(PAGE_DOWN)` |
| 4 | MOUSE | トラックボールのautomouse（`automouse-layer = <4>`） | なし |
| 5 | SCROLL | Iのホールド、トラックボールの `scroll-layers = <5>` | なし |
| 6 | layer_6 | INT_HENKANのホールド | なし（Bluetooth操作・bootloader） |

- 上流の `roBa.keymap` では、以前の `#define NUM 6` は削除されている。
- 数字で指定している箇所（`&lt 2`、`&lt_to_layer_0 6`、`&lt_to_layer_0 3` など）は、変更時に全件確認すること。

## 2. エンコーダ（左側）の現状

- 物理的には `roBa_L.overlay` の `&left_encoder`（EC11）で、右側のエンコーダは無効。
- 現状は「レイヤー0でページ送り」「ARROWレイヤーでタブ切替（Ctrl+PgUp/PgDn）」の2つだけ。

## 3. 消費者コード（曲送り・音量）の有無

- 現在のキーマップには、消費者コード（`C_NEXT`、`C_PREVIOUS`、`C_VOLUME_UP`、`C_VOLUME_DOWN`、`C_MUTE`、`C_PLAY_PAUSE` など）が**どのレイヤーにも存在しない**。
- そのため、曲送りが動かなかったのは、割り当てが誤っていたのではなく、そもそも割り当てが無かったことが原因だと思われる。

## 4. 提案（未適用・要判断）

エンコーダは1レイヤーにつき1つの動作しか持てないので、レイヤーごとに役割を分ける案。

| レイヤー | 役割 | 提案のsensor-bindings（時計回り, 反時計回り） |
|---|---|---|
| 0 default_layer | 曲送り／戻し | `&inc_dec_kp C_NEXT C_PREVIOUS`（要確認：回転方向） |
| 1 FUNCTION | 音量 | `&inc_dec_kp C_VOLUME_UP C_VOLUME_DOWN` |
| 5 SCROLL | スクロール（エンコーダ版） | `&msc SCRL_DOWN &msc SCRL_UP`（要確認：behavior名と引数の順） |

補足:
- 曲送り・音量の消費者コード名は、ZMKの `dt-bindings/zmk/keys.h`（v0.3-branch）で確認済み（`C_NEXT`、`C_PREVIOUS`、`C_VOLUME_UP`、`C_VOLUME_DOWN`）。
- スクロールの `SCRL_UP`／`SCRL_DOWN` は、`dt-bindings/zmk/pointing.h`（v0.3-branch）で定義を確認済み。
- ただし `&msc` の behavior 名は、ZMKの公式ドキュメントを取得できず未確認。ビルドで確かめる。
- `inc_dec_kp` に消費者コードを渡せるかも、ビルドで確かめる。
- SCROLLレイヤーでは、トラックボール自体がスクロールできるので、エンコーダのスクロールは補助的な役割になる。
- FUNCTIONレイヤーに入る `&lt 1 ENTER` を音量に使うのは、ENTERのタップ動作とぶつからないか要確認。

## 5. JISとUSの記号（要確認）

HIDのキー位置は同じでも、JIS（106）とUSでは印字・出力が異なる。現時点での見立ては次のとおり。

| HID（ZMKの名前） | US（期待出力） | JIS（期待出力） | 確度 |
|---|---|---|---|
| 0x2D `MINUS` | `-` | `-` | 高 |
| 0x2E `EQUAL` | `=` | `^` | 高 |
| 0x2F `LEFT_BRACKET` | `[` | `@` | 高 |
| 0x30 `RIGHT_BRACKET` | `]` | `[` | 高 |
| 0x31 `BACKSLASH` | `\` | `]` | 要確認 |
| 0x32 `NON_US_HASH` | なし | `]`（要確認） | 要確認 |
| 0x89 `INTERNATIONAL_3` | なし | `\`／`¥`（要確認） | 要確認 |

- 特に `NUM` レイヤーの `&kp EQUAL`（JISでは `^`）、`&kp LEFT_BRACKET`、`&kp RIGHT_BRACKET`、`&kp BACKSLASH` は、JIS解釈とUS解釈で出力が変わるため、Phase 1 の比較対象にする。
- 実際の出力は、Windowsの設定（JIS／US）と、ULE4JISの状態によって変わる。

## 6. 未確認・次の作業

- 上記の表の「要確認」を、実機またはビルド結果で埋める。
- 4の提案を、USモード用の専用レイヤーと合わせて設計する。
- 2の割り当てを変更すると、レイヤー番号の参照（`&trackball` の `automouse-layer`、`scroll-layers` など）に影響しないか、Phase 2 で確認する。
