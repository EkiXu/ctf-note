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

import 'openzeppelin-solidity/contracts/math/SafeMath.sol';

contract CoinFlip {

  using SafeMath for uint256;
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number.sub(1)));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue.div(FACTOR);
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
```

``uint256(block.blockhash(block.number.sub(1)));`` 表示负一高度的区块哈希而一个交易是被打包在一个区块里的，通过攻击合约去调用Lottery合约，那么他们的区块信息都是一样的。所以可以写一个合约和他交易算出正反面

```

```