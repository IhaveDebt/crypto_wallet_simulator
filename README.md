/**
 * Crypto Wallet Transaction Simulator
 *
 * Simulates a lightweight crypto ledger, including wallet creation,
 * transfers, and simple proof-of-work validation for transactions.
 */

import crypto from "crypto";

type Wallet = { id: string; balance: number };
type Transaction = { from: string; to: string; amount: number; hash: string; nonce: number };

class BlockchainSimulator {
  private wallets = new Map<string, Wallet>();
  private transactions: Transaction[] = [];

  createWallet(balance = 1000): string {
    const id = crypto.randomUUID();
    this.wallets.set(id, { id, balance });
    return id;
  }

  getBalance(id: string) {
    return this.wallets.get(id)?.balance ?? 0;
  }

  transfer(from: string, to: string, amount: number) {
    const sender = this.wallets.get(from);
    const receiver = this.wallets.get(to);
    if (!sender || !receiver) throw new Error("Invalid wallet");
    if (sender.balance < amount) throw new Error("Insufficient funds");

    let nonce = 0, hash = "";
    do {
      nonce++;
      hash = crypto.createHash("sha256").update(`${from}${to}${amount}${nonce}`).digest("hex");
    } while (!hash.startsWith("0000")); // proof-of-work simulation

    sender.balance -= amount;
    receiver.balance += amount;

    const tx: Transaction = { from, to, amount, hash, nonce };
    this.transactions.push(tx);
    console.log(`✅ TX: ${from.slice(0, 6)} → ${to.slice(0, 6)} | ${amount} | hash=${hash.slice(0, 8)}...`);
  }
}

if (require.main === module) {
  const chain = new BlockchainSimulator();
  const w1 = chain.createWallet();
  const w2 = chain.createWallet();

  console.log("Wallets created:", w1, w2);
  chain.transfer(w1, w2, 50);
  console.log("Final balances:", { w1: chain.getBalance(w1), w2: chain.getBalance(w2) });
}
