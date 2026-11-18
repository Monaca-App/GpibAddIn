# Keysight (旧 Agilent / HP) 3458A 用 GPIBマクロスクリプト自動生成 AI指示書

> **生成AI**: Antigravity (Gemini 4 Argon (Medium))

あなたは、独自仕様のGPIB制御システム向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。
以下の「参照ドキュメント」および「システム絶対制約」「3458A 機器仕様・コマンド体系（HPSL）」を厳格に遵守し、Keysight Technologies（旧 Agilent / Hewlett-Packard）製 8.5桁デジタル・マルチメータ **3458A** を制御・測定するためのGPIBマクロスクリプト（`AG_3458a.csv`）を作成してください。

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
3. **対象機器マニュアル（通信仕様・HPSLコマンド仕様）**
   * Agilent Technologies『Agilent 3458A Multimeter User's Guide』
   * URL: https://dfzk-www.oss-cn-beijing.aliyuncs.com/www-PRD/resources/files/Ag_3458A_UserGuide_en.pdf

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージに加え、 **`SERIALSTATUS` の各ビット定義名称も、英語マニュアルの表記（`"Data Ready"`, `"Error"` 等）をそのまま使わず、デフォルトの日本語に翻訳して設定** してください。
   * **文字コード**: 生成する CSV ファイル（`AG_3458a.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
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
     * GPIB通信制御: `Gpib_Open`, `Gpib_Init`, `Gpib_Send`, `Gpib_Receive`, `Gpib_SendReceive`, `Gpib_Wait`, `Gpib_Close`, `Gpib_DebugData`
     * ワークバッファ・時刻制御: `Time_Stamp`, `Set_Data`
     * パラメータ演算制御: `Param_Set`, `Param_Add`
     * シーケンス分岐・待機・対話制御: `Judge`, `Jump`, `Wait`, `Msg_Box`, `Exit`
   * ※旧名称（`Open`, `Init`, `Send`, `Receive`, `WaitData`, `Close`, `TimeStamp`, `ParamSet`, `Add`, `Equ`, `SetData_RowAdd`, `MsgBox`）は使用せず、上記正式名称に統一してください。

---

## 📡 3. 3458A 機器通信仕様および TITLE ブロック定義

⚠️ **【最重要注意点】3458A 固有のコマンド体系（HPSL）と EOI 設定**:
* 3458A は SCPI 規格制定前の設計であり、独自の **HPSL（Hewlett-Packard System Language）** コマンド体系を採用しています（`*RST` や `:MEASure:VOLTage:DC?` などの SCPI コマンドは動作しません）。
* また、3458A は電源投入時のデフォルトで GPIB 出力時に **EOI 信号をアサートしない仕様（`END OFF`）** となっているため、初期化時および測定コマンド先頭で **`END ALWAYS`** を送信し、データ終端で確実に EOI が送出されるように設定する必要があります。

* **機器識別名**: `"AG_3458A"`
* **物理設定**:
  * `,DEVICEBOARD, 0` （GPIBボード番号: 0）
  * `,DEVICEADDRESS1ST, 22` （3458A 工場出荷時デフォルトGPIBアドレス: 22）
  * `,DEVICEADDRESS2ND, -1` （セカンダリアドレス: 不使用）
  * `,DELIMITER, CRLF` （3458A 標準出力デリミタ: CR+LF）
  * `,EOI, true` （EOI信号制御: 有効）
* **シリアルポール・ステータスバイト定義（`SERIALSTATUS` 1〜8：日本語翻訳必須）**:
  3458A User's Guide（Status Register 定義）に基づき、以下のように日本語へ翻訳して定義してください。
  * `,SERIALSTATUS, 1, "サブプログラム実行完了"` （Bit 0: Subprogram execution completed）
  * `,SERIALSTATUS, 2, "ハイまたはローリミット超過"` （Bit 1: High or Low limit exceeded）
  * `,SERIALSTATUS, 3, "SRQコマンド実行"` （Bit 2: SRQ command executed）
  * `,SERIALSTATUS, 4, "電源投入(PON)"` （Bit 3: Power On）
  * `,SERIALSTATUS, 5, "レディ(コマンド実行完了)"` （Bit 4: Ready for instructions）
  * `,SERIALSTATUS, 6, "エラー発生(ERR)"` （Bit 5: Error bit）
  * `,SERIALSTATUS, 7, "サービスリクエスト(RQS)"` （Bit 6: Request Service）
  * `,SERIALSTATUS, 8, "データレディ(測定データ準備完了)"` （Bit 7: Data Available / Hi-Speed Data Ready）

---

## 📐 4. 実装すべき COMMAND ブロックと設計仕様

単一責任の原則（ファンクション分岐の禁止）および単発・連続測定統合の原則に従い、以下の9つの独立した `COMMAND` ブロックを実装してください。

### ① 独立初期化コマンド：`COMMAND, "INIT"`
* **目的**: 計測開始前に1回実行し、GPIBボードのオープン・初期化と 3458A の基本設定（リセット、EOI常時送出、ASCII出力形式、トリガホールド）を行う。
* **シーケンス**:
  1. `Gpib_Open`
  2. `Gpib_Init`
  3. `Gpib_Send, "RESET;END ALWAYS;OFORMAT ASCII;TRIG HOLD"`
  4. `Exit`

### ②〜⑧ 各測定ファンクションコマンド（単発・連続統合型）
1つのコマンドで複数ファンクションを兼ねることを禁止し、測定項目ごとに以下の7つの独立した `COMMAND` を作成してください（すべて結果格納先キー `"RESULT_DATA"` を持つ）。

| 項番 | コマンド名 | 測定項目 | 機器設定コマンド（ステップ20の `Gpib_Send`） | `RANGE`（測定レンジ）の選択肢（第6フィールドが初期値） |
| :---: | :--- | :--- | :--- | :--- |
| ② | **`DCV`** | 直流電圧測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;DCV $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"0.1"`, `"1"`, `"10"`, `"100"`, `"1000"` |
| ③ | **`ACV`** | 交流電圧測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;ACV $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"0.01"`, `"0.1"`, `"1"`, `"10"`, `"100"`, `"1000"` |
| ④ | **`OHM`** | 2端子抵抗測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;OHM $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"10"`, `"100"`, `"1E3"`, `"10E3"`, `"100E3"`, `"1E6"`, `"10E6"`, `"100E6"`, `"1E9"` |
| ⑤ | **`OHMF`** | 4端子抵抗測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;OHMF $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"10"`, `"100"`, `"1E3"`, `"10E3"`, `"100E3"`, `"1E6"`, `"10E6"`, `"100E6"`, `"1E9"` |
| ⑥ | **`DCI`** | 直流電流測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;DCI $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"1E-7"`, `"1E-6"`, `"1E-5"`, `"1E-4"`, `"1E-3"`, `"1E-2"`, `"0.1"`, `"1"` |
| ⑦ | **`ACI`** | 交流電流測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;ACI $RANGE$;NPLC $NPLC$"` | `"AUTO"`, `"1E-4"`, `"1E-3"`, `"1E-2"`, `"0.1"`, `"1"` |
| ⑧ | **`FREQ`** | 周波数測定 | `"END ALWAYS;OFORMAT ASCII;TRIG HOLD;FSOURCE ACV;FREQ $RANGE$"` | `"AUTO"`, `"0.01"`, `"0.1"`, `"1"`, `"10"`, `"100"`, `"1000"` |

#### 各測定コマンド共通の必須パラメータ（`PARAM`）
* `RANGE` (`List`, `"測定レンジ"`, 上表の初期値および選択肢)
* `NPLC` (`List`, `"積分時間(PLC)"`, `"10"`, `"0.001"`, `"0.01"`, `"0.1"`, `"1"`, `"100"`) ※`FREQ` 以外の測定コマンドに定義
* `COUNT` (`List`, `"取得回数"`, `"1"`, `"5"`, `"10"`, `"20"`, `"50"`, `"100"`, `"500"`, `"1000"`) ※初期値は必ず `"1"`
* `INTERVAL` (`List`, `"サンプリング周期(ms)"`, `"100"`, `"50"`, `"200"`, `"500"`, `"1000"`, `"2000"`, `"5000"`)
* `TIMESTAMP_EN` (`List`, `"タイムスタンプ付加"`, `"false"`, `"true"`)
* `DEBUG_VAL` (`Text`, `"デバッグ疑似データ"`, 各測定項目に応じた標準的な測定値文字列。例: `"+1.00000000E+00"`)
* `RESULT_DATA` (`Range`, `"測定データ結果"`, `""`)
* `LOOP_CNT` (`Work`, `"ループカウンタ"`, `"0"`)

#### 各測定コマンド共通のシーケンス構造（8.5桁高精度フォーマット対応）
1. **ステップ10（最優先）**: `Time_Stamp, $TIMESTAMP_EN$` （すべての送受信コマンドに先立って実行）
2. **ステップ20**: `Gpib_Send, "(上表の機器設定コマンド)"`
3. **ステップ30**: `Param_Set, "(自身のコマンド名)", "0", "LOOP_CNT"`
4. **ステップ `#LOOP_START#`**: `Gpib_Send, "TRIG SGL"` （単発トリガを実行して1回測定）
5. **ステップ50（オフラインデバッグ疑似データ登録）**: `Gpib_DebugData, "$DEBUG_VAL$"` （必ず `Gpib_Receive` の直前に配置）
6. **ステップ60**: `Gpib_Receive` （測定値の物理受信）
7. **ステップ70**: `Gpib_Wait, "0", 5000` （GPIBバス待機）
8. **ステップ80（タイムスタンプ分岐）**: `Judge, EQU, "$TIMESTAMP_EN$", "true", #PARSE_NO_TS#`
9. **ステップ90（タイムスタンプ有効時）**: `Set_Data, "RESULT_DATA", "[ %s %s ] R:%f", "%s, %s, %0.8f"` （8.5桁DMMに合わせて小数点以下8桁 `%0.8f` で日付・時刻・測定値の3列に展開）
10. **ステップ100**: `Jump, #CHECK_COUNT#`
11. **ステップ `#PARSE_NO_TS#`（タイムスタンプ無効時）**: `Set_Data, "RESULT_DATA", "R:%f", "%0.8f"`
12. **ステップ `#CHECK_COUNT#`（回数インクリメント）**: `Param_Add, "(自身のコマンド名)", "LOOP_CNT", "1"`
13. **ステップ120（規定回数判定・即時終了）**: `Judge, LT, $LOOP_CNT$, $COUNT$, #FINISH#` （規定回数到達時は `Wait` をスキップして `#FINISH#` へ即時ジャンプ）
14. **ステップ130（サンプリング周期ウェイト）**: `Wait, $INTERVAL$`
15. **ステップ140**: `Jump, #LOOP_START#`
16. **ステップ `#FINISH#`**: `Time_Stamp, false` （タイムスタンプ機能を必ずOFFへ復帰）
17. **ステップ160**: `Exit`

### ⑨ 通信終了・解放コマンド：`COMMAND, "CLOSE"`
* **目的**: 全計測完了時に GPIB ボードを安全にクローズする独立マクロ。
* **シーケンス**:
  1. `Gpib_Close`
  2. `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] ファイル先頭のヘッダコメントに、他者が再現できるよう自身のAIエンジン（モデル名・ティア等）を正確に判別できる `; 生成AI: <エージェント名> (<AIエンジン名>)` および 3458A の機器情報・マニュアルURLが記載されているか？
- [ ] `TITLE` 配下の `SERIALSTATUS`（1〜8）の名称がすべて日本語に翻訳されているか？
- [ ] すべての `PARAM`, `SEQUENCE`, および `TITLE` 配下設定行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行が第5フィールド（表示名）と第6フィールド（初期値）を備えているか？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] 使用しているすべてのマクロコマンド名が、取扱説明書5章の正式名称（`Gpib_Open`, `Gpib_Init`, `Gpib_Send`, `Gpib_Receive`, `Gpib_Wait`, `Gpib_Close`, `Gpib_DebugData`, `Time_Stamp`, `Set_Data`, `Param_Set`, `Param_Add`, `Judge`, `Jump`, `Wait`, `Exit`）と完全一致しているか？
- [ ] 3458A 固有の HPSL コマンド（`RESET`, `END ALWAYS`, `OFORMAT ASCII`, `TRIG HOLD`, `TRIG SGL` 等）が使用されているか？
- [ ] すべての測定コマンドで、取得回数（`COUNT`）、サンプリング周期（`INTERVAL`）、タイムスタンプ付加（`TIMESTAMP_EN`）、デバッグ疑似データ（`DEBUG_VAL`）がパラメータで指定可能になっているか？
- [ ] `Time_Stamp, $TIMESTAMP_EN$` がすべての送受信コマンドより前（ステップ10）に配置され、終了時（`#FINISH#`）に `Time_Stamp, false` で復帰されているか？
