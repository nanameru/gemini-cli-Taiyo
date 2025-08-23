# ツール実行スケジューラー（CoreToolScheduler）完全ガイド
### 〜文系でも分かる高度なタスク管理システム〜

---

## 📚 はじめに

このドキュメントでは、Gemini CLIプロジェクトの心臓部とも言える「ツール実行スケジューラー（CoreToolScheduler）」について、文系の方でも理解できるよう、身近な例えを使って詳しく説明します。

**技術用語が分からなくても大丈夫です！** レストランの注文管理システムに例えながら、この複雑なシステムの仕組みを分かりやすく解説していきます。

---

## 🏪 概要編：高級レストランの注文管理システム

### レストランで理解するツール実行スケジューラー

想像してみてください。あなたは人気の高級レストランのマネージャーです。このレストランでは：

- **ツール（道具）** = 調理器具や設備（オーブン、コンロ、ミキサーなど）
- **ツール実行スケジューラー** = 注文を効率的に管理する優秀なマネージャー（あなた）
- **ツールコール（道具の呼び出し）** = お客様からの注文

### マネージャーの仕事（スケジューラーの役割）

1. **注文受付**: お客様から料理の注文を受ける
2. **安全確認**: アレルギーや特別な要求をチェック
3. **調理順序決定**: どの料理をどの順番で作るか決める
4. **進行状況管理**: 各料理の調理状況を常に把握
5. **問題対応**: 調理中の問題や変更要求に対応
6. **完成確認**: 料理が完成したらお客様に提供

### なぜこんな複雑なシステムが必要なのか？

- **効率性**: 複数の注文を同時に、最適な順序で処理
- **安全性**: 危険な作業（熱い油を使う料理など）は事前確認
- **柔軟性**: 注文変更やキャンセルにも対応
- **品質管理**: すべての工程で品質をチェック
- **統計収集**: サービス改善のためのデータ収集

---

## 🔄 状態遷移編：注文の7つの段階

ツール実行スケジューラーでは、すべてのツールコール（注文）が7つの状態を経て処理されます。レストランの注文プロセスで説明しましょう。

### 1. **validating（検証中）** - 注文内容の確認
```
お客様: 「パスタをお願いします」
マネージャー: 「どちらのパスタでしょうか？アレルギーはありますか？」
→ 注文内容が正しく、実現可能か確認している状態
```

### 2. **scheduled（調理予定）** - 調理待ちリストに追加
```
マネージャー: 「承知いたしました。調理順が来たら始めます」
→ 問題なく、調理開始を待っている状態
```

### 3. **awaiting_approval（承認待ち）** - 特別確認が必要
```
お客様: 「このワインは大丈夫ですか？」
マネージャー: 「高級ワインですが、よろしいですか？」
→ 危険性や高コストな作業で、お客様の最終確認を待つ状態
```

### 4. **executing（実行中）** - 調理作業中
```
シェフ: 「パスタを茹でています」
→ 実際に調理（ツール実行）が進行している状態
```

### 5. **success（成功完了）** - 料理完成・提供済み
```
マネージャー: 「お待たせしました、パスタです」
→ 調理が完了し、お客様に提供完了した状態
```

### 6. **error（エラー）** - 調理失敗・問題発生
```
シェフ: 「申し訳ありません、材料が足りませんでした」
→ 予期しない問題で調理が失敗した状態
```

### 7. **cancelled（キャンセル）** - 注文取り消し
```
お客様: 「やはり、この注文はキャンセルで」
→ お客様の意思で注文が取り消された状態
```

### 状態の流れ図（レストラン版）

```
注文受付 → 検証中(validating) → 問題なし? 
                                      ↓ YES
                               調理予定(scheduled) → 実行中(executing) → 成功(success)
                                      ↓ NO                   ↓ 失敗
                               承認待ち(awaiting_approval) → エラー(error)
                                      ↓ キャンセル
                                 キャンセル済み(cancelled)
```

---

## 🍳 並行処理編：効率的な厨房運営

### 複数の注文を同時に処理する技術

普通のレストランでは一度に一つの料理しか作れませんが、このスケジューラーは**高性能な厨房システム**のように動きます。

### 並行実行（Parallel Execution）の例

```javascript
// 3つの注文が同時に入った場合
注文A: 「サラダをお願いします」        （調理時間: 5分）
注文B: 「ステーキをお願いします」      （調理時間: 20分）  
注文C: 「スープをお願いします」        （調理時間: 10分）

// スマートな処理順序
00:00 - ステーキの調理開始（一番時間がかかるため）
00:05 - スープの調理開始（ステーキ調理中に）
00:10 - サラダの調理開始（他の調理中に）
00:15 - サラダ完成・提供
00:15 - スープ完成・提供  
00:20 - ステーキ完成・提供
```

### 依存関係がある場合の順次実行

```javascript
// 一つの注文に複数の工程がある場合
注文: 「コースメニューをお願いします」
工程1: 前菜の準備 → 工程2: メイン料理 → 工程3: デザート

// この場合は順次実行（一つずつ順番に）
前菜完成 → メイン料理開始 → メイン完成 → デザート開始 → デザート完成
```

### スケジューラーが管理する要素

1. **実行キュー（調理待ちリスト）**: `requestQueue`
2. **現在の調理状況**: `toolCalls`配列
3. **同時実行制御**: 危険な作業は同時にしない
4. **リソース管理**: オーブンは一度に一つの料理だけ

---

## ✅ 承認フロー編：安全な調理のための確認システム

### なぜ承認（確認）が必要なのか？

レストランでも、以下のような場合は事前確認が必要です：

- **高級食材の使用**: 「トリュフを使いますが、追加料金は○○円です」
- **アレルギー対応**: 「ナッツアレルギーですが、この料理は大丈夫ですか？」
- **危険な調理法**: 「フランベ（火を使った演出）をしますが、よろしいですか？」
- **時間のかかる料理**: 「この料理は2時間かかりますが、お待ちいただけますか？」

### 承認モードの種類

#### 1. **YOLO Mode（なんでもOKモード）**
```
マネージャー: 「すべての注文、確認なしで調理開始！」
→ 開発・テスト環境で使用。すべて自動承認
```

#### 2. **Interactive Mode（対話確認モード）**
```
マネージャー: 「この料理は高級食材を使います。よろしいですか？」
お客様: 「はい、お願いします」
→ 人間が一つずつ判断して承認
```

### 承認が必要な操作の例（技術的内容）

- **ファイル削除**: 重要なファイルを削除する前に確認
- **システム変更**: 設定ファイルを変更する前に確認  
- **外部接続**: インターネットに接続する前に確認
- **権限変更**: ファイルの権限を変更する前に確認

### 承認フローの詳細処理

```typescript
// 承認が必要かチェック
const needsConfirmation = await tool.shouldConfirmExecute();

if (needsConfirmation) {
    // お客様に確認を求める
    this.setStatus('awaiting_approval', confirmationDetails);
    
    // お客様の回答を待つ
    const response = await waitForUserResponse();
    
    if (response === 'approved') {
        this.setStatus('scheduled'); // 調理開始
    } else {
        this.setStatus('cancelled'); // 注文キャンセル
    }
}
```

---

## 📝 エディター統合編：注文内容の修正システム

### 注文変更への対応

お客様から「やっぱりパスタの具材を変更したい」と言われたとき、どう対応しますか？

### 修正機能の流れ

1. **変更要求の受付**
```
お客様: 「パスタのトマトソースをクリームソースに変更できますか？」
```

2. **現在の注文内容の確認**
```typescript
// 元の注文: { pasta: "tomato", size: "large" }
const currentOrder = getCurrentContent(originalParams);
```

3. **変更内容の編集**
```typescript
// エディターで修正
const updatedOrder = await editWithPreferredEditor(currentOrder);
// 修正後: { pasta: "cream", size: "large" }
```

4. **変更点の差分表示（Diff表示）**
```diff
- パスタ: トマトソース（大盛り）
+ パスタ: クリームソース（大盛り）
```

5. **最終確認と適用**
```
マネージャー: 「クリームソースに変更ということですね。承知いたしました」
```

### 対応エディター種類

- **VSCode**: プロの開発者向け高機能エディター
- **Vim**: コマンドライン系の高速エディター  
- **システムデフォルト**: OSの標準テキストエディター

### インライン編集機能

お客様が直接、注文票に修正を書き込める機能：

```typescript
// お客様が直接内容を修正
const userModification = payload.newContent;

// システムが自動で差分を計算
const diff = Diff.createPatch(
    filePath,           // 注文票の場所
    originalContent,    // 元の注文内容
    userModification,   // 修正後の内容
    'Current',          // 元の注文
    'Proposed'          // 修正案
);
```

---

## 🚨 エラー処理編：問題発生時の対応マニュアル

### レストランで起こりうる問題（エラータイプ）

#### 1. **TOOL_NOT_REGISTERED（調理器具が見つからない）**
```
注文: 「ソフトクリームをお願いします」
問題: 「申し訳ございません、ソフトクリームマシンがありません」
対応: 代替案の提示（アイスクリームなど）
```

#### 2. **INVALID_TOOL_PARAMS（注文内容が不正）**
```
注文: 「パスタを-5人前お願いします」
問題: 「マイナスの人数は作れません」
対応: 正しい注文内容の確認
```

#### 3. **UNHANDLED_EXCEPTION（予期しない問題）**
```
調理中: 「停電で調理器具が全て止まりました」
問題: システム全体の予期しない障害
対応: システム復旧とお客様への説明
```

### エラーハンドリングの仕組み

```typescript
const createErrorResponse = (request, error, errorType) => ({
    callId: request.callId,           // 注文番号
    error,                           // 問題の詳細
    responseParts: [{                // お客様への回答
        functionResponse: {
            id: request.callId,
            name: request.name,
            response: { error: error.message }
        }
    }],
    resultDisplay: error.message,    // 表示用メッセージ
    errorType,                      // エラーの種類
});
```

### エラー発生時の状態変化

```
実行中(executing) → エラー発生 → エラー状態(error)
                               ↓
                        お客様への報告とお詫び
                               ↓
                           代替案の提示
```

---

## 📊 テレメトリー編：サービス改善のための統計システム

### レストランの営業統計に例えると

優秀なレストランマネージャーは、以下の情報を記録・分析します：

#### 収集する情報（ToolCallEvent）

1. **注文情報**
   - 注文番号（callId）
   - 料理名（toolName）
   - 注文時刻（startTime）

2. **処理時間情報**
   - 調理開始時刻
   - 調理完了時刻  
   - 合計調理時間（durationMs）

3. **結果情報**
   - 成功/失敗/キャンセル
   - エラーが発生した場合の原因
   - お客様の満足度

### 統計収集の実装

```typescript
// 各注文が完了するたびに統計を記録
for (const call of completedCalls) {
    logToolCall(this.config, new ToolCallEvent(call));
}

// 収集される情報の例
const toolCallEvent = {
    callId: "order-001",
    toolName: "pasta-carbonara", 
    status: "success",
    durationMs: 1200000,        // 20分（ミリ秒）
    startTime: 1640995200000,
    endTime: 1640996400000,
    customerSatisfaction: "high"
};
```

### 統計の活用方法

1. **効率性分析**
```
「パスタ類の平均調理時間は15分」
「ステーキ類は平均25分かかっている」
→ 調理時間の予測精度向上
```

2. **エラー分析**
```
「調理器具エラーが多い時間帯: 19:00-21:00」
「最も失敗が多い料理: 複雑なソース系」
→ 設備メンテナンスやスタッフ訓練の改善
```

3. **顧客満足度追跡**
```
「承認待ち時間が長いとキャンセル率が上がる」
「リアルタイム進捗表示で満足度向上」
→ ユーザーエクスペリエンスの改善
```

---

## 🔧 技術詳細編：内部の仕組み

### CoreToolSchedulerクラスの内部構造

```typescript
export class CoreToolScheduler {
    private toolRegistry: ToolRegistry;      // 利用可能な調理器具リスト
    private toolCalls: ToolCall[] = [];     // 現在処理中の注文リスト
    private requestQueue: Array<...> = [];  // 注文待ちキュー
    private isFinalizingToolCalls = false;  // 会計処理中フラグ
    private isScheduling = false;           // 注文受付処理中フラグ
    
    // 各種ハンドラー（イベント処理関数）
    private outputUpdateHandler?: OutputUpdateHandler;           // 進捗報告
    private onAllToolCallsComplete?: AllToolCallsCompleteHandler; // 全注文完了時
    private onToolCallsUpdate?: ToolCallsUpdateHandler;          // 状況更新時
}
```

### 主要メソッドの説明

#### 1. **schedule()** - 注文受付メイン処理
```typescript
async schedule(request, signal): Promise<void> {
    // 他の注文処理中なら待機キューに追加
    if (this.isRunning() || this.isScheduling) {
        return this.queueRequest(request, signal);
    }
    
    // 実際の注文処理開始
    return this._schedule(request, signal);
}
```

#### 2. **setStatusInternal()** - 注文状態の変更
```typescript
private setStatusInternal(callId: string, newStatus: Status, auxiliaryData?: unknown) {
    // 該当する注文を見つけて状態変更
    this.toolCalls = this.toolCalls.map((call) => {
        if (call.request.callId === callId) {
            // 状態に応じた処理
            switch (newStatus) {
                case 'success': 
                    return { ...call, status: 'success', response: auxiliaryData };
                case 'error':
                    return { ...call, status: 'error', response: auxiliaryData };
                // その他の状態変更...
            }
        }
        return call;
    });
    
    // 状況更新の通知
    this.notifyToolCallsUpdate();
    this.checkAndNotifyCompletion();
}
```

#### 3. **attemptExecutionOfScheduledCalls()** - 調理開始処理
```typescript
private attemptExecutionOfScheduledCalls(signal: AbortSignal) {
    // すべての注文が準備完了かチェック
    const allCallsReady = this.toolCalls.every(call => 
        call.status === 'scheduled' || // 調理予定
        call.status === 'completed'    // 完了済み
    );
    
    if (allCallsReady) {
        // 調理予定の注文を一斉に開始
        const readyToCook = this.toolCalls.filter(call => 
            call.status === 'scheduled'
        );
        
        readyToCook.forEach(call => {
            this.setStatusInternal(call.request.callId, 'executing');
            this.startCooking(call, signal); // 実際の調理開始
        });
    }
}
```

---

## 🌟 実際の使用例：具体的なシナリオ

### シナリオ1: 複数のファイル操作

```
ユーザー: 「3つのファイルを読み込んで、内容を統合したファイルを作成して」

1. validating: "file1.txt読み込み" - 内容確認中
2. validating: "file2.txt読み込み" - 内容確認中  
3. validating: "file3.txt読み込み" - 内容確認中
   ↓ 全て問題なし
4. scheduled: 3つの読み込み処理が調理待ち
   ↓ 並行実行開始
5. executing: file1.txt読み込み中...
   executing: file2.txt読み込み中...  
   executing: file3.txt読み込み中...
   ↓ 順次完了
6. success: file1.txt読み込み完了
   success: file2.txt読み込み完了
   success: file3.txt読み込み完了
   ↓ 次の処理開始
7. validating: "統合ファイル作成" - 内容確認中
8. scheduled: 統合ファイル作成が調理待ち
9. executing: 統合ファイル作成中...
10. success: merged.txt作成完了！
```

### シナリオ2: 危険な操作での承認フロー

```
ユーザー: 「システムの設定ファイルを更新して」

1. validating: "config.json更新" - 内容確認中
   ↓ 危険な操作と判定
2. awaiting_approval: 承認待ち
   「重要な設定ファイルを変更します。よろしいですか？」
   
   [差分表示]
   - timeout: 30
   + timeout: 60
   - debug: false  
   + debug: true
   
   ユーザー選択:
   - [承認] → scheduled状態へ
   - [キャンセル] → cancelled状態へ
   - [修正] → エディターで内容修正
```

---

## 🎯 まとめ：なぜこのシステムが優秀なのか

### 1. **安全性の確保**
- 危険な操作は必ず事前確認
- エラー発生時の適切な対応
- ユーザーの意図しない変更の防止

### 2. **効率性の最大化**
- 複数作業の同時並行処理
- 依存関係を考慮した最適な実行順序
- リソースの効率的な活用

### 3. **柔軟性の提供**
- リアルタイムでの注文変更
- 様々なエディターでの内容修正
- キャンセルや中断への対応

### 4. **透明性の維持**
- すべての処理状況をリアルタイム表示
- 詳細な実行ログの記録
- 問題発生時の明確なエラー説明

### 5. **継続的な改善**
- 処理統計の収集と分析
- パフォーマンス改善のためのデータ活用
- ユーザーエクスペリエンス向上

---

## 📚 用語集

| 技術用語 | 日常的な説明 | レストランでの例え |
|---------|-------------|------------------|
| ToolScheduler | 作業管理システム | 注文管理マネージャー |
| ToolCall | 作業の依頼 | お客様からの注文 |
| Status | 作業の状態 | 注文の処理段階 |
| Validation | 内容の確認 | 注文内容のチェック |
| Execution | 作業の実行 | 料理の調理 |
| Approval | 事前確認 | 特別メニューの承認 |
| Error Handling | 問題対応 | 調理失敗時の対応 |
| Telemetry | 統計収集 | 営業データの記録 |
| Queue | 待ち行列 | 注文待ちリスト |
| Concurrent | 同時並行 | 複数料理の同時調理 |

---

このように、ツール実行スケジューラーは高級レストランの優秀なマネージャーのような役割を果たし、複雑なタスクを安全かつ効率的に管理する、非常に高度なシステムなのです。

文系の方でも、この例えを通じて理解していただけたでしょうか？技術の世界も、日常生活の延長にあることがお分かりいただけたと思います。