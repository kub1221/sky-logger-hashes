Sky Logger Hash Archive

Sky Logger App — Tamper-proof Log Hash Repository

このリポジトリは、Sky Logger App が自動生成した撮影ログの SHA-256 ハッシュ値を公開保存するためのアーカイブです。
アップロード処理は Firebase Cloud Functions により自動で行われ、すべての記録が GitHub 上で改ざん不能な形で保持されます。

📂 Repository Structure
/hashes/
    {logId}.txt     ← Sky Logger が生成した SHA-256 ハッシュ（1行のみ）
ledger.json         ← 公開用の簡易台帳（id / videoId / hash / createdAt）
verify.html         ← 3者照合ツール（URL → Firestore → GitHub）
auto.html           ← 自動照合版（内部用）


各ファイルについて：

🔹 hashes/{logId}.txt

1つの Firestore ログ（public_logs/{logId}）に対応

内容は SHA-256 ハッシュ値 1行のみ

App で生成されたハッシュを Functions がそのまま保存します
（＝GitHub 側では一切計算せず “受け取った値を固定化”）

🔹 ledger.json

すべての公開ログの 最小限のメタデータ台帳

id

videoId

hash

createdAt (UTC)

Full JSON（位置情報・UUIDなど）は非公開で、プライバシーを保護します。

🔹 verify.html

公開ログの真偽を次の 3者で自動照合します：

QRコードの URL パラメータ

Firestore public_logs

GitHub /hashes のハッシュ

一致した場合、ログは改ざんされていないことが確認できます。

🔍 Verification Logic

Sky Logger のログには以下の6項目が含まれます：

uuid
videoId
finalFileName
lat
lng
createdAt


アプリ側では以下の手順でハッシュを生成しています：

上記6項目のみを抽出

キーを辞書順（アルファベット順）でソート

Minify JSON に変換

UTF-8 バイト列に対して SHA-256 計算

結果を Firestore → GitHub に保存

✔️ Example
$ cat hashes/abc123.txt
4e08f05a9d83df0a7791df272c6e19c50f6bff7d95f75c979b1f34cd7b0f6021


照合には verify.html を利用してください：

https://kub1221.github.io/sky-logger-hashes/verify.html?videoId=abc123


このページは
Firestore → GitHub → URL
の三方向の整合性を確認し、改ざんの痕跡がないか自動検証します。

🔒 Security Philosophy

Sky Logger は
「位置情報や UUID の完全記録」 と
「公開時のプライバシー保護」
を両立させるために次の設計となっています。

内部 Firestore logs：位置情報・UUID を完全保持

公開 Firestore public_logs：位置情報・UUID を含まない

GitHub：純粋なハッシュ値のみ

verify.html：ハッシュの一致のみを検証（内容の公開はしない）

これにより、

証明強度は C2PA に近い水準

プライバシーは完全に保持

第三者は内容を知らなくても「改ざんの有無」だけを確認できる

という最高強度 × 安全設計になっています。

📄 License & Disclaimer

This repository contains tamper-evident cryptographic records only.
It does not include personal information, location data, or images.
These hashes are published to provide independent third-party verification.
