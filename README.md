Post-Quantum Secure Banking Communication — Prototype & Benchmark

Overview

This project is a hands-on comparison of a classical key-exchange pipeline (RSA-2048 + AES) against a
post-quantum pipeline (ML-KEM-768 + HKDF + AES-GCM), built to understand the practical migration path
banks will need as NIST and regulators push toward quantum-resistant cryptography. It is a learning/
prototype project, not a production-ready library.

Both pipelines are implemented end-to-end and benchmarked on the same machine so the performance
trade-off is measured directly rather than assumed.


Problem Statement

RSA and ECC, which secure almost all banking communication today, are breakable by a sufficiently large
quantum computer (Shor's algorithm). Data intercepted and stored today can be decrypted later once such
hardware exists ("harvest now, decrypt later"), which is why NIST finalized ML-KEM as a standard and why
financial regulators are beginning to require migration planning. This project simulates that migration
on a small scale: same transaction, two cryptographic pipelines, measured side by side.


What's Implemented

Classical pipeline (baseline)


RSA-2048 keypair generation
RSA-OAEP encryption/decryption of an AES-256 session key
AES-256-CBC encryption/decryption of a simulated banking transaction
30-run benchmark of RSA keygen, encryption, and decryption, exported to CSV and charted


Post-quantum pipeline


ML-KEM-768 key encapsulation (via liboqs-python) — keypair generation, encapsulation, and
decapsulation, verified to produce matching shared secrets on both sides
HKDF-SHA256 derivation of an AES-256 key from the ML-KEM shared secret
AES-256-GCM authenticated encryption/decryption of a test message
Benchmark of ML-KEM encapsulation, decapsulation, and AES-GCM encryption


Results


ML-KEM operations are measurably slower than AES-GCM, as expected, but fast enough in absolute terms
(sub-millisecond to low-millisecond range on a MacBook Pro M2) that the overhead is unlikely to be
the bottleneck in a real banking transaction flow.
Ciphertext and key sizes for ML-KEM-768 are larger than RSA-2048's, which has bandwidth/storage
implications worth considering for high-throughput systems.
Full numeric results are in rsa_benchmark_results.csv and the benchmark charts in the notebook.



What's Not Implemented (yet)

Being upfront about scope, since it's easy for a README to imply more than the code does:


No true hybrid mode. The classical and PQC pipelines are built and benchmarked separately, not
combined into a single hybrid handshake (e.g. ECDH + ML-KEM concatenated into one derived key). That's
the natural next step and is closer to what real hybrid TLS 1.3 deployments do.
No integration with an actual transport protocol (TLS, mTLS, or a banking API). This is a
cryptographic pipeline demonstration, not a network-level implementation.
The classical path uses CBC, not an authenticated mode. This was intentional to mirror a legacy-style
baseline, but it also means the classical and PQC paths aren't cryptographically equivalent — a fair
production comparison would use GCM on both sides.
No formal security proof, side-channel analysis, or key-rotation/lifecycle policy. Those are
substantial areas of their own and are noted as future work, not gaps in this prototype's scope.



Architecture

Classical:      RSA-2048 (key exchange)  →  AES-256-CBC (data encryption)
Post-Quantum:   ML-KEM-768 (key encapsulation) → HKDF-SHA256 (key derivation) → AES-256-GCM (data encryption)


Technologies Used


Python
oqs-python (Open Quantum Safe / liboqs) for ML-KEM
cryptography library for RSA, AES, HKDF
pandas / matplotlib for benchmarking and visualization



Next Steps


Combine classical and PQC key material into a genuine hybrid handshake
Standardize both pipelines on AES-GCM for a like-for-like comparison
Explore integration with TLS 1.3 hybrid key exchange
Add key-rotation and secure key-storage considerations relevant to a banking deployment



Author

Tripti Gupta
Physics Postgraduate | Quantum Computing & Cryptography
