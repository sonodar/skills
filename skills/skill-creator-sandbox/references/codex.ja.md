# Codexでの環境分離

[English](codex.md) | 日本語（原本）

この手順はCodex CLI 0.162.1を前提にしている。

## サブエージェントを使う条件

`fork_turns="none"`がある環境では、親の会話履歴を渡さずに起動できる。
ただし、AGENTS.md、メモリ、skills、developer instructionsが除外される保証にはならない。
[カスタムサブエージェント仕様](https://learn.chatgpt.com/docs/agent-configuration/subagents)には、メモリ・プラグイン・skillsを制限する設定がある。
起動ツールがその設定を受け付けるかは環境ごとに確認する。

親の履歴と対象外のローカル指示を除外でき、実際の入力や設定で確認できる場合に限り、サブエージェントを独立した動作検証に使う。
`fork_turns="none"`しか指定できない場合は、履歴を渡さないレビューとして扱い、環境分離済みと報告しない。

## 新規セッションで確認する

`codex exec --help`、`codex debug prompt-input --help`と[設定リファレンス](https://learn.chatgpt.com/docs/config-file/config-reference)を確認する。
`fork`は既存の履歴から分岐し、`queue`は既存セッションへ依頼を送る。どちらも指示を消す操作ではない。

プロジェクト外の検証用ディレクトリに、対象SKILL一式と入力だけを置く。
次の設定は追加のコンテキストを減らすための候補であり、これだけで完全に分離できるとは限らない。

| 設定・オプション | 役割と限界 |
| --- | --- |
| `project_doc_max_bytes=0` | プロジェクトのAGENTS.mdの読み込みを止める。グローバルの指示まで消えたと推測しない。 |
| `memories.use_memories=false` | 既存メモリの注入を止める。 |
| `features.memories=false` | メモリ機能を無効にする。 |
| `features.plugins=false` | プラグイン機能を無効にする。 |
| `skills.include_instructions=false` | skillsの指示ブロックを省く。個々のSKILLを利用不能にする設定ではない。 |
| `skills.bundled.enabled=false` | 同梱skillsを無効にする。 |
| `developer_instructions=""` | 設定から追加するdeveloper instructionsを空にする。管理者の指示は別に残り得る。 |
| `--ignore-user-config` | `codex exec`でユーザーのconfig.tomlを読まない。AGENTS.mdの除外とは別。認証は通常のCODEX_HOMEを使う。 |

入力を確認する際は、同じ作業ディレクトリと適用できる同じ設定で`codex debug prompt-input`を使う。
手元では`debug`に`--strict-config`を付けるとエラーになったため、対応を確認せず付けない。
`exec`専用オプションがあると完全に同じ条件にはならないので、違いも記録する。
入力全体にはローカル情報が含まれ得る。確認結果は必要な項目だけに絞り、外部へ丸ごと送らない。

このreferenceの作成時には、表の`-c`設定を指定しても、モデルに渡す入力にグローバルAGENTS.mdの本文が残った。
skills一覧とメモリ要約が消えていても、それだけでは不十分だった。
この組み合わせを、分離済みの起動コマンドとして使わない。

## グローバルの指示が残る場合

ユーザー設定を持ち込まない別OSユーザーや、既に利用を認められたコンテナなど、別の環境で新規セッションを使う。
普段のホームやプロジェクトを丸ごとマウントせず、対象SKILLと必要な入力だけを渡す。
認証はその環境で承認された方法を使い、認証ファイルを無断でコピーしない。
実行環境の新設が必要なら、必要な権限・依存ツール・費用を確認してから進める。

そこで設定と入力を確認し、対象外のAGENTS.md、メモリ、skills、追加指示が入っていないことを確かめる。
管理者の指示を解除して分離を達成しようとしない。残る場合は影響を記録する。
分離を確認できた環境内でのみ、対象SKILLを明示的に読む依頼を`request.txt`に書き、新規実行する。
入力確認に使った`-c`設定は、検証実行にもすべて渡す。呼び出しごとの設定なので、自動では引き継がれない。
次は基本形であり、確認済みの設定を足してから使う。設定が多い場合は実行用ファイルにまとめ、確認時と実行時の差を点検する。

```sh
codex exec --skip-git-repo-check --ephemeral --sandbox read-only --json - < request.txt > events.jsonl
```

書き込みが必要なテストは、許可された検証用ディレクトリに限って権限を設定する。
`--ephemeral`は履歴を保存しない指定であり、指示の分離やファイルアクセスの制限ではない。
実行中の読み取り先も確認し、対象外の指示を読んだ場合は検証結果に記録する。

別環境を用意できなければ、通常環境でのレビューまで行い、独立した動作検証は未実施と報告する。
Claude Codeでの検証に切り替える場合は、Codex固有の振る舞いを検証したとは扱わない。
