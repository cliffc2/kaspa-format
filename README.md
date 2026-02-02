# kaspa-format
kaspa address format
Kaspa wallet addresses use a modern, Bech32-encoded format (similar to Bitcoin's Bech32 but customized for Kaspa). They always include a network prefix followed by a colon (:) and then a long string of lowercase alphanumeric characters (Bech32 charset: qpzry9x8gf2tvdw0s3jn54khce6mua7l).Mainnet (Live Kaspa Network – Most Common)Prefix: kaspa:
Full format: Starts with kaspa: followed by ~59–62 Bech32 characters (total length usually around 62–68 characters including the prefix).
Examples (real public ones from explorers/docs):kaspa:qpauqsvk7yf9unexwmxsnmg547mhyga37csh0kj53q6xxgl24ydxjsgzthw5j
kaspa:qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqkx9awp4e (famous "burn address")
kaspa:qr7c9k6903nwr8qttefcsl8g78fwaz2820sf3x3l7dhrxlj6dn2e5rqkxmv08

When copying a wallet address for receiving/sending KAS on the main network (e.g., from Kaspium, Rusty Kaspa Wallet, Kaspa Web Wallet, or exchanges like KuCoin/Gate.io), it must start with kaspa:. Omitting the prefix or using the wrong one will cause transactions to fail.Testnet / Other NetworksKaspa has multiple networks with different prefixes (useful for devs/testing):Testnet-10/11: kaspatest: (e.g., kaspatest:qqnapngv3zxp305qf06w6hpzmyxtx2r99jjhs04lu980xdyd2ulwwmx9evrfz)
Simnet (local simulation): kaspasim:
Devnet: kaspadev:

For normal users/miners/traders in 2026, you almost always want the mainnet kaspa: format.Key DetailsEncoding: Bech32 (human-readable, error-detecting via checksum; case-insensitive but usually lowercase).
Address Types Supported (behind the scenes):Schnorr P2PK (most common/modern)
ECDSA P2PK
P2SH (Pay-to-Script-Hash, rarer)

All valid addresses include a built-in checksum — wallets/explorers will reject invalid ones.
Length variation: Slight differences based on pubkey type (Schnorr vs ECDSA), but always starts with the prefix + colon.
Derivation: Modern wallets use BIP-32/44 path m/44'/111111'/0' (coin type 111111' is Kaspa's SLIP-0044 number). Legacy wallets (old KDX, wallet.kaspanet.io) used non-standard paths and are deprecated — migrate if needed.

Quick Validation TipsPaste into Kaspa explorer 

explorer.kaspa.org

 — if it loads the address page with balance/history, it's valid.
Most wallets auto-validate on paste.
Never send to a kaspatest: address on mainnet (funds lost forever).

https://kaspa.aspectron.org/wallets/primitives/addresses.html

