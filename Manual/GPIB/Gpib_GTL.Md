# コマンドリファレンス：Gpib_GTL

## 概要

**Gpib_GTL** は、現在選択されている対象 GPIB 機器に対して IEEE 488 アドレス指定コマンド **GTL（Go To Local: `0x01`）** を送出し、その測定器をリモート制御状態（REM）からフロントパネル手動操作可能なローカル状態（LOC）へ復帰させるコマンドです。

> 💡 **補足**: REN ライン全体を OFF にする [`Gpib_REN`](./Gpib_REN.Md)（バス上の全機器がローカル化）とは異なり、現在アクティブな対象測定器 1 台のみをローカル操作に戻したい場合に使用します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Gpib_GTL, "[DeviceAddress]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `DeviceAddress` | 任意 | 整数（`ParameterSpec`: 既定値 `0`, `0`〜`30` / `Func`: `0` 以上） | 互換性のための任意パラメータです（`Func` 実行時は引数指定時に `0` 以上の整数であるか検証のみ行われ、実際の GTL 送出は常に現在選択中のカレント機器に対して実行されます）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: SCPI コマンド等、引数文字列自体にカンマを含める場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `GTL`, `GpibGTL`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマ（`,`）から開始してください（行頭スペースのみでは認識されません）。なお、`;` で始まる行やクォート外の `;` 以降はコメントとして自動除去されます。

```csv
COMMAND, "SAMPLE_GPIB_GTL", ""
,SEQUENCE, 010, Gpib_Open, 0
,SEQUENCE, 020, Gpib_Send, "AUTOSCALE"
,SEQUENCE, 030, Wait, "1000"
; 選択中のオシロスコープをローカル状態(GTL)に戻し、作業者が波形をパネル操作で確認できるようにする
,SEQUENCE, 040, Gpib_GTL
,SEQUENCE, 050, Msg_Box, "オシロスコープの波形を確認し、OKを押してください"
,SEQUENCE, 060, Gpib_Close
,SEQUENCE, 070, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態**: GPIB 通信環境および対象コマンドの PARAM 初期値が設定され、ステップ実行待機中である状態。
* **実行後の状態**: `Gpib_Send, "AUTOSCALE"` により対象機器がリモート制御状態になります。 `Gpib_GTL` の実行によりカレント機器へ GTL（ローカル移行）が送出され、機器の `RemoteStatus` が `Local`（メイン画面のボタン表示は `GTR`）へ更新されてフロントパネル操作が有効に戻ります。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "シーケンス実行エンジン"
    participant Cmd as "Gpib_GTLCommand"
    participant Parent as "IGpibAnalysisParent (GpibMainForm)"
    participant Dll as "GpibDLL (DisableRemoteControl)"

    Script->>Cmd: Func(parent, param)
    Cmd->>Cmd: ValidateParamRange(0, 1) & TryGetInt(DeviceAddress >= 0)
    alt 引数バリデーション成功
        alt メイン画面連携時 (parent != null)
            Cmd->>Parent: SendGtlGtr(ButtonStatus.Off)
            Parent->>Dll: DisableRemoteControl(currentDevice)
            Parent-->>Cmd: UI ボタン状態更新 (Local / GTR 表示)
        else ヘッドレス実行時 (parent == null && targetDevice != null)
            Cmd->>Dll: DisableRemoteControl(targetDevice)
            Dll-->>Cmd: RemoteStatus = Local 更新完了
        end
        Cmd-->>Script: 正常終了 (Next Step)
    else 負数・非数値・余剰引数指定時
        Cmd->>Cmd: SetErrorAndHalt (エラーログ記録)
        Cmd-->>Script: 安全停止 (IsCommandGo = false)
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **引数個数および最小値（`0` 以上）の事前検証**:
   - `ValidateParamRange(0, 1)` 違反（2 個以上の引数指定）または第 1 引数 `DeviceAddress` に負数（`"-1"` 等）や非数値文字列が指定された場合は、`_log.LogError` を記録し `SetErrorAndHalt` によりシーケンスを安全停止します。
2. **カレント機器への統一送出とドライバエラー時の継続**:
   - 第 1 引数に数値を指定した場合でも送出先は常に現在の実行対象機器（`targetDevice`）となります。物理ドライバがエラーを返した場合は `GpibDLL` 内でエラーログを記録しますが、シーケンスは強制停止せず次ステップへ継続します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Gpib_GTLCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `GpibDLL.DebugDLL` および `GpibDLL.DebugReceiveData` を元の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `GetGtlButtonStatus()`、`ResolveButtonTargetStatus(Flicker, true/false)`、および `GetNextRemoteStatus(Off)` を実行 | `Off`、トグル `Off` / `On`、および `RemoteStatusEnum.Local` を返す | GTL ボタン状態解決およびローカルステータス遷移計算の検証 |
| **2** | 正常系・異常系 | `RemoteStatus = Remote` の `"DEV1"` に対しアドレス `("1")` を指定して `Func` を実行、および負のアドレス `("-1")` を指定して `Func` を実行 | 正常指定時は `!HasError` かつ `dev.RemoteStatus == RemoteStatusEnum.Local` となり、負数指定時は `HasError == true` かつ `IsCommandGo == false` で安全停止する | シミュレーション環境下でのリモート解除（GTL）動作および範囲外 `Address`（`min: 0`）指定時のエラー停止検証 |
