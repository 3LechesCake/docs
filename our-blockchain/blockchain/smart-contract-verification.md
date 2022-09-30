# Smart Contract Verification

Smart Contract VerificationDetails about the verification microserviceA microservice written in Rust provides fast and efficient contract verification. The application runs as an HTTP server and verification requests are made using REST API.

* ​[Basic Info](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#basic-info)​
* ​[Library Initialization](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#library-initialization)​
* ​[Solidity and Vyper Verfication](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#solidity-and-vyper-verification)​
* ​[Sourcify Verification](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#sourcify-verification)​
* ​[Verification Algorithm](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#verification-algorithm)​
* ​[Http API](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#verification-http-api)​

## Basic Info <a href="#basic-info" id="basic-info"></a>

### **Location** <a href="#location" id="location"></a>

* Application: [https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier](https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier)​
* Http API: [https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier-http](https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier-http)​

In order to succesffully intall and configure the servie docker and docker compose are needed.

{% embed url="https://linuxhostsupport.com/blog/how-to-install-and-configure-docker-compose-on-ubuntu-20-04/" %}

Service Configuration

```

[Unit]
Description=smart-contract-verifier-http
Requires=docker.service
After=docker.service

[Service]
Restart=10
User=root
Group=docker
WorkingDirectory=/usr/local/blockscout-rs/smart-contract-verifier-http
# Shutdown container (if running) when unit is started
ExecStartPre=/usr/bin/docker-compose -f docker-compose.yaml down
# Start container when unit is started
ExecStart=/usr/bin/docker-compose -f /usr/local/blockscout-rs/smart-contract-verifier-http/docker-compose.yaml up
# Stop container when unit is stopped
ExecStop=/usr/bin/docker-compose -f docker-compose.yaml down

[Install]
WantedBy=multi-user.target


```

### **Activation** <a href="#activation" id="activation"></a>

* Update Blockscout to the 4.1.8 release
* Add the following ENV variables:
  * `ENABLE_RUST_VERIFICATION_SERVICE=true`
  * `RUST_VERIFICATION_SERVICE_URL=...` (verifier endpoint, see [http configuration](https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier-http) for more details)
* If running locally, the smart-contract-verifier is integrated with [`docker-compose`](https://github.com/blockscout/blockscout/tree/master/docker-compose)`​`

### Architecture <a href="#architecture" id="architecture"></a>

The service consists of 2 parts, a verification library and a transport layer that serves requests.

* **Verification Library**: Provides verification and a communication interface for verification functions. Includes modules for Solidity, Sourcify, and Vyper.
* **Transport Layer**: Verification requests are sent via HTTP.

### Benefits <a href="#benefits" id="benefits"></a>

We've seen a 10X improvement in verification speed over the previous implementation. Benefits of the new microservice include:

* **Scalability**: Remove CPU intensive tasks from the primary Blockscout instance. Modularity provides the ability to scale quickly.
* **Speed**: Using native compilers greatly increases compilation speed relative to the JS wrapped versions running on Node.js used previously.
* **Improved efficiency**. New algorithm supports contracts that contain several metadata hashes, improving efficiency.

## Library Initialization <a href="#library-initialization" id="library-initialization"></a>

On startup, the library must retrieve all available compilers. Compiler information consists of a link to download the compiler and its hashsum \[[example](https://github.com/blockscout/solc-bin/blob/main/list.json)]. A separate thread also starts during initialization to update this list periodically.Library processes differ based on verification for Solidity/Vyper contracts or verification via Sourcify. Both are detailed below.

## Solidity and Vyper Verification <a href="#solidity-and-vyper-verification" id="solidity-and-vyper-verification"></a>

Verification processes are similar for contracts written in Solidity and Vyper.For Solidity contracts, verification is available for multiple files or standard json input. Single file verification is also possible - users still verify via the multiple file method and only upload a single file.For Vyper contracts, only multiple file verification is available.

### Request Values <a href="#request-values" id="request-values"></a>

Each request contains the following values:

* **Deployed bytecode**: bytecode of the contract stored on chain
* **Creation transaction (tx) input**: input data provided in transaction to create the contract
* **Compiler version**: Version of compiler used to compile the contract
* **Content**: Verification method related fields. E.g. for Solidity multiple files those fields are:
  * **Sources**: mapping from source files to their contents
  * **Evm version**: version of the EVM used to compile sources
  * **Optimization runs**: optional parameter that enables optimizations and specifies number of optimization runs
  * **Contract libraries**: optional parameter that specifies addresses of the contract libraries used in compiled contracts

### Verification Steps <a href="#verification-steps" id="verification-steps"></a>

1. 1.**Download the compiler** using the specified compiler version from the url retrieved on [initialization](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#initialization) (or update).
2. 2.**Compilation**: compile using the provided and slightly modified (see [Verification Algorithm](https://docs.blockscout.com/for-developers/information-and-settings/smart-contract-verification#verification-algorithm) section) sources.
3. 3.**Verification**: compare compiler output from step 2 with bytecodes (deployed bytecode and creation tx input) provided by the caller. During verification all contracts obtained through local compilation are checked one by one, and if any corresponds to provided bytecodes, the verification is considered successful. Contract paths and names are already indexed from available contracts, and constructor arguments are extracted.
4. 4.**Return Values**: See below.

### Returned Values <a href="#returned-values" id="returned-values"></a>

As a result of successful verification the following struct is returned:

* **Compiler input**: Data used as an input for the compiler
* **Compiler version**: version of the compiler used in verification (corresponds the the version provided in request)
* **File path**: path of the file containing verified contract
* **Contract name**: name of the verified contract
* **Abi**: application binary interface of the verified contract ([https://docs.soliditylang.org/en/v0.8.13/abi-spec.html?highlight=abi](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html?highlight=abi))
* **Constructor arguments**: optionally returned constructor arguments used during on chain creation of the contract.

In case of an error, enum is returned with the corresponding error and its description (e.g. if compilation failed, a Compilation error is returned with the error data obtained from the compiler).

## Sourcify Verification <a href="#sourcify-verification" id="sourcify-verification"></a>

The request is sent via proxy to the [Sourcify](https://sourcify.dev/) service. It accepts the following arguments:

1. 1.Address of the contract to be verified.
2. 2.Chain id where the contract is deployed.
3. 3.Files required for Sourcify verification.
4. 4.(optionally)Contract index, if the provided files contain multiple metadata files (i.e. multiple contracts).

These arguments are sent to the Sourcify Api, which returns the verification status.

### **Successful Verification** <a href="#successful-verification" id="successful-verification"></a>

Either the data provided was valid and the contract was verified by Sourcify, or the contract was previously verified. To obtain the actual data used for verification we send another request to retrieve source files used in verification. We expect data used for verification to be in the `metadata.json` file, so it is parsed and the information retrieved. This includes file and contract names, compiler and evm versions, info about optimizations, contract libraries, abi, and sources.**Note**: Constructor arguments cannot currently be obtained from the metadata via Sourcify. This requires the input used for contract creation transaction and the bytecode resulting from local compilation, which are not provided.

### Failed Verification <a href="#failed-verification" id="failed-verification"></a>

If verification fails we return an error with description obtained from Sourcify response

## Verification Algorithm <a href="#verification-algorithm" id="verification-algorithm"></a>

Verification is divided into 2 parts to effectively deal with metadata hash differences in contracts. Often, very similar contracts will have very different metadata hashes based on small changes within the contract (ie an extra space in the source code). To address this, the algorithm:

1. 1.Extracts all metadata hash fields from c_reationTxInput_.
2. 2.Compares non-metadata sections of the onchain bytecode with the results of local compilation.

Metadata hashes are extracted by compiling the same contract twice (with a slight modification to the 2nd contract that does not change any other aspect of the resultant bytecode). The resulting c_reationTxInput_s are compared and metadata hashes extracted by finding which parts of the inputs differ.

### Example <a href="#example" id="example"></a>

An additional space in the source code can be used for the modification process. In **Example 1** and **Example 2** the only difference between both contracts is a space in the beginning of **Example 2**. However, this still completely changes the metadata hash (metadata hash differences shown in **bold red**).**Note**: To update the metadata hash in production we use an additional library passed into the compiler. It does not effect the resultant bytecode, but the compilation input is altered, leading to updated metadata and an updated metadata hash.

#### Example 1 <a href="#example-1" id="example-1"></a>

// pragma solidity ^0.8.7; contract Main {}_creationTxInput:_ 6080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfea264697066735822**12203cc19b771ee83d8cc6995191bd9092a82f3e66b95aacaf1cfc893d3dc27abcd2**64736f6c63430008070033

#### Example 2 <a href="#example-2" id="example-2"></a>

// pragma solidity ^0.8.7; contract Main {}_creationTxInput:_ 6080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfea264697066735822**1220b3902f5797f05bb62858c8d41f09402093c85957c4ae1d773e4eb1cb2000f093**64736f6c63430008070033

### Verification Process <a href="#verification-process" id="verification-process"></a>

1. 1.Compile the original contracts using data provided by the caller then compile those contracts again while passing in the additional library. The result is two slightly different `CreationTxInputs` if there is any encoded metadata.
2. 2.Separate the `MainPart` and the `MetadataPart`.
   1. 1.`MainPart` consists of consecutive bytes which are not related to the metadata hash (all bytes are the same for both local c_reationTxInputs_). `MetadataPart` consists of consecutive bytes related to metadata hash (some bytes differ between local c_reationTxInputs_).
3. 3.Iterate through the two local contracts byte by byte to find the first byte that differs.
   1. 1.If there are no such bytes, then there is no metadata hash and the whole _creationTxInput_ consists of just one `MainPart`.
   2. 2.If any differences are found, identify it as a hash section of the metadata hash.
   3. 3.Iterate back byte by byte to find the beginning of that metadata hash. When we find the beginning we know where that metadata hash starts and ends. All non-categorized bytes before the start are identified as the `MainPart` and bytes between the start and end identified as the `MetadataPart`. Continue until there are no uncategorized bytes.
   4. 4.Once the entire local _creationTxInput_ is categorized, we compare those parts with the corresponding remote _creationTxInput_ bytes. For the `MainPart` remote _creationTxInput_ must have exact equivalence. For the `MetadataPart` we compare some parts of the metadata (e.g. that encoded solc versions are the same), but in general we do not require them to be equal.

**Example**_**LocalCreationTxInput1:**_ 608060405234801561001057600080fd5b506040518060200161002190610050565b6020820181038252601f19601f820116604052506000908051906020019061004a92919061005c565b5061015f565b605c806101ac83390190565b8280546100689061012e565b90600052602060002090601f01602090048101928261008a57600085556100d1565b82601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b5b808211156100fb5760008160009055506001016100e3565b5090565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052602260045260246000fd5b6000600282049050600182168061014657607f821691505b602082108103610159576101586100ff565b5b50919050565b603f8061016d6000396000f3fe6080604052600080fdfea2646970667358221220187a584947e37ebca43172929f7e159fe28696b0edd97bbfad5b0a265f6f886964736f6c634300080e00336080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfea2646970667358221220805aa9fe4ec055702024afa5ac21c2104f0b14be0ffab086f0d6e9b5701073f864736f6c634300080e0033**Parts 1:** \[ MainPart(608060405234801561001057600080fd5b506040518060200161002190610050565b6020820181038252601f19601f820116604052506000908051906020019061004a92919061005c565b5061015f565b605c806101ac83390190565b8280546100689061012e565b90600052602060002090601f01602090048101928261008a57600085556100d1565b82601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b5b808211156100fb5760008160009055506001016100e3565b5090565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052602260045260246000fd5b6000600282049050600182168061014657607f821691505b602082108103610159576101586100ff565b5b50919050565b603f8061016d6000396000f3fe6080604052600080fdfe), MetadataPart(a264697066735822**1220187a584947e37ebca43172929f7e159fe28696b0edd97bbfad5b0a265f6f8869**64736f6c634300080e0033), MainPart(6080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfe), MetadataPart(a264697066735822**1220805aa9fe4ec055702024afa5ac21c2104f0b14be0ffab086f0d6e9b5701073f8**64736f6c634300080e0033) ]​_**LocalCreationTxInput2:**_ 608060405234801561001057600080fd5b506040518060200161002190610050565b6020820181038252601f19601f820116604052506000908051906020019061004a92919061005c565b5061015f565b605c806101ac83390190565b8280546100689061012e565b90600052602060002090601f01602090048101928261008a57600085556100d1565b82601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b5b808211156100fb5760008160009055506001016100e3565b5090565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052602260045260246000fd5b6000600282049050600182168061014657607f821691505b602082108103610159576101586100ff565b5b50919050565b603f8061016d6000396000f3fe6080604052600080fdfea26469706673582212202e82fb6222f966f0e56dc49cd1fb8a6b5eac9bdf74f62b8a5e9d8812901095d664736f6c634300080e00336080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfea2646970667358221220bd9f7fd5fb164e10dd86ccc9880d27a177e74ba873e6a9b97b6c4d7062b26ff064736f6c634300080e0033​**Parts 2:**\[ MainPart(608060405234801561001057600080fd5b506040518060200161002190610050565b6020820181038252601f19601f820116604052506000908051906020019061004a92919061005c565b5061015f565b605c806101ac83390190565b8280546100689061012e565b90600052602060002090601f01602090048101928261008a57600085556100d1565b82601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b5b808211156100fb5760008160009055506001016100e3565b5090565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052602260045260246000fd5b6000600282049050600182168061014657607f821691505b602082108103610159576101586100ff565b5b50919050565b603f8061016d6000396000f3fe6080604052600080fdfe),MetadataPart(a264697066735822**12202e82fb6222f966f0e56dc49cd1fb8a6b5eac9bdf74f62b8a5e9d8812901095d6**64736f6c634300080e0033),MainPart(6080604052348015600f57600080fd5b50603f80601d6000396000f3fe6080604052600080fdfe),MetadataPart(a264697066735822**1220bd9f7fd5fb164e10dd86ccc9880d27a177e74ba873e6a9b97b6c4d7062b26ff0**64736f6c634300080e0033)]

#### Constructor arguments <a href="#constructor-arguments" id="constructor-arguments"></a>

Constructor arguments are extracted at the end of the verification process. Constructor arguments are appended at the end of the _creationTxInput_ returned by the compiler. So, to extract these arguments we remove parts presented in local _creationTxInput_, and consider everything else as likely constructor arguments.After that we attempt to map potential constructor args to the constructor abi of the contract. If successful, extracted args are returned as actual constructor arguments.

## Verification Http Api <a href="#verification-http-api" id="verification-http-api"></a>

Currently, verification requests are sent via HTTP requests. This module runs an HTTP server and provides endpoints through which external services (main Blockscout instance, in our case) send verification requests and receive responses. Endpoints provided by the service and configuration details are described in the [corresponding Readme file](https://github.com/blockscout/blockscout-rs/tree/main/smart-contract-verifier-http).Currently only synchronous endpoints are available. In the future a new service that handles asynchronous requests on top of the current smart-contract-verifier may be implemented.
