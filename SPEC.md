# Zkopru Protocol Specification

## 1. Account & Elliptic Curve Cryptography

Zkopru uses Baby Jubjub curve for its Elliptic Curve Cryptography. The arithmeric is defined in the reference [paper](https://iden3-docs.readthedocs.io/en/latest/_downloads/33717d75ab84e11313cc0d8a090b636f/Baby-Jubjub.pdf) written by Barry Whitehat and Jordi Baylina.

#### [1.1] - Scalar field of BabyJubjub curve


Let
$$
p = 21888242871839275222246405745257275088548364400416034343698204186575808495617
$$
and, $\mathbb{F}_p$ be the finite field which modular is $p$.

#### [1.2] - Group definition for the Babyjubjub curve points

Let $p$ and $\mathbb{F}_p$ be defined in [1.1].

The Montgomerry form of the Baby Jubjub curve 
$$
E_M: v^2 = u^3 + 168698u^2 + u
$$
The order of $E_M$ is
$$
n = 21888242871839275222246405745257275088614511777268538073601725287587578984328 \\
= 8 \times r
$$
where $r = 2736030358979909402780800718157159386076813972158567259200215660948447373041$ is a prime number.

Let $\mathbb{F}_r$ be the finite field which modular is $r$.

Denote $\mathbb{G}$ the subgroup of points of order $r$, that is
$$
\mathbb{G} = \{\mathbf{P} \in E(\mathbb{F}_p) | r\mathbf{P} = \mathbf{O}\}
$$
 where $\mathbf{O}$ is the infinite zero.

#### [1.3] - 

Let $p$ and $\mathbb{F}_p$ be defined in [1.1].
Let $E_M$, $n$ be defined in [1.2].

The Edward form of the curve $E$ that is birationally equivalent to $E_M$ defined over $\mathbb{F}_p$
$$
E: x^2 + y^2 = 1 + dx^2y^2
$$
where
$d = 9706598848417545097372247223557719406784115219466060233080913168975159366771$.

If $(u, v)$ is a Baby Jubjub point in the Montgomerry form, its equivalent Edward form
$$
(x,y) = \left( \frac{1 + y}{1 - y}, \frac{1 + y}{(1 - y)x} \right)
$$

If $(x, y)$ is a Baby Jubjub point in the Edward form, its equivalent Montgomerry form
$$
(u,v) = \left( \frac{u}{v}, \frac{u - 1}{u + 1} \right)
$$

 
#### [1.4]
$\mathsf{poseidon_n}$ is a poseidon hash function which consumes $n$ number of inputs.

Let
$$
y = \mathsf{poseidon_n}([x_1, x_2, ..., x_n]) \\
$$
Then, 
$$
x_i, y \in \mathbb{F}_p\\
$$
Where $i$ is an integer and $0 < i \le n$

The number of rounds of $\mathsf{poseidon_n}$ depends on $n$, and its recommended value is defined in the table2 and table 4 of the [Poseidon hash research paper](https://eprint.iacr.org/2019/458.pdf)



| $n$ | $t$ | $R_F$ | $R_P$ |
| -------- | -------- | -------- | -------- |
| 2     | 3     | 8     | 57     |
| 3     | 4     | 8     | 56     |
| 4     | 5     | 8     | 60     |

The implementation that Zkopru uses are
* Javascript: https://github.com/iden3/circomlib/blob/cf853c1cc96fa537cb1030f70a6f78e5d80ed0e4/src/poseidon.js
* Circuit: https://github.com/iden3/circomlib/blob/cf853c1cc96fa537cb1030f70a6f78e5d80ed0e4/circuits/poseidon.circom


#### [1.5]
$\mathbf{G}$ is a constant generator point on the babyjubjub curve and its value is $(995203441582195749578291179787384436505546430278305826713579947235728471134,\\ 5472060717959818805561601436314318772137091100104008585924551046643952123905)$.

#### [1.6]
Let $\mathbf{G}$ be as defined in [1.5]
$\mathbf{B} = 8 \cdot \mathbf{G}$.

$\mathbf{B}$ is the base point for the EdDSA in Zkopru that has 8 for its cofactor and 2736030358979909402780800718157159386076813972158567259200215660948447373041 for its order.


#### [1.7]
$s$ is the 32 bytes private key of the account. For the seamless user experience, Zkopru uses Ethereum account's private key or deterministically derived key from it for its corresponding Zkopru account's private seed key.

#### [1.8]
Let $r, \mathbb{F}_r$ be defined in [1.2].
Let $s$ be defined in [1.7].

$a \in \mathbb{F}_r$ is the scalar multiplier that is generated from the private seed key $s$ by the [RFC8032 5.1.5 Key Generation](https://tools.ietf.org/html/rfc8032#section-5.1.5) protocol with blake512 hash function.

1. Hash the 32-byte private key $s$ using $\mathsf{blake512}$, storing the digest in a 64-octet large buffer, denoted h. Only the lower 32 bytes are used for generate the scalar multiplier $a$.
2. Prune the buffer: Discard the lowest three bits of the first octet and the highest bit of the last octet. And set the second highest bit of the last octet.
3. Interpret the buffer as the little-endian integer and modularize with $r$, forming a secret scalar $a$.

#### [1.9]
Let $\mathbf{B}$ be defined in [1.6]
Let $a$ be defined in [1.8]
$\mathbf{A} \in \mathbb{G}$ is the EdDSA public key that is $\mathbf{A} = a \cdot \mathbf{B}$.

#### [1.10]

Let $\mathbb{F}_r$ be defined in [1.2].
Let $s$ be defined in [1.7].
$v \in \mathbb{F}_r$ is used for a viewing key and a nullifier seed where

$$
v = keccak256(s)\ mod\ r
$$

#### [1.11]

Let $\mathsf{poseidon_n}$ be defined in [1.4]
Let $\mathbf{A}$ be defined in [1.9]
Let $v$ be defined in [1.10]
$\mathsf{Pub_{sk}} \in \mathbb{F}_p$ is the public spending key of Zkopru that is
$$
\mathsf{Pub_{sk}} = \mathsf{poseidon_3}(x, y, v)
$$
where $(x,y) = A$.

#### [1.12]

Let $\mathbb{G}$ be defined in [1.2].
Let $\mathbf{B}$ be defined in [1.6].
Let $v$ be defined in [1.10].

$\mathbf{V} \in \mathbb{G}$ is the public viewing key that is
$$
\mathbf{V} = v \cdot \mathbf{B}
$$

#### [1.13]

Let $s$ be defined in [1.7]
Let $\mathsf{Pub_{sk}}$ be defined in [1.11]
Let $\mathbf{V}$ be defined in [1.12]

$\mathcal{Z}$ is the Zkopru account address from the private key $s$.

1. Prepare a 512 bits buffer.
2. Store $\mathsf{Pub_{sk}}$ to the first 256 bits in little-endian mode.
3. Pack $\mathbf{V}$ into 256 bits data.
    :::info
    Point encode/decode protocol: https://tools.ietf.org/html/rfc8032#section-5.1.2
    :::
    1. Let $\mathbf{V} = (x,y)$
    2. Prepare a 256 bits buffer.
    3. Store $y$ into the buffer in little-endian mode.
    4. If $x$ is an odd number fill the most significant bit of the last octet.
4. Store the packed $\mathbf{V}$ to the last 256 bits.
5. Digest the prepared buffer with keccak256 hash function and take the first 32 bits from the resulting hash value as the checksum data.
6. Encode the concatenation of the 512 bits data and its 32 bits checksum with Base58 method.
7. Finally $\mathcal{Z}$ is the encoded result.

#### [1.14]

Let $\mathbb{F}_p$ be defined in [1.1]
Let $\mathsf{poseidon}_5$ be defined in [1.4]
Let $s$ be defined in [1.7]
Let $\mathbf{A}$ be defined in [1.9]

Then $(\mathbf{R}, S)$ is the EdDSA signature of the EdDSA public vector $\mathbf{A}$ for about the message $m \in \mathbb{F}_p$.
$$
\mathsf{EdDSA}(\mathbf{A}, m) = (\mathbf{R}, S)
$$

It follows the [RFC8032](https://tools.ietf.org/html/rfc8032#section-5.1.6) with blake512 hash and $\mathsf{poseidon}_5$ hash function.

1. Sign
   1. hash the 32 bytes private key $s$ with $\mathsf{blake512}$ hash function. Let $\mathsf{h}$ denote the resulting digest.

for the private key hashing and the prefix generation.

:::info
Zkopru is using iden3's EdDSA implementation [here](https://github.com/iden3/circomlib/blob/86c6a2a6f5e8de4024a8d366eff9e35351bc1a2e/src/eddsa.js)
:::


#### [1.15]

Zkopru uses groth16 and BN254(= BN128) curve for the SNARK pairing. As defined in [2.1. Bilinear Groups](https://eprint.iacr.org/2016/260.pdf) of the groth16 paper, $\mathbb{G_1}$ and $\mathbb{G_2}$ are the group of order $q = 21888242871839275222246405745257275088696311157297823662689037894645226208583$.

## 2. Note

#### [2.1]
Zkopru note (denoted $\mathsf{N}$) is a tuple $(S, \mathsf{asset}, \mathsf{salt} )$ where
$S$ is defined in [1.11].
$\mathsf{asset}$ is defined in [2.2]
$\mathsf{salt}$: is a random 16 bytes salt data.

#### [2.2]
Let $p$ be defined in [1.1]
Let $\mathsf{poseidon_n}$ be defined in [1.4]

$\mathsf{asset}$ is a tuple of $\mathsf{(v_{eth}, addr_{token}, v_{erc20}, v_{nft})}$ where
* $\mathsf{v_{eth}}$: The spendable amount of Ether which is less than ${2^{245}}$.
* $\mathsf{addr_{token}}$: The token contract address if the note contains ERC20 or NFT. The value is less than ${2^{160}}$.
* $\mathsf{v_{erc20}}$: The spendable amount of ERC20 token if the $\mathsf{addr_{token}}$ is a registered ERC20 token address on the $\mathsf{ZkopruContract}$. Otherwise $\mathsf{v_{erc20}} = 0$.
* $\mathsf{v_{nft}}$: The token id of the NFT that the note owns. This is less than $p$ and $0$ means it does not own any NFT. Likewise $\mathsf{v_{erc20}}$, ${v_{nft}}$ can be non-zero when only $\mathsf{addr_{token}}$ is a registered ERC721 token address on the $\mathsf{ZkopruContract}$.
    :::warning
    NFTs with ID 0 cannot be deposited on the $\mathsf{ZkopruContract}$.
    ::    :

#### [2.3]
Let $\mathsf{asset, (v_{eth}, addr_{token}, v_{erc20}, v_{nft})}$ be defined in [2.2]
Then, the hash of the asset is computed by:
$$
\mathsf{hash(\mathsf{asset})} = \mathsf{poseidon_4}(\mathsf{v_{eth}, addr_{token}, v_{erc20}, v_{nft}})
$$



#### [2.4]
Let $\mathsf{poseidon_n}$ be defined in [1.4]
Let $S, \mathsf{N, asset, salt}$ be defined in [2.1]
Let $\mathsf{hash(\mathsf{asset})}$ be defined in [2.3]
Then, the hash value of the note is computed by:
$$
\mathsf{hash}(\mathsf{N}) = \mathsf{poseidon_3}(S, \mathsf{salt}, \mathsf{hash(\mathsf{asset})})
$$

## 3.Transaction 

:::info
Please note that the information written in $\mathcal{Calligraphic}$ typeface is known to everyone while information written in $\mathrm{Roman}$ typeface is only known to the prover.
:::

### 3.1 Basic definitions

#### [3.1.1]

Let $\mathsf{N}$ be defined in [2.1].
Let $\mathsf{hash}(\mathsf{N})$ be defined in [2.4].

The transaction output of Zkopru
$$
\mathrm{Out} = (\mathsf{N}, t, \mathcal{N})
$$

where $t$ is the type of the output and $\mathcal{N}$ is the public signals of the output that is

$\mathcal{N} = \mathsf{(to, v'_{eth}, addr'_{token}, v'_{erc20}, v'_{nft}, fee_{L1})} \\
\ = \left\{\begin{array}{lr}
        (0, 0, 0, 0, 0, 0) & \text{for } t = 0 & \text{(utxo)}\\
        \mathsf{(recipient, v_{eth}, addr_{token}, v_{erc20}, v_{nft}, fee_{caller})} & \text{for } t = 1 & \text{(withdrawal)}\\
        \mathsf{(dest, v_{eth}, addr_{token}, v_{erc20}, v_{nft}, fee_{migration})} & \text{for } t = 2 &  \text{(migration)}
        \end{array}\right\}$

Then, the public outflow data is defined as

$$
\mathcal{Out} = (\mathsf{hash}(\mathsf{N}), t, \mathcal{N})
$$

#### [3.1.2]

Let $\mathsf{t}$ be defined in [3.1.1]
$\mathrm{U}$ is a UTXO type raw transaction output $\mathrm{Out}$ which $t = 0$ while $\mathcal{U}$ is its corresponding public output $\mathcal{Out}$.

Then, $\mathsf{hash}(\mathrm{U}) = \mathsf{hash(N)}$.

#### [3.1.3]
Let $\mathsf{t}$ be defined in [3.1.1]
$\mathrm{W}$ is a withdrawal type raw transaction output $\mathrm{Out}$ which $t = 1$ while $\mathcal{W}$ is its corresponding public output $\mathcal{Out}$.

Then,
$\mathsf{hash}(\mathcal{W}) = keccak256(\mathsf{encodePacked(\mathcal{W})})$

where $\mathsf{encodePacked(\mathcal{W})}$ is 200 bytes data that is encoded with big-endian by the following table:
| Position  | Size       | Data     |
| --------- | ---------  | -------- |
| 0-32      | 32 bytes   | $\mathsf{hash(N)}$        |
| 32-52     | 20 bytes   | $\mathcal{N}.\mathsf{to}$ |
| 52-84     | 32 bytes   | $\mathcal{N}.\mathsf{v_{eth}}$ |
| 84-104    | 20 bytes   | $\mathcal{N}.\mathsf{addr_{token}}$ |
| 104-136   | 32 bytes   | $\mathcal{N}.\mathsf{v_{erc20}}$ |
| 136-168   | 32 bytes   | $\mathcal{N}.\mathsf{v_{nft}}$ |
| 168-200   | 32 bytes   | $\mathcal{N}.\mathsf{fee_{L1}}$ |



#### [3.1.4]
Let $\mathsf{t}$ be defined in [3.1.1]
$\mathrm{M}$ is a migration type raw transaction output $\mathrm{Out}$ which $t = 2$ and $\mathsf{hash}(\mathrm{M}) = \mathsf{hash(N)}$.
And $\mathcal{M}$ is its corresponding public output $\mathcal{Out}$

#### [3.1.5]

Let $\mathrm{U}$ be defined in [3.1.2].
Let $\mathsf{Tree_\mathsf{utxo}}$ be defined in [5.3.1].

$\mathsf{pos}(\mathrm{U})$ is the leaf index of $\mathrm{U}$ in the UTXO Merkle tree $\mathsf{Tree}_\mathsf{utxo}$.

#### [3.1.6]

Let $v$ be defined in [1.10]
Let $\mathrm{U}$ be defined in [3.1.2].
Let $\mathsf{pos}(\mathrm{U})$ be defined in [3.1.5].

Then the nullifier that prevents double spending is computed by:

$$
\mathsf{nullifier}(\mathrm{U}) = \mathsf{poseidon_2}(v, \mathsf{pos}(\mathrm{U}))
$$

#### [3.1.7]

Let $\mathsf{hash}(\mathsf{N})$ be defined in [2.4].
Let $\mathrm{U}$ be defined in [3.1.1].
Let $\mathsf{pos}(\mathrm{U})$ be defined in [3.1.5].
Let $\mathsf{Tree_\mathsf{utxo}}$ be defined in [5.3.1].
Let $\mathsf{root_{utxo}}^{(n)}$ be defined in [6.1.2].

Let the root of $\mathsf{Tree}_\mathsf{utxo}$ at the $n$-th layer 2 block be denoted $\mathsf{root_{utxo}}^{(n)}$.
$\mathsf{Sib}_\mathsf{utxo}^{(n)}(\mathrm{U})$ is the set of all sibling node data of UTXO $\mathrm{U}$ at the $n$-th layer 2 block.

Then $\mathsf{Merkle}^{(n)}_\mathsf{utxo}(\mathrm{U}) = (\mathsf{hash}(\mathrm{U}), \mathsf{pos}(\mathrm{U}), \mathsf{root_{utxo}}^{(n)}, \mathsf{Sib}_\mathrm{U}^{(n)}(\mathrm{U}))$ that satisfies the Merkle tree inclusion proof.

#### [3.1.8]

Let $\mathsf{Merkle}_\mathsf{utxo}^{(n)}$ be defined in [3.1.7]

The confidential data of the $i$-th input note
$$
\mathrm{In}_i = (\mathrm{U}, (\mathbf{A}, v), \mathbf{R}, S, \mathsf{Merkle}_\mathsf{utxo}^{(n)}(\mathrm{U}), \mathsf{nullifier}(\mathrm{U}))
$$

where

1. $\mathsf{poseidon}_3(x, y, v) == \mathsf{Pub_{sk}}$  where $\mathbf{A} = (x, y)$
2. $\mathsf{hash}(\mathrm{U}), \mathbf{A}, \mathbf{R}, S$ satisfies [1.14]
3. $\mathsf{nullifier}(\mathrm{U})$ satisfies [3.1.6]

Then, its public inflow data is defined as
$$
\mathcal{In} = (\mathsf{nullifier}(\mathrm{U}), \mathsf{root_{utxo}}^{(n)})
$$


#### [3.1.9]

Let $\mathrm{U}$ be defined in [3.1.2]
Let $\mathrm{W}$ be defined in [3.1.3]
Let $\mathrm{M}$ be defined in [3.1.4]
Let $\mathrm{In}$ and $\mathcal{In}$ be defined in [3.1.8]
Let $\mathrm{Out}$ and $\mathcal{Out}$ be defined in [3.1.1]

Then, $\mathrm{Flow}$ is the set of raw inflow and outflow:
$$
\mathrm{Flow} = (\mathrm{[In_1, ..., In_m], [Out_1, ..., Out_n]})
$$

while $\mathcal{Flow}$ is the hidings of $\mathrm{Flow}$ which corresponds to $\mathrm{Flow}$
$$
\mathcal{Flow} = (\mathcal{[In_1, ..., In_m], [Out_1, ..., Out_n]})
$$

#### [3.1.10]

Let $\mathrm{Flow}$ be defined in [3.1.9]

Then, a raw transaction $\mathrm{Tx}$ is

$$
\mathrm{Tx} = (\mathrm{Flow}, f, \mathsf{swap})
$$


where $f$ is the fee for the layer 2 block proposer and $\mathsf{swap}$ is the desired atomic swap output from its paired transaction as defined in ==X.Y.Z==.

### 3.2 ZKP

#### [3.2.1]

$\mathsf{C}_{(x,y)}$ is a Zkopru circuit that consumes $x$ inputs and emits $y$ outputs. 

#### [3.2.2]

Let $\mathsf{C}_{(x,y)}$ be defined in [3.2.1].

$\mathsf{zPK}_{(x,y)}$ is the proving key for the circuit $\mathsf{C}_{(x,y)}$ and should be setup by the multi party computation.

#### [3.2.3]

Let $\mathsf{C}_{(x,y)}$ be defined in [3.2.1].
Let $\mathsf{zPK}_{(x,y)}$ be defined in [3.2.2].

$\mathsf{zVK}_{(x,y)}$ is the verifying key for the circuit $\mathsf{C}_{(x,y)}$ that corresponds to the proving key $\mathsf{zPK}_{(x,y)}$.

#### [3.2.4]

Let $\mathrm{In}$ and $\mathcal{In}$ be defined in [3.1.8]
Let $\mathrm{Flow}, \mathcal{Flow}$ be defined in [3.1.9]
Let $f, \mathsf{swap}$ be defined in [3.1.10]
Let $\mathsf{C}_{(x,y)}$ be defined in [3.2.1].

Then a prover can generate a witness
$$
\mathsf{w} \leftarrow \mathsf{witness}(\mathsf{C}_{(m,n)}, \mathrm{Flow}, \mathcal{Flow}, f, \mathsf{swap})
$$
when$(\mathsf{C}_{(m,n)}, \mathrm{Flow}, \mathcal{Flow}, f, \mathsf{swap})$ satisfies the conditions:

<div style="padding-left: 5rem">

#### [3.2.4.1] - Nullifier proof

Let $\mathrm{In}$ be the part of $\mathrm{Flow}$ as defined in [3.1.9]

Each $\mathrm{In}$ should satisfy the conditions in [3.1.6]


#### [3.2.4.2] - Ownership proof

Let $\mathrm{In}$ be the part of $\mathrm{Flow}$ as defined in [3.1.9]

Each $\mathrm{In}$ should satisfy the conditions in [3.1.8]

#### [3.2.4.3] - Existence Proof(inclusion proof)

Let $\mathrm{U}$ be the part of $\mathrm{In}$ as defined in [3.1.8]

Each $\mathrm{U}$ should satisfy the conditions in [3.1.7]

#### [3.2.4.4] - Public signal proof

Let $\mathrm{Out}$ be the part of $\mathrm{Flow}$ as defined in [3.1.9]
Let $\mathcal{Out}$ be the part of $\mathcal{Flow}$ as defined in [3.1.9]

Then,

$\mathcal{Out}_i$ and $\mathrm{Out}_i$ should satisfty the condition [3.1.1]

#### [3.2.4.5] - Range limit proof to prevent overflow

Let $\mathrm{In}$ be the part of $\mathrm{Flow}$ as defined in [3.1.9]
Let $\mathrm{Out}$ be the part of $\mathrm{Flow}$ as defined in [3.1.9]

Then,

$\mathsf{v_{eth}}$ of $\mathrm{In}$ should be less than $2^{245}$
$\mathsf{v_{erc20}}$ of $\mathrm{In}$ should be less than $2^{245}$
$\mathsf{v_{eth}}$ of $\mathrm{Out}$ should be less than $2^{245}$
$\mathsf{v_{erc20}}$ of $\mathrm{Out}$ should be less than $2^{245}$


#### [3.2.4.6] - Zero-sum proof

#### [3.2.4.6.1] - Ether

Let $\mathsf{v^{(\mathrm{In})}_{eth}}$ be the Ether value of the note $\mathsf{N}$ of $\mathrm{In}$
Let $\mathsf{v^{(\mathrm{Out})}_{eth}}$ be the Ether value of the note $\mathsf{N}$ of $\mathrm{Out}$
Let $f$ be defined in [3.1.10]

Then,

$$
\Sigma_{i=1}^{x} \mathsf{v^{(\mathrm{In}_i)}_{eth}} = \Sigma_{i=1}^{y} \mathsf{v^{(\mathrm{Out}_i)}_{eth}} + f
$$

#### [3.2.4.6.2] - ERC20

Let $\mathsf{v^{(\mathrm{In})}_{erc20}}$ be the ERC20 value of the note $\mathsf{N}$ of $\mathrm{In}$
Let $\mathsf{v^{(\mathrm{Out})}_{erc20}}$ be the ERC20 value of the note $\mathsf{N}$ of $\mathrm{Out}$

Let $\mathsf{addr^{(\mathrm{In})}_{token}}$ be the token value of the note $\mathsf{N}$ of $\mathrm{In}$.
Let $\mathsf{addr^{(\mathrm{Out})}_{token}}$ be the token value of the note $\mathsf{N}$ of $\mathrm{Out}$.

Let's define a fillter function $ftr(x,y)$ as $ftr(x,y) = \left\{\begin{array}{lr}
        0 & \text{for } x \neq y\\
        1 & \text{for } x  = y\\
        \end{array}\right\}$

Then for each $\mathsf{addr} \in \{\mathsf{addr^{(\mathrm{In}_1)}_{token}}, ..., \mathsf{addr^{(\mathrm{In}_x)}_{token}}, \mathsf{addr^{(\mathrm{Out}_1)}_{token}}, ..., \mathsf{addr^{(\mathrm{Out}_y)}_{token}}\}$

It should satisfy
$$
\Sigma_{i=1}^{x} \mathsf{v^{(\mathrm{In}_i)}_{erc20}}\cdot ftr(\mathsf{addr}, \mathsf{addr^{(\mathrm{In}_i)}_{token}}) = \Sigma_{i=1}^{y} \mathsf{v^{(\mathrm{Out}_i)}_{erc20}}\cdot ftr(\mathsf{addr}, \mathsf{addr^{(\mathrm{Out}_i)}_{token}})
$$

#### [3.2.4.6.3] - ERC721

Let $\mathsf{v^{(\mathrm{In})}_{nft}}$ be the NFT id of the note $\mathsf{N}$ of $\mathrm{In}$
Let $\mathsf{v^{(\mathrm{Out})}_{nft}}$ be the NFT id of the note $\mathsf{N}$ of $\mathrm{Out}$

Let $\mathsf{addr^{(\mathrm{In})}_{token}}$ be the token value of the note $\mathsf{N}$ of $\mathrm{In}$.
Let $\mathsf{addr^{(\mathrm{Out})}_{token}}$ be the token value of the note $\mathsf{N}$ of $\mathrm{Out}$.

Let $ftr$ be defined in [3.2.4.6.2]

Then for each $\mathsf{addr}$ and $\mathsf{nft}$ where
$$
\mathsf{addr} \in \{\mathsf{addr^{(\mathrm{In}_1)}_{token}}, ..., \mathsf{addr^{(\mathrm{In}_x)}_{token}}, \mathsf{addr^{(\mathrm{Out}_1)}_{token}}, ..., \mathsf{addr^{(\mathrm{Out}_y)}_{token}}\} \\
\mathsf{nft} \in \{\mathsf{v^{(\mathrm{In}_1)}_{nft}}, ..., \mathsf{v^{(\mathrm{In}_x)}_{nft}}, \mathsf{v^{(\mathrm{Out}_1)}_{nft}}, ..., \mathsf{v^{(\mathrm{Out}_y)}_{nft}}\}
$$

It should satisfy
$$
\Sigma_{i=1}^{x} ftr(\mathsf{nft}, \mathsf{v^{(\mathrm{In}_i)}_{nft}})\cdot ftr(\mathsf{addr}, \mathsf{addr^{(\mathrm{In}_i)}_{token}}) = \Sigma_{i=1}^{y} ftr(\mathsf{nft}, \mathsf{v^{(\mathrm{Out}_i)}_{nft}})\cdot ftr(\mathsf{addr}, \mathsf{addr^{(\mathrm{Out}_i)}_{token}})
$$

</div>

#### [3.2.5] - Proof Generation

Let $\mathbb{G_1}$ and $\mathbb{G_2}$ be defined in [1.15]
Let $\mathsf{C}_{(x,y)}$ be defined in [3.2.1].
Let $\mathsf{zPK}_{(x,y)}$ be defined in [3.2.2].
Let $\mathsf{w}$ be defined in [3.2.4]

Let $\mathsf{prove_{groth16}}$ be the SNARK proving system defined in [https://eprint.iacr.org/2016/260.pdf](https://eprint.iacr.org/2016/260.pdf)

Then the prover can generate the SNARK proof $\pi$ using the proving key $\mathsf{zPK}_{(x,y)}$:
$$
\pi \leftarrow \mathsf{prove_{groth16}}(\mathsf{w}, \mathsf{zPK}_{(x,y)}) \\
\pi = (\mathbf{A}, \mathbf{B}, \mathbf{C})
$$
where $\mathbf{A, C} \in \mathbb{G_1}$ and $\mathbf{B} \in \mathbb{G_2}$

#### [3.2.6] - Proof Verification

Let $\mathsf{C}_{(x,y)}$ be defined in [3.2.1].
Let $\mathsf{zVK}_{(x,y)}$ be defined in [3.2.3].
Let $\mathcal{Flow}$ be defined in [3.1.9]
Let $f, \mathsf{swap}$ be defined in [3.1.10]

Then define the public signal set $\mathcal{P}$ as

$$
\mathcal{P} = (\mathcal{Flow}, f, \mathsf{swap})
$$

then verifiers can verify the zero-knowledge proof using

$$
\mathsf{verify_{groth16}}(\mathcal{P}, \pi, \mathsf{zVK}_{(x,y)}) \in \{0, 1\}
$$

### 3.3 Memo

As all information is shielded properly, there should be a memo field to help the recipient decode the receiving note correctly. As it's an optimistic rollup, the size of calldata increases the transaction cost. Therefore, memo filed only supports output notes that have only one value among Ether, ERC20 and NFT.

Note that the memo field is used for the easy communication without constructing any additional p2p networking layer between wallets.

#### [3.3.1] - Encryption

Let $\mathbf{B}$ be defined in [1.6]
Let $\mathbf{V}$ be defined in [1.12]
Let $\mathsf{salt}$ be defined in [2.1]
Let $\mathrm{Tx}$ be defined in [3.1.10]
Let $\mathsf{tokenId()}$ be defined in [4.6.1].
Let $\mathsf{encode()}$ be the point encode function that is defined in [1.13]

1. The transaction builder pick 1 output $\mathrm{Out}$ from $\mathrm{Tx}$ to include in the memo.
2. Generate a public shared key using ECDH
    1. Get the viewing public key $\mathbf{V}_{}$ by parsing the zk address of $\mathrm{Out}$ as defined in [1.13]
    2. Create a random 16 bytes ephemeral secret key $e$
    3. Generate the public ephemeral key $\mathbf{E}$ where $\mathbf{E} = e \cdot \mathbf{B}$.
    4. Generate the shared public key $\mathbf{S}$ where $\mathbf{S} = e \cdot \mathbf{V} = v \cdot \mathbf{E}$ where $v$ is the secret viewing key of $\mathrm{Out}$ owner.
    5. The encryption key $\mathsf{key} = \mathsf{encode(\mathbf{S})}$
3. Prepare the data to encrypt
    1. Prepare a 49 bytes buffer
    1. Store $\mathsf{salt}$ to the first 16 bytes with big-endian.
    2. Get the 1 byte $\mathsf{tokenId(addr_{token})}$ of $\mathrm{Out}$ and store it to the 17-th byte.
    3. $\mathsf{v}$ is one of $\mathsf{v_{eth}}$ or $\mathsf{v_{erc20}}$ or $\mathsf{v_{nft}}$. Store $\mathsf{v}$ into the 18-th bytes with big-endian.

4. Run encryption using the shared key with [chacha20](https://tools.ietf.org/html/rfc7539)
     $\mathsf{secret} = \mathsf{(salt, tokenId, v)}$
     $\mathsf{ciphertext} \leftarrow \mathsf{chacha20(secret, key)}$

5. Pack
    1. Prepare 81 bytes buffer
    2. Store $\mathsf{encode(\mathbf{E})}$ to the first 32 bytes.
    3. Store the encrypted 49 bytes of cipher text to the 33-rd byte and complete the 81 bytes memo field.
    $$
    \mathsf{memo} = (\mathsf{encode}(\mathbf{E}), \mathsf{ciphertext})
    $$

#### [3.3.2] - Decryption

To receive Zkopru note, recipient should try to decrypt memo field to find own notes.

Let $\mathcal{Out}$ be defined in [3.1.1]
Let $\mathcal{Flow}$ be defined in [3.1.9]
Let $\mathsf{encode()}$ be the point encode function that is defined in [1.13]

1. Parse the first 31 bytes of public ephemeral key data and decode it to $\mathbf{E}$
2. Compute the scalar multiplication with the viewing key $v$ and get the shared public key $\mathbf{S}$.
3. Encode the public shared key $S$ and get the chacha20 cipher $\mathsf{key}$
4. Parse the remaining 49 bytes of the memo field and decrypt the value using $\mathsf{key}$ and get $\mathsf{secret' = (salt', tokenId', v')}$. Please note that we're not sure $\mathsf{secret}'$ is a correctly decrypted one.
5. Fetch registered token addresses from the layer 1 smart contract. And filter them and get addresses which token id is same with $\mathsf{tokenId'}$
6. For each token addresses, try to construct a output note with 3 cases:
    a. $\mathsf{v_{eth} = v', v_{erc20} = 0, v_{nft} = 0}$
    b. $\mathsf{v_{eth} = 0, v_{erc20} = v', v_{nft} = 0}$
    c. $\mathsf{v_{eth} = 0, v_{erc20} = 0, v_{nft} = v'}$
7. Compute the contructed output note's hash value and compare them to $\mathcal{Out}$s.
8. If it succeeds to find the exact same hash, then the decryption suceeds. Or the transaction is not containing a proper memo field for that viewing key owner.

### 3.4 Shielded transaction

#### [3.4.1]

Let $\pi$ be defined in [3.2.5]
Let $\mathcal{P}$ be defined in [3.2.6]
Let $\mathsf{memo}$ be defined in [3.3.1]

Then, the shielded transaction $\mathcal{Tx}$ is

$$
\mathcal{Tx} = (\mathcal{P}, \pi, \mathsf{memo})
$$
where the public signals
$$
\mathcal{P} = (\mathcal{[In_1, ..., In_m], [Out_1, ..., Out_n]}, f, \mathsf{swap})
$$


## 4. Optimistic Roll Up

### 4.1 Layer 1

#### [4.1.1] Storage Variable
Zkopru's optimistic rollup manages the singleton storage variable `chain` which type is
```solidity
struct Blockchain {
    bytes32 genesis;
    bytes32 latest;
    // For coordinating
    uint256 proposedBlocks;
    mapping(address => Proposer) proposers;
    mapping(bytes32 => Proposal) proposals;
    mapping(bytes32 => bool) finalized; // blockhash => finalized?
    mapping(bytes32 => bool) slashed; // blockhash => slashed
    // For inclusion reference
    mapping(bytes32 => bytes32) parentOf; // childBlockHash => parentBlockHash
    mapping(bytes32 => uint256) utxoRootOf; // blockhash => utxoRoot
    mapping(uint256 => bool) finalizedUTXORoots; // all finalized utxo roots
    // For deposit
    MassDeposit stagedDeposits;
    uint256 stagedSize;
    uint256 massDepositId;
    mapping(bytes32 => uint256) committedDeposits;
    // For withdrawal
    mapping(bytes32 => uint256) withdrawalRootOf; // header => withdrawalRoot
    mapping(bytes32 => bool) withdrawn;
    mapping(bytes32 => address) newWithdrawalOwner;
    // For migrations
    mapping(bytes32 => bool) migrationRoots;
    mapping(bytes32 => mapping(bytes32 => bool)) transferredMigrations;
    // For ERC20 and ERC721
    mapping(address => bool) registeredERC20s;
    mapping(address => bool) registeredERC721s;
}

contract Zkopru ... {
    ...
    Blockchain chain;
    ...
}
```

Let's denote the `chain` variable on the address `addr` as `Zkopru(addr).chain`. In addition we will denote the latest block of a Zkopru network on address `addr`  like `Zkopru(addr).chain.latest`

#### [4.1.2] Configurations


| Symbol | Value | Description |
| -------- | -------- | -------- |
| $C_{utxo-tree-depth}$ | 48 | UTXO tree depth. It affects the SNARK proving time. |
| $C_{max-utxo}$ | 281474976710656 | We can use this tree for about 45000 years when we have 100 TPS with 2 output notes for 1 transaction. |
| $C_{withdrawal-tree-depth}$ | 48 | UTXO tree depth. |
| $C_{max-withdrawal}$ | 281474976710656 | We can use this tree for about 90000 years when we have 100 TPS with 1 withdrawal note for 1 transaction.|
| $C_{nullifier-tree-depth}$ | 254 | Nullifier tree's depth is 254. |
| $C_{utxo-sub-tree-depth}$ | 5 | A UTXO subtree's depth is 5.|
| $C_{utxo-sub-tree-size}$ | 32 | A UTXO subtree has 32 leaves. |
| $C_{withdrawal-sub-tree-depth}$ | 5 | A Withdrawal subtree's depth is 5. |
| $C_{withdrawal-sub-tree-size}$ | 32 | A withdrawal subtree has 32 leaves. |
| $C_{max-block-size}$ | 200000 | Block should not be too large. Unit in byte. |
| $C_{max-validation-gas}$ | 6000000 | Block proposer should not submit a block which validation process exceeds the given gas limit |
| $C_{challenge-period}$ | 46253     | Challenge period in block number unit. |
| $C_{minimum-stake}$ | 32e18     | Minimum amount of Ether staking to propose a block. |
| $C_{ref-depth}$ | 128 | Recent $C_{ref-depth}$ UTXO tree roots are available to be used for the UTXO Merkle proof reference. |

Its solidity form looks like:
```solidity
contract Config {
    uint256 public constant UTXO_TREE_DEPTH = 48;
    uint256 public constant MAX_UTXO = (1 << UTXO_TREE_DEPTH);
    uint256 public constant WITHDRAWAL_TREE_DEPTH = 48;
    uint256 public constant MAX_WITHDRAWAL = (1 << WITHDRAWAL_TREE_DEPTH);
    uint256 public constant NULLIFIER_TREE_DEPTH = 254;

    uint256 public constant UTXO_SUB_TREE_DEPTH = 5; // 32 items at once
    uint256 public constant UTXO_SUB_TREE_SIZE = 1 << UTXO_SUB_TREE_DEPTH;
    uint256 public constant WITHDRAWAL_SUB_TREE_DEPTH = 5; // 32 items at once
    uint256 public constant WITHDRAWAL_SUB_TREE_SIZE =
        1 << WITHDRAWAL_SUB_TREE_DEPTH;

    uint256 public MAX_BLOCK_SIZE = 200000; // 3.2M gas for calldata
    uint256 public MAX_VALIDATION_GAS = 6000000; // 6M gas
    // 46523 blocks when the challenge period is 7 days and average block time is 13 sec
    uint256 public CHALLENGE_PERIOD = 46523;
    uint256 public MINIMUM_STAKE = 32 ether;
    uint256 public REF_DEPTH = 128;
}
```

### 4.2 Deposit

#### [4.2.1] - Struct Definition

Let $\mathsf{Tree_\mathsf{utxo}}$ be defined in [5.3.1].

Let's define deposit $\mathrm{D} = (\mathrm{U}, f)$ where $f$ is fee for the block proposer and $\mathrm{U}$ is the UTXO that'll be included in $\mathsf{Tree_\mathsf{utxo}}$ later.

#### [4.2.2] - Mass Deposit

#### [4.2.2.1] - Merging Leaves

Let's define $keccak256_2$ as
$keccak256_2(a, b) = keccak256(c)$
Where $c$ is a 64 bytes data which is the concatenation of two 32 bytes numbers $a$ and $b$ with big-endian.

Zkopru does not store the deposit notes on the contract to minimize the storage gas cost. Instead of storing the deposit notes, it only stores the merged value of all deposits in one storage slot. It could also use a Merkle tree but sequential merging is much gas efficient.

So, let's say we have an array of hashes like $[h_1, h_2, ..., h_n]$.

where 
$\mathsf{merge}([]) = 0$
$\mathsf{merge}([h1]) = keccak256_2(0, h_1)$
$\mathsf{merge}([h_1, h_2]) = keccak256_2(h_1, h_2)$
$\mathsf{merge}([h_1, h_2, h_3]) = keccak256_2(\mathsf{merge}([h_1, h_2]), h_3)$
$\mathsf{merge}([h_1, h_2, ..., h_n])
= keccak256_2(\mathsf{merge}([h_1, h_2, ..., h_{n-1}]), h_{n})$

So we can express this $\mathsf{merge}$ function as an recursive form like,

$$
\mathsf{merge}([h_1, h_2, ..., h_n])
= \mathsf{merge}([\mathsf{merge}([h_1, h_2, ..., h_{n-1}]), h_{n}])
$$

#### [4.2.2.2] - Staged Deposits

Smart contract has one staged deposits that can be a Mass Deposit. Every `deposit()` function will update the staged deposits and will merge the deposit data into it.

Here is how a deposit $\mathrm{D}$ by `deposit()` updates the staged deposits $\mathrm{MD}_{stage}$.

Let $[\mathrm{D_1},\mathrm{D_2}, ...,\mathrm{D_n}]$ are appended to the staged deposits.
Let $\mathsf{merge()}$ be defined in [4.2.2.1].
Let $f_i$ is the deposit fee for $\mathrm{D}_i$.

Then,

$$
\mathrm{MD}_{stage} = (\mathsf{merged}, f_{MD})
$$

where 

$$
\mathsf{merged} = \mathsf{merge}([\mathsf{hash}(\mathrm{D_1}), \mathsf{hash}(\mathrm{D_2}), ..., \mathsf{hash}(\mathrm{D_n})]) \\
f_{MD} = \Sigma_{i=0}^{n} f_i
$$

Finally, the `deposit()` transaction transfers assets and stores the updated $\mathrm{MD}_{stage}$ to `Zkopru.chain.stagedDeposits`.

#### [4.2.2.3] - Commitment of Mass Deposit

Anyone can commit the staged deposits and create a Mass Deposit from it by calling `commitMassDeposit()`. Then, the layer 1 contract records it as committed and starts a new empty staged deposit object.


Let $[\mathrm{D_1},\mathrm{D_2}, ...,\mathrm{D_n}]$ construct a mass deposit $\mathrm{MD}$.

Then, the mass deposit $\mathrm{MD}$ is expressed with
$$
\mathrm{MD} = (\mathsf{merged}_\mathrm{MD}, f_\mathrm{MD})
$$

where

$$
\mathsf{merged}_\mathrm{MD} = \mathsf{merge}([\mathsf{hash}(\mathrm{D_1}), \mathsf{hash}(\mathrm{D_2}), ..., \mathsf{hash}(\mathrm{D_n})])
$$

Finally, `commitMassDeposit()` stores $\mathrm{MD}$ to the `Zkopru.chain.committedDeposits` setting the key with its hash value.

The hash of a mass deposit is
$$
\mathsf{hash}(\mathrm{MD}) = keccak256(\mathsf{encodePacked}(\mathsf{merged}_\mathrm{MD}, f_\mathrm{MD})).
$$

### 4.3 Withdrawal

To withdraw $\mathrm{W}$ out of the Zkopru network, there should be an existence proof and the ownership proof.

#### 4.3.1. Withdrawal Note Existence Proof

#### [4.3.1.1] - Withdrawal Note Position

Let $\mathcal{W}$ be defined in [3.1.3].
Let $\mathsf{Tree_\mathsf{withdrawal}}$ be defined in [5.3.2].

$\mathsf{pos}(\mathcal{W})$ is the leaf index of $\mathcal{W}$ in the Withdrawal Merkle tree $\mathsf{Tree}_\mathsf{withdrawal}$.

#### [4.3.1.2] - Merkle Proof of Withdrawal Note

Let $\mathsf{hash}(\mathcal{W})$ be defined in [3.1.3].
Let $\mathsf{pos}(\mathcal{W})$ be defined in [4.3.1.1].
Let $\mathsf{Tree_\mathsf{withdrawal}}$ be defined in [5.3.2].
Let $\mathsf{root_{withdrawal}}^{(n)}$ be defined in [6.1.2].

Let the root of $\mathsf{Tree}_\mathsf{withdrawal}$ at the $n$-th layer 2 block be denoted $\mathsf{root_{withdrawal}}^{(n)}$.
$\mathsf{Sib}_\mathsf{withdrawal}^{(n)}(\mathcal{W})$ is the set of all sibling node data of Withdrawal $\mathcal{W}$ at the $n$-th layer 2 block.

Then the block $\mathsf{Block}^{(n)}$ and its corresponding $\mathsf{root_{withdrawal}}^{(n)}$ should be recorded as finalized on the smart contract by the optimistic rollup consensus.

Then, `withdraw()` transaction should include a Merkle Proof that proves
$$
\mathsf{Merkle}^{(n)}_\mathsf{withdrawal}(\mathcal{W}) = (\mathsf{hash}(\mathcal{W}), \mathsf{pos}(\mathcal{W}), \mathsf{root_{withdrawal}}^{(n)}, \mathsf{Sib}_\mathcal{W}^{(n)}(\mathcal{W}))
$$

#### [4.3.1.3] - Caller Fee

Let $\mathsf{N}, \mathcal{N}$ be defined in [3.1.1]
Let $\mathcal{W}$ be defined in [3.1.3]

Layer 1 transaction `withdraw()` should include $\mathsf{hash(N)}$ and $\mathcal{N}$ that satisfy $\mathsf{hash(\mathcal{W})}$.

Then, the submitted $\mathsf{fee_{caller}}$ amount of Ether goes to the `withdraw()` transaction executor.

#### [4.3.1.4] - Pay In Advance

As the user should wait the finalization, $\mathcal{W}$ owner can request an instant withdrawal to prepayers who are willing to pay the fund in advance to earn fee.

For the instant withdrawal with pay in advance feature,

1. $\mathcal{W}$ owner select a prepayer and generate a message that follows the EIP712 spec with the following structure.
    ```
	struct PrepayRequest {
	    address prepayer;
	    bytes32 withdrawalHash;
	    uint256 prepayFeeInEth;
	    uint256 prepayFeeInToken;
	    uint256 expiration;
	}
    ```
   Since the $\mathcal{W}$ might be a ERC20 only withdrawal note, the owner can choose how to pay the fee for instant withdrawal to the prepayer.
2. Generate a ECDSA with the correct account which address is $\mathcal{N}.\mathsf{to}$ and send a request to the prepayer using a communication channels like HTTP.
3. If the request look profitable, the prepayer verifies the validity calls the `payInAdvance()` function with the received ECDSA signature and correct amount of assets to pay in advance for the original owner.
4. Smart contract verifies the relationship between $\mathsf{hash(\mathcal{W})}$, ECDSA, transaction signer and the expiration timestamp.
    * Computed withdrawal hash of the given details equals to `withdrawalHash`.
    * ECDSA satisfy EIP712 sign spec with its own domain using chain Id and contract address.
    * Current block timestamp is smaller than the given expiration.
5. If it passes all verifications, it records the transferred ownership of the $\mathcal{W}$ and it transfers assets to the original owner.

### 4.4 Migration

#### [4.4.1] Mass Migration Construction

Let $\mathcal{M}$ be defined in [3.1.4]
Let $M$ be the set of all migration type transaction outputs of a block.
Let $A_\mathsf{dest}$ be the set of all destination addresses of the migration type transaction outputs of the block.
Let $A_\mathsf{token}$ be the set of all token addresses that exist in the migration outputs.
Let $\mathsf{merge()}$ be defined in [4.2.2.1]

Then, for each $\mathsf{dest} \in A_\mathsf{dest}$, $\mathsf{token} \in A_\mathsf{token}$ we can construct a set of migrations with
$M_{\mathsf{dest}} =\{ \mathcal{M} | \mathcal{M}.\mathsf{to} = \mathsf{dest} \land \mathcal{M}.\mathsf{token} = \mathsf{token} \} = \{\mathcal{M_1, M_2, ..., M_k}\}$

Let $ftr$ be defined in [3.2.4.6.2]

Then, for each $(\mathsf{dest}, \mathsf{token})$ pair where $\mathsf{dest} \in A_\mathsf{dest}$ and $\mathsf{token} \in A_\mathsf{token}$ has its corresponding mass migration $\mathrm{MM}$ defined as

$\mathrm{MM} = (\mathsf{dest}, \mathsf{asset_{migration}}, \mathrm{MD}_\mathsf{migration})$

where
* $\mathsf{asset_{migration}} = (\mathsf{eth, token, amount})$
* $\mathrm{MD}_\mathsf{migration} = (\mathsf{merge(\mathcal{[M_1, ..., M_k]})}, \Sigma_{i=0}^{k}\mathcal{M}.\mathsf{fee_{migration}})$

#### [4.4.2] Destination & Token address pair

Let a block contain mass migrations $[\mathrm{MM}_1, ..., \mathrm{MM}_n]$
and $(\mathsf{dest}, \mathsf{token})_i = (\mathrm{MM}_i.\mathsf{dest}, \mathrm{MM}_i.\mathsf{token})$

Then $(\mathsf{dest}, \mathsf{token})_i \neq (\mathsf{dest}, \mathsf{token})_j$ when $i \neq j$.

### 4.5 Proposal & Finalization

#### [4.5.1] Getting Proposal Rights

Zkopru can have various types of proposer selection logic. And it can be simply fetched by calling `isProposable(address)` function which type is
```solidity
function isProposable(address proposer) pure returns (bool);
```

This function asks the proposability of the given address to the `ConsensusProvider` which default is the "BurnAuction" for now.

#### [4.5.2] Stake

To propose a block, the coordinator should have staked more than 32 ETH in the contract. Coordinator can stake ETH by calling `register()`.

#### [4.5.3] Propose

Let $C_{max-block-size}$ be defined in [4.1.2].

Once the coordinator has proposer amount of stakes, proposer can submit a serialized form of $\mathtt{Block}$ as defined in [6.7.1] by calling `propose(bytes calldata)` function.

To call the function `propose()`, it requires
* The `msg.sender` should have staked more than 32 ETH.
* $len(\mathtt{Block}^{(n)}) < C_{max-block-size}$.
* $\mathsf{Block}^{(n)}.\mathsf{proposer}$ equals to the `msg.sender`.
* There is no duplicated proposal that has same block proposal checksum which is defined in [6.7.2].

Once the function `propose()` is called,

* It records the block hash by chaining with its parent hash value.
* It saves the proposal checksum for its future challenge.
* It records the utxo root for the utxo inclusion proof reference.
* It records the withdrawal root for the withdrawal proof of [4.3.1.2].
* It extends the `Zkopru.chain.proposers.exitAllowance` to its challenge due block number that is `block.number` +$C_{challenge-preiod}$.
* And commit the latest staged deposits.

#### [4.5.4] Exit

Coordinators can withdraw their staked ETH whenever `Zkopru.chain.proposers.exitAllowance` is behind the block number.

#### [4.5.5] Finalization

Let $\mathtt{Finalization}^{(n)}$ be defined in [6.7.2].

Anyone can call the `finalize(bytes calldata)` function by submitting the finalization data $\mathtt{Finalization}^{(n)}$.

Using the $\mathsf{checksum}$ of the proposal parsed from $\mathtt{Finalization}^{(n)}$ retrieve the proposal object `proposal` from `Zkopru.chain.proposals[checksum]`.

To finalize a block, it requires

* $\mathsf{depositRoot}^{(n)}$ should equal to the hash of the submitted $\mathtt{MDs}^{(n)}$.
* $\mathsf{migrationRoot}^{(n)}$ should equal to the hash of the submitted $\mathtt{MMs}^{(n)}$.
* $\mathsf{hash}(\mathsf{Header}^{(n)})$ should equal to the stored hash in `proposal`.
* `proposal` should not be slashed.
* `proposal` should not be finalized.
* `Zkopru.chain.latest` should equal to the $\mathsf{parent}$ of $\mathsf{Header}^{(n)}$.
* `proposal.challengeDue` should be behind `block.number`.
* Every $\mathtt{MD}$ should be committed in the `Zkopru.chain.committedDeposits`.
* $\mathsf{migrationRoot}^{(n)}$ should not exist in the `Zkopru.chain.migrationRoots`.

Once the function `finalize()` is called, 
* It removes the every $\mathtt{MD}$ from `Zkopru.chain.committedDeposits`.
* It marks $\mathsf{migrationRoot}^{(n)}$ as true in `Zkopru.chain.migrationRoots`. It allows `migrateFrom()` function call in [4.5.6].
* It gives the $\mathsf{fee}^{(n)}$ to the proposer.
* It marks the block header hash as finalized in `Zkopru.chain.finalized`
* It marks the utxo root as finalized in `Zkopru.chain.finalizedUTXORoots`
* It updates the latest block hash `latest`.
* It deletes `proposal`.

#### [4.5.6] Finalization & `migrateFrom`

Let $\mathrm{MM}$ be defined in [4.4.1]
Let $\mathsf{migrationRoot}^{(n)}$ be defined in [4.5.5] and $\mathrm{MM}^{(n)}_i$ is one of migrations in that Merkle tree.

After $\mathsf{migrationRoot}^{(n)}$ is finalized on the contract, the destination network of $\mathrm{MM}^{(n)}_i$ can call `migrateFrom` function to migrate $\mathsf{asset}_\mathsf{migration}$ with its MerkleProof.

Then, it records $(\mathsf{migrationRoot}^{(n)}, \mathsf{hash}(\mathrm{MM}^{(n)}_i))$ as transferred in `Zkopru(source).chain.transferredMigratios`. Simultaneously, it transfers the given ETH and tokens to the $\mathsf{dest}$ network adding $\mathrm{MD}_\mathsf{migration}$ to `Zkopru(dest).chain.committedDeposits`.

### 4.6 Token Registration

#### [4.6.1] Token Id

Zkopru transaction's encrypted memo field uses token id instead of its full address to reduce the data size.

If the token address is $a$, $\mathsf{tokenId(a)} = a \ mod \ 256$.

#### [4.6.2] ERC20 Token Registration

Anyone can register any kind of ERC20 Token that follows the standard interface. Once the testing transaction is succeed, Zkopru contract will register the token address into the `Zkopru.chain.registeredERC20s`.

#### [4.6.3] ERC721 Token Registration

Anyone also can register any kind of ERC721 Token that implements ERC165 standard. If the token contract's ERC165 interface returns true against the query for ERC721 support, Zkopru contract will register the token address into the `Zkopru.chain.registeredERC721s`.

### 4.7 Validations

Let's assume that a proposer submitted $\mathsf{Block}^{(n)}$.

#### [4.7.1] Deposit Validation - D1: Mass Deposit is not Committed

Let $\mathrm{MD}$ is one of the submitted mass deposit in $\mathsf{Block}^{(n)}$.

Then, if any of the $\mathsf{hash}(\mathrm{MD})$ does not exists in `Zkopru.chain.committedDeposits`, the validation contract returns `slashable = true` with code D1.

#### [4.7.2.1] Header Validation - H1: Deposit Root

Let $\mathsf{MerkleRoot}$ be defined in [7.1]
Let $\mathtt{MDs}$ is the array of submitted mass deposits in $\mathsf{Block}^{(n)}$.
Let $\mathsf{depositRoot}^{(n)}$ be defined in [6.2.1]

When $\mathsf{MerkleRoot}_{keccak256}(\mathtt{MDs})$ does not equal to $\mathsf{depositRoot}^{(n)}$, the validation contract returns `slashable = true` with code H1.

#### [4.7.2.2] Header Validation - H2: Transaction Root

Let $\mathsf{MerkleRoot}$ be defined in [7.1]
Let $\mathtt{TXs}$ is the array of submitted transactions in $\mathsf{Block}^{(n)}$.
Let $\mathsf{txRoot}^{(n)}$ be defined in [6.2.1]

When $\mathsf{MerkleRoot}_{keccak256}(\mathtt{TXs})$ does not equal to $\mathsf{txRoot}^{(n)}$, the validation contract returns `slashable = true` with code H2.

#### [4.7.2.3] Header Validation - H3: Migration Root

Let $\mathsf{MerkleRoot}$ be defined in [7.1]
Let $\mathtt{MMs}$ is the array of submitted mass migrations in $\mathsf{Block}^{(n)}$.
Let $\mathsf{migrationRoot}^{(n)}$ be defined in [6.2.1]

When $\mathsf{MerkleRoot}_{keccak256}(\mathtt{MMs})$ does not equal to $\mathsf{migrationRoot}^{(n)}$, the validation contract returns `slashable = true` with code H3.


#### [4.7.2.4] Header Validation - H4: Total Fee

The total fee for block proposer should equal to

$$
\mathsf{fee}^{(n)} = \Sigma_{i=0}^{n_{tx}} \mathcal{Tx}_i.\mathcal{P}.f + \Sigma_{i=0}^{n_{md}} \mathrm{MD}_i.f_{MD}
$$

Or validation contract returns `slashable = true` with code H4.

#### [4.7.2.5] Header Validation - H5: Parent Block

Then, $\mathsf{parent}^{(n)}$ should not be a slashed block. So if `Zkopru.chain.slashed[parent]` exists, the validation contract returns `slashable = true` with code H5.

#### [4.7.3.1] Migration Validation - M1: Duplicated MassMigration

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If it does not satisfy the condition [4.4.2] that every migration should have different destination, the validation contract returns `slashable = true` with code M1.

#### [4.7.3.2] Migration Validation - M2: MassMigration is carrying invalid amount of ETH.

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If it does not satisfy the following condition,
$$
\mathrm{MM}.\mathsf{asset_{migration}}.\mathsf{eth} = \Sigma \mathcal{M}.\mathsf{v_{eth}}
$$

If the sum of total ETH does not equal to the $\mathsf{eth}$ defined in the migrating asset, and the validation contract returns `slashable = true` with code M2.

#### [4.7.3.3] Migration Validation - M3: MassMigration is carrying invalid amount of Token.

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If it does not satisfy the following condition,
$$
\mathrm{MM}.\mathsf{asset_{migration}}.\mathsf{amount} = \Sigma \mathcal{M}.\mathsf{v_{erc20}}
$$
If the sum of total token amount does not equal to the $\mathsf{amount}$ defined in the migrating asset, and the validation contract returns `slashable = true` with code M3.

#### [4.7.3.4] Migration Validation - M4: Merged Leaves

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If the mass deposit fot the destination does not equal to the on-chain computed mass deposit
$$
\mathrm{MD}_\mathsf{migration} = (\mathsf{merge(\mathcal{[M_1, ..., M_k]})}, \Sigma_{i=0}^{k}\mathcal{M}.\mathsf{fee_{migration}})
$$

the validation contract returns `slashable = true` with code M4.

#### [4.7.3.5] Migration Validation - M5: Migration Fee

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If it does not satisfy the $\mathsf{fee}$ condition in [4.4.1]
$$
\mathrm{MD} = (\mathsf{merge(\mathcal{[M_1, ..., M_k]})}, \Sigma_{i=0}^{k}\mathcal{M}.\mathsf{fee_{migration}})
$$

the validation contract returns `slashable = true` with code M5.

#### [4.7.3.6] Only registered ERC20 tokens are supported for mass migration

Let $\mathrm{MM}$ is one of the submitted mass migration in $\mathsf{Block}^{(n)}$.

If $\mathcal{M}.\mathsf{token}$a does not exists in the contract storage `Zkopru.chain.registeredERC20s` it returns `slashable = true` with code M6.

#### [4.7.3.7] Migration Validation - M7: Missing Migration

For every $\mathcal{Tx}$, a mass migration should exists in the $\mathrm{MM}[]$ list that correspondes its destination and carrying token address. If the block body misses any destination or token, the validation contract returns `slashable = true` with code M7.

#### [4.7.4] Nullifier Validation - N1: Nullifier root is not correct

Let $\mathtt{TXs}^{(n)}$ and $\mathtt{Tx}^{(n)}_i$ be defined in [6.6.5]
Let $\mathcal{Tx}^{(n)}_i$ be the deserialized form of $\mathtt{TX}^{(n)}_i$.
Let $\mathsf{Tree_{nullifier}}^{(n)}$ be defined in [5.3.3].

Let $\mathsf{nullifiers}^{(n)} = \{\mathcal{Tx_i^{(n)}.P.In_j.\mathsf{nullifier}} | 1 \leq i \leq n_{tx},  1 \leq j \leq 4 \}$

Then, for every $\mathsf{nullifier} \in \mathsf{nullifiers}^{(n)}$, should not exist in the $\mathsf{Tree}_{nullifier}$.

And appending all $\mathsf{nullifier}$ to $\mathsf{Tree_{nullifier}}^{(n-1)}$, its updated root should equal to  $\mathsf{root}^{(n)}_\mathsf{nullifier}$.

Otherwise, the validation contract returns `slashable = true` with code N1.

#### [4.7.5] Utxo Tree Validation

Let $\mathtt{TXs}^{(n)}$ and $\mathtt{Tx}^{(n)}_i$ be defined in [6.6.5]
Let $\mathtt{MDs}^{(n)}$ and $\mathrm{MD}^{(n)}_i$ be defined in [6.6.5]
Let $\mathsf{Tree_{utxo}}^{(n)}$ be defined in [5.3.3].
Let $C_{utxo-sub-tree-size}$ be defined in [4.1.2].

Let $\mathrm{MD}^{(n)}_i = [\mathrm{D}_{i_1}, \mathrm{D}_{i_2}, ..., \mathrm{D}_{i_{n_i}}]$.

Then the list of deposit notes becomes $\mathsf{utxos}_{deposits}^{(n)} = [\underbrace{\mathrm{D}_{1_1}, \mathrm{D}_{1_2}, ..., \mathrm{D}_{1_{n_1}}}_{\mathrm{MD}_1}, \underbrace{\mathrm{D}_{2_1}, \mathrm{D}_{2_2}, ..., \mathrm{D}_{2_{n_2}}}_{\mathrm{MD}_2}, ..., \underbrace{\mathrm{D}_{k_1}, \mathrm{D}_{k_2}, ..., \mathrm{D}_{k_{n_k}}}_{\mathrm{MD}_{n_{md}=k}}]$

Let $\mathrm{U}_{i_j} = \mathcal{Tx_i^{(n)}.P.Out_j}$.

Then the list of deposit notes becomes $\mathsf{utxos}_{txs}^{(n)} = [\underbrace{\mathrm{O}_{1_1}, \mathrm{O}_{1_2}, ..., \mathrm{O}_{1_{n_1}}}_{\mathcal{Tx}_1}, \underbrace{\mathrm{O}_{2_1}, \mathrm{O}_{2_2}, ..., \mathrm{O}_{2_{n_2}}}_{\mathcal{Tx}_2}, ..., \underbrace{\mathrm{O}_{l_1}, \mathrm{O}_{l_2}, ..., \mathrm{O}_{l_{n_l}}}_{\mathcal{Tx}_{n_{tx} = l}}]$

Then the total list of all utxos becomes

$$
\mathsf{utxos}^{(n)} = [\underbrace{\mathrm{D}_{1_1}, \mathrm{D}_{1_2}, ..., \mathrm{D}_{k_{n_k}}}_{\mathsf{utxos}^{(n)}_{deposits}}, \underbrace{\mathrm{O}_{1_1}, \mathrm{O}_{1_2}, ..., \mathrm{O}_{l_{n_l}}}_{\mathsf{utxos}^{(n)}_{txs}}, \underbrace{0, 0, ..., 0}_\text{padded zeroes}]
$$

Here, the padded zeroes are added to make the length of $\mathsf{utxos}^{(n)}$ be the multiple of $C_{utxo-sub-tree-size}$.

Therefore
$$
len(\mathsf{utxos}^{(n)}) \ mod \ C_{utxo-sub-tree-size} = 0
$$

#### [4.7.5.1] UTXO Tree Validation - U1: Index Validation

Let $\mathsf{utxos}^{(n)}$ be defined in [4.7.5].
Let $\mathsf{index_{utxos}}^{(n)}$ be defined in [6.2.1].

Then,
$$
len(\mathsf{utxos}^{(n)}) + \mathsf{index_{utxos}}^{(n-1)} = \mathsf{index_{utxos}}^{(n)}
$$

Otherwise, the validation contract returns `slashable = true` with code U1.

#### [4.7.5.2] UTXO Tree Validation - U2: Max UTXOs

Let $\mathsf{utxos}^{(n)}$ be defined in [4.7.5].
Let $\mathsf{index_{utxo}}^{(n)}$ be defined in [6.2.1].
Let $C_{max-utxo}$ be defined in [4.1.2].

Then,
$$
len(\mathsf{utxos}^{(n)}) + \mathsf{index_{utxos}}^{(n-1)} \leq C_{max-utxo}
$$

Otherwise, the validation contract returns `slashable = true` with code U2.

#### [4.7.5.3] UTXO Tree Validation - U3: UTXO Tree Root

Let $\mathsf{Tree_{utxo}}^{(n)}$ be defined in [5.3.1].
Let $\mathsf{utxos}^{(n)}$ be defined in [4.7.5].
Let $\mathsf{root_{utxo}}^{(n)}$ be defined in [6.2.1].
Let $C_{max-utxo}$ be defined in [4.1.2].

Then, appending the hash of each items in $\mathsf{utxos}^{(n)}$ to $\mathsf{Tree_{utxo}}^{(n-1)}$ should have an updated root that equals to $\mathsf{root_{utxo}}^{(n)}$.

Otherwise, the validation contract returns `slashable = true` with code U3.

#### [4.7.6] Withdrawal Tree Validation

Let $\mathtt{TXs}^{(n)}$ and $\mathtt{Tx}^{(n)}_i$ be defined in [6.6.5]
Let $\mathsf{Tree_{withdrawal}}^{(n)}$ be defined in [5.3.2].
Let $C_{withdrawal-sub-tree-size}$ be defined in [4.1.2].

Let $\mathcal{W}_{i_j} = \mathcal{Tx_i^{(n)}.P.Out_j}$ only if $\mathcal{Tx_i^{(n)}.P.Out_j}.t = 1$.

Then the list of deposit notes becomes $\mathsf{withdrawals}^{(n)} = [\underbrace{\mathcal{W}_{1_1}, \mathcal{W}_{1_2}, ..., \mathcal{W}_{1_{n_1}}}_{\mathcal{Tx}_1}, \underbrace{\mathcal{W}_{2_1}, \mathcal{W}_{2_2}, ..., \mathcal{W}_{2_{n_2}}}_{\mathcal{Tx}_2}, ..., \underbrace{\mathcal{W}_{k_1}, \mathcal{W}_{k_2}, ..., \mathcal{W}_{k_{n_k}}}_{\mathcal{Tx}_{n_{tx} = k}}, \underbrace{0, 0, ..., 0}_\text{padded zeroes}]$

Here, the padded zeroes are added to make the length of $\mathsf{withdrawals}^{(n)}$ be the multiple of $C_{withdrawal-sub-tree-size}$.

Therefore
$$
len(\mathsf{withdrawals}^{(n)}) \ mod \ C_{withdrawal-sub-tree-size} = 0
$$

#### [4.7.6.1] Withdrawal Tree Validation - W1: Index Validation

Let $\mathsf{withdrawals}^{(n)}$ be defined in [4.7.6].
Let $\mathsf{index_{withdrawals}}^{(n)}$ be defined in [6.2.1].

Then,
$$
len(\mathsf{withdrawals}^{(n)}) + \mathsf{index_{withdrawals}}^{(n-1)} = \mathsf{index_{withdrawals}}^{(n)}
$$

Otherwise, the validation contract returns `slashable = true` with code W1.

#### [4.7.6.2] Withdrawal Tree Validation - W2: Max withdrawals

Let $\mathsf{withdrawals}^{(n)}$ be defined in [4.7.6].
Let $\mathsf{index_{withdrawal}}^{(n)}$ be defined in [6.2.1].
Let $C_{max-withdrawal}$ be defined in [4.1.2].

Then,
$$
len(\mathsf{withdrawals}^{(n)}) + \mathsf{index_{withdrawals}}^{(n-1)} \leq C_{max-withdrawal}
$$

Otherwise, the validation contract returns `slashable = true` with code W2.

#### [4.7.6.3] Withdrawal Tree Validation - W3: withdrawal Tree Root

Let $\mathsf{Tree_{withdrawal}}^{(n)}$ be defined in [5.3.2].
Let $\mathsf{withdrawals}^{(n)}$ be defined in [4.7.6].
Let $\mathsf{root_{withdrawal}}^{(n)}$ be defined in [6.2.1].
Let $C_{max-withdrawal}$ be defined in [4.1.2].
Let $\mathsf{hash}(\mathcal{W})$ be defined in [3.1.3].

Then, appending the withdrawal hash of each items in $\mathsf{withdrawals}^{(n)}$ to $\mathsf{Tree_{withdrawal}}^{(n-1)}$ should have an updated root that equals to $\mathsf{root_{withdrawal}}^{(n)}$.

Otherwise, the validation contract returns `slashable = true` with code W3.

#### [4.7.7] Transaction Validation

#### [4.7.7.1] Transaction Validation - T1: Inclusion Proof

Let $C_{ref-depth}$ be defined in [4.1.2].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, In_2, ...], [Out_1, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then for every $\mathcal{In} = (\mathsf{nullifier(\mathrm{U})}, \mathsf{root_{utxo}}^{(k)})$, it should satisfy 
$$
n - C_{ref-depth} \leq k
$$
or
$\mathsf{root_{utxo}}^{(k)}$ should be stored in the `Zkopru.chain.finalizedUTXORoots`.

Otherwise, the validation contract returns `slashable = true` with code T1.

#### [4.7.7.2] Transaction Validation - T2: Outflow Type

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then for every $\mathcal{Out}$, 

$$
0 \leq t \leq 2
$$

Otherwise, the validation contract returns `slashable = true` with code T2.

#### [4.7.7.3] Transaction Validation - T3: Outflow Public Data

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then, for every $\mathcal{Out}$,
$\mathcal{N}$ should be $(0, 0, 0, 0, 0, 0)$ if and only if $t = 0$.

Otherwise, the validation contract returns `slashable = true` with code T3.

#### [4.7.7.4] Transaction Validation - T4: Not a Registered Token

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

For every $\mathcal{Out}$,
$\mathcal{N}.\mathsf{addr_{token}}$ should be registered on `Zkopru.chain.registeredERC20s` or `Zkopru.chain.registeredERC721s`.

Otherwise, the validation contract returns `slashable = true` with code T4.

#### [4.7.7.5] Transaction Validation - T5: NFT field is not empty

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

For every $\mathcal{Out}$,
$\mathcal{N}.\mathsf{v'_{nft}}$ should be zero if $\mathcal{N}.\mathsf{addr_{token}}$ is not registered as an ERC721 on `Zkopru.chain.registeredERC721s`.

Otherwise, the validation contract returns `slashable = true` with code T5.

#### [4.7.7.6] Transaction Validation - T6: ERC20 field is not empty

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

For every $\mathcal{Out}$,
$\mathsf{v'_{erc20}}$ should be zero if $\mathcal{N}.\mathsf{addr_{token}}$ is not registered as an ERC20 on `Zkopru.chain.registeredERC20s`.

Otherwise, the validation contract returns `slashable = true` with code T6.

#### [4.7.7.7] Transaction Validation - T7: NFT id 0 is not allowed

Let $\mathcal{Out} = (\mathsf{hash(N)}, t, \mathcal{N})$ as defined in [3.1.1].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

For every $\mathcal{Out}$,
$\mathcal{N}.\mathsf{v'_{nft}}$ should not be zero if $\mathcal{N}.\mathsf{addr_{token}}$ is registered as an ERC721 on `Zkopru.chain.registeredERC721s`.

Otherwise, the validation contract returns `slashable = true` with code T7.

This is because the SNARK circuit is designed not to support NFT id 0 by its technical limitation.

#### [4.7.7.8] Transaction Validation - T8: Atomic Swap Pair Not Exists

Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}_i), \pi_i, \mathsf{memo}_i)$ as defined in [3.4.1].

If $\mathsf{swap}_i$ is not zero, there should exist another $\mathcal{Tx}^{(n)}_j$ that has $\mathsf{swap}_i$ for one of its outputs while $\mathsf{swap}_j$ is one of $\mathcal{Tx}^{(n)}_i$'s output.

If there does not exist correct pair, the validation contract returns `slashable = true` with code T8.

#### [4.7.7.9] Transaction Validation - T9: Used Nullifiers

Let $\mathcal{In}$ be defined in [3.1.7].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then for every $\mathcal{In} = (\mathsf{nullifier(\mathrm{U})}, \mathsf{root_{utxo}}^{(k)})$, appending $\mathsf{nullifier(\mathrm{U})}$ to $\mathsf{Tree_{nullifier}}^{(n-1)}$ should not update the tree.

Otherwise, it is considered as a used one and the validation contract returns `slashable = true` with code T9.

#### [4.7.7.10] Transaction Validation - T10: Duplicated Nullifiers

Let $\mathcal{In}$ be defined in [3.1.7].
Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ...], [Out_1, Out_2, ...]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then for every $\mathcal{In} = (\mathsf{nullifier(\mathrm{U})}, \mathsf{root_{utxo}}^{(k)})$, $\mathsf{nullifier(\mathrm{U})}$ should be unique within the whole nullifiers of other transactions in the same block.

Otherwise, it is considered as a used one and the validation contract returns `slashable = true` with code T10.

#### [4.7.7.11] Transaction Validation - S1: Not a supported circuit

Let $\mathcal{Tx}^{(n)}_i = ((\mathcal{[In_1, ..., In_x], [Out_1, ..., Out_y]}, f, \mathsf{swap}), \pi, \mathsf{memo})$ as defined in [3.4.1].

Then, verifyig key for circuit $\mathsf{C}_{(x,y)}$ should exists on `Zkopru.vks`.

Otherwise, the validation contract returns `slashable = true` with code S1.

#### [4.7.7.12] Transaction Validation - S2: SNARK fails

Let $\mathcal{Tx}^{(n)}_i = (\mathcal{P}, \pi, \mathsf{memo})$ as defined in [3.4.1].
Let $\mathsf{zVK}_{(x,y)}$ be the verifying key for circuit $\mathsf{C}_{(x,y)}$.

Then,
$$
\mathsf{verify_{groth16}}(\mathcal{P}, \pi, \mathsf{vPK}_{(x,y)}) = 1
$$

Otherwise, the validation contract returns `slashable = true` with code S2.

#### [4.7.7.13] Transaction Validation - S3: Range Check fails

Let $p$ be defined in [1.1].
Let $\mathcal{Tx}^{(n)}_i = (\mathcal{P}, \pi, \mathsf{memo})$ as defined in [3.4.1].

Then, every value of $\mathcal{P}$ should be less than $p$.

Otherwise, the validation contract returns `slashable = true` with code S3.

## 5. Tree

### 5.1 Sparse Merkle Tree

#### [5.1.1] Tree definition

Sparse Merkle Tree is a fixed depth Merkle tree that all leaves have a defined initial value. It is defined with $\mathsf{SMT  = (depth, hash, null)}$.

#### [5.1.2] Structure

Let $\mathsf{SMT}$ be defined in [5.1.1].

Then, the tree has $\mathsf{depth} + 1$ number of layers. For example when the $\mathsf{depth} = 3$,
```
Depth 0 (Level 3):               d
Depth 1 (Level 2):       c-------^-------c
Depth 2 (Level 1):   b---^---b       b---^---b   
Depth 3 (Level 0): a-^-a   a-^-a   a-^-a   a-^-a
```

And it can include $2^\mathsf{depth}$ of items.

#### [5.1.3] Node index

Let $\mathsf{SMT}$ be defined in [5.1.1].

Then, it has $2^{\mathsf{depth} + 1} - 1$ tree nodes and each node has its own index and value.

$$
\mathsf{node = (index, value)}
$$

Index starts from the root node with value 1. After then, every left child node's index is the double of its parent's index and the right child node's index is plus one of its sibling left node.

If we express the index in binary format the index map looks like below when the depth is 3:

```
Depth 0 (Level 3):                               (1)
Depth 1 (Level 2):               (10)-------------^-------------(11)
Depth 2 (Level 1):      (100)-----^-----(101)           (110)-----^-----(111)
Depth 3 (Level 0): (1000)-^-(1001) (1010)-^-(1011) (1100)-^-(1101) (1110)-^-(1111)
```


#### [5.1.4] Tree Node

Merkle tree has three types of node.

* Leaf node
* Branch node
* Root node

#### [5.1.4.1] Leaf node

Let $\mathsf{depth, null}$ be defined in [5.1.1].

Every leaf node has no child and its initial value is $\mathsf{null}$.
Then index of leaf node should be greater or equal than $2^{\mathsf{depth}}$ and less than $2^{\mathsf{depth} + 1}$.

As $\mathsf{SMT}$ usually stores items into the leaf nodes, leaf nodes also have $\mathsf{pos}$ value that is 
$$
\mathsf{pos} = \mathsf{node.index} - 2^\mathsf{depth}
$$

$\mathsf{pos}$ starts from 0 and less than $2^\mathsf{depth}$

#### [5.1.4.2] Branch node

Let $\mathsf{hash}$ be defined in [5.1.1].

Every node except leaf node is kind of the branch node, and their value is decided by the values of their children nodes.

Let $\mathsf{parent}$ be a branch node which has $\mathsf{left, right}$ for its children nodes.

Then,

$$
\mathsf{parent.value} = \mathsf{hash(left.value, right.value)}
$$


Then index of leaf node should be greater or equal than 1 and less than $2^{\mathsf{depth}}$

#### [5.1.4.3] Root node

Root node is also a branch node which index is 1. The value of the root node is a compressed state of the tree.

#### [5.1.5] Item & Merkle Root

Let $\mathsf{SMT, hash, depth}$ be defined in [5.1.1].
Let $\mathsf{node}$ be defined in [5.1.3].
Let $\mathsf{pos}$ be defined in [5.1.4.1].
Let $\mathsf{root}$ is the value of the root node of the $\mathsf{SMT}$.

Let $\mathsf{item}$ be defined as
$$
\mathsf{item} = (\mathsf{pos}, \mathsf{value})
$$


Then, we can compute the root value using the $\mathsf{siblings}$ and $\mathsf{item}$

$$
\mathsf{root} = \mathsf{SMT.computeRoot(item, siblings)}
$$

where $\mathsf{siblings}$ are the value of all sibling nodes of its Merkle path.

Using $\mathsf{item}$ and $\mathsf{siblings}$ we can prove an inclusion of the $\mathsf{item}$ in the $\mathsf{SMT}$. And also as $\mathsf{SMT}$ is a Sparse Merkle tree that has initial $\mathsf{null}$ value, we can also prove the non-inclusion proof using $\mathsf{null}$ and $\mathsf{pos}$.

Here's the reference solidity code of the Merkle Root computation.
```solidity
uint256 immutable public DEPTH;

function computeRoot(
    function(bytes32, bytes32) pure returns(bytes32) hash,
    uint256 pos,
    bytes32 item,
    bytes32[] sibligns
)
public
pure
returns (bytes32)
{
    require(siblings.length == DEPTH);
    uint256 path = pos;
    uint256 node = item;
    for (uint256 i = 0; i < siblings.length; i++) {
        if (path % 2 == 0) {
            // right sibling
            node = hash(node, siblings[i]);
        } else {
            // left sibling
            node = hash(siblings[i], node);
        }
        path >>= 1;
    }
    return node;
}
```

### 5.2 Tree updates

#### [5.2.1] Append-only Merkle Tree

Let $\mathsf{computeRoot}$ be defined in [5.1.5].

Let $\mathsf{index}$ be the $\mathsf{pos}$ of the leaf node to update. Append-only merkle tree is a sparse merkle tree that updates leaves from left to right by incrementing $\mathsf{index}$.

Then we can define the $\mathsf{append}$ function for $\mathsf{SMT}$ as

$$
\mathsf{(root_{next}, index_{next}, siblings_{next}) = SMT.append(root_{prev}, index_{prev}, siblings_{prev}, item)}
$$

where

$\mathsf{index_{next}} = \mathsf{index_{prev}} + 1$
$\mathsf{root_{prev}} == \mathsf{SMT.computeRoot(index_{prev}, 0, siblings_{prev})}$
$\mathsf{root_{next}} == \mathsf{SMT.computeRoot(index_{next}, 0, siblings_{next})}$
$\mathsf{root_{next}} == \mathsf{SMT.computeRoot(index_{prev}, item, siblings_{prev})}$

#### [5.2.2] Batch append

Let $\mathsf{append}$ be defined in [5.2.1]

Then, we can define $\mathsf{batchAppend}$ as

$$
\mathsf{(root_{next}, index_{next}, siblings_{next}) = SMT.batchAppend(root_{prev}, index_{prev}, siblings_{prev}, items)}
$$

where 

$\mathsf{(root_{i+1}, index_{i+1}, siblings_{i+1}) = SMT.append(root_{i}, index_{i}, siblings_{i}, item_i)}$
$\mathsf{root}_1 = \mathsf{root_{prev}}$
$\mathsf{root}_{n+1} = \mathsf{root_{next}}$
$\mathsf{[item_1, item_2, ..., item_n] = items}$


#### [5.2.3] Sub-tree Append

To update the Merkle tree we can insert a small sub-tree instead of updating each leaf. For example, let's assume we're tyring to add 256 items to a tree which depth is 32. Then, we can update the tree with only $32$ hash computations while we have to compute $32 \times 256$ hashes when we try to update the tree item by item.

First, let's divide a Sparse Merkle tree with sub-trees and its parent tree. For example,
```
                                    y
                        y                       y        
parent tree       y           y           y           y    
--------------------------------------------------------
sub trees      x     x     x     x     x     x     x     x  
             x   x x   x x   x x   x x   x x   x x   x x   x 
             ----- ----- ----- ----- ----- ----- ----- -----
             sub1  sub2  sub3  sub4  sub5  sub6  sub7  sub8
```


Then we can define the sub-tree and parent tree as Sparse Merkle trees like

$\mathsf{SubTree} = (d_{sub}, \mathsf{hash}, \mathsf{null})$
$\mathsf{ParentTree} = (\mathsf{depth} - d_{sub}, \mathsf{hash}, \mathsf{SubTree.initialRoot})$

And, divide the items into a fixed size chunks as

$\mathsf{chunks} = [\mathsf{chunk_1, ..., chunk_k}] = [[sub^{(1)}_1, ..., sub^{(1)}_{n_1}], [sub^{(2)}_1, ..., sub^{(2)}_{n_2}], ..., [sub^{(k)}_1, ..., sub^{(k)}_{n_k}]]$

where

$\mathsf{items} = [\mathsf{item_1, ..., item_n}] = [sub^{(1)}_1, ..., sub^{(1)}_{n_1}, sub^{(2)}_1, ..., sub^{(2)}_{n_2}, ..., sub^{(k)}_1, ..., sub^{(k)}_{n_k}]$
$n_1 = n_2 = ... = n_{k-1} = 2^{d_{sub}}$
$n_k = n - (2^{d_{sub}}\times (k - 1))$

Using the chunks, construct sub-trees and calculate their roots as

$\mathsf{subRoots} = [\mathsf{root}_{sub^{(1)}}, ..., \mathsf{root}_{sub^{(k)}}]$
where 
$(\mathsf{root}_{sub^{(i)}}, ,) = \mathsf{SubTree.batchAppend}(\mathsf{SubTree.initialRoot}, 0, \mathsf{SubTree.initialSiblings}, \mathsf{chunks}_{i})$

Finally, we can define the subtree appending as

$$
\mathsf{(root_{next}, index_{next}, siblings_{next})} = \mathsf{subTreeAppend}(d_{sub}, \mathsf{root_{prev}, index_{prev}, siblings_{prev}, items})
$$

where

$\mathsf{(root_{next}, index_{next}, siblings_{next}) = ParentTree.batchAppend(root_{prev}, index_{prev}, siblings_{prev}, subRoots)}$

### 5.3 Trees

#### [5.3.1] UTXO Tree

Let $C_{utxo-tree-depth} = 48$
Let $C_{utxo-subtree-depth} = 5$
Let $\mathsf{poseidon_n}$ be defined in [1.4]
Let $\mathsf{SMT}$ be defined in [5.1.1]
Let $\mathsf{subTreeAppend}$ be defined in [5.2.3]

$\mathsf{Tree_{utxo}} = \mathsf{SMT}(C_{utxo-tree-depth}, \mathsf{poseidon_2}, 0)$ is a Sparse Merkle Tree that stores newly created UTXOs using $\mathsf{subTreeAppend}$ with $d_{sub} = C_{utxo-subtree-depth}$.

#### [5.3.2] Withdrawal Tree

Let $C_{withdrawal-tree-depth} = 48$
Let $C_{withdrawal-subtree-depth} = 5$
Let $\mathsf{keccak256_2}$ be defined in [4.2.2.1]
Let $\mathsf{Tree}$ be defined in [5.1.1]
Let $\mathsf{subTreeAppend}$ be defined in [5.2.3]

$\mathsf{Tree_{withdrawal}} = \mathsf{SMT}(C_{withdrawal-tree-depth}, \mathsf{poseidon_2}, 0)$ is a Sparse Merkle Tree that stores newly created Withdrawals using $\mathsf{subTreeAppend}$ with $d_{sub} = C_{withdrawal-subtree-depth}$.

#### [5.3.3] Nullifier Tree

Let $C_{nullifier-tree-depth} = 254$
Let $\mathsf{keccak256_2}$ be defined in [4.2.2.1]
Let $\mathsf{Tree}$ be defined in [5.1.1]
Let $p$ be defined in [1.1]

$\mathsf{Tree_{nullifier}} = \mathsf{SMT}(C_{nullifier-tree-depth}, \mathsf{keccak256_2}, 0)$ is a Sparse Merkle Tree that stores $1$ into the leaf node where its $\mathsf{pos} = nullifier$ to mark that $nullifier$ as included in the tree.

To cover all possible nullifiers the number of items of the nullifier tree $\mathsf{Tree_{nullifier}} = 2^{C_{nullifier-tree-depth}} >= p$

## 6. Serialization

### 6.1 Block Definition

#### [6.1.1] Block

The $n$-th Zkopru layer 2 block is denoted as $\mathsf{Block}^{(n)} = (\mathsf{Header}^{(n)}, \mathsf{Body}^{(n)})$.

#### [6.1.2] Header

Let $\mathsf{Header}^{(n)}$ be the header of $n$-th block and consists of

| Symbol | Description |
| -------- | -------- | -------- |-------- |
| $\mathsf{proposer}^{(n)}$ | Ethereum address of the proposer of this block | 
| $\mathsf{parent}^{(n)}$ | The hash value of its parent block |
| $\mathsf{fee}^{(n)}$ | Fee for the block proposer |
| $\mathsf{root}^{(n)}_\mathsf{utxo}$ | $\mathsf{root}$ of the updated $\mathsf{Tree_{utxo}}$|
| $\mathsf{index}^{(n)}_\mathsf{utxo}$ | Starting position of new leaf of $\mathsf{Tree_{utxo}}$ |
| $\mathsf{root}^{(n)}_\mathsf{nullifier}$ | $\mathsf{root}$ of the updated $\mathsf{Tree_{nullifier}}$|
| $\mathsf{root}^{(n)}_\mathsf{withdrawal}$ | $\mathsf{root}$ of the updated $\mathsf{Tree_{withdrawal}}$|
| $\mathsf{index}^{(n)}_\mathsf{withdrawal}$ | Starting position of new leaf of $\mathsf{Tree_{withdrawal}}$ |
| $\mathsf{txRoot}^{(n)}$ | Merkle root of transaction hashes|
| $\mathsf{depositRoot}^{(n)}$ | Merkle root of mass deposit hashes|
| $\mathsf{migrationRoot}^{(n)}$ | Merkle root of mass migration hashes|

#### [6.1.3] Body

Let $\mathsf{Body}^{(n)}$ be the body of $n$-th block and consists of

| Symbol | Description |
| -------- | -------- |
| $\mathrm{TXs}^{(n)}$ | $=[\mathcal{Tx}_1^{(n)}, ..., \mathcal{Tx}_{n_{tx}}^{(n)}]$. Array of transactions.| 
| $\mathrm{MDs}^{(n)}$ | $=[\mathrm{MD}_1^{(n)}, ..., \mathrm{MD}_{n_{md}}^{(n)}]$. Array of mass deposits.| 
| $\mathrm{MMs}^{(n)}$ | $=[\mathrm{MM}_1^{(n)}, ..., \mathrm{MM}_{n_{mm}}^{(n)}]$. Array of mass migrations.|

Where 
$\mathcal{Tx}_i^{(n)}$ is the$ i$-th transaction of the $n$-th block.
$\mathrm{MD}_i^{(n)}$ is the $i$-th mass deposit of the $n$-th block.
$\mathrm{MM}_i^{(n)}$ is the $i$-th mass migration of the $n$-th block.

### 6.2 Header Serialization

#### [6.2.1]

Let $\mathsf{proposer}^{(n)}$, $\mathsf{parent}^{(n)}$, $\mathsf{fee}^{(n)}$, $\mathsf{root}^{(n)}_\mathsf{utxo}$, $\mathsf{index}^{(n)}_\mathsf{utxo}$, $\mathsf{root}^{(n)}_\mathsf{withdrawal}$, $\mathsf{index}^{(n)}_\mathsf{withdrawal}$, $\mathsf{root}^{(n)}_\mathsf{nullifier}$, $\mathsf{txRoot}^{(n)}$, $\mathsf{depositRoot}^{(n)}$, and $\mathsf{migrationRoot}^{(n)}$ be defined in [6.1.2]

Then, we can define their serialized form as below:

| Value | Serialized | Serialization |
| -------- | -------- | -------- |-------- |
| $\mathsf{proposer}^{(n)}$ | $\mathtt{proposer}^{(n)}$ |20 bytes / big-endian |
| $\mathsf{parent}^{(n)}$ | $\mathtt{parent}^{(n)}$ | 32 bytes / big-endian |
| $\mathsf{fee}^{(n)}$ | $\mathtt{fee}^{(n)}$ | 32 bytes / big-endian |
| $\mathsf{root}^{(n)}_\mathsf{utxo}$ | $\mathtt{root}^{(n)}_\mathsf{utxo}$ | 32 bytes / big-endian |
| $\mathsf{index}^{(n)}_\mathsf{utxo}$ |  $\mathtt{index}^{(n)}_\mathsf{utxo}$ | 32 bytes / big-endian |
| $\mathsf{root}^{(n)}_\mathsf{nullifier}$  |  $\mathtt{root}^{(n)}_\mathsf{nullifier}$  | 32 bytes / big-endian |
| $\mathsf{root}^{(n)}_\mathsf{withdrawal}$  | $\mathtt{root}^{(n)}_\mathsf{withdrawal}$  | 32 bytes / big-endian | 
| $\mathsf{index}^{(n)}_\mathsf{withdrawal}$ | $\mathtt{index}^{(n)}_\mathsf{withdrawal}$ | 32 bytes / big-endian |
| $\mathsf{txRoot}^{(n)}$ |  $\mathtt{txRoot}^{(n)}$ | 32 bytes / big-endian |
| $\mathsf{depositRoot}^{(n)}$ |  $\mathtt{depositRoot}^{(n)}$ | 32 bytes / big-endian |
| $\mathsf{migrationRoot}^{(n)}$ |  $\mathtt{migrationRoot}^{(n)}$ | 32 bytes / big-endian |

Then $\mathtt{Header}^{(n)}$, the serialized form of $\mathsf{Header}^{(n)}$, is the concatenation of $(\mathsf{proposer}^{(n)}, \mathsf{parent}^{(n)}, \mathsf{fee}^{(n)}, \mathsf{root}^{(n)}_\mathsf{utxo}, \mathsf{index}^{(n)}_\mathsf{utxo}, \mathsf{root}^{(n)}_\mathsf{nullifier}, \mathsf{root}^{(n)}_\mathsf{withdrawal}, \\
\mathsf{index}^{(n)}_\mathsf{withdrawal}, \mathsf{txRoot}^{(n)}, \mathsf{depositRoot}^{(n)}, \mathsf{migrationRoot}^{(n)})$

### 6.3 Transaction Serialization

#### [6.3.1] Dynamic sized buffer

Prepare a dynamic sized buffer, $\mathtt{buff}_{tx_{i}}$. We will push bytes data to the buffer by the following sequences. The final state of $\mathtt{buff}_{tx_{i}}$ after appending all data becomes $\mathtt{Tx}_i$.

#### [6.3.2] Input note length

Let $m$ be the number of input notes that is defined in [3.4.1].
Store $m$ into a single byte and push it to $\mathtt{buff}_{tx_{i}}$.

#### [6.3.3] Input notes

Let $\mathcal{In}_i$ be the $i$-th input note of $\mathcal{Tx}$ as it's defined in [3.4.1]
Let $m$ be the number of input notes that is defined in [3.4.1].

Starting from $i$ = 1 and repeat the below steps until $i$ is less than or equal to $m$:

1. Let $(\mathsf{nullifier}(\mathrm{U}), \mathsf{root_{utxo}}^{(n)}) = \mathcal{In}_i$
2. Serialize $\mathsf{nullifier}(\mathrm{U})$ into 32 bytes buffer with big-endian.
3. Serialize $\mathsf{root_{utxo}}^{(n)}$ in to 32 bytes buffer with big-endian.
4. Concatenate the serialized $\mathsf{nullifier}(\mathrm{U})$ and $\mathsf{root_{utxo}}^{(n)}$ into 64 bytes buffer and let it be $\mathtt{In}_i$
5. Push $\mathtt{In}_i$ to $\mathtt{buff}_{tx_{i}}$

#### [6.3.4] Output note length

Let $n$ be the number of output notes that is defined in [3.4.1].
Store $n$ into a single byte and push it to $\mathtt{buff}_{tx_{i}}$.

#### [6.3.5] Output notes

Let $\mathcal{Out}_i$ be the $i$-th output note of $\mathcal{Tx}$ as it's defined in [3.4.1]
Let $n$ be the number of output notes that is defined in [3.4.1].

Starting from $i$ = 1 and repeat the below steps until $i$ is less than or equal to $n$:

1. Let $(\mathsf{hash}(\mathsf{N}), t, \mathcal{N}) = \mathcal{Out}_i$ and 
$\mathcal{N} = \mathsf{(to, v'_{eth}, addr'_{token}, v'_{erc20}, v'_{nft}, fee_{L1})}$ as defined in [3.1.1]
2. Prepare a dynamic sized buffer $\mathtt{buff}_{out_i}$.
3. Serialize $\mathsf{hash}(\mathsf{N})$ into 32 bytes data with big-endian and push them to $\mathtt{buff}_{out_i}$.
4. Store $t$ into a single byte with big-endian and push it to $\mathtt{buff}_{out_i}$.
5. If $t$ is not zero,
    1. Prepare 168 bytes buffer $\mathtt{buff}_{public_i}$
    2. Serialize $\mathsf{to}$ to a 20 bytes buffer with big-endian.
    3. Serialize $\mathsf{v'_{eth}}$ to a 32 bytes data with big-endian.
    4. Serialize $\mathsf{addr'_{token}}$ to a 20 bytes data with big-endian.
    5. Serialize $\mathsf{v'_{erc20}}$ to a 32 bytes data with big-endian.
    6. Serialize $\mathsf{v'_{nft}}$ to a 32 bytes data with big-endian.
    7. Serialize $\mathsf{fee_{L1}}$ to a 32 bytes data with big-endian.
    8. Concatenate the serialized data and store them to $\mathtt{buff}_{public_i}$.
    9. Push $\mathtt{buff}_{public_i}$ to $\mathtt{buff}_{out_i}$
6. Push $\mathtt{buff}_{out_i}$ to $\mathtt{buff}_{tx_i}$

#### [6.3.6] Transaction Fee

Let $f$ be the transaction fee for $\mathcal{Tx}$ as defined in [3.2.6].
Serialize $f$ into a 32 bytes buffer with big-endian, and push it to $\mathtt{buff}_{tx_i}$.

#### [6.3.7] SNARK

Let $\pi = (\mathbf{A}, \mathbf{B}, \mathbf{C})$ be defined in [3.2.5].

1. Prepare 256 bytes empty buffer $\mathtt{buff}_{snark}$.
2. Serialize $\mathbf{A}.x$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
3. Serialize $\mathbf{A}.y$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
4. Serialize $\mathbf{B}.{x_1}$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
5. Serialize $\mathbf{B}.{x_2}$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
6. Serialize $\mathbf{B}.{y_1}$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
7. Serialize $\mathbf{B}.{y_2}$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
8. Serialize $\mathbf{C}.x$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
9. Serialize $\mathbf{C}.y$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{snark}$.
10. Push $\mathtt{buff}_{snark}$ to $\mathtt{buff}_{tx_i}$.

#### [6.3.8] Extra data

Serialized form of transaction can have some extra data. Zkopru expresses the existence of extra data using 2 bits.

1. Prepare a single byte $\mathtt{b}$.
2. Let $\mathsf{swap}$ be defined in [3.2.6]. If $\mathsf{swap}$ is not zero, store 1 on its right bit position.
3. If $\mathtt{Tx}$ has a memo field, store 1 on its second right bit position.
4. Push the byte $\mathtt{b}$ into $\mathtt{buff}_{tx_i}$
5. If the right most bit of $\mathtt{b}$ is 1, serialize $\mathsf{swap}$ to a 32 bytes buffer with big-endian and push it to $\mathtt{buff}_{tx_i}$.
6. Let $\mathsf{memo}$ be defined in [3.3.1]. If the second right most bit of $\mathtt{b}$ is 1, push that 81 bytes $\mathsf{memo}$ data to $\mathtt{buff}_{tx_i}$

#### [6.3.9] $\mathtt{TX}_i$

Freeze the dynamic sized buffer $\mathtt{buff}_{tx_i}$ and let it be $\mathtt{TX}_i$

### 6.4 Mass Deposit Serialization

#### [6.4.1] Mass Deposit

Let $\mathrm{MD}_i$ be the $i$-th Mass Deposit of $\mathsf{Block}^{(n)}$.

1. Prepare a 64 bytes buffer $\mathtt{buff}_{md_i}$.
2. Let $(\mathsf{merged}, f_{MD}) = \mathrm{MD}_i$ as defined in [4.2.2.3]
3. Serialize $\mathsf{merged}$ into 32 bytes buffer with big-endian.
4. Serialize $f_{MD}$ into 32 bytes buffer with big-endian.
5. Concatenate the serialized $\mathsf{merged}$ and $f_{MD}$ into $\mathtt{buff}_{md_i}$ and let it be $\mathtt{MD}_i$

### 6.5 Mass Migration Serialization

#### [6.5.1] Mass Migration


Let $\mathrm{MM}_i$ be the $i$-th Mass Migration of $\mathsf{Block}^{(n)}$ and $\mathrm{MM} = (\mathsf{dest}, \mathsf{asset_{migration}}, \mathrm{MD}_\mathsf{migration})$ as defined in [4.4.1].

Then, let $(\mathsf{eth, token, amount}) = \mathsf{asset_{migration}}$ and $(\mathsf{merged}, f_\mathrm{MD}) = \mathrm{MD}_\mathsf{migration}$
1. Prepare 168 bytes buffer $\mathtt{buff}_{mm_{i}}$.
2. Serialize $\mathsf{dest}$ into 20 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
3. Serialize $\mathsf{eth}$ into 32 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
4. Serialize $\mathsf{token}$ into 20 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
5. Serialize $\mathsf{amount}$ into 32 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
6. Serialize $\mathsf{migration}.\mathsf{merged}$ into 32 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
7. Serialize $f_\mathsf{MD}$ into 32 bytes buffer with big-endian and push it into $\mathtt{buff}_{mm_i}$.
8. Finally set $\mathtt{buff}_{mm_i}$ as $\mathtt{MM}_i$

### 6.6 Body Serialization

#### [6.6.1] Dynamic Sized Buffer

Prepare a dynamic sized buffer, $\mathtt{buff}_{body}$. We will push bytes data to the buffer by the following sequences. The final state of $\mathtt{buff}_{body}$ after appending all data becomes $\mathtt{Body}^{(n)}$.

#### [6.6.2] Add Transactions

Let $\mathsf{Block}^{(n)}$ has $n_{tx}$ number of transactions.

1. Prepare a dynamic sized buffer $\mathtt{buff}_{tx}$ to store all serialized transactions.
2. Store $n_{tx}$ into a single byte with big-endian and push it into $\mathtt{buff}_{tx}$.
3. Starting from $i$ = 1 and repeat the below steps until $i$ is less than or equal to $n_{tx}$:
    1. Serialize $\mathcal{Tx}_i$ to $\mathtt{Tx}_i$ as defined in [6.3.9]
    2. Push $\mathtt{Tx}_i$ to $\mathtt{buff}_{tx}$.
4. Let $\mathtt{buff}_{tx}$ be $\mathtt{TXs}^{(n)}$ and push it to $\mathtt{buff}_{body}$

#### [6.6.3] Add Mass Deposits

Let $\mathsf{Block}^{(n)}$ has $n_{md}$ number of mass deposits.

1. Prepare a dynamic sized buffer $\mathtt{buff}_{md}$ to store all serialized mass deposits.
2. Store $n_{md}$ into a single byte with big-endian and push it into $\mathtt{buff}_{md}$.
3. Starting from $i$ = 1 and repeat the below steps until $i$ is less than or equal to $n_{md}$:
    1. Serialize $\mathrm{MD}_i$ to $\mathtt{MD}_i$ as defined in [6.4.1]
    2. Push $\mathtt{MD}_i$ to $\mathtt{buff}_{md}$.
4. Let $\mathtt{buff}_{md}$ be $\mathtt{MDs}^{(n)}$ and push it to $\mathtt{buff}_{body}$

#### [6.6.4] Add Mass Migrations

Let $\mathsf{Block}^{(n)}$ has $n_{mm}$ number of mass migrations.

1. Prepare a dynamic sized buffer $\mathtt{buff}_{mm}$ to store all serialized transactions.
2. Store $n_{mm}$ into a single byte with big-endian and push it into $\mathtt{buff}_{mm}$.
3. Starting from $i$ = 1 and repeat the below steps until $i$ is less than or equal to $n_{mm}$:
    1. Serialize $\mathrm{MM}_i$ to $\mathtt{MM}_i$ as defined in [6.3.9]
    2. Push $\mathtt{MM}_i$ to $\mathtt{buff}_{mm}$.
4. Let $\mathtt{buff}_{mm}$ be $\mathtt{MMs}^{(n)}$ and push it to $\mathtt{buff}_{body}$


#### [6.6.5] $\mathtt{Body}^{(n)}$

$\mathtt{Body}^{(n)}$ is the serialized form of $\mathsf{Body}^{(n)}$ and consists of

| Symbol | Description |
| -------- | -------- |
| $\mathtt{TXs}^{(n)}$ | $=[\mathtt{TX}_1^{(n)}, ..., \mathtt{TX}_{n_{tx}}^{(n)}]$| 
| $\mathtt{MDs}^{(n)}$ | $=[\mathtt{MD}_1^{(n)}, ..., \mathtt{MD}_{n_{md}}^{(n)}]$| 
| $\mathtt{MMs}^{(n)}$ | $=[\mathtt{MM}_1^{(n)}, ..., \mathtt{MM}_{n_{mm}}^{(n)}]$|

Where 
$\mathtt{TX}_i^{(n)}$ is the serialized form of $\mathcal{Tx}_i$, the $i$-th transaction of the $n$-th block.
$\mathtt{MD}_i^{(n)}$ is the serialized form of $\mathrm{MD}_i$, the $i$-th mass deposit of the $n$-th block.
$\mathtt{MM}_i^{(n)}$ is the serialized form of $\mathrm{MM}_i$, the $i$-th mass migration of the $n$-th block.

Freeze the dynamic sized buffer $\mathtt{buff}_{body}$ and let it be $\mathtt{Body}^{(n)}$

### 6.7 Block Serialization

#### [6.7.1] Block

Let $\mathtt{Header}^{(n)}$ be defined in [6.2.1].
Let $\mathtt{Body}^{(n)}$ be defined in [6.6.5].

Then, $\mathtt{Block}^{(n)}$ is the serialized form of $\mathsf{Block}^{(n)}$ and equals the concatenation of $\mathtt{Header}^{(n)}$ and $\mathtt{Body}^{(n)}$.

#### [6.7.2] Finalization Data

To finalize a block proposal, it requires the header data, and the mass deposits and mass migrations.

Let $\mathtt{Header}^{(n)}$ be defined in [6.2.1].
Let $\mathtt{MDs}^{(n)}$ be defined in [6.6.5].
Let $\mathtt{MMs}^{(n)}$ be defined in [6.6.5].

First, compute the data checksum of original block data using keccak256:
$$
\mathsf{checksum} = keccak256(\mathtt{Block}^{(n)})
$$

Then, $\mathtt{Finalization}^{(n)}$ is the concatenation of $(\mathsf{checksum}, \mathtt{Header}^{(n)}, \mathtt{MDs}^{(n)}, \mathtt{MMs}^{(n)})$.

## 7. Miscellaneous

#### [7.1]: MerkleRoot

$\mathsf{MerkleRoot_{h}([item_1, ...., item_n])}$ means the root of a [hash tree](https://www.researchgate.net/profile/Ralph-Merkle/publication/220713913_Protocols_for_Public_Key_Cryptosystems/links/00b495384ecda07784000000/Protocols-for-Public-Key-Cryptosystems.pdf) that uses $\mathsf{h}$ for its hash function.
