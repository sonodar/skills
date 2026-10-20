# sonodar/skills

[English](README.md) | 日本語（原本）

Claude CodeやCodexで使うSKILLをまとめたリポジトリです。
[skills CLI](https://github.com/vercel-labs/skills)で、必要なSKILLを選んで追加できます。

## SKILL一覧

| SKILL | 用途 |
| --- | --- |
| [skill-creator-sandbox](skills/skill-creator-sandbox/README.ja.md) | 自分以外の人も使うSKILLが作成者の端末にしかない指示や別のSKILLに依存し、利用者の端末では同じように動かない問題を防ぐ。 |

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

## 使う

一覧から使いたいSKILLのREADMEを開き、導入方法と使い方を確認してください。
エージェントへの依頼に、そのSKILLの名前を添えてください。

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
