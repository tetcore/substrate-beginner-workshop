# Join the Network

You can use the `substrate-node-template` you compiled earlier and this chain specification file to connect to our network.

1. Save the [`spec.json`](https://denver.bootnodes.net/spec.json) file in the `substrate-node-template` folder.

2. Launch your node:

	```bash
	./target/release/node-template \
		--chain spec.json \
		--name YourNodeName
	```

> You can use the flag `--help` to learn about lots more flags available

It will use these bootstrap nodes:

```json
"bootNodes": [
	"/dns4/denver.bootnodes.net/tcp/30333/p2p/QmVQ8v2tDEVeSaCyn1tc58rk6wG87B3auXtAyTeyrf9tXb",
	"/dns4/denver.bootnodes.net/tcp/30303/p2p/QmZ9KWNfuQvurQpoF8TWJndFqR9uiRudUm99Jngsw3yKj7"
],

```

Notice that your node now appears on the telemetry page.

<!-- slide:break-70 -->

## Discussion

* Raw Spec
* Node Roles
	* Boot Nodes
	* Validators
	* Full Nodes
