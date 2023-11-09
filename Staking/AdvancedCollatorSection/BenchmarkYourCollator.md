# Benchmark Your Collator

To enable benchmarking, the collator must enable the benchmarking feature from a new build of the `curio-parachain`.


> ⚠️ Don't use this binary for running the Collator!


```bash
cargo build --release --features runtime-benchmarks
```

The benchmarks can be run to compare the server's hardware capabilities against the referenced hardware. At the moment, we have benchmarked the Curio Parachain Mainnet and Curio Parachain Testnet runtimes on an AMD Ryzen 7 1700X with 32GB RAM and an NVMe SSD. After executing the benchmarks on a server compare the weights to the official Curio weights. Your weight results should at least be similar to the official ones and the lower yours are, the better.

Below is an example of benchmarking for the the `balances` pallet.

To release benchmarks you need to run script, which place at /curio-parachain/script. To run this script, you should run it from directory curio-parachain:

```bash
./scripts/run_all_benchmarks.sh
```

To run benchmarks for testnet or mainnet, you should change parameter

```bash
runtime=${1-"curio-devnet"}
```

to

```bash
runtime=${1-"curio-testnet"}
```

or

```bash
runtime=${1-"curio-mainnet"}
```