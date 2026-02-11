# kaspa-format notes (braindump)


testnet-12
https://medium.com/@azard_news/kaspas-testnet-12-relaunch-2-a-major-leap-toward-programmable-money-on-layer-1-5af8b79828ac

kaspa address format (IGRALABS KASWALLET COMPARISON BELOW)

(testnet 10)

To send a transaction and get it written to the Kaspa blockchain you need to: build a Kaspa transaction object, submit it via a Kaspa node’s RPC, then wait for it to be accepted into the DAG.

Below is the practical “format” and flow.

1. Transaction structure (conceptual)
A Kaspa transaction (simplified) has:

Version: usually 0.

Inputs: each input has:

Outpoint: previous transactionId (hex string) and index (u32).
​

Script/signature data for unlocking (Schnorr/Secp script, similar to Bitcoin).

Outputs: each output has:

amount: integer in sompi (1 KAS = 100,000,000 sompi).
​

scriptPublicKey:

version: typically 0.

scriptPublicKey: hex‑encoded locking script for the destination address.

In RPC form (gRPC RpcTransaction), one output looks like:
​

text
message RpcTransactionOutput {
  uint64 amount = 1;
  RpcScriptPublicKey scriptPublicKey = 2;
}
message RpcScriptPublicKey {
  uint32 version = 1;
  bytes scriptPublicKey = 2;
}
2. Building the transaction (UTXO → outputs)
Typical steps (matching what your Rust wallet would do):

Pick UTXOs (unspent outputs) from your address:

Each has transactionId, index, amount, and a scriptPublicKey.
​

Decide:

amountToSend to the recipient.

amountAsChange back to your own address.

fee (e.g. 3000 sompi).
​

Construct:

Input referencing one UTXO:

prevTxId = <utxo.txid>

outputIndex = <utxo.index>

Output to recipient:

amount = amountToSend

scriptPublicKey = Script(new Address(recipient)).toBuffer().toString("hex") (in JS libs).

Output for change:

amount = amountAsChange

same script format, but your own address.
​

Example (JS style, but same structure for Rust):
​

js
const tx = new Transaction();
tx.setVersion(0);

const txInput = new Transaction.Input.PublicKey({
  prevTxId: selectedUtxo.outpoint.transactionId,
  outputIndex: selectedUtxo.outpoint.index,
  script: selectedUtxo.utxoEntry.scriptPublicKey.scriptPublicKey,
  sequenceNumber: 0,
});

const txOutput = new Transaction.Output({
  script: new Script(new Address(kaspaAddress)).toBuffer().toString('hex'),
  satoshis: amountToSend,
});

const txChange = new Transaction.Output({
  script: new Script(new Address(kaspaAddress)).toBuffer().toString('hex'),
  satoshis: amountAsChange,
});

tx.addInput(txInput);
tx.addOutput(txOutput);
tx.addOutput(txChange);
Your Rust code would mirror this logic, just with Rust types.

3. Signing the transaction
For each input, compute the appropriate sighash (Kaspa uses Schnorr/ECDSA depending on tools) and sign with the private key.

Place the signature (and any pubkey data) in the input’s unlocking script (Kaspa’s Script format, analogous to Bitcoin’s scriptSig / witness).

In Rust, it looks like:

rust
let msg = secp256k1::Message::from_slice(sig_hash.as_bytes())?;
let sig: [u8; 64] = *schnorr_key.sign_schnorr(msg).as_ref();

// signature_script = OP_DATA_65 <64-byte-sig || 1-byte-sighash_type>
mutable_tx.tx.inputs[i].signature_script =
    std::iter::once(65u8)
        .chain(sig)
        .chain([SIG_HASH_ALL.to_u8()])
        .collect();
Your KaspaWallet::create_transaction in Rust can encapsulate all of this: assemble inputs/outputs, compute fee, sign inputs, and return a ready‑to‑broadcast transaction.

4. Submitting the transaction to Kaspa
Kaspa nodes expose RPC methods like SubmitTransaction:
​

text
// SubmitTransactionRequestMessage submits a transaction to the mempool
message SubmitTransactionRequestMessage {
  RpcTransaction transaction = 1;
  bool allowOrphan = 2;
}

message SubmitTransactionResponseMessage {
  string transactionId = 1;
  RPCError error = 1000;
}
So the “format to send” is:

A RpcTransaction (or equivalent JSON/GRPC-encoded struct) with:

version

inputs[] (outpoint + unlocking script)

outputs[] (amount + scriptPublicKey).
​

Send it via:

gRPC (Rust: using rusty-kaspa RPC client), or

an existing CLI (kaspawallet / rusty-kaspa CLI) with a hex-encoded transaction.

Example offline flow people use (you can replicate from Rust):
​

Generate unsigned tx: kaspawallet create-unsigned-transaction ...

Sign on an offline machine: kaspawallet sign ...

Broadcast signed hex: kaspawallet broadcast --transaction <tx_hex>.
​

Your Rust wallet can skip the external CLI and directly:

Build a RpcTransaction struct from your internal Transaction.

Call SubmitTransaction on your node.

Never send to a kaspatest: address on mainnet (funds lost forever).

https://kaspa.aspectron.org/wallets/primitives/addresses.html

How Kaswallet works (IGRALABS)
1. Transaction Structure ✅
My notes describe:

text
Version → Inputs → Outputs

Kaswallet implements this through:

create module: Transaction building
sign module: Signing logic
broadcast module: Transaction submission
2. Building Transactions ✅
Your notes show:

rust
// Pick UTXOs → Decide amounts → Construct inputs/outputs → Sign → Broadcast

Kaswallet CLI commands mirror this exactly:

bash
# Step 1: Get UTXOs
kaswallet-cli get-utxos

# Step 2: Create unsigned transaction  
kaswallet-cli create-unsigned-transaction --to <address> --amount <sompi>

# Step 3: Sign (if created elsewhere)
kaswallet-cli sign --transaction <tx_hex>

# Step 4: Broadcast
kaswallet-cli broadcast --transaction <tx_hex>

3. gRPC Integration ✅
Your notes mention:

text
SubmitTransaction RPC method
RpcTransaction struct

Kaswallet architecture:

┌─────────────────────────────────────────┐
│         kaswallet-daemon (gRPC)         │
├─────────────────────────────────────────┤
│  ┌──────────────┐  ┌───────────────┐    │
│  │ Transaction  │  │ RPC Client    │    │
│  │ Builder      │  │ (kaspad)      │    │
│  └──────────────┘  └───────────────┘    │
└─────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│           kaswallet-cli                 │
│  balance, send, create, sign, broadcast │
└─────────────────────────────────────────┘


How to Use Kaswallet on Testnet12
Setup Kaswallet
bash
# Install
git clone https://github.com/IgraLabs/kaswallet.git
cd kaswallet
./install.sh

# Create wallet on testnet
kaswallet-create --testnet

# Start daemon (connects to local kaspad)
kaswallet-daemon --testnet --server='grpc://127.0.0.1:16110'

Fund via Testnet12
bash
# Option 1: Use Rothschild from testnet12
# (get address from rothschild output)

# Option 2: Mine with kaspa-miner
kaspa-miner --testnet --mining-address <kaswallet-address> -p 16210 -t 1

# Check balance
kaswallet-cli --testnet balance

Send Transaction
bash
# Create and broadcast
kaswallet-cli --testnet send --to <recipient-address> --amount 100000000

# Or: Create unsigned, sign separately, broadcast
kaswallet-cli --testnet create-unsigned-transaction --to <addr> --amount 1000 > unsigned.tx
kaswallet-cli --testnet sign --transaction unsigned.tx > signed.tx  
kaswallet-cli --testnet broadcast --transaction signed.tx

