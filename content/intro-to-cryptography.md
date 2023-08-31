---
title: Cryptography and the Solana Network
objectives:
- Understand symmetric and asymmetric cryptography
- Explain keypairs
- Use `@solana/web3.js` to generate a new keypair
- Use `@solana-developers/node-helpers` to load a keypair from an env file
---

# TL;DR

- **Keypair** refers to a matching pair of **public key** and **secret key**. 
- The **public key** is used as an “address” that points to an account on the Solana network. 
- The **secret key** is used to verify identity or authority. As the name suggests, you should always keep secret keys *secret*.
- `@solana/web3.js` provides helper functions for creating a brand new keypair, or for constructing a keypair using an existing secret key. 

# Overview

## Symmatric and Asymmetric Cryptography

Cryptography is literally the art of hiding information. There's two main types of cryptography you'll encounter day to day:

**Symmetric Cryptography** is where the same key is used to encrypt and decrypt. It's hundreds of years old, and has been used by everyone from the ancient Egyptians to Queen Elizabeth I.

There's a variety of symmetric cryptography algorithms, but the most common is AES.

**Asymmetric Cryptography**

- Asymmetric cryptography was developed in the 1970s. In asymmetric crypto, participants have pairs of keys (or **keypairs**). Each keypair consists of a **secret key** and a **public key**. Asymmetric encyrption works different, and can do different things:

- **Encryption**: if it's encrypted with a public key, only the matching secret key can be used to read it
- **Signatures**: if it's encrypted with a secret key, the matching public key can be used to prove the holder of the secret key signed it.
- You can even use asymmetric cryptography to work out a good key to use for symmetric cryptography! This is called **key exchange** where you use your public keys and a recipients public key to come up with a 'session' key. 

Asymmetric encryption is very popular: 
 - You bank card has a secret key inside it, that's used to sign transactions. Your bank can confirm you made the transaction by checking your public key.
 - Websites include a public key in their certificate, your browser will use his public key to encrypt the data it sends. The website has the matching private key, so the website can read the data. 
 - Your electronic passport was signed by the country that issued it, to ensure the passport isn't forged. The electronic passport gates can confirm this usin the public key of your issuing country.
 - The messaging apps on your phone use key exchange to make a session key. 

## Solana uses asymmetric encryption

In Solana, people participating in the network have at least one keypair.

- The public key is used as an “address” that points to an account on the Solana network. 

- The secret key is used to verify identity or authority. As the name suggests, you should always keep secret keys *secret*.

## ⚠️ Treat secret keys carefully

Don't include secret keys in your source code. Put them in a `.env` file and add `.env` to `.gitignore` so they're not committed.

### Making a keypair

In order to do things like send tokens, write data, or send NFTs on Solana, you'll need your own keypair.

To make a new keypair, the `@solana/web3.js` module provides `Keypair.generate()`. 

### Installation

Make a new directory, install TypeScript, Solana web3.js and esrun:

```bash
mkdir generate-keypair
cd generate-keypair
npm init -y
npm install typescript @solana/web3.js @digitak/esrun @solana-developers/node-helpers
```

Make a new file called `generate-keypair.ts`

```ts
import { Keypair } from "@solana/web3.js";
const ownerKeypair = Keypair.generate();
console.log(`✅ Generated keypair!`)
```

Run `npx esrun generate-keypair.ts`. You should see the text:

```
✅ Generated keypair!
```

Each `Keypair` has a `publicKey` and `secretKey` property. Update the file:

```ts
import { Keypair } from "@solana/web3.js";

const ownerKeypair = Keypair.generate()

console.log(`The public key is: `, ownerKeypair.publicKey.toBase58())
console.log(`You shouldn't log secret keys, but we're just testing here. The secret key is: `, base58.encode(ownerKeypair.secretKey))
console.log(`✅ Finished!`)
```

Run `npx esrun generate-keypair.ts`. You should see the text:

```
The public key is:  (some very long string)
You shouldn't log secret keys, but we're just testing here. The secret key is: (some very long string)
✅ Finished!
```

## Using an existing keypair

If you already have a keypair you’d like to use, you can create a `Keypair` from an existing secret key. If you're using node, `@solana-developers/node-helpers` includes some extra functions:

 - To use an `.env` file use `getKeypairFromEnvironment()`
 - To use a Solana CLI file use `getKeypairFromFile()`

To ensure that your secret key stays secure, we recommend injecting the secret key through an environment variable and not committing your `.env` file. 

Make a new file called `.env` with the following contents:

```env
# use this keypair for testing only
SECRET_KEY="some very long string"
```

We can then load the keypair from the environment:

```ts
import { Keypair } from "@solana/web3.js";
import * as dotenv from "dotenv";
import base58 from "bs58";
import { getKeypairFromEnvironment } from "@solana-developers/node-helpers"

dotenv.config();

const ownerKeypair = getKeypairFromEnvironment('SECRET_KEY')

console.log(`The public key is: `, ownerKeypair.publicKey.toBase58())

// Note: we normally wouldn't log secret keys. This is just for demo purposes.
console.log(`The secret key is: `, base58.encode(ownerKeypair.secretKey))
console.log(`✅ Finished!`)

```


