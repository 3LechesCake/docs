# What Is a Dropped Transaction?

A dropped transaction is a transaction that was never mined. The transaction was broadcast to the network but dropped from the connected nodes. Dropped transactions typically occur for a number of reasons:

* The connected nodes drop a pending transaction from the transaction pool (usually due to a lower gas price compared to other pending transactions)
* A node can only have so many pending transactions before certain transactions are dropped from its pending pool (network dependent)
* The gas fee provided for the transaction was too low
* The transaction would fail if it got mined

When a transaction is dropped, it is no longer broadcasted by the network nodes. As a result, it won’t be processed and is not recorded on the blockchain. Assets and gas fees used in dropped transactions are returned to the sending address.

## Replaced Transactions

Dropped transactions can theoretically still be mined until they are replaced. Dropped transactions are replaced when another transaction using the same nonce is broadcasted and confirmed by the network. A nonce indicates the number of transactions made by an address. It starts at 0 and increases by 1 with every transaction. BlockWallet automatically resets the nonce to the nonce used during the dropped transaction, making replaced transactions possible.\
\
Replaced transactions are typically broadcasted for the following reasons:

* The initial transaction had a low gas price and would take a long time to confirm, so users initiate a second transaction with a higher gas price to replace it
* The node was not fully synced, resulting in an incorrect nonce being used
* Initiated by users to replace or cancel a pending transaction

As dropped transactions are not recorded on the blockchain, another transaction is initiated with the previous nonce value. This means that the second transaction has replaced the dropped transaction by taking its position in the nonce sequence.&#x20;
