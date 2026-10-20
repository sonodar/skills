# sonodar/skills

[English](README.md) | 日本語（原本）

Claude CodeやCodexで使うSKILLをまとめたリポジトリです。
[skills CLI](https://github.com/vercel-labs/skills)で、必要なSKILLを選んで追加できます。

## SKILL一覧

| SKILL | 用途 |
| --- | --- |
| [skill-creator-sandbox](skills/skill-creator-sandbox/SKILL.ja.md) | 自分以外の人も使うSKILLが作成者の端末にしかない指示や別のSKILLに依存し、利用者の端末では同じように動かない問題を防ぐ。 |

## 追加する

Node.jsとnpmを用意し、次を実行してください。

```sh
npx skills add sonodar/skills
```

対話画面で、追加するSKILL、利用するエージェント、インストール先を選びます。
複数のプロジェクトで使う場合は、ユーザー共通のインストール先を選んでください。

追加できるSKILLを確認するだけなら、`--list`を付けます。

```sh
npx skills add sonodar/skills --list
```

SKILLを指定する場合は、`--skill`にSKILL名を渡します。
たとえば、skill-creator-sandboxだけを追加するには次を実行します。

```sh
npx skills add sonodar/skills --skill skill-creator-sandbox
```

## 使う

エージェントへの依頼に、使いたいSKILLの名前を添えてください。
用途、必要なツール、併用するSKILLは、一覧から各SKILL.mdを参照できます。

skill-creatorでSKILLを作成・更新・検証するときに読み込み、最初に自分以外の人も使うか確認します。
自分以外の人も使うなら依存レビューと検証環境の分離を適用し、自分だけで使うなら通常のskill-creatorで進めます。
未定なら、後から自分以外の人も使う可能性に備えて適用を提案し、ユーザーが選んだ場合に適用します。
会話ですでに誰が使うかわかっていれば聞き直しません。
社内やチーム内での共有も対象です。
作成者の端末では、ローカルの指示が説明を補ったり、別のSKILLが処理を助けたりして、検証に通ることがあります。
利用者の端末には同じ指示やSKILLがあるとは限らないため、その検証だけでは自分以外の人も使うSKILLが正しく動くと判断できません。
skill-creator-sandboxは、こうした依存を確認し、対象SKILLと必要な入力だけで検証するための手順を加えます。
自分だけで使うSKILLでは、依存レビューと検証環境の分離を省きます。

Claude CodeとCodexに対応しています。
普段使っているskill-creatorがあれば、そのまま使えます。
未導入の場合は、[Anthropicのskill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)を追加できます。

```sh
npx skills add anthropics/skills --skill skill-creator
```

依頼の例です。

> skill-creatorとskill-creator-sandboxを使って、コードレビュー用のSKILLを作ってください。

背景と不要になる条件は、[skill-creator-sandboxのREADME](skills/skill-creator-sandbox/README.ja.md)を参照してください。

## 更新する

skills CLIで追加したSKILLは、次で更新できます。

```sh
npx skills update
```

## SKILLを追加・修正する

SKILLごとに`skills/<SKILL名>/`を作り、`SKILL.md`を置きます。
frontmatterには`name`と`description`を書いてください。
補足資料やスクリプトが必要なら、同じディレクトリの中にまとめてSKILL.mdから参照します。

```text
skills/
├── skill-creator-sandbox/
│   ├── SKILL.md
│   ├── SKILL.ja.md
│   ├── README.ja.md
│   ├── README.md
│   └── references/
│       ├── claude.md
│       ├── claude.ja.md
│       ├── codex.md
│       └── codex.ja.md
└── <SKILL名>/
    └── SKILL.md
```

各文書は`.md`を英語の本体、対応する`.ja.md`を日本語の原本とします。
変更時は両方を更新してください。
追加したSKILLは、英語版と日本語版のREADMEの一覧にも載せてください。
構成やfrontmatterの詳細は、[skills CLIのSKILL作成ガイド](https://github.com/vercel-labs/skills#creating-skills)を参照してください。

リポジトリのルートで次を実行すると、手元のファイルからSKILLが見つかるか確認できます。
インストールは行いません。

```sh
npx skills add . --list
```
