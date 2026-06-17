# GamePhilosophy 共有ナレッジ

BeerGame（beernosusume.net）配下の**ゲーム開発**で、プロジェクトを跨いで効く知見を貯める場所。
「あのとき詰まったやつ、別タイトルでもまた踏むやつ」を 1 トピック 1 ファイルで残す。
特定タイトル固有の話は各プロジェクトの `docs/` に書き、ここには**横展開できる教訓だけ**置く。

> 姉妹リポジトリにアプリ開発側の正本 [0000_beernosusumesoft](https://github.com/kyamamoto5963/0000_beernosusumesoft)
> がある（Flutter アプリ群）。**ゲームは別エンジンを使う前提**なので、あちらの技術骨格は**参考程度**。
> 開発元・2拠点・git 作法・進め方などエンジン非依存の運用ルールはこちらに自己完結でコピーしている。

## 基本ドキュメント

- **[philosophy.md](philosophy.md)** — ゲーム開発の基本思想・土台（横断思想の揃え方／命名一貫性／作業スコープ／新規タイトルの進め方／シンプル基調＋演出）。**これが正本**。技術スタック（エンジン）は固定せず、タイトルごとに選定する。
- **[environment.md](environment.md)** — 開発環境（家/会社の2拠点・ツールパス・GitHub 命名・R: ドライブの git 回避策）。
- **[conventions.md](conventions.md)** — 開発者のクセ・進め方（「コミット」＝push まで、日付は絶対日付、リリース前チェック）。
- **[cross-project-changes.md](cross-project-changes.md)** — 横展開チェンジレジストリ。複数タイトルに同じ修正を波及させる変更を ID（CPC-NNNN）付きで管理。
- **[knowledge/](knowledge/)** — 罠・教訓（症状→原因→対処→影響→参考）。
- **[research/](research/README.md)** — 調査メモ（測ってみた／調べてみた）。

## ナレッジ索引

| トピック | 一言 | 影響範囲 |
|---|---|---|
| [git オブジェクト書き込みが内容依存で ENOSYS（Function not implemented）](knowledge/git_object-write-enosys-av-falsepositive.md) | `core.fsync none` 後も特定ファイルだけ書き込み失敗。`createObject rename` でも直らず、文面を変えたら通る＝セキュリティソフトの誤検知疑い | R: ドライブ上の全リポジトリ（当環境） |

## 書き方ルール

- `knowledge/<カテゴリ>_<短い説明>.md` でファイルを作る（例: `engine_*`, `build_*`, `git_*`, `perf_*`）。
- 各ファイルの先頭は **症状 → 原因 → 対処 → 影響プロジェクト → 参考** の順だと探しやすい。
- 「なぜそうなるか(Why)」と「次に踏んだらどう直すか(How)」を必ず書く。再現できる粒度で。
- 新規ファイルを足したら、この README の索引に 1 行追加する。
- 日付は絶対日付で書く（「先週」ではなく「2026-06-18」）。
