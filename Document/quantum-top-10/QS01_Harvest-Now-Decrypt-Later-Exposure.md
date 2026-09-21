## QS01:2026 - Harvest-Now-Decrypt-Later の露出 (Harvest-Now-Decrypt-Later Exposure)

**説明:**

敵対者は暗号化されたトラフィックや保存された暗号文をすでにキャプチャしており、暗号関連量子コンピュータ (CRQC) が存在するまで将来の復号化に向けて保持しています。関連する演算は一般的に、RSA、有限体 Diffie-Hellman、あるいは楕円曲線 Diffie-Hellman によって保護された鍵確立のステップですが、いずれも Shor のアルゴリズムによって多項式時間で破られてしまいます。セッション鍵やラッピング鍵が復元されると、対称暗号文がそれに続きます。このエントリでは、信頼できないネットワーク境界を通過する際に敵対者が暗号文を記録する、**転送時** に捕捉されるデータと、CRQC が存在する前の任意の時点ですでに暗号化されて保存しているものを敵対者が持ち出して時を待つ、**保存時** にすでに置いてあるデータである、このような収集のすべての領域をカバーします。いずれも、敵対者が現時点で入手可能であり、後に無効となるアルゴリズムによって保護された暗号文であるという点で、同じ危険があり、暗号文が現在どこにあるかのみが異なります。

Mosca の不等式は両者の領域における効果的な計画指針です。量子安全暗号への移行のための時間 (X) にデータに求められる機密保持期間 (Y) を足すと、CRQC 登場までの時間 (Z) を超える場合、CRQC が実際にいつ登場するかに関わらず、そのデータはすでに危険にさらされています。転送時のデータについては、これはチャネルがどの程度急いで移行する必要があるかを決定します。保存時のデータ (アーカイブ、バックアップ、規制されている個人データ、アイデンティティ記録、知的財産) については、既存のストアをどの程度急いで再暗号化する必要があるかを決定します。なぜなら、数十年の保持期間要件のある記録は、たとえ直ちに移行を開始したとしても、現時点でこの不等式を満たさない可能性があるためです。金融記録、健康データ、ソースコード、知的マテリアル、契約や商取引の秘密など、機密性の存続期間が重要なデータを扱う組織は、これらのプリミティブをベースとする現行の TLS、VPN、保存時の暗号化を、将来的に読み取り可能なものとして取り扱わなければなりません。このリスクは、量子ハードウェアの実現に左右されるものではなく、すでに現実のものとなっています。

**脆弱性のよくある例:**

1. 多くの規制当局が使用を想定する CRQC 計画 (2030 ～ 2035 年) を超えて機密性要件を維持するデータセット (医療記録、知的財産、長期保持を義務付けられた個人データ、政府および防衛データ) のうち、転送時のものやすでに保存されているもので、従来の公開鍵暗号のみで保護されているもの。
2. 脆弱な鍵確立を使用する通信経路やチャネルの保護: RSA、ECDH、有限体 DH に依存しており、受動的に記録および保存を可能とする信頼できない境界を通過する、TLS エンドポイント、VPN トンネル、暗号化バックアップチャネル、衛星通信、マイクロ波リンク。
3. 現在の暗号化ポリシー以前のもので、長期保持要件の下で保持されるアーカイブやバックアップであり、機密保持期間と移行所要時間を足すと Mosca の不等式 での CRQC 到来次期をすでに超えている。
4. AES-256 で暗号化されているが RSA または ECC でラップされている保存データ。量子脆弱な層は対称鍵よりも上位に位置するため、攻撃者の標的は暗号ではなくラッピング鍵となる。
5. PQC に移行されたセッション暗号化のうち、アーカイブデータを保護する、長期有効な証明書や鍵ラップ用鍵が従来のアルゴリズムのまま残されているもの。

**防御方法:**

1. Migrate vulnerable channels to hybrid post-quantum TLS using ML-KEM (FIPS 203) where the platform supports it.
2. Classify data by confidentiality lifetime, not just sensitivity - a medium-sensitivity record with a 30-year retention requirement may outrank a high-sensitivity record retained for two years, and Mosca's inequality is the tool for making that ranking explicit.
3. For the highest-priority archives identified by that classification, layer a PQC-protected encryption envelope over the existing classical encryption, sequenced against migration capacity.
4. Rotate symmetric data keys protected by quantum-vulnerable wrapping more frequently, to reduce the volume of data exposed by the recovery of any single wrapping key.
5. Reduce data retention where the business case allows - data not retained cannot be harvested or decrypted later.

**攻撃シナリオの例:**

Scenario #1: An adversary passively records TLS-protected traffic as it crosses an untrusted network boundary today. The handshake used RSA or ECDH key establishment. The captured ciphertext is archived. Once a CRQC becomes available, the adversary recovers the session key via Shor's algorithm and decrypts years of previously confidential traffic retroactively.

Scenario #2: A regulated entity retains personal records for a statutory 30-year period, encrypted with an RSA-wrapped AES key, held at rest rather than transmitted. An adversary exfiltrates the encrypted store today - no interception is needed, since the ciphertext already sits somewhere reachable. Applying Mosca's inequality, the confidentiality lifetime alone exceeds the CRQC horizon, so the records are effectively already compromised: the attacker recovers the wrapping key once a CRQC exists and decrypts the full archive, well within its required protection window.

**参考情報リンク:**

<!-- This entry merges the former QS01 (Harvest-Now-Decrypt-Later Exposure) and QS02 (Long-Lived Sensitive Data), consolidating the in-transit and at-rest confidentiality surfaces under Mosca's inequality as a single prioritisation frame, per the discussion in #11. QS02's integrity content (long-lived signed artefacts, identity and credential records) moves to QS03, which already covers signature trust. References verified 2026-08-05 against authoritative canonical sources. -->

1. [NIST FIPS 203 (ML-KEM)](https://csrc.nist.gov/pubs/fips/203/final): Key-establishment standard for post-quantum migration.
2. [NIST IR 8547 (Draft) - Transition to Post-Quantum Cryptography Standards](https://csrc.nist.gov/pubs/ir/8547/ipd): Transition planning guidance referencing Mosca's inequality.
3. [UK NCSC - Timelines for migration to post-quantum cryptography (March 2025)](https://www.ncsc.gov.uk/guidance/pqc-migration-timelines): Migration timelines calling out long-lived sensitive data as a priority class.
4. [EU Coordinated Implementation Roadmap for PQC (June 2025)](https://digital-strategy.ec.europa.eu/en/library/coordinated-implementation-roadmap-transition-post-quantum-cryptography): End-2030 deadline prohibiting standalone quantum-vulnerable PKC for high-risk use cases.
5. [White House National Security Memorandum 10 (NSM-10)](https://bidenwhitehouse.archives.gov/briefing-room/statements-releases/2022/05/04/national-security-memorandum-on-promoting-united-states-leadership-in-quantum-computing-while-mitigating-risks-to-vulnerable-cryptographic-systems/) and [OMB M-23-02](https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf): Cite HNDL as the primary driver of US migration urgency.
6. [EU Cyber Resilience Act - Regulation (EU) 2024/2847, Annex I](https://eur-lex.europa.eu/eli/reg/2024/2847/oj/eng): State-of-the-art protection required through the product support period, which for many products extends past 2030.

**規格や規制のマッピング:**

> **TODO:** This section is carried over from the source document and is not part of `_template.md`. Confirm whether to keep it in the final entry format, and verify each standard/citation.

NIST FIPS 203 (ML-KEM) for key establishment. NIST IR 8547 (Draft) on transition planning and Mosca's inequality. NCSC Timelines for migration to post-quantum cryptography (March 2025), long-lived data prioritisation. EU Coordinated Implementation Roadmap (June 2025), end-2030 high-risk deadline and standalone-PKC prohibition. NSA CNSA 2.0 prioritises network encryption and long-lived secrets. NIS2 Article 21(2)(h) cryptographic policy obligation; DORA Article 9 confidentiality and integrity at rest, in use and in transit. NSM-10 and OMB M-23-02 cite HNDL as the migration driver. CRA Annex I requires state-of-the-art protection through the product support period.
