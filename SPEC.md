# Zkopru protocol specification - v2.0.0-alpha.0


:::info
Please note that current version is still in WIP. ==Marked texts== will get unmarked when they have more detail explanations.
:::

## 1. Account & Elliptic Curve Cryptography

Zkopru uses Baby Jubjub curve for its Elliptic Curve Cryptography. The arithmeric is defined in the reference [paper](https://iden3-docs.readthedocs.io/en/latest/_downloads/33717d75ab84e11313cc0d8a090b636f/Baby-Jubjub.pdf) written by Barry Whitehat and Jordi Baylina.

#### [1.1]

Let
$$
p = 21888242871839275222246405745257275088548364400416034343698204186575808495617
$$
and, $\mathbb{F}_p$ be the finite field which modular is $p$.

#### [1.2]
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

#### [1.3]

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
$s$ is the 32 bytes private key of the account. For the seamless user experience, Zkopru uses Ethereum account's private key for its corresponding Zkopru account's private seed key.

#### [1.8]
Let $r, \mathbb{F}_r$ be defined in [1.2].
Let $s$ be defined in [1.7].

$a \in \mathbb{F}_r$ is the scalar multiplier that is generated from the private seed key $s$ by the [RFC8032 5.1.5 Key Generation](https://tools.ietf.org/html/rfc8032#section-5.1.5) protocol with blake512 hash function.

1. Hash the 32-byte private key $ss$ using $\mathsf{blake512}$, storing the digest in a 64-octet large buffer, denoted h. Only the lower 32 bytes are used for generate the scalar multiplier $a$.
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

$\mathcal{N} = \mathsf{(to, v'_{eth}, addr'_{token}, v'_{erc20}, v'_{nft}, fee_{caller})} \\
\ = \left\{\begin{array}{lr}
        (0, 0, 0, 0, 0, 0) & \text{for } t = 0 & \text{(utxo)}\\
        \mathsf{(recipient, v_{eth}, addr_{token}, v_{erc20}, v_{nft}, fee_{caller})} & \text{for } t = 1 & \text{(withdrawal)}\\
        \mathsf{(dest, v_{eth}, addr_{token}, v_{erc20}, v_{nft}, fee_{caller})} & \text{for } t = 2 &  \text{(migration)}
        \end{array}\right\}$

Then, the public outflow data is defined as

$$
\mathcal{Out} = (\mathsf{hash}(\mathsf{N}), t, \mathcal{N})
$$

#### [3.1.2]

Let $t$ be defined in [3.1.1]
$\mathrm{U}$ is a UTXO type transaction output which $t = 0$.

#### [3.1.3]
Let $t$ be defined in [3.1.1]
$\mathrm{W}$ is a withdrawal type transaction output  which $t = 1$.

#### [3.1.4]
Let $t$ be defined in [3.1.1]
$\mathrm{M}$ is a migration type transaction output  which $t = 2$.

#### [3.1.5]

Let $\mathrm{U}$ be defined in [3.1.2].
Let $\mathcal{T_\mathsf{utxo}}$ be defined in ==[7.1.1]==.

$\mathsf{pos}(\mathrm{U})$ is the leaf index of $\mathrm{U}$ in the UTXO Merkle tree $\mathcal{T}_\mathsf{utxo}$.

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
Let $\mathcal{T_\mathsf{utxo}}$ be defined in ==[7.1.1]==.
Let $\mathsf{root_{utxo}}$ be defined in ==[7.1.1]==.

Let the root of $\mathcal{T}_\mathsf{utxo}$ at the $n$-th layer 2 block be denoted $\mathsf{root_{utxo}}^{(n)}$.
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
Let $\mathsf{C}_{(x,y)}$ be defined in ==X.Y.Z==.
Let $\mathsf{pk}_{(x,y)}$ be defined in ==X.Y.Z==.

$\mathrm{Tx} = (\mathrm{[In_1, ..., In_m], [Out_1, ..., Out_n]})$ is a raw Zkopru transaction where

Then the prover calculate the witness:
$$
\mathsf{w} \leftarrow \mathsf{witness}(\mathsf{C}_{(m,n)}, \mathsf{Tx_{raw}}, \mathsf{hiding}(\mathsf{Tx_{raw}}))
$$
and generate the SNARK proof using the proving key $\mathsf{pk}_{(m, n)}$:
$$
\pi \leftarrow \mathsf{prove_{groth16}}(\mathsf{w}, \mathsf{pk}_{(m,n)})
$$

Then the shielded transaction

$$
\mathcal{Tx} = (\mathcal{P}, \pi, \mathsf{memo})
$$
where the public signals
$$
\mathcal{P} = (\mathcal{[In_1, ..., In_m], [Out_1, ..., Out_n]})
$$


