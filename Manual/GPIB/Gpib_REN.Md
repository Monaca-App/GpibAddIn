# コマンドリファレンス：Gpib_REN

## 概要

**Gpib_REN** は、オープン中の全 GPIB ボードの **REN（Remote Enable）単一制御ライン** をアサート（有効化: リモート制御許可）またはデアサート（無効化: 全機器ローカル制御復帰・LLO 解除）するバス制御コマンドです。

> 💡 **補足**: IEEE 488 規格において、測定器を PC からのリモート制御状態に移行させるための前提となるハードウェア制御ラインを制御します。`"ON"` / `"1"` 等で有効化、`"OFF"` / `"0"` 等で無効化でき、[`Gpib_LLO`](./Gpib_LLO.Md) によるローカルロックアウトの一括解除にも使用できます。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Gpib_REN, "[State: 1 / 0 / ON / OFF / TRUE / FALSE / ENABLE / DISABLE]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `State` | 任意 | `"1"` / `"0"` / `"ON"` / `"OFF"` / `"TRUE"` / `"FALSE"` / `"ENABLE"` / `"DISABLE"` または `0` 以上の整数（既定値: `"1"`［ON］） | `"1"` / `"ON"` / `"TRUE"` / `"ENABLE"`（大文字・小文字非区別、および `1` 以上の整数）または省略時は REN ラインを ON（アサート）し、`"0"` / `"OFF"` / `"FALSE"` / `"DISABLE"` 指定時は OFF（デアサート）します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: SCPI コマンド等、引数文字列自体にカンマを含める場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `REN`, `GpibREN`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマ（`,`）から開始してください（行頭スペースのみでは認識されません）。なお、`;` で始まる行やクォート外の `;` 以降はコメントとして自動除去されます。

```csv
COMMAND, "SAMPLE_GPIB_REN", ""
,SEQUENCE, 010, Gpib_Open, 0
; 1. RENラインをアサート(ON)してリモート制御を有効化
,SEQUENCE, 020, Gpib_REN, "ON"
,SEQUENCE, 030, Gpib_Send, "VOLT 12.0"
; 2. 試験終了時にRENラインをデアサート(OFF)してフロントパネル操作を許可
,SEQUENCE, 040, Gpib_REN, "OFF"
,SEQUENCE, 050, Gpib_Close
,SEQUENCE, 060, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: GPIB 通信環境および対象コマンドの PARAM 初期値が設定され、ステップ実行待機中である状態。
* **実行後の状態**: `Gpib_REN, "ON"` により全 GPIB ボードの REN ラインが ON（メイン画面の `REN` ボタン水色点灯）となり、続く `Gpib_Send` で測定器がリモートモードになります。 `Gpib_REN, "OFF"` により REN ラインが OFF（`REN` ボタン消灯）となり、バス上の測定器が一斉にローカルモードへ戻ります。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "シーケンス実行エンジン"
    participant Cmd as "Gpib_RENCommand"
    participant Parent as "IGpibAnalysisParent (GpibMainForm)"
    participant Dll as "GpibDLL / IGpibDriver"

    Script->>Cmd: Func(parent, param)
    Cmd->>Cmd: ValidateParamRange(0, 1) & State 文字列/非負整数判定
    alt 有効な State 指定 ("1"/"ON"/"0"/"OFF" 等 または 省略)
        alt メイン画面連携時 (parent != null)
            Cmd->>Parent: BtnRENSend(ButtonStatus.On / Off)
            Parent->>Dll: SetGpibRen(enable)
            Parent-->>Cmd: REN ボタン背景色更新 (Aqua / Gainsboro)
        else ヘッドレス実行時 (parent == null)
            Cmd->>Dll: SetGpibRen(enable)
            Dll-->>Cmd: REN ライン制御完了
        end
        Cmd-->>Script: 正常終了 (Next Step)
    else 負数 ("-1" 等)・未定義文字列・余剰引数指定時
        Cmd->>Cmd: SetErrorAndHalt (エラーログ記録)
        Cmd-->>Script: 安全停止 (IsCommandGo = false)
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **引数個数および不正 `State` 指定の事前検証**:
   - `ValidateParamRange(0, 1)` 違反（2 個以上の引数指定）、負の整数（`"-1"` 等）、または有効キーワード以外の非数値文字列が指定された場合は、`TryGetInt` バリデーションによりエラーログを記録し `HasError = true`・`IsCommandGo = false` でシーケンスを安全停止します。
2. **ポート未接続時の安全スキップ**:
   - メイン画面側でポート未接続（`CanSetRen == false`）の場合は物理バスへの送出を行わず安全にスキップします。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Gpib_RENCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `GpibDLL.DebugDLL` および `GpibDLL.DebugReceiveData` を元の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `ResolveRenStatus` に `(On, false)`・`(Off, true)`・`(Flicker, false)`・`(Flicker, true)` を指定して実行 | それぞれ `true`・`false`・`true`・`false` に状態解決される | REN オン／オフ／トグル反転ステータス解決ロジックの検証 |
| **2** | 正常系 | `GetRenButtonColor(true)` および `GetRenButtonColor(false)` を実行 | `true` 時は `Color.Aqua`、`false` 時は `Color.Gainsboro` を返す | REN 状態に応じた UI ボタン背景色判定の検証 |
| **3** | 正常系・異常系 | `CanSetRen(true)` / `(false)` 判定、および負数 `("-1")` を指定して `Func` を実行 | オープン時のみ `true` を返し、`("-1")` 指定時は `HasError == true`・`IsCommandGo == false` で安全停止する | ポート状態に応じた実行可否判定および負数指定時の安全停止検証 |
