# コマンドリファレンス：Gpib_Reset

## 概要

**Gpib_Reset** は、接続中の対象 GPIB デバイス（または指定アドレスの測定器）に対してデバイスクリア（SDC）および IEEE 488.2 標準リセットコマンド（`*RST;*CLS`）を送出し、測定器を工場出荷時（または既定）の初期測定状態へリセットするコマンドです。

> 💡 **補足**: 通信インターフェース層のリセット（SDC）と機器内部設定層のリセット（`*RST;*CLS`）を 1 ステップで確実に実行します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Gpib_Reset, "[DeviceAddress]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `DeviceAddress` | 任意 | 整数（`0`〜`30`） | リセット対象の GPIB プライマリアドレスを指定します。省略時は現在 `Gpib_Open` されている対象機器をリセットします。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: SCPI コマンド等、引数文字列自体にカンマを含める場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Reset`, `GpibReset`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（;）は記述しないでください。

```csv
COMMAND, "SAMPLE_GPIB_RESET", ""
,SEQUENCE, 010, Gpib_Open, "5"
; 1. カレントデバイス(アドレス5)を完全リセットして初期状態に戻す
,SEQUENCE, 020, Gpib_Reset
,SEQUENCE, 030, Wait, "200"
; 2. 初期状態からDC電圧レンジを設定して測定実行
,SEQUENCE, 040, Gpib_Send, "CONF:VOLT:DC 10"
,SEQUENCE, 050, Gpib_SendReceive, "READ?", "@VOLT_VAL"
,SEQUENCE, 060, Gpib_Close
,SEQUENCE, 070, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: GPIB 通信環境および対象コマンドの PARAM 初期値が設定され、ステップ実行待機中である状態。
* **実行後の状態**: `Gpib_Reset` により、対象機器に対して Selected Device Clear（SDC）および `"*RST;*CLS"` が送出され、内部受信バッファ `SBuf` もクリアされます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Gpib_ResetCommand (GpibAnalysis.GPIB)"
    participant Drv as "IGpibDriverAdapter (NI-488.2 / VISA)"
    participant SBuf as "GPIB 受信電文バッファ (SBuf)"

    Script->>Cmd: Execute(context, deviceAddr)
    Cmd->>SBuf: Clear()
    Cmd->>Drv: ResetDevice(deviceAddr) (SDC + "*RST;*CLS" 送出)
    alt リセット成功
        Drv-->>Cmd: リセット完了
        Cmd-->>Script: 正常終了 (Next Step)
    else バスエラーまたはアドレス範囲外
        Drv-->>Cmd: GpibException / ArgumentOutOfRangeException
        Cmd-->>Script: エラーログ出力・シーケンス停止
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **アドレス範囲（`0`〜`30`）の検証**:
   - 第 1 引数に `0`〜`30` 以外の数値が指定された場合は `ArgumentOutOfRangeException` を送出します。
2. **未接続時のエラー通知**:
   - デバイスがオープンされておらず第 1 引数も省略されている場合は、`InvalidOperationException` を送出します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Gpib_ResetCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `GpibDLL.DebugDLL` および `GpibDLL.DebugReceiveData` を元の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `GpibDevice`（`EquipmentId = 1`）設定下での引数省略 `()` 実行、およびボード番号 `("0")` 明示指定での `Func` 実行 | 両ケースとも `!HasError` かつ `IsCommandGo == true` で物理リセットが正常完了する | 引数省略時（対象機器 ID）および ID 明示指定時のリセット動作検証 |
| **2** | 異常系 | 非数値文字列 `("INVALID")` および余剰引数 `("0", "EXTRA")` を指定して `Func` を実行 | 両ケースとも `HasError == true` かつ `IsCommandGo == false` となり安全停止する | 不正数値指定および引数個数超過時のバリデーション防護検証 |
