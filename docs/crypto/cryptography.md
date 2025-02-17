# Cryptography

## Overview

LibrePass employs a robust cryptographic framework to ensure the security and privacy of user data.
This documentation outlines the key cryptographic mechanisms used in the project,
including password hashing, key pair generation, data encryption, and decryption.

![Cryptography diagram](../diagrams/cryptography.drawio.svg)

## Encryption Workflow

### **Password Derivation using Argon2id**

LibrePass uses argon2id, a memory-hard and secure password hashing function,
to derive a cryptographic key from the user's password.
This process involves multiple iterations, making it computationally expensive and resistant to brute-force attacks.

### **Key Exchange using X25519**

#### **Private Key**

The X25519 private key is the hash derived from the user's password using argon2id algorithm.
This key is not sent to the server, it is securely stored locally.

```
userPrivateKey = passwordHash
```

#### **Public Key**

The public key is derived from the user's private key using X25519 algorithm.
This key is sent to the server for user authentication.

```
X25519RecoveryPublic(userPrivateKey) -> userPublicKey
```

#### **Encryption Key**

The encryption is derived from the user's private and public key using X25519 algorithm.
This key is not sent to the server and is used to encrypt the vault.

```
X25519ComputeSharedSecret(userPrivateKey, userPublicKey) -> userEncryptionKey
```

#### **User Authentication**

A shared key with the server is generated to authenticate the user,
it is generated using the user's private key and the server's public key.

```
X25519ComputeSharedSecret(userPrivateKey, serverPublicKey) -> authenticationSharedKey
```

Then, the server verifies this key by doing the same thing, but uses its private key, and the user's private key.

```
X25519ComputeSharedSecret(serverPrivateKey, userPublicKey) -> authenticationSharedKey
```

If the key provided by the user and the key generated by the server are the same, the authorization has succeeded.

### **Vault Encryption**

The ciphers are encrypted using the 256-bit symmetric encryption algorithm named AES in GCM mode
using the user's encryption key. Then the encrypted data is sent to the server and stored in the database.

```
AesGcmEncrypt(userEncryptionKey, cipherData) -> encryptedCipherData
```
