## QS01:2026 - Harvest-Now-Decrypt-Later の露出 (Harvest-Now-Decrypt-Later Exposure)

**説明:**

敵対者は暗号化されたトラフィックや保存された暗号文をすでにキャプチャしており、暗号関連量子コンピュータ (CRQC) が存在するまで将来の復号化に向けて保持しています。関連する演算は一般的に、RSA、有限体 Diffie-Hellman、あるいは楕円曲線 Diffie-Hellman によって保護された鍵確立のステップですが、いずれも Shor のアルゴリズムによって多項式時間で破られてしまいます。セッション鍵が復元されると、対称暗号文がそれに続きます。金融記録、健康データ、ソースコード、知的マテリアル、契約や商取引の秘密など、機密性の存続期間が重要なデータを扱う組織は、これらのプリミティブをベースとする現行の TLS、VPN、保存時の暗号化を、将来的に読み取り可能なものとして取り扱わなければなりません。このリスクは、量子ハードウェアの実現に左右されるものではなく、すでに現実のものとなっています。

**脆弱性のよくある例:**

1. 機密性要件が2030年を超えて延びているデータセット (医療記録、知的財産、長期保持を義務付けられた個人データ、政府および防衛データ) のうち、従来の公開鍵暗号のみで保護されているもの。
2. 脆弱な鍵確立を使用する通信経路やチャネルの保護: RSA、ECDH、有限体 DH に依存している TLS エンドポイント、VPN トンネル、暗号化バックアップチャネル、アーカイブストレージの暗号化、衛星通信、マイクロ波リンク。
3. 信頼できない境界を通過する暗号化されたトラフィックのうち、受動的に記録および保存され、後に復号の恐れがあるもの。
4. PQC に移行されたセッション暗号化のうち、長期有効な証明書や鍵ラップ用鍵が従来のアルゴリズムのまま残されているもの。

**防御方法:**

1. プラットフォームがサポートしている場合、脆弱なチャネルを、ML-KEM (FIPS 203) を使用するハイブリッドの耐量子 TLS に移行します。
2. 保存時のデータについては、機密性が極めて高いデータセットに対し、既存の従来型暗号に加え、PQC で保護された暗号エンベローブを重ねます。
3. 量子脆弱なラップによって保護されている対称データ鍵の入れ替え頻度を高め、単一の復元鍵によって露出される量を低減します。
4. ビジネスケースが許す限り、データ保持期間を短縮します。保持されていないデータは後から復号されることはありません。

**攻撃シナリオの例:**

シナリオ #1: 敵対者は、現時点で信頼できないネットワーク境界を通過する TLS 保護トラフィックを、受動的に記録します。ハンドシェイクは RSA や ECDH 鍵確立を使用します。捕捉した暗号文がアーカイブされます。CRQC が利用可能になると、敵対者は Shor のアルゴリズムでセッション鍵を復元し、過去数年にわたり機密とされていたトラフィックを遡って復号します。

シナリオ #2: 組織は長期保存バックアップを保存時に AES-256 で暗号化していますが、AES データキーは RSA でラップされています。攻撃者は暗号化されたバックアップとラップされたキーを密かに持ち出します。量子脆弱な層は対称暗号ではなく RSA キーラッピングであるため、攻撃者は将来 CRQC でラッピングキーを復元して AES キーをアンラップし、アーカイブ全体を露出します。

**参考情報リンク:**

<!-- References verified 2026-07-13 against authoritative canonical sources. -->

1. [NIST FIPS 203 (ML-KEM)](https://csrc.nist.gov/pubs/fips/203/final): Key-establishment standard for post-quantum migration.
2. [UK NCSC - Timelines for migration to post-quantum cryptography (March 2025)](https://www.ncsc.gov.uk/guidance/pqc-migration-timelines): Migration timelines calling out long-lived sensitive data as a priority class.
3. [EU Coordinated Implementation Roadmap for PQC (June 2025)](https://digital-strategy.ec.europa.eu/en/library/coordinated-implementation-roadmap-transition-post-quantum-cryptography): End-2030 deadline prohibiting standalone quantum-vulnerable PKC for high-risk use cases.
4. [White House National Security Memorandum 10 (NSM-10)](https://bidenwhitehouse.archives.gov/briefing-room/statements-releases/2022/05/04/national-security-memorandum-on-promoting-united-states-leadership-in-quantum-computing-while-mitigating-risks-to-vulnerable-cryptographic-systems/) and [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf): Cite HNDL as the primary driver of US migration urgency.

**規格や規制のマッピング:**

> **TODO:** This section is carried over from the source document and is not part of `_template.md`. Confirm whether to keep it in the final entry format, and verify each standard/citation.

NIST FIPS 203 (ML-KEM) for key establishment. NCSC Timelines for migration to post-quantum cryptography (March 2025). EU Coordinated Implementation Roadmap (June 2025), end-2030 high-risk deadline. NSA CNSA 2.0 prioritises network encryption and long-lived secrets. NIS2 Article 21(2)(h) cryptographic policy obligation; DORA Article 9 confidentiality and integrity at rest, in use and in transit. NSM-10 and OMB M-23-02 cite HNDL as the migration driver.
