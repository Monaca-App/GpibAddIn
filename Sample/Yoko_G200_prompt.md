# 横河計測 GS200 (G200) 用 GPIBマクロスクリプト自動生成 AI指示書

> **生成AI**: Antigravity (Gemini 4 Argon (Medium))

あなたは、独自仕様のGPIB制御システム向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。
以下の「参照ドキュメント」および「システム絶対制約」「GS200 機器仕様・SCPIコマンド体系」を厳格に遵守し、横河計測（YOKOGAWA）製 直流電圧・電流源 **GS200 (G200)** を制御・測定するためのGPIBマクロスクリプト（`Yoko_G200.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/GpibAddIn/マクロスクリプト取扱説明書.Md`（GPIBマクロスクリプト 総合取扱説明書 兼 AI自動生成指示書）
2. **通信・制御個別仕様（GPIB 専用コマンドセットおよび共通基盤）**
   * `Manual/GpibAddIn/GPIB/Gpib_Open.Md` / `Manual/GpibAddIn/GPIB/Gpib_Init.Md` / `Manual/GpibAddIn/GPIB/Gpib_Close.Md`（GPIB ボード接続・初期化・切断）
   * `Manual/GpibAddIn/GPIB/Gpib_Send.Md` / `Manual/GpibAddIn/GPIB/Gpib_Receive.Md` / `Manual/GpibAddIn/GPIB/Gpib_Wait.Md` / `Manual/GpibAddIn/GPIB/Gpib_DebugData.Md`（GPIB 送受信・バス待機・デバッグ疑似データ設定）
   * `Manual/Core/Work/Time_Stamp.Md` / `Manual/Core/Work/Set_Data.Md`（タイムスタンプ制御・受信データ抽出）
   * `Manual/Core/Param/Param_Set.Md` / `Manual/Core/Param/Param_Add.Md` / `Manual/Core/Param/Exit.Md`（パラメータ設定・加算・終了制御）
   * `Manual/Core/Sequence/Judge.Md` / `Manual/Core/Sequence/Jump.Md` / `Manual/Core/Sequence/Wait.Md`（比較判定・ジャンプ・周期ウェイト制御）
3. **対象機器マニュアル（通信仕様・SCPIコマンド仕様）**
   * 横河計測株式会社『GS200 ユーザーズマニュアル』（IM GS210-01JA）
   * URL: https://cdn.tmi.yokogawa.com/18/6218/files/IMGS210-01JA.pdf

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージに加え、 **`SERIALSTATUS` の各ビット定義名称も、マニュアルの英語表記をそのまま使わず、デフォルトの日本語に翻訳して設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Yoko_G200.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSVファイルの最先頭ヘッダコメント行（`;`）に、対象機器情報（メーカー名、型番、準拠マニュアル名・URL）とともに **他者が同一条件で再現できるよう、実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Argon (Medium))`）** を必ず記述してください（エージェント名のみの記載や、推測・例示の丸写しによる不正確なモデル名の記載は厳禁）。
   * **パス表記**: ファイル内に開発環境固有のローカル絶対パスや個人名を含めることは厳禁です。必ず相対パスまたはURLのみで記述してください。

2. **行頭判定と「第2フィールド」の鉄則**
   * **親ブロック（`TITLE`, `COMMAND`）**: 必ず行頭の1文字目（第1フィールド）から記述してください。
   * **子要素（`PARAM`, `SEQUENCE`, および `TITLE` 配下の設定行）**: 必ず **行頭をカンマ `,`（第1フィールドを空）** から開始し、第2フィールドにキーワードを記述してください。スペースやタブによるインデントは一切禁止です。

3. **PARAM 定義における「表示名（第5）」「初期値（第6）」の省略完全禁止（最重要）**
   * 書式: `,PARAM, [パラメータキー], [データ型], [表示名], [初期値], [追加選択肢...]`
   * データ型は `Text`, `List`, `Range`, `Value`, `Label`, `Work` の6種類のみ使用可能です。
   * **いかなるデータ型（`Range` 型や `Work` 型を含む）であっても、第5フィールド（表示名）および第6フィールド（初期値）の省略は絶対禁止** です（4フィールド以下で記述すると `IndexOutOfRangeException` が発生します）。初期値が空の場合は必ず `""` を明記してください。
     * 正しい例（Range型）: `,PARAM, "RESULT_DATA", Range, "測定データ結果", ""`
     * 正しい例（Work型）: `,PARAM, "LOOP_CNT", Work, "ループカウンタ", "0"`

4. **COMMAND ブロック配下での行末コメント禁止**
   * `TITLE` ブロック配下では行末コメント（`; ...`）が許可されますが、`COMMAND` ブロック配下の `PARAM` 行および `SEQUENCE` 行では **行末コメントは完全禁止** です。コメントは必ず独立した行（行頭 `;`）として記述してください。

5. **正式マクロコマンド名の厳守（旧コマンド名・存在しないコマンド名の使用禁止）**
   * `SEQUENCE` 行の第4フィールドに指定するマクロコマンド名は、`Manual/GpibAddIn/マクロスクリプト取扱説明書.Md` の「5. マクロコマンド詳細リファレンス」に定義された正式名称のみを使用してください。
   * **使用すべき正式コマンド名**:
     * GPIB通信制御: `Gpib_Open`, `Gpib_Init`, `Gpib_Send`, `Gpib_DebugData`, `Gpib_Receive`, `Gpib_SendReceive`, `Gpib_Wait`, `Gpib_Close`
     * ワークバッファ・時刻制御: `Time_Stamp`, `Set_Data`
     * パラメータ演算制御: `Param_Set`, `Param_Add`
     * シーケンス分岐・待機・対話制御: `Judge`, `Jump`, `Wait`, `Msg_Box`, `Exit`
   * ※旧名称（`Open`, `Init`, `Send`, `Receive`, `WaitData`, `Close`, `TimeStamp`, `ParamSet`, `Add`, `Equ`, `SetData_RowAdd`, `MsgBox`）は使用せず、必ず上記の正式名称を使用してください。

---

## 📡 3. GS200 (G200) 機器通信仕様および TITLE ブロック定義

`IM GS210-01JA` マニュアルに基づき、`TITLE` ブロックを以下の通り定義してください。

* **機器識別名**: `"YOKO_G200"`
* **物理設定**:
  * `,DEVICEBOARD, 0` （GPIBボード番号: 0）
  * `,DEVICEADDRESS1ST, 1` （GS200 デフォルトGPIBアドレス: 1）
  * `,DEVICEADDRESS2ND, -1` （セカンダリアドレス: 不使用）
  * `,DELIMITER, LF` （GS200 標準ターミネータ: LF + EOI）
  * `,EOI, true` （EOI信号送出: 有効）
* **シリアルポール・ステータスバイト定義（`SERIALSTATUS` 1〜8：日本語翻訳必須）**:
  GS200 のステータスバイト（IEEE 488.2 STB）定義に基づき、以下のように日本語で定義してください。
  * `,SERIALSTATUS, 1, "機能拡張用(未使用)"` （Bit 0）
  * `,SERIALSTATUS, 2, "機能拡張用(未使用)"` （Bit 1）
  * `,SERIALSTATUS, 3, "エラー発生(EAV)"` （Bit 2: Error Available）
  * `,SERIALSTATUS, 4, "クエスチョナブルステータス(QSB)"` （Bit 3: Questionable Summary Bit）
  * `,SERIALSTATUS, 5, "メッセージあり(MAV)"` （Bit 4: Message Available）
  * `,SERIALSTATUS, 6, "標準イベントサマリ(ESB)"` （Bit 5: Event Summary Bit）
  * `,SERIALSTATUS, 7, "サービスリクエスト(RQS/MSS)"` （Bit 6: Request Service / Master Summary Status）
  * `,SERIALSTATUS, 8, "オペレーションステータス(OSB)"` （Bit 7: Operation Summary Bit）

---

## 📐 4. 実装すべき COMMAND ブロックと設計仕様

初期化の独立化規約および単一責任の原則に従い、以下の7つの `COMMAND` ブロックを実装してください。

### ① 独立初期化コマンド：`COMMAND, "INIT"`
* **目的**: 計測開始前に1回だけ実行し、GPIBボードのオープン・初期化と機器のステータスクリアを行う独立マクロ。
* **シーケンス**:
  1. `Gpib_Open` でボード接続を開始
  2. `Gpib_Init` でインターフェース初期化を実行
  3. `Gpib_Send, "*CLS;:OUTPut OFF"` でステータスクリアおよび安全のため出力OFFを設定
  4. `Exit` で終了

### ② 電圧測定コマンド（単発・連続統合型）：`COMMAND, "MEAS_VOLT", "RESULT_DATA"`
* **目的**: GS200 の補助機能であるモニタ（測定）機能（`:SENSe` サブシステム）を電圧測定モードに設定し、指定回数・指定サンプリング周期で測定データを連続取得する（単発測定は取得回数=`1`で兼用）。
* **必須パラメータ（`PARAM`）**:
  1. `NPLC` (`List`, `"積分時間(PLC)"`, `"1"`, `"5"`, `"10"`, `"25"`)
  2. `COUNT` (`List`, `"取得回数"`, `"1"`, `"5"`, `"10"`, `"20"`, `"50"`, `"100"`, `"500"`, `"1000"`) ※初期値は必ず `"1"`
  3. `INTERVAL` (`List`, `"サンプリング周期(ms)"`, `"100"`, `"50"`, `"200"`, `"500"`, `"1000"`, `"2000"`, `"5000"`)
  4. `TIMESTAMP_EN` (`List`, `"タイムスタンプ付加"`, `"false"`, `"true"`)
  5. `RESULT_DATA` (`Range`, `"測定データ結果"`, `""`) ※結果格納先
  6. `LOOP_CNT` (`Work`, `"ループカウンタ"`, `"0"`) ※内部ループ制御用
  7. `DEBUG_VAL` (`Text`, `"デバッグ疑似データ"`, `"+1.23456E+00"`) ※Excelから指定できるよう必ず `Text` 型で定義（`Work` 型禁止）
* **シーケンス設計規約**:
  1. **最優先 Time_Stamp 設定**: 最初の送受信コマンド（`Gpib_Send`）より前のステップ10で必ず `Time_Stamp, $TIMESTAMP_EN$` を実行する。
  2. **測定機能設定**: `Gpib_Send, ":SENSe:FUNCtion VOLTage;:SENSe:NPLC $NPLC$;:SENSe:TRIGger IMMediate;:SENSe:STATe ON"` を送信し、電圧モニタを有効化する。
  3. **カウンタ初期化**: `Param_Set, "MEAS_VOLT", "0", "LOOP_CNT"` でループカウンタを `"0"` に初期化する。
  4. **測定ループ (`#LOOP_START#`)**:
     * `Gpib_Send, ":MEASure?"` （または `:FETCh?`）を送出して測定値を要求
     * **`Gpib_DebugData, "$DEBUG_VAL$"` を必ず `Gpib_Receive` の直前に配置** する
     * `Gpib_Receive` で応答を受信
     * `Gpib_Wait, "0", 5000` でGPIBバス待機
  5. **タイムスタンプ有無によるパース分岐**:
     * `Judge, EQU, "$TIMESTAMP_EN$", "true", #PARSE_NO_TS#` で判定（一致時は次行へ進み、不一致時は `#PARSE_NO_TS#` へジャンプ）
     * **タイムスタンプ有効時（`true`）**: `Set_Data, "RESULT_DATA", "[ %s %s ] R:%f", "%s, %s, %0.6f"` により、日付・時刻・測定値をカンマ区切り3列として格納し、`Jump, #CHECK_COUNT#` へ飛ぶ
     * **タイムスタンプ無効時（`false`：`#PARSE_NO_TS#`）**: `Set_Data, "RESULT_DATA", "R:%f", "%0.6f"` により、受信識別子 `R:` を除いた数値のみを格納する
  6. **回数判定と単発即時終了 (`#CHECK_COUNT#`)**:
     * `Param_Add, "MEAS_VOLT", "LOOP_CNT", "1"` でカウンタを +1 インクリメント
     * `Judge, LT, $LOOP_CNT$, $COUNT$, #FINISH#` で「`LOOP_CNT < COUNT`」を判定。規定回数に達した場合（条件不成立時）は **`Wait` をスキップして即座に `#FINISH#` へジャンプ** させる
     * 未到達時（条件成立時）は `Wait, $INTERVAL$` でサンプリング周期分待機し、`Jump, #LOOP_START#` でループ先頭へ戻る
  7. **終了処理 (`#FINISH#`)**:
     * `#FINISH#`: **必ず `Time_Stamp, false` を実行してタイムスタンプ設定をOFFに復帰** させた上で、`Exit` する。

### ③ 電流測定コマンド（単発・連続統合型）：`COMMAND, "MEAS_CURR", "RESULT_DATA"`
* **目的**: GS200 のモニタ（測定）機能を電流測定モード（`:SENSe:FUNCtion CURRent`）に設定し、指定回数・指定サンプリング周期で電流測定データを取得する。
* **パラメータおよびシーケンス構造**:
  * `MEAS_VOLT` と同一のパラメータ構成（`NPLC`, `COUNT`, `INTERVAL`, `TIMESTAMP_EN`, `RESULT_DATA`, `LOOP_CNT`, `DEBUG_VAL` [初期値 `"+1.00000E-02"`]）を持つこと。
  * 機器設定コマンドを `Gpib_Send, ":SENSe:FUNCtion CURRent;:SENSe:NPLC $NPLC$;:SENSe:TRIGger IMMediate;:SENSe:STATe ON"` とし、`Param_Set` / `Param_Add` の対象コマンド名を `"MEAS_CURR"` とすること。

### ④ 定電圧出力設定コマンド：`COMMAND, "SRC_VOLT"`
* **目的**: GS200 を定電圧源モード（`:SOURce:FUNCtion VOLTage`）に設定し、レンジ・出力電圧値・電流リミット値を一括設定する。
* **必須パラメータ（`PARAM`）**:
  1. `SRC_RANGE` (`List`, `"電圧レンジ(V)"`, `"10"`, `"10E-3"`, `"100E-3"`, `"1"`, `"30"`)
  2. `SRC_LEVEL` (`Text`, `"出力電圧値(V)"`, `"5.0000"`)
  3. `CURR_LIMIT` (`Text`, `"電流リミット(A)"`, `"0.100"`)
* **シーケンス**:
  * `Gpib_Send, ":SOURce:FUNCtion VOLTage;:SOURce:RANGe $SRC_RANGE$;:SOURce:LEVel $SRC_LEVEL$;:SOURce:PROTection:CURRent $CURR_LIMIT$"` を送信して `Exit`。

### ⑤ 定電流出力設定コマンド：`COMMAND, "SRC_CURR"`
* **目的**: GS200 を定電流源モード（`:SOURce:FUNCtion CURRent`）に設定し、レンジ・出力電流値・電圧リミット値を一括設定する。
* **必須パラメータ（`PARAM`）**:
  1. `SRC_RANGE` (`List`, `"電流レンジ(A)"`, `"10E-3"`, `"1E-3"`, `"100E-3"`, `"200E-3"`)
  2. `SRC_LEVEL` (`Text`, `"出力電流値(A)"`, `"0.01000"`)
  3. `VOLT_LIMIT` (`Text`, `"電圧リミット(V)"`, `"10.0"`)
* **シーケンス**:
  * `Gpib_Send, ":SOURce:FUNCtion CURRent;:SOURce:RANGe $SRC_RANGE$;:SOURce:LEVel $SRC_LEVEL$;:SOURce:PROTection:VOLTage $VOLT_LIMIT$"` を送信して `Exit`。

### ⑥ 出力 ON/OFF 制御コマンド：`COMMAND, "OUTP_SET"`
* **目的**: GS200 の出力状態（`:OUTPut`）を ON または OFF に切り替える。
* **必須パラメータ（`PARAM`）**:
  1. `OUTP_STATE` (`List`, `"出力状態"`, `"OFF"`, `"ON"`)
* **シーケンス**:
  * `Gpib_Send, ":OUTPut $OUTP_STATE$"` を送信して `Exit`。

### ⑦ 通信終了・解放コマンド：`COMMAND, "CLOSE"`
* **目的**: 全計測終了時に機器出力を安全に OFF にし、GPIB ボード接続をクローズする独立マクロ。
* **シーケンス**:
  1. `Gpib_Send, ":OUTPut OFF"`
  2. `Gpib_Close`
  3. `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

スクリプト生成後、以下の項目をすべて満たしているか検証してから出力してください。

- [ ] ファイル先頭のヘッダコメントに、他者が再現できるよう自身のAIエンジン（モデル名・ティア等）を正確に判別できる `; 生成AI: <エージェント名> (<AIエンジン名>)` および GS200 の機器情報・マニュアルURLが記載されているか？
- [ ] `TITLE` 配下の `SERIALSTATUS`（1〜8）の名称がすべて日本語に翻訳されているか？
- [ ] すべての `PARAM`, `SEQUENCE`, および `TITLE` 配下設定行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行が第5フィールド（表示名）と第6フィールド（初期値）を備えているか（4フィールド以下の行が存在しないか）？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] 使用しているすべてのマクロコマンド名が、取扱説明書5章の正式名称（`Gpib_Open`, `Gpib_Init`, `Gpib_Send`, `Gpib_Receive`, `Gpib_Wait`, `Gpib_Close`, `Time_Stamp`, `Set_Data`, `Param_Set`, `Param_Add`, `Judge`, `Jump`, `Wait`, `Exit`）と完全一致しているか？
- [ ] `MEAS_VOLT` と `MEAS_CURR` の両方で、取得回数（`COUNT`）、サンプリング周期（`INTERVAL`）、タイムスタンプ付加（`TIMESTAMP_EN`）がパラメータで指定可能になっているか？
- [ ] `Time_Stamp, $TIMESTAMP_EN$` がすべての送受信コマンドより前（ステップ10）に配置されているか？
- [ ] タイムスタンプ有効時は `Set_Data, "RESULT_DATA", "[ %s %s ] R:%f", "%s, %s, %0.6f"`、無効時は `Set_Data, "RESULT_DATA", "R:%f", "%0.6f"` でパースされているか？
- [ ] 規定回数到達時に `Wait` をスキップして `#FINISH#` へジャンプし、`Time_Stamp, false` を実行してから `Exit` しているか？
