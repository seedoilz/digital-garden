---
aliases: 
title: Information Securtiy Input
date created: 2024-09-03 19:09:00
date modified: 2024-09-04 18:09:99
tags:
  - input
---

## Attack
### Passive Attack
![CleanShot 2024-09-03 at 19.31.17.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-09-03%20at%2019.31.17.png)

### Active Attack
![CleanShot 2024-09-03 at 19.31.43.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-09-03%20at%2019.31.43.png)

## Threat Model
>defines the scope and assumptions of security

### Security Properties(CIA)
![CleanShot 2024-09-03 at 19.34.31.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-09-03%20at%2019.34.31.png)
#### Confidentiality (privacy)
Information is not exposed to unauthorized parties

#### Integrity
Information is not modified by unauthorized parties or modifications can be detected

#### Availability
Information can be accessed by authorized parties at proper time

## Different Kinds of cryptanalysts
### Ciphertext-only
the attacker has access only to a collection of ciphertexts or code texts.
> [!NOTE]  Security
> The weakest security model for a cryptosystem where attacker has the least power.

### Known-plaintext
the attacker has a set of ciphertexts to which he knows the corresponding plaintext (attacker can do fixed try)

### Chosen-Plaintext
the attacker can obtain the ciphertexts (plaintexts) corresponding to an arbitrary set of plaintexts (ciphertexts) of his own choosing. (CPA and CCA security)
> [!NOTE] CCA security
> the strongest security model where attacker has the most power. Even under this, the algorithm is secure.

## Different Cipher
### Basic (Symmetric) Cipher Scenario
#### Adversary
Very clever with extremely powerful computers, can run brute-force attack.
![CleanShot 2024-09-03 at 20.06.02.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-09-03%20at%2020.06.02.png)

### Stream Cipher
#### One Time Pad
>Not Practical, But Very Secure

##### Functions
Use XOR（异或）to encrypt the message.

> [!NOTE] Why not Secure When Using the Key Second Time?
> People can try 知道两个足够长的明文的异或值，非常可能执行频率分析（frequency analysis， 攻破[古典密码](https://zhida.zhihu.com/search?q=%E5%8F%A4%E5%85%B8%E5%AF%86%E7%A0%81&zhida_source=entity&is_preview=1)常用的）并且恢复明文本身。

### Block Cipher
![CleanShot 2024-09-03 at 20.30.00.png](https://typora-tes.oss-cn-shanghai.aliyuncs.com/picgo/CleanShot%202024-09-03%20at%2020.30.00.png)

### Stream Cipher v.s. Block Cipher
#### Stream Cipher:
- Processing message bit by bit (or byte by byte);
- Typically has a Pseudorandom Generator (PRG) to generate a key stream;
- Use XOR as the basic operation to combine key and plaintext (or ciphertext);
- The whole security is built on the security of PRG; 
- Normally would be simpler and faster.
##### Block Cipher:
- Processing message block by block (with fixed block size);
- Each block will go through multiple rounds of Permutations and Substitutions;
- Multiple operations, like rotation and shifting;
- Key dependent substitution (S-Boxes);
- Has dedicated key scheduling process (to generate keys for each round)