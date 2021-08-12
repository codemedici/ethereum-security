# Manticore

The aim of this tutorial is to show how to use Manticore to automatically find bugs in smart contracts.

The first part introduces a set of the basic features of Manticore: running under Manticore and manipulating smart contracts through API, getting throwing path, adding constraints. The second part is exercise to solve.

**Table of contents:**

* [Installation](https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/manticore#installation)
* [Introduction to symbolic execution](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore/symbolic-execution-introduction.md): Brief introduction to symbolic execution
* [Running under Manticore](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore/running-under-manticore.md): How to use Manticore's API to run a contract
* [Getting throwing paths](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore/getting-throwing-paths.md): How to use Manticore's API to get specific paths
* [Adding constraints](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore/adding-constraints.md): How to use Manticore's API to add paths' constraints
* [Exercises](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/manticore/exercises)

Join the team on Slack at: [https://empireslacking.herokuapp.com/](https://empireslacking.herokuapp.com/) \#ethereum, \#manticore

### Installation

Manticore requires &gt;= python 3.6. It can be installed through pip or using docker.

### Manticore through docker

```text
docker pull trailofbits/eth-security-toolbox
docker run -it -v "$PWD":/home/training trailofbits/eth-security-toolbox
```

_The last command runs eth-security-toolbox in a docker that has access to your current directory. You can change the files from your host, and run the tools on the files from the docker_

Inside docker, run:

```text
solc-select 0.5.11
cd /home/trufflecon/
```

#### Manticore through pip

```text
pip3 install --user manticore
```

solc 0.5.11 is recommended.

#### Running a script

To run a python script with python 3:

```text
python3 script.py
```

