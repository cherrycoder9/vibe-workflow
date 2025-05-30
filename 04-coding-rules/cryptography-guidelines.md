# 🔐 암호화 가이드라인: 안전하고 효율적인 암호화 구현을 위한 약속

**목표:** 이 문서는 우리 프로젝트에서 암호화 관련 기술을 선택하고 구현할 때, **일관되고 검증된 안전한 접근 방식**을 따르도록 돕기 위한 핵심 지침입니다. 모든 개발자 및 LLM은 이 가이드라인을 숙지하고 암호화 관련 코드 작성 시 반드시 준수해야 합니다.

*(주요 참고 자료: OWASP Password Storage Cheat Sheet, libsodium 문서, NIST 진행 중인 표준화 문서 등 최신 보안 권고 사항)*

---

## 0. 암호화 설계의 핵심 원칙

암호화는 복잡하게 겹겹이 쌓는다고 강해지는 것이 아닙니다. 다음 원칙들을 항상 염두에 두세요.

1. **단순함이 강력함이다 (Simple ≠ Weak):** 복잡한 자체 구성 대신, **검증된 표준 알고리즘 하나**를 선택하고 올바르게 사용하는 데 집중하세요.
2. **메모리-하드(Memory-Hard) KDF가 답이다:** 단순 반복 해싱(예: `sha256(sha256(...))`)은 GPU/ASIC 공격에 취약합니다. `Argon2id`와 같이 메모리 사용량이 높은 KDF를 사용하세요.
3. **운용 난이도 vs. 보안 이득:** 관리 및 운영의 복잡성이 실제 얻는 보안 이득보다 크다면, 더 단순하고 안전한 다른 방식을 고려하세요. (예: 복잡한 키 관리 체계 대신 HSM/Secrets Manager 활용)
4. **Salt는 공개, Pepper는 비밀:** Salt는 사용자별로 고유하게 생성되는 공개된 랜덤 값입니다. Pepper는 서버 측에만 안전하게 보관되는 비밀 값으로, Salt와 함께 사용하여 해시 강도를 높입니다. (유출 시 추가 방어선)
5. **Nonce/키 재사용은 절대 금지:** AEAD(Authenticated Encryption with Associated Data) 암호화 방식 사용 시, 동일한 (키, Nonce) 쌍을 **절대로** 재사용해서는 안 됩니다. 이는 암호 해독의 직접적인 원인이 됩니다.

---

## 1. 패스워드 해싱 / 키 스트레칭 (Password Hashing / Key Stretching)

사용자의 패스워드를 안전하게 저장하기 위한 KDF(Key Derivation Function) 선택 및 설정입니다.

| 항목                                                                                                                                                                                                               | 권장 값 / 지침                                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **주요 KDF 알고리즘**                                                                                                                                                                                                  | **`Argon2id`** (libsodium의 `crypto_pwhash` 함수 사용 권장)                                                                                                   |
| **`Argon2id` 파라미터**                                                                                                                                                                                              | 최소: `iterations (t) = 2`, `memory (m) = 19 MiB (2^14.2 KiB)`, `parallelism (p) = 1` (OWASP 권고)                                                         |
| **서버 성능 허용 시 권장:** `iterations (t) = 3` 이상, `memory (m) = 1 GiB (2^20 KiB)` 근처까지 상향 조정. ([OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)) |                                                                                                                                                        |
| **Salt**                                                                                                                                                                                                         | 사용자마다 **최소 16 바이트 이상**의 암호학적으로 안전한 랜덤 값(CSPRNG)으로 생성. 해시된 패스워드와 함께 DB에 저장 (예: `salt$argon2id$v=19$m=...,t=...,p=...$hashed_password`).                 |
| **Pepper (선택 사항)**                                                                                                                                                                                               | **최소 32 바이트 이상**의 암호학적으로 안전한 랜덤 값. 애플리케이션 전체에 대해 하나의 값을 사용하거나, 주기적으로 교체. 환경 변수, HSM, 클라우드 Secrets Manager 등을 통해 안전하게 보관. **절대 DB에 Salt와 함께 저장하지 말 것.** |
| **해싱 프로세스 (Pepper 사용 시)**                                                                                                                                                                                        | 1. `derived_key = Argon2id(password, salt, params)`                                                                                                    |

2. `final_hash = HMAC-SHA256(derived_key, pepper)` (HMAC을 사용한 추가 강화)
   *Pepper를 Argon2id 입력의 일부로 포함하는 방식도 있으나, HMAC 방식이 더 유연할 수 있음.*                                                   |
   \| **패스워드 비교 (검증)**  | **반드시 Constant-Time 비교 함수** 사용 (예: libsodium의 `crypto_verify_16`, `crypto_verify_32`, `crypto_verify_64` 또는 언어별 안전 비교 함수). 단순 문자열 비교(`==`)는 타이밍 공격에 취약.                                                   |

*차선책 (레거시 시스템 호환 목적 외 신규 도입 금지):* `scrypt` (파라미터 예: `N=2^17, r=8, p=1`). `PBKDF2-HMAC-SHA256`는 더 이상 권장되지 않음.

---

## 2. 대칭키 데이터 암호화 (Symmetric Data Encryption - AEAD 방식 필수)

저장 데이터(Data at Rest) 또는 전송 중 데이터(Data in Transit, TLS 외 추가 암호화 필요 시)를 위한 암호화입니다. **반드시 AEAD(Authenticated Encryption with Associated Data) 모드를 사용**하여 기밀성과 무결성/인증을 함께 보장해야 합니다.

### 2.1. 패스워드 기반 암호화 (Password-Based Encryption - PBE)

```text
# 1. 키 파생 (Password-Based Key Derivation Function)
encryption_key = Argon2id(password, salt_for_encryption, argon2_params, pepper_optional)  # 예: 256-bit (32 바이트)

# 2. 암호화 (AEAD 사용)
nonce = generate_random_nonce() # 예: 24 바이트 for XChaCha20
ciphertext, authentication_tag = XChaCha20-Poly1305_encrypt(encryption_key, nonce, plaintext_data, associated_data_optional)

# 3. 저장 형식 (예시)
stored_data_format = "xchacha20poly1305:argon2id_params:" + base64(salt) + ":" + base64(nonce) + ":" + base64(ciphertext) + ":" + base64(tag)
```

### 2.2. 마스터 키 기반 암호화 (Master Key Based Encryption)

```text
# 1. 데이터 암호화 키 파생 (HKDF 사용)
DEK = HKDF-SHA256(master_key, salt_optional, info_string, output_length=32)

# 2. 암호화 (AEAD 사용)
nonce = generate_random_nonce()
ciphertext, tag = XChaCha20-Poly1305_encrypt(DEK, nonce, plaintext, aad_optional)

# 3. 저장 형식 (예시)
stored_data_format = "xchacha20poly1305:hkdf_salt_if_any:" + base64(nonce) + ":" + base64(ciphertext) + ":" + base64(tag)
```

### 2.3. 권장 AEAD 알고리즘

| 상황 / 우선순위   | 1순위 (소프트웨어)            | 하드웨어 가속 시 고려 (Nonce 재사용 저항) |
| ----------- | ---------------------- | --------------------------- |
| 모든 플랫폼 (권장) | **XChaCha20-Poly1305** | AES-256-GCM-SIV             |

> **금지 (CRITICAL):**
>
> * AES-GCM에서 Nonce 재사용
> * 자체 암호화 모드 조합 ("Roll your own crypto")

---

## 3. 키 파생 및 도메인 분리 (Key Derivation and Domain Separation)

* **HKDF 사용 권장**: HKDF(root\_key, salt, info, length)
* `info`에 키 용도를 명확하게 표시 (예: "user\_data\_v1").

---

## 4. 암호화 구현 체크리스트 ✅

* [ ] Salt: 16바이트 이상, CSPRNG, 고유 저장
* [ ] Pepper: 32바이트 이상, 안전 보관
* [ ] KDF 파라미터: OWASP 권고 이상
* [ ] Nonce: (키,Nonce) 재사용 금지
* [ ] 인증 태그 검증 필수
* [ ] Constant-Time 비교 사용
* [ ] 검증된 라이브러리만 사용

---

## 5. 흔한 안티패턴 💣

| ❌ 안티패턴                     | 이유                        |
| -------------------------- | ------------------------- |
| KDF 중첩 (Argon2id 후 scrypt) | 보안 향상 미미, 서버 부하만 증가       |
| 이중 AEAD 암호화                | 검증되지 않은 방식, 성능 저하         |
| PBKDF2 또는 bcrypt 신규 도입     | GPU 공격 취약, 메모리 하드니스 부족    |
| Pepper를 DB에 저장             | Pepper 의미 상실, DB 유출 시 무력화 |
| 고정 Nonce 사용                | 치명적 취약점, AEAD 보안 무너짐      |
| 자체 암호화 알고리즘 사용             | 검증되지 않음, 취약점 유발           |

---

## 6. 암호화 슬로건 🚀

> "Keep It Simple, Secure, and Standard."

---

**LLM 코딩 가이드:** 암호화 코드 작성 시 이 가이드를 최우선으로 참고하고, 금지 항목을 절대 생성하지 마세요. 의심되면 더 안전한 옵션을 제안하고, 근거를 설명해주세요.
