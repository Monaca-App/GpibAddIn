# コマンドリファレンス：Gpib_Reset

---

## 概要

**Gpib_Reset** は、接続中の対象 GPIB デバイス（または指定アドレスの測定器）に対してデバイスクリア（SDC）および IEEE 488.2 標準リセットコマンド（`*RST;*CLS`）を送出し、測定器を工場出荷時（または既定）の初期測定状態へリセットするコマンドです。

> 💡 **補足**: 通信インターフェース層のリセット（SDC）と機器内部設定層のリセット（`*RST;*CLS`）を 1 ステップで確実に実行します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE,Gpib_Reset, "[DeviceAddress]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `DeviceAddress` | 任意 | 整数（`0`〜`30`） | リセット対象の GPIB プライマリアドレスを指定します。省略時は現在 `Gpib_Open` されている対象機器をリセットします。 |

---

## 🔄 特殊置換規約・分岐規約の適用

1. **変数展開（`$` / `@` / `#`）**:
   - 第 1 引数 `DeviceAddress` には変数を指定可能です（例: `"$DMM_ADDR"`）。
2. **レガシー別名互換**:
   - 旧表記の `GpibReset` や `GPIB_RESET` は内部正規化により本 `Gpib_Reset` コマンドへマッピングされます。

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

```csv
,SEQUENCE,Gpib_Open, "5"

; 1. カレントデバイス(アドレス5)を完全リセットして初期状態に戻す
,SEQUENCE,Gpib_Reset
,SEQUENCE,Wait, "200"

; 2. 初期状態からDC電圧レンジを設定して測定実行
,SEQUENCE,Gpib_Send, "CONF:VOLT:DC 10"
,SEQUENCE,Gpib_SendReceive, "READ?", "@VOLT_VAL"
,SEQUENCE,Gpib_Close
,SEQUENCE,Exit
```

### 実行時の挙動・変数変化

1. `Gpib_Reset` により、対象機器に対して Selected Device Clear（SDC）および `"*RST;*CLS"` が送出され、内部受信バッファ `SBuf` もクリアされます。

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

### テスト実行前提条件
- **対象クラス**: `GpibAnalysis.GPIB.Gpib_ResetCommand`（`Gpib_ResetCommand.cs`）
- **モック環境**: `IGpibDriverAdapter` モックが注入されていること。

### テストケース一覧

| テストID | テスト観点 | 入力スクリプト | 期待される動作・検証項目 |
| :--- | :--- | :--- | :--- |
| `UT-GPIB-RESET-01` | 正常系（カレント機器リセット） | `,SEQUENCE,Gpib_Reset` | オープン中の機器に対して `ResetDevice` が呼ばれ、`SBuf` がクリアされること。 |
| `UT-GPIB-RESET-02` | 正常系（アドレス指定リセット） | `,SEQUENCE,Gpib_Reset, "12"` | アドレス `12` に対して `ResetDevice(12)` が呼ばれること。 |
| `UT-GPIB-RESET-03` | 異常系（アドレス範囲外） | `,SEQUENCE,Gpib_Reset, "99"` | `ArgumentOutOfRangeException` が送出されること。 |

