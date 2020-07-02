This is a fork of tornado cash. The tornado cash mixer is adjusted to allow votes. This opens the possibility for instant private (mixed) voting.

# Tornado governance
Below first the description of tornado cash for background information. Then the description of changes for tornado voting.

#### Tornado cash 
Tornado Cash is a non-custodial Ethereum and ERC20 privacy solution based on zkSNARKs. It improves transaction privacy by breaking the on-chain link between recipient and destination addresses. It uses a smart contract that accepts ETH deposits that can be withdrawn by a different address. Whenever ETH is withdrawn by the new address, there is no way to link the withdrawal to the deposit, ensuring complete privacy.

To make a deposit user generates a secret and sends its hash (called a commitment) along with the deposit amount to the Tornado smart contract. The contract accepts the deposit and adds the commitment to its list of deposits.

Later, the user decides to make a withdrawal. In order to do that, the user should provide a proof that he or she possesses a secret to an unspent commitment from the smart contract’s list of deposits. zkSnark technology allows that to happen without revealing which exact deposit corresponds to this secret. The smart contract will check the proof, and transfer deposited funds to the address specified for withdrawal. An external observer will be unable to determine which deposit this withdrawal came from.

You can read more about it in [this medium article](https://medium.com/@tornado.cash/introducing-private-transactions-on-ethereum-now-42ee915babe0)

#### Tornado voting
Tornado voting is an iteration of tornado cash and Continuous Approval Voting (CAV). It uses the principle of the tornado cash mixer to mix votes in the CAV style governance system currently used by MakerDao. 

The design is as follows:
1. Holders of the governance token deposit their token in a pool. This pool can receive interest (for example DSR style) to incentivice participation. Deposits to the pool are only allowed for fixed values (for example 0.1, 1, 10 and 100). When depositing to the pool the token holder needs to submit a commit (same as tornado cash).
2. Next a user can add their vote by revealing the nullifier of the commit and an amount < or == then their deposit (0.1, 1, 10 or 100). This vote can be made by adding the address they are voting for (based on CAV). Their nullifier is stored in the contract to check double votes. The contract adds the vote to the candidate.
3. During CAV a user can change their vote by simply adding a new vote with their nullifier, this removes the old vote for their nullifier and adds the new vote weight. Their nullifier can never have a vote weight more then their deposit.
4. To remove the token from the pool simply set vote total to 0 and withdraw using the nullifier. 

The main benefits of this implementation are:
- The pool (potential) interest is an incentive for token holders to add to the privacy pool. Further it mitigates the risk of oppertunity cost, because the token holder locks the token for voting instead of earning interest.
- Tornado governance fixes the low liquidaty in mixers for governance tokens. Since governance tokens are expected to only move for a vote a mixer is expected to have a low volume of open deposits (privacy). Using a pool and instant vote from the pool uses the full liquidity of the voters and in addition the non voters that like to earn interest from the pool.
- Since the mixer is embedded every governance participant automatically benefits from the added privacy.

## Specs
- Deposit gas const: 1088354 (43381 + 50859 * tree_depth)
- Withdraw gas cost: 301233
- Circuit Constraints = 28271 (1869 + 1325 * tree_depth)
- Circuit Proof time = 10213ms (1071 + 347 * tree_depth)
- Serverless

![image](docs/diagram.png)

## Whitepaper
**[https://tornado.cash/Tornado.cash_whitepaper_v1.4.pdf](https://tornado.cash/Tornado.cash_whitepaper_v1.4.pdf)**

## Was it audited?

Tornado.cash protocols, circuits, and smart contracts were audited by a group of experts from [ABDK Consulting](https://www.abdk.consulting), specializing in zero knowledge, cryptography, and smart contracts.

During the audit no critical issues were found and all outstanding issues were fixed. The results can be found here:

* Cryptographic review https://tornado.cash/Tornado_cryptographic_review.pdf
* Smart contract audit https://tornado.cash/Tornado_solidity_audit.pdf
* Zk-SNARK circuits audit https://tornado.cash/Tornado_circuit_audit.pdf

Underlying circomlib dependency is currently being audited, and the team already published most of the fixes for found issues.

*Tornado governance note: the tornado governance adjustment only affect the tornado.sol smart contract by adding a vote function (and a limitation to the withdraw function). Because these adjustments are limited and do not affect the zero knowlegde code, much of the code and therefore safety is the same as tornado cash. See the above for the tornado cash code audits. Further see the changes for the adjustments related to tornado governance.*

## Requirements
1. `node v11.15.0`
2. `npm install -g npx`

## Usage

You can see example usage in cli.js, it works both in console and in browser.

1. `npm install`
1. `cp .env.example .env`
1. `npm run build` - this may take 10 minutes or more
1. `npx ganache-cli`
1. `npm run test` - optionally runs tests. It may fail on the first try, just run it again.

Use browser version on Kovan:

1. `vi .env` - add your Kovan private key to deploy contracts
1. `npm run migrate`
1. `npx http-server` - serve current dir, you can use any other static http server
1. Open `localhost:8080`

Use with command line version. Works for Ganache, Kovan and Mainnet:
### Initialization
1. `cp .env.example .env`
1. `npm run download`
1. `npm run build:contract`

### Ganache
1. make sure you complete steps from Initialization
1. `ganache-cli -i 1337`
1. `npm run migrate:dev`
1. `./cli.js test`
1. `./cli.js --help`

### Kovan, Mainnet
1. make sure you complete steps from Initialization
1. Add `PRIVATE_KEY` to `.env` file
1. `./cli.js --help`

Example:
```bash
./cli.js deposit ETH 0.1 --rpc https://kovan.infura.io/v3/27a9649f826b4e31a83e07ae09a87448
```
> Your note: tornado-eth-0.1-42-0xf73dd6833ccbcc046c44228c8e2aa312bf49e08389dadc7c65e6a73239867b7ef49c705c4db227e2fadd8489a494b6880bdcb6016047e019d1abec1c7652
> Tornado ETH balance is 8.9
> Sender account ETH balance is 1004873.470619891361352542
> Submitting deposit transaction
> Tornado ETH balance is 9
> Sender account ETH balance is 1004873.361652048361352542

```bash
./cli.js withdraw tornado-eth-0.1-42-0xf73dd6833ccbcc046c44228c8e2aa312bf49e08389dadc7c65e6a73239867b7ef49c705c4db227e2fadd8489a494b6880bdcb6016047e019d1abec1c7652 0x8589427373D6D84E98730D7795D8f6f8731FDA16 --rpc https://kovan.infura.io/v3/27a9649f826b4e31a83e07ae09a87448 --relayer https://kovan-frelay.duckdns.org
```

> Relay address:  0x6A31736e7490AbE5D5676be059DFf064AB4aC754
> Getting current state from tornado contract
> Generating SNARK proof
> Proof time: 9117.051ms
> Sending withdraw transaction through relay
> Transaction submitted through the relay. View transaction on etherscan https://kovan.etherscan.io/tx/0xcb21ae8cad723818c6bc7273e83e00c8393fcdbe74802ce5d562acad691a2a7b
> Transaction mined in block 17036120
> Done

## Deploy ETH Tornado Cash
1. `cp .env.example .env`
1. Tune all necessary params
1. `npx truffle migrate --network kovan --reset --f 2 --to 4`

## Deploy ERC20 Tornado Cash
1. `cp .env.example .env`
1. Tune all necessary params
1. `npx truffle migrate --network kovan --reset --f 2 --to 3`
1. `npx truffle migrate --network kovan --reset --f 5`

**Note**. If you want to reuse the same verifier for all the instances, then after you deployed one of the instances you should only run 4th or 5th migration for ETH or ERC20 contracts respectively (`--f 4 --to 4` or `--f 5`).

## How to resolve ENS name to DNS name for a relayer
1. Visit https://etherscan.io/enslookup and put relayer ENS name to the form.
2. Copy the namehash (1) and click on the `Resolver` link (2)
![enslookup](docs/enslookup.png)
3. Go to `Contract` tab. Click on `Read Contract` and scrolldown to the `5. text` method.
4. Put the values:
![resolver](docs/resolver.png)
5. Click `Query` and you will get the DNS name. Just add `https://` to it and use it as `relayer url`

## Credits

Special thanks to @barryWhiteHat and @kobigurk for valuable input,
and to @jbaylina for awesome [Circom](https://github.com/iden3/circom) & [Websnark](https://github.com/iden3/websnark) framework

## Minimal demo example
1. `npm i`
1. `ganache-cli -d`
1. `npm run download`
1. `npm run build:contract`
1. `cp .env.example .env`
1. `npm run migrate:dev`
1. `node minimal-demo.js`

## Emulate MPC trusted setup ceremony
```bash
cargo install zkutil
npx circom circuits/withdraw.circom -o build/circuits/withdraw.json
zkutil setup -c build/circuits/withdraw.json -p build/circuits/withdraw.params
zkutil export-keys -c build/circuits/withdraw.json -p build/circuits/withdraw.params -r build/circuits/withdraw_proving_key.json -v build/circuits/withdraw_verification_key.json
zkutil generate-verifier -p build/circuits/withdraw.params -v build/circuits/Verifier.sol
sed -i -e 's/pragma solidity \^0.6.0/pragma solidity 0.5.17/g' ./build/circuits/Verifier.sol
```
