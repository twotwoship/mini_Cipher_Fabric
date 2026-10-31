# Key protection standards: reading guide

Checked: 2026-10-31 (Asia/Seoul). Original publisher PDFs were downloaded without modification. This is a reading guide, not a conformance or certification finding.

## International standards and edition boundaries

- ISO/IEC 19790:2025: Security requirements for cryptographic modules. Official catalog: https://www.iso.org/standard/82423.html
- ISO/IEC 24759:2025: Test requirements for cryptographic modules. Official catalog: https://www.iso.org/standard/82424.html
- Full ISO texts were not downloaded: no openly downloadable full text was obtained from the official publisher. Obtain licensed copies through ISO or an authorized national standards body.
- FIPS 140-3 is a US federal standard, not an ISO International Standard. Its published requirements reference ISO/IEC 19790:2012 and ISO/IEC 24759:2017. The downloaded 2026 implementation guidance still explicitly discusses these editions. Do not attribute its quoted clause numbers to ISO/IEC 19790:2025 without checking that edition.

## Downloaded sources and relevant clauses

1. FIPS-140-3.pdf (11 PDF pages)
   Source: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.140-3.pdf
   Purpose: federal cryptographic-module standard and its relationship to the ISO requirements. This short document is not a free replacement for the complete ISO standards.

2. FIPS-140-3-IG-2026-08-19.pdf (249 PDF pages; last update 2026-08-19)
   Source: https://csrc.nist.gov/csrc/media/Projects/cryptographic-module-validation-program/documents/fips%20140-3/FIPS%20140-3%20IG.pdf
   Read IG 9.5.A, especially PDF pages 86 and 91-92: plaintext entry/output is not universally prohibited. Applicable level, establishment method, interface, and operating environment matter. For the cited Level 3 manual entry/output rules, plaintext secret/private keys require split knowledge and a trusted channel; a normal bus is not automatically a trusted channel.
   Read IG 9.6.A, PDF pages 93-94: internal SSP storage may be plaintext or encrypted. Claiming cryptographic protection requires approved protection mechanisms; unprotected parameters remain subject to zeroization requirements.
   Interpretation: plaintext MEK in an explicitly protected module is different from exposing MEK to an untrusted host. Neither the PS/PL boundary nor the user/kernel boundary automatically defines the validated cryptographic boundary.

3. NIST-SP-800-38F-Key-Wrapping.pdf (32 PDF pages)
   Source: https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-38F.pdf
   Read section 5.2, printed page 9 / PDF page 17: authenticated unwrapping returns the recovered plaintext or FAIL. Wrapping protects a key representation; it does not mean AES-XTS directly operates on the wrapped representation.
   Read Appendix A.2, printed page 19 / PDF page 27: disclosure of the KEK can compromise protected keys. KEK protection is therefore essential.
   This is a NIST recommendation, not an ISO International Standard.

4. TCG-Opal-SSC-v2.30.pdf (101 PDF pages; published 2025-01-30)
   Source: https://medias.cdn.trustedcomputinggroup.org/production/TCG_Storage_Opal_SSC_v2_30_pub_f4e29a2c4b.pdf
   Read section 2.4, printed page 16 / PDF page 17: full-disk encryption of host-accessible user data on media and AES support.
   Read section 4.3.5.5, printed page 88 / PDF page 89: AES key tables and indirect key modification through GenKey.
   This is a storage industry specification, not ISO/IEC 19790 and not automatically applicable to a generic Linux FPGA accelerator. Do not attribute the OCP requirement below verbatim to this document.

5. OCP-Datacenter-SAS-SATA-v1.5.pdf (139 PDF pages; version 1.5, 2025-06-27)
   Source: https://www.opencompute.org/documents/datacenter-sas-sata-device-specification-1-5-2025-06-27-pdf
   Read section 13.4.7, requirement TCG-LOCKING-4, printed/PDF page 132.
   Meaning: the K_AES_256 table's key values must not be readable by the host or accessible outside the device. This directly matches a host-inaccessible media-key requirement.
   This is an OCP datacenter SAS/SATA device industry specification. It is not an ISO International Standard or a universal requirement for all AES-XTS software.

## Application to mini_Cipher_Fabric (engineering interpretation)

The current README proposes user-space MEK unwrapping followed by a normal dm-crypt/Crypto API/FPGA key-loading path. That does not meet a project requirement that Linux/PS must never obtain plaintext MEK. It is not, merely because it uses plaintext runtime keys, proof of violating all international cryptographic-module standards.

For a host-inaccessible-MEK design, explicitly define Linux/PS as outside the protected key boundary. Generate or unwrap keys inside that boundary; prevent host key readback and key-bearing debug output; protect the wrapping/root key; and expose authorized operations or key references instead of raw MEK. Also evaluate configuration/debug access and key erasure. These are design implications, not a claim that a standard mandates one exact FPGA architecture. Copying this pattern does not establish Opal, OCP, ISO, or FIPS certification.

The README itself was not changed during this standards investigation.
