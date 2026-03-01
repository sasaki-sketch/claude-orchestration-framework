# Sub-agent Orchestration Guide
# サブエージェント統括ガイド

**Created**: 2026-02-15
**Updated**: 2026-03-01（セクション7.2 学習成果、セクション10.3 プロンプトテンプレート改善）

## このファイルの役割

**目的**: サブエージェントの統括方法を標準化し、プロジェクト実行の一貫性を確保

**対象読者**: メインエージェント、サブエージェント統括者

**このファイルを参照するタイミング**:
- サブエージェントを起動する前
- ハンドオフドキュメントを作成する時
- エラー処理・進捗報告時

**技術ドメイン別ナレッジ**:
- `/learnings/domains/{technology}/`

**オブザーバビリティ**:
- セットアップガイド: `/docs/observability-setup.md`
- 起動: `./scripts/start-observability.sh`
- 停止: `./scripts/stop-observability.sh`
- ダッシュボード: http://localhost:5173

**MCP (Model Context Protocol)**:
- グローバル設定: `~/.claude.json` または `~/.config/claude-code/`
- プロジェクト設定: `.mcp.json`（グローバル設定を上書き）

---

## 0. MCP Server Configuration (MCPサーバー設定)

### MCPツールの使用方法

```
1. ToolSearchで検索
   ToolSearch: query="google sheets" または "select:mcp__google-sheets__get_sheet_data"

2. 返されたツールを使用
   例: mcp__google-sheets__get_sheet_data
```

### MCPツール使用時の注意

1. **ToolSearchで必ず検索**: MCPツールは遅延ロードされるため、使用前にToolSearchで読み込む
2. **認証**: 初回使用時に認証が必要な場合がある（google-sheets等）
3. **エラー時の対応**: 認証エラーの場合はユーザーに再認証を依頼

---

## 1. Agent Roles (エージェントの役割)

### 標準ロール

| Role | Responsibility | When to Use |
|------|----------------|-------------|
| **{project}-pm** | Project management, coordination, approval | Progress reviews, go/no-go decisions, task prioritization |
| **{project}-engineer** | Technical implementation, PoC, architecture | Technical setup, workflows, technical design |
| **{project}-reviewer** | Quality assurance, testing, validation | Test execution, acceptance criteria verification |
| **{project}-researcher** | Industry knowledge, analysis | Background research, best practices |
| **{project}-designer** | Form design, workflow design | UI/UX design, process flow |
| **{project}-integrator** | Integration, synthesis | Combining outputs, creating alternatives |
| **{project}-checker** | Deliverable verification | Checklist execution, final validation |

**詳細なロール定義**: `/.claude/agents/_roles.md`

### Agent Configuration Files

```
/.claude/agents/
├── _common.md              ← 共通設定（テンプレート変数付き）
├── _roles.md               ← 標準ロール定義
├── templates/
│   ├── spawn-template.md
│   └── agent-template.md   ← 新規エージェント作成用
└── project/
    └── {project}/          ← プロジェクト固有エージェント
        ├── {project}-pm.md
        ├── {project}-designer.md
        └── ...
```

---

## 2. When to Use Sub-agents (サブエージェントを使うタイミング)

### Use Sub-agent Team (サブエージェントチームを使う)

| Condition | Example |
|-----------|---------|
| Foundation/base tasks | Creating PoC spreadsheets, initial setup |
| Multi-step complex tasks | AppSheet development, workflow implementation |
| Tasks requiring project context | Design decisions affecting production |
| Quality-critical tasks | Testing, validation, reviews |
| Learning opportunities | Building team knowledge |

### Direct Execution OK (直接実行でOK)

| Condition | Example |
|-----------|---------|
| Simple, urgent tasks | Quick fixes, minor edits |
| Single-file changes | Updating one document |
| Exploratory tasks | Initial research, file reading |

### Decision Checklist (判断チェックリスト)

```markdown
Before starting a task, ask:
□ Is this a foundation/base task? → Yes: Use sub-agent team
□ Does it require multiple steps? → Yes: Use sub-agent team
□ Does it affect other tasks? → Yes: Check dependencies first
□ Is there learning value? → Yes: Consider sub-agent team
□ Is it urgent and simple? → Yes: Direct execution OK
```

---

## 3. Handoff Process (引き継ぎプロセス)

### Standard Handoff Steps

```
Step 1: Create Handoff Document
├── Use template below
├── Save to /context/{task}-handoff.md
└── Include ALL relevant file paths

Step 2: Update Session Handoff
├── /context/session-handoff.md
└── Mark completed tasks, add new tasks

Step 3: Get Approvals
├── PM review (project perspective)
├── Engineer review (technical perspective)
└── Document decisions

Step 4: Spawn Sub-agents with Context
├── EXPLICITLY tell agents about existing files
├── Provide full file paths
└── Include summary of decisions made
```

### Handoff Document Template

```markdown
# [Task Name] Handoff Document

**Created**: YYYY-MM-DD
**From**: [Source agent/session]
**To**: [Target agent(s)]

## Completed Tasks
- [x] Task 1
- [x] Task 2

## Created Files
| File | Path | Description |
|------|------|-------------|
| File1 | /full/path/to/file | Description |

## Design Decisions
1. Decision 1: [Rationale]
2. Decision 2: [Rationale]

## Remaining Tasks
- [ ] Task A (assigned to: agent)
- [ ] Task B (assigned to: agent)

## Important Notes
- Note 1
- Note 2
```

---

## 4. Communication Patterns (コミュニケーションパターン)

### Main Agent → Sub-agent

```markdown
When spawning a sub-agent, ALWAYS include:

1. EXPLICIT file paths
   - "The handoff document is at /full/path/to/file.md"
   - "Please READ this file first"

2. Context summary
   - What was done
   - What decisions were made
   - What questions remain

3. Clear task definition
   - What to do
   - What to output
   - What format to use
```

### Sub-agent → Main Agent

```markdown
Sub-agent responses should include:

1. Confirmation of files read
   - "I read /path/to/file.md"

2. Clear assessment
   - Go / No-go decision
   - Reasoning

3. Next steps
   - Specific action items
   - Assigned owners

4. Agent ID for resume
   - "agentId: xxxxx"
```

---

## 5. Error Handling Patterns (エラー処理パターン)

### Common Errors and Solutions

| Error Type | Example | Solution |
|------------|---------|----------|
| **API Not Enabled** | "Google Sheets API has not been used in project..." | Provide activation URL, wait for user to enable |
| **File Not Found** | Sub-agent can't find handoff document | Always provide EXPLICIT full paths when spawning |
| **Permission Denied** | Can't access spreadsheet | Check sharing settings, verify account |
| **Format Error** | Excel file not readable | Convert to Google Sheets format first |
| **Tool Rejected** | User rejects tool execution | Stop, ask user how to proceed |

### Error Response Template

```markdown
## Error Occurred (エラーが発生しました)

**Error**: [Error message]
**Cause**: [Explanation in simple terms]

### How to Fix (解決方法)
1. Step 1
2. Step 2
3. Step 3

### After Fixing
Tell me: "[confirmation phrase]"
```

### Retry Strategy

```
1. First failure: Explain error, provide solution
2. Second failure: Try alternative approach
3. Third failure: Escalate to user, ask for guidance
```

---

## 5.5 User Manual Work Policy (ユーザー手動作業ポリシー)

### 原則: ユーザーに手動作業を依頼する前に、エージェントチームで解決を試みる

### エージェント運用Tips

| # | Tips | 重要度 |
|---|------|--------|
| G-1 | エージェントチームに作業を依頼する前に手動作業を依頼しない | 高 |
| G-2 | ToolSearchでMCPツールを検索してから「できない」と判断する | 高 |

**禁止事項**:
- 一度の失敗で「ユーザーに手動でお願いします」と言う
- エージェントチームに依頼せず直接ユーザーに作業を振る
- ToolSearchを試さずに「できない」と判断する

**必須チェックリスト（手動作業依頼前）**:
```
□ エージェントチーム（Task tool）に依頼したか？
□ ToolSearchでMCPツールを探したか？
□ 別のアプローチを試したか？
□ 3回以上試行したか？
```

### 外部サービス操作時のフロー

```
外部サービス操作が必要
    │
    ▼
[1] ToolSearchでMCPツール検索
    │
    ├── 見つかった → MCPツール使用
    │
    └── 見つからない
            │
            ▼
[2] エージェントチームに依頼（Task tool）
    │
    ├── 成功 → 完了
    │
    └── 失敗
            │
            ▼
[3] 別アプローチを試行
    │
    ├── 成功 → 完了
    │
    └── 失敗（3回以上）
            │
            ▼
[4] ユーザーに手動作業を依頼（最終手段）
```

---

## 5.6 Orchestrator Anti-Patterns (オーケストレーターのアンチパターン)

### 原則: オーケストレーターは委譲者であり、作業者ではない

### アンチパターンチェックリスト

**タスク開始前チェック（必須）**:
```
□ このタスクはエージェントチームに委譲できないか？
□ 自分で作業するより、エージェントに依頼した方が効果的ではないか？
□ エージェントの教育機会を逃していないか？
```

**分析・判断前チェック（必須）**:
```
□ 全ての関連資料を参照したか？（MECE確認）
  - 紙版資料
  - デジタル版資料（Google Form、Spreadsheetなど）
  - 過去の議事録・決定事項
□ 分析をエージェントに依頼したか？
□ 自分の判断はユーザー要件と矛盾していないか？
```

**報告前チェック（必須）**:
```
□ エージェントの判断をユーザー要件と照合したか？
□ 「除外推奨」「Phase 2以降」がユーザー要件と矛盾していないか？
□ 専門用語・プロジェクト用語を定義したか？
□ ユーザーが判断できる材料を提示しているか？
```

### 禁止事項

| 禁止行動 | 正しい行動 |
|---------|----------|
| オーケストレーターが直接ファイルを読んで分析 | エージェントに分析を依頼 |
| 技術的理由で「除外推奨」と即断 | ユーザー要件との整合性を確認 |
| 一部の資料のみ参照して判断 | 全資料をMECEに参照 |
| 「Phase 2」を定義せずに使用 | 用語の定義を先に説明 |
| エージェント判断をそのまま報告 | ユーザー要件との矛盾を検証して報告 |

---

## 6. Progress Reporting Format (進捗報告フォーマット)

### Session Progress Report

```markdown
## Progress Report: YYYY-MM-DD

### Session Summary
- **Duration**: X hours
- **Main Task**: [Task description]
- **Status**: 🟢 Complete / 🟡 In Progress / 🔴 Blocked

### Completed Items
| # | Item | Result |
|---|------|--------|
| 1 | [Item] | ✅ Done |
| 2 | [Item] | ✅ Done |

### In Progress
| # | Item | Progress | Blocker |
|---|------|----------|---------|
| 1 | [Item] | 70% | None |

### Decisions Made
1. [Decision]: [Rationale]

### Files Created/Modified
| File | Action | Path |
|------|--------|------|
| file.md | Created | /path/to/file.md |

### Next Steps
1. [ ] Next task 1
2. [ ] Next task 2
```

### PM Update Format

```markdown
## PM Update Request

**Task**: [Task name]
**Status**: [Status]

### Key Updates
- Update 1
- Update 2

### Questions for PM
1. Question 1?
2. Question 2?

### Recommendation
[Your recommendation]
```

### Sub-agent Task Completion Report

```markdown
## Task Completion: [Task Name]

**Agent**: [Agent name]
**Agent ID**: [ID for resume]
**Status**: ✅ Complete / ⚠️ Partial / ❌ Failed

### What Was Done
1. Action 1
2. Action 2

### Output
[Description or link to output]

### Issues Encountered
- Issue 1: [Resolution]

### Recommendations
- Recommendation 1
```

---

## 7. Lessons Learned (学んだこと)

### Best Practices Discovered

1. **Pre-spawn checklist**: Before spawning sub-agent, list all relevant files
2. **Handoff before complex tasks**: Create handoff doc BEFORE starting multi-step work
3. **PM approval for design decisions**: Get PM sign-off on decisions affecting production
4. **User confirmation points**: Use AskUserQuestion for structured input
5. **Requirements alignment before implementation**: PRD作成後、実装前に依頼者と認識を揃える
6. **Respect original design intent**: 元フォームの「任意/必須」設定には業務上の理由がある
7. **Agent team before user manual work**: ユーザーに手動作業を依頼する前にエージェントチームで解決を試みる
8. **ToolSearch for external services**: 外部サービス操作時は必ずToolSearchでMCPツールを検索
9. **Orchestrator delegates, not executes**: オーケストレーターは作業を委譲し、自分で実行しない
10. **User requirements override technical convenience**: ユーザー要件は技術的な「省略可能」より優先
11. **MECE source verification**: 分析前に全ての関連資料をMECEに確認する
12. **Define terms before using**: 専門用語・プロジェクト用語は定義してから使用
13. **Validate agent judgment against user requirements**: エージェントの判断をユーザー要件と照合してから報告
14. **Review Wave is mandatory**: 同一プロンプトでも品質にばらつきが出るため、成果物作成後のレビューWaveは必須
15. **Split large documents**: 800行超のドキュメントは分割指示または詳細チェックリストを提供
16. **Reference structure file first**: 高品質成果物のパターン：「構造参考ファイル」を最初に読む指示が効果的

### 7.2 Google Sheets オーケストレーション学習 (2026-03-01)

#### サブエージェント品質のばらつき対策

| 学習 | 具体的な改善策 |
|------|--------------|
| 同一プロンプトでも品質にばらつきが出る | レビューWave必須（パターン3参照） |
| 長文ドキュメント(800行超)で欠落発生しやすい | 分割指示 or 詳細チェックリスト提供 |
| 高品質例(data_table.md)の要因分析 | 「構造参考ファイル」を最初に読む指示が効果的 |

#### プロンプト設計の改善点

| 学習 | 具体的な改善策 |
|------|--------------|
| `fontFamily`欠落が発生 | プロンプトに「全textFormatにfontFamilyを含める」を明示 |
| 参照ファイル指定で一貫性向上 | 構造的一貫性が必要な場合は参照ファイルを明示 |
| 比較対象の明示でレビュー精度向上 | レビュー時に「何と比較するか」を明示指示 |

#### 推奨Waveパターン（成果物作成時）

```
Wave N（並列）: 成果物作成
├── Agent A: ドキュメントA作成
├── Agent B: ドキュメントB作成
└── Agent C: ドキュメントC作成
         ↓
Wave N+1: レビュー（必須）← LLMの確率的性質によるばらつき検出
└── Reviewer: 全成果物をレビュー（構造・プロパティ一貫性）
         ↓
Wave N+2（並列）: 推奨対応の実装
├── Agent A: 指摘事項対応
├── Agent B: 指摘事項対応
└── Agent C: 指摘事項対応
```

---

## 8. 技術ドメインナレッジ参照

### ナレッジファイル構造

技術ドメイン別の知見は `/learnings/domains/` に格納:

```
/learnings/domains/
├── appsheet/            ← AppSheet関連
│   ├── appsheet-tips.md
│   ├── appsheet-troubleshooting.md
│   └── ...
└── {technology}/        ← その他技術ドメイン
```

### 技術別参照ガイド

| 技術 | 参照パス | 優先度 |
|------|---------|--------|
| AppSheet | `/learnings/domains/appsheet/` | 必読 |
| その他 | `/learnings/domains/{technology}/` | 必要時 |

---

## 9. Quick Reference (クイックリファレンス)

### Key File Locations

```
/context/
├── project-context.md      # Project overview
├── session-handoff.md      # Current session state
└── {task}-handoff.md       # Task-specific handoffs

/learnings/
├── shared-learnings.md           # 共有学習（必読）
├── domains/                      # 技術ドメイン別ナレッジ
└── project/                      # プロジェクト固有learnings
    └── {project}/

/.claude/
├── agents/                 # Agent definitions
└── orchestration-guide.md  # This file
```

### Useful Commands

| Action | How |
|--------|-----|
| Resume agent | `Task` tool with `resume: "agentId"` |
| Spawn PM | `Task` tool with `subagent_type: "general-purpose"`, prompt includes PM role |
| Structured questions | `AskUserQuestion` tool |
| Track tasks | `TaskCreate`, `TaskUpdate`, `TaskList` tools |

---

## 10. エージェント起動の必須手順

### 10.1 subagent_type の制約

Claude Code の Task tool は以下の `subagent_type` のみ使用可能:
- "Bash", "general-purpose", "statusline-setup", "Explore", "Plan", "claude-code-guide"

**カスタムエージェント定義（{project}-*.md）を subagent_type で直接指定することは技術的に不可能**

→ `subagent_type: "general-purpose"` + プロンプトで定義ファイル参照を指示する

### 10.2 プロンプト必須要素

| 要素 | 必須 | 説明 | 例 |
|------|------|------|-----|
| 定義ファイル読み込み指示 | 必須 | `/.claude/agents/{project}-{role}.md を読んでください` | `{project}-pm.md` |
| 共通設定読み込み | 必須 | `/.claude/agents/_common.md を読んでください` | - |
| shared-learnings参照 | 必須 | `/learnings/shared-learnings.md を読んでください` | - |
| タスク内容 | 必須 | 具体的な実行内容 | 「進捗をレビューしてください」 |
| 稼働ログ記録 | 必須 | `/logs/agent-activity.md に記録してください` | - |

### 10.3 標準プロンプトテンプレート

```markdown
あなたは {project}-{role} エージェントです。

## 必須読み込み（この順序で）
まず以下のファイルを読んでください:
1. [構造参考ファイル]（**最初に読む** - 成果物の構造・フォーマットの参考）
2. /.claude/agents/{project}-{role}.md（エージェント定義）
3. /.claude/agents/_common.md（共通設定）
4. /learnings/shared-learnings.md（共有学習）

## タスク
上記を読んだ上で、以下のタスクを実行してください:
[具体的なタスク内容]

## 品質要件（必須）
- 全てのプロパティを明示（省略しない）
- textFormat には必ず `fontFamily` を含める
- 構造参考ファイルと同じフォーマットを維持
- 不明な点は推測せず確認する

## 完了後
タスク完了後、/logs/agent-activity.md に稼働記録を追記してください。
学びがあれば /learnings/shared-learnings.md にも追記してください。
```

### 10.3.1 プロンプト設計のポイント

| ポイント | 理由 | 効果 |
|---------|------|------|
| 構造参考ファイルを最初に読ませる | 期待する出力形式を先に理解させる | 構造的一貫性向上 |
| 品質要件を明示的に記載 | LLMは指示されないと省略する傾向 | プロパティ欠落防止 |
| 「省略しない」を明記 | 暗黙の省略を防止 | 完全性向上 |

### 10.3.2 レビュー用プロンプトテンプレート

```markdown
あなたは {project}-reviewer エージェントです。

## 必須読み込み
1. [比較基準ファイル]（**最初に読む** - 正しい構造・フォーマットの例）
2. [レビュー対象ファイル群]

## レビュー観点
1. **構造的一貫性**: 比較基準ファイルと同じ構造か
2. **プロパティ完全性**: 必須プロパティ（fontFamily等）が欠落していないか
3. **値の妥当性**: カラー値、サイズ値が仕様に準拠しているか

## 出力形式
| 優先度 | 対象ファイル | 指摘内容 | 推奨対応 |
|--------|------------|---------|---------|
| P1 | xxx.md | 〇〇が欠落 | △△を追加 |
```

### 10.4 禁止事項

| 禁止行動 | 理由 | 正しい方法 |
|---------|------|----------|
| プロンプトに「あなたは{project}-XXです」と書くだけ（定義ファイル参照なし） | エージェント定義が活用されない | 定義ファイル読み込み指示を含める |
| subagent_type に "{project}-pm" 等を指定 | 技術的に不可能 | "general-purpose" を使用 |
| shared-learnings.md の参照を省略 | 過去の学びが活用されない | 必ず参照指示を含める |
| 稼働ログ記録の省略 | 追跡ができない | 記録指示を必ず含める |

### 10.5 関連ファイル

| ファイル | 用途 |
|---------|------|
| `/.claude/agents/templates/spawn-template.md` | 起動テンプレート（詳細例） |
| `/.claude/agents/verification-checklist.md` | 検証チェックリスト |
| `/logs/agent-activity.md` | 稼働ログ |
| `/logs/agent-dashboard.md` | 稼働集計ダッシュボード |

### 10.6 検証

#### 即時検証（起動直後）

- [ ] エージェントが定義ファイルを読んだことを確認（報告あり）
- [ ] shared-learnings.md への言及があるか確認
- [ ] 出力フォーマットが定義に準拠しているか確認
- [ ] 稼働ログに記録されたか確認

#### 週次検証

- [ ] `/logs/agent-dashboard.md` で定義ファイル参照率を確認（目標: 100%）
- [ ] 稼働0のエージェントがあれば原因調査

---

## 11. 並列実行の標準パターン

### 11.1 原則

**エージェントは可能な限り並列実行する**

依存関係のないタスクは同時に起動し、効率を最大化する。

### 11.2 Wave実行モデル

```
Wave 1（並列）: 依存関係のないタスク群
    ├── Agent A: タスクA
    └── Agent B: タスクB
         ↓
Wave 2（並列）: Wave 1完了後のタスク群
    ├── Agent C: タスクC（A,Bの成果物を使用）
    └── Agent D: タスクD
         ↓
Wave 3: 統合・最終確認
    └── Agent E: 統合レビュー
```

### 11.3 並列実行の判断基準

| 条件 | 並列実行 | 順次実行 |
|------|---------|---------|
| 成果物の依存関係なし | Yes | - |
| 同一ファイルを更新 | - | Yes |
| 前タスクの結果で判断が変わる | - | Yes |
| 調査・設計タスク | Yes | - |
| 統合・レビュータスク | - | Yes（最終Wave） |

### 11.4 典型的なWave構成

#### パターン1: 設計→実装→統合

```
Wave 1（並列）:
├── {project}-designer: UI/データモデル設計
└── {project}-researcher: 法規制・業界標準調査

Wave 2（並列）:
├── {project}-engineer: 実装（設計結果を反映）
└── {project}-reviewer: 設計レビュー

Wave 3:
└── {project}-pm: 統合・最終報告
```

#### パターン2: レビュー→統合→更新

```
Wave 1（並列）:
├── {project}-reviewer: 品質レビュー
├── {project}-designer: 設計レビュー
└── {project}-researcher: 法規制レビュー

Wave 2:
└── {project}-pm: レビュー統合・PRD更新
```

#### パターン3: 成果物作成→レビュー（必須パターン）

```
Wave N（並列）: 成果物作成
├── {project}-designer: ドキュメントA作成
├── {project}-engineer: ドキュメントB作成
└── {project}-researcher: ドキュメントC作成
         ↓
Wave N+1: 成果物レビュー（必須）
└── {project}-reviewer: 全成果物をレビュー
         ↓
Wave N+2（並列）: 推奨対応の実装
├── {project}-designer: R-xx対応
├── {project}-engineer: E-xx対応
└── {project}-researcher: H-xx対応
```

**重要**: 複数エージェントが成果物を作成した後は、必ず {project}-reviewer によるレビューWaveを設ける。

### 11.5 実行方法

単一メッセージ内で複数のTask tool呼び出しを行うことで並列実行される。

```
// 並列実行（1つのメッセージで複数Task呼び出し）
Task 1: designer - 設計タスク
Task 2: engineer - 調査タスク
→ 同時に実行開始

// 順次実行（別々のメッセージ）
メッセージ1: Task 1 実行 → 完了を待つ
メッセージ2: Task 2 実行
```

---

## 12. コスト意識したエージェント運用

### 12.1 原則

**トークン消費を意識し、コスト効率の良いエージェント運用を行う**

### 12.2 モデル選択

| タスク複雑度 | 推奨モデル | spawn時の指定 |
|-------------|-----------|--------------|
| 軽量（ログ更新、定型作業） | Haiku | `"model": "haiku"` |
| 標準（実装、レビュー） | Sonnet | 省略可（デフォルト） |
| 複雑（設計、アーキテクチャ） | Opus | `"model": "opus"` |

### 12.3 コスト管理コマンド

| コマンド | 用途 | タイミング |
|---------|------|-----------|
| `/cost` | セッションコスト確認 | タスク完了後 |
| `/stats` | 使用パターン確認 | 日次 |
| `/clear` | コンテキストクリア | タスク切り替え時 |
| `/compact` | コンテキスト圧縮 | 長時間セッション中 |

### 12.4 日次チェックリスト

```
セッション開始:
□ /cost でベースライン確認
□ 前回の続きか？ → /resume 検討

タスク完了時:
□ /cost で消費確認
□ 次タスクと関連なし？ → /clear

セッション終了:
□ /cost で最終コスト確認
□ /rename でセッション名付け
```

---

## 13. Agent Teams（実験的機能）

### 13.1 概要

Agent Teamsは複数のClaude Codeインスタンスが協調作業を行う機能。現行のSubagents（Wave方式）とは異なり、チームメイト間で直接通信が可能。

**有効化**: `/.claude/settings.json` で `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

### 13.2 Subagents vs Agent Teams

| 項目 | Subagents（現行） | Agent Teams |
|------|------------------|-------------|
| 通信 | オーケストレーター経由 | 直接メッセージ |
| タスク管理 | Wave方式（手動） | 共有タスクリスト（自動） |
| 最適用途 | 独立した並列タスク | 協調が必要なタスク |
| コスト | 低 | 高（約7倍） |

### 13.3 使用判断

| 状況 | 推奨 |
|------|------|
| 独立タスクの並列実行 | Subagents（Wave） |
| 相互議論・協調が必要 | Agent Teams |
| コスト重視 | Subagents |
| 品質・深い分析重視 | Agent Teams |

### 13.4 制限事項

| 制限 | 説明 |
|------|------|
| セッション再開 | `/resume`でチームメイト復元不可 |
| 1セッション1チーム | 同時複数チーム不可 |
| ネストなし | チームメイトは子チーム作成不可 |

---

**Last Updated**: 2026-03-01 (セクション7.2 学習成果追加、セクション10.3 プロンプトテンプレート改善)
**Next Review**: As needed
