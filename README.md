# GenesisZ
Zcash (forks) genesis block mining script. Runs an external miner for finding valid Equihash solutions.
Inspired by [GenesisH0](https://github.com/lhartikk/GenesisH0), but written from scratch.

## Features
- Modify every parameter that influences the block header hash (see _Usage_)
- Support for [silent army](https://github.com/mbevand/silentarmy) and [tromp equihash](https://github.com/tromp/equihash) solvers.
- Sensible defaults
- Placeholders of the form `{BTC}`, `{ETH}` or `{ZEC}` in the `TIMESTAMP` input string get translated to the currency's latest block number and hash.
- Uses the [`python-zcashlib`](https://github.com/sebastianst/python-zcashlib), which is a (still very much unfinished) extension of the well-known [`python-bitcoinlib`](https://github.com/petertodd/python-bitcoinlib).

# Dependencies
On Ubuntu, make sure that you have installed the python dev packages and python venv:
```bash
sudo apt-get install python3-venv python3-dev pypy-dev
```

## Getting started
Clone this repo, create a **python 3** virtualenv and install dependencies with pip:
```bash
git clone --recursive https://github.com/sebastianst/GenesisZ
python3 -m venv GenesisZ
cd GenesisZ
source bin/activate
pip install -r requirements.txt
```

Make sure you have a working and supported equihash solver. Currently, the
[silent army](https://github.com/mbevand/silentarmy) GPU solver (only for
mainnet and testnet parameters `N,k=200,9`) and [tromp equihash](https://github.com/tromp/equihash) CPU solver are supported.

Keep in mind that you may need to specify how many rounds to run using the -r parameter, otherwise it will default (sensibly) to only trying once.

#### python-zcashlib submodule
Note that the zcashlib is used as a submodule, since I haven't uploaded it to PyPI yet (and because it's easier for the current interdependent development). That's why you must use the `--recursive` flag during cloning. When you update this repo, don't forget to update the submodule as well, i.e., run `git pull && git submodule update` to update.

## Examples

### Zcash mainnet
Mine the zcash mainnet gensis block with the silentarmy solver by calling
```bash
./genesis.py -s "/path/to/sa-solver" -r 5000 -t 1477641360
```
or cheat, because you already know the right nonce:
```bash
./genesis.py -s "/path/to/sa-solver" -n 1257 -t 1477641360
```

### Zcash testnet
This time using Tromp's solver with 4 threads, checking up to 10 nonces.
```bash
./genesis.py -c testnet -t 1477648033 -b 0x2007ffff -E 0x1f07ffff -s '/path/to/equihash/equi' -T 4 -r 10
```
It will find the right solution for nonce `6`.

### Zcash regtest
Using Tromp's `48,5` solver with
```bash
./genesis.py -c regtest -t 1296688602 -b 0x200f0f0f -E 0x1f07ffff -s '/path/to/equihash/eq485' -T 4 -r 10
```
will find the correct regtest solution for nonce `9`.

### Zclassic mainnet

Zclassic decided to use a custom extra nonce of `0x1d00ffff`. Let's mine their
mainnet solution (already setting the right nonce to `0x00...021d`) with

```bash
./genesis.py -c mainnet -t 1478403829 -E 0x1d00ffff -C Zclassic -z "No taxation without representation. BTC #437541 - 00000000000000000397f175a94dd3f530b957182eb2a9f7b79a44a94a5e0450" -s '/path/to/equihash/equi' -T 4 -n 21d
```

## Usage
```
usage: genesis.py [-h] [-c {mainnet,testnet,regtest}] [-t TIME] [-C COINNAME]
                  [-z TIMESTAMP] [-Z PSZTIMESTAMP] [-p PUBKEY] [-b BITS]
                  [-E EXTRANONCE] [-V VALUE] [-n NONCE] [-r ROUNDS]
                  [-s SOLVER] [-S {tromp,silentarmy}] [-T THREADS] [-v]

This script uses any Equihash solver to find a solution for the specified
genesis block

optional arguments:
  -h, --help            show this help message and exit
  -c {mainnet,testnet,regtest}, --chainparams {mainnet,testnet,regtest}
                        Select the core chain parameters for PoW limit and
                        parameters N and K.
  -t TIME, --time TIME  unix time to set in block header (defaults to current
                        time)
  -C COINNAME, --coinname COINNAME
                        the coin name prepends the blake2s hash of timestamp
                        in pszTimestamp
  -z TIMESTAMP, --timestamp TIMESTAMP
                        the pszTimestamp found in the input coinbase
                        transaction script. Will be blake2s'd and then
                        prefixed by coin name. Default is Zcash's mainnet
                        pszTimestamp. You may use tokens of the form {XYZ},
                        which will be replaced by the current block index and
                        hash of coin XZY (BTC, ETH or ZEC). Always the latest
                        block is retrieved, regardless of time argument.
  -Z PSZTIMESTAMP, --pszTimestamp PSZTIMESTAMP
                        Specify the pszTimestamp directly. Will ignore options
                        -C and -z
  -p PUBKEY, --pubkey PUBKEY
                        the pubkey found in the output transaction script
  -b BITS, --bits BITS  the target in compact representation, defining a
                        difficulty of 1
  -E EXTRANONCE, --extra-nonce EXTRANONCE
                        Usually, the coinbase script contains the nBits as
                        fixed first data, which in bitcoin is also referred to
                        as extra nonce. This conventional behaviour can be
                        changed by specifying this parameter (not recommended
                        for mainnet, useful for testnet).
  -V VALUE, --value VALUE
                        output transaction value in zatoshi (1 ZEC = 100000000
                        zatoshi)
  -n NONCE, --nonce NONCE
                        nonce to start with when searching for a valid
                        equihash solution; parsed as hex, leading zeros may be
                        omitted.
  -r ROUNDS, --rounds ROUNDS
                        how many nonces to check at most
  -s SOLVER, --solver SOLVER
                        path to solver binary. Currently supported are
                        silentarmy (sa-solver) and Tromp (equi/equi485).
                        Command line arguments may be passed, although that
                        should be unnecessary.
  -S {tromp,silentarmy}, --solver-type {tromp,silentarmy}
                        Set the type of solver explicitly. Otherwise GenesisZ
                        tries to infer the type from the binary name (equi* ->
                        tromp, sa-solver -> silentarmy)
  -T THREADS, --threads THREADS
                        How many CPU threads to use when solving with Tromp.
  -v, --verbose         verbose output
```

### Tromp solver
Make sure to select the right binary with `-s` when using Tromp's equihash solver:

\#threads | main/testnet (`N,K=200,9`) | regtest (`N,K=48,5`)
----------|----------------------------|--------------------
1         | `equi1`                    | `eq4851`
\>1       | `equi`                     | `eq485`

Note that `make` only builds `equi{,1}`, so you have to run `make eq485{,1}` in Tromp's source directory if you need the solver for regtest.

## TODO

- [ ] More structured and complete output of intermediate information and
  results. Currently, you need to specify verbose output to see all necessary information.
- [ ] Use solvers' native APIs instead of reading `stdout`. None of the
  supported solvers expose such an API now. Maybe write little C wrapper...
- [ ] Make block number selectable for the `TIMESTAMP` placeholders. Like `{BTC:1234}` for block #1234.

## License
Released under the GPLv3, see `LICENSE` file.
