# Claude Codeでの環境分離

[English](claude.md) | 日本語（原本）

この手順はClaude Code 2.1.301を前提にしている。

## 推奨する方法

SKILL本文の動作検証には、新規CLIセッションで`--setting-sources ""`と`CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`を併用する。
ユーザー・プロジェクト・ローカルの設定を外し、auto memoryも別に止める。
MCPが不要なテストでは`--strict-mcp-config`も付ける。
対象SKILLをファイルとして明示的に読ませるため、これはdescriptionによる自動選択のテストではない。

公式の[設定読み込み仕様](https://code.claude.com/docs/en/agent-sdk/claude-code-features)、[CLI reference](https://code.claude.com/docs/en/cli-reference)、手元の`claude --help`を確認して使う。
設定を外しても、製品標準の指示、組み込みskills、管理者の設定、`~/.claude.json`などは残る。
すべてのローカル情報を遮断する方法や、ファイルアクセスを制限するサンドボックスとして扱わない。

## 新規セッションで対象SKILLを実行する

1. プロジェクト外に検証用ディレクトリを作り、対象SKILL一式と必要な入力だけをコピーする。周辺のCLAUDE.md、AGENTS.md、rules、設定ファイルは持ち込まない。
2. `request.txt`に、対象SKILLのコピーをReadで読んで実行する依頼を書く。期待する答えや作成時の会話は渡さない。
3. 検証用ディレクトリで新規起動し、標準出力・標準エラー・debugログを保存する。`--resume`、`--continue`、`--fork-session`は使わない。

読み取りだけで完了するテストでは、次を起動例にする。
ユーザーに実行を任せる場合は、実際のパスを確定した実行用ファイルに書き、短い起動コマンドだけを案内する。

```sh
env -u CLAUDE_CODE_SAFE_MODE -u CLAUDE_CODE_DISABLE_CLAUDE_MDS \
  CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 \
  claude -p --setting-sources "" --strict-mcp-config \
    --permission-mode dontAsk --allowedTools Read \
    --output-format stream-json --verbose --no-session-persistence \
    --debug-file ./eval.debug.log \
    < request.txt > events.jsonl 2> stderr.txt
```

先頭の`env -u`は、safe modeの親から継承した変数を外すための指定である。
管理者が強制する設定を解除する目的には使わない。
実測では、ネストしたCLI起動に使われるセッション関連の環境変数も外して比較した。
親から起動するときは、継承した環境を確認し、ネスト起動が拒否されたら、その実行を成功扱いせず独立したターミナルから新規起動する。
環境変数やログに含まれる認証情報を表示・転記しない。

`--allowedTools Read`はReadへの許可であり、ツール一覧をReadだけにする指定ではない。
使えるツールをさらに絞る場合は`--tools`を使い、テスト対象に必要な能力を失わないようにする。
書き込みや外部ツールが必要なテストでは、許可済みの作業範囲と依存先に合わせて起動設定を決める。
操作を拒否された場合は、SKILLの不具合と区別する。

対象SKILLを検証用ディレクトリの外に置く場合は、必要なコピー先だけを`--add-dir`で追加する。
日常のホームやプロジェクト全体は追加しない。
`--settings`で追加した設定は、`--setting-sources ""`でも有効なので、観測用フックなど明示したものに限る。

## サブエージェントを使う条件

通常の非forkサブエージェントは親の会話履歴を引き継がないが、指示ファイルなどは別経路で読み込む。
`omitClaudeMd: true`だけでは、他のskillsやプラグインのフックまで除外できない。
実測では、`omitClaudeMd: true`と`tools: ["Read"]`を指定したサブエージェントへ、インストール済みプラグインの`SubagentStart`フックが追加指示を渡していた。
サブエージェントの自己申告には現れず、debugログで見つかった。

サブエージェントを使うなら、親の設定からフック・skills・メモリの経路も点検する。
そのうえで、履歴を引き継がず、永続メモリを指定せず、対象SKILLと最小限の入力だけを渡す。
委任文に作成側の原則が混ざっていないかも確認する。

[公式のサブエージェント仕様](https://code.claude.com/docs/en/subagents)では、カスタムサブエージェントに独自promptを指定すると、Claude Code標準のsystem promptを置き換える。
通常のClaude Codeでの動作を確かめたい場合は、新規CLIセッションのメインスレッドを使う。
`omitClaudeMd`を`--agent`によるメインセッション起動に使っても、同じ効果は得られない。

## 分離できたか確認する

起動設定、指示の読み込みを記録したログ、実際のツール呼び出し、出力を合わせて確認する。
識別用の文字列を入れた架空の指示ファイルと別SKILLを使い、通常起動では読み込まれることも比較すると、確認漏れを見つけやすい。
文字列が出なかったことや、期待どおり動いたことだけでは、指示が入っていない証明にはならない。

- `InstructionsLoaded`フックで読み込みを記録する場合、AGENTS.mdの直接読み込みなど、イベントが出ない経路もある。ツール履歴やdebugログと照合する。
- debugログの`additionalContext`は、フックから追加指示が入る経路を調べる手がかりになる。
- initの`plugins`には、safe modeで無効になったプラグインも表示された。一覧だけで有効・無効を判定しない。
- initの`skills`とモデルに見えるskills一覧は一致するとは限らない。自動選択を調べる場合は、実際の`Skill`呼び出しを確認する。

生ログは検証用ディレクトリに保存し、集計は保存済みのファイルから行う。
ログを共有する前に認証情報や不要なローカル情報が含まれないか確認する。

## safe modeとbareの位置づけ

`--safe-mode`でも、架空のローカル指示を除外し、対象SKILLを明示的に読ませて実行できた。
ただしユーザー設定の`language`が残り、`--agents`は無視され、観測用に渡したフックも動かなかった。
公式仕様では、管理者のpolicy settingsは残る一方、managed CLAUDE.md・skills・pluginsなども無効になる。
このため、管理者から配布された指示を保って比較する用途では、safe modeを第一候補にしない。

`--bare`は認証方法と標準promptにも違いがある。
OAuthを使う通常セッションと同じ条件で比較する方法として扱わない。

## 確認済みの範囲と限界

Claude Code自身による小規模な比較実験では、推奨構成で架空のCLAUDE.md・AGENTS.md・rules・auto memory・別SKILLが読み込まれず、対象SKILLから期待する出力を得た。
ただし比較実験はサブエージェントも起動しており、結果が出た後、待機処理が制限時間に達してプロセスを打ち切った。
上のメインスレッドだけで行う起動例そのものの正常終了は未検証であり、次回の実行で確認する。

管理者の指示が実際に維持されるかは、この端末では未検証である。
組み込みskillsや`~/.claude.json`など、残る入力も明示する。
descriptionによる自動選択、書き込みを伴うSKILL、サブディレクトリの指示を後から読むケースは、この実験では検証していない。
自動選択を検証する場合は対象SKILLを探索対象に含める別の構成を用意し、本文の動作検証と混同しない。
