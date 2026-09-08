# agent-skills

[English](README.md) | **日本語**

[`gh skill`](https://cli.github.com/manual/gh_skill)でインストールできるagent skill集。

| スキル | 目的 |
| --- | --- |
| `agmsg-team` | [agmsg](https://github.com/fujibee/agmsg)と`claude` / `codex`両方のCLIが入った環境で、セッションで選択したモデルをTask Ownerにしてピアチームを組み、実装・調査・QA・レビュー・相談を役割分担して進める。Claude Code / Codexのどちらからでも起動できる。 |
| `project-scaffold` | Org Standardを作成し、各プロジェクトに適用する。Built-in Starterを使う方法、既存プロジェクトやドキュメントからBootstrapする方法、既存のOrg Standardを使う方法に対応する。 |
| `project-scaffold-audit` | プロジェクトとOrg Standardの差分を確認し、`Local` / `Promote` / `Remove-Migrate` / `Needs-decision`に分類する。内容を確認・調整したうえで、承認された変更をOrg Standardに反映する。 |
| `project-scaffold-maintain` | このリポジトリのメンテナー向け。`project-scaffold-audit`で`Global Promote`とされた改善候補を確認し、Built-in Starterに取り込むためのPRを作成する。 |

## インストール

```sh
gh skill install bracelabs/agent-skills <skill-name>
```

## project-scaffoldの考え方と使い方

AIエージェント向けの初回セットアップ資料は、英語の
[project-scaffold setup guide](docs/project-scaffold-setup.md)を参照してください。

### 3つのレイヤー

- **Built-in Starter** — スキルに同梱の汎用テンプレート。
- **Org Standard** — チームや組織で共通して使う、プロジェクト構成やドキュメント運用の標準。本体は
  `$PROJECT_SCAFFOLD_HOME`（既定`~/.config/agent-skills/project-scaffold/`）配下の`scaffold/`に置かれる。Git管理すればチームや組織全体に配布・共有可能。
- **Project** — Org Standardを適用した個々のリポジトリ。

### project-scaffold — Org Standardを作って適用する

初回は、Org Standardをどう用意するか3つから選ぶ:

1. **Built-in Starterから作る** — 同梱テンプレートをベースにする。
2. **既存資産からBootstrapする** — 参照プロジェクト・AGENTS.md・開発ガイドラインなどを一度だけ読み取って作る（`project-scaffold-audit`が必要）。作成後に、参照プロジェクトで複数回auditを実行することを推奨。
3. **既存のOrg Standardを使う** — すでにあるもの（ローカルパスまたはGitリモート）を指定する。

- 1・2の場合は、作成後にGitの初期化とリモートリポジトリ作成まで案内する（不要なら省略可）。リモートは作っておく方がよい。各プロジェクトのmarkerにはそのURLが記録されるので、他のメンバーがauditしても同じStandardに解決される。ローカルパスだと解決できないか、その人のローカルStandardに解決されてしまう。
- 用意できたら各プロジェクトに適用する。適用は必ず「変更計画を提示 → 承認 → 最小変更」の順。CI・hook・Skillのように置いた時点で動くファイルは、ドキュメントとは分けて承認を取る。

### project-scaffold-audit — Org Standardを育てる

読み取る範囲はあらかじめ決まっている。ルート直下の`AGENTS.md` / `README.md` / `.gitignore`、`docs/README.md`、`docs/AGENTS.md`、`docs/00_templates/`、浅いディレクトリ構成まで。アプリケーションコード、シークレット、製品仕様は読まない。
Org Standardの`scaffold.config.json`に`operationalFiles`（プロジェクト相対のファイルパス一覧）を足せば、共通Skill、PRテンプレート、CI、独自の運用ガイドも対象にできる。列挙してよいのは運用ファイルだけで、開いてしまったファイルは読まなかったことにできない。
詳細は[検査範囲の定義](skills/project-scaffold/references/scope.md)を参照。
`tmp/`と`docs/`はStarterの既定値で、組織に合わせて変更できる。
Bootstrapでは参照プロジェクトの適用元から検査範囲を取得し、再適用では新旧両方の範囲を確認する。再適用後も残す旧パスと由来はProjectのmarkerに記録し、次回以降のauditでも読む。Standardの作業ツリーに未コミット変更がある場合は通知するが、適用は妨げない。

初回に作るOrg Standardは内容が薄く、複数回auditを繰り返すことで内容が充実していく。Standardに足したい・直したい点がたまってきたら`project-scaffold-audit`を実行する。

1回のaudit:

1. 適用時のOrg Standardとの差分を洗い出し、現行版にも照合する。取り込み済みの改善は再提案しない。
2. 各差分を分類する:
   - `Local` — このプロジェクト固有。そのまま残す。
   - `Promote` — 継続して使える技術非依存の運用で、他のプロジェクトにも再利用できる候補。1プロジェクトでの観測でも候補にでき、採用は人間が判断する。
   - `Remove-Migrate` — 古い・重複・標準と矛盾。整理する候補。
   - `Needs-decision` — 判断材料が足りない。
3. 決まったヘッダー（新旧のベースライン、検査範囲、確認できなかった範囲、分類ごとの件数）を持つレポートを書く。複数プロジェクト・複数回のレポートを並べて比較できるようにするため。出力先はプロジェクトの未追跡の作業用ディレクトリで、それ以外は変更しない。
4. 提案ごとに変更先を明記し、Org Standardへの変更は承認したものだけ反映する
   （Git管理ならブランチ・コミットを作成し、PR作成に対応したリモートがあればPR、なければローカルの差分報告まで。Git管理していなければ直接編集）。Project側の整理は
   `project-scaffold`へ引き渡し、変更計画の提示と承認を経て適用する。

Git管理ではPR作成やローカルでの作業完了後に開始時のブランチ（通常はmain）へ戻す。提案ブランチはレビュー用に残し、次回auditで現行標準と混同しないようにする。

汎用性が高くBuilt-in Starterにも入れるべきものは`Global Promote`としてフラグするだけに留める。取り込みは`project-scaffold-maintain`（メンテナー向け）。

## このリポジトリの保守

- `gh skill publish --dry-run`が通ること。リリースは`gh skill publish --tag vX.Y.Z`（GitHubリリースを作成し、利用者はタグからインストールする）。
- `skills/project-scaffold/starter/`は手編集しない — `project-scaffold-maintain`の役割。
- ローカル反復は`skills/<name>`を`~/.claude/skills/`と`~/.codex/skills/`にシンボリックリンクする。同じ名前のシンボリックリンクと`gh skill install`/`update`は併用しない（片方を消してからもう片方を使う）。
