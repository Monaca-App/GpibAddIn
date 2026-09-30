# コマンドリファレンス：DebugData

## 概要
DebugData は、GPIB通信のデバッグ実行時に使用される疑似受信データ（`GpibDLL.DebugReceiveData`）を動的に設定・更新する制御用コマンドです。物理デバイスが接続されていないデバッグ環境において、マクロシーケンス経由で受信応答データをモックする場合に使用します。

> [!IMPORTANT]
> **配置順序の絶対制約（Receive 直前に指定）**  
> `DebugData` コマンドは、デバッグ実行時（`DebugDLL=true`）に `Receive` コマンドが読み出す内部バッファ（`GpibDLL.DebugReceiveData`）へ疑似データを事前設定するものです。  
> したがって、**必ず `Receive` コマンドの直前（前）に配置してください**。  
> `Receive` より後に配置しても、受信処理には反映されません。
>
> **データ回収の標準手順**:  
> `[クエリ送信]` ➔ **`DebugData`** ➔ **`Receive`** ➔ `WaitData` ➔ `SetData`

## 💡 書式・パラメータ仕様
```text
,SEQUENCE, [ステップキー], DebugData, [設定する疑似受信データ]
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| 引数1 | 設定する疑似受信データ | 受信データとして返したい任意のテキスト。 | 固定値 / PARAM変数キー / 動的置換（$）可能 |

## 🔄 特殊置換規約・分岐規約の適用
* **動的置換（$）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。
* **条件ジャンプ（#）**: 本コマンドはシーケンス制御を行わないため、条件ジャンプは適用されません。

## 💡 動作例（正しい構文での記述例）
```text
,PARAM, 送信データ, List, "送信データ", ""
,PARAM, デバッグデータ, Value, "デバッグデータ", "5.0123"
,PARAM, 測定値, Value, "測定値", "0.0"

,SEQUENCE, 10, Call, "送信", "送信データ", ":MEAS?"
,SEQUENCE, 20, DebugData, "$デバッグデータ$"
,SEQUENCE, 30, Receive
,SEQUENCE, 40, WaitData
,SEQUENCE, 50, SetData, "測定値", "%f", "%0.6f"
```

### 実行時の挙動
* **実行前の状態**: `GpibDLL.DebugReceiveData` には `"Test Data"`（初期値）が設定されています。
* **実行後の状態**: `GpibDLL.DebugReceiveData` の値が `"ACK_OK"` に更新され、以降の `ReceiveGpibData` 呼び出し（デバッグモード時）でこの値が返るようになります。

---

## 🧪 ユニットテスト仕様

本コマンドクラスに実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件
* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより即座に早期リターン）
* **検証結果出力**: `ErrorLogger.LogTest` によるログ出力。検証失敗時は `ErrorLogger.LogError`、予期せぬ例外発生時は `ErrorLogger.LogCritical` で記録

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :--- | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `sequence.Param(1) = "TestParamValue"` | `GpibDLL.DebugReceiveData == "TestParamValue"` | 指定した値が `GpibDLL.DebugReceiveData` に正しく設定されること。 |
