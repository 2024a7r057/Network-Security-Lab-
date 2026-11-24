# Experiment 8: Protected Email Exchange with OpenPGP and Thunderbird

## Objective

To use Thunderbird's OpenPGP support to exchange emails with Harleen in both directions, protecting message content through encryption and verifying sender identities using digital signatures.

## Account and Key Record

| Participant | Address / Shown Name | Key ID |
|---|---|---|
| Aakriti Singh (Me) | `lightsingy123@gmail.com` | `0xC0C83C39934244E2` |
| Harleen | `2024a7r039@mietjammu.in` | `0x12DE1760A3ABF8CA` |

## Message Path at a Glance

```text
Aakriti Singh (Me)                                  Harleen
Aakriti private key                                 Harleen private key
       |                                                    |
       |<------ Public-key exchange / identity check ------>|
       |                                                    |
       | Encrypt to Harleen + sign as Aakriti               |
       |--------------------------------------------------->|
       |                                  Decrypt + validate signature
       |                                                    |
       | Encrypt to Aakriti + sign as Harleen               |
       |<---------------------------------------------------|
       | Decrypt + validate signature                      |
```

## Workflow

1. I set up and selected the OpenPGP key pair associated with my Thunderbird account and shared my public key with Harleen.
2. I imported Harleen's public key into Thunderbird. Both public keys were exchanged for the email communication.
3. For the outgoing email, I addressed the message to Harleen and enabled both encryption and digital signing before sending it.
4. Harleen used her private key to decrypt the received email, and Thunderbird verified the digital signature using my public key.
5. Harleen then sent a reply by encrypting the email for me and signing it with her private key. I opened the received message using my private key and checked the encryption and signature details in Thunderbird.

## Results Recorded from the Screenshots

| Message Direction | Message Opened Successfully | Signature Verification | Associated Key IDs |
|---|---|---|---|
| Aakriti Singh → Harleen | Yes: Encryption/decryption successful | Valid | Aakriti `0xC0C83C39934244E2`; Harleen `0x12DE1760A3ABF8CA` |
| Harleen → Aakriti Singh | Yes: Encryption/decryption successful | Valid | Harleen `0x12DE1760A3ABF8CA`; Aakriti `0xC0C83C39934244E2` |

Both email exchanges were completed successfully. The messages were decrypted successfully, and Thunderbird displayed valid digital signatures in both directions.

## What the Result Means

Encryption ensures confidentiality because only the intended recipient, who has the matching private key, can decrypt the message. Digital signatures help verify message integrity and confirm that the message was signed using the private key corresponding to the sender's public key.

The public-key fingerprint should be verified through a trusted communication channel to establish the identity of the key owner. Short key IDs help identify keys but do not independently prove identity.

## Evidence to Include

- [ ] My sent email security view showing successful encryption and a valid signature.
- [ ] My received reply security view showing successful decryption and a valid signature.
- [ ] My OpenPGP key details: `0xC0C83C39934244E2`.
- [ ] Harleen's OpenPGP key details: `0x12DE1760A3ABF8CA`.
- [ ] Fingerprint verification evidence, if required by the experiment handout.

## Conclusion

I successfully performed a two-way protected email exchange with Harleen using OpenPGP in Thunderbird. I sent an encrypted and digitally signed email to Harleen and received an encrypted and digitally signed reply from her. Both messages were decrypted successfully, and their digital signatures were verified as valid. This experiment demonstrated how OpenPGP provides confidentiality, message integrity and sender authentication through public-key encryption and digital signatures.
