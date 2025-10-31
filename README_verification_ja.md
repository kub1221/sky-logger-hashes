# ☁️ Sky Logger 検証ガイド（Verification Guide）

このドキュメントは **Sky Logger App** によって生成された  
撮影ログおよびハッシュ値を、第三者が検証できるように設計された  
**公式検証ガイド** です。

> 本アーカイブは [`kub1221/sky-logger-hashes`](https://github.com/kub1221/sky-logger-hashes)  
> として運用されています。

---

## 📸 Sky Logger の目的

Sky Logger は、映像の真正性を保証するためのシステムです。

> 「いつ・どこで・誰が撮影した映像か」  
> 「その映像がデジタル的にも物理的にも改ざんされていないか」

を技術的に証明可能にすることを目的としています。

撮影時にアプリが生成するログ情報を  
- 🔹 **Firebase Firestore（原本）**  
- 🔹 **GitHub（公開アーカイブ）**

の二重に保存し、改ざんの有無を第三者が独立して確認できる仕組みを備えています。

---

## 🔍 物理的証明レイヤー（Physical Verification Layer）

Sky Logger の最大の特徴は、  
**アプリが生成したQRコードを撮影カメラが実際に映像として撮影する** 点にあります。

このプロセスにより：

1. QRコードには UUID・位置情報・時刻・ハッシュ が含まれ、アプリが署名的に生成  
2. そのQRコードが映像のフレーム内に「物理的証拠」として記録  
3. 同時に Firestore と GitHub に同一データが保存  

結果として、
> 「映像内のQRコード」＝「当時アプリが発行した真正な撮影ログ」  
という関係が確立され、映像の真実性を物理的・暗号的に担保します。

---

## ⚙️ ハッシュと二重記録の仕組み

1️⃣ 撮影時、アプリは UUID・時刻・位置情報などをまとめ、 **SHA-256 ハッシュ** を生成  
2️⃣ そのハッシュを **GitHub（`hashes/`）** に自動アップロード  
3️⃣ 同時に **Firestore** にも同一ログを保存  
4️⃣ どちらかが改ざんされても、照合時に「不一致」として検出  

この二重記録により、  
**撮影時点で生成されたログが後から改変されていない** ことを保証します。

---

## 🔗 リポジトリ構成

| ページ / フォルダ | 役割 |
|------------------|------|
| [verify.html](verify.html) | QRコードから開く、撮影ログの内容確認ページ |
| [firestore_viewer.html](firestore_viewer.html) | FirestoreとGitHubのログ整合性を自動検証（全体・個別モード対応） |
| [hashes/（GitHub上で開く）](https://github.com/kub1221/sky-logger-hashes/tree/main/hashes) | 公開ハッシュ原本フォルダ（各ログ1ファイル） |
| [README_verification_ja.md](README_verification_ja.md) | 本ドキュメント。システム構造・検証手順・信頼モデルを記述 |

---

## 🧾 検証手順（第三者向け）

1️⃣ **撮影映像に映る QR コード** をスマートフォンやPCでスキャン  
　→ `verify.html` が開く  

2️⃣ 表示された **UUID・位置情報・撮影時刻・ハッシュ値** を確認  

3️⃣ ページ下のリンク  
　「🔗 Firestore と GitHub の照合結果を見る」 をタップ  
　→ `firestore_viewer.html` が開く  

4️⃣ Firestore と GitHub 上の同一 `videoId` のハッシュが一致していることを確認  

5️⃣ 必要に応じて  
　[hashes/](https://github.com/kub1221/sky-logger-hashes/tree/main/hashes) 内の  
　該当ログファイルを開き、原本ハッシュを直接参照  

6️⃣ 一致していれば「改ざんなし」と判定可能  

> ※専門家は必要に応じて、Firestore からエクスポートした JSON をもとに  
> キーをアルファベット順にソート → minified JSON に変換 → UTF-8 → SHA-256 を計算し、  
> `hashes/` の値と一致することを確認できます。

---

## 🧠 信頼モデル（3層構造）

| 層 | 役割 | 内容 |
|----|------|------|
| ① 物理層 | カメラでQRを撮影 | 「現実にその瞬間が存在した」光学的証拠 |
| ② デジタル層 | Firestore と GitHub | 「改ざんされていない」二重記録による証拠 |
| ③ 検証層 | verify.html / firestore_viewer.html | 誰でも検証できる公開透明性 |

これら3層の連携により、  
**「この映像は特定の時間・場所で実際に撮影された現実の記録である」**  
ことを技術的に保証します。

---

## 🔒 信頼の根拠（3要素）

1️⃣ **アプリ署名（App Signature）**  
　UUID・緯度経度・時刻・ハッシュ → Firestore & GitHub による暗号的証明  

2️⃣ **光学署名（Optical Signature）**  
　QRコードが映り込む映像 → 物理的・視覚的な現実証拠  

3️⃣ **公開検証（Public Verification）**  
　誰でも照合可能 → 改ざん不可能な第三者検証モデル  

---

## 🧩 公開運用モデル

- Firestore：撮影アプリが内部で保持する**原本データベース**  
- GitHub：`sky-logger-hashes` に自動送信される**公開ログアーカイブ**  
- ledger.json：全公開ログの整合履歴を固定化する**ハッシュ台帳**  

この三層を組み合わせることで、  
Sky Logger は「映像の真正性」をオープンかつ検証可能な形で保証します。

---

## 🌐 公式ポータル（検証入口）

📍 [https://kub1221.github.io/sky-logger-hashes/](https://kub1221.github.io/sky-logger-hashes/)

ここから  
- QR検証ページ（verify.html）  
- Firestore整合性検証ページ（firestore_viewer.html）  
- 公開ハッシュリスト（hashes/）  
- 本ガイド（README_verification_ja.md）  

にアクセスできます。

---

## © 2025 Sky Logger Project
Maintained by [kub1221](https://github.com/kub1221)  
All rights reserved.
