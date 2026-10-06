# Area 4: Cryptography

Use this checklist for every changed call into a cryptographic library, every new or changed key, token, nonce, salt, or password hash, and every change to certificate or TLS configuration.

Method note: an algorithm name alone is not a finding. Trace each primitive to its use site, and report only when the use is security-relevant and the patch introduces or keeps the weak choice. This follows from the reportable-finding filter in SKILL.md.

Tags: `[S]` sourced. `[O]` originated; anchor given or "no anchor".

## Algorithms and modes

**CR-1 [S] Approved algorithms only.** Hash functions used for signatures, HMAC, key derivation, or random bit generation are on the approved list. A disallowed hash, such as MD5, is not used for any cryptographic purpose, including integrity tags and file identifiers that an attacker can influence.
- *Look for:* `md5`, `sha1` in signature, HMAC, token, or key-derivation code; a "checksum" function used to authenticate data.
- *Source:* ASVS 5.0 11.4.1.

**CR-2 [S] Insecure block modes and padding.** ECB mode and weak padding schemes (such as PKCS#1 v1.5 for encryption) are not used.
- *Look for:* `AES.MODE_ECB`, `"AES/ECB"`, `Cipher.getInstance("RSA/ECB/PKCS1Padding")`, a padding oracle-prone decryption path.
- *Source:* ASVS 5.0 11.3.1.

**CR-3 [S] Approved ciphers and authenticated modes.** Symmetric encryption uses an approved cipher in an authenticated mode, such as AES-GCM. Unauthenticated modes are a finding when the ciphertext is used without a separate integrity check.
- *Look for:* AES in CBC or CTR with no MAC; a changed mode from GCM to CBC.
- *Source:* ASVS 5.0 11.3.2; OWASP-CRYPTO, Cipher Modes section (authenticated modes where available).

**CR-4 [S] Minimum strength.** Primitives and key sizes provide at least 128 bits of security. Symmetric keys are at least 128 bits, and ideally 256 bits where the project supports it. Asymmetric key sizes meet the strength of the chosen algorithm.
- *Look for:* RSA keys below 2048 bits; AES with a 64-bit key; a changed key-size constant.
- *Source:* ASVS 5.0 11.2.3; OWASP-SCR, Cryptography checklist, "strong algorithms" (AES-256, RSA-2048+, ECDSA P-256+ as the modern set); OWASP-CRYPTO, Algorithms section (symmetric key size).

**CR-5 [S] IV and nonce handling.** Initialisation vectors and nonces follow the rules of the selected mode. Where the mode needs uniqueness, the code guarantees it per key. Where the mode needs unpredictability, the code draws from a CSPRNG.
- *Look for:* a fixed IV or nonce; a counter reset on restart; a nonce derived from the message or a timestamp for GCM.
- *Source:* OWASP-SCR, Cryptography checklist, "IV/nonce handling" (per-mode rules).

**CR-6 [S] Password storage.** Passwords are stored with an approved, computationally intensive password-hashing function, with parameters set to current guidance. A plain or fast hash (a single SHA digest, or an unsalted hash) is a finding.
- *Look for:* `hashlib.sha256(password)` as the stored value; `bcrypt` or `argon2` with a low cost factor or fixed salt; a changed cost parameter that lowers the work factor.
- *Source:* ASVS 5.0 11.4.2 (L2); OWASP-AUTHN, "store passwords in a secure fashion".

## Randomness and tokens

**CR-7 [S] CSPRNG for security values.** Every random value that must be unguessable (tokens, session IDs, reset codes, keys, nonces, salts, OTP seeds) comes from a CSPRNG and has at least 128 bits of entropy. UUIDs are not a substitute.
- *Look for:* `random.random()`, `Math.random()`, `rand()`, a general-purpose PRNG seeded with time; `uuid4()` used as a secret; a token that is a truncated hash of a counter.
- *Source:* ASVS 5.0 11.5.1 (L2; the requirement text notes that UUIDs do not meet the condition); ASVS 5.0 7.2.3 for reference session tokens.

**CR-8 [O] RNG quality in code.** A changed generator is checked for the source it draws from, not just its name. A wrapper that calls a CSPRNG and then reduces the output in a biased or truncated way is a finding when the bias or truncation leaves usable entropy below 128 bits.
- *Look for:* modulo reduction of random bytes into a small alphabet, a short token length chosen to fit a UI, a random seed drawn from a timestamp or process ID.
- *Source:* anchored to ASVS 5.0 11.5.1. The bias and truncation checks are originated.

## Comparison and transport

**CR-9 [S] Constant-time comparison.** Comparisons of MACs, tokens, password hashes, and API keys use a constant-time function, not `==`.
- *Look for:* `hmac_received == hmac_computed`; `token == stored_token` on a secret.
- *Source:* OWASP-AUTHN, section on comparing password hashes with safe functions (constant-time comparison, safe input length, explicit types). No ASVS anchor verified in this survey.

**CR-10 [S] Certificate and TLS validation.** Certificate validation includes chain and hostname verification. Turning off verification, or accepting any certificate, in production code is a finding.
- *Look for:* `verify=False`, `InsecureSkipVerify: true`, `rejectUnauthorized: false`, a custom trust manager that accepts all certificates, a hostname check removed.
- *Source:* OWASP-SCR, Cryptography checklist (certificate validation item, including hostname verification). No ASVS anchor verified in this survey (ASVS chapter 12 was not read).

**CR-11 [S] Library currency.** A changed cryptographic library or a new one is checked for the project's documented minimum version. A downgrade of a crypto library is a finding.
- *Look for:* a pinned library moved to an older version; a new cryptographic dependency in a non-standard package.
- *Source:* OWASP-SCR, Cryptography checklist (library maintenance item). The dependency status itself is checked under supply-chain.md.

## Key lifecycle

**CR-12 [O] Key lifecycle in code.** A changed key path supports the lifecycle the project documents. Check each of the following that the diff touches:
- a key is generated by an approved method, not hand-written or derived from a guessable input;
- the key carries an identifier or version, so that rotation can happen without breaking old data;
- a key is used for one purpose only, and a signing key is not used for encryption, or the reverse;
- the key is not written to source, logs, or error messages (see secrets.md);
- old keys can be retired, and the code has a defined path to do so.

- *Look for:* one key constant used for both HMAC and AES; encrypted data with no key ID, so rotation would make it unreadable; a key derived from a password without a KDF.
- *Source:* anchored to ASVS 5.0 11.1.1 (documented key lifecycle following a key-management standard) and 11.1.2 (cryptographic inventory). The code-level checks are originated.
