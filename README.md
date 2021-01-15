## ฝากเหรียญ mDai แล้วได้ GOPGAP(DAPP) token. 
บทนำ - สร้างธนาคารดิจิตอลที่สามารถรับดอกเบี้ยได้(reward/interest)จากการฝากเงินดิจิตอล เข้าไปในระบบ

## ขั้นตอนการทำงาน
คือ เริ่มจาก clone โปรเจ็ค defi_tutorial มาจาก https://github.com/dappuniversity/defi_tutorial.git  
อ่อ ทำการสร้าง dir เพื่อ clone โปรเจ็คมาใส่ด้วยนะ ด้วยคำสั่ง mkdir ตามด้วยชื่อfolder  ตามด้วย cd เท่ากับว่าตอนนี้เราอยู่ใน dir นี้แล้วนะ 
จากนั้นใช้คำสั่ง 
```
mkdir DAPP-stake
cd DAPP-stake/
git clone -b starter-code https://github.com/dappuniversity/defi_tutorial.git
git checkout -b master 
```
## หากต้องการข้ามขั้นตอนการสร้าง file จากจุดเริ่มต้น คุณสามารถใช้คำสั่งใน terminal ได้ดังนี้
```
git clone https://github.com/galalap/DAPP-stake.git
npm install
truffle migrate
npm run start
```
หากต้องการเริ่มต้นการทำงานด้วยโปรเจ็คนี้ ควรเช็ค version ของ node ก่อน ด้วยคำสั่ง node -v.  
จากนั้น ควรกำหนด version node เป็น 12.18.3
ต่อจากนั้นเปิด ganache แล้ว ทำการสร้าง new workspece แล้วกำหนด networkId 1337 หรือ 5777 

![unbox.png](https://i.postimg.cc/MTM5wsjK/Screen-Shot-2564-01-15-at-17-04-43.png)
![unbox.png](https://i.postimg.cc/Lsrftmn2/Screen-Shot-2564-01-15-at-17-04-36.png)

เสร็จแล้วกด save workspace

ตรวจสอบ truffle ว่าในเครื่องหรือไม่ หากว่าไม่มีให้ใช้คำสั่ง
```
sudo npm install --g truffle@5.1.39
```

ถ้าหากว่ามีแล้วให้ใช้คำสั่ง truffle version เพื่อ check ค่า ให้พร้อมสำหรับเริ่มทำงาน

[![unbox.png](https://i.postimg.cc/cLfFt2N1/138284470-777593626175114-932504662699652668-n.png)

เริ่มต้นด้วยการใช้คำสั่งใน terminal เปิด program coding ขึ้นมา ผมใช้ vscode จึงใช้คำสั่งนี้
```
code.
```
เปิด terminal ใน vscode เสร็จแล้ว ทำการ install ด้วยคำสั่ง
```
npm install
```
เสร็จแล้ว เรามาเริ่มสร้างไฟล์ใหม่ชื่อ TokenFarm.sol กัน โค้ดตามนี้
```
pragma solidity ^0.5.0;

import "./DappToken.sol";
import "./DaiToken.sol";

contract TokenFarm {
    string public name = "Dapp Token Farm";
    address public owner;
    DappToken public dappToken;
    DaiToken public daiToken;

    address[] public stakers;
    mapping(address => uint) public stakingBalance;
    mapping(address => bool) public hasStaked;
    mapping(address => bool) public isStaking;


    constructor (DappToken _dappToken, DaiToken _daiToken) public {
        dappToken = _dappToken;
        daiToken = _daiToken;
        owner = msg.sender;
    }
    function stakeTokens(uint _amount) public {
        // Require amount greater than 0
        require(_amount > 0, "amount cannot be 0");

        // Trasnfer Mock Dai tokens to this contract for staking
        daiToken.transferFrom(msg.sender, address(this), _amount);

        // Update staking balance
        stakingBalance[msg.sender] = stakingBalance[msg.sender] + _amount;

        // // Add user to stakers array *only* if they haven't staked already
        if(!hasStaked[msg.sender]) {
            stakers.push(msg.sender);
        }

         // Update staking status
        isStaking[msg.sender] = true;
        hasStaked[msg.sender] = true;
    }

    // Unstaking Tokens (Withdraw)
    function unstakeTokens() public {
        // Fetch staking balance
        uint balance = stakingBalance[msg.sender];

        // Require amount greater than 0
        require(balance > 0, "staking balance cannot be 0");

        // Transfer Mock Dai tokens to this contract for staking
        daiToken.transfer(msg.sender, balance);

        // Reset staking balance
        stakingBalance[msg.sender] = 0;

        // Update staking status
        isStaking[msg.sender] = false;
    }

        // Issuing Tokens
    function issueTokens() public {
        // Only owner can call this function
        require(msg.sender == owner, "caller must be the owner");

        // Issue tokens to all stakers
        for (uint i=0; i<stakers.length; i++) {
            address recipient = stakers[i];
            uint balance = stakingBalance[recipient];
            if(balance > 0) {
                dappToken.transfer(recipient, balance);
            }
        }
    }
}
```
เสร็จแล้วสร้าง file ใหม่ชื่อ 2_deploy_contracts.js

```
const DappToken = artifacts.require('DappToken')
const DaiToken = artifacts.require('DaiToken')
const TokenFarm = artifacts.require('TokenFarm')

module.exports = async function(deployer, network, accounts) {
  // Deploy Mock DAI Token
  await deployer.deploy(DaiToken)
  const daiToken = await DaiToken.deployed()

  // Deploy Dapp Token
  await deployer.deploy(DappToken)
  const dappToken = await DappToken.deployed()

  // Deploy TokenFarm
  await deployer.deploy(TokenFarm, dappToken.address, daiToken.address)
  const tokenFarm = await TokenFarm.deployed()

  // Transfer all tokens to TokenFarm (1 million)
  await dappToken.transfer(tokenFarm.address, '1000000000000000000000000')

  // Transfer 100 Mock DAI tokens to investor
  await daiToken.transfer(accounts[1], '100000000000000000000')
}
```
ต่อจากนั้นเปิด terminal ขึ้นมาแล้วเขียนคำสั่ง truffle console ดังนี้
```
truffle console
> mDai = await DaiToken.deployed()
> accounts = await web3.eth.getAccounts()
> accounts[1]
> balance = await mDai.balanceOf(accounts[1])
> balance.toString()
> formattedBalance = web3.utils.fromWei(Balance)
> web3.utils.toWei('1.5', 'eth')
```
เสร็จแล้ว มาสร้าง file test โดยใช้คำสั่ง
```
mkdir test
touch test/TokenFarm.test.js
```
แล้ว copy code ไปใส่ file ตามนี้
```
const DaiToken = artifacts.require('DaiToken')
const DappToken = artifacts.require('DappToken')
const TokenFarm = artifacts.require('TokenFarm')

require('chai')
  .use(require('chai-as-promised'))
  .should()

  function tokens(n) {
    return web3.utils.toWei(n, 'ether');
  }
  

  contract('TokenFarm', ([owner, investor]) => {
    let daiToken, dappToken, tokenFarm

    before(async () => {
        // Load Contracts
        daiToken = await DaiToken.new()
        dappToken = await DappToken.new()
        tokenFarm = await TokenFarm.new(dappToken.address, daiToken.address)

        await dappToken.transfer(tokenFarm.address, tokens('1000000'))
        
        // Send tokens to investor
        await daiToken.transfer(investor, tokens('100'), { from: owner })
        

    })

    describe('Mock DAI deployment', async () => {
        it('has a name', async () => {
            const name = await daiToken.name()
            assert.equal(name, 'Mock DAI Token')

        })
    })

    describe('Dapp Token deployment', async () => {
        it('has a name', async () => {
            const name = await dappToken.name()
            assert.equal(name, 'DApp Token')
        
         })
    })

    describe('Token Farm deployment', async () => {
        it('has a name', async () => {
            const name = await tokenFarm.name()
            assert.equal(name, 'Dapp Token Farm')
        
         })

    })
    
    it('contract has tokens', async () => {
        let balance = await dappToken.balanceOf(tokenFarm.address)
        assert.equal(balance.toString(), tokens('1000000'))
      })

      describe('Farming tokens', async () => {

        it('rewards investors for staking mDai tokens', async () => {
          let result
    
          // Check investor balance before staking
          result = await daiToken.balanceOf(investor)
          assert.equal(result.toString(), tokens('100'), 'investor Mock DAI wallet balance correct before staking')
    
          // Stake Mock DAI Tokens
          await daiToken.approve(tokenFarm.address, tokens('100'), { from: investor })
          await tokenFarm.stakeTokens(tokens('100'), { from: investor })
    
          // Check staking result
          result = await daiToken.balanceOf(investor)
          assert.equal(result.toString(), tokens('0'), 'investor Mock DAI wallet balance correct after staking')
    
          result = await daiToken.balanceOf(tokenFarm.address)
          assert.equal(result.toString(), tokens('100'), 'Token Farm Mock DAI balance correct after staking')
    
          result = await tokenFarm.stakingBalance(investor)
          assert.equal(result.toString(), tokens('100'), 'investor staking balance correct after staking')
    
          result = await tokenFarm.isStaking(investor)
          assert.equal(result.toString(), 'true', 'investor staking status correct after staking')
    
          // Issue Tokens
          await tokenFarm.issueTokens({ from: owner })
    
          // Check balances after issuance
          result = await dappToken.balanceOf(investor)
          assert.equal(result.toString(), tokens('100'), 'investor DApp Token wallet balance correct after issuance')
    
          // Ensure that only owner can issue tokens
          await tokenFarm.issueTokens({ from: investor }).should.be.rejected;
    
          // Unstake tokens
          await tokenFarm.unstakeTokens({ from: investor })
    
          // Check results after unstaking
          result = await daiToken.balanceOf(investor)
          assert.equal(result.toString(), tokens('100'), 'investor Mock DAI wallet balance correct after staking')
    
          result = await daiToken.balanceOf(tokenFarm.address)
          assert.equal(result.toString(), tokens('0'), 'Token Farm Mock DAI balance correct after staking')
    
          result = await tokenFarm.stakingBalance(investor)
          assert.equal(result.toString(), tokens('0'), 'investor staking balance correct after staking')
    
          result = await tokenFarm.isStaking(investor)
          assert.equal(result.toString(), 'false', 'investor staking status correct after staking')
        })

      })
    
})
```
เราสามารถใช้ truffle test เพื่อตรวจสอบข้อผิดพลาดได้และสามารถใช้ได้ตลอด ทุกครั้งที่เพิ่มหรือแก้ไข file.sol
เสร็จแล้วใน DappToken.sol เพิ่มโค้ดหรือ replace โค้ดด้วยโค้ดชุดนี้
```
pragma solidity ^0.5.0;

contract DappToken {
    string  public name = "DApp Token";
    string  public symbol = "DAPP";
    uint256 public totalSupply = 1000000000000000000000000; // 1 million tokens
    uint8   public decimals = 18;

    event Transfer(
        address indexed _from,
        address indexed _to,
        uint256 _value
    );

    event Approval(
        address indexed _owner,
        address indexed _spender,
        uint256 _value
    );

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    constructor() public {
        balanceOf[msg.sender] = totalSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= balanceOf[_from]);
        require(_value <= allowance[_from][msg.sender]);
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        emit Transfer(_from, _to, _value);
        return true;
    }
}
```
จากนั้นเราแก้ไข DaiToken.sol ตามโค้ดชุดนี้
```
pragma solidity ^0.5.0;

contract DaiToken {
    string  public name = "Mock DAI Token";
    string  public symbol = "mDAI";
    uint256 public totalSupply = 1000000000000000000000000; // 1 million tokens
    uint8   public decimals = 18;

    event Transfer(
        address indexed _from,
        address indexed _to,
        uint256 _value
    );

    event Approval(
        address indexed _owner,
        address indexed _spender,
        uint256 _value
    );

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    constructor() public {
        balanceOf[msg.sender] = totalSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool success) {
        require(balanceOf[msg.sender] >= _value);
        balanceOf[msg.sender] -= _value;
        balanceOf[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value) public returns (bool success) {
        allowance[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function transferFrom(address _from, address _to, uint256 _value) public returns (bool success) {
        require(_value <= balanceOf[_from]);
        require(_value <= allowance[_from][msg.sender]);
        balanceOf[_from] -= _value;
        balanceOf[_to] += _value;
        allowance[_from][msg.sender] -= _value;
        emit Transfer(_from, _to, _value);
        return true;
    }
}
```
เรามาสร้างfile issue-token.js โดยเปิด terminal
```
mkdir scripts
touch scripts/issue-token.js
```
จากนั้น copy code ชุดนี้ลงไป 
```
const TokenFarm = artifacts.require('TokenFarm')

module.exports = async function(callback) {
  let tokenFarm = await TokenFarm.deployed()
  await tokenFarm.issueTokens()
  // Code goes here...
  console.log("Tokens issued!")
  callback()
}
```
กลับไปเปิด terminal แล้วใส่คำสั่ง จะได้รับค่า Token issued! ถือว่าสำเร็จ
```
truffle exec scripts/issue-token.js
```
จะได้รับค่า Token issued! ถือว่าสำเร็จ. 

จากนั้น ทำการ compile โดยใช้คำสั่ง truffle compile 
```
(base) thanaporns-mbp:DAPP-stake galalap$ truffle compile

Compiling your contracts...
===========================
> Compiling ./src/contracts/DaiToken.sol
> Compiling ./src/contracts/DappToken.sol
> Compiling ./src/contracts/Migrations.sol
> Compiling ./src/contracts/TokenFarm.sol
> Artifacts written to /Users/galalap/Desktop/intro to defi/DAPP-stake/src/abis
> Compiled successfully using:
   - solc: 0.5.16+commit.9c3226ce.Emscripten.clang

(base) thanaporns-mbp:DAPP-stake galalap$ truffle migrate --reset

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.

```
และต่อด้วย truffle migrate
```
(base) thanaporns-mbp:DAPP-stake galalap$ truffle migrate 

Compiling your contracts...
===========================
> Everything is up to date, there is nothing to compile.



Starting migrations...
======================
> Network name:    'development'
> Network id:      1337
> Block gas limit: 6721975 (0x6691b7)


1_initial_migration.js
======================

   Replacing 'Migrations'
   ----------------------
   > transaction hash:    0xf9073e46600a8966064f6a37091f3f8c0abe70d463d9bba57aafbcba06539e35
   > Blocks: 0            Seconds: 0
   > contract address:    0xD16c1616459ACABA2643a845D60FCFaA9b2F4958
   > block number:        41
   > block timestamp:     1610707010
   > account:             0xcd77C2110288607c17d38CCc00e1a6DAdc21BdDD
   > balance:             99.7181357
   > gas used:            225237 (0x36fd5)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.00450474 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.00450474 ETH


2_deploy_contracts.js
=====================

   Replacing 'DaiToken'
   --------------------
   > transaction hash:    0x00a89f7e410be8bbe708e33a7ca69d567adc3bee55b30eef6b4d9fc86f2edc85
   > Blocks: 0            Seconds: 0
   > contract address:    0x5167098dAeD0F154FA8AB8D3d37fe9ABC25C7816
   > block number:        43
   > block timestamp:     1610707011
   > account:             0xcd77C2110288607c17d38CCc00e1a6DAdc21BdDD
   > balance:             99.70241932
   > gas used:            743456 (0xb5820)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.01486912 ETH


   Replacing 'DappToken'
   ---------------------
   > transaction hash:    0xecd9ad418bef3bce421e86615c224254b15b2eb6b36876c5bfe3df765f343eb5
   > Blocks: 0            Seconds: 0
   > contract address:    0xE24F44e49d35F4eA088D70b8BBd5eF6fC1035B16
   > block number:        44
   > block timestamp:     1610707011
   > account:             0xcd77C2110288607c17d38CCc00e1a6DAdc21BdDD
   > balance:             99.6875514
   > gas used:            743396 (0xb57e4)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.01486792 ETH


   Replacing 'TokenFarm'
   ---------------------
   > transaction hash:    0x169a832eb50e8a3196059cde6df66545158fc9a26d0042a8a1995fd1a936eecf
   > Blocks: 0            Seconds: 0
   > contract address:    0xd2b9542058fFF4994c622eABda0B643E7f1d9E00
   > block number:        45
   > block timestamp:     1610707011
   > account:             0xcd77C2110288607c17d38CCc00e1a6DAdc21BdDD
   > balance:             99.66948106
   > gas used:            903517 (0xdc95d)
   > gas price:           20 gwei
   > value sent:          0 ETH
   > total cost:          0.01807034 ETH


   > Saving migration to chain.
   > Saving artifacts
   -------------------------------------
   > Total cost:          0.04780738 ETH


Summary
=======
> Total deployments:   4
> Final cost:          0.05231212 ETH
```

พอเราทำ โค้ดที่เป็นส่วนของ back-end สำเร็จแล้ว เราก็จะมาทำส่วนที่เป็น front-end และ design ต่อ
ตอนนี้เรายังอยู่ใน terminal ที่ต่อจากการใช้คำสั่ง truffle migrate และเรากำลังจะเปิดหน้าเว็บไซต์ขึ้นมาด้วยคำสั่ง
```
npm run start
```

หน้าเว็บไซต์จะได้ประมาณนี้  


![unbox.png](https://i.postimg.cc/s2BPp938/137578421-2947902685490830-5180944901416980877-n.png)


ตอนนี้ถ้าหน้าเว็บไซต์ยังไม่แสดง address ก็ยังไม่ต้องแปลกใจอะไรนะครับ เรามาทำการเชื่อม metamask กับ ganache เข้าด้วยกันก่อน
เราเริ่มจาก
เปิด metamask ขึ้นมา จากนั้นทำการสร้าง rpc ใหม่ กำหนดค่าดังนี้ ปล.หากว่า ganache กำหนด chainId/networkId เป็น 5777 ต้องกำหนด 5777 ในmetamask ด้วยครับ 


![unbox.png](https://i.postimg.cc/KYnWP2GC/138461317-227968955578920-5637049998886959270-n.png)


เสร็จแล้วเรามานำ wallet/account เข้ามาใน metamask กันต่อ ตามรูปดังนี้


![unbox.png](https://i.postimg.cc/bvzx5c07/138373427-441990223592398-8683336885160990042-n.png)


เลือกคำสั่งนำเข้าบัญชี


![unbox.png](https://i.postimg.cc/mgSMCpx8/139237002-773721083233992-8328053209753810708-n.png)


จากนั้น เปิด ganache ขึ้นมาแล้ว copy private key มาจาก index 0 ดังนี้


![unbox.png](https://i.postimg.cc/2jTP9rQD/137384173-982472139242699-6444990222648079240-n.png)


กดที่รูปกุญแจจะมีหน้าต่างดังนี้ จากนั้น copy private key


![unbox.png](https://i.postimg.cc/85wNbWk8/139581603-165501908307587-5350730457329101649-n.png)


นำมาใส่ใน metamask ตรงนำเข้าบัญชีจะได้แบบนี้ครับ


![unbox.png](https://i.postimg.cc/g2bQnjd3/138566971-1997002007119735-3417290072226921835-n.png)


เสร็จแล้วเรามาเชื่อม localhost:3000 เข้ากับ metamask และ ganache 

โดยไปที่ App.js (โค้ดตรงส่วนนี้เป็น react.js) แล้วเพิ่มเติมโค้ดให้สมบูรณ์ดังนี้
```
import React, { Component } from 'react'
import Web3 from 'web3'
import DaiToken from '../abis/DaiToken.json'
import DappToken from '../abis/DappToken.json'
import TokenFarm from '../abis/TokenFarm.json'
import Navbar from './Navbar'
import Main from './Main'
import './App.css'

class App extends Component {


  async componentWillMount() {
    await this.loadWeb3()
    await this.loadBlockchainData()
   
  }
  async loadBlockchainData(){
    const web3 = window.web3
    const accounts = await web3.eth.getAccounts()
    this.setState({ account: accounts[0] })
    
    const networkId = await web3.eth.net.getId()
    
    console.log(networkId)
   
   // Load DaiToken
    const daiTokenData = DaiToken.networks[networkId]
    if(daiTokenData) {
      const daiToken = new web3.eth.Contract(DaiToken.abi, daiTokenData.address)
      this.setState({ daiToken })
      let daiTokenBalance = await daiToken.methods.balanceOf(this.state.account).call()
      this.setState({ daiTokenBalance: daiTokenBalance.toString() })
      
    } else {
      window.alert('DaiToken contract not deployed to detected network.')
    }
     // Load DappToken
     const dappTokenData = DappToken.networks[networkId]
     if(dappTokenData) {
       const dappToken = new web3.eth.Contract(DappToken.abi, dappTokenData.address)
       this.setState({ dappToken })
       let dappTokenBalance = await dappToken.methods.balanceOf(this.state.account).call()
       this.setState({ dappTokenBalance: dappTokenBalance.toString() })
     } else {
       window.alert('DappToken contract not deployed to detected network.')
     }
     // Load TokenFarm
    const tokenFarmData = TokenFarm.networks[networkId]
    if(tokenFarmData) {
      const tokenFarm = new web3.eth.Contract(TokenFarm.abi, tokenFarmData.address)
      this.setState({ tokenFarm })
      let stakingBalance = await tokenFarm.methods.stakingBalance(this.state.account).call()
      this.setState({ stakingBalance: stakingBalance.toString() })
    } else {
      window.alert('TokenFarm contract not deployed to detected network.')
    }

    this.setState({ loading: false })
 
  }

  async loadWeb3() {
    if (window.ethereum) {
      window.web3 = new Web3(window.ethereum)
      await window.ethereum.enable()
    }
    else if (window.web3) {
      window.web3 = new Web3(window.web3.currentProvider)
    }
    else {
      window.alert('Non-Ethereum browser detected. You should consider trying MetaMask!')
    }
  }
  stakeTokens = (amount) => {
    this.setState({ loading: true })
    this.state.daiToken.methods.approve(this.state.tokenFarm._address, amount).send({ from: this.state.account }).on('transactionHash', (hash) => {
      this.state.tokenFarm.methods.stakeTokens(amount).send({ from: this.state.account }).on('transactionHash', (hash) => {
        this.setState({ loading: false })
      })
    })
  }
  unstakeTokens = (amount) => {
    this.setState({ loading: true })
    this.state.tokenFarm.methods.unstakeTokens().send({ from: this.state.account }).on('transactionHash', (hash) => {
      this.setState({ loading: false })
    })
  }

  constructor(props) {
    super(props)
    this.state = {
      account: '0x0',
      daiToken: {},
      dappToken: {},
      tokenFarm: {},
      daiTokenBalance: '0',
      dappTokenBalance: '0',
      stakingBalance: '0',
      loading: true
    }
  }

  render() {
    let content
    if(this.state.loading) {
      content = <p id="loader" className="text-center">Loading...</p>
    } else  {
        content = <Main 
        daiTokenBalance={this.state.daiTokenBalance}
        dappTokenBalance={this.state.dappTokenBalance}
        stakingBalance={this.state.stakingBalance}
        stakeTokens={this.stakeTokens}
        unstakeTokens={this.unstakeTokens}
        />
    }
    return (
      <div>
        <Navbar account={this.state.account} />
        <div className="container-fluid mt-5">
          <div className="row">
            <main role="main" className="col-lg-12 ml-auto mr-auto" style={{ maxWidth: '600px' }}>
              <div className="content mr-auto ml-auto">
                <a
                  href="http://www.dappuniversity.com/bootcamp"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                </a>

               {content}

              </div>
            </main>
          </div>
        </div>
      </div>
    );
  }
}

export default App;
```
เมื่อเพิ่มเติมโค้ดสมบูรณ์แล้ว
สร้าง file Main.js ขึ้นมา แล้วเติมโค้ดลงไปตามนี้
```
import React, { Component } from 'react'
import dai from '../dai.png'

class Main extends Component {


  render() {
    return (
      <div id="content" className="mt-3">
        <table className="table table-borderless text-muted text-center">
          <thead>
            <tr>
              <th scope="col">Staking Balance</th>
              <th scope="col">Reward Balance</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{window.web3.utils.fromWei(this.props.stakingBalance, 'Ether')} mDAI</td>
              <td>{window.web3.utils.fromWei(this.props.dappTokenBalance, 'Ether')} GOPGAP</td>
            </tr>
          </tbody>
        </table>
        <div className="card mb-4" >

        <div className="card-body">
        <form className="mb-3" onSubmit={(event) => {
                event.preventDefault()
                let amount
                amount = this.input.value.toString()
                amount = window.web3.utils.toWei(amount, 'Ether')
                this.props.stakeTokens(amount)
              }}>
              <div>
                <label className="float-left"><b>Stake Tokens</b></label>
                <span className="float-right text-muted">
                  Balance: {window.web3.utils.fromWei(this.props.daiTokenBalance, 'Ether')}
                </span>
              </div>
              <div className="input-group mb-4">
                <input
                  type="text"
                  ref={(input) => { this.input = input }}
                  className="form-control form-control-lg"
                  placeholder="0"
                  required />
                <div className="input-group-append">
                  <div className="input-group-text">
                    <img src={dai} height='32' alt=""/>
                    &nbsp;&nbsp;&nbsp; mDAI
                  </div>
                </div>
              </div>
              <button type="submit" className="btn btn-primary btn-block btn-lg">STAKE!</button>
            </form>
            <button
              type="submit"
              className="btn btn-link btn-block btn-sm"
              onClick={(event) => {
                event.preventDefault()
                this.props.unstakeTokens()
              }}>
                UN-STAKE...
              </button>
      </div>
      </div>
      </div>
    );
  }
}

export default Main;
```
ทีนี้เราลองเช็ค networkId ด้วยการกดคลิกขวาที่ localhost:3000 


![unbox.png](https://i.postimg.cc/Bv7dQGgF/Screen-Shot-2564-01-16-at-01-38-15.png)


แล้วกด console เช็คค่า networkId ว่าตรงกันหรือไม่ ใน ganache และ metamask ค่าที่แสดงออกมา จะมาจากคำสั่ง console.log(networkId)ที่ได้รับค่ามาจาก ganache


![unbox.png](https://i.postimg.cc/V6jBZqdN/Screen-Shot-2564-01-16-at-01-38-32.png)


ถ้าหากสังเกตุดูจะเห็นว่ามี public address แสดงอยู่ที่มุมขวาบน หากว่าแสดงแล้ว การทำงานคือสมบูรณ์ดี


ทีนี้เราลองมากด stake ดูตามนี้


![unbox.png](https://i.postimg.cc/Jh8hcjVx/Screen-Shot-2564-01-16-at-01-47-08.png)


กดเสียค่า gas 


![unbox.png](https://i.postimg.cc/02pj6Vhj/Screen-Shot-2564-01-16-at-01-48-57.png)


เราจะเห็นว่ามีตัวเลขขึ้น 10000 mDai 


![unbox.png](https://i.postimg.cc/yxhJznHD/Screen-Shot-2564-01-16-at-01-49-56.png)


แต่ตัวเลข reward ที่ได้เป็น GOPGAP(DAPP) ยังไม่ขึ้น เราสามารถทำให้ขึ้นได้ด้วยการเปิด terminal แล้วพิมคำสั่งดังนี้
```
truffle exec scripts/issue-token.js
```
terminal ตอบกลับแบบนี้


![unbox.png](https://i.postimg.cc/CxJwpXfy/137319891-3931854216866573-7411792053353950421-n.png)

เราก็จะได้ reward ดังนี้


![unbox.png](https://i.postimg.cc/2y3z0w5p/Screen-Shot-2564-01-16-at-01-59-43.png)


นอกจากนี้ใน code ของเรา ส่วนของ mDai Token หรือ mock dai token เราเขียนให้สามารถ unstake ได้อีกด้วย โดยการกดที่ unstake... ใน localhost:3000

ขอจบเพียงเท่านี้ขอบคุณครับ























