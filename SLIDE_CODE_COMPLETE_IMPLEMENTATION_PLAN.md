# 🎯 Slide Code - スライド生成特化CLIエージェント 完全実装計画書

## 📋 プロジェクト概要

### プロジェクト名: **Slide Code**
- **目的**: スライド生成に特化したCLIエージェント
- **ベース**: Gemini CLIのアーキテクチャを参考とした独立プロジェクト
- **特徴**: コード修正機能を排除し、スライド生成のみに集中した軽量エージェント

### 核となる価値提案
- **🚀 瞬時のスライド生成**: ワンコマンドで完成したスライドを生成
- **🔧 自動セットアップ**: 依存関係の自動インストールと設定
- **📊 3つの出力形式**: Marp（MD）、PPTX、Google Slideに対応
- **🎨 プロンプトテンプレート**: カスタムプロンプトによる個人ブランディング
- **💻 完全ローカル**: セキュリティとプライバシーを重視

---

## 🏗️ Gemini CLIから継承すべき核となる構造

### 必須コンポーネントの詳細分析

#### 1. ターミナル入力システム（InputPrompt.tsx）
**参照**: `/packages/cli/src/ui/components/InputPrompt.tsx`

Gemini CLIの核となるターミナル入力システムをSlide Codeに適用：

```typescript
// Slide Code版 InputPrompt実装のベース
interface SlideInputPromptProps {
  buffer: TextBuffer;
  onSubmit: (value: string) => void;
  config: SlideConfig;
  slashCommands: readonly SlideCommand[];
  placeholder?: string;
  focus?: boolean;
  onSlideGeneration: (params: SlideParams) => void;
}

// 主要機能：
// - リアルタイムテキスト入力
// - スラッシュコマンド補完（/slide, /template など）
// - 入力履歴管理
// - キーボードショートカット
// - Vim モードサポート（オプション）
```

#### 2. メインアプリケーション構造（App.tsx）
**参照**: `/packages/cli/src/ui/App.tsx`

React + Ink を使用したターミナルUI構造：

```typescript
// Slide Code版 App実装のベース
const SlideApp = ({ config, settings }: SlideAppProps) => {
  const [streamingState, setStreamingState] = useState<SlideStreamingState>();
  const [currentSlide, setCurrentSlide] = useState<SlideData | null>(null);
  
  return (
    <Box flexDirection="column" height="100%">
      <SlideHeader version={version} />
      <SlideContent 
        slides={currentSlide}
        streamingState={streamingState}
      />
      <SlideInputPrompt 
        onSubmit={handleSlideGeneration}
        config={config}
      />
      <SlideFooter status={streamingState} />
    </Box>
  );
};

// 主要機能：
// - ストリーミング状態管理
// - スライドプレビュー表示
// - プログレス表示
// - エラーハンドリング
// - レスポンシブ端末サイズ対応
```

#### 3. ツール実行基盤（CoreToolScheduler）
**参照**: `/packages/core/src/core/coreToolScheduler.ts`

ツール実行とストリーミングの中核システム：

```typescript
// Slide Code版 ToolScheduler実装のベース
export class SlideToolScheduler {
  async executeSlideGeneration(
    params: SlideGeneratorParams,
    updateOutput?: (output: string) => void
  ): Promise<SlideResult> {
    
    // ツール実行状態の管理
    const toolCall: ExecutingSlideToolCall = {
      status: 'executing',
      toolName: 'slide-generator',
      params,
      startTime: Date.now()
    };
    
    // ストリーミング出力の処理
    return await this.runSlideGeneration(toolCall, updateOutput);
  }
}

// 主要機能：
// - ツール実行の状態管理（validating, executing, success, error）
// - リアルタイム出力更新
// - エラーハンドリングと復旧
// - タイムアウト管理
// - 結果のフォーマット
```

### 継承すべきGemini CLI設計パターン

#### A. Ink/Reactターミナル描画システム
```typescript
// ターミナルUIのレイアウト構造
<Box flexDirection="column">
  <Header />        // タイトル、ステータス
  <MainContent />   // スライド内容、プレビュー
  <InputArea />     // コマンド入力欄
  <Footer />        // プログレス、ヘルプ
</Box>
```

#### B. ストリーミングレスポンス処理
```typescript
// リアルタイム更新システム
const useSlideStream = () => {
  const [output, setOutput] = useState('');
  
  const updateOutput = useCallback((chunk: string) => {
    setOutput(prev => prev + chunk);
  }, []);
  
  return { output, updateOutput };
};
```

#### C. 設定管理システム
```typescript
// 設定ファイル構造（Gemini CLIと同様）
~/.slide-code/
├── config.json          // 基本設定
├── settings.json         // ユーザー設定
├── templates/           // カスタムテンプレート
└── auth/               // 認証情報
```

---

## 🏗️ プロジェクト構造

### モノレポ構成（Gemini CLIベース）
```
slide-code/
├── packages/
│   ├── cli/                    # CLIアプリケーション
│   │   ├── src/
│   │   │   ├── slide.tsx      # メインエントリーポイント
│   │   │   ├── ui/
│   │   │   │   ├── App.tsx    # メインアプリコンポーネント
│   │   │   │   ├── components/
│   │   │   │   │   ├── SlidePreview.tsx
│   │   │   │   │   ├── FormatSelector.tsx
│   │   │   │   │   ├── TemplateSelector.tsx
│   │   │   │   │   └── ProgressIndicator.tsx
│   │   │   │   └── commands/
│   │   │   │       ├── slideCommand.ts
│   │   │   │       ├── templateCommand.ts
│   │   │   │       └── configCommand.ts
│   │   │   ├── config/
│   │   │   │   ├── config.ts
│   │   │   │   └── settings.ts
│   │   │   └── utils/
│   │   │       ├── installer.ts
│   │   │       └── launcher.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── core/                   # コアロジック
│   │   ├── src/
│   │   │   ├── tools/
│   │   │   │   ├── slide-generator.ts
│   │   │   │   ├── marp-exporter.ts
│   │   │   │   ├── pptx-exporter.ts
│   │   │   │   ├── google-slide-exporter.ts
│   │   │   │   └── template-manager.ts
│   │   │   ├── services/
│   │   │   │   ├── dependency-installer.ts
│   │   │   │   ├── app-launcher.ts
│   │   │   │   └── prompt-engine.ts
│   │   │   ├── templates/
│   │   │   │   ├── template-registry.ts
│   │   │   │   └── prompts/
│   │   │   │       ├── professional.ts
│   │   │   │       ├── creative.ts
│   │   │   │       └── academic.ts
│   │   │   └── types/
│   │   │       └── slide-types.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── test-utils/             # テストユーティリティ
├── scripts/
│   ├── setup.sh              # 初期セットアップスクリプト
│   ├── install-deps.js       # 依存関係インストール
│   └── build.js             # ビルドスクリプト
├── templates/               # スライドテンプレート
├── package.json             # ルートパッケージ
├── lerna.json              # モノレポ設定
├── tsconfig.json           # TypeScript設定
└── README.md
```

---

## 🚀 オンボーディングシステム設計

### 初回起動時の完全ガイドフロー

#### 1. ウェルカムスクリーン
```typescript
// packages/cli/src/ui/components/WelcomeScreen.tsx
export const WelcomeScreen: React.FC = () => {
  return (
    <Box flexDirection="column" alignItems="center" padding={2}>
      <Text color="cyan" bold>
        🎯 Slide Code へようこそ！
      </Text>
      <Text>スライド生成特化CLIエージェント</Text>
      <Box marginTop={1}>
        <Text color="dim">
          AIを使って瞬時にプロフェッショナルなスライドを生成します
        </Text>
      </Box>
      
      <Box flexDirection="column" marginTop={2}>
        <Text color="green">✅ 3つの出力形式対応</Text>
        <Text color="dim">   📄 Marp（マークダウン）</Text>
        <Text color="dim">   📊 PowerPoint（PPTX）</Text>
        <Text color="dim">   🌐 Google Slide</Text>
        
        <Text color="green" marginTop={1}>✅ 自動セットアップ</Text>
        <Text color="dim">   必要な依存関係を自動インストール</Text>
        
        <Text color="green" marginTop={1}>✅ プロンプトテンプレート</Text>
        <Text color="dim">   カスタムプロンプトでブランディング</Text>
      </Box>
      
      <Text color="yellow" marginTop={2}>
        Press Enter to start setup...
      </Text>
    </Box>
  );
};
```

#### 2. セットアップウィザード
```typescript
// 段階的セットアップフロー
const SETUP_STEPS = [
  {
    id: 'api-key',
    title: 'Gemini API設定',
    description: 'AI機能を使用するためのAPIキーを設定します',
    action: 'setApiKey'
  },
  {
    id: 'output-preference',
    title: '出力形式の選択', 
    description: 'デフォルトの出力形式を選択してください',
    action: 'selectOutputFormat'
  },
  {
    id: 'dependencies',
    title: '依存関係のインストール',
    description: 'PptxGenJSとClaspを自動インストールします',
    action: 'installDependencies'
  },
  {
    id: 'demo',
    title: 'デモスライド生成',
    description: '最初のスライドを生成してみましょう！',
    action: 'generateDemo'
  }
];

export const SetupWizard: React.FC = () => {
  const [currentStep, setCurrentStep] = useState(0);
  const [setupData, setSetupData] = useState<SetupData>({});
  
  return (
    <Box flexDirection="column">
      <SetupHeader 
        step={currentStep + 1}
        totalSteps={SETUP_STEPS.length}
        title={SETUP_STEPS[currentStep].title}
      />
      
      <SetupStepContent 
        step={SETUP_STEPS[currentStep]}
        data={setupData}
        onUpdate={setSetupData}
        onNext={() => setCurrentStep(prev => prev + 1)}
      />
    </Box>
  );
};
```

#### 3. API設定ガイド
```typescript
// APIキー設定の詳細ガイド
export const ApiKeySetup: React.FC = ({ onComplete }) => {
  const [apiKey, setApiKey] = useState('');
  const [isValidating, setIsValidating] = useState(false);
  
  return (
    <Box flexDirection="column">
      <Text color="cyan" bold>🔑 Gemini API設定</Text>
      <Box marginTop={1}>
        <Text>AIを使用するにはGemini APIキーが必要です</Text>
      </Box>
      
      <Box flexDirection="column" marginTop={2} paddingX={2}>
        <Text color="yellow">📋 手順:</Text>
        <Text>1. https://aistudio.google.com でGoogleアカウントにログイン</Text>
        <Text>2. 「Get API Key」をクリック</Text>
        <Text>3. 新しいAPIキーを作成</Text>
        <Text>4. 下記にAPIキーを入力してください</Text>
      </Box>
      
      <Box marginTop={2}>
        <Text color="dim">APIキー: </Text>
        <TextInput
          value={apiKey}
          onChange={setApiKey}
          placeholder="AIza..."
          mask="*"
        />
      </Box>
      
      {isValidating && (
        <Box marginTop={1}>
          <Spinner type="dots" />
          <Text color="dim"> APIキーを検証中...</Text>
        </Box>
      )}
      
      <Text color="dim" marginTop={1}>
        💡 APIキーは暗号化されて保存され、外部に送信されません
      </Text>
    </Box>
  );
};
```

#### 4. 初回デモスライド生成
```typescript
// 最初のスライド生成体験
export const DemoSlideGeneration: React.FC = () => {
  const [isGenerating, setIsGenerating] = useState(false);
  const [generatedSlide, setGeneratedSlide] = useState<string | null>(null);
  
  const generateDemo = async () => {
    setIsGenerating(true);
    
    // デモスライド生成
    const result = await slideGenerator.generate({
      topic: 'Slide Codeへようこそ',
      slides_count: 5,
      style: 'creative',
      output_format: 'marp',
      auto_open: false,
    });
    
    setGeneratedSlide(result.content);
    setIsGenerating(false);
  };
  
  return (
    <Box flexDirection="column">
      <Text color="cyan" bold>🎬 デモスライド生成</Text>
      <Text marginTop={1}>
        最初のスライドを生成してSlide Codeの威力を体験しましょう！
      </Text>
      
      {!isGenerating && !generatedSlide && (
        <Box marginTop={2}>
          <Text color="green">
            Press Enter to generate "Slide Codeへようこそ" slides
          </Text>
        </Box>
      )}
      
      {isGenerating && (
        <Box marginTop={2}>
          <Spinner type="dots" />
          <Text color="yellow"> AIがスライドを生成中...</Text>
          <Text color="dim" marginTop={1}>
            ✨ プロフェッショナルなスライドを作成しています
          </Text>
        </Box>
      )}
      
      {generatedSlide && (
        <Box flexDirection="column" marginTop={2}>
          <Text color="green" bold>🎉 生成完了！</Text>
          <Box marginTop={1} paddingX={2} borderStyle="single">
            <Text>{generatedSlide.slice(0, 200)}...</Text>
          </Box>
          
          <Text color="cyan" marginTop={1}>
            完全なスライドは ~/slide-code-demo.md に保存されました
          </Text>
          
          <Text color="dim" marginTop={1}>
            次回からは以下のコマンドで素早く生成できます：
          </Text>
          <Text color="yellow">
            slide new "あなたのトピック"
          </Text>
        </Box>
      )}
    </Box>
  );
};
```

### オンボーディング完了後の案内

```typescript
// セットアップ完了画面
export const SetupComplete: React.FC = () => {
  return (
    <Box flexDirection="column" alignItems="center" padding={2}>
      <Text color="green" bold>🎉 セットアップ完了！</Text>
      
      <Box flexDirection="column" marginTop={2} alignItems="center">
        <Text color="cyan">Slide Code の準備が整いました</Text>
        
        <Box flexDirection="column" marginTop={2}>
          <Text color="yellow" bold>📚 基本コマンド:</Text>
          
          <Box paddingLeft={2}>
            <Text color="green">slide new "トピック"</Text>
            <Text color="dim">  → 新しいスライドを生成</Text>
            
            <Text color="green">slide pptx "マーケティング戦略"</Text>
            <Text color="dim">  → PowerPoint形式で生成</Text>
            
            <Text color="green">slide google "プロジェクト報告"</Text>
            <Text color="dim">  → Google Slideで生成</Text>
            
            <Text color="green">slide template keitaro-slide-prompt "DX戦略"</Text>
            <Text color="dim">  → プロンプトテンプレート使用</Text>
            
            <Text color="green">slide help</Text>
            <Text color="dim">  → ヘルプとコマンド一覧</Text>
          </Box>
        </Box>
        
        <Box marginTop={2}>
          <Text color="cyan">💡 ヒント: </Text>
          <Text>生成後、PowerPointやブラウザが自動で開きます</Text>
        </Box>
        
        <Text color="dim" marginTop={2}>
          Press Enter to start using Slide Code...
        </Text>
      </Box>
    </Box>
  );
};
```

---

## 💻 ターミナルUIの詳細実装

### Ink/React ベースのコンポーネント設計

#### 1. リアルタイム入力システム
```typescript
// packages/cli/src/ui/components/SlideInputPrompt.tsx
import React, { useState, useCallback } from 'react';
import { Box, Text, useInput } from 'ink';
import TextInput from 'ink-text-input';

export const SlideInputPrompt: React.FC<SlideInputPromptProps> = ({
  onSubmit,
  placeholder = "💬 Type your slide topic or use /slide, /template commands...",
  isGenerating = false,
}) => {
  const [query, setQuery] = useState('');
  const [suggestions, setSuggestions] = useState<string[]>([]);
  
  // スラッシュコマンドの自動補完
  const handleQueryChange = useCallback((value: string) => {
    setQuery(value);
    
    if (value.startsWith('/')) {
      const matchingCommands = SLIDE_COMMANDS
        .filter(cmd => cmd.startsWith(value))
        .slice(0, 5);
      setSuggestions(matchingCommands);
    } else {
      setSuggestions([]);
    }
  }, []);
  
  const handleSubmit = useCallback(() => {
    if (query.trim()) {
      onSubmit(query);
      setQuery('');
      setSuggestions([]);
    }
  }, [query, onSubmit]);
  
  // Enterキーでの送信
  useInput((input, key) => {
    if (key.return && query.trim()) {
      handleSubmit();
    }
  });
  
  return (
    <Box flexDirection="column" marginTop={1}>
      {/* 入力欄 */}
      <Box>
        <Text color="cyan">🎯 </Text>
        <Box flex={1}>
          <TextInput
            value={query}
            onChange={handleQueryChange}
            placeholder={placeholder}
            disabled={isGenerating}
          />
        </Box>
        
        {isGenerating && (
          <Text color="yellow"> ⏳ Generating...</Text>
        )}
      </Box>
      
      {/* コマンド補完表示 */}
      {suggestions.length > 0 && (
        <Box flexDirection="column" marginTop={1} paddingLeft={3}>
          <Text color="dim">Suggestions:</Text>
          {suggestions.map((suggestion, i) => (
            <Text key={i} color={i === 0 ? 'green' : 'dim'}>
              {i === 0 ? '→ ' : '  '}{suggestion}
            </Text>
          ))}
        </Box>
      )}
    </Box>
  );
};

const SLIDE_COMMANDS = [
  '/slide new',
  '/slide pptx', 
  '/slide google',
  '/slide marp',
  '/slide template',
  '/template list',
  '/config show',
  '/help',
];
```

#### 2. ストリーミングコンテンツ表示
```typescript
// packages/cli/src/ui/components/SlideStreamingDisplay.tsx
export const SlideStreamingDisplay: React.FC = ({
  content,
  isStreaming,
  progress,
}) => {
  const [displayedContent, setDisplayedContent] = useState('');
  
  // タイプライター効果でのコンテンツ表示
  useEffect(() => {
    if (!isStreaming) {
      setDisplayedContent(content);
      return;
    }
    
    let currentIndex = 0;
    const interval = setInterval(() => {
      if (currentIndex < content.length) {
        setDisplayedContent(content.slice(0, currentIndex + 1));
        currentIndex++;
      } else {
        clearInterval(interval);
      }
    }, 10); // 高速タイピング効果
    
    return () => clearInterval(interval);
  }, [content, isStreaming]);
  
  return (
    <Box flexDirection="column" padding={1}>
      {progress && (
        <Box marginBottom={1}>
          <Text color="yellow">
            📊 Progress: {progress.current}/{progress.total} - {progress.stage}
          </Text>
        </Box>
      )}
      
      <Box borderStyle="round" padding={1}>
        <Text>{displayedContent}</Text>
        {isStreaming && (
          <Text color="green">▊</Text> // カーソル点滅
        )}
      </Box>
      
      {!isStreaming && content && (
        <Text color="dim" marginTop={1}>
          ✅ Generation complete! Files saved and applications launched.
        </Text>
      )}
    </Box>
  );
};
```

#### 3. プログレス表示システム
```typescript
// packages/cli/src/ui/components/SlideProgressIndicator.tsx
export const SlideProgressIndicator: React.FC = ({
  stages,
  currentStage,
  currentProgress,
}) => {
  return (
    <Box flexDirection="column" paddingX={2}>
      <Text color="cyan" bold>🎯 Slide Generation Progress</Text>
      
      <Box flexDirection="column" marginTop={1}>
        {stages.map((stage, index) => {
          const isCompleted = index < currentStage;
          const isCurrent = index === currentStage;
          const isUpcoming = index > currentStage;
          
          return (
            <Box key={stage.id} marginBottom={1}>
              <Text color={
                isCompleted ? 'green' : 
                isCurrent ? 'yellow' : 'dim'
              }>
                {isCompleted ? '✅' : isCurrent ? '⏳' : '⏸️'} {stage.name}
              </Text>
              
              {isCurrent && currentProgress && (
                <Box marginLeft={3}>
                  <ProgressBar 
                    current={currentProgress.value}
                    total={currentProgress.total}
                    width={30}
                  />
                  <Text color="dim" marginLeft={1}>
                    {stage.description}
                  </Text>
                </Box>
              )}
            </Box>
          );
        })}
      </Box>
    </Box>
  );
};

const ProgressBar: React.FC = ({ current, total, width }) => {
  const filled = Math.floor((current / total) * width);
  const empty = width - filled;
  
  return (
    <Text>
      <Text color="green">{'█'.repeat(filled)}</Text>
      <Text color="dim">{'░'.repeat(empty)}</Text>
      <Text color="yellow"> {Math.round((current / total) * 100)}%</Text>
    </Text>
  );
};
```

#### 4. 形式選択インターフェース
```typescript
// packages/cli/src/ui/components/FormatSelector.tsx
import SelectInput from 'ink-select-input';

export const FormatSelector: React.FC = ({ onSelect }) => {
  const formatOptions = [
    {
      label: '📄 Marp (Markdown) - ブラウザ/エディタで開く',
      value: 'marp'
    },
    {
      label: '📊 PowerPoint (PPTX) - PowerPointで開く',
      value: 'pptx'
    },
    {
      label: '🌐 Google Slide - ブラウザで開く',
      value: 'google'
    }
  ];
  
  return (
    <Box flexDirection="column">
      <Text color="cyan" bold>📝 出力形式を選択してください:</Text>
      
      <Box marginTop={1}>
        <SelectInput
          items={formatOptions}
          onSelect={({ value }) => onSelect(value)}
        />
      </Box>
      
      <Text color="dim" marginTop={1}>
        💡 生成後、対応するアプリケーションが自動で開きます
      </Text>
    </Box>
  );
};
```

---

## 🎯 コア機能実装詳細

### 1. メインエントリーポイント（slide.tsx）

```typescript
// packages/cli/src/slide.tsx
import React from 'react';
import { render } from 'ink';
import { SlideApp } from './ui/App.js';
import { loadSlideConfig } from './config/config.js';
import { ensureDependencies } from './utils/installer.js';

export async function main() {
  // 初回起動時の依存関係確認とインストール
  await ensureDependencies();
  
  const config = await loadSlideConfig();
  
  const { waitUntilExit } = render(
    <React.StrictMode>
      <SlideApp config={config} />
    </React.StrictMode>
  );
  
  await waitUntilExit();
}

if (import.meta.url === `file://${process.argv[1]}`) {
  main().catch((error) => {
    console.error('Slide Code error:', error);
    process.exit(1);
  });
}
```

### 2. スライド生成ツール（slide-generator.ts）

```typescript
// packages/core/src/tools/slide-generator.ts
import {
  BaseDeclarativeTool,
  BaseToolInvocation,
  Kind,
  ToolResult,
} from './base-tool.js';

export interface SlideGeneratorParams {
  topic: string;
  style?: 'professional' | 'creative' | 'academic' | 'casual';
  slides_count?: number;
  output_format: 'marp' | 'pptx' | 'google';
  template?: string; // プロンプトテンプレート名
  auto_open?: boolean; // 生成後の自動オープン
}

export class SlideGeneratorTool extends BaseDeclarativeTool<
  SlideGeneratorParams,
  ToolResult
> {
  constructor() {
    super(
      'slide-generator',
      'スライド生成',
      'AIを使用してプレゼンテーションスライドを生成し、指定された形式で出力します',
      Kind.SlideGeneration, // カスタムKind
      {
        type: 'object',
        properties: {
          topic: {
            type: 'string',
            description: 'プレゼンテーションのトピック',
          },
          style: {
            type: 'string',
            enum: ['professional', 'creative', 'academic', 'casual'],
            default: 'professional',
          },
          slides_count: {
            type: 'number',
            minimum: 3,
            maximum: 50,
            default: 10,
          },
          output_format: {
            type: 'string',
            enum: ['marp', 'pptx', 'google'],
            description: '出力形式',
          },
          template: {
            type: 'string',
            description: 'プロンプトテンプレート名（例: keitaro-slide-prompt）',
          },
          auto_open: {
            type: 'boolean',
            default: true,
            description: '生成後にアプリケーションを自動で開く',
          },
        },
        required: ['topic', 'output_format'],
      },
      true, // isOutputMarkdown
      true, // canUpdateOutput
    );
  }

  protected createInvocation(params: SlideGeneratorParams) {
    return new SlideGeneratorInvocation(params);
  }
}
```

### 3. PPTX出力ツール（pptx-exporter.ts）

```typescript
// packages/core/src/tools/pptx-exporter.ts
import { exec } from 'node:child_process';
import { promisify } from 'node:util';
import { ensurePptxGenJS } from '../services/dependency-installer.js';
import { launchPowerPoint } from '../services/app-launcher.js';

const execAsync = promisify(exec);

export class PptxExporter {
  async export(slideData: SlideData, outputPath: string): Promise<string> {
    // PptxGenJSが未インストールの場合、自動インストール
    await ensurePptxGenJS();
    
    // PptxGenJSを使用してPPTX生成
    const pptx = await this.generatePptx(slideData);
    
    // ファイル保存
    const filePath = `${outputPath}/${slideData.title}.pptx`;
    await pptx.writeFile({ fileName: filePath });
    
    // PowerPointを自動起動
    if (slideData.autoOpen) {
      await launchPowerPoint(filePath);
    }
    
    return filePath;
  }

  private async generatePptx(slideData: SlideData) {
    const PptxGenJS = await import('pptxgenjs');
    const pptx = new PptxGenJS.default();
    
    // スライドマスターの設定
    pptx.defineSlideMaster({
      title: 'SLIDE_CODE_MASTER',
      background: { color: slideData.style.backgroundColor },
      objects: [
        {
          placeholder: {
            options: { name: 'title', type: 'title' },
            text: '{{title}}',
          },
        },
        {
          placeholder: {
            options: { name: 'body', type: 'body' },
            text: '{{content}}',
          },
        },
      ],
    });
    
    // 各スライドの生成
    for (const slide of slideData.slides) {
      const slideObj = pptx.addSlide({ masterName: 'SLIDE_CODE_MASTER' });
      
      slideObj.addText(slide.title, {
        placeholder: 'title',
        fontSize: 28,
        bold: true,
        color: slideData.style.titleColor,
      });
      
      slideObj.addText(slide.content.join('\n'), {
        placeholder: 'body',
        fontSize: 18,
        color: slideData.style.textColor,
      });
      
      // 画像がある場合の処理
      if (slide.image) {
        slideObj.addImage({
          path: slide.image,
          x: '50%',
          y: '60%',
          w: '40%',
          h: '30%',
        });
      }
    }
    
    return pptx;
  }
}
```

### 4. Google Slideエクスポーター（google-slide-exporter.ts）

```typescript
// packages/core/src/tools/google-slide-exporter.ts
import { ensureClasp } from '../services/dependency-installer.js';
import { GoogleAuth } from 'google-auth-library';
import { slides_v1, google } from 'googleapis';

export class GoogleSlideExporter {
  async export(slideData: SlideData): Promise<string> {
    // Claspが未インストールの場合、自動インストール
    await ensureClasp();
    
    // Google認証の確認と設定
    const auth = await this.ensureGoogleAuth();
    
    // Google Slides API初期化
    const slides = google.slides({ version: 'v1', auth });
    
    // 新しいプレゼンテーション作成
    const presentation = await slides.presentations.create({
      requestBody: {
        title: slideData.title,
      },
    });
    
    const presentationId = presentation.data.presentationId!;
    
    // 既存のスライドを削除（テンプレートスライドを除去）
    await this.clearDefaultSlides(slides, presentationId);
    
    // スライドデータからGoogle Slidesを生成
    await this.populateSlides(slides, presentationId, slideData);
    
    // プレゼンテーションURLを返す
    const url = `https://docs.google.com/presentation/d/${presentationId}/edit`;
    
    // ブラウザで自動オープン
    if (slideData.autoOpen) {
      const open = await import('open');
      await open.default(url);
    }
    
    return url;
  }
  
  private async ensureGoogleAuth(): Promise<GoogleAuth> {
    // Google認証の設定と確認
    // 初回の場合は認証フローを実行
    return new GoogleAuth({
      scopes: ['https://www.googleapis.com/auth/presentations'],
      keyFilename: process.env.GOOGLE_APPLICATION_CREDENTIALS,
    });
  }
  
  private async populateSlides(
    slides: slides_v1.Slides,
    presentationId: string,
    slideData: SlideData
  ) {
    const requests = [];
    
    for (let i = 0; i < slideData.slides.length; i++) {
      const slide = slideData.slides[i];
      
      // スライド作成リクエスト
      requests.push({
        createSlide: {
          objectId: `slide_${i}`,
          slideLayoutReference: {
            predefinedLayout: 'TITLE_AND_BODY',
          },
        },
      });
      
      // タイトル追加リクエスト
      requests.push({
        insertText: {
          objectId: `slide_${i}`,
          text: slide.title,
        },
      });
      
      // コンテンツ追加リクエスト
      requests.push({
        insertText: {
          objectId: `slide_${i}`,
          text: slide.content.join('\n'),
        },
      });
    }
    
    // バッチリクエスト実行
    await slides.presentations.batchUpdate({
      presentationId,
      requestBody: { requests },
    });
  }
}
```

---

## 🔧 依存関係自動インストールシステム

### 依存関係インストーラー（dependency-installer.ts）

```typescript
// packages/core/src/services/dependency-installer.ts
import { exec } from 'node:child_process';
import { promisify } from 'node:util';
import { existsSync } from 'node:fs';
import { join } from 'node:path';

const execAsync = promisify(exec);

export class DependencyInstaller {
  private static installedDeps = new Set<string>();
  
  async ensurePptxGenJS(): Promise<void> {
    if (DependencyInstaller.installedDeps.has('pptxgenjs')) return;
    
    try {
      // 既存インストール確認
      await import('pptxgenjs');
      DependencyInstaller.installedDeps.add('pptxgenjs');
    } catch {
      console.log('📦 PptxGenJSをインストール中...');
      await execAsync('npm install pptxgenjs --save');
      DependencyInstaller.installedDeps.add('pptxgenjs');
      console.log('✅ PptxGenJSインストール完了');
    }
  }
  
  async ensureClasp(): Promise<void> {
    if (DependencyInstaller.installedDeps.has('clasp')) return;
    
    try {
      await execAsync('clasp --version');
      DependencyInstaller.installedDeps.add('clasp');
    } catch {
      console.log('📦 Google Apps Script CLIをインストール中...');
      await execAsync('npm install -g @google/clasp');
      DependencyInstaller.installedDeps.add('clasp');
      console.log('✅ Claspインストール完了');
      
      // 初回認証の案内
      console.log('🔐 Google認証が必要です。以下のコマンドを実行してください:');
      console.log('clasp login');
    }
  }
  
  async ensureMarp(): Promise<void> {
    if (DependencyInstaller.installedDeps.has('marp')) return;
    
    try {
      await execAsync('marp --version');
      DependencyInstaller.installedDeps.add('marp');
    } catch {
      console.log('📦 Marpをインストール中...');
      await execAsync('npm install -g @marp-team/marp-cli');
      DependencyInstaller.installedDeps.add('marp');
      console.log('✅ Marpインストール完了');
    }
  }
  
  async ensureAllDependencies(): Promise<void> {
    await Promise.all([
      this.ensurePptxGenJS(),
      this.ensureClasp(),
      this.ensureMarp(),
    ]);
  }
}

// エクスポート関数
export const ensurePptxGenJS = () => new DependencyInstaller().ensurePptxGenJS();
export const ensureClasp = () => new DependencyInstaller().ensureClasp();
export const ensureMarp = () => new DependencyInstaller().ensureMarp();
export const ensureDependencies = () => new DependencyInstaller().ensureAllDependencies();
```

---

## 🎨 プロンプトテンプレートシステム

### テンプレート管理システム（template-registry.ts）

```typescript
// packages/core/src/templates/template-registry.ts
import { readFileSync, existsSync } from 'node:fs';
import { join } from 'node:path';

export interface SlideTemplate {
  id: string;
  name: string;
  author: string;
  description: string;
  prompt: string;
  style: SlideStyle;
  tags: string[];
}

export interface SlideStyle {
  backgroundColor: string;
  titleColor: string;
  textColor: string;
  font: string;
  layout: 'clean' | 'modern' | 'academic' | 'creative';
}

export class TemplateRegistry {
  private templates: Map<string, SlideTemplate> = new Map();
  
  constructor() {
    this.loadBuiltInTemplates();
    this.loadCustomTemplates();
  }
  
  private loadBuiltInTemplates() {
    // Keitaroさんのテンプレート
    this.registerTemplate({
      id: 'keitaro-slide-prompt',
      name: 'Keitaro Professional',
      author: 'keitaro',
      description: 'プロフェッショナルなビジネスプレゼンテーション用テンプレート',
      prompt: `
あなたは経験豊富なビジネスコンサルタントです。
以下の条件でプロフェッショナルなプレゼンテーションを作成してください：

トピック: {{topic}}
スライド数: {{slides_count}}
対象者: {{audience}}

# 構成要件
1. 課題設定の明確化
2. データに基づく分析
3. 具体的な解決策の提示
4. ROIと効果測定指標
5. アクションプランと次のステップ

# スタイルガイドライン
- 簡潔で力強いメッセージ
- データと事実に基づく論理構成
- 視覚的に分かりやすい情報整理
- ビジネス価値を明確に伝達

各スライドは実務で即座に使用できる品質で作成してください。
      `,
      style: {
        backgroundColor: '#FFFFFF',
        titleColor: '#1E40AF',
        textColor: '#374151',
        font: 'Arial',
        layout: 'clean',
      },
      tags: ['business', 'professional', 'consulting'],
    });
    
    // クリエイティブテンプレート
    this.registerTemplate({
      id: 'creative-visual',
      name: 'Creative Visual',
      author: 'slide-code',
      description: 'クリエイティブで視覚的インパクトのあるプレゼンテーション',
      prompt: `
あなたはクリエイティブディレクターです。
視覚的に魅力的で記憶に残るプレゼンテーションを作成してください：

トピック: {{topic}}
スライド数: {{slides_count}}

# 創造性要件
1. ストーリーテリングを重視
2. 感情に訴えかける構成
3. 視覚的メタファーの活用
4. インタラクティブな要素
5. 記憶に残るキーメッセージ

視聴者の心に響く、印象的なプレゼンテーションにしてください。
      `,
      style: {
        backgroundColor: '#0F172A',
        titleColor: '#F59E0B',
        textColor: '#F1F5F9',
        font: 'Helvetica',
        layout: 'creative',
      },
      tags: ['creative', 'visual', 'storytelling'],
    });
  }
  
  private loadCustomTemplates() {
    // カスタムテンプレートディレクトリから読み込み
    const customPath = join(process.cwd(), 'templates');
    if (existsSync(customPath)) {
      // カスタムテンプレートの読み込みロジック
    }
  }
  
  registerTemplate(template: SlideTemplate) {
    this.templates.set(template.id, template);
  }
  
  getTemplate(id: string): SlideTemplate | undefined {
    return this.templates.get(id);
  }
  
  listTemplates(): SlideTemplate[] {
    return Array.from(this.templates.values());
  }
  
  searchTemplates(query: string): SlideTemplate[] {
    const lowQuery = query.toLowerCase();
    return this.listTemplates().filter(template =>
      template.name.toLowerCase().includes(lowQuery) ||
      template.description.toLowerCase().includes(lowQuery) ||
      template.tags.some(tag => tag.includes(lowQuery))
    );
  }
}
```

---

## 🚀 アプリケーション自動起動システム

### アプリケーション起動サービス（app-launcher.ts）

```typescript
// packages/core/src/services/app-launcher.ts
import { exec } from 'node:child_process';
import { promisify } from 'node:util';
import { platform } from 'node:os';
import { existsSync } from 'node:fs';

const execAsync = promisify(exec);

export class AppLauncher {
  async launchPowerPoint(filePath: string): Promise<void> {
    const currentPlatform = platform();
    
    try {
      switch (currentPlatform) {
        case 'darwin': // macOS
          await this.launchOnMac(filePath);
          break;
        case 'win32': // Windows
          await this.launchOnWindows(filePath);
          break;
        case 'linux': // Linux
          await this.launchOnLinux(filePath);
          break;
        default:
          console.warn(`プラットフォーム ${currentPlatform} はサポートされていません`);
      }
    } catch (error) {
      console.warn('PowerPointの自動起動に失敗しました:', error);
      console.log(`手動でファイルを開いてください: ${filePath}`);
    }
  }
  
  private async launchOnMac(filePath: string): Promise<void> {
    // Microsoft PowerPointが利用可能かチェック
    try {
      await execAsync('which /Applications/Microsoft\\ PowerPoint.app/Contents/MacOS/Microsoft\\ PowerPoint');
      await execAsync(`open -a "Microsoft PowerPoint" "${filePath}"`);
      console.log('✅ PowerPointでプレゼンテーションを開きました');
    } catch {
      // Keynoteをフォールバック
      try {
        await execAsync(`open -a "Keynote" "${filePath}"`);
        console.log('✅ Keynoteでプレゼンテーションを開きました');
      } catch {
        await execAsync(`open "${filePath}"`);
        console.log('✅ デフォルトアプリでファイルを開きました');
      }
    }
  }
  
  private async launchOnWindows(filePath: string): Promise<void> {
    // PowerPointの実行ファイルパスを探索
    const possiblePaths = [
      'C:\\Program Files\\Microsoft Office\\root\\Office16\\POWERPNT.EXE',
      'C:\\Program Files (x86)\\Microsoft Office\\root\\Office16\\POWERPNT.EXE',
      'C:\\Program Files\\Microsoft Office\\Office16\\POWERPNT.EXE',
      'C:\\Program Files (x86)\\Microsoft Office\\Office16\\POWERPNT.EXE',
    ];
    
    for (const path of possiblePaths) {
      if (existsSync(path)) {
        await execAsync(`"${path}" "${filePath}"`);
        console.log('✅ PowerPointでプレゼンテーションを開きました');
        return;
      }
    }
    
    // デフォルトアプリで開く
    await execAsync(`start "" "${filePath}"`);
    console.log('✅ デフォルトアプリでファイルを開きました');
  }
  
  private async launchOnLinux(filePath: string): Promise<void> {
    // LibreOffice Impressで開く
    try {
      await execAsync(`libreoffice --impress "${filePath}"`);
      console.log('✅ LibreOffice Impressでプレゼンテーションを開きました');
    } catch {
      await execAsync(`xdg-open "${filePath}"`);
      console.log('✅ デフォルトアプリでファイルを開きました');
    }
  }
  
  async launchBrowser(url: string): Promise<void> {
    const open = await import('open');
    await open.default(url);
    console.log(`✅ ブラウザでプレゼンテーションを開きました: ${url}`);
  }
  
  async launchMarkdownViewer(filePath: string): Promise<void> {
    // Marpプレビューまたはデフォルトマークダウンビューア
    try {
      await execAsync(`marp --preview "${filePath}"`);
      console.log('✅ Marpプレビューでスライドを表示しました');
    } catch {
      const open = await import('open');
      await open.default(filePath);
      console.log('✅ デフォルトエディタでマークダウンを開きました');
    }
  }
}

// エクスポート関数
export const launchPowerPoint = (filePath: string) => 
  new AppLauncher().launchPowerPoint(filePath);
export const launchBrowser = (url: string) => 
  new AppLauncher().launchBrowser(url);
export const launchMarkdownViewer = (filePath: string) => 
  new AppLauncher().launchMarkdownViewer(filePath);
```

---

## 💻 CLIコマンド体系

### メインスライドコマンド（slideCommand.ts）

```typescript
// packages/cli/src/ui/commands/slideCommand.ts
import { SlashCommand, CommandContext, SlashCommandActionReturn } from './types.js';

export const slideCommand: SlashCommand = {
  name: 'slide',
  description: 'スライドを生成します',
  kind: CommandKind.BUILT_IN,
  
  subCommands: [
    {
      name: 'new',
      description: '新しいスライドを作成',
      kind: CommandKind.BUILT_IN,
      action: async (context: CommandContext, args: string) => {
        const parts = args.trim().split(' ');
        const topic = parts[0];
        
        if (!topic) {
          return {
            type: 'message',
            messageType: 'error',
            content: 'トピックを指定してください。例: /slide new "AIの未来"',
          };
        }
        
        // インタラクティブな形式選択
        return {
          type: 'dialog',
          dialog: 'slide-format-selector',
          data: { topic, args: args.slice(topic.length).trim() },
        };
      },
    },
    
    {
      name: 'marp',
      description: 'Marpマークダウンスライドを生成',
      kind: CommandKind.BUILT_IN,
      action: async (context: CommandContext, args: string) => {
        return {
          type: 'tool',
          toolName: 'slide-generator',
          toolArgs: {
            topic: args,
            output_format: 'marp',
            auto_open: true,
          },
        };
      },
    },
    
    {
      name: 'pptx',
      description: 'PowerPoint形式でスライドを生成',
      kind: CommandKind.BUILT_IN,
      action: async (context: CommandContext, args: string) => {
        return {
          type: 'tool',
          toolName: 'slide-generator',
          toolArgs: {
            topic: args,
            output_format: 'pptx',
            auto_open: true,
          },
        };
      },
    },
    
    {
      name: 'google',
      description: 'Google Slideでプレゼンテーションを作成',
      kind: CommandKind.BUILT_IN,
      action: async (context: CommandContext, args: string) => {
        return {
          type: 'tool',
          toolName: 'slide-generator',
          toolArgs: {
            topic: args,
            output_format: 'google',
            auto_open: true,
          },
        };
      },
    },
    
    {
      name: 'template',
      description: 'プロンプトテンプレートを使用してスライド作成',
      kind: CommandKind.BUILT_IN,
      action: async (context: CommandContext, args: string) => {
        const parts = args.trim().split(' ');
        const templateId = parts[0];
        const topic = parts.slice(1).join(' ');
        
        if (!templateId || !topic) {
          return {
            type: 'message',
            messageType: 'error',
            content: '使用例: /slide template keitaro-slide-prompt "デジタル変革戦略"',
          };
        }
        
        return {
          type: 'tool',
          toolName: 'slide-generator',
          toolArgs: {
            topic,
            template: templateId,
            output_format: 'pptx', // デフォルト
            auto_open: true,
          },
        };
      },
      
      completion: async (context: CommandContext, partialArg: string) => {
        const templateRegistry = context.services.config?.getTemplateRegistry();
        if (!templateRegistry) return [];
        
        return templateRegistry.listTemplates()
          .filter(template => template.id.startsWith(partialArg))
          .map(template => template.id);
      },
    },
  ],
};
```

---

## 📦 パッケージ設定とビルドシステム

### ルートpackage.json

```json
{
  "name": "slide-code",
  "version": "1.0.0",
  "description": "スライド生成特化CLIエージェント",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/yourusername/slide-code.git"
  },
  "type": "module",
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "build": "npm run build --workspaces",
    "build:cli": "npm run build -w packages/cli",
    "build:core": "npm run build -w packages/core",
    "start": "npm run start -w packages/cli",
    "dev": "npm run dev -w packages/cli",
    "test": "npm run test --workspaces",
    "test:ci": "npm run test:ci --workspaces",
    "typecheck": "npm run typecheck --workspaces",
    "lint": "npm run lint --workspaces",
    "setup": "./scripts/setup.sh",
    "install-deps": "node scripts/install-deps.js"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0",
    "vitest": "^1.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### CLI package.json

```json
{
  "name": "@slide-code/cli",
  "version": "1.0.0",
  "description": "Slide Code CLI",
  "type": "module",
  "main": "dist/index.js",
  "bin": {
    "slide": "dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/index.js",
    "test": "vitest run",
    "test:ci": "vitest run --coverage",
    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext .ts,.tsx"
  },
  "dependencies": {
    "@slide-code/core": "workspace:*",
    "@google/genai": "^1.13.0",
    "ink": "^6.1.1",
    "ink-select-input": "^6.2.0",
    "ink-spinner": "^5.0.0",
    "react": "^19.1.0",
    "open": "^10.1.2"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "typescript": "^5.0.0"
  }
}
```

---

## 🎯 初回セットアップスクリプト

### セットアップスクリプト（setup.sh）

```bash
#!/bin/bash

# Slide Code セットアップスクリプト
echo "🎯 Slide Code セットアップを開始します..."

# Node.jsバージョン確認
echo "📋 システム要件をチェック中..."
if ! command -v node &> /dev/null; then
    echo "❌ Node.jsがインストールされていません"
    echo "   https://nodejs.org/ からインストールしてください"
    exit 1
fi

NODE_VERSION=$(node -v | sed 's/v//')
REQUIRED_VERSION="18.0.0"

if ! node -e "process.exit(require('semver').gte('$NODE_VERSION', '$REQUIRED_VERSION') ? 0 : 1)" 2>/dev/null; then
    echo "❌ Node.js バージョン $REQUIRED_VERSION 以上が必要です（現在: $NODE_VERSION）"
    exit 1
fi

echo "✅ Node.js $NODE_VERSION"

# 依存関係インストール
echo "📦 依存関係をインストール中..."
npm install

# ビルド実行
echo "🔨 プロジェクトをビルド中..."
npm run build

# Slide Code CLI をグローバルインストール
echo "🌐 Slide Code CLI をグローバルインストール中..."
npm link

# 設定ディレクトリ作成
echo "📁 設定ディレクトリを作成中..."
SLIDE_CODE_DIR="$HOME/.slide-code"
mkdir -p "$SLIDE_CODE_DIR/templates"
mkdir -p "$SLIDE_CODE_DIR/output"

# デフォルト設定ファイル作成
echo "⚙️  デフォルト設定を作成中..."
cat > "$SLIDE_CODE_DIR/config.json" << EOL
{
  "defaultOutputFormat": "pptx",
  "defaultStyle": "professional",
  "autoOpen": true,
  "outputDirectory": "$SLIDE_CODE_DIR/output",
  "templatesDirectory": "$SLIDE_CODE_DIR/templates",
  "gemini": {
    "model": "gemini-2.0-flash",
    "apiKey": ""
  }
}
EOL

# セットアップ完了
echo "🎉 セットアップ完了！"
echo ""
echo "📋 次のステップ:"
echo "1. Gemini API キーを設定: slide config set-api-key YOUR_API_KEY"
echo "2. 最初のスライドを作成: slide new \"はじめてのプレゼンテーション\""
echo ""
echo "📚 使用例:"
echo "  slide new \"AIの未来\"              # 基本的なスライド作成"
echo "  slide pptx \"マーケティング戦略\"    # PPTX形式で作成"
echo "  slide google \"プロジェクト報告\"    # Google Slideで作成"
echo "  slide template keitaro-slide-prompt \"ビジネス戦略\" # テンプレート使用"
echo ""
echo "🚀 Slide Code の準備が整いました！"
```

---

## 📋 使用例とワークフロー

### 基本的な使用フロー

```bash
# 1. セットアップ
git clone https://github.com/yourusername/slide-code.git
cd slide-code
./scripts/setup.sh

# 2. API設定
slide config set-api-key YOUR_GEMINI_API_KEY

# 3. 基本的なスライド作成
slide new "AIがもたらすビジネス変革"

# 4. 特定形式でのスライド作成
slide pptx "デジタルマーケティング戦略" --slides 15 --style professional

# 5. Google Slideで作成
slide google "プロジェクト進捗報告" --style academic

# 6. プロンプトテンプレート使用
slide template keitaro-slide-prompt "企業のDX推進戦略"

# 7. Marpマークダウン作成
slide marp "技術勉強会：React最新動向"

# 8. テンプレート一覧確認
slide templates list

# 9. 設定確認
slide config show
```

### カスタムプロンプトテンプレートの作成

```typescript
// ~/.slide-code/templates/my-template.js
export default {
  id: 'my-presentation-style',
  name: 'My Custom Style',
  author: 'Your Name',
  description: '私専用のプレゼンテーションスタイル',
  prompt: `
あなたは私の個人的なプレゼンテーションアシスタントです。
以下の特徴で資料を作成してください：

トピック: {{topic}}
- 実体験に基づく具体例を多用
- データよりもストーリーを重視
- 聴衆との対話を促す構成
- アクションに繋がる明確な提案

私らしい、親しみやすくも説得力のある資料にしてください。
  `,
  style: {
    backgroundColor: '#F8FAFC',
    titleColor: '#0EA5E9',
    textColor: '#1E293B',
    font: 'Helvetica',
    layout: 'modern',
  },
  tags: ['personal', 'storytelling', 'engaging'],
};
```

---

## 🔧 設定とカスタマイゼーション

### 設定ファイル（~/.slide-code/config.json）

```json
{
  "defaultOutputFormat": "pptx",
  "defaultStyle": "professional",
  "autoOpen": true,
  "outputDirectory": "/Users/username/.slide-code/output",
  "templatesDirectory": "/Users/username/.slide-code/templates",
  "gemini": {
    "model": "gemini-2.0-flash",
    "apiKey": "your_api_key_here",
    "temperature": 0.7,
    "maxTokens": 4000
  },
  "formats": {
    "marp": {
      "theme": "default",
      "enableHtml": true,
      "allowLocalFiles": true
    },
    "pptx": {
      "defaultTemplate": "professional",
      "autoSave": true,
      "enableAnimations": false
    },
    "google": {
      "autoShare": false,
      "defaultPermission": "private"
    }
  },
  "ui": {
    "showProgress": true,
    "enablePreview": true,
    "theme": "auto"
  }
}
```

---

## 🧪 テスト戦略

### ユニットテスト例

```typescript
// packages/core/src/tools/slide-generator.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { SlideGeneratorTool } from './slide-generator.js';

describe('SlideGeneratorTool', () => {
  let tool: SlideGeneratorTool;
  
  beforeEach(() => {
    tool = new SlideGeneratorTool();
  });
  
  it('should generate slides with correct parameters', async () => {
    const params = {
      topic: 'テスト用プレゼンテーション',
      output_format: 'marp' as const,
      slides_count: 5,
    };
    
    const invocation = tool.createInvocation(params);
    const result = await invocation.execute(new AbortController().signal);
    
    expect(result.llmContent).toContain('# テスト用プレゼンテーション');
    expect(result.llmContent).toContain('---'); // Marp分割記号
  });
  
  it('should handle invalid parameters gracefully', async () => {
    const params = {
      topic: '',
      output_format: 'invalid' as any,
    };
    
    const invocation = tool.createInvocation(params);
    const result = await invocation.execute(new AbortController().signal);
    
    expect(result.error).toBeDefined();
  });
});
```

---

## 🎯 期待される成果物

### 完成後の機能一覧

✅ **基本機能**
- ワンコマンドスライド生成
- 3つの出力形式（Marp、PPTX、Google Slide）
- 自動アプリケーション起動
- プロンプトテンプレートシステム

✅ **自動化機能**
- 依存関係の自動インストール（PptxGenJS、clasp）
- Google認証の自動設定
- PowerPoint/Google Slide/Marpの自動起動

✅ **カスタマイゼーション**
- カスタムプロンプトテンプレート作成
- スタイルテーマのカスタマイズ
- 出力先ディレクトリの設定

✅ **開発者体験**
- TypeScript完全対応
- 包括的なテストスイート
- 豊富な設定オプション
- 拡張可能なアーキテクチャ

---

## 🚀 実装開始への準備

### ClaudeCode実装時の注意点

1. **モノレポ構造**: lerna/npm workspacesを使用
2. **TypeScript**: 厳格な型定義を維持
3. **エラーハンドリング**: 外部依存関係のエラーを適切に処理
4. **クロスプラットフォーム**: Windows/macOS/Linuxでの動作確保
5. **セキュリティ**: API キーの安全な管理

### 推奨実装順序

1. **基本構造の構築**（packages/core、packages/cli）
2. **スライド生成ツールの実装**
3. **依存関係インストーラーの実装**
4. **各出力形式のエクスポーター実装**
5. **プロンプトテンプレートシステム**
6. **CLIコマンド体系の実装**
7. **テストの充実**
8. **ドキュメントの整備**

---

---

## 🎯 CLIエージェント体験の核心設計

### ターミナル操作フロー

#### 基本的な使用体験
```bash
$ slide
🎯 Slide Code v1.0.0 - スライド生成特化CLIエージェント

💬 Type your slide topic or use /slide, /template commands...
🎯 > _
```

#### リアルタイム補完とストリーミング
```bash
🎯 > /slide new "AIの未来について"

📊 Progress: 1/5 - AI構造設計中...
✨ AIがプレゼンテーション構造を設計しています...

📊 Progress: 2/5 - スライド内容生成中...
# AIの未来について

## アジェンダ
1. 現在のAI技術動向
2. 産業への影響...

📊 Progress: 3/5 - PPTX形式に変換中...
📊 Progress: 4/5 - PowerPoint起動中...
📊 Progress: 5/5 - 完了！

✅ Generation complete! 
   📄 File: ~/slides/ai-future-20250823.pptx
   🚀 PowerPoint opened automatically

💬 Type your slide topic or use /slide, /template commands...
🎯 > _
```

### Gemini CLIとの共通体験要素

#### 1. **同一のターミナル入力欄**
- プロンプトプレフィックス: `🎯 > `（Gemini CLIの`> `に相当）
- リアルタイム入力とEnterキー送信
- 履歴機能（上下キーでナビゲーション）
- Ctrl+Cで中断

#### 2. **ストリーミングレスポンス**
- AIの思考過程をリアルタイム表示
- プログレス表示とステージ更新
- 生成中のキャンセル機能

#### 3. **設定管理**
```bash
🎯 > /config show
📋 Slide Code Configuration:
   API Key: ●●●●●●●●AIza (configured)
   Default Format: pptx
   Auto Open: enabled
   Templates Dir: ~/.slide-code/templates

🎯 > /config set default-format marp
✅ Default format updated to: marp
```

---

## 🔧 実装時の重要なポイント

### 1. Gemini CLIからの継承必須要素

#### A. InputPrompt.tsxの完全移植
```typescript
// 必須機能:
- TextBuffer システム
- useInput フック
- キーボードショートカット処理
- コマンド補完システム
- 履歴管理（useInputHistory）
- Vim モードサポート（オプション）
```

#### B. App.tsxの構造継承
```typescript
// 必須コンポーネント:
- ストリーミング状態管理
- エラーハンドリング
- リサイズ対応（useTerminalSize）
- フォーカス管理（useFocus）
- React + Ink レンダリング
```

#### C. CoreToolSchedulerの移植
```typescript
// 必須機能:
- ツール実行状態の管理
- ストリーミング出力処理
- エラーハンドリング
- タイムアウト管理
- 結果フォーマット
```

### 2. 新規実装が必要な部分

#### A. スライド特化UI
- SlideProgressIndicator
- FormatSelector  
- TemplateSelector
- SlidePreview

#### B. 依存関係管理
- PptxGenJS自動インストール
- Clasp自動インストール
- アプリケーション自動起動

#### C. プロンプトシステム
- テンプレート管理
- keitaro-slide-prompt等のカスタムプロンプト
- 使用統計とフィードバック

### 3. 開発優先順位

#### 🥇 Phase 1: 基盤構築（1週間）
1. Gemini CLIの InputPrompt.tsx 完全移植
2. App.tsx の基本構造移植
3. 基本的なスライド生成ツール実装
4. Marp出力のみ対応

#### 🥈 Phase 2: 機能拡張（2週間）
1. PPTX出力機能（PptxGenJS統合）
2. Google Slide出力（Clasp統合）
3. 依存関係自動インストール
4. アプリケーション自動起動

#### 🥉 Phase 3: 高度機能（1ヶ月）
1. プロンプトテンプレートシステム
2. オンボーディング体験
3. 設定管理UI
4. エラーハンドリング強化

---

## 🎓 ClaudeCode実装ガイダンス

### 実装開始時のチェックリスト

- [ ] Gemini CLIのInputPrompt.tsxを理解し、同様の仕組みを実装
- [ ] React + Inkでのターミナル描画システムを構築
- [ ] useInput、useTerminalSize等のフック活用
- [ ] ストリーミングレスポンス処理の実装
- [ ] ツール実行基盤（スケジューラー）の構築
- [ ] オンボーディングフローの実装
- [ ] 依存関係自動管理システムの構築
- [ ] 3つの出力形式対応
- [ ] プロンプトテンプレートシステム
- [ ] エラーハンドリングの充実

### 技術的注意事項

#### TypeScript設定
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true
  }
}
```

#### 必須依存関係
```json
{
  "dependencies": {
    "@google/genai": "^1.13.0",
    "ink": "^6.1.1",
    "ink-text-input": "^6.2.0",
    "ink-select-input": "^6.2.0",
    "ink-spinner": "^5.0.0",
    "react": "^19.1.0",
    "pptxgenjs": "^3.12.0",
    "@google/clasp": "^2.4.2",
    "open": "^10.1.2"
  }
}
```

### パフォーマンス要件

- **起動時間**: < 3秒
- **スライド生成時間**: < 30秒（10スライド）
- **メモリ使用量**: < 256MB
- **依存関係インストール**: < 60秒

### 品質保証

- **TypeScript**: 厳格な型チェック
- **テストカバレッジ**: > 80%
- **エラーハンドリング**: 全API呼び出し
- **クロスプラットフォーム**: Windows/macOS/Linux対応

---

この包括的な実装計画書により、ClaudeCodeが独立した「Slide Code」プロジェクトを、Gemini CLIと同等のターミナル体験で構築できるようになります。段階的な開発アプローチと詳細な技術仕様により、高品質な製品を効率的に開発できるでしょう。