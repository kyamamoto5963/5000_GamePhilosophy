# 開発環境メモ（拠点・ツール・GitHub）

最終更新: 2026-06-18

BeerGame 配下を開発する環境の前提。**機密（パスワード・鍵本体）はここに書かない**
（このリポジトリは GitHub に push される）。

## 2拠点開発（家 / 会社）

- **家と会社の2拠点**で同じプロジェクトを開発する。
- **ツールのドライブ/パスは拠点ごとに違う**。だから絶対パスをコードに埋め込まず、毎回その場で確認する。
- **セッション開始時に開発者が拠点（家／会社）を申告し、アシスタントは拠点依存の設定を切り替える。
  申告がなければ拠点依存の操作前に問いかける**（→ [philosophy.md](philosophy.md)「開発拠点をセッション開始時に申告する」）。
- 拠点差が出る値（パス等）は、ベタ書きせず**コメントアウトや条件分岐（define）で切り替えられる形**にしておく。
  拠点差が判明したら下表に追記する。

### 拠点差がある値の一覧

| 項目 | 会社 | 自宅 | 切り替え手段 | 備考 |
|---|---|---|---|---|
| （例）プロジェクトルート | | | コメントアウト | |
| ゲームエンジンのパス | | | その場確認 | エンジン未選定（TBD） |

## ツールの場所

- **ゲームエンジンは未選定（TBD）**。タイトルごとに選ぶ（→ philosophy「技術スタックは固定しない」）。
- 選定後、ここに「エンジン名・バージョン・拠点ごとの実例パス」を追記する。
- パスは拠点で異なる前提。絶対パスを埋め込まず、その場で確認するのが原則。

## GitHub

- ルート: **https://github.com/kyamamoto5963/**
- **リポジトリ名 ＝ プロジェクトフォルダ名**（この正本は `5000_GamePhilosohy`）。
- git identity: `user.name=yamamoto` / `user.email=yamamoto@dank.ne.jp`。
- 「コミット」と言われたら **push まで**やるのが既定 → [conventions.md](conventions.md)。

## R: ドライブの git 回避策（当環境のクセ）

R: ドライブ上のリポジトリでは、git のオブジェクト書き込みでドライブ依存のエラーが出ることがある。
このリポジトリでは下記をローカル設定済み（`.git/config`・git 管理外なのでクローンごとに必要）:

```bash
git config core.fsync none
git config core.fsyncObjectFiles false   # 古い git 向け
git config core.createObject rename       # link() が ENOSYS を返す環境向け
```

- **`fatal: fsync error on 'loose object file': Bad file descriptor`** → `core.fsync none` で回避。
- **`unable to write file .git/objects/.. : Function not implemented`（ENOSYS）が特定ファイルだけで出る** →
  内容依存で再現し、`createObject rename` でも直らない。**セキュリティソフトが git オブジェクト（zlib圧縮後の中身）を
  誤検知してブロックしている疑い**。文面を実質的に書き換えると通る。詳細・切り分け手順は
  [knowledge/git_object-write-enosys-av-falsepositive.md](knowledge/git_object-write-enosys-av-falsepositive.md)。
- 恒久的には、セキュリティソフトに **`.git/` をスキャン除外**に追加するのが本筋（要管理者操作）。
