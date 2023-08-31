---
title: Read Data From The Solana Network
objectives:
- Understand accounts and their addresses
- Understand SOL and lamports
- Use web3.js to connect to Solana and read an account balance
---

## TL;DR

- **SOL** is the name of Solana’s native token. Each Sol is made from 1 billion **Lamports**. 
- **Accounts** store programs and data. For now we’ll focus on accounts that store SOL. 
- **Addresses** point to accounts on the Solana network. Anyone can read the data in a given address. 
- **`@solana/web3.js`** is an npm module that allows you use Solana in JavaScript.

# Overview

## Accounts

All data stored on Solana is stored in accounts. Accounts can store SOL, and also be used to store custom data and executable programs.

### SOL

SOL is Solana's native token - Sol is used to pay transaction fees, pay rent for accounts, and more. SOL is sometimes shown with the `◎` symbol. Each Sol is made from 1 billion **Lamports**. In the same way that finance apps typically do math in cents (for USD), pence (for GBP), Solana apps typically use Lamports and convert to SOL to display data.  

### Addresses

Addresses uniquely identify accounts. Addresses are 256-bit and they are often shown as base-58 encoded strings like `7C4jsPZpht42Tw6MjXWF56Q5RQUocjBBmciEjDa8HRtp`. Most addresses on Solana are also public keys - whoever controls the matching secret key controls the account - for example, the person with the secret key can send tokens from the account.

We’ll cover a lot of the [web3.js](https://docs.solana.com/developing/clients/javascript-reference) gradually throughout this course, but you can also check out the [official web3.js documentation](https://docs.solana.com/developing/clients/javascript-reference).

## Your first program

### Installation

Make a new directory, install TypeScript, Solana web3.js and esrun:

```bash
mkdir balance-checker
cd balance-checker
npm init -y
npm install typescript @solana/web3.js @digitak/esrun 
```

### Connect to the Network

Every interaction with the Solana network using `@solana/web3.js` is going to happen through a `Connection` object. This object establishes a connection with a specific Solana network, called a 'cluster'. For now we'll use the `Devnet` cluster rather than `Mainnet`. As the name suggests, this cluster is designed for developer use and testing.

Make a new file called `check-balance.ts` with the following contents:

```ts
import { Connection, clusterApiUrl } from "@solana/web3.js";

const connection = new Connection(clusterApiUrl("devnet"));
console.log(`✅ Connected!`)
```

Then run `npx esrun check-balance.ts`. You should see 

```
✅ Connected!
```

Showing the script successfully completed.

### Read from the Network

Let's expand `check-balance.ts` to read the balance of an account:

```ts
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";

const connection = new Connection(clusterApiUrl("devnet"));
const address = new PublicKey('CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN');
const balance = await connection.getBalance(address);

console.log(`The balance of the account at ${address} is ${balance} lamports`); 
console.log(`✅ Finished!`)
```

Then run `npx esrun runme.ts`. 

The balance returned is in lamports. A lamport is the minor unit for Sol, like cents is to US Dollars, or pence is to British pounds. A single lamport represents 0.000000001 SOL. Most of the time when dealing with SOL the system will use lamports instead of SOL. Web3.js provides the constant `LAMPORTS_PER_SOL` for making quick conversions.

```ts
import { Connection, PublicKey, clusterApiUrl, LAMPORTS_PER_SOL } from "@solana/web3.js";

const connection = new Connection(clusterApiUrl("devnet"));
const address = new PublicKey('CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN');
const balance = await connection.getBalance(address);
const balanceInSol = balance / LAMPORTS_PER_SOL;

console.log(`The balance of the account at ${address} is ${balanceInSol} SOL`); 
console.log(`✅ Finished!`)
```

Then run `npx esrun check-balance.ts`. You should now see something like:

```
The balance of the account at CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN is 0.00114144 SOL
✅ Finished!
```

...and just like that, you are reading data from the Solana blockchain! 

## Using web3.js in a web app

Let’s practice what we’ve learned, and create a simple website that lets users check the balance at a particular address.

It’ll look something like this:

![Screenshot of demo solution](../assets/intro-frontend-demo.png)

In the interest of staying on topic, we won’t be working entirely from scratch, so [download the starter code](https://github.com/Unboxed-Software/solana-intro-frontend/tree/starter). The starter project uses Next.js and Typescript. If you’re used to a different stack, don’t worry! The web3 and Solana principles you’ll learn throughout these lessons are applicable to whichever frontend stack you’re most comfortable with.

### 1. Get oriented

Once you’ve got the starter code, take a look around. Install the dependencies with `npm install` and then run the app with `npm run dev`. Notice that no matter what you put into the address field, when you click “Check SOL Balance” the balance will be a placeholder value of 1000.

Structurally, the app is composed of `index.tsx` and `AddressForm.tsx`. When a user submits the form, the `addressSubmittedHandler` in `index.tsx` gets called. That’s where we’ll be adding the logic to update the rest of the UI.

### 2. Install dependencies

Use `npm install @solana/web3.js` to install our dependency on Solana’s Web3 library.

### 3. Set the address balance

First, import `@solana/web3.js` at the top of `index.tsx`.

Now that the library is available, let’s go into the `addressSubmittedHandler()` and create an instance of `PublicKey` using the address value from the form input. Next, create an instance of `Connection` and use it to call `getBalance()`. Pass in the value of the public key you just created. Finally, call `setBalance()`, passing in the result from `getBalance`. If you’re up to it, try this independently instead of copying from the code snippet below.

```ts
import type { NextPage } from 'next'
import { useState } from 'react'
import styles from '../styles/Home.module.css'
import AddressForm from '../components/AddressForm'
import * as Web3 from '@solana/web3.js'

const Home: NextPage = () => {
  const [balance, setBalance] = useState(0)
  const [address, setAddress] = useState('')

  const addressSubmittedHandler = async (address: string) => {
    setAddress(address)
    const key = new Web3.PublicKey(address)
    const connection = new Web3.Connection(Web3.clusterApiUrl('devnet'));
    const balance = await connection.getBalance(key);
    setBalance(balance / Web3.LAMPORTS_PER_SOL);
  }
  ...
}
```

Most of the time when dealing with SOL, the system will use lamports instead of SOL. Since computers are better at handing whole numbers than fractions, we generally do most of our transactions in whole lamports, only converting back to Sol to display the value to users. This is why we take the balance returned by Solana and dividing it by `LAMPORTS_PER_SOL`. 

Before setting it to our state, we also convert it to SOL using the `LAMPORTS_PER_SOL` constant.

At this point you should be able to put a valid address into the form field and click “Check SOL Balance” to see both the Address and Balance populate below.

### 4. Handle invalid addresses

We’re just about done. The only remaining issue is that using an invalid address doesn’t show any error message or change the balance shown. If you open the developer console, you’ll see `Error: Invalid public key input`. When using the `PublicKey` constructor, you need to pass in a valid address or you’ll get this error.

To fix this, let’s wrap everything in a `try-catch` block and alert the user if their input is invalid.

```ts
const addressSubmittedHandler = async (address: string) => {
  try {
    setAddress(address);
    const key = new Web3.PublicKey(address);
    const connection = new Web3.Connection(Web3.clusterApiUrl("devnet"));
    const balance = await connection.getBalance(key)
    setBalance(balance / Web3.LAMPORTS_PER_SOL);
  } catch (error) {
    setAddress("");
    setBalance(0);
    alert(error);
  }
};
```

Notice that in the catch block we also cleared out the address and balance to avoid confusion.

We did it! We have a functioning site that reads SOL balances from the Solana network. You’re well on your way to achieving your grand ambitions on Solana. If you need to spend some more time looking at this code to better understand it, have a look at the complete [solution code](https://github.com/Unboxed-Software/solana-intro-frontend). Hang on tight, these lessons will ramp up quickly.

# Challenge

Since this is the first challenge, we’ll keep it simple. Go ahead and add on to the frontend we’ve already created by including a line item after “Balance”. Have the line item display whether or not the account is an executable account or not. Hint: there’s a `getAccountInfo()` method.

Since this is DevNet, your regular mainnet wallet address will _not_ be executable, so if you want an address that _will_ be executable for testing, use `CenYq6bDRB7p73EjsPEpiYN7uveyPUTdXkDkgUduboaN`.

![Screenshot of final challenge solution](../assets/intro-frontend-challenge.png)

If you get stuck feel free to take a look at the [solution code](https://github.com/Unboxed-Software/solana-intro-frontend/tree/challenge-solution).
