## ฝากเหรียญ mDai แล้วได้ GOPGAP(DAPP) token. 
เนื่องจากต้องการศึกษาการทำงานของ code จึงทำการ clone ตัว starter เพื่อทำความเข้าใจ

# ขั้นตอนการทำงาน
คือ เริ่มจาก clone โปรเจ็ค defi_tutorial มาจาก https://github.com/dappuniversity/defi_tutorial.git  
อ่อ ทำการสร้าง dir เพื่อ clone โปรเจ็คมาใส่ด้วยนะ ด้วยคำสั่ง mkdir ตามด้วยชื่อfolder  ตามด้วย cd เท่ากับว่าตอนนี้เราอยู่ใน dir นี้แล้วนะ 
จากนั้นใช้คำสั่ง 
```
mkdir DAPP-stake
cd DAPP-stake/
git clone -b starter-code https://github.com/dappuniversity/defi_tutorial.git
git checkout -b master 
```
### หากต้องการข้ามขั้นตอนการสร้าง file จากจุดเริ่มต้น คุณสามารถใช้คำสั่งใน terminal ได้ดังนี้
```
git clone https://github.com/galalap/DAPP-stake.git
npm install
truffle migrate
npm run start
```
หากต้องการเริ่มต้นการทำงานด้วยโปรเจ็คนี้ ควรเช็ค version ของ node ก่อน ด้วยคำสั่ง node -v.  
จากนั้น ควรกำหนด version node เป็น 12.18.3
ต่อจากนั้นเปิด ganache แล้ว ทำการสร้าง new workspece
ตรวจสอบ truffle ว่าในเครื่องหรือไม่ หากว่าไม่มีให้ใช้คำสั่ง
```
sudo npm install --g truffle@5.1.39
```
ถ้าหากว่ามีแล้วให้ใช้คำสั่ง truffle version เพื่อ check ค่า ให้พร้อมสำหรับเริ่มทำงาน. 
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
ต่อจากนั้นเปิด terminal ขึ้นมาแล้วเขียนคำสั่งดังนี้
```
truffle console
> mDai = await DaiToken.deployed()
> accounts = await web3.eth.getAccounts()
> accounts[1]
> balance = await mDai.balanceOf(accounts[1])
> balance.toString()
```



















