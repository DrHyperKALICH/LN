## Go Lightnetwork

Official golang implementation of the Lightnetwork protocol.

## Building the source


Building geth requires both a Go (version 1.7 or later) and a C compiler.
You can install them using your favourite package manager.
Once the dependencies are installed, run

    make geth

or, to build the full suite of utilities:

    make all

## Executables

The go-Lightnetwork project comes with several wrappers/executables found in the `cmd` directory.

| Command    | Description |
|:----------:|-------------|
| **`geth`** | Our main Lightnetwork CLI client. It is the entry point into the Lightnetwork network (main-, test- or private net), capable of running as a full node (default), archive node (retaining all historical state) or a light node (retrieving data live). It can be used by other processes as a gateway into the Lightnetwork network via JSON RPC endpoints exposed on top of HTTP, WebSocket and/or IPC transports. `geth --help` and the [CLI Wiki page](https://github.com/Lightnetwork/go-Lightnetwork/wiki/Command-Line-Options) for command line options. |
| `abigen` | Source code generator to convert Lightnetwork contract definitions into easy to use, compile-time type-safe Go packages. It operates on plain [Lightnetwork contract ABIs](https://github.com/Lightnetwork/wiki/wiki/Lightnetwork-Contract-ABI) with expanded functionality if the contract bytecode is also available. However it also accepts Solidity source files, making development much more streamlined. Please see our [Native DApps](https://github.com/Lightnetwork/go-Lightnetwork/wiki/Native-DApps:-Go-bindings-to-Lightnetwork-contracts) wiki page for details. |
| `bootnode` | Stripped down version of our Lightnetwork client implementation that only takes part in the network node discovery protocol, but does not run any of the higher level application protocols. It can be used as a lightweight bootstrap node to aid in finding peers in private networks. |
| `evm` | Developer utility version of the LVM (Lightnetwork Virtual Machine) that is capable of running bytecode snippets within a configurable environment and execution mode. Its purpose is to allow isolated, fine-grained debugging of LVM opcodes (e.g. `evm --code 60ff60ff --debug`). |
| `gethrpctest` | Developer utility tool to support our [Lightnetwork/rpc-test](https://github.com/Lightnetwork/rpc-tests) test suite which validates baseline conformity to the [Lightnetwork JSON RPC](https://github.com/Lightnetwork/wiki/wiki/JSON-RPC) specs. Please see the [test suite's readme](https://github.com/Lightnetwork/rpc-tests/blob/master/README.md) for details. |
| `rlpdump` | Developer utility tool to convert binary RLP ([Recursive Length Prefix](https://github.com/Lightnetwork/wiki/wiki/RLP)) dumps (data encoding used by the Lightnetwork protocol both network as well as consensus wise) to user friendlier hierarchical representation (e.g. `rlpdump --hex CE0183FFFFFFC4C304050583616263`). |
| `swarm`    | Swarm daemon and tools. This is the entrypoint for the Swarm network. `swarm --help` for command line options and subcommands. See [Swarm README](https://github.com/Lightnetwork/go-Lightnetwork/tree/master/swarm) for more information. |
| `puppeth`    | a CLI wizard that aids in creating a new Lightnetwork network. |

## Running geth

Going through all the possible command line flags is out of scope here (please consult our
[CLI Wiki page](https://github.com/Lightnetwork/go-Lightnetwork/wiki/Command-Line-Options)), but we've
enumerated a few common parameter combos to get you up to speed quickly on how you can run your
own Geth instance.

### Full node on the main Lightnetwork network

By far the most common scenario is people wanting to simply interact with the Lightnetwork network:
create accounts; transfer funds; deploy and interact with contracts. For this particular use-case
the user doesn't care about years-old historical data, so we can fast-sync quickly to the current
state of the network. To do so:

```
$ geth console
```

This command will:

 * Start geth in fast sync mode (default, can be changed with the `--syncmode` flag), causing it to
   download more data in exchange for avoiding processing the entire history of the Lightnetwork network,
   which is very CPU intensive.
 * Start up Geth's built-in interactive [JavaScript console](https://github.com/Lightnetwork/go-Lightnetwork/wiki/JavaScript-Console),
   (via the trailing `console` subcommand) through which you can invoke all official [`web3` methods](https://github.com/Lightnetwork/wiki/wiki/JavaScript-API)
   as well as Geth's own [management APIs](https://github.com/Lightnetwork/go-Lightnetwork/wiki/Management-APIs).
   This tool is optional and if you leave it out you can always attach to an already running Geth instance
   with `geth attach`.

### Full node on the Lightnetwork test network

Transitioning towards developers, if you'd like to play around with creating Lightnetwork contracts, you
almost certainly would like to do that without any real money involved until you get the hang of the
entire system. In other words, instead of attaching to the main network, you want to join the **test**
network with your node, which is fully equivalent to the main network, but with play-Ether only.

```
$ geth --testnet console
```

The `console` subcommand have the exact same meaning as above and they are equally useful on the
testnet too. Please see above for their explanations if you've skipped to here.

Specifying the `--testnet` flag however will reconfigure your Geth instance a bit:

 * Instead of using the default data directory (`~/.lightnet` on Linux for example), Geth will nest
   itself one level deeper into a `testnet` subfolder (`~/.lightnet/testnet` on Linux). Note, on OSX
   and Linux this also means that attaching to a running testnet node requires the use of a custom
   endpoint since `geth attach` will try to attach to a production node endpoint by default. E.g.
   `geth attach <datadir>/testnet/geth.ipc`. Windows users are not affected by this.
 * Instead of connecting the main Lightnetwork network, the client will connect to the test network,
   which uses different P2P bootnodes, different network IDs and genesis states.
   
*Note: Although there are some internal protective measures to prevent transactions from crossing
over between the main network and test network, you should make sure to always use separate accounts
for play-money and real-money. Unless you manually move accounts, Geth will by default correctly
separate the two networks and will not make any accounts available between them.*

### Configuration

As an alternative to passing the numerous flags to the `geth` binary, you can also pass a configuration file via:

```
$ geth --config /path/to/your_config.toml
```

To get an idea how the file should look like you can use the `dumpconfig` subcommand to export your existing configuration:

```
$ geth --your-favourite-flags dumpconfig
```

*Note: This works only with geth v1.6.0 and above.*


### Programatically interfacing Geth nodes

As a developer, sooner rather than later you'll want to start interacting with Geth and the Lightnetwork
network via your own programs and not manually through the console. To aid this, Geth has built-in
support for a JSON-RPC based APIs ([standard APIs](https://api.lightnetwork.io). These can be
exposed via HTTP, WebSockets and IPC (unix sockets on unix based platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by Geth, whereas the HTTP
and WS interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

  * `--rpc` Enable the HTTP-RPC server
  * `--rpcaddr` HTTP-RPC server listening interface (default: "localhost")
  * `--rpcport` HTTP-RPC server listening port (default: 8545)
  * `--rpcapi` API's offered over the HTTP-RPC interface (default: "eth,net,web3")
  * `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--wsaddr` WS-RPC server listening interface (default: "localhost")
  * `--wsport` WS-RPC server listening port (default: 8546)
  * `--wsapi` API's offered over the WS-RPC interface (default: "eth,net,web3")
  * `--wsorigins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: "admin,debug,eth,miner,net,personal,shh,txpool,web3")
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect
via HTTP, WS or IPC to a Geth node configured with the above flags and you'll need to speak [JSON-RPC](http://www.jsonrpc.org/specification)
on all transports. You can reuse the same connection for multiple requests!

**Note: Please understand the security implications of opening up an HTTP/WS based transport before
doing so! Hackers on the internet are actively trying to subvert Lightnetwork nodes with exposed APIs!
Further, all browser tabs can access locally running webservers, so malicious webpages could try to
subvert locally available APIs!**

## License

The go-lightnetwork library (i.e. all code outside of the `cmd` directory) is licensed under the
[GNU Lesser General Public License v3.0](https://www.gnu.org/licenses/lgpl-3.0.en.html), also
included in our repository in the `COPYING.LESSER` file.

The go-lightnetwork binaries (i.e. all code inside of the `cmd` directory) is licensed under the
[GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html), also included
in our repository in the `COPYING` file.
