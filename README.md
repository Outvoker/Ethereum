
# Ethereum

Mercle tree
在最新的区块里保存整个state

## Geth

进入一个目录`geth --networkid 123 --dev --dev.period 1 --datadir data1 --rpc --rpcaddr 192.168.170.128 --rpcport 8989 --port 3000 --allow-insecure-unlock`
另开一个终端`geth attach ipc:geth.ipc`
解锁`personal.unlockAccount(eth.accounts[0])`
转账`eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})`
部署智能合约<https://www.jianshu.com/p/a5af042f32e7>

```shell
echo "var storageOutput=`solc --optimize --combined-json abi,bin,interface test.sol`" > test.js
```

`var storageContractAbi = storageOutput.contracts['test.sol:test'].abi`
`var storageContract = eth.contract(JSON.parse(storageContractAbi))`
`var storageBinCode = "0x" + storageOutput.contracts['test.sol:test'].bin`
`var deployTransationObject = { from: eth.accounts[0], data: storageBinCode, gas: 1000000 };`
`var storageInstance = storageContract.new(deployTransationObject)`

### 创世区块

```json
{
    "config": {
        "chainID": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "4",
    "gasLimit": "2100000",
    "alloc": {
        "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": {
            "balance": "300000"
        },
        "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": {
            "balance": "400000"
        }
    }
}
```

### truffle+geth

`geth --datadir data0 --networkid 1108 --allow-insecure-unlock --rpc --rpcaddr 0.0.0.0 --rpcport 8545 --rpcapi "admin,debug,eth,miner,net,personal,shh,txpool,web3" -rpccorsdomain "*" --nodiscover --ipcdisable console 2>>geth.log`
silk skate illness always trick fatal soap lift uniform model horse basic

### Geth指令

|||
|----|----|
|查看账户余额|eth.getBalance(eth.accounts[0])|
|查看挖矿账户|eth.coinbase|
|设置挖矿账户|miner.setEtherbase(eth.accounts[0])|
|查看区块高度|eth.blockNumber|
|开始挖矿|miner.start()|
|结束挖矿|miner.stop()|
|创建账户|personal.newAccount("123456")|
|预估手续费|web3.eth.estimateGas({data:code})|

## truffle react-box

This box comes with everything you need to start using smart contracts from a react app. This is as barebones as it gets, so nothing stands in your way.
<https://github.com/truffle-box/react-box>
`truffle unbox react`
遇到问题：卡在setup 改为进行
`git clone https://github.com/truffle-box/react-box.git`
`cd client`
`sudo npm cli`
这里遇到root下也要加sudo才成功 不然permission denied
`npm run start`

### 投票demo

旧版<https://blog.csdn.net/oulingcai/article/details/85090989>
contract

```js
pragma solidity ^0.5.0;

contract SimpleStorage {
  string[] public candidates=new string[](0);
  mapping(string=>uint) ballots;

  constructor() public{
  }
  function checkCandidate(string memory _candidate) public view returns(bool){
      for(uint i = 0; i < candidates.length; i++){
          if(hashCompareInternal(candidates[i],_candidate)){
              return true;
          }
      }
      return false;
  }
  function vote(string memory _candidate) public{
      assert(checkCandidate(_candidate));
      ballots[_candidate] += 1;
  }
  function getBallot(string memory _candidate) public view returns(uint){
      assert(checkCandidate(_candidate));
      return ballots[_candidate];
  }
  function getCandidatesCount() public view returns(uint){
      return candidates.length;
  }
  function hashCompareInternal(string memory a, string memory b) private pure returns (bool) {
        return keccak256(bytes(a)) == keccak256(bytes(b));
    }
  function addCandidate(string memory _person) public{
    if(checkCandidate(_person)){
        return;
    }else{
        candidates.push(_person);
    }
  }
  function getCandidates(uint index) public view returns(string memory){
     return candidates[index];
  }
}
```

client/src/App.js

```js
import React, { Component } from "react";
import SimpleStorageContract from "./contracts/SimpleStorage.json";
import getWeb3 from "./getWeb3";

import "./App.css";

class App extends Component {
  state = { storageValue: 0, candidates:[], web3: null, accounts: null, contract: null };

  componentDidMount = async () => {
    try {
      // Get network provider and web3 instance.
      const web3 = await getWeb3();

      // Use web3 to get the user's accounts.
      const accounts = await web3.eth.getAccounts();

      // Get the contract instance.
      const networkId = await web3.eth.net.getId();
      const deployedNetwork = SimpleStorageContract.networks[networkId];
      const instance = new web3.eth.Contract(
        SimpleStorageContract.abi,
        deployedNetwork && deployedNetwork.address,
      );

      // Set web3, accounts, and contract to the state, and then proceed with an
      // example of interacting with the contract's methods.
      this.setState({ web3, accounts, contract: instance }, this.runExample);
    } catch (error) {
      // Catch any errors for any of the above operations.
      alert(
        `Failed to load web3, accounts, or contract. Check console for details.`,
      );
      console.error(error);
    }
  };

  runExample = async () => {
    // eslint-disable-next-line
    const { accounts, candidates, contract } = this.state;
    const response=await contract.methods.getCandidatesCount().call();
    console.log(parseInt(response));

    for (var i = 0; i < parseInt(response); i++) {
      var element={name:"",count:0};
      element.name=await contract.methods.getCandidates(i).call();
      let response=await contract.methods.getBallot(element.name).call();
      element.count=parseInt(response);
      candidates[i]=element;
      console.log(candidates[i]);
    }
    this.setState({ storageValue: parseInt(response),candidates: candidates});
  };

  render() {
    if (!this.state.web3) {
      return <div>Loading Web3, accounts, and contract...</div>;
    }
    return (
      <div className="App">
        <h1>Good to Go!</h1>
        <h1>Number of candidate: {this.state.storageValue}</h1>
        <ul>{
          this.state.candidates.map((person,i)=>{
            return <li key={i}>Candidate: {person.name}   ballots:{person.count}
             <button onClick={async ()=>{
              const {candidates,accounts,contract } = this.state;
              await contract.methods.vote(person.name).send({ from:accounts[0] });
              let response=await contract.methods.getBallot(person.name).call();
              candidates[i].count=parseInt(response);
              this.setState({candidates: candidates});
            }}>ballot</button></li>
          })
        }
        </ul>
        <input ref="candidateName" style={{width:200,height:20}}></input>
        <button onClick={async ()=>{
          let value=this.refs.candidateName.value;
          const {storageValue,candidates,accounts,contract } = this.state;
          console.log(value+"===="+candidates.length);
          await contract.methods.addCandidate(value).send({ from:accounts[0] });
          var element={name:value,count:0};
          candidates[candidates.length]=element;
          this.setState({storageValue: storageValue+1,candidates: candidates});
        }}>add new candidate</button>
      </div>
    );
  }
}

export default App;

```

## 附录

### npm设置代理

```shell
npm config set proxy http://127.0.0.1:52453
npm config set https-proxy http://127.0.0.1:52454
```

取消代理

```shell
npm config delete proxy
npm config delete https-proxy
```

### 更改npm源

1.查看源：`npm config get registry`

2.更改为npm淘宝源：`npm config set registry https://registry.npm.taobao.org`

3.更改为npm官方源：`npm config set registry https://registry.npmjs.org`

4.临时修改npm源

`npm install module_name --registry https://registry.npm.taobao.org`

### npm code EISGIT ERR 修复

`rm -rf node_modules/*/.git/`
