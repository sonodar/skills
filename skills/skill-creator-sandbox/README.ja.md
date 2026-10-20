# skill-creator-sandbox

[English](README.md) | 日本語（原本）

skill-creatorでSKILLを作成・更新・検証するときに併用する補助SKILLです。
作成者の端末では検証に通っても、ローカルの指示や別のSKILLに依存しているために、利用者の端末では動かない問題を防ぎます。

## 追加する

Node.jsとnpmを用意し、次を実行してください。

```sh
npx skills add sonodar/skills --skill skill-creator-sandbox
```

Claude CodeとCodexに対応しています。
元のskill-creatorと併用するため、普段使っているskill-creatorがあれば、そのまま使えます。
未導入の場合は、[Anthropicのskill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)を追加できます。

```sh
npx skills add anthropics/skills --skill skill-creator
```

## 使う

依頼の例です。

> skill-creatorとskill-creator-sandboxを使って、コードレビュー用のSKILLを作ってください。

最初に自分以外の人も使うか確認します。会話ですでに誰が使うかわかっていれば聞き直しません。
社内やチーム内での共有も対象です。
自分以外の人も使うなら依存レビューと検証環境の分離を適用し、自分だけで使うなら通常のskill-creatorで進めます。
未定なら、後から自分以外の人も使う可能性に備えて適用を提案し、ユーザーが選んだ場合に適用します。
手順は[SKILL.ja.md](SKILL.ja.md)、エージェントごとの補足は[Claude Code](references/claude.ja.md)と[Codex](references/codex.ja.md)を参照してください。

## 背景と不要になる条件

作成者の端末では、CLAUDE.md・AGENTS.md・メモリが対象SKILLに足りない説明を補ったり、別のSKILLが処理を助けたりすることがあります。
利用者の端末には同じ指示やSKILLがあるとは限らないため、作成者の端末で検証に通っただけでは、利用者の端末でも動くと判断できません。
この補助SKILLは、こうした依存を確認し、対象SKILLと必要な入力だけで検証するための手順を加えます。
利用者にも必要な依存先は明示します。

skill-creatorに関する以下のIssue・PRで、関連する問題が扱われています。

- [Issue #1427](https://github.com/anthropics/skills/issues/1427): 並列実行で作られるコマンドの複製や既存のCLAUDE.mdからの参照が、トリガー検出に干渉する問題。
- [PR #1755](https://github.com/anthropics/skills/pull/1755): 検証処理を、呼び出し元のプロジェクトや競合するインストール済みSKILLから分離する修正。
- [PR #1298](https://github.com/anthropics/skills/pull/1298): トリガー検証を分離し、実行時の障害に対処する修正。

上記3件が問題の解消を伴ってクローズされ、使っているskill-creatorに修正が取り込まれれば、この補助SKILLは不要にする方針です。
修正なしのクローズや、別のPRに引き継ぐための未マージでのクローズだけでは、解消したと判断しません。
3件は主にトリガー検証を扱っているため、廃止前には、作成時のローカル指示への依存レビューと、利用するエージェントでの検証環境の分離も上流の手順で補えるか確認してください。
