# Install and Configure your Node

#### Ubuntu via PPAs <a href="#ubuntu-via-ppas" id="ubuntu-via-ppas"></a>

The easiest way to install Geth on Ubuntu-based distributions is with the built-in launchpad PPAs (Personal Package Archives). A single PPA repository is provided, containing stable and development releases for Ubuntu versions `xenial`, `trusty`, `impish`, `focal`, `bionic`.

The following command enables the launchpad repository:

```
sudo add-apt-repository -y ppa:ethereum/ethereum
```

Then, to install the stable version of go-ethereum:

```
sudo apt-get update
sudo apt-get install ethereum
```

Or, alternatively the develop version:

```
sudo apt-get update
sudo apt-get install ethereum-unstable
```

These commands install the core Geth software and the following developer tools: `clef`, `devp2p`, `abigen`, `bootnode`, `evm`, `rlpdump` and `puppeth`. The binaries for each of these tools are saved in `/usr/local/bin/`. The full list of command line options can be viewed [here](https://geth.ethereum.org/docs/interface/command-line-options) or in the terminal by running `geth --help`.

Updating an existing Geth installation to the latest version can be achieved by stopping the node and running the following commands:

```
sudo apt-get update
sudo apt-get install ethereum
sudo apt-get upgrade geth
```

When the node is started again, Geth will automatically use all the data from the previous version and sync the blocks that were missed while the node was offline.

#### Windows <a href="#windows" id="windows"></a>

The easiest way to install Geth is to download a pre-compiled binary from the [downloads](https://geth.ethereum.org/downloads/) page. The page provides an installer as well as a zip file containing the Geth source code. The install wizard offers the user the option to install Geth, or Geth and the developer tools. The installer adds `geth` to the system’s `PATH` automatically. The zip file contains the command `.exe` files that can be run from the command prompt. The full list of command line options can be viewed [here](https://geth.ethereum.org/docs/interface/command-line-options) or in the terminal by running `geth --help`.

Updating an existing Geth installation can be achieved by stopping the node, downloading and installing the latest version following the instructions above. When the node is started again, Geth will automatically use all the data from the previous version and sync the blocks that were missed while the node was offline.

Download treslechesmainnet.json or treslechestestnet.json from github.

get init --datadir data treslechesmainnet.json

