# Total Encryption Standard

This document describes the specifications for the **Total Encryption Standard**, used to encrypt data on **Totenpass**.

The latest version of this document can be found on the following GitHub repository: [https://github.com/totenpass/tes-specs](https://github.com/totenpass/tes-specs)

Notice: even if the algorithms used in this standard are the best in class in their categories, remember that the security of your data depends also on the security of the key, so always use strong passphrases to encrypt your data!

The decryption is a multi-steps process: first the ciphertext is extracted and decoded from the QRCode, then the ciphertext is decrypted into a binary buffer, and finally the original plaintext is recovered.
Both the ciphertext and plaintext encodings include the information about the specification version, to allow future extensions.

# Decryption

This section is a step-by-step guide to decrypt data stored on a Totenpass with the Total Encryption Standard.

## 1. QRCode content

The content of the QRCode can be read by any QRCode reader application, it's been designed to maximize the interoperability.
The content is a URL pointing to an on-line decoder, with the **encrypted** data following the fragment identifier (#) at the end of the URL. In this way, the encrypted data is never sent to remote servers, not even accidentally.
The ciphertext, input of the next step, is the Base64 (URL safe variant) decoding of the encrypted URL fragment.

## 2. Ciphertext decryption

The first byte of the ciphertext identifies the ciphertext encoding version. The only allowed version is currently 0 (0x00 in hex). This version identifies the use of **Argon2id** as key-derivation algorithm and **XChaCha20-Poly1305** as encryption algorithm with integrity check.

The next 17 bytes contains the parameters to be used with Argon2id:
- 1 byte for the number of *iterations* and the *memory*: the first 3 bits are the iterations (e.g. 011b for 3), while the last 5 bits are the memory divided by 64 MB (eg. 00100b for 4 * 64 * 1024 * 1024 = 268,435,456 bytes)
- 16 bytes for the *salt*

The next 24 bytes are the *nonce* to be used with XChaCha20-Poly1305.

The derived key and the nonce can then be used to decrypt and validate the remaining bytes in the buffer with XChaCha20-Poly1305.


## 3. Plaintext retrieval

The first byte of the decrypted data identifies the plaintext encoding version. The only allowed version is currently 0 (0x00 in hex). This version allows to encode text (UTF-8 string) and (binary) files.

- if the second byte is 0x00, then the remaining bytes contain the UTF-8 encoding of the plaintext 
- if the second byte is 0x01, then the remaining bytes contain a UTF-8 NULL-terminated string of the filename, followed by the binary file content
- other values for the second byte may be added in the future, for example to support compressed plaintext

# Reference implementations

We aim to provide reference implementations of the decrypt functionality, as library and CLI, in different programming languages.

Currently we have already published the following implementations:
- [JavaScript](https://github.com/totenpass/tes-js) (for both Node.js and browsers)
- [Java](https://github.com/totenpass/tes-java) (usable in Android, too, and with any other JVM language like Kotlin, Scala and Groovy)

# Test vectors

We provide a test vector for each scenario described above.
The passphrase to decrypt them is always `My Secret Passphrase!`

## Text

Ciphertext:

    AIIo5iih-FcSXacIUdKRoOXVXvgWocImR26ReogzrdJjufnNzTD9V3ea9XSCpX_v-1VrM6M_nRZBv8GqLrFS3IIEHWJ6Una0MfKh6ibfxgVYmYHOBe-lVEUX_CGt6UhbQyoYBdgiDEIggsGcZTl7hyvp3viPoZrzFbaeZElD

Plaintext:

    Totenpass is a permanent digital storage drive made of solid gold.

## File

Ciphertext:

    AIIyx9vZQTJ-AxIl8mGDmjRWnenYvoMkOLhBzrWklASDGg8HsqUEeDx4TGxi0vm_5RW3QCUuXyFvGvxG6R0clY4N7pXLU_SaITRlpaRTQf09ZjIcl7UYofc82JMCzrLNSuayk34uQ9AMotOtC6b4HA8Oat0w0puycvZJrrdfGCuKNB819MsnPE0J_R5UVHatKIpjKY5-Lea5UGzJYMygKBqFcFOvQyicWvM-Fncb3Ohs7eDEXGGaGfkr7kRBWcJcgsuwVWmxfMiy-feX5RK4cXQJr4NDcPmRqqwibbFT4Rt6Tq1SDx_d-4GZfSc2suCEDpmkS-eX6L7741VaIvjTqxn2pZszOqnlAKwmT4i5m2QVFeB8daX5DtSpbIl9Ph2cwo9rkRoKJ8auCzS59UxDF0R1-kACTwX7XwtxZ-Poy22B6x5dfllv66ndwcUhrJu1omGXwJeT_L_IwXJhgR92oxW3rOk9ny73q0OWjEl0naiKQV0lnbty7aac6S86ypp_V7lC444ACzxr1ioB46pGcMiosBkOTa6KLG9xOQi5BGjJQHrBJ82Qy473a9Vtc_tvLRKDnJottTvWrExQfPIqs9syaCn0nHpcI1G2QbJVu65WjvpBXgpZt_QpUkbAR_JdY7-wbpcaKIT-v7vbXH3lH4zZzQOkMcQOW5ylKNocBQ-AFwrS2bBAn8htebaA1z1AkTnChD1x2VUldNVUw5VPyhWg_X8aI3O7fR_0zmx4-3RUO8tmoZrJEcevjeMMyITobsQp7bP8LFIibyT63DKnJf4rx9QXqkSHQtMvwPigDSpFLjglZFJB_V_7qzdidECCZvAYhrLPO3wRj36UzAQ7bUd8zxitvMIgsG2RhOTZb3Uqg1og5Fk0vrar1Kri9gGnzcv8UIVyfnuQ1VYR-IIxPazb6Hnq-_n8RuW5lV99zeX9Smuzm-JvzZhn0tjdgWqrRh-W7obKt4ABDZiEL5rUyCTLCpIoqw5iP9r5vdy-Axq_F5At-SzA_0HiqC-VX8ZJ3HxHfSVzaJwex5ebBwzl2Db7J5kJdMoH9avfO0yOnt03NsUIg2D0r8ghYT12w83c9rGx0UgGkMm2vBtfQh_OtknfITggP9ScjRCD6ehMcgd1Yf2DR-RK7YrsQ1eHoStXQfcmkxB2A1jRs_5XbDy0PmWLHtPcBHQalYbbQu-mf1oQ-QbFxlLptOD5HRPjy6wx4M8y4xdQefl181tbMja022KUNTWc48zASJ8IHY0l_Y0FjQZ2jk2BrC22udKCdnR7rQUSIWDla3HSbkj5jH34IJwk6hT7wqezbSoRPB63VHTs4PJsdiaNBWdjj8FGurYYEyHh5ebCQ3P2drJsFgb4AEkknqtF7dD_dvxih_DUsBI88M5rveS0h2ODLruabngGsoBMwqhI4KTa-NqoedAq4gQTtL5vGOvj1o6d_pEntKdccqLTQbVEXQll0dM3vWpVCmTP-fJuS6G2P35dyWQnn3HlcOoB98EXcCJo5nN95fbQy5F-rkFWXJ4h92wAclchqQJrzNbGmaUtMme4ZtV64Jswj-kGomWJF9N9QGbc6vvtd1nfADdgAX1uDzRTndTXWJ6Tp0zRzW2whaPe6qmNVP7cjxNlgntAlE6rv9qSfS2xBcBnxHdJlYzdUNNDqVDpmv7IwFR4GPvx5-AdhsgR2L0LRL-_37VFzz_tXy66PQHcYJ5r91tdp4W5D0OxlFZWX_XjxSzZbSmYMeasA9gjV-AVRtSfO0I_QLzZ9LK8bMZ4dhmCgM0lEHBmXSzreL_mhX34t5GKKJfa873zPYy-8_uRqzo3pPA3ya1-TmORfYdh287A3jcekkRAycu6a0VYeyflIQbyvsja5H1fn7v7qwNzX5m0vJ0yJm0O9p9ifIivjsz2xxgn3NuGjJhSJllkxR5f5XUXjBbVlqqXlGMq5R9BbYz1nCoqizlV9BqVmTewKP3Teo3t4H7BhyL7oB3oHC5eT7LChM8psG3csI4z-1a0rhHCCk20RDCs76arvMzKSowt4dyLeVz2Unz0f6AmxfqdjXrI8UIXdM3SvIfhG11hj4_5hqA6u73cEgvy5xwoZsXvBbuq0lWPG6TzyEQQ9eUN3Pcs5-Bs4G0rk8RiLAb6pv_ffDPoGG-bDJOQL6JdcrW3OzssC3dS4sm6oJ--PbOUpEQQO_-uf01gTve7DpL9nNiRyZyb_JEDds8z7a58mSofXXhK8uZwHVBstDMiQ2yWKq_379z3qcXzjLMLtAPIRuLFUarUEEEgAkdxJaFbBi2OgvyhXMsYuIG9oTavOwxf2N9rjV8LzBFQeyvuzmcusT2jb-H84zulhS5xLAUBUev9lQvtui9WFgSBPjujSV9CwTIoNkrRLUX_WupAc2DumVn-9yuAIv3ifLy7r718btX0SFJoHrSmtc4wYXSEYpI8Jah8aP-lMG5Qdx1kPjNj-Pp3fpYsnlFJFqPPU2oVJS83hmGvnFnz07pXbpgXPykr6dmvVuvbFx_8qro-uLHaQauNaVFfIzcwMdJgrGGCO2oyhfagKiEgXf6So0kMwi6Wh8DfVztCjNwYpQ-X0xafcpg

Filename: `Totenpass Logo.png`

Content's hash (SHA-256): `0x0b9e166430d4e2107f5a459703b9a9d380bd2b126835693a2317fb603788ec5f`