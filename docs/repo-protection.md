# GitHub リポジトリの保護ルール(個人開発の OBS プラグイン用)

KMOdan5/clipboard-image-overlay で設定した内容を、ほかのリポジトリでも使えるようにまとめたもの。
新しいリポジトリを公開したら、上から順に設定する。

- 前提: 個人開発。外部からのコードの PR は受け付けない。不具合・要望は Google フォームと Issue のテンプレートで受け付ける
- 画面の名前は GitHub の英語表示に合わせている
- `<OWNER>/<REPO>` は自分のリポジトリに置き換える

---

## 1. General(Settings → General)

### Features
| 項目 | 設定 | 理由 |
|---|---|---|
| Wikis | オフ | 文書はリポジトリの `docs/` に置く |
| Issues | オン | 不具合・要望を受け付ける |
| Pull requests | オン・Collaborators only | 外部からのコードの PR は受け付けないため(`CONTRIBUTING.md`) |
| Sponsorships | オン(Sponsors を使うとき) | `.github/FUNDING.yml` と合わせて使う |
| Discussions | オフ | 窓口を増やさない |
| Projects | オフ | ロードマップは親 Issue と子 Issue で管理する |
| Preserve this repository | オン(既定) | GitHub Archive Program で長期保存される |

### Pull Requests
| 項目 | 設定 | 理由 |
|---|---|---|
| Allow merge commits | オン | `main` を `develop` に取り込む PR は「Create a merge commit」でマージする |
| Allow squash merging | オン | |
| Allow rebase merging | オン | |
| Always suggest updating pull request branches | オフ | |
| Allow auto-merge | オフ | マージは必ず自分の目で確認してから行う |
| Automatically delete head branches | **オン** | マージ後の作業ブランチを自動で消す |

### その他
- Default branch: `main`(README とマニュアルは `main` の内容が公開されるため)

---

## 2. ブランチの保護(Settings → Rules → Rulesets)

「New branch ruleset」で 1 つ作る。

| 項目 | 設定 |
|---|---|
| Ruleset name | `Protect branch` |
| Enforcement status | Active |
| Bypass list | なし(自分も含めて、必ず PR を通す) |
| Target branches | `Include default branch`<br>git-flow のときは `develop` も追加する(`Include by pattern` → `develop`) |
| Restrict deletions | オン |
| Block force pushes | オン |
| Require a pull request before merging | オン |
| └ Required approvals | `0`(1 人で開発するため。1 以上にすると自分の PR をマージできない) |
| └ Allowed merge methods | Merge / Squash / Rebase すべて |

これで、`main`(と `develop`)には直接 push できず、削除も強制 push もできなくなる。

### ブランチの運用
- 小さく更新が少ないリポジトリ: `main` と作業ブランチ(`feature/…`)だけ。保護は `main` だけ
- リリースしながら直していくリポジトリ: git-flow(`main` / `develop` / `feature/…` / `hotfix/…`)。保護は `main` と `develop`
  - 作業の PR は `develop` へ。リリースは `develop` → `main` の PR をマージしてからタグを付ける

---

## 3. タグの保護(Settings → Rules → Rulesets)

「New tag ruleset」で 1 つ作る。リリースのタグを、自分以外(と Claude 以外)が作ったり動かしたりできないようにする。

| 項目 | 設定 |
|---|---|
| Ruleset name | `Protect release tags` |
| Enforcement status | Active |
| Bypass list | `Repository admin`(Always allow)<br>Claude の GitHub アプリ(Always allow。Claude にタグを任せない場合は不要) |
| Target tags | `Include all tags` |
| Restrict creations | オン |
| Restrict updates | オン |
| Restrict deletions | オン |
| Block force pushes | オン |

- 注意: Claude の作業環境は、通信の制限でタグを作れない。タグは「Releases」の画面で、リリースを公開するときに自分で作る
- 一度公開したタグは、変更・削除しない(配布したファイルとタグの中身を一致させるため)

---

## 4. セキュリティ(Settings → Advanced Security)

| 項目 | 設定 | 内容 |
|---|---|---|
| Private vulnerability reporting | **オン** | 脆弱性を非公開で報告してもらう窓口。`SECURITY.md` から案内する |
| Dependency graph | オン | 公開リポジトリでは既定でオン |
| Dependabot alerts | オン | 依存しているものに脆弱性が見つかったら知らせる |
| Dependabot security updates | オン | 脆弱性を直す PR を自動で作る |
| Secret Protection(Secret scanning) | オン | パスワードやトークンをコミットしていないか調べる |
| Push protection | **オン** | トークンなどを含む push をその場で止める |
| Code scanning(CodeQL analysis) | **オン・Default setup** | コードの脆弱性を自動で調べる。下の設定で使う |

CodeQL の Default setup の設定(clipboard-image-overlay と同じ):
- Languages: `GitHub Actions` と `C/C++`(プラグインの言語。リポジトリの中身から自動で選ばれる)
- Query suite: `Default`
- Scan schedule: Weekly(週 1 回。`main` への push と PR でも動く)

文書だけのリポジトリ(`.github` など)では、調べるコードがないので不要。

---

## 5. GitHub Actions(Settings → Actions → General)

| 項目 | 設定 | 理由 |
|---|---|---|
| Actions permissions | Allow all actions(テンプレートの CI がほかのリポジトリの Action を使うため) | |
| Approval for running fork pull request workflows from contributors | **Require approval for all external contributors** | 外部の PR で、勝手に CI を動かされないようにする |
| Workflow permissions | **Read repository contents and packages permissions** | 書き込みが必要なジョブは、ワークフローのファイルの中で個別に許可する |
| Allow GitHub Actions to create and approve pull requests | オフ | |

※ この節は clipboard-image-overlay で設定値を API から読めなかった(Claude の作業環境の制限)。新しいリポジトリでは画面で上のとおりになっているかを確かめる。

---

## 6. リポジトリに置くファイル

| ファイル | 内容 |
|---|---|
| `LICENSE` | GPL-2.0(OBS 本体に合わせる) |
| `SECURITY.md` | 対象のバージョン(最新のリリースだけ)と、脆弱性は公開の Issue に書かず「Security」タブの「Report a vulnerability」から報告してもらうこと。日本語と英語 |
| `CONTRIBUTING.md` | 外部からのコードの PR は当面受け付けないこと。不具合・要望の送り方 |
| `.github/ISSUE_TEMPLATE/config.yml` | `blank_issues_enabled: false`(テンプレート以外で Issue を作れないようにする)と、Google フォームへのリンク(`contact_links`) |
| `.github/ISSUE_TEMPLATE/bug_report.yml` | 不具合報告のフォーム(OBS とプラグインの版、OS、手順、ログ) |
| `.github/ISSUE_TEMPLATE/feature_request.yml` | 要望のフォーム |
| `.github/FUNDING.yml` | GitHub Sponsors などの支援の案内(使うとき) |
| `CLAUDE.md` | Claude に守ってほしい進め方(ブランチ・コメント・文言の確認など) |

Google フォームは、URL の `entry.…=` に製品名を入れておくと、どのプラグインへの報告かが最初から選ばれた状態になる。リポジトリごとに URL を変える。

---

## 7. ラベル(Issues → Labels)

既定のラベルに、次の 3 つを足す。

| ラベル | 用途 |
|---|---|
| `backlog` | すぐには作らない案 |
| `roadmap` | 段階ごとの親 Issue |
| `accessibility` | 見やすさ・使いやすさの改善 |

---

## 8. 設定を確かめるコマンド(読むだけ)

`gh` コマンドで、設定が入っているかを確かめられる。何も変更しない。

```sh
R=repos/<OWNER>/<REPO>

# 1. General とセキュリティ
gh api $R --jq '{default_branch, has_wiki, has_projects, has_discussions, delete_branch_on_merge, allow_auto_merge, security_and_analysis}'

# 2・3. ルールセット(ブランチとタグ)
gh api $R/rulesets --jq '.[] | {id, name, target, enforcement}'
gh api $R/rulesets/<id>   # 中身を見る

# 4. 脆弱性の非公開報告
gh api $R/private-vulnerability-reporting

# 4. CodeQL(Default setup)
gh api $R/code-scanning/default-setup

# 5. Actions
gh api $R/actions/permissions
gh api $R/actions/permissions/workflow
```

---

## 設定を終えたらすること

- [ ] `main` に直接 push しようとして、断られることを確かめる
- [ ] 作業ブランチから PR を作り、マージ後にブランチが自動で消えることを確かめる
- [ ] 「Security」タブに「Report a vulnerability」が出ていることを確かめる
- [ ] 「New issue」でテンプレートとフォームへのリンクだけが出ることを確かめる
