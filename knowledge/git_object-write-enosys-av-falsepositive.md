# git オブジェクト書き込みが内容依存で ENOSYS（Function not implemented）

初出: 2026-06-18 / 5000_GamePhilosohy のセットアップ中（R: ドライブ）

## 症状

R: ドライブ上のリポジトリで、`git add` / `git commit` / `git hash-object -w` が特定のファイルだけ次で失敗する:

```
error: unable to write file .git/objects/f4/c912e8aa2e596a6e980bda0600ea537b6a8e61: Function not implemented
error: <file>: failed to insert into database
fatal: adding files failed
```

- `core.fsync none`（別件の `fsync error: Bad file descriptor` 回避策）を入れた**後**でも出る別エラー。
- **特定ファイルの内容で決定的に再現**する。`hello world` や別ファイルは書けるのに、その内容（別名にコピーしても）だと
  同じオブジェクトハッシュで必ず失敗する。

## 切り分け（実際にやったこと）

1. **空き容量・ドライブ種別は無関係** … 969GB 空き、固定 NTFS。`ENOSPC` ではなく `ENOSYS`。
2. **生のファイル書き込みは成功** … `.git/objects/<dir>/` への `printf > file` は OK（最終パスへの直書きも OK）。mkdir も OK。
   → ディレクトリ作成・通常書き込み自体は壊れていない。
3. **`core.createObject rename` でも直らない** … link() が ENOSYS を返す環境向けの設定だが、効果なし。
   → 原因は loose object 確定時の link()/rename() ではない。
4. **内容依存と確定** … `hello world` / 既存 README は `hash-object -w` 成功。問題ファイルは別名コピーでも失敗。
   ファイル末尾に少し足して**実質的に内容を変える**と、別ハッシュで**成功**した。
5. **半分ずつだと通る** … 問題ファイルを上半分・下半分に分けると各々は書ける。全体（をまとめて圧縮したバイト列）だと失敗。

## 原因（推定）

**セキュリティソフト（アンチウイルス/EDR/DLP）が、git の loose object（zlib 圧縮後のバイト列）を
スキャンして誤検知し、ファイル作成をブロックしている**疑いが濃厚。Windows のブロックを MSYS（Git for Windows）が
`ENOSYS`（Function not implemented）にマップしていると見られる。

- 内容で決定的に再現 → 圧縮後のバイト列が特定パターン（誤検知シグネチャ）に一致するため。
- 半分なら通る → 圧縮結果が変わりパターンから外れるため。
- 文面を変えると通る → 同上。`fsync`/`link()` 等の I/O 設定で直らないのもこれで説明がつく（I/O ではなく内容スキャン）。

## 対処

- **応急**: 引っかかったファイルの**文面を実質的に書き換える**（言い回しを変える・順序を変える等）。圧縮結果が変わって通る。
  ※ 末尾に空白を足すだけ等の微差では、保持される本文の同じ窓が残り**効かないことがある**ので、本文を変える。
- **恒久（本筋）**: セキュリティソフトの**スキャン除外に `.git/`（または開発ドライブ R:）を追加**する（要管理者操作）。
  これが入れば内容に関係なく通る。
- 併せてこのリポジトリでは下記をローカル設定済み（R: ドライブの別 git 問題対策）:
  ```bash
  git config core.fsync none
  git config core.fsyncObjectFiles false
  git config core.createObject rename
  ```

## 影響プロジェクト

- R: ドライブ上で git を使う**全リポジトリ**（当環境固有・拠点依存の可能性）。
- 姉妹リポジトリ `0000_beernosusumesoft` の environment.md には `fsync error → core.fsync none` までは記載があるが、
  **この ENOSYS（内容依存・誤検知）は未記載**。同じ環境なら向こうにも横展開記載するとよい。

## 参考

- [../environment.md](../environment.md)「R: ドライブの git 回避策」
- 関連既知: R: ドライブでビルド時 `要求されたセクターが見つかりません`（CMake I/O）が一過性で出る → 再実行で通る（アプリ側 environment.md より）。
