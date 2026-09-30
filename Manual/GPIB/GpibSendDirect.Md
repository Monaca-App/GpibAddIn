# コマンドリファレンス：SendDirect

## 概要
`SendDirect` は、機器登録（イクウィップメントID）を事前に行わず、ボード番号と宛先アドレスリストを直接指定して、ターゲット機器群へコマンドや設定データを直接送信するためのコマンドです。

## 🔄 動作フロー（シーケンス図）
本コマンド実行時における、シーケンス実行エンジン（GpibExecution）、コマンドクラス（GpibSendDirect）、物理通信層（GpibDLL）、および接続機器間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    actor User as シーケンス実行
    participant Engine as GpibExecution
    participant Cmd as GpibSendDirect
    participant DLL as GpibDLL
    participant Bus as GPIBバス / 機器

    User->>Engine: マクロ実行 (SendDirect)
    Engine->>Cmd: Func(parent, sequence, param)
    Cmd->>Cmd: 引数の検証・パース (BoardNum, Addrs, Data)
    Cmd->>DLL: TrySendDirect(boardNum, addrs, buffer)
    DLL->>Bus: GpibSend (物理データ送信)
    Bus-->>DLL: 送信完了ステータス
    DLL-->>Cmd: 結果 (true/false)
    Cmd-->>Engine: 実行完了
```

---

## 💡 書式・パラメータ仕様

```csv
,SEQUENCE, [ステップキー], SendDirect, [ボード番号], [宛先アドレスリスト], [送信データ]
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| **引数1** | ボード番号 | 通信に使用するGPIBボードの番号。 | 整数（例: `"0"`）またはパラメータ動的置換 |
| **引数2** | 宛先アドレスリスト | 送信先の機器プライマリアドレス。カンマ区切りで複数指定可能。 | カンマ区切りの文字列（例: `"1"` や `"1,2"`） |
| **引数3** | 送信データ | 送信するデータ文字列。 | 固定文字列またはパラメータ動的置換 |

---

## 🔄 特殊置換規約・分岐規約の適用
*   **動的置換（`$`）**: すべての引数において `$PARAMキー$` を用いた変数置換が可能です。
*   **カンマのエスケープ**: 引数にカンマを含める場合は二重引用符で囲んでください。

---

## 🚫 エラーハンドリング・安全設計
*   引数不足やパース失敗時には、エラーログを出力して処理を安全に早期リターン（処理スキップ）します。
*   物理層からの送信結果がエラーとなった場合は、内部エラーログにエラー内容を出力します。
