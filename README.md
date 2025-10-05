# sky-logger-hashes
Sky Logger App の撮影ログ用ハッシュ公開リポジトリ
# Sky Logger Hash Archive

This repository contains SHA-256 hashes automatically uploaded from the Sky Logger App.
Each file under /hashes corresponds to one Firestore log document.

- Path format: hashes/{logId}.txt
- Each file contains a single SHA-256 hash string.

## Verification

To verify a record:

1. Retrieve the original JSON data (uuid, videoId, finalFileName, lat, lng, createdAt).
2. Sort keys alphabetically and encode as minified JSON.
3. Compute SHA-256 of the UTF-8 bytes.
4. Compare with the hash stored in this repository.

If the hashes match, the record has not been tampered with.

Example:

```bash
$ cat hashes/abc123.txt
4e08f05a9d83df0a7791df272c6e19c50f6bff7d95f75c979b1f34cd7b0f6021
