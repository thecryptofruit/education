# Zero knowledge, from proofs to rollups
`ZKP`: zero-knowledge proof; `ZKR`: zero-knowledge rollup  
Desideratum ZKR: primarily, the scalability of a blockchain-based system, secondly, any privacy enhancement is welcomed.

*The document is in an eternal progress ...*

## Contents
* [ZKP](#zkp)
  * [Historical intro](#historical-intro)
  * [Overview](#overview)
   * [Properties](#properties)
   * [Variants](#variants)
     * [zk-SNARK](#zk-snark)
     * [BULLETPROOFS](#bulletproofs)
     * [zk-STARK](#zk-stark)
      * [zk-SHARK](#zk-shark)
      * [Some comparisons](#some-comparisons)
 * [ZKR](#zkr)
   * [History](#history)
 * [Rollups](#rollups)
 * [Companies and projects](#companies-and-projects)
   * [OR projects](#or-projects)
   * [ZKR projects](#zkr-projects)
   * [General ZKP](#general-zkp)

# ZKP
## Historical intro
In the paper “The knowledge complexity of interactive proof-systems”, a 3-member team from MIT and University of Toronto coined the term zero knowledge (Shafi Goldwasser, Silvio Micali, Charles Rackoff; 1982-1988; [available online](https://people.csail.mit.edu/silvio/Selected%20Scientific%20Papers/Proof%20Systems/The_Knowledge_Complexity_Of_Interactive_Proof_Systems.pdf)). Curiously, the seemingly magical inner workings of ZKP didn’t catch on immediately. Oded Goldreich, a prominent CSc professor from Israel, who made great contributions in this field, mentions that it took 4 years and 3 conference rejections before their paper was accepted! It was circulated for the first time in 1982, when they were still graduate students, bravely poking into the unknown. Their finally accepted version was submitted in 1985, but published in full only in 1988. After that, nothing was the same. GMR received the first ever Gödel Prize in 1993, awarded to the best papers in theoretical computer science. Prof. Goldwasser and prof. Micali received the Turing Award in 2012 for this major contribution in the computing field. And now, ZKP is one of the hottest topics in cryptography.

  
For inquisitive explorers, prof. Goldreich has an enormous amount of materials freely available on his website, for ZKP see: [http://www.wisdom.weizmann.ac.il/~oded/zk-tut02.html](http://www.wisdom.weizmann.ac.il/~oded/zk-tut02.html)). Cryptography is one of those things that needs years and decades to mature enough to be implemented in practice, yet time and again, it is cracked sooner rather than later. Applying bleeding edge cryptography in immutable blockchain systems is asking for troubles, the weakest links in blockchain are for all to see, forever. Alas, nowadays, it seems there is no longer any time to wait for anything and anyone, we either move fast and break things, or don’t move and rot. It takes patience and survival skills to find a middle way.

## Overview
The original definition from GMR is: “**Zero-knowledge proofs are defined as those proofs that convey no additional knowledge other than the correctness of the proposition in question.**”

 It is mind-boggling, but powerful. Now we can have cryptographic protocols, that enable actors to perform actions without revealing the secrets used in those actions.

Process: `statement -> circuit -> R1CS -> QAP -> ZK algorithm`

Example: [https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649)

A statement is a claim of knowledge or of membership. It claims that there is a relation between 𝑥 (which is public, a.k.a. **instance**) and ⍵ (which is confidential, a.k.a. **witness**). E.g. “I’m older than 18, but won’t disclose my exact age.” A statement is represented through Boolean or arithmetic circuits (a.k.a. flattening).

A circuit is a DAG (directed acyclic graph). There are tools that can transform statements into circuits.  

R1CS (rank-1 constraint system) is used to transform circuits into constraints. R1CS checks the correctness of the computation. It is a common format that ZKP systems ingest. R1CS is represented as groups of vectors `A, B, C` that satisfy a defined equation.

Next, R1CS is transformed into polynomial representation, QAP (quadratic arithmetic program), to enable the evaluation of all constraints through four QAP polynomials: `PA, PB, PC, z` (a.k.a. a vanishing polynomial).

Finally, QAP is fed into the ZKP algorithm, whichever used.  

If the prover and the verifier are to be non-interactive (NIZK), they require some prior common knowledge, a common starting point. Usually, this is CRS (Common Reference String), made by a trusted-setup pre-processing phase, where a trusted entity injects randomness. Since it is trusted, it is also called “*toxic waste*”. Instead of relying on a single entity, the process can be improved using secure MPC (multi-party computation) for generating this randomness in an honest way as long as at least one participant is honest.

Prove: `(state, m) -> (state, 𝜋)`  
state = {setupP, 𝑥, ⍵}  
𝜋: proof

Verify: `(state, 𝜋) -> (state, m)`  
state = {setupV, 𝑥}  
m: {accept, reject}

Usually, `setupP = setupV`.

## Properties
There are **three major properties** for ZKP, required from the `prove` and `verify` actions:

1.  COMPLETENESS - can prove all true statements  
    If 𝑥∈𝐿, then ∃𝜋, V(𝑥,𝜋) = ACCEPT  
    
2.  SOUNDNESS - prover can’t prove false statements  
    If 𝑥∉𝐿, then ∀𝜋, V(𝑥,𝜋) = REJECT  
    
3.  ZK-ness - verifier learns only the fact that the statement is true  

**Interactiveness**
Non-interactive ZKP requires no roundtrip interaction between the prover and the verifier, the prover sends only one message to the verifier, that is proof 𝜋, who can then decide to accept/reject it.

Interactive ZKP are usually simpler and more efficient. After an IZK is designed, it can be turned into NIZK by using Fiat-Shamir transformation, to the trade-off of losing some efficiency and potentially reducing security from statistical (i.e. the probability of attack of an unbounded adversary) to computational (i.e. resource-bounded adversary only). Note that IZK are impractical for public verifications, e.g. in blockchain, because of the requirement for the parties to be online and the prover’s difficulty of ordering the incoming plethora of challenges.  

A proof is **succinct** if it is small (that is, smaller than the statement itself).  

Proces with code examples: [https://media.consensys.net/introduction-to-zksnarks-with-examples-3283b554fc3b](https://media.consensys.net/introduction-to-zksnarks-with-examples-3283b554fc3b)

## Variants
There are three major variants:
1.  zk-SNARK
2.  BULLETPROOFS
3.  zk-STARK

It is about arguments (computationally bounded provers), not proofs (unbounded provers)!

### zk-SNARK
**zk succinct non-interactive argument of knowledge**
[November 2012; [https://eprint.iacr.org/2011/443.pdf](https://eprint.iacr.org/2011/443.pdf) ]

Requires trusted setup, has small proof and fast to prove & verify.

**New SNARKS on the block:**

 - SPARTAN (MS) = almost without a trusted setup [May 2019]
 - SONIC = universal and updatable trusted setup (one setup for all circuits, updateable by anyone) [January 2019]
 - AuroraLight = faster, smaller than SONIC, only batched/parallel proofs [May 2019]
 - Marlin = more efficient than SONIC [September 2019]
 - PLONK = generally faster than Marlin (unless many fan-in gates), using fan-in-two gates instead of R1CS [August 2019]

Some of these are SNORKs (Succinct oecumenical (universal) arguments of knowledge) ^^.

### Bulletproofs
NIZK
[November 2017; [https://eprint.iacr.org/2011/443.pdf](https://eprint.iacr.org/2011/443.pdf) ]

Suited for range proofs on-chain.
Implement MPC + Fiat-Shamir challenge

Have a small size, but slow to prove & verify.

### zk-STARK
**zk scalable transparent argument of knowledge** (n.b. focus on scalable, not succinct)

[March 2018; [https://eprint.iacr.org/2018/046.pdf](https://eprint.iacr.org/2018/046.pdf) ]

Without trusted setup, significantly larger proof size, fast.

### zk-SHARK
**zero-knowledge succinct hybrid arguments of knowledge**

Combine the fast verification of zk-SNARKs with the no-trusted-setup of some non-succinct NIZKs.

[https://dci.mit.edu/zksharks](https://dci.mit.edu/zksharks)

### Some comparisons
Comparisons are nice in general, but are not correct in all cases. Now, there exist SNARKs practically without trusted setups, proving can be parallelized (e.g. see: Wu, Zheng, Chiesa, Ada Popa, Stoica: [Distributed Zero Knowledge Proof System](https://eprint.iacr.org/2018/691); 2018), there are sharks, snorks, storks, what-have-you. It also very much depends on the complexity of the statements.

![ZK-proofs comparison](images/zkp_comparison.png?raw=true "ZK-proofs comparison")

Comprehensive and up-to-date documentation for zero-knowledge proofs can be found in ZKProof Community Reference (ZKProof Community Reference. Version 0.2. Ed. by D. Benarroch,L. T. A. N. Brandão, E. Tromer. Pub. by zkproof.org. Dec. 2019. Updated versions at [https://zkproof.org/reference.pdf](https://zkproof.org/reference.pdf)).

In 2019, BIU Winter School on Cryptography was dedicated to ZK: [https://cyber.biu.ac.il/event/the-9th-biu-winter-school-on-cryptography/](https://cyber.biu.ac.il/event/the-9th-biu-winter-school-on-cryptography/)

A list of ZKP resources is maintained by Matter Labs at: [https://github.com/matter-labs/awesome-zero-knowledge-proofs](https://github.com/matter-labs/awesome-zero-knowledge-proofs)


# ZKR
What is zero-knowledge rollup?  *Zero-knowledge verification of honest execution of delegated computation* is a powerful solution for outsourcing hard computation work and be sure that received results are correct.

**rollup** = batching of transactions and verifying just the proof instead of all TXs by all nodes. Verifying the proof is much faster than the computation of the original payload of processing all the {inputs, code, outputs}. In ZKR, the challenge is usually that the generation of the proof takes lots of resources (esp. with SNARKs, could be mitigated by first publishing the compressed data and the proof only later) or the proof size is too big to be put on-chain (STARKs).
  
Rollups can solve data availability problems (e.g. publishing compressed data on-chain), but there are also rollup constructions that don’t (e.g. [RollupNC](https://github.com/rollupnc/RollupNC) ).

## History
A breakthrough paper in 2013 described efficiency improvements over state-of-the-art proofs of programs execution, but also a practical implementation: ”SNARKs for C: Verifying Program Executions Succinctly and in Zero Knowledge” [16 August, 2013; [https://eprint.iacr.org/2013/507](https://eprint.iacr.org/2013/507); Eli Ben-Sasson, Alessandro Chiesa, Daniel Genkin, Eran Tromer, Madars Virza]. The paper described a (non-blockchain, duh) system with zk-proofs for executions of arbitrary programs in C language. A [21-minutes video talk is also available](https://www.youtube.com/watch?v=nS3smRAfUd8).

Quick overview of the paper indicates a lot of work on zk-work-delegation has been done between 2007-2013. They note a less-standard assumptions in these systems:
“*Many constructions achieving some form of succinct verification are only computationally sound: their security is based on cryptographic assumptions, and therefore are secure only against bounded-size provers. Computational soundness seems inherent in many of these cases ... Proofs (whether interactive or not) that are only computationally sound are also known as arguments ...*” - This distinction has been explained in details in “Separating Succinct Non-Interactive Arguments From All Falsifiable Assumptions” [2011/2013; [https://eprint.iacr.org/2010/610.pdf](https://eprint.iacr.org/2010/610.pdf) ; Gentry, Wichs].

Three days later, on August 19, 2013, Gregory Maxwell proposed blockchain compression called CoinWitness, building on the above idea:
[https://bitcointalk.org/index.php?topic=277389.0](https://bitcointalk.org/index.php?topic=277389.0)

Vitalik Buterin was thinking along similar lines in 2014, as he described in the blog post “Scalability, Part 1: Building on Top” [17 September, 2014; [https://blog.ethereum.org/2014/09/17/scalability-part-1-building-top/](https://blog.ethereum.org/2014/09/17/scalability-part-1-building-top/) ], though not using zk-proofs yet.  

zk rollup systems in Ethereum are generally considered to have started in September 2018. Vitalik Buterin posted “On-chain scaling to potentially ~500 tx/sec through mass tx validation” ([22 September, 2018](https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477)), without a reference or mentioning barryWhiteHat's group.

However, at the time of Vitalik’s post, there was already a group working on actual code. They published their research in October “Roll_up / roll_back snark side chain ~17000 tps” ([3 October, 2018](https://ethresear.ch/t/roll-up-roll-back-snark-side-chain-17000-tps/3675); BarryWhitehat, Alex Gluchowski, HarryR, Yondon Fu, Philippe Castonguay), but the initial code was committed to Github already on [5 September, 2018](https://github.com/barryWhiteHat/roll_up/commit/1ed311342aba80974ef353247a82c43fb090b800).

Zero-knowledge in rollups has nothing to do with privacy. There are lots of new rollups coming to life since 2019, mostly classified as “*optimistic rollups*”, though there also exists a product with that exact same name. Some came from sidechain designs, some from Plasma designs, e.g. Ignis, but renamed and renamed again. Just ‘coz.

# Rollups
Rollup techniques need not involve zero knowledge. The name *rollups* implies that a batch of off-chain transactions is “rolled-up” into a single, one on-chain transaction. They come with all the data, which is compressed and can be either guaranteed to be true (a.k.a. a *proof of validity*), or can be only "certain" about it when we survive the challenge period (a.k.a. a *fraud proof*), depending on the techniques used. 

Rollups work on an idea of a *commit-chain*, where the operator/aggregator publishes sidechain checkpoints on the main chain. See the paper from 2018: [Commit-Chains: Secure, Scalable Off-ChainPayments](https://eprint.iacr.org/2018/642.pdf) by Rami Khalil, Alexei Zamyatin, Guillaume Felley, Pedro Moreno-Sanchez, Arthur Gervais - both approaches are mentiond: using challenge-response checkpoints (a.k.a. NOCUST) and the ZKP-guaranteed state transitions (a.k.a. NOCUST-ZKP).

![Blockchain rollup concept](images/rollup_concept.png?raw=true "The concept of rollups")

State roots are the tips of cryptographic accumulators (usually Merkle trees, though there are other/better ones, e.g. RSA accumulators). Accumulators are used to prove membership in a set, that compared to Merkle proofs don't grow logarithmically, but can be constant in size.  
Longer description of accumulators by Georgios Konstantopoulos, January 2019: [https://blog.goodaudience.com/deep-dive-on-rsa-accumulators-230bc84144d9](https://blog.goodaudience.com/deep-dive-on-rsa-accumulators-230bc84144d9).  
See the paper about RSA inside SNARKs from December 2019: [Scaling Verifiable Computation Using Efficient Set Accumulators](https://eprint.iacr.org/2019/1494.pdf) by Alex Ozdemir, Riad S. Wahby, Dan Boneh.  

The entity posting rollups on-chain is called an **operator** or sometimes an **aggregator**. As centralized as this sounds, this entity is non-custodial and not trusted. Even in the case of non-operational operator, it would be picked up by others who could either step in, or there's a pool of operators ([see the multi-operator model by Matter Labs](https://medium.com/matter-labs/introducing-matter-testnet-502fab5a6f17)), or the sidechain switches to a limp-mode ("exit" - withdrawals only).  

Lots of things to think about when doing rollups. Which property is the most important to a particular use case, where to start, who to ask? There are practically no live projects in production, but 2020 is the year! Many of the properties in the diagram are correlated.

![Blockchain rollup considerations](images/rollup_dimensions.png?raw=true "Things to consider about rollups")

If all you need is simple transfers, then *universality* is no longer an issue and simple rollups are (almost) ready. *Interactiveness* can be changed, at least in the case of zk-based rollups (IZK to NIZK is possible). *Data availability* is a difficult one, though it feels that if we manage to have compressed data on-chain (maybe not as part of the state, but historical call arguments (calldata) and/or logs, or erasure codes or [moving it inside a ZKP](https://www.reddit.com/r/ethereum/comments/9d65e7/scaling_ethereum_with_snarks/e5fmc1r)), we have a workable solution. *Privacy* - while that is what is usually gained from ZKP, in rollups the privacy is not affected. Note however that doesn't mean ZKR can't be used in privacy settings (see ZK-ZK setups). *Finality* is tightly connected to interactiveness - if a rollup uses validity proofs, it doesn’t need challenges as everything that is accepted as valid, is final on the sidechain and eventually final on the main chain; if fraud proofs are used for challenging optimistic blocks, then there must be a challenge period and finality depends on it. *Scalability* is the big improvement with rollups, numbers going as high as 10-100x, which would take us pretty far in the future. *Security* - as always, a nightmare for some, a life-time challenge for others.  

There’s a handful of projects doing rollups for scalability purposes. Some as old as 15 months even. The pace of research in this field is indicating that there indeed exists a ray of hope for scalability. Not just for Ethereum and not just for Ethereum 1.x. [Even Bitcoin](https://www.reddit.com/r/btc/comments/e8u89w/this_is_what_happens_when_the_devs_are_paid_for/faeph4f) might someday incorporate similar principles! Say, in a couple of years?  
See also commit-chain presentation on Bitcoin in 2019: [Non Custodial Sidechains for Bitcoin utilizing Plasma Cash and Covenants](https://telaviv2019.scalingbitcoin.org/files/plasma-cash-towards-more-efficient-plasma-constructions.pdf) by Georgios Konstantopoulos.

![Blockchain rollup implementations](images/rollups.png?raw=true "Rollup projects")

Which one to choose? It seems to me that this question is at least a little bit easier than which Plasma design to pick was. There are prominent experts working on it, there’s tens of millions of dollars deployed, the expectations are extremely high, but so are the initial measurements of rollup improvements.

Most of the discussion these days is around zk-rollups, which we’ve described as being presented in 2018, vs. optimistic rollups, which came around June 2019, mostly from John Adler, then working at Consensys, now at Fuel Labs ([https://ethresear.ch/t/minimal-viable-merged-consensus/5617](https://ethresear.ch/t/minimal-viable-merged-consensus/5617) ). They are similar, but mainly differ in that the OR is easier to implement for general use, but since it doesn’t use undisputable ZKP for guaranteeing the validity of the new state, it is optimistic that the new state is correct and thus needs an added verification game which enables dispute resolution about new states, meaning it takes a bit more time and storage for it to work.

Great comparison on Coinmonks: [ZK Rollup & Optimistic Rollup](https://medium.com/coinmonks/zk-rollup-optimistic-rollup-70c01295231b)  


#### Some references to projects:  
Zk-rollup, the father of it all: [https://github.com/barryWhiteHat/roll_up](https://github.com/barryWhiteHat/roll_up)

ZK-rollup calculations from Iden3: [https://iden3.io/post/istanbul-zkrollup-ethereum-throughput-limits-analysis](https://iden3.io/post/istanbul-zkrollup-ethereum-throughput-limits-analysis)

ZK-rollup benefits from Loopring: [https://medium.com/loopring-protocol/we-take-the-ultimate-non-custodial-test-b5528fafbec2](https://medium.com/loopring-protocol/we-take-the-ultimate-non-custodial-test-b5528fafbec2)

ZK Sync from Matter Labs: [https://medium.com/matter-labs/introducing-matter-testnet-502fab5a6f17](https://medium.com/matter-labs/introducing-matter-testnet-502fab5a6f17)

Optimistic rollup story by John Adler: [https://medium.com/@adlerjohn/the-why-s-of-optimistic-rollup-7c6a22cbb61a](https://medium.com/@adlerjohn/the-why-s-of-optimistic-rollup-7c6a22cbb61a)

Optimistic rollup from Plasma: [https://medium.com/plasma-group/ethereum-smart-contracts-in-l2-optimistic-rollup-2c1cef2ec537](https://medium.com/plasma-group/ethereum-smart-contracts-in-l2-optimistic-rollup-2c1cef2ec537)

StarkDEX proofs: [https://medium.com/starkware/starkdex-deep-dive-the-stark-core-engine-497942d0f0ab](https://medium.com/starkware/starkdex-deep-dive-the-stark-core-engine-497942d0f0ab)

StarkWare comparing validity and fraud proofs: [https://medium.com/starkware/validity-proofs-vs-fraud-proofs-4ef8b4d3d87a](https://medium.com/starkware/validity-proofs-vs-fraud-proofs-4ef8b4d3d87a)

Arbitrum description and discussion: [https://ethresear.ch/t/introducing-arbitrum-a-new-layer-2-solution/3825](https://ethresear.ch/t/introducing-arbitrum-a-new-layer-2-solution/3825)  

Plasma vs. rollups, an excellent article by Ashwin Ramachandran & Haseeb Qureshi: [The Life and Death of Plasma](https://medium.com/dragonfly-research/the-life-and-death-of-plasma-b72c6a59c5ad)


#### Attacks:
Attack on OR, by Alex Gluchowski: [https://ethresear.ch/t/nearly-zero-cost-attack-scenario-on-optimistic-rollup/6336](https://ethresear.ch/t/nearly-zero-cost-attack-scenario-on-optimistic-rollup/6336)

Censorship attacks in rollups, by Ed Felten: [https://medium.com/offchainlabs/fighting-censorship-attacks-on-smart-contracts-c026a7c0ff02](https://medium.com/offchainlabs/fighting-censorship-attacks-on-smart-contracts-c026a7c0ff02)

#### Privacy-focused ZKR
AZTEC is working on *universal recursion for SNARKs*, ZK<sup>2</sup>:  
[https://medium.com/aztec-protocol/zk-zk-rollup-code-release-i-97216347b3bc](https://medium.com/aztec-protocol/zk-zk-rollup-code-release-i-97216347b3bc)  
  
Using money orders, see *Account-based anonymous rollup with unlinkable transactions*, by Olivier Bégassat, Alexandre Belling, Nicolas Liochon:  
[https://ethresear.ch/uploads/short-url/3sC7fdYGibrC5vrCcgyPm4GPZWK.pdf](https://ethresear.ch/uploads/short-url/3sC7fdYGibrC5vrCcgyPm4GPZWK.pdf)

#### The three articles from Vitalik Buterin on this matter:  
September 2014: [https://blog.ethereum.org/2014/09/17/scalability-part-1-building-top/](https://blog.ethereum.org/2014/09/17/scalability-part-1-building-top/)  
September 2018: [https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477](https://ethresear.ch/t/on-chain-scaling-to-potentially-500-tx-sec-through-mass-tx-validation/3477)  
August 2019: [https://vitalik.ca/general/2019/08/28/hybrid_layer_2.html](https://vitalik.ca/general/2019/08/28/hybrid_layer_2.html)

# Companies and projects
## OR projects
Fuel Labs  
[https://github.com/FuelLabs/fuel-core](https://github.com/FuelLabs/fuel-core)  
Fuel based on OR, UTXO model, Yul language, fast withdrawals possible [via liquidity providers](https://ethresear.ch/t/trustless-two-way-bridges-with-side-chains-by-halting/5728)

Aurora Labs  
O2 rollup, Optimized Optimistic Rollup, using IDEX tokens, data availability challenges force data to onchain  
[IDEX exchange](https://blog.idex.io/all-posts/o2-rollup-overview)

Interstate Network  
[https://www.dropbox.com/s/4yy78n1uhwjgq4x/interstate%20whitepaper%20v1.pdf?dl=0](https://www.dropbox.com/s/4yy78n1uhwjgq4x/interstate%20whitepaper%20v1.pdf?dl=0)  
Interstate One based on OR

NutBerry  
[https://github.com/NutBerry/stack](https://github.com/NutBerry/stack)

  
Optimism (renamed from Plasma Group on [15th January 2020](https://medium.com/ethereum-optimism/optimism-cd9bea61a3ee))  
[https://github.com/plasma-group/ovm/tree/master/specs#ovm-layer-2-constructions](https://github.com/plasma-group/ovm/tree/master/specs#ovm-layer-2-constructions)  
OVM OR as L1 of L2 scaling solutions ¯\_(ツ)_/¯  
Monorepo for wizards: [https://github.com/plasma-group/pigi](https://github.com/plasma-group/pigi)  

LeapDAO (stopped their Plasma Leap mainnet in [March 2020](https://leapdao.org/blog/mainnet-shutdown/))  
_Don't really know which way they go now?_  

SKALE Network  
[https://github.com/skalenetwork](https://github.com/skalenetwork)  
BLS-rollup (not ZK, not OR)

## ZKR projects
Loopring Project  
[https://loopring.org](https://loopring.org)  
Loopring protocol 3, built on ZKR, has on-chain withdrawal requests, LRC token  
[https://wedex.io/](https://wedex.io/) the first DEX on ZKR

Iden3  
[zk-rollup](https://docs.iden3.io/#/rollup/rollup) using iden3's circom libs  
As an identity provider, why rollups? [Read here, it's due to making lots of claims, gaslessly.](https://blog.iden3.io/zkrollup-to-universal-identities.html).

Matter Labs  
[https://matter-labs.io/](https://matter-labs.io/)  
ZK Sync, built on ZKR, also has high-priority withdrawal tx
  
Offchain Labs  
[https://offchainlabs.com/](https://offchainlabs.com/)  
Arbitrum Rollup  
[https://github.com/OffchainLabs](https://github.com/OffchainLabs)

Starkware  
[https://starkware.co/](https://starkware.co/)  
StarkDEX, rollup on STARKs, March 2019  
[https://youtu.be/H16nWlj3C_M](https://youtu.be/H16nWlj3C_M)

## General ZKP
iden3  
[Circom](https://github.com/iden3/circom) for constructing circuits

liszt (by ConsenSys)  
[https://github.com/ConsenSys/liszt](https://github.com/ConsenSys/liszt)  
cross-rollup transfers

Zokrates  
[https://github.com/Zokrates/ZoKrates](https://github.com/Zokrates/ZoKrates)  
Higher-level lang

EthSnarks  
[https://github.com/HarryR/ethsnarks](https://github.com/HarryR/ethsnarks)  
Lowest-level, multiple DSLs
  
ZEXE
[https://github.com/scipr-lab/zexe](https://github.com/scipr-lab/zexe)  
"decentralized private computation (DPC) scheme"; a POC library implementing off-chain ledger-based txs with public zkSNARK proofs

Tools collections: [https://zkp.science/](https://zkp.science/)
