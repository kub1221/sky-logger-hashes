# Sky Logger Hash Archive

**Sky Logger App — Tamper-proof Log Hash Repository**

このリポジトリは、Sky Logger App が自動生成した撮影ログの SHA-256 ハッシュ値を公開保存するためのアーカイブです。
アップロード処理は Firebase Cloud Functions により自動で行われ、すべての記録が GitHub 上で改ざん不能な形で保持されます。

## 📂 Repository Structure
* `/hashes/{logId}.txt`     ← Sky Logger が生成した SHA-256 ハッシュ（1行のみ）
* `ledger.json`         ← 公開用の簡易台帳（id / videoId / hash / createdAt）
* `verify.html`         ← 3者照合ツール（URL → Firestore → GitHub）
* `auto.html`           ← 自動照合版（内部用）


### 各ファイルについて：

**🔹 hashes/{logId}.txt**
* 1つの Firestore ログ（public_logs/{logId}）に対応
* 内容は SHA-256 ハッシュ値 1行のみ
* App で生成されたハッシュを Functions がそのまま保存します（＝GitHub 側では一切計算せず “受け取った値を固定化”）

**🔹 ledger.json**
* すべての公開ログの 最小限のメタデータ台帳
* `id` / `videoId` / `hash` / `createdAt (UTC)`
* Full JSON（位置情報・UUIDなど）は非公開で、プライバシーを保護します。

**🔹 verify.html**
公開ログの真偽を次の 3者で自動照合します：
1. QRコードの URL パラメータ
2. Firestore public_logs
3. GitHub /hashes のハッシュ

一致した場合、ログは改ざんされていないことが確認できます。

---

## 🔍 Verification Logic

Sky Logger のログのハッシュ計算には、以下の基本6項目が使用されます：
* `uuid`
* `videoId`
* `finalFileName`
* `lat`
* `lng`
* `createdAt`

さらに、**ペア撮影モード**（空と地上の同時記録）が有効な場合は、以下の2項目が追加され、最大8項目で計算されます：
* `pairGroupId` （空と地上を結びつける共通ID）
* `role` （"sky" または "ground"）

アプリ側では以下の手順で改ざん防止ハッシュを生成しています：
1. 上記の該当項目を抽出（ペア情報がない場合は基本6項目のみ）
2. キーを辞書順（アルファベット順）でソート
3. Minify JSON に変換
4. UTF-8 バイト列に対して SHA-256 計算
5. 結果を Firestore → GitHub に保存

---

## ✔️ Example
```bash
$ cat hashes/abc123.txt
4e08f05a9d83df0a7791df272c6e19c50f6bff7d95f75c979b1f34cd7b0f6021
