r00t{sp11t_k3y_w13n3r_h4rdw4r3_l3ak}


chaklenge2 Trinity

r00t{fr4nkl1n_r31t3r_r3l4t3d_m3ss4g3s}

challenge3 

r00t{sp11t_k3y_w13n3r_h4rdw4r3_l3ak}

r00t{c0mm0n_gr0und_1s_d4ng3r0us_t3rr1t0ry}

guest: r00t{qpe_g4uss_ph4s3_3ch0_r3c0v3ry_v14_m34n}




1. Challenge: Thin Ice (Wiener's Attack / Multi-Prime RSA)

📝 Challenge Description

FastCrypt implemented a proprietary "Split-Key" RSA architecture using a three-prime modulus (N = p × q × r). The first prime (p) was publicly exposed in the `device.cfg` file as `hw_device_id` for device verification. To guarantee a decryption latency under 10ms to meet strict SLA compliance, the private exponent (d) was tightly constrained to fit within a 120-bit register (d < 2¹²⁰), while the total modulus N was roughly 640 bits.

🔴 Vulnerability Analysis

When the private exponent d in an RSA system is smaller than \(\frac{1}{3} N^{1/4}\), the system is vulnerable to **Wiener's Attack**. Because d was forced to be incredibly small relative to the total size of N, the security bounds broke down entirely. This allows an attacker to recover the private exponent d using **Continued Fractions** directly from the public key parameters (N, e) without needing to factor the remaining session primes.

🛠️ Solution Steps

1. Extracted the leaked batch system prime p from `device.cfg`: `0x910e4130380ead292217fc70bd714929`.
2. Reduced the complexity by dividing N by p to focus on the remaining secret product M = q × r.
3. Developed a Python script to compute the continued fraction expansion of \(\frac{e}{N}\) and generate its convergents.
4. Tested the convergents until the correct private exponent d was uncovered, allowing successful decryption of `encrypted.b64`.

🏆 Flag

text

```
r00t{th1n_1c3_w13n3r_4tt4ck_m0dul_f4ct0r}
```

Use code with caution.

---

2. Challenge: Déjà Vu (Franklin-Reiter Related Message Attack)

📝 Challenge Description

The RouteSecure™ message router transmitted the exact same message to two different regional endpoints. Before encrypting the message for the second regional destination ("EU-WEST"), a known linear transformation was applied to the plaintext in the form of a deterministic `offset` (2⁶⁵ - 49). Crucially, the system utilized a very small public exponent of **e = 3**.

🔴 Vulnerability Analysis

When two ciphertexts encrypt plaintexts that share a known linear relationship (m₂ = m₁ + Δ) under the same RSA modulus N, and the public exponent e is small, the system becomes entirely vulnerable to the **Franklin-Reiter Related Message Attack**. This algebraic vulnerability allows an attacker to compute the original plaintext m by finding the common root of two polynomials without needing the private key or factoring N.

🛠️ Solution Steps

Using the specific algebraic simplification of the Franklin-Reiter attack for e = 3, the plaintext message can be directly computed using the following formula:  
\(m\equiv \frac{\text{offset}\cdot (C_{2}+2C_{1}-\text{offset}^{3})}{C_{2}-C_{1}+2\text{offset}^{3}}\mathinner{\;\left(\mod \,N\right)}\)  
We implemented this relationship in Python, computed the modular inverse of the denominator using the Extended Euclidean Algorithm (Extended GCD), and successfully recovered the plaintext flag.

🏆 Flag

text

```
r00t{fr4nkl1n_r31t3r_r3l4t3d_m3ss4g3s}
```

Use code with caution.

---

3. Challenge: Shared Grounds (RSA Common Modulus Attack)

📝 Challenge Description

KeyCorp managed a multi-tenant document encryption platform where a single classified memo was intercepted. The memo was encrypted and sent to two separate entities: Department A and Department B. The leaked source code (`infrastructure.py`) revealed a fatal architectural flaw: the `KeyManager` reused the exact same RSA modulus N across all departments, only generating distinct exponent pairs (e, d) for each client tenant.

🔴 Vulnerability Analysis

Encrypting the identical plaintext message m multiple times using a shared or **Common Modulus** N with different public exponents (e₁ and e₂) is a textbook cryptographic failure. If the two public exponents are coprime, meaning \(\gcd(e_1, e_2) = 1\), an attacker can use **Bézout's Identity** to compute integers a and b such that a ⋅ e₁ + b ⋅ e₂ = 1.

🛠️ Solution Steps

1. Extracted e₁ and e₂ from the respective `dept_a.pem` and `dept_b.pem` public keys and verified that \(\gcd(e_1, e_2) == 1\).
2. Used the Extended Euclidean Algorithm to calculate the Bezout coefficients a and b.
3. Reconstructed the original plaintext by solving the common modulus equation:  
    \(m\equiv (C_{1}^{a}\cdot C_{2}^{b})\mathinner{\;\left(\mod \,N\right)}\)  
    _(When a coefficient was negative, we computed the modular multiplicative inverse of the respective ciphertext before performing the modular exponentiation)._

🏆 Flag

text

```
r00t{c0mm0n_gr0und_1s_d4ng3r0us_t3rr1t0ry}
```

Use code with caution.