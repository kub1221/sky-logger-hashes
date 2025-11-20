☁️ Sky Logger 検証ガイド（Verification Guide）

このドキュメントは Sky Logger App によって生成された
撮影ログおよびハッシュ値を、第三者が独立して検証できるように設計された
公式の検証ガイドです。

本アーカイブは
kub1221/sky-logger-hashes

として運用されています。

🎯 Sky Logger の目的

Sky Logger は、映像の真正性を技術的・暗号的に保証することを目的とした記録システムです。

具体的には：

その映像が実際に撮影されたものか

いつ・どこで・誰によって撮られたか

撮影後にデータ改ざんが行われていないか

これらを 暗号学的証明（SHA-256）＋物理的証拠（QRコードの撮影）＋公開検証（GitHub） によって担保します。

Sky Logger のログは

🔹 Firebase Firestore（内部原本）

🔹 GitHub（公開ハッシュアーカイブ）
の二重構造で保存され、
第三者が内容を独立に照合できるよう設計されています。

🔍 物理層（Physical Proof）

Sky Logger の最大の特徴は：

アプリが生成した QR コードを、撮影カメラが実際に映像として記録すること

この工程により、次の関係が成立します：

QRコードには UUID・位置情報・時刻・ハッシュが含まれる（アプリ署名）

その QR をカメラが物理的に「撮影」する（光学署名）

同じデータが Firestore と GitHub にも保存される（改ざん検知）

結果として、

映像内のQRコード = 撮影時点の真正ログ
という強い対応関係が生まれ、
現実の出来事として撮影された証拠になります。

⚙️ ハッシュと二重記録（Tamper-evidence）

撮影時、Sky Logger App は次の6項目から 正式ハッシュ（SHA-256） を生成します：

uuid  
videoId  
finalFileName  
lat  
lng  
createdAt（UTC）


手順：

この6項目を抽出

キーを辞書順にソート

Minify JSON 化

UTF-8 に変換

SHA-256 を計算

生成されたハッシュはそのまま GitHub に保存され
Firestore public_logs にも記録されます。

🔒 どちらかが改ざんされても、照合時に必ず「不一致」になります

（この原理により、後からの書き換えを完全に検出できます）

📂 リポジトリの構成
パス / ページ	内容
verify.html
	QRコードから開く、ログ照合用ページ
firestore_viewer.html
	Firestore ↔ GitHub の整合性を一括検証するツール
hashes/
	公開ハッシュファイル（ログ1件ごとに1ファイル）
ledger.json	全公開ログのミニマム台帳（id / hash / createdAt）
README_verification_ja.md	このドキュメント（検証ガイド）
🧾 検証手順（第三者検証用）
1️⃣ 映像内の QR コードをスキャン

スマホまたはPCでQRを読み取ると verify.html が開きます。

2️⃣ 表示された videoId・hash・createdAt を確認
3️⃣ ボタンから Firestore ↔ GitHub の照合ページへ移動
4️⃣ 自動検証の結果を確認

緑色（完全一致） → 改ざんなし
赤色（不一致） → データの改変が疑われる

5️⃣ 必要に応じて GitHub の原本ハッシュを直接確認

hashes/{videoId}.txt

6️⃣ 専門家向け（任意）

Firestoreログをエクスポートし、
自分で SHA-256 を再計算し、GitHub の値と一致するか検証できます。

🧠 信頼モデル（3階層）
層	名前	内容
① 物理層	Optical Proof	QRコードをカメラが実際に撮影したという光学的証拠
② デジタル層	Cryptographic Proof	ハッシュによる改ざん検知と Firestore–GitHub 二重記録
③ 公開層	Public Verification	誰でも検証可能な透明性のある仕組み

この3層により、
「この映像は特定の時間と場所で実際に撮影された現実の記録である」
という高強度の証明を実現します。

🔒 強度の根拠（3本柱）

アプリ署名（App Signature）
UUID・位置・時刻の6項目を署名的にハッシュ化

光学署名（Optical Signature）
QRコードを映像として撮影 → 「現実に存在した」物理証拠

公開検証（Public Verification）
GitHub＋Firestore による第三者検証と透明性

→ この三要素により、
Sky Logger は C2PA に近い強度 の真正性証明を、スマホアプリ単体で実現しています。

🧩 公開運用モデル

Firestore（内部原本）
位置情報・UUID を含むフルログを保持

GitHub（公開ログ）
ハッシュ値のみを公開し、プライバシーを保護

ledger.json（公開ミニ台帳）
各ログの存在証明（Proof-of-Existence）

透明性とプライバシー保護を両立し、
誰でも検証できる強い証明モデルを採用しています。

🌐 公式ポータル（Verification Portal）

📍 https://kub1221.github.io/sky-logger-hashes/

ここから次のすべてにアクセスできます：

QRログ照合ページ（verify.html）

Firestore整合性チェック（firestore_viewer.html）

公開ハッシュ一覧（hashes/）

公開台帳（ledger.json）

本ドキュメント（README_verification_ja.md）

© 2025 Sky Logger Project

Maintained by kub1221

All rights reserved.
