# Zkopru protocol specification - v2.0.0-alpha.0


:::info
Please note that current version is still in WIP. ==Marked texts== will get unmarked when they have more detail explanations.
:::

## 1. Account

#### [1.1]
Elliptic Curve Cryptography in Zkopru uses Babyjubjub curve that is described in [here](https://iden3-docs.readthedocs.io/en/latest/iden3_repos/research/publications/zkproof-standards-workshop-2/baby-jubjub/baby-jubjub.html)
#### [1.2]
$poseidon_{X}$ is a poseidon hash function which consumes $X$ inputs as described [here](https://github.com/iden3/circomlib/blob/master/src/poseidon.js).
#### [1.3]
$G$ is a constant generator point on the babyjubjub curve.
#### [1.4]
$p$ is the private spending key of the note owner.
#### [1.5]
$P$ is the point on the babyjubjub curve that is $P=p\cdot G$.
#### [1.6]
$v$ is the private viewing key and also the nullifier seed that satisfies $v = keccak256(p)\ mod \ r$ where $r = 21888242871839275222246405745257275088548364400416034343698204186575808495617$.
#### [1.7]
$S$ is the public spending key that is $S = poseidon_3(x, y, v)$ where $(x,y) = P$.
#### [1.8]
$V$ is public viewing key that is $V = v \cdot G$.
#### [1.9]
$li_{U}({U})$ is the leaf index of UTXO ${U}$ in the $Tree_{U}$.
#### [1.10]
$nullifier({U}) = poseidon_2(v, li_U({U}))$.

## 2. Note

#### [2.1]
$h({N})$ is the hash of Zkopru note ${N}$.
#### [2.2]
Zkopru note ${N}$ contains ETh and token.
#### [2.3]
$asset_{eth}({N})$ is the amount of Ether that the note ${N}$ contains.
#### [2.4]
$asset_{token}({N})$ is the token address if the note ${N}$ contains ERC20 or ERC721 tokens.
#### [2.5]
$asset_{erc20}({N})$ is the amount of token if the $asset_{token}$ is registered as an ERC20 token on the layer 1 or $0$.
#### [2.6]
$asset_{nft}({N})$ is the amount of token if the $asset_{token}$ is registered as an ERC721 token on the layer 1 or $0$.
#### [2.7]
Asset of ${N}$ is $Asset({N}) = \{asset_{eth}({N}), asset_{token}({N}), asset_{erc20}({N}), asset_{nft}({N})\}$.
#### [2.8]
Asset hash is $h(Asset({N})) = poseidon_4(asset_{eth}({N}), asset_{token}({N}), asset_{erc20}({N}), asset_{nft}({N}))$.
#### [2.9]
$salt({N})$ is a random 16 bytes value for ${N}$.
#### [2.10]
 The hash of the note $h({N}) = poseidon_3(S({N}), salt({N}), h(Asset({N})))$ where $S({N})$ is the public spending key for ${N}$.

## 3.Transaction 
### 3.1 Basic definitions
#### [3.1.1]
A Zkopru transaction ${TX}^{(n)}_i$ is the $i$-th transaction of the $n$-th block.

#### [3.1.2]

A zkopru transaction, that consumes $m$ UTXOs and creates $n$ outputs, should have a proof against SNARK circuit $Circuit(m,n)$.

#### [3.1.2]
$m$ and $n$ both belong to the set $\{1 ,2, 3, 4\}$.

#### [3.1.3]
${U}$ is a UTXO type transaction output and it contains note ${N}$

#### [3.1.4]
${W}$ is a withdrawal type transaction output and it contains $\{{N}, fee\}$ where $fee$ is the additional fee for instant withdrawal as defiend in ==X.Y.Z==

#### [3.1.5]
${M}$ is a migration type transaction output and it contains $\{{N}, dest, fee\}$ where $dest$ is the contract address of the destination network and $fee$ is the additional fee for the destination network block proposer as defiend in ==X.Y.Z==

#### [3.1.6]

Let's define transaction output ${O} = \left\{\begin{array}{lr}
        {U} & \text{for } t({O}) = 0\\
        {W} & \text{for } t({O}) = 1\\
        {M} & \text{for } t({O}) = 2\\
        \end{array}\right\}$
where $t({O})$ means the output type.

#### [3.1.7]
$root_{UTXO}^{(n)}$ is the root of the UTXO tree at the $n$-th layer 2 block.

#### [3.1.8]
$Sib_{UTXO}^{(n)}({U})$ is the set of all sibling node data of UTXO ${U}$ at the $n$-th layer 2 block.

### 3.2 Spending
#### [3.2.1]
$i$ belongs to the set $\{1, ..., m\}$ and ${U}_i$ is an input UTXO of Zkopru transaction.
#### [3.2.2]
To spend UTXO ${U}_i$, prover knows $p({U}_i)$, $P({U}_i)$, $nullifier({U}_i)$, $salt({U}_i)$, $asset_{eth}({U}_i)$, $asset_{token}({U}_i)$, $asset_{erc20}({U}_i)$, $asset_{nft}({U}_i)$, ${li({U}_i)}$, $Sib_{UTXO}^{(n)}({U}_i)$, $root_{UTXO}^{(n)}$, and $v({U}_i)$ for each ${U}_i$.
#### [3.2.3]
Prover reveals $root_{UTXO}^{(n)}$ and $nullifier({U}_i)$ while hiding $p({U}_i)$, $P({U}_i)$, $v({U}_i)$, $salt({U}_i)$, $asset_{eth}({U}_i)$, $asset_{token}({U}_i)$, $asset_{erc20}({U}_i)$, $asset_{nft}({U}_i)$, ${li({U}_i)}$, and $Sib_{UTXO}^{(n)}({U}_i)$ of each ${U}_i$
#### [3.2.4]
Revealed data $root_{UTXO}^{(n)}$ and $nullifier({U}_i)$ satisfies the optimistic rollup condition ==[X.Y.Z]==
#### [3.2.5]
For each ${U}_i$, prover calculates the EdDSA signatures using private key $p$ and target value $h({U}_i)$ and get $sigR8({U}_i)$ and $sigS({U}_i)$.
#### [3.2.6]
Prover provides $sigR8({U}_i)$, $sigS({U}_i)$, $P({U}_i)$, $n({U}_i)$, $salt_i({U}_i)$, $asset_{eth}({U}_i)$, $asset_{token}({U}_i)$, $asset_{erc20}({U}_i)$, $asset_{nft}({U}_i)$, ${li({U}_i)}$, ${Sib^{(n)}_UTXO({U}_i)}$, $root_{UTXO}^{(n)}$, and $nullifier({U}_i)$ into the $Circuit(m,n)$ and they satisfy the following SNARK conditions.

<div style="padding-left: 5rem">

#### [3.2.5.1]
$S({U}_i)$ is calculated inside $Circuit(m,n)$ and the $\{P({U}_i), n({U}_i)\}$ satisfies condition [[1.7]](#1.7).
#### [3.2.5.2]
$h({U}_i)$ is calculated inside $Circuit(m,n)$ and $\{ h({U}_i), ({U}_i), salt({U}_i)\}$ satisfies condition [[2.10]](#2.10).
#### [3.2.5.3]
$EdDSA(sigR8({U}_i), sigS({U}_i), h({U}_i)) == P({U}_i)$
#### [3.2.5.4]
$nullifier({U}_i)$ is calculated inside $Circuit(m,n)$ and $\{nullifier({U}_i), v({U}_i), li({U}_i)\}$ satisfies condition [[1.10]](#1.10).
#### [3.2.5.5]
$MerkleRoot_{Poseidon_2}(li({U}_i), {U}_i, Sib_{UTXO}^{(n)}({U}_i)) == root_{UTXO}^{(n)}$
#### [3.2.5.6]
$asset_{eth}({U}_i)$ and $asset_{erc20}({U}_i)$ are less than $2^{245}$ to prevent overflow in SNARK.
#### [3.2.5.7]
$asset_{erc20}({U}_i)$ or $asset_{nft}({U}_i)$ should be zero.

</div>

### 3.3 Outputs


#### [3.3.1]
$j$ belongs to the set $\{1, ..., n\}$ and ${O}_j$ is a transaction output of a Zkopru transaction.
#### [3.3.2]
Output has 3 types and the type value $t_j$ belongs to the set $\{0, 1, 2\}$.
#### [3.3.3]
To create new outputs ${O}_j$, prover knows $S({O}_j)$, $salt({O}_j)$, $asset_{eth}({O}_j)$, $asset_{token}({O}_j)$, $asset_{erc20}({O}_j)$, $asset_{nft}({O}_j)$, $h({O}_j)$, and $t({O}_j)$ for each new output notes.
#### [3.3.4]
Prover reveals $h({O}_j)$, $t({O}_j)$ while hiding $S({O}_j)$, $salt({O}_j)$, $asset_{eth}({O}_j)$, $asset_{token}({O}_j)$, $asset_{erc20}({O}_j)$, $asset_{nft}({O}_j)$.
#### [3.3.5]
$data_{to}({O}_j)$, $data_{eth}({O}_j)$, $data_{token}({O}_j)$, $data_{erc20}({O}_j)$, $data_{erc721}({O}_j)$, and $data_{fee}({O}_j)$ are public data set for layer 1 interactions such as withdrawal or migration.
#### [3.3.6]
If $t({O}_j)$ is $0$, $data_{to}({O}_j)$, $data_{eth}({O}_j)$, $data_{token}({O}_j)$, $data_{erc20}({O}_j)$, $data_{erc721}({O}_j)$, and $data_{fee}({O}_j)$ are all 0.
#### [3.3.7]
If $t({O}_j)$ is $1$ or $2$, the prover knows $\{data_{to}({O}_j), data_{eth}({O}_j), data_{token}({O}_j), data_{erc20}({O}_j), data_{erc721}({O}_j), data_{fee}({O}_j)\}$ and reveals them.
#### [3.3.8]
If $t_j$ is 0, it satisfies optimistic rollup condition ==[X.Y.Z]== to update $Tree_{U}$.
#### [3.3.9]
If $t_j$ is 1, it satisfies optimistic rollup condition ==[X.Y.Z]== to update $Tree_{Withdrawal}$.
#### [3.3.10]
If $t_j$ is 2, it satisfies optimistic rollup condition ==[X.Y.Z]== for the inter layer-2 migration.
#### [3.3.11]
Prover provides $S({O}_j)$, $salt({O}_j)$, $asset_{eth}({O}_j)$, $asset_{token}({O}_j)$, $asset_{erc20}({O}_j)$, $asset_{nft}({O}_j)$, $h({O}_j)$, $t({O}_j)$, $data_{to}({O}_j)$, $data_{eth}({O}_j)$, $data_{token}({O}_j)$, $data_{erc20}({O}_j)$, $data_{erc721}({O}_j)$, and $data_{fee}({O}_j)$ to the $Circuit(m,n)$ and they satisfy the following SNARK conditions.
<div style="padding-left: 5rem">

#### [3.3.11.1]
$\{S({O}_j), salt({O}_j), asset_{eth}({O}_j), asset_{token}({O}_j), asset_{erc20}({O}_j), asset_{nft}({O}_j), h({O}_j)\}$ satisfies condition [[2.10]](#2.10).
#### [3.3.11.2]
$\{t({O}_j), data_{to}({O}_j), data_{eth}({O}_j), data_{token}({O}_j), data_{erc20}({O}_j), data_{erc721}({O}_j), data_{fee}({O}_j)\}$ satisfies condition [[3.3.6]](#3.3.6). and condition [[3.3.7]](#3.3.7).
#### [3.3.11.3]
$asset_{eth}({O}_j)$, $asset_{erc20}({O}_j)$ are less than $2^{245}$ to prevent overflow in SNARK.
#### [3.3.11.4]
$asset_{erc20}({O}_j)$ or $asset_{nft}({O}_j)$ should be zero.

</div>

### 3.4 Zero-sum

Let's define $filter(x,y)$ as $filter(x,y) = \left\{\begin{array}{lr}
        0 & \text{for } x \neq y\\
        1 & \text{for } x  = y\\
        \end{array}\right\}$

#### [3.4.1]

To prove the sum of input assets is equal to the sum of output assets, the prover knows $\{asset_{eth}({U}), asset_{eth}({O}), asset_{token}({U}), asset_{token}({O}), asset_{erc20}({U}), asset_{erc20}({O}), asset_{nft}({U}), asset_{nft}({O}), data_{fee}({O}), f\}$ where $U \in [U_1, ..., U_m]$ and $O \in [O_1, ..., O_n]$. And they satisfies [[3.4.2]](#3.4.2), [[3.4.3]](#3.4.3), and [[3.4.4]](#3.4.4).

#### [3.4.2]
$\Sigma_{i = 1}^{m}asset_{eth}({U}_i) = \Sigma_{j = 1}^n asset_{eth}({O}_j) + \Sigma_{j = 1}^n data_{fee}({O}_j) + f$ where $f$ is the layer-2 transaction fee.

#### [3.4.3]
Let $\mathbb{U} = [U_1, ..., U_m]$, and $\mathbb{O} = [O_1, ..., O_n]$.
For every $addr \in (asset_{token}(\mathbb{U}) \cup asset_{token}(\mathbb{O}))$,
$\Sigma_{i = 1}^{m}asset_{erc20}({U}_i)\cdot filter(addr, asset_{token}({U}_i)) = \Sigma_{j = 1}^{n}asset_{erc20}({O}_j)\cdot filter(addr, asset_{token}({O}_j))$

#### [3.4.4]

For every $addr \in (asset_{token}(\mathbb{U}) \cup asset_{token}(\mathbb{O}))$ and every $nft \in (asset_{nft}(\mathbb{U}) \cup asset_{nft}(\mathbb{O}))$, 

$\Sigma_{i = 1}^{m} filter(asset_{nft}({U}_i), nft)\cdot filter(addr, asset_{token}({U}_i)) \\
= \Sigma_{j = 1}^{n} filter(asset_{nft}({O}_j), nft)\cdot filter(addr, asset_{token}({O}_j)) \\
= existence \in {0, 1}$


## 4. Deposit

#### [4.1]

Let's define deposit ${D} = \{{U}, f\}$ where ${U}$ is a UTXO and $f$ is fee for block proposer.

#### [4.2]

$\{asset_{eth}({U}), asset_{token}({U}), asset_{erc20}({U}), asset_{nft}({U}), S({U}), salt({U})\}$ is revealed via layer-1 transaction calldata and used to compute the deposit hash $h = h({U})$.

#### [4.3]

$[{D}_1, ..., {D}_n]$ constructs a mass deposit ${MD}$.

#### [4.4]
Let $merge$ be as defined in [[11.1]](#11.1). Then mass deposit ${MD} = \{merge^{keccak256}([h_1, h_2, ..., h_n]), \Sigma^n_{i=1} f_i \}$ where $h_i = h({U}_i)$ and ${D}_i = \{{U}_i, f_i\}$
#### [4.4]

A deposit ${D}$ should transfer defined assets from the message sender to the Zkopru contract and update the latest ${MM}$.

#### [4.5]

Deposits can be merged only into the latest mass deposit.

#### [4.6]

Mass deposit should be frozen to be included in the layer-2 blocks.


## 5. Withdrawal

#### [5.1]
A withdrawal ${W} = \{{U}, f\}$ where ${U}$ is a UTXO to withdraw out to the layer 1 and $f$ is the fee for pre-payer for the instant withdrawal.

#### [5.2]
When ${W} = \{{U}, f\}$, ${U}$ should be created by a transaction that has output type $t({U})=1$ as defined in [[3.3.9]](#3.3.9).

#### [5.3]

$\{asset_{eth}({U}), asset_{token}({U}), asset_{erc20}({U}), asset_{nft}({U}), S({U}), salt({U})\}$ is public data as defined in [[3.3.7]](#3.3.7).

#### [5.4]

Withdrawals $[{W}_1, ..., {W}_n]$


Let a mass migration ${MM} = \{dest, asset, {MD}\}$, then $[{M_1}, ..., {M_n}]$ then $dest({MM}) = dest({M_i}) = dest({M_j})$ where $(i, j) \in [1, ..., n]$.


#### [5.5]

If a block includes mass migrations $[{MM_1}, ..., {MM_n}]$, $dest({MM_i}) \neq dest({MM_j})$ where $(i, j) \in [1, ..., n]$


#### [5.6]

Every migration output becomes a new deposit for its destination network and the set of migration that has same $dest$ address becomes the mass deposit for its destination network. Therefore, a mass migration ${MM} = \{dest, asset, {MD}\}$ has $dest$, the address of destination network, $asset$, the total amount of asset including Ether, ERC20 and NFTs, and ${MD}$.

#### [5.7]

$asset=\{asset_{eth}({MD}), [\{addr_1, amount_1\},...,\{addr_m, amount_m\}], [\{addr_1, [nft^{(1)}_1, ..., nft^{(1)}_{k_1}]\},...,\{addr_n, [nft^{(n)}_1, ..., nft^{(n)}_{k_n}]\}] \}$ should have exactly same amount of the sum of all ${U}$s that are included in the ${MD}$ of the mass migration.


## 6. Migration

#### [6.1]

A migration ${M} = \{{U}, f, dest\}$ where ${U}$ is a UTXO to be used in destination network and $f$ is fee for the block proposer of the destination network and $dest$ is the contract address of the destination network.

#### [6.2]
When ${M} = \{{U}, f, dest\}$, ${U}$ should be created by a transaction that has output type $t({U})=2$ as defined in [[3.3.10]](#3.3.10).

#### [6.3]

$\{asset_{eth}({U}), asset_{token}({U}), asset_{erc20}({U}), asset_{nft}({U}), S({U}), salt({U})\}$ is public data as defined in [[3.3.7]](#3.3.7).


#### [6.4]

$[{M_1}, ..., {M_n}]$ constructs a mass migration ${MM}$

#### [6.5]

${MM} = \{dest, asset, {MD}^{'}\}$ where $dest$ is the address of the destination network and ${MD}^{'}$ becomes the mass deposit for the destination network. ${MM}$ should transfer $asset$ from Zkopru contract to the $dest$ contract when it is finalized by the block finalization logic.
#### [6.6]

Let $merge$ be as defined in [[11.1]](#11.1). Then, ${MD}^{'} = \{merge^{keccak256}([h_1, h_2, ..., h_n]), \Sigma^n_{i=1} f_i \}$  where $h_i = h({U}_i), {M}_i = \{{U}_i, f_i\}$

#### [6.7]

$dest = dest({M_i}) = dest({M_j})$ where $(i, j) \in [1, ..., n]$.


#### [6.8]

If a block includes mass migrations $[{MM_1}, ..., {MM_n}]$, $dest({MM_i}) \neq dest({MM_j})$ where $(i, j) \in [1, ..., n]$

#### [6.9]

$asset_{eth}(MM) = \Sigma_{i = 0}^{n}eth(M)$
$asset^{(tokenA)}_{erc20}(MM) = \Sigma_{i = 0}^{n}tokenA(M)$
$asset^{(tokenB)}_{erc721}(MM) = [nft_1, ..., nft_k]$
==need more detail explanations==

## 7. Tree

### 7.1 Constants

#### [7.1.1] 

$C_{utxo-tree-depth}$, $C_{nullifier-tree-depth}$, and $C_{withdrawal-tree-depth}$ are constants and written on the smart contract.

#### [7.1.2] 

$C_{utxo-subtree-depth}$ and $C_{withdrawal-subtree-depth}$ are constants and written on the smart contract.

### 7.2 Sparse Merkle Tree

#### [7.2.1]

Let $(root_{next}, index_{next}, siblings_{next}) = smtAppend(h, root_{prev}, index_{prev}, siblings_{prev}, leaf)$ where
$h$: hash function to calculate branch node.
$root$: The root node value of the tree.
$index$: The starting index to start appending leaves.
$leaf$: An item to append to the tree.

#### [7.2.2]

Then, $merkleProof(h, index_{prev}, 0, siblings_{prev}, root_{prev}) = 1$

#### [7.2.3]

$merkleProof(h, index_{prev}, leaf, siblings_{prev}, root_{next}) = 1$

#### [7.2.4]

$merkleProof(h, index + 1, 0, siblings_{next}, root_{next}) = 1$

### 7.3 Batch append

Let $(root_{next}, index_{next}, siblings_{next}) = smtBatchAppend(h, root_{prev}, index_{prev}, siblings_{prev}, leaves)$where
$h$: hash function to calculate branch node.
$root_{prev}$: The root node value of the tree.
$index_{prev}$: The starting index to start appending leaves.
$leaves$: $[leaf_1, ..., leaf_n]$


#### [7.3.1]

Let $smtAppend$ be defined in [[7.2.1]](#7.2.1). Then,
$(root_{i+1}, index_{i+1}, siblings_{i+1}) = smtAppend(h, root_{i}, index_{i}, siblings_{i}, leaf_{i})$  where $root_1 = root_{prev}$ and $root_{n+1} = root_{next}$.

For $i \in [1, ..., n]$

#### [7.3.2]

$merkleRoot(h, index_{i}, 0, siblings_{i}) = root_{i}$

#### [7.3.3]

$merkleRoot(h, index_{i}, leaf_{i}, siblings_{i}) = root_{i+1}$

#### [7.3.4]

$merkleRoot(h, index_{i+1}, 0, siblings_{i+1}) = root_{i+1}$

### 7.4 Sub tree append

Let a tree has $d_{SMT}$ for its depth and $d_{sub}$ for its subtree's depth. And Let $(root_{next}, index_{next}, siblings_{next}) = smtSubtreeAppend(h, root_{prev}, index_{prev}, siblings_{prev}, leaves)$.

#### [7.4.1]

$d_{sub} < d^{SMT}$

#### [7.4.2]

Let $leaves = [item_1, ..., item_n] = [[sub^{(1)}_1, ..., sub^{(1)}_{n_1}], [sub^{(2)}_1, ..., sub^{(2)}_{n_2}], ..., [sub^{(k)}_1, ..., sub^{(k)}_{n_k}]]$, then 

$n_1 = n_2 = ... = n_{k-1} = 2^{d_{sub}}$
and
$n_{k} = n - 2^{d_{sub}} \times (k - 1)$

#### [7.4.3]

$root_{sub^{(i)}} = merkleRoot(sub^{(i)}_1, ..., sub^{(i)}_{n_i})$
$root_{sub^{(k)}} = merkleRoot(sub^{(k)}_1, ..., sub^{(k)}_{n_k}, \underbrace{0, ..., 0}_{(2^{d_{sub}} - n_k)})$

#### [7.4.4]

Then, $merkleProof(h, index, 0, siblings, root) = 1$

#### [7.4.5]

Let
$siblings' = [sib_1, sib_2, ..., sib_{d_{SMT} - d_{sub}}]$ when $siblings = [sib_1, sib_2, ..., sib_{d_{SMT}}]$
$subRoots = [root_{sub^{(1)}}, ..., root_{sub^{(k)}}]$
$index' = index << d_{sub}$ and $index = index' >> d_{sub}$
then,

#### [7.4.6]

$(root_{next}, index'_{next}, siblings'_{next}) = smtBatchAppend(h, root_{prev}, index'_{prev}, siblings'_{prev}, subRoots)$

### 7.5 UTXO Tree

#### [7.5.1]

$Tree_{U}$ is a sparse merkle tree to store UTXO leaves which $depth$ is $C_{utxo-tree-depth}$.

#### [7.5.2]

Let $l$ is a left child node and $r$ is a right child node of a branch node $b$ of $Tree_{U}$. Then, $b = poseidon_2(l, r)$

#### [7.5.3]

Let ${B}_n$ is the $n$-th block and its body contains $[{MD}_1, ..., {MD}_m]$ and $[{TX}_1, ..., {TX}_n]$. Then,

$\mathbb{O} = [{O^{{TX}_1}_1}, ..., {O^{{TX}_1}_{k_1}}] \cup [{O^{{TX}_2}_1}, ..., {O^{{TX}_2}_{k_2}}] \cup \cdots \cup [{O^{{TX}_m}_1}, ..., {O^{{TX}_m}_{k_m}}]$

$\mathbb{U}^{deposit} = [{U^{{MD}_1}_1}, ..., {U^{{MD}_1}_{k_1}}] \cup [{U^{{MD}_2}_1}, ..., {U^{{MD}_2}_{k_2}}] \cup \cdots \cup [{U^{{MD}_m}_1}, ..., {U^{{MD}_m}_{k_m}}]$
$\mathbb{U}^{tx} = \{ O | O \in \mathbb{O}, t(O) = 0\}$.

#### [7.5.4]


Let $leaves_U = \mathbb{U}^{deposit} \cup \mathbb{U}^{tx}$, then
$(root^{(n+1)}_U, index^{(n+1)}_U, siblings^{(n+1)}_U) = smtSubtreeAppend(poseidon_2, root^{(n)}_U, index^{(n)}_U, siblings^{(n)}_U, leaves)$

### 7.6 Withdrawal Tree

#### [7.6.1]

$Tree_{W}$ is a sparse merkle tree to store Withdrawal outputs which $depth$ is $C_{utxo-tree-depth}$.

#### [7.6.2]

Let $l$ is a left child node and $r$ is a right child node of a branch node $b$ of $Tree_{W}$. Then, $b = keccak256(l, r)$

#### [7.6.3]

Let ${B}_n$ is the $n$-th block and its body contains $[{TX}_1, ..., {TX}_n]$. Then,

$\mathbb{O} = [{O^{{TX}_1}_1}, ..., {O^{{TX}_1}_{k_1}}] \cup [{O^{{TX}_2}_1}, ..., {O^{{TX}_2}_{k_2}}] \cup \cdots \cup [{O^{{TX}_m}_1}, ..., {O^{{TX}_m}_{k_m}}]$
$\mathbb{W} = \{ O | O \in \mathbb{O}, t(O) = 1\} = \{W_1, ..., W_{n_W}\}$

#### [7.6.4]

Let $leaves_W = [W_1, ..., W_{n_W}]$, then
$(root^{(n+1)}_W, index^{(n+1)}_W, siblings^{(n+1)}_W) = smtSubtreeAppend(keccak256, root^{(n)}_W, index^{(n)}_W, siblings^{(n)}_W, leaves_W)$

### 7.7 Nullifier Tree

#### [7.7.1]

$Tree_{N}$ is a sparse merkle tree to store nullifiers which $depth$ is $C_{nullifier-tree-depth}$.

#### [7.7.2]

$2^{C_{nullifier-tree-depth}} > size(\mathbb{F})$

#### [7.7.3]

Let $l$ is a left child node and $r$ is a right child node of a branch node $b$ of $Tree_{N}$. Then, $b = keccak256(l, r)$

#### [7.7.4]

Let ${B}_n$ is the $n$-th block and its body contains $[{TX}_1, ..., {TX}_n]$. Then,

$\mathbb{Nullifiers} = \{ nullifier(U) | U \in \mathbb{U^{tx}}\}$

#### [7.7.5]

Let $(root_{next}) = nullify(h, root_{prev}, index, siblings)$ where
$h$: hash function to calculate branch node.
$root$: The root node value of the tree.
$index$: The starting index to start appending leaves.

then for every $N \in \mathbb{Nullifiers}$,
#### [7.7.6]

$merkleProof(keccak256, N, 0, siblings, root_{prev}) = 1$

#### [7.7.7]

$merkleProof(keccak256, N, C_{exist}, siblings, root_{next}) = 1$

## 8. Block

#### [8.1]

The $n$-th block $B^{(n)} = \{Header^{(n)}, Body^{(n)}\}$

#### [8.2]

$Header^{(n)} = \{proposer^{(n)},  h(B^{(n-1)}), fee^{(n)}, root^{(n)}_U, index^{(n)}_U, root^{(n)}_N, root^{(n)}_W, index^{(n)}_W, \\ \ merkleRoot(\mathbb{Tx^{(n)}}), merkleRoot(\mathbb{MD^{(n)}}), , merkleRoot(\mathbb{MM^{(n)}})\}$

#### [8.3]

$Body^{(n)} = \{\mathbb{Tx}^{(n)}, \mathbb{MM}^{(n)}, \mathbb{MD}^{(n)}\}$ where
$\mathbb{Tx}^{(n)} = [TX^{(n)}_1, TX^{(n)}_2, ..., TX^{(n)}_{L_{\mathbb{Tx}}}]$
$\mathbb{MD}^{(n)} = [MD^{(n)}_1, MD^{(n)}_2, ..., MD^{(n)}_{L_{\mathbb{MD}}}]$
$\mathbb{MM}^{(n)} = [MM^{(n)}_1, MM^{(n)}_2, ..., MM^{(n)}_{L_{\mathbb{MM}}}]$

#### [8.4]

$fee^{(n)} = \Sigma_{i = 0}^{L_{\mathbb{Tx}}} fee(TX^{(n)}_{i}) + \Sigma_{i = 0}^{L_{\mathbb{MD}}} fee(MD^{(n)}_{i}) + \Sigma_{i = 0}^{L_{\mathbb{MM}}} fee(MM^{(n)}_{i})$

## 9. Serialization

### 9.1. Transaction

#### [9.1.1]

The first byte of Zk transaction data is the length of the input notes.

#### [9.1.2]

If the parsed length from [[9.1.1]](#9.1.1) is $m$, the next $64 \cdot m$ bytes from [[9.1.1]](#9.1.1) are the concatenation of $[(root_{UTXO}^{k_1}, nullifier({U^{in}}_1)), ..., (root_{UTXO}^{k_m}, nullifier({U^{in}}_m))]$ where $N_{latest} - c_{ref}\leq k < N_{latest}$. here, $N_{latest}$ is the latest block number and $c_{ref}$ is defined in [[11.4]](#11.4).

#### [9.1.3]

The next byte right after [[9.1.2]](#9.1.2) is the length of the output notes.

#### [9.1.4]

If the parsed length from [[9.1.3]](#9.1.3) is $n$, the next bytes from [[9.1.3]](#9.1.3) should be the concatenation of $[data({O}_1), ..., data({O}_n)]$

<div style="padding-left: 5rem">

#### [9.1.4.1]

The first 32 bytes of $data({O}_i)$ is the $h({O}_i)$

#### [9.1.4.2]

The next 1 bytes from [[9.1.4.1]](#9.1.4.1) is $t({O}_i) \in [0, 1, 2]$

#### [9.1.4.3]

If $t({O}_i)$ is $0$, ${O}_i$ is a UTXO type and [[9.1.4.2]](#9.1.4.2) is the end of the data.

#### [9.1.4.4]

If $t({O}_i)$ is $1$, ${O}_i$ is a withdrawal type and has 168 bytes of public data $pd({O}_i)$.


<div style="padding-left: 5rem">

#### [9.1.4.4.1]

The first 20 bytes of $pd({O}_i)$ is $data_{to}({O}_i)$ which means the Ethereum address of the withdrawal destination. That address should be able to make the withdrawal transaction.


#### [9.1.4.4.2]

The next 32 bytes from [[9.1.4.4.1]](#9.1.4.4.1) is $data_{eth}({O}_i)$ which means the amount of Ether to withdraw.

#### [9.1.4.4.3]

The next 20 bytes from [[9.1.4.4.2]](#9.1.4.4.2) is $data_{token}({O}_i)$ which means the address of the token if it contains ERC20 tokens or an NFT.

#### [9.1.4.4.4]

The next 32 bytes from [[9.1.4.4.3]](#9.1.4.4.3) is $data_{erc20}({O}_i)$ which means the amount of the token to withdraw if $data_{token}({O}_i)$ is registered as an ERC20 token on the contract. Or it should be zero.


#### [9.1.4.4.5]

The next 32 bytes from [[9.1.4.4.4]](#9.1.4.4.4) is $data_{nft}({O}_i)$ which means the NFT id to withdraw if $data_{token}({O}_i)$ is registered as an ERC721 token on the contract. Or it should be zero.

:::warning
Note that Zkopru does not support NFT id 0.
:::

#### [9.1.4.4.6]

The next 32 bytes from [[9.1.4.4.5]](#9.1.4.4.5) is $data_{fee}({O}_i)$ which means the additional fee for the pre-payer when the withdrawer wants to withdraw the note instantly.

</div>

#### [9.1.4.5]

If $t({O}_i)$ is $2$, ${O}_i$ is a migration type and has 168 bytes of public data $pd({O}_i)$.


<div style="padding-left: 5rem">

#### [9.1.4.5.1]

The first 20 bytes of $pd({O}_i)$ is $data_{to}({O}_i)$ which means the destination contract address of the migration. That address should have `acceptMigration(bytes32, bytes32, uint256)` function to support migration.


#### [9.1.4.5.2]

The next 32 bytes from [[9.1.4.5.1]](#9.1.4.5.1) is $data_{eth}({O}_i)$ which means the amount of Ether to migrate.

#### [9.1.4.5.3]

The next 20 bytes from [[9.1.4.5.2]](#9.1.4.5.2) is $data_{token}({O}_i)$ which means the address of the token if it contains ERC20 tokens or an NFT.

#### [9.1.4.5.4]

The next 32 bytes from [[9.1.4.5.3]](#9.1.4.5.3) is $data_{erc20}({O}_i)$ which means the amount of the token to migrate if $data_{token}({O}_i)$ is registered as an ERC20 token on the contract. Or it should be zero.

#### [9.1.4.5.5]

The next 32 bytes from [[9.1.4.5.4]](#9.1.4.5.4) is $data_{nft}({O}_i)$ which means the NFT id to migrate if $data_{token}({O}_i)$ is registered as an ERC721 token on the contract. Or it should be zero.

:::warning
Note that Zkopru does not support NFT id 0.
:::

#### [9.1.4.5.6]

The next 32 bytes from [[9.1.4.5.5]](#9.1.4.5.5) is $data_{fee}({O}_i)$ which means the fee for the destination contract's block proposer.

</div>

</div>

#### [9.1.5]

The next 32 bytes after [[9.1.4]](#9.1.4) is the transaction fee for the block proposer.

#### [9.1.6]

The next 256 bytes after [[9.1.5]](#9.1.5) is $proof({TX}) = (a, b, c)$ which is the concatenation of $[a.x, a.y, b.x_1, b.x_2, b.y_1, b.y_2, c.x, c.y]$.

#### [9.1.7]

The next byte after [[9.1.6]](#9.1.6) is the indicator of extra data.

#### [9.1.8]

If the indicator that is parsed from [[9.1.7]](#9.1.7) has 1 on its right most bit position, the next 32 bytes after [[9.1.7]](#9.1.7) is $h({O_{swap}})$ which should be exists in its paired transaction's output notes.

#### [9.1.9]

If the indicator that is parsed from [[9.1.7]](#9.1.7) has 1 on its second right most bit position, the next 81 bytes after [[9.1.8]](#9.1.8) is $memo({TX})$.


<div style="padding-left: 5rem">

#### [9.1.9.1]

The first 32 bytes of $memo({TX})$ is the ephemeral public key $E = e \cdot G$.

#### [9.1.9.2]

The next 49 bytes after [[9.1.9.1]](#9.1.9.1) is the encrypted data using the shared key $SK = n \cdot E = e \cdot N$ where $n$ is the private viewing key and $N$ is the recipient's public viewing key.

#### [9.1.9.3]

The first byte of the cipher text is $data_{token}({O_{recipient}}) (mod\ 256)$

#### [9.1.9.4]

The next 16 bytes of the cipher text after [[9.1.9.3]](#9.1.9.3) is $salt({O_{recipient}})$

#### [9.1.9.5]

The next 32 bytes of the cipher text after [[9.1.9.4]](#9.1.9.4) is $asset_{eth}({O_{recipient}})$ or $asset_{erc20}({O_{recipient}})$ or $asset_{nft}({O_{recipient}})$

</div>

### 9.2 Mass Deposit

#### [9.2.1]

The first 32 bytes of mass deposit data is 

#### [9.2.2]

The next 32 bytes after [[9.2.1]](#9.2.1) is 
### 9.3 Mass Deposit

#### [9.2.1]

#### [9.2.1]

The first 32 bytes of mass deposit data is $h(h(h(h(h(0, d_1), d_2)...), d_{n-1}), d_n)$, which is the merged leaves, where ${MD} = [d_1, d_2, ..., d_n] = [h({U^{deposit}}_1), h({U^{deposit}}_2),..., h({U^{deposit}}_n)]$

#### [9.2.2]

The next 32 bytes after [[9.2.1]](#9.2.1) is the accumulated fee for the block proposer that is $fee({MD}) = \Sigma fee({Deposit})$.


## 10. Optimistic rollup

### 10.1 Block proposal

#### [10.1.1]

When a block $B^{(n)}$ is submitted to the layer-1 chain, the proposal transaction succeeds when only

<div style="padding-left: 5rem">

#### [10.1.1.1]

Smart contract returns `true` for `proposable(proposer)`  when `proposer` is the address of $proposer^{(n)}$

#### [10.1.1.2]

The transaction data size is less than $C_{max-block-size}$.

#### [10.1.1.3]

$proposer^{(n)}$ equals the address of the transaction signer.

#### [10.1.1.4]

The checksum hash of the block data should be submitted once.

#### [10.1.1.5]

$stake(proposer)$ means the staked deposit of $proposer$ for the challenge system, then 
$stake(proposer^{(n)}) \geq C_{minimum-stake}$

</div>

#### [10.1.2]

And the proposal transaction records following data on-chain

<div style="padding-left: 5rem">

#### [10.1.2.1]

Let $exit(proposer)$ is the allowed timestmap to withdraw staked asset, then $exit(proposer^{(n)}) = timestamp(B_{L1}^{(n)}) + C_{challenge-period}$

#### [10.1.2.2]

New block proposal freezes the staged mass deposit by [[10.1.2.5]](#10.1.2.5).

#### [10.1.2.3]

New block proposal emits an event `NewProposal(uint256, bytes32)` log to notify the data to layer 2 node operators.

#### [10.1.2.4]

New block proposal increments the number of proposals `proposedBlocks`.

</div>

#### [10.1.2.5]

Anyone can freeze the staged mass deposit if it exists. Then the hash of the staged mass deposit is recorded on the smart contract and gets frozen not to allow any updates.

### 10.2 Challenge: deposit validation

#### [10.2.1]

Let $MD^{(n)}_i$ is the $i$-th mass deposit of block $B^{(n)}$, then challenge succeeds when the hash of mass deposit $h(MD^{(n)}_i)$ is not recorded on the contract by [[10.1.2.5]].

### 10.3 Challenge: header validation

#### [10.3.1]

A challenge succeeds when the deserialized block data $B^{(n)}$ does not satisfy [[8.2]](#8.2)

#### [10.3.2]

A challenge succeeds when the deserialized block data $B^{(n)}$ does not have correct fee by [[8.4]](#8.4)

### 10.4 Challenge: migration validation

#### [10.4.1]

A challenge succeeds when it violates [[6.8]](#6.8)

#### [10.4.2]

A challenge succeeds when it violates [[6.9]](#6.9)

### 10.5 Challenge: nullifier validation

#### [10.5.1]

A challenge succeeds when any nullifier from the block does not update the root of nullifier tree.

### 10.6 Challenge: transaction validation

#### [10.6.21

A challenge succeeds when SNARK failes.

#### [10.6.2]

A challenge succeeds when the root reference is not finalized or is too out-dated.

#### [10.6.3]

A challenge succeeds when any nullifier from the block does not update the root of nullifier tree.

### 10.7 Challenge: nullifier validation

#### [10.7.1]

A challenge succeeds when any nullifier from the block does not update the root of nullifier tree.

### 10.7 Challenge: withdrawal validation

#### [10.7.1]

A challenge succeeds when it violates [[7.6.1]](#7.6.1) or [[7.6.2]](#7.6.2) or [[7.6.3]](#7.6.3) or [[7.6.4]](#7.6.4).

### 10.8 Challenge: utxo validation

#### [10.8.1]

A challenge succeeds when it violates [[7.5.1]](#7.5.1) or [[7.5.2]](#7.5.2) or [[7.5.3]](#7.5.3) or [[7.5.4]](#7.5.4).

## 11. Misc

#### [11.1]

$merge^{hash}([a_1, a_2, ..., a_n]) = hash(hash(hash(0, a_1), a_2)..., a_n)$

#### [11.2]

==merkleProof==

#### [11.3]

==merkleRoot==

#### [11.4]

Constant $c_{ref}$ is the number of blocks from the latest block that is allowed to be referenced in Zkopru transactions.
