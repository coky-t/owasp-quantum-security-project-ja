## QS03:2026 - 脆弱なシグネチャとコード署名 (Vulnerable Signatures and Code-Signing)

**説明:**

公開鍵シグネチャは、コード署名、サプライチェーン完全性、文書有効性、アイデンティティ証明書、トランザクション、長期的な否認防止を下支えしています。RSA, DSA, ECDSA, EdDSA はいずれも Shor のアルゴリズムによって破られるため、これらのシグネチャを検証して信頼を確立するあらゆるシステム (ソフトウェア更新、コンテナイメージ、ファームウェア、パッケージマネージャ、TLS 証明書階層、SBOM アテステーション、署名済み文書、ブロックチェーントランザクション) は CRQC が存在するとリスクがあります。機密性の侵害とは異なり、シグネチャ偽装は能動的な攻撃 (悪意のある更新、偽のアイデンティティ、不正なトランザクション、サプライチェーン侵害) を可能にします。耐量子シグネチャ標準 (ML-DSA / FIPS 204 および SLH-DSA / FIPS 205) はより大きな鍵やシグネチャを持ち、運用特性が異なり、ハードウェア信頼の起点、制約のあるデバイス、証明書エコシステムに無視できない影響を及ぼします。

機密性の露出 (QS01) に適用される Mosca の不等式は、シグネチャの信頼性にも同様に当てはまります。ただし、機密性ではなく完全性として不等式を読みます。移行のための時間 (X) に、シグネチャやクレデンシャルが信頼性を維持しなければならない期間 (Y) を足すと、CRQC 登場までの時間 (Z) を超える場合、そのシグネチャはすでに危険にさらされています。コントラクト、規制当局への書類、証跡記録、署名済みソフトウェアリリース、および政府発行のアイデンティティ、専門クレデンシャル、ルート証明書といった長期有効なアイデンティティやクレデンシャルの発行は、CRQC が実際にいつ登場するかに関わらず、現時点でこのテストを合格できないほど長い有効期間を設定していることがよくあります。

このスコープには、再確立の失敗も含みます。移行は、新しいシグネチャ検証、新しい証明書チェーン、アルゴリズムが準拠していることなど、すべての暗号論的対策を正しく満たしているとしても、新しいクレデンシャルを発行するプロセスはそうするために不適切な証跡を受け入れます。このエントリが偽造可能とすでに分類している鍵から所持証明を行い、それに基づいて強力な新しいクレデンシャルを発行するような登録フローは、検証可能な二つの証明書と、正規の移行と区別できない記録を生成します。このリスクはシグネチャが失敗することではありません。侵害されたアンカーをその代替のものを発行するために十分な権限を持つものとして再発行が受け入れられることです。以下の防御方法の項目 7 がこの失敗モードに特化した緩和策です。

**脆弱性のよくある例:**

1. 認証局の階層 (ルート CA、中間 CA、発行 CA) において RSA-2048 または ECDSA P-256 で署名しており、複数年の有効期間を持っているもの。
2. コード署名鍵: OS アップデートへの署名、ファームウェアへの署名、コンテナイメージへの署名 (Sigstore, cosign, Notary)、パッケージへの署名、モバイルアプリへの署名、CI/CD での署名。
3. その他の長期有効なシグネチャトラストアンカー: UEFI セキュアブート鍵、TPM エンドースメント鍵、JWT/SAML 発行者鍵、ドキュメント署名証明書、ブロックチェーンウォレット鍵。
4. 従来の鍵を信頼しており、容易にアップグレードできない、大きなベリファイア集団 (デバイス、サービス、アーティファクト)。
5. 有効性が何年も永続しなければならない長期有効な署名付きアーティファクト (コントラクト、規制当局への提出書類、証拠記録、署名付きソフトウェアリリース、ブロックチェーン取引) において、機密性に関する Mosca の不等式推論を完全性に適用すると、そのアーティファクトがすでにさらされていることを示しているもの。
6. 複数年の有効期間を持つアイデンティティやクレデンシャル (政府発行の身分証明書、専門資格、ルート証明書) において、従来のアルゴリズムで署名されており、適切な移行期間をかなり過ぎても信頼できるものとして維持することが期待されているもの。

**防御方法:**

1. Migrate code-signing and CA hierarchies before broader estate migration - they have the longest blast radius and lead time.
2. Adopt hybrid signing during transition: produce both a classical and a PQC signature on the same artefact so non-PQC verifiers keep working while PQC-aware verifiers gain forward security.
3. Use ML-DSA (FIPS 204) for general digital signatures; use SLH-DSA (FIPS 205) for very long-lived, high-assurance signatures where stateless hash-based security is preferred.
4. Plan for shorter certificate lifetimes during transition (e.g. the CA/Browser Forum 47-day TLS maximum effective 2029) to reduce the exposure window.
5. Engage PKI, code-signing, and certificate-authority vendors on PQC roadmaps - failure here is a supply-chain blocker.
6. For firmware and software signing specifically, CNSA 2.0 lists LMS and XMSS (NIST SP 800-208) as the approved stateful hash-based schemes, and does not approve SLH-DSA for NSS use (SLH-DSA remains valid per FIPS 205 outside that scope - item 3 above still applies generally). This approval covers single-tree LMS and XMSS only - HSS and XMSS^MT, the multi-tree variants, are not approved for NSS use, even though they are the more common library default for high signature counts. ML-DSA is also CNSA 2.0-approved for this use case and may suit setups needing more signatures than a single LMS/XMSS key can produce, or distributed signing. LMS and XMSS are stateful: each key can produce only a fixed number of signatures, and reusing a state index breaks the security guarantee. State management should be handled in hardware such as an HSM, and any flow that transfers or duplicates key material - backup, restore, failover - must guarantee state is never reused.
7. For long-lived signed artefacts already failing the Mosca's-inequality test - contracts, filings, releases, and other fixed-content documents - plan re-signing or counter-signing with PQC schemes (ML-DSA, SLH-DSA) before the classical signature scheme is deprecated, rather than treating the artefact as settled once issued. Re-signing is sufficient here because the artefact's content is unchanged; the new signature only re-attests something that was already true.
8. For long-lived credentials failing the same test, do not treat re-signing as equivalent to remediation. A credential asserts that a named subject controls a key, and a PQC signature over the same claim only re-asserts it with stronger cryptography - it does not re-establish the claim itself. Require re-issuance to rest on evidence independent of the credential being replaced (fresh identity proofing, a still-trusted anchor, or hardware attestation), not on the outgoing key vouching for its own successor.

**攻撃シナリオの例:**

Scenario #1: An attacker with a future CRQC recovers the private key of a code-signing certificate still on ECDSA P-256. They sign malware that passes verification on every device trusting that anchor, distributing a malicious "update" through the legitimate update channel.

Scenario #2: An organisation migrates its leaf TLS certificates to PQC but leaves the root and intermediate CAs on RSA. An attacker forges an intermediate CA signature with a CRQC and issues trusted certificates for arbitrary domains - the chain is only as strong as its weakest classical link.

Scenario #3: A vendor issues software releases signed with ECDSA, with signatures expected to remain valid for the product's decade-long support lifetime. Once a CRQC exists, an attacker recovers the signing key and forges signatures on malicious updates that still validate against the long-lived, un-rotated trust anchor - the artefact was exposed from the day it was signed, under the same Mosca's-inequality logic that governs confidentiality.

**参考情報リンク:**

<!-- References verified 2026-08-05 against authoritative canonical sources. This entry absorbs the integrity content (long-lived signed artefacts, identity and credential records, Scenario #3) formerly in QS02, per the QS01/QS02 merge discussed in #11; QS02's confidentiality content merges into QS01. -->

1. [NIST FIPS 204 (ML-DSA)](https://csrc.nist.gov/pubs/fips/204/final): Module-lattice digital signature standard.
2. [NIST FIPS 205 (SLH-DSA)](https://csrc.nist.gov/pubs/fips/205/final): Stateless hash-based signature standard for high-assurance, long-lived use.
3. [NSA Commercial National Security Algorithm Suite 2.0 (CNSA 2.0)](https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/3148990/nsa-releases-future-quantum-resistant-qr-algorithm-requirements-for-national-se/): Software and firmware signing exclusively CNSA 2.0 by 2030.
4. [IETF LAMPS Working Group](https://datatracker.ietf.org/wg/lamps/about/): PQC X.509 and CMS extensions.
5. [EU Cyber Resilience Act, Annex I](https://eur-lex.europa.eu/eli/reg/2024/2847/oj/eng): State-of-the-art integrity and authenticity requirements.
6. [NIST SP 800-208](https://csrc.nist.gov/pubs/sp/800/208/final): Recommendation for Stateful Hash-Based Signature Schemes (LMS, XMSS).
7. [NSA - The Commercial National Security Algorithm Suite 2.0 and Quantum Computing FAQ, v2.1](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF): PP-24-4014, December 2024. Algorithm-allowance table: LMS/XMSS approved for firmware and software signing; SLH-DSA not approved for NSS use.
8. [NIST IR 8547 (Draft) - Transition to Post-Quantum Cryptography Standards](https://csrc.nist.gov/pubs/ir/8547/ipd): Transition planning guidance referencing Mosca's inequality, applied here to signature and credential lifetime.

**規格や規制のマッピング:**

> **TODO:** This section is carried over from the source document and is not part of `_template.md`. Confirm whether to keep it in the final entry format, and verify each standard/citation.

NIST FIPS 204 (ML-DSA), FIPS 205 (SLH-DSA). NIST IR 8547 (Draft) on transition planning and Mosca's inequality. NSA CNSA 2.0: software and firmware signing exclusively CNSA 2.0 by 2030. NCSC recommends ML-DSA-65 for most use cases. EU CRA Annex I requires state-of-the-art mechanisms for integrity and authenticity, applicable through the product support period for long-lived signed artefacts. IETF LAMPS working group on PQC X.509 and CMS extensions. DORA Articles 28-44 on third-party risk management apply to PKI and signing service vendors.
