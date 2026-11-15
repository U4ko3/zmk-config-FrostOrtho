# ZMK v0.4 移行と DYA Studio からのマウスジェスチャー編集

[zmk-config-moNa2-v2 (DYA Studio 対応版)](https://github.com/shakushakupanda/zmk-config-moNa2-v2) を参考に、
ZMK を v0.3 系 (`cormoran/zmk:v0.3-branch+dya`, Zephyr 3.5) から
v0.4 系 (`cormoran/zmk:main+dya`, Zephyr 4.1) へ移行しました。

## 既存ファームウェアから更新するとき

ZMK v0.4 系への移行により、以前のキーマップ・トラックボール設定・BLE 情報は引き継がれません。

1. OS 側で既存の FrostOrtho の Bluetooth 登録を削除する
2. 左右それぞれに `settings_reset` を書き込む
3. 左右それぞれに通常のファームウェア（`FrostOrtho_R` / `FrostOrtho_L`）を書き込む
4. Bluetooth を再ペアリングする

左右で異なるバージョンのファームウェアを使うと分割通信が正常に動作しないことがあります。必ず同じビルドを書き込んでください。

> ビルド成果物の名前は `build.yaml` の `artifact-name` により
> `FrostOrtho_R.uf2` / `FrostOrtho_L.uf2` / `settings_reset.uf2` になります
> （以前の `*-seeeduino_xiao_ble-zmk.uf2` から変わります）。

## v0.4 移行で変えた点

| 変更 | 対応箇所 |
| --- | --- |
| ZMK 本体を `cormoran/zmk:main+dya` (ZMK v0.4 / Zephyr 4.1) に変更 | `config/west.yml` |
| DYA Studio モジュールを `zmk-v0.3.0.0` → `main` に変更、`zmk-feature-custom-settings` を追加 | `config/west.yml` |
| ボード名 `seeeduino_xiao_ble` → `xiao_ble/nrf52840/zmk` | `build.yaml`, `FrostOrtho.zmk.yml` |
| GitHub Actions の再利用ワークフローを `@v0.3.0` → `@main` | `.github/workflows/build.yml` |
| `zmk-rgbled-widget` を `v0.3.0` → `main` | `config/west.yml` |
| バッテリー履歴モジュール（無効にしていたもの）を削除 | `config/west.yml`, 各 overlay / conf |
| 静的コンボ (`zmk,combos`) を DYA Studio で編集できる `runtime_combo_defaults` へ移行 | `config/FrostOrtho.keymap` |
| ランタイムマクロ `&rmacro <0〜7>` を追加 | `config/FrostOrtho.keymap`, `FrostOrtho_R.conf` |
| DYA Studio 診断タブ（キースイッチ・スタック使用量・デバイス情報・ウォッチドッグなど）を有効化 | `config/west.yml`, `FrostOrtho_R.conf`, `FrostOrtho_L.conf` |
| Zephyr 4.1 での BLE コントローラの assert 再起動を回避 | `FrostOrtho_R.conf` |

## マウスジェスチャーを DYA Studio から編集する

マウスジェスチャーのエンジンを `kot149/zmk-mouse-gesture` から
[shakushakupanda/zmk-mouse-gesture](https://github.com/shakushakupanda/zmk-mouse-gesture)（`suppress-cursor-while-active` ブランチ）へ変更し、
[zmk-module-mouse-gesture-rpc](https://github.com/shakushakupanda/zmk-module-mouse-gesture-rpc) を追加しました。
これにより、ファームウェアを再ビルドせずにジェスチャーを変更できます。

### 使い方

1. 右手側を USB ケーブルで PC に接続する
2. [DYA Studio](https://studio.dya.cormoran.works/) を開いて接続し、`&studio_unlock`（BLUETOOTH レイヤー）でアンロックする
3. 左サイドバーの **Subsystems** ページを開くと `cormoran__mouse_gesture` が表示されるので、リンクから Mouse Gesture の Web UI を開く
   （直接 <https://shakushakupanda.github.io/zmk-module-mouse-gesture-rpc/> を開いて Connect してもよい）
4. Web UI でジェスチャーの方向とキーを設定して保存する

保存した内容は右手側の Flash に保存され、再起動後も保持されます。
`config/FrostOrtho.keymap` の `&zip_mouse_gesture` に書いたジェスチャーは **初期値（セット 0）** で、
Web UI の **Reset to Defaults** でこの値に戻ります。

Web UI ではストロークサイズ・アイドルタイムアウト・クールダウンや、慣性スクロール（既定は無効）も変更できます。

### ジェスチャーキー

| キー | 動作 |
| --- | --- |
| `&mg_kp <セット> <キー>` | 長押しでジェスチャー（指定セット）、タップでキー入力。初期キーマップでは MOUSE レイヤーの T 位置に `&mg_kp 0 T` |
| `&mg_set <セット>` | 押している間、指定セットのジェスチャーを有効にする |
| `&mg_set_0` / `&mg_set_1` / `&mg_set_2` | 上と同じ（パラメータなし版）。DYA Studio のキーマップエディタでは「Mouse Gesture Key 0/1/2」として選べる |

ジェスチャーセットは 3 つ（`CONFIG_ZMK_MOUSE_GESTURE_RPC_NUM_SETS=3`）あり、
押したキーに応じて別々のジェスチャーを割り当てられます。
ジェスチャーキーを押している間はカーソルが止まります（旧設定の `suppress-movement` 相当）。
