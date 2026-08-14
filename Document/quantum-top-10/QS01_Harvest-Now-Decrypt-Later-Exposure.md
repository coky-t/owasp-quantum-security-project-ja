## QS01:2026 - Harvest-Now-Decrypt-Later の露出 (Harvest-Now-Decrypt-Later Exposure)

**説明:**

敵対者は暗号化されたトラフィックや保存された暗号文をすでにキャプチャしており、暗号関連量子コンピュータ (CRQC) が存在するまで将来の復号化に向けて保持しています。関連する演算は一般的に、RSA、有限体 Diffie-Hellman、あるいは楕円曲線 Diffie-Hellman によって保護された鍵確立のステップですが、いずれも Shor のアルゴリズムによって多項式時間で破られてしまいます。セッション鍵が復元されると、対称暗号文がそれに続きます。金融記録、健康データ、ソースコード、知的マテリアル、契約や商取引の秘密など、機密性の存続期間が重要なデータを扱う組織は、これらのプリミティブをベースとする現行の TLS、VPN、保存時の暗号化を、将来的に読み取り可能なものとして取り扱わなければなりません。このリスクは、量子ハードウェアの実現に左右されるものではなく、すでに現実のものとなっています。

**脆弱性のよくある例:**

1. 機密性要件が2030年を超えて延びているデータセット (医療記録、知的財産、長期保持を義務付けられた個人データ、政府および防衛データ) のうち、従来の公開鍵暗号のみで保護されているもの。
2. 脆弱な鍵確立を使用する通信経路やチャネルの保護: RSA、ECDH、有限体 DH に依存している TLS エンドポイント、VPN トンネル、暗号化バックアップチャネル、アーカイブストレージの暗号化、衛星通信、マイクロ波リンク。
3. 信頼できない境界を通過する暗号化されたトラフィックのうち、受動的に記録および保存され、後に復号の恐れがあるもの。
4. PQC に移行されたセッション暗号化のうち、長期有効な証明書や鍵ラップ用鍵が従来のアルゴリズムのまま残されているもの。

**防御方法:**

1. Migrate vulnerable channels to hybrid post-quantum TLS using ML-KEM (FIPS 203) where the platform supports it.
2. For data at rest, layer a PQC-protected encryption envelope over existing classical encryption for the highest-sensitivity datasets.
3. Rotate symmetric data keys protected by quantum-vulnerable wrapping more frequently to reduce the volume exposed by any single recovered key.
4. Reduce data retention where the business case allows - data not retained cannot be decrypted later.

**攻撃シナリオの例:**

Scenario #1: An adversary passively records TLS-protected traffic as it crosses an untrusted network boundary today. The handshake used RSA or ECDH key establishment. The captured ciphertext is archived. Once a CRQC becomes available, the adversary recovers the session key via Shor's algorithm and decrypts years of previously confidential traffic retroactively.

Scenario #2: An organisation encrypts long-retention backups at rest with AES-256, but the AES data key is wrapped with RSA. An attacker exfiltrates the encrypted backups and the wrapped keys. Because the quantum-vulnerable layer is the RSA key-wrapping - not the symmetric cipher - the attacker recovers the wrapping key with a future CRQC and unwraps the AES keys, exposing the entire archive.

**参考情報リンク:**

<!-- References verified 2026-07-13 against authoritative canonical sources. -->

1. [NIST FIPS 203 (ML-KEM)](https://csrc.nist.gov/pubs/fips/203/final): Key-establishment standard for post-quantum migration.
2. [UK NCSC - Timelines for migration to post-quantum cryptography (March 2025)](https://www.ncsc.gov.uk/guidance/pqc-migration-timelines): Migration timelines calling out long-lived sensitive data as a priority class.
3. [EU Coordinated Implementation Roadmap for PQC (June 2025)](https://digital-strategy.ec.europa.eu/en/library/coordinated-implementation-roadmap-transition-post-quantum-cryptography): End-2030 deadline prohibiting standalone quantum-vulnerable PKC for high-risk use cases.
4. [White House National Security Memorandum 10 (NSM-10)](https://bidenwhitehouse.archives.gov/briefing-room/statements-releases/2022/05/04/national-security-memorandum-on-promoting-united-states-leadership-in-quantum-computing-while-mitigating-risks-to-vulnerable-cryptographic-systems/) and [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf): Cite HNDL as the primary driver of US migration urgency.

**規格や規制のマッピング:**

> **TODO:** This section is carried over from the source document and is not part of `_template.md`. Confirm whether to keep it in the final entry format, and verify each standard/citation.

NIST FIPS 203 (ML-KEM) for key establishment. NCSC Timelines for migration to post-quantum cryptography (March 2025). EU Coordinated Implementation Roadmap (June 2025), end-2030 high-risk deadline. NSA CNSA 2.0 prioritises network encryption and long-lived secrets. NIS2 Article 21(2)(h) cryptographic policy obligation; DORA Article 9 confidentiality and integrity at rest, in use and in transit. NSM-10 and OMB M-23-02 cite HNDL as the migration driver.
