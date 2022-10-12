#  Chapter 9. Solidity 함수, 이벤트

Solidity 함수를 정의하고, view/pure, 가시성, payable을 설정하는 문법을 배운다. 생성자를 통해 객체가 만들어지면 다시는 또 만들 수 없으며, 또한 제약조건을 지정하는 함수가 있다는 특징이 있다. 블록체인에서의 인터럽트는 가능하지 못하여, 함수를 호출하는 시점에 이벤트를 발생하는 것으로 대신해서 프로그래밍할 수 있다. 또한 fallback함수라고 하는 일종의 예외적 상황에서 특이한 호출이 가능하다.

# 1. 함수의 선언

Solidity 함수를 크게 구분하면 다음과 같다.

(1) 제공되는 함수 **built-in functions** 함수는 내장되어 있고 지금까지 배워왔던 ```web3.eth.getAccounts()``` 뿐만 아니라 ```keccak256()```, ```sha256()```과 같이 여럿이 해당된다. 

반면에 (2) **사용자 정의함수 user-defined functions**는 우리가 직접 정의하고 만들게 된다.

함수 선언 문법을 보면 다음과 같다.

```python
function (param types) {internal|external} [pure|view|payable] [returns (return types)] varName;
```

사용자 정의함수를 만들 때는, 문법에 맞추어, 앞에 ```function```이라고 적어 함수 선언을 한다.
또한 함수의 입력  ```param types``` 및 출력  ```return types```를 정의해서 넣어주는 것이 필요하다.
가시성 ```internal, external``` 등과 수행하는 기능의 성격에 따라 ```pure```, ```view```, ```payable```의 수식어를 덧붙인다.

# 2. 생성자

생성자는 함수이기는 하지만, 조금 특별나다. 이전에는 보통의 함수와 같이 선언하고 만들어 주었지만, 버전 0.5부터는 그냥 constructor라는 별칭을 그대로 적어준다. 이 생성자 ```constructor()```를 통해 컨트랙의 객체가 생성된다.

생성자를 사용하는 이유는 컨트랙의 소유자를 정하거나, 필요한 변수를 설정하거나 초기화할 경우 사용한다.
생성자는 객체를 생성할 때 실행되는 코드이며, 배포하는 시점 딱 한 번만 실행이 된다.
다른 시점에 다른 코드에서 객체를 생성해도 생성자가 다시 실행되지는 않는다는 의미이다. 
* 생성자는 구현하지 않아도 되며, 그럴 경우 기본 생성자가 주어진다. 기본 생성자에는 물론 arg가 없다.
* 생성자는 1개만 허용된다. overloading은 할 수 없다는 의미이다.
* 반환 return이 없다
* 생성자는 외부에서 사용할 수 있도록 한다. 버전 0.7부터는 public 키워드가 필요하지 않다.
또는 **internal**로 선언하면, 자신말고는 다른 컨트랙이 생성할 수 없게 된다.

```python
contructor() public { ... } --> 이전 방식
contructor() { ... }   ---> 버전 0.7부터는 public 키워드 없이 적어준다.
```

# 3. 함수의 수식어

## 3.1 view, pure

Solidity에서는 함수를 실행하면, 그 결과로 인해 블록체인이 변경되는지 아닌지에 따라 구분하고 있다. 블록체인의 상태를 변경하지 않는 함수는 ```view```, ```pure```로 표기해야 한다.

```view```는 이전 ```constant```를 대체한 명령어로서, 상태를 수정하지 않는 경우, 즉 읽기 전용 read only 함수이다. 

```pure```는  ```view```보다 더 제약이 많은 함수에 붙이는 수식어이다. 함수의 지역변수만을 사용하며, 함수 밖의 그 어떤 변수를 수정하지 못할 뿐만 아니라, 상태 값을 읽지도 않는 경우에 사용한다. 

구분 | 설명
----------|----------
view | 상태를 수정하지 않는 경우
pure | 상태를 수정하지도 않으며, 읽지도 않는 경우

블록체인을 변경하는 **view, pure가 아닌 경우**에는 transaction이 발생하므로 **Transaction hash**가 반환된다. 즉 처리결과 값을 반환return 하지 않는다 점에 주의한다. 따라서 결과 값을 확인하는 방법은 event를 사용해서만 가능하다. event가 로그에 값을 쓰고, 로그에서 그 값을 읽어서 비로서 확인할 수 있다.

즉 ```pure``` 함수는 다음과 같은 경우는 할 수 없다.
* 상태변수 state variables를 읽는 경우,
* 잔고 ```this.balance``` 또는 ```<address>.balance```를 읽는 경우,
* 전역변수 ```block```, ```tx```, ```msg``` (```msg.sig```, ```msg.data``` 제외)를 읽는 경우,
* ```pure```로 표기되지 않은 함수를 호출하는 경우,
* ```opcodes```로 작성된 인라인 코드

또한 다음과 같은 경우는 블록체인을 수정하는 작업이기 때문에 ```view```는 사용할 수 없다.
- 상태변수를 저장하는 경우 (당연히 블록체인에 쓰는 경우이다)
- 이벤트 발생 (이벤트가 발생하면 로그에 기록이 남겨진다.)
- 다른 컨트랙 생성
- ```selfdestruct``` 컨트랙을 삭제하는 경우
- ```call``` 함수로 송금하는 경우
- ```view``` 또는 ```pure``` 아닌 함수를 호출하는 경우
- low-level calls을 호출하는 경우
- ```opcodes```로 작성된 인라인 코드


## 실습문제: 앞서 만들었던 함수의  view, pure 확인

앞서 작성했던 GlobalVarsTest.sol을 다시 돌아보자.
pure, view를 사용한 경우가 위에서 설명한 바와 같은지 확인하자.
송금하는 컨트랙도 잔고를 구할 경우 pure, view 어떤 수식어를 사용했는지 확인해보자. 


## 3.2 Payable

```payable``` 함수는 Ether를 입출금하는 경우에 사용된다.
web3 호출하는 측에서 { value: 0 }에 ether를 입력하면 전역변수 ```msg.value```로 전달된다.
컨트랙에서 호출하는 경우는 ```value()```를 사용한다.

## 3.3 가시성

* ```public``` 내, 외부에서 누구나 사용할 수 있다. 가장 개방적이다.
* ```internal``` 기본default 가시성으로, 내부 또는 상속관계에서만 사용할 수 있다.
* ```private``` 자신만 사용할 수 있다.
* ```external```은 ```internal```의 반대로서 외부에서만 사용할 수 있다. 내부에서 호출할 수 있지만 this를 통해서 ```this.f()```로 적어주어야 한다.

## 3.4 함수를 적어주는 순서

컨트랙 내에 함수를 배치할 때는 다음과 같은 순서로 적어준다 (참조 Solidity Style Guide)
- 생성자
- fallback 함수
- external 함수
- public 함수
- internal 함수
- private 함수

## 실습문제: 함수

앞서 설명한 다양한 성격의 함수를 정의하고, 구현해보자.

줄 | 설명
-----|-----
4 ~ 7 ```constructor()``` | 생성자는 앞에 위치시킨다.
8 ~ 11 ```increatementX()``` | state var를 수정하므로 ```view```, ```pure```가 아님.
13 ~ 15 ```doubleX()``` | 내부함수 ```internal X2()```를 호출함. state var 수정하므로 ```view```, ```pure``` 아님.
17 ~ 19 ```divideBy(int by)``` | 0으로 나누어도 오류를 발생하지 않는다. 아직 소수는 지원하지 않으므로 필요한 경우 라이브러리를 사용한다.
21 ~ 23 ```getX_()``` | ```view```, ```pure```가 아니므로```returns(int)```해도 반환 값을 받지 못함. 해시값을 돌려줌.
25 ~ 27 ```getX()``` | state var를 읽으므로 ```view```를 사용한다.  ```pure```를 사용하지 않는다.
28 ~ 30 ```getBalance()``` | 잔고 조회는 view도 가능함.
32 ~ 33 ```deposit()``` | ```payable```로 선언해야 입금이 가능. 함수가 비워있어도 ```msg.value```를 입금.
35 ~ 37 ```X2()``` | ```internal```로 선언되어서 내부에서만 사용가능.
38 ~ 40 ```getBlockNumber()``` | 전역변수를 읽을 경우 ```view```

```python
[파일명: src/FuctionTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract FunctionTest {
    int x;
    //constructor
    constructor() { //constructor() public {
        x = 0;
    }
    // updating state var x
    function incrementX() public {
        x += 1;
    }
    function doubleX() public {
        X2();  //아래에 internal로 정의. 현재 public에서 호출.
    }
    // float not supported. try 0, 1/3...
    function divideBy(int by) view public returns(int) {
        return x/by;
    }
    // actually NONE returned
    function getX_() view public returns(int) {
        return x;        
    }
    // try pure
    function getX() view public returns(int) {
        return x;
    }
    function getBalance() view public returns(uint) {
        return(address(this).balance);
    }
    // 'payable' means that msg.value can be deposited
    function deposit() public payable {
    }
    // can not be used in public
    function X2() internal {
        x *= 2;
    }
    function getBlockNumber() view public returns(uint) {
        return block.number;
    }
}
```

컴파일을 해보고, 오류가 있는지 확인하자. 
```python
pjt_dir> solc-windows.exe src/FuctionTest.sol
```

# 4. 함수의 제약조건을 지정하는 modifier

함수의 **제약조건**으로 사용된다. 즉 어떤 함수가 실행되려면 만족해야 하는 조건을 modifier로 정할 수 있다. Owner인 경우, 일정한 기간에 포함되는 경우, 짝수에 해당하는 경우 등 조건을 걸어 그런 경우에만 함수를 실행할 수 있게 한다. 이와 같이 modifier를 적용하면 **if문을 제거하는 효과**가 있게 된다.

또한 modifier는 보통 함수가 그런 것처럼 **상속**이 가능하다. 또한 override할 수도 있다.

## 실습문제: 밑줄 명령어 적용

modifier를 사용할 때 밑줄 underscore ```_``` 명령어를 통해 어디에 호출 소스코드가 포함되는지 분명히 할 수 있다.
아래 코드에서 포함되는 시점을 전 또는 후로 정할 수 있다.


```python
[파일명: src/UnderscoreTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract UnderscoreTest {
    string season = "";
    function getSeason() view public returns(string memory) {
        return season;
    }
    function setWinter() public setSummerAfter {
        season = "winter";
    }
    function setSpring() public setSummerBefore {
        season = "spring";
    }
    modifier setSummerAfter() {
        season = "summer";
        _;
    }
    modifier setSummerBefore() {
        _;
        season = "summer";
    }
}
```

```python
pjt_dir> solc-windows.exe src/UnderscoreTest.sol
```

REMIX에서 실행해보자.
* setSpring() 한 다음에 getSeason() 호출하면 summer가 출력된다
    * ```_```코드가 먼저 있고, 그 ```_```자리에 spring이 먼저 실행되고 다음에 summer가 실행되기 때문이다.
* 반면에 setWinter() 하고, getSeason() 하면 winter가 출력되는데, 그 이유는 summer가 먼저 설정되고 ```_```자리에 winter가 실행되므로 그렇다. 

## 실습문제: 은행 BankV3

은행에는 입출금 관련 제한을 걸어 놓는 경우가 있다. 입금시 최소시간, 출금시 최소한도 제한 규정을 구현해보자.

### 단계 1: 컨트랙 개발

앞의 은행 코드에 fallback, modifier를 넣어서 수정한다.
* 입금시 최소시간 제한 modifier
입금을 연달아 할 경우, 반드시 일정시간이 지난 후에야 입금을 가능하게 하고 있다. ```onlyAfter()``` 함수, 정확히는 modifier는 지난 입금 시점 후 10초로 설정하고 있다. 코드로 쓰면 ```block.timestamp + 10 seconds```이다.

* 출금시 최소잔고 제한 modifier
```minBalance()``` 함수( modifier)는 일정 잔고 이상인지 제한하고 있다. 이 modifier를 제한조건으로 ```withdrawAll()``` 함수는 최소잔고를 초과할 경우에만 출금이 되도록 한다.

* fallback() 함수는 단순히 이벤트 ```PrintLog("Fallback called")```만 발생하도록 작성한다.

REMIX에서 버튼을 눌러서 기능을 테스트 하면서 진행하자. 조회 버튼과 달리, deposit() 함수를 테스트할 경우에, 상단의 'VALUE'와 버튼 옆의 함수 인자 두 필드에 동일한 금액을 넣어준다. 즉 실제 금액과 인자 금액이 일치하도록 하자.

```python
[파일명: src/BankV3.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0; //^0.5.0;

contract BankV3 {
    address owner;
    uint balance;
    uint256 timeToDeposit;
    
    event PrintLog(string);
    event Sent(address from, address to, uint amount );
    constructor() { //constructor() public {
        owner = msg.sender;
        balance = 0;
    }
    fallback() external { // v0.5.0 function() external {
        emit PrintLog("Fallback called");
    }
    function forwardTo(address payable _receiver) public payable onlyOwner {
        //require(msg.sender == owner);
        _receiver.transfer(msg.value);
        emit Sent(msg.sender, _receiver, msg.value);
    }
    function getBalance() public view returns(uint, uint) {
        return (balance, address(this).balance);
    }
    function deposit(uint amount) public payable onlyAfter {
        timeToDeposit = block.timestamp + 10 seconds; //timeToDeposit = now + 10 seconds;
        require(msg.value == amount);
        balance += amount;
    }
    function withdrawAll() public onlyOwner minBalance {
        balance -= address(this).balance;
        //require(msg.sender == owner);
        //v0.5.0 msg.sender.transfer(address(this).balance);
        payable(msg.sender).transfer(address(this).balance); 
    }
    function getTimeToDeposit() view public returns(uint256) { return timeToDeposit; }
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
    modifier onlyAfter {
        require(block.timestamp >= timeToDeposit); //require(now >= timeToDeposit);
        _;
    }
    modifier minBalance {
        require(address(this).balance>101 wei);
        _;
    }
}
```

### 단계 2: 컴파일

컴파일 해서, abi와 bytecode를 파일에 저장한다.

```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/BankV3.sol > src/BankV3.json
```

### 단계 3: 컨트랙 배포

이번에는 ```async```, ```await```로 변경하여 코드를 작성해보자.
배포함수를 ```async function deploy()```로 선언한다.
그리고 그 안에 ```await web3.eth.getAccounts()```로 계정을 구해서 이를 활용한다.
컨트랙을 생성하기 위해 블록체인에 전송하는 ```await send()``` 함수도 비동기적으로 처리하여, 주소를 출력한다.

```python
[파일명: src/BankV3DeployFromFile.js]
var Web3 = require('web3');
var _abiBinJson = require('./BankV3.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/BankV3.sol:BankV3']
console.log("- contract name: ", contractName);
_abi=_abiBinJson.contracts[contractName].abi;
_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin=_abiBinJson.contracts[contractName].bin;

//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 259210}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

배포 코드에 gas를 수정하면서 어떤 오류가 발생하는지 경험해보자. 
* gas를 쓰지 않으면 gas 한도 관련 오류가 발생 "Error check your gas limit"
* gas를 대략 259210 미만을 적으면 gas가 너무 낮다는 오류가 발생 "intrinsic gas too low"

```python
pjt_dir> node src/BankV3DeployFromFile.js

- contract name:  [ 'src/BankV3.sol:BankV3' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0xd81a2aceb7edfccfade9b2ff7f94fa5928459e0975275a1615232398f7858425
---> The contract deployed to: 0x0042048e0e97BA996CA18fC7d027379ed786Af7a
```

### 단계 4: 사용

node에서 대화형식으로 해보자.  주의! geth@8445에서 하려면 deposit(), withdrawAll() 함수들은 마이닝이 필요하다. 

프로그램에 설정한 다음의 제약조건이 지켜지는지 살펴보면서 실행해보자.
* 10초 이내 저축
* 잔고 101보다 적은데 출금

```python
bank 객체는 스스로 만들어 보자 (아래 코드를 참조해서 만들어도 된다)
node> bank.methods.getBalance().call().then(console.log);              잔고 0
> 0
> bank.methods.deposit(100).send({from:"0x4D2fF...", value:100});    입금 100 (geth@8445는 마이닝 필요)
(geth@8445는 마이닝이 필요하다 geth> miner.start(1);admin.sleepBlocks(1);miner.stop())
> bank.methods.getBalance().call().then(console.log);               입금금액 조회 100.
> 100                                                                
> bank.methods.deposit(100).send({from:"0x4D2fF...", value:111});   입금 'value'와 '인자'가 서로 다르면 입금 실패
(geth@8445는 마이닝이 필요하다)
> bank.methods.getBalance().call().then(console.log);              실패하였으므로 잔고는 계속 100
> 100
> bank.methods.deposit(111).send({from:"0x4D2fF...", value:111});    입금 111
(geth@8445는 마이닝이 필요하다)
> bank.methods.getBalance().call().then(console.log);              입금 100+111 = 211
> 211
> bank.methods.deposit(111).send({from:"0x4D2fF...", value:111});    (10초 지나서) 입금 111, 마이닝하면 금액 증가함
(geth@8445는 마이닝이 필요하다)
> bank.methods.getBalance().call().then(console.log);              입금 100+111+111 = 322
> 322
> bank.methods.deposit(111).send({from:"0x4D2fF...", value:111});    입금 111, 마이닝하면 금액 증가함
> (geth@8445는 마이닝이 필요하다 geth> miner.start(1);admin.sleepBlocks(1);miner.stop())
> bank.methods.deposit(111).send({from:"0x4D2fF...", value:111});    입금 111, 10초 이내이므로 마이닝해도 금액 증가하지 않음
(geth@8445는 마이닝이 필요하다)
> bank.methods.getBalance().call().then(console.log);              여러번 입금해도 10초 이내 거래는 실패하고 잔고가 늘지 않음
> 433
```

이번에는 묶어서 일괄실행해보자. 블록체인에 send()가 필요한 함수는 비동기적으로 처리하기 위해 ```await```로 처리한다.
비동기적으로 처리하면, 예를 들어 입금 ```deposit()```하고 ```getBalance()```하면 잔고에 입금분만큼 반영이 되어있는 것을 알 수 있다.

```python
[파일명: src/BankV3Use.js]
var Web3=require('web3');
var _abiBinJson = require('./BankV3.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/BankV3.sol:BankV3']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
var bank = new web3.eth.Contract(_abiArray,"0x0042048e0e97BA996CA18fC7d027379ed786Af7a");
//var filter = bank.PrintLog(function (error, result) {
//  if (!error)
//    console.log(result);
//});
//console.log(bank.sendTo(0x778ea91cb0d0879c22ca20c5aea6fbf8cbeed480, 100,{from:web3.eth.accounts[0],gas:100000}));

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    bank.methods.getBalance().call().then(console.log);
    await bank.methods.deposit(111).send({from: accounts[0], value:111});
    bank.methods.getBalance().call().then(console.log);
    await bank.methods.withdrawAll().send({from: accounts[0]});    //greater than 101
    bank.methods.getBalance().call().then(console.log);
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));    
}
doIt()
```

```python
pjt_dir> node src/BankV3Use.js

- ABI: [object Object],[object Object],[object Object],[object Object],[object Object],[object Object],[object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999978775921999995002     거래 이전의 잔고
Result { '0': '0', '1': '0' }             잔고반환은 2개를 하고 있다. 검증목적으로 2개가 동일해야 한다.
Result { '0': '111', '1': '111' }         111을 입금
Balance after: 999978617241999995002
Balance diff: 158679999971328             (1)
Result { '0': '0', '1': '0' }             전액 출금 (101 제약보다 큰 잔고가 있으므로 가능하다)
```

위 출력 (1)을 잠깐 보자. balanceBefore와 balanceAfter의 연산이 정확하지 않다. 눈치를 채었는가?
왜 그런지 자바스크립트 콘솔에서 확인해도 오류가 있다.
```
Javascript > 999978775921999995002-999978617241999995002
> 158679999971328
```

자바스크립트는 정수는 2의 53승까지, 9007199254740992가 최대 값이다. 그 수를 넘어가게 되면 연산이 올바르게 되지 않는다. 계산기로 해보면, 출력값과 다르다 (158,680,000,000,000).

# 5. 이벤트

이벤트는 프로그램에 의해 감지될 수 있는 동작을 의미한다.
키보드에서 어떤 키를 누르거나, 마우스로 클릭하거나 하는 것이 모두 이벤트에 해당한다.

블록체인에서는 이런 이벤트가 발생할 수 없다고 하겠다.
키보드나 마우스가 운영체제의 인터럽트 interrupt를 발생하는 것인데,
블록체인에는 그 특성 상 이러한 마우스 클릭이나 키를 누르거나 하는 이벤트가 발생할 수 없다.
반면에 어떤 함수가 호출되면 그 것을 알려주는 이벤트가 발생하는 것으로 처리하고 있다.
블록체인에서의 이벤트는 **로그**에 기록이 된다.
그 로그를 계속 듣고 있다가, 자신이 원하는 것이 포착되면 그 것이 이벤트로 인식된다.

## 5.1 이벤트 발생하기

### 단계 1. 이벤트 설정

컨트랙의 멤버변수 자리에 이벤트를 설정한다.
명령어 ```event 이벤트명```을 아래에 보이는 예와 같이 적어준다.
```python
event PrintLog()
```

이벤트명은 카멜케이스로 단어 첫 글자를 대문자로 적어준다.
이벤트에는 매개변수를 넣어줄 수 있다.


### 단계 2. 함수를 호출할 때 이벤트가 발생하도록 연결한다.

함수 내, 이벤트가 발생하게 되는 곳에 ```emit 이벤트명```이라고 적어준다.
그렇게 되면 함수가 호출되고
**emit 명령어를 만나게 되면 이벤트가 발생**하게 되는데 이런 연결을 바인딩 **binding**이라고 한다.

```python
function fireEvent() {
    emit PrintLog()  이 시점에 이벤트가 발생한다. 즉 로그에 PrintLog()가 적힌다.
    ...
}
```

### 단계 3. 이벤트의 발생을 callback 함수로 리스닝한다.

이벤트가 발생했는지 리스닝하려면
객체의 events.이벤트명으로 접근해야 한다.

```events.PrintLog()``` 이런 식으로 리스닝하게 된다.
이벤트 함수에 매개변수가 있어도 생략할 수 있다.
JavaScript callbacks을 통해 이벤트의 발생을 리스닝하는 기간을 정할 수 있다.
```fromBlock```의 시작부터 ```toBlock```의 종료시점까지 특정 블록을 정해서 리스닝할 수 있다.

```python
1) 이벤트를 생성하고 리스닝한다.
> var myEvent = myInstance.events.PrintLog({from: web3.eth.accounts[0]}, {
    fromBlock: 0,     //시작 블록 수
    toBlock: 'latest' //끝 블록 수
  });
> myEvent.watch(function (error, result) {
    if (!error) {
        console.log("Event triggered ===> ",result);
        process.exit(1);
    }
});

2) 또는 한 명령어로 합쳐서 다음과 같이 리스닝할 수 있다.
var myEvent = myInstance.events.PrintLog({fromBlock: 0}, fromfunction(error, event) {
    if (!error)
        console.log(event);
});
```


### 단계 4. 함수호출로 이벤트 발생.

이벤트는 당연히 발생해야 알 수가 있다.
이벤트가 설정된 함수 호출되면 이벤트가 발생하는 것이다.
```_instance.MyFunction()``` or ```_instance.MyEvent.watch()```

```python
> myInstance.fireEvent();
```

### 단계 5. 이벤트 watch 중지
```python
> myEvent.stopWatching();
```

### event indexing

매개변수를 인덱싱한다는 것은, 빠른 검색을 위해 색인을 만들어 놓는다는 의미이다.
매개변수 3개까지는 로그에서 인덱스로 사용할 수 있고,
매개변수의 해시 값이 로그의 ```topic```으로 출력된다.

예를 들어, Transfer(address indexed _from, address indexed _to, uint256 _value) 이렇게 인덱스가 잡혀 있다면
이 경우 전체가 아니라, 송신주소, 수신주소로 국한지어 검색할 수 있게 된다.
```python
contract.Event({from:"0xc32c36e7cea09c00e0dbd4bb4e72b8baaa51fd45"},{fromBlock: 0, toBlock: 'latest'});
```

### event overloading

이벤트는 **상속**할 수 있어서, 부모가 정의한 이벤트를 자식이 물려받아 발생시킬 수 있다.
컨트랙에 이벤트를 여러 개 가질 수 있는지의 문제이다.
이벤트 **오버로딩**도 가능한지 해보자.

## 5.2 웹소켓

### 연결과 해제

이벤트가 발생하고 이를 포착하기 위해서는 웹소켓에 연결해야 한다.
연결하기 위해서는 먼저 provider 객체를 만들고, 이를 ```Web3(provider 객체)``` 생성자에 매개변수로 넘겨주어야 한다. 이 때 ```ws://``` 프로토콜을 사용한다.
```
> var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345")); 웹소켓에 연결 개시
undefined
> web3.currentProvider.connected 연결상태 true 출력
true
> web3.currentProvider.connection.close() 웹소켓을 해제
undefined
> web3.currentProvider.connected 연결상태 false 출력
false
```

### 설정

웹소켓의 연결을 설정할 수 있다. 연결 상태를 ```on()``` 함수를 이용해서 알아볼 수 있다.
```
> var myprovider = new Web3.providers.WebsocketProvider("ws://localhost:8345", {
    clientConfig: {
        keepalive:true, keepaliveInterval:10000
    } 
  });
> myprovider.on('connect', function() { console.log("--> Websocket connected"); });
undefined
> myprovider.on('close', function() { console.log("--> Websocket closed"); });
undefined
> myprovider.on('end', function() { console.log("--> Websocket closed"); });
undefined
> myprovider.on('error', function(error) { console.error(error); });
undefined
> var web3 = new Web3(myprovider);
```

## 실습문제: 웹소켓 연결과 해제

웹소켓에 연결해보자.
연결한 후 상태를 출력하고, 즉시 해제한다.
해제하면서 어떤 상태가 출력이 되는지 확인해보자.


```python
[파일명: src/webSocketTest.js]
var Web3 = require('web3');
const myProvider = new Web3.providers.WebsocketProvider("ws://localhost:8345", {
    clientConfig: {
        keepalive:true, keepaliveInterval:10000
    } 
  });
var web3 = new Web3(myProvider);
console.log("(1) websocket url: ", myProvider.connection.url); //web3.currentProvider.connection.url
myProvider.on('connect', function() {
    console.log("(2) connecting websocket: "+web3.currentProvider.connected);
    //myProvider.disconnect();
    web3.currentProvider.connection.close();
    console.log("(3) disconnecting Websocket: "+web3.currentProvider.connected);
});
myProvider.on('close', function() { console.log("--> Websocket closed"); });
myProvider.on('end', function() { console.log("--> Websocket ended"); });
myProvider.on('error', function(error) { console.error(error); });

```


```python
pjt_dir> node src/webSocketTest.js

(1) websocket url:  ws://localhost:8345
(2) connecting websocket: true
(3) disconnecting Websocket: false
--> Websocket ended
```

## 실습문제: 간단한 이벤트 발생

이벤트를 만들고 함수가 호출되는 시점에 발생하도록 해보자.
이벤트는 발생하는 시점에 "Hello World!" 문자열이 출력되도록 한다.
이벤트가 발생하는지는 클라이언트에서 리스닝해서 알아 낸다.
이벤트가 발생하면 로그에 기록이 되고, 리스닝한다는 것은 이런 로그의 기록에 이벤트가 발생했는지를 확인하는 것이다.

### 단계 1: 컨트랙 개발


```python
[파일명: src/EventTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract EventTest {
    event MyLog(string my);
    function myFunction() public {
        emit MyLog("Hello World!");
    }
}
```

### 단계 2: 컴파일


```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/EventTest.sol > src/EventTest.json
```

### 단계 3: 컨트랙 배포

사설망에 배포할 경우에는 계정 암호 풀어야 ```authentication needed: password or unlock``` 오류가 발생하지 않는다.


```python
[파일명: src/EventTestDeployFromFile.js]
var Web3 = require('web3');
var _abiBinJson = require('./EventTest.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/EventTest.sol:EventTest']
console.log("- contract name: ", contractName);
_abi=_abiBinJson.contracts[contractName].abi;
_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin=_abiBinJson.contracts[contractName].bin;

//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 259210}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

```python
pjt_dir> node src/EventTestDeployFromFile.js

- contract name:  [ 'src/EventTest.sol:EventTest' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0xb7095ec9d332c32b6f456692746f8811d1da25deabf5cad5346d47d8592c5bb1
---> The contract deployed to: 0x8911bA097c812Bf0B3ff22F90eaf2A905112C5a6
```

위에서 주어진 ```transactionHash```를 가지고 처리결과를 알 수 있다.
```
geth> eth.getTransactionReceipt("해시번호를 여기에 적어준다")
```
컨트랙주소, gas 사용량, 거래 hash도 찾을 수 있다.

### 단계 4: 사용

```JSON.stringify```는 Object의 내용을 문자열로 출력한다.
파일에서 읽은 JSON을 그대로 사용해도 오류가 발생하지 않는다.

```python
[파일명: src/EventTestHttpNoEventFiredUse.js]
var Web3=require('web3');
var _abiBinJson = require('./EventTest.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/EventTest.sol:EventTest']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
var _test = new web3.eth.Contract(_abiArray,"0x8911bA097c812Bf0B3ff22F90eaf2A905112C5a6");
var event = _test.events.MyLog({fromBlock: 0}, function (error, result) {
    if (!error) {
        console.log("Event fired: " + JSON.stringify(result.returnValues));
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    const value = await _test.methods.myFunction()
        .send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(value.events.MyLog.returnValues));});
    console.log("---> myFunction called " + JSON.stringify(value.events.MyLog.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}
doIt()
```

HttpProvider는 Event를 호출하지 못하고 있다. 함수 호출하면서 발생한 로그를 ```value.events.MyLog.returnValues```를 통해 출력할 수 있다.

```python
pjt_dir> node src/EventTestHttpNoEventFiredUse.js

- ABI: [object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999978310622999995002
---> myFunction called {"0":"Hello World!","my":"Hello World!"}
Balance after: 999978287795999995002
Balance diff: 22827000004608
```

#### WebSocketProvider

위 프로그램에서 HttpProvider --> WebsocketProvider으로 변경한다. 사설망을 사용하는 경우, WebSocket이 가능하도록 설정해야 한다.

#### 파일에 로그 쓰기

이벤트가 발생하면 다음 시나리오를 생각해보자.
로컬에 다른 프로세스를 호출하는 작업을 할 수 있다. 
다른 함수, 예를 들면, 로컬에서 주문상품을 배송하게 할 수 있다.
파일에 쓰는 시나리오도 가능하다.

```fs.writeFile(파일명, 데이터, 인코딩방식, callback함수)```함수를 사용하여 이벤트 발생 로그를 로컬 파일에 써보자.
```fs.appendFile()```은 파일에 추가하는 기능이다.

계정 암호 해제해야 한다. (node:19884) UnhandledPromiseRejectionWarning: Error: Returned error: authentication needed: password or unlock


```python
[파일명: src/EventTestWsUse.js]
var Web3=require('web3');
var fs = require('fs');
var _abiBinJson = require('./EventTest.json');      //importing a javascript file

//var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://localhost:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/EventTest.sol:EventTest']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
//var _test = new web3.eth.Contract(_abiArray,"0x8911bA097c812Bf0B3ff22F90eaf2A905112C5a6");
//var event = _test.events.MyLog({fromBlock: 0}, function (error, result) {
//    if (!error) {
//        console.log("Event fired: " + JSON.stringify(result.returnValues));
//    }
//});

async function doIt() {
    var _test = new web3.eth.Contract(_abiArray, '0x8911bA097c812Bf0B3ff22F90eaf2A905112C5a6');
    var event = _test.events.MyLog({fromBlock: 0}, function (error, result) {
        if (!error) {
            log = JSON.stringify(result.returnValues);
            console.log("Event fired: " + log);
            //fs.writeFile("src/EventTestLog.txt", log, "utf-8", function(e) {
            fs.appendFile("src/EventTestLog.txt", log, "utf-8", function(e) {
                if(!e) {
                    console.log(">> Writing to file");
                }
            });
        }
    });
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    const value = await _test.methods.myFunction()
        .send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(value.events.MyLog.returnValues));});
    console.log("---> myFunction called " + JSON.stringify(value.events.MyLog.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    process.exit(1); //force exit to disconnect websocket
}

doIt()
```

IP가 올바르지 않는 경우 ```Error: connection not open``` 오류가 발생한다.
자, 이번에는 이벤트가 발생하는가? 아래 출력에서  ```Event fired: {"0":"Hello World!","my":"Hello World!"}```가 출력되고 있다.

```python
pjt_dir> node src/EventTestWsUse.js


- ABI: [object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Event fired: {"0":"Hello World!","my":"Hello World!"}
Balance before: 999978082352999995002
>> Writing to file
Event fired: {"0":"Hello World!","my":"Hello World!"}
>> Writing to file
---> myFunction called {"0":"Hello World!","my":"Hello World!"}
Balance after: 999978059525999995002
Balance diff: 22827000004608
```

이벤트 발생하면서 파일에는 아래와 같이 JSON 형식으로 작성된다.

```python
pjt_dir> type src\EventTestLog.txt

{"0":"Hello World!","my":"Hello World!"}{"0":"Hello World!","my":"Hello World!"}{"0":"Hello World!","my":"Hello World!"}{"0":"Hello World!","my":"Hello World!"}{"0":"Hello World!","my":"Hello World!"}
```

## 실습문제: 주문하면서 복수의 이벤트 사용

### 단계 1: 컨트랙 개발

변수 ```_itemId```는 ```bytes2```으로 선언되어있다. 값을 입력할 경우 16진수라서 4자리를 넣었다 (예: "0x1234")

```python
[파일명: src/OrderEvent.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract OrderEvent {
    uint unitPrice = 10;
    event OrderLog(string);
    event OrderLog(bytes2 _itemId, uint _value);
    event OrderLog(uint256 timestamp);
    event OrderLog(address indexed _from, bytes2 _itemId, uint indexed _value);

    function order(bytes2 _itemId, uint quantity) public payable {
        //uint256 orderTime = now;
        uint256 orderTime = block.timestamp;
        uint256 orderAmount = quantity * unitPrice;
        require(msg.value == orderAmount);
        emit OrderLog("Ordered");
        emit OrderLog(orderTime);
        emit OrderLog(msg.sender, _itemId, msg.value);
    }
}
```

### 단계 2: 컴파일

```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/OrderEvent.sol > src/OrderEvent.json
```

### 단계 3: 컨트랙 배포

컴파일하고 출력되는 abi, bin을 복사해서 배포 프로그램에 넣어주자.
사설망(geth@8445)에 배포하려면, 계정을 unlock하는 것을 잊지 말자.

```python
geth> personal.unlockAccount(eth.accounts[0]);
Unlock account 0x21c704354d07f804bab01894e8b4eb4e0eba7451
Passphrase: 
```

```python
[파일명: src/OrderEventDeploy.js]
var Web3 = require('web3');
var _abiBinJson = require('./OrderEvent.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/OrderEvent.sol:OrderEvent']
console.log("- contract name: ", contractName);
_abiArray=_abiBinJson.contracts[contractName].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin=_abiBinJson.contracts[contractName].bin;

//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 259210}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```


```python
pjt_dir> node src/OrderEventDeploy.js

- contract name:  [ 'src/OrderEvent.sol:OrderEvent' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0xddf0be098411f1e36a9906c953a240eca4f141634853bf15fc5b7d6e1c6a532d
---> The contract deployed to: 0x3E04292870AA4Ef2bc44A1638B19A50BCD99b04D
```

### 단계 4: 사용

event는 async방식으로 ```order()```함수가 호출되면 실행된다.
Order에 넘겨주는 매개변수는 바이트는 2바이트를 채워서 "0x1234", 주문량은 정수 3으로 맞추어 준다.
호출한 클라이언트에 event의 argument를 받아볼 수 있다 (예: result.args._from은 주소)

currentProvider는 MetaMask, Mist와 같은 Wallet을 사용하는 경우 자동으로 설정된다.

라인 | 설명
-----|-----
6 | WebsocketProvider으로 설정한다.
17 | ```filter: {_from: accounts[0], _value: 30}``` 인덱싱 걸어놓은 이벤트에 대해 해당 조건에 맞는 경우만 출력한다. 따라서 40, 100은 출력하지 않는다.
18 | ```fromBlock: 'latest', toBlock: 'pending'``` 최근부터 대기하는 이벤트를 출력한다. ```fromBlock```의 ```default```는 ```latest```이다. 따라서 생략해도 된다. ```fromBlock```: 0, ```toBlock```: 'latest'는 처움 블록부터 발생한 이벤트를 모두 출력한다.
35 ~ 39 | 이벤트 로그에 대해 인덱싱에 해당되지 않아, ```my.events.OrderLog.returnValues``` 필드가 없게 되어 ```undefined```로 출력하고 있다.

```python
[파일명: src/OrderEventUse.js]
var Web3=require('web3');
//var fs = require('fs');
var _abiBinJson = require('./OrderEvent.json');      //importing a javascript file

//var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://localhost:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/OrderEvent.sol:OrderEvent']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
//var _test = new web3.eth.Contract(_abiArray,"0x8911bA097c812Bf0B3ff22F90eaf2A905112C5a6");
//var event = _test.events.MyLog({fromBlock: 0}, function (error, result) {
//    if (!error) {
//        console.log("Event fired: " + JSON.stringify(result.returnValues));
//    }
//});

async function doIt() {
    var _order = new web3.eth.Contract(_abiArray, '0x3E04292870AA4Ef2bc44A1638B19A50BCD99b04D');
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    var event = _order.events.OrderLog({
            //filter: {_from: accounts[0], _value: [20,50]},
            filter: {_from: accounts[0], _value: 30},
            fromBlock: 'latest', toBlock: 'pending'
        }, function (error, result) {
        if (!error) {
            //console.log("Event fired: " + JSON.stringify(result) + "\n---> " + JSON.stringify(result.returnValues));
            console.log("Event fired: " + JSON.stringify(result.returnValues));
            //process.exit(1);
        }
    });
    var value;
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    // this will fire an event
    my = await _order.methods.order("0x1234", 3)
        .send({from: accounts[0], gas: 100000, value:30})
        //.then(function(my) {console.log("---> MyFunction called " + JSON.stringify(my.events.OrderLog.returnValues));});
    console.log("---> MyFunction called " + JSON.stringify(my.events.OrderLog.returnValues));
    // this will fire another event
    my = await _order.methods.order("0x1234", 4).send({from: accounts[0], gas: 100000, value:40});
    console.log("---> MyFunction called " + JSON.stringify(my.events.OrderLog.returnValues));
    // this will NOT fire another event
    my = await _order.methods.order("0x1234", 10).send({from: accounts[0], gas: 100000, value:100});
    console.log("---> MyFunction called " + JSON.stringify(my.events.OrderLog.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));    
    process.exit(1); //force exit to disconnect websocket
}

doIt()
```

첫째 인자는 키를 가지고 있다.

```python
pjt_dir> node src/OrderEventUse.js

- ABI: [object Object],[object Object],[object Object],[object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999977145423999994462
Event fired: {"0":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","1":"0x1234","2":"30","_from":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","_itemId":"0x1234","_value":"30"}
---> MyFunction called undefined
---> MyFunction called undefined
---> MyFunction called undefined
Balance after: 999976974723999994292
Balance diff: 170700000067584
```

## 실습문제: Multiply7 연산결과를 확인하려면 이벤트를 통해서 한다.

앞서 Multiply7을 구현해 보았다.
문제는 '연산결과를 어떻게 알아볼 수 있을까?'
블록체인에서는 반환을 하지 않아서, 그 결과를 알아보려면 멤버변수에 저장해야 할 경우도 있다고 하였다.
컨트랙에 이벤트를 추가하는데, 여기에 반환 값을 매개변수로 넣어서 알아보자.
아래 주어진 미완성 코드를 구현해서 노드에서 이벤트를 띄워보자.

* 함수는 param4인자를 받아서 곱하기 7 연산을 한다. 8과 16의 연산을 실행하자. 그 결과는 56과 112가 출력될 것이다.
* 이벤트로 넘겨주는 매개변수 param1은 누가 이 함수를 호출했는지 (address), param2는 언제 호출되었는지 (timestamp), 마지막으로 param3는 연산결과를 출력하도록 한다.

```python
contract Multiply7Event {
    event Print(param1, param2, param3);
    function multiply(param4);
}
```

### 단계 1: 컨트랙 개발

```multiply()``` 함수는 ```pure```로 선언되어야 마땅하지만, ```event```를 발생하므로 ```pure``` 또는 ```view```가 될 수가 없다.
따라서 값을 반환하는 것이 의미가 없고, 다음과 같이 
```function multiply(uint input) public``` 선언하는 것이 바람직하다.


```python
[파일명: src/Multiply7Event.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Multiply7Event {
    event Print(address _addr, uint256 timestamp, uint res);
    //function multiply(uint input) public returns(uint) {
    function multiply(uint input) public {
        uint res = input * 7;
        emit Print(msg.sender, block.timestamp, res); //emit Print(msg.sender, now, res);
        //return res;
    }
}
```

### 단계 2: 컴파일

```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/Multiply7Event.sol > src/Multiply7Event.json
```

### 단계 3: 컨트랙 배포

위 abi, bin을 복사해서 붙여 넣고 배포해보자.

```python
[파일명: src/Multiply7EventDeploy.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var _abiArray=[{"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_addr","type":"address"},{"indexed":false,"internalType":"uint256","name":"timestamp","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"res","type":"uint256"}],"name":"Print","type":"event"},{"inputs":[{"internalType":"uint256","name":"input","type":"uint256"}],"name":"multiply","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"nonpayable","type":"function"}];
var _bin="0x" + "6080604052348015600f57600080fd5b506101278061001f6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063c6888fa114602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b6000806007830290507f91da4985ab616136202f4e81fd2d8cac1eb12591132d609cece407f7c6fb9205334283604051808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001838152602001828152602001935050505060405180910390a18091505091905056fea26469706673582212201f2b58641bcadee708439f7e52d3b7d33168278dbdc6285f760ef41356a006df64736f6c63430006010033";
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000, gasPrice: '1000000000'}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance) {
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}

deploy()

```

```python
[파일명: src/Multiply7EventDeploy.js]
var Web3 = require('web3');
var _abiBinJson = require('./Multiply7Event.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/Multiply7EventDeploy.sol:Multiply7EventDeploy']
console.log("- contract name: ", contractName);
_abiArray=_abiBinJson.contracts[contractName].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin=_abiBinJson.contracts[contractName].bin;

//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 259210}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```


```python
pjt_dir> node src/Multiply7EventDeploy.js

- contract name:  [ 'src/Multiply7Event.sol:Multiply7Event' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x0999eeb551e76a473873d2880a1f6c8088a42d3f30e2c439ac1b1d13cb78f35b
---> The contract deployed to: 0x33add2effA3E32050aCD8446d826b0EFFB93A515
```

### 단계 4: 사용

```m7.Print(function (error, result)``` 명령문으로 이벤트를 리스닝하고, 이벤트가 발생하는 경우, 그 결과 ```result```를 출력하고 있다.
이 이벤트는 ```multiply()``` 함수 내에서 발생되고 있다.
그런 까닭에 **```multiply()``` 함수는 ```pure```로 선언할 수 있지만 ```event```를 발생하기 때문에 그렇지 못했다**.
따라서 ```web3```에서 ```transaction```으로 호출해야 하지만. 아래 예제는 **```call()```로 호출**하였다.
```call()``` 함수는 트랜잭션을 발생시키지 않으므로 결과는 알 수 있지만, **이벤트는 발생하지 않는다**.

```multiply()```함수는 state variable을 갱신하지 않으므로, ```call()```로 호출해도 연산 계산을 할 수 있고, 이벤트로 값이 전달된다.

```python
[파일명: src/Multiply7EventUse.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    var web3 = new Web3(new Web3.providers.WebsocketProvider("http://localhost:8345"));
}
var _abiArray=[{"anonymous":false,"inputs":[{"indexed":false,"internalType":"address","name":"_addr","type":"address"},{"indexed":false,"internalType":"uint256","name":"timestamp","type":"uint256"},{"indexed":false,"internalType":"uint256","name":"res","type":"uint256"}],"name":"Print","type":"event"},{"inputs":[{"internalType":"uint256","name":"input","type":"uint256"}],"name":"multiply","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"nonpayable","type":"function"}];
var m7 = new web3.eth.Contract(_abiArray, '0x85c1cA1755f706c5228107235f7e5C510e77d392');
var event = m7.events.Print({fromBlock: 0}, function (error, result) {
    if (!error) {
        //console.log("Event fired: " + JSON.stringify(result) + "\n---> " + JSON.stringify(result.returnValues));
        console.log("Event fired: " + JSON.stringify(result.returnValues));
        //process.exit(1);
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    //const value = m7.methods.multiply(8).call();
    const value = m7.methods.multiply(8)
        .send({from: accounts[0]})
        //.then(function(value) {console.log("---> MyFunction called " + JSON.stringify(value) +
        //                               '\n---> '+ JSON.stringify(value.events.Print.returnValues));});
    console.log("---> MyFunction called " + JSON.stringify(value));
        //+ '\n---> '+ JSON.stringify(value.events.Print.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}

doIt()

```

```python
[파일명: src/Multiply7EventUse.js]
var Web3=require('web3');
var _abiBinJson = require('./Multiply7Event.json');      //importing a javascript file

//var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/Multiply7Event.sol:Multiply7Event']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
var m7 = new web3.eth.Contract(_abiArray, '0x33add2effA3E32050aCD8446d826b0EFFB93A515');
var event = m7.events.Print({fromBlock: 0}, function (error, result) {
    if (!error) {
        //console.log("Event fired: " + JSON.stringify(result) + "\n---> " + JSON.stringify(result.returnValues));
        console.log("Event fired: " + JSON.stringify(result.returnValues));
        //process.exit(1);
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    //const value = m7.methods.multiply(8).call();
    const value = m7.methods.multiply(8)
        .send({from: accounts[0]})
        //.then(function(value) {console.log("---> MyFunction called " + JSON.stringify(value) +
        //                               '\n---> '+ JSON.stringify(value.events.Print.returnValues));});
    console.log("---> MyFunction called " + JSON.stringify(value));
        //+ '\n---> '+ JSON.stringify(value.events.Print.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    process.exit(1); //force exit to end waiting
}

doIt()
```

```sendTransaction()``` 함수로 트랜잭션을 발생시켜야 블록체인에 기록이되고, 그 로그에서 이벤트가 발생이 된다.


```python
pjt_dir> node src/Multiply7EventUse.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Event fired: {"0":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","1":"1651956296","2":"56","_addr":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","timestamp":"1651956296","res":"56"}
Balance before: 999976287563999994122
---> MyFunction called {"_events":{}}
Balance after: 999976287563999994122
Balance diff: 0
```

# 6. fallback 함수

## 어떤 함수를 호출했는데, 그 함수가 없는 경우 fallback을 실행

어떤 함수를 호출하였다고 하자. 그 함수가 **존재하지 않을 경우**에 fallback 함수가 대신 호출된다.
아래에서 ```callA()``` 함수를 호출해야 하는데 잘 못하여 ```callB()```를 호출했다고 하자.
그러면 존재하지 않는 ```callB```()를 실행하지 못하고 ```fallback()```을 실행한다.

또한 ether를 송금했는데 (value 필드에 값이 있는 경우), 수신측에서 이를 받는 함수가 존재하지 않으면, 송금이 거절되지만 ```fallback```이 호출된다. 이 경우는 **```fallback() payable```**로 선언해야 한다.

## 함수는 이름없이 extern으로 선언한다

fallback() 함수는 보통 함수와는 다른 기능을 가진다.
사용자가 직접 호출해서 실행되지 않는다. 따라서 **이름이 없다.**.
외부에서만 호출할 수 있도록 **external**으로 선언한다.
보통 인자가 없고, 반환 값도 없지만, 최신 버전에서는 정해진 입력과 반환 값을 가질 수 있다.
fallback함수가 없으면 **예외**가 발생하게 된다.

```
fallback () external [payable]
또는 최신 버전 0.7이상에서는 입력 ```bytes calldata```, 출력 ```bytes memeory```를 가질 수 있다.
fallback (bytes calldata _input) external [payable] returns (bytes memory _output)
```

잘 못된 선언의 예를 보자.
```
fallback() {} //error must be extern 
function fallback() external payable {} //warning. function 키워드 없이 fallback 사용해야 함.
function() external payable {} //오류. 이전 버전 0.6.x 이하에서는 이렇게 선언하였다.
fallback() external payable {} //정상. 송금있는 경우 사용.
fallback() external {} //정상. 송금없는 경우 사용.
fallback() external payable { emit PrintLog("fallback");} //정상
fallback(bytes calldata _input) external returns(bytes memory _output) {} //정상. 그러나 최신버전에서만.
```

컨트랙에는 **한 개의 fallback 함수**만 존재할 수 있다.
아래와 같이 하나 이상의 fallback() 함수를 가져서는 안된다.
최신 버전에서는 fallback 함수에 인자를 넣어서 구현을 해도 되지만, 이러한 오버로딩 역시 가능하지 않다.
```
//오류 코드. 하나 이상의 fallback.
contract FallbackTest1 {
    fallback() external {}
    fallback() external payable {}
```

## receive 함수가 있으면 fallback에 앞서 호출된다

Solidity 버전 0.6.x부터는 ```receive()```함수가 제공되고 있다.
이 함수는 송금을 하지만 (value 필드에 값이 있는 경우) calldata가 없는 경우 호출된다.
따라서 ```receive() external payable```로 사용해야 한다.
```receive()```함수가 없으면 ```fallback()```함수가 호출되게 된다.

calldata는, 앞서 ABI 명세에서 설명한 바와 같이, 함수시그니처와 매개변수를 의미하며 아래 data 필드에 적어준다.
```
web3.eth.sendTransaction({
    from: ...,
    data: "0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001",
    gas: ...}
)
```

몇 가지 오류와 정상 코드를 예로 들면 다음과 같다.
```
//receive() {} //error must be extern
//receive() external {} //error must be payable
receive() external payable {} //ok use without function keyword
//function receive() external payable {} //warn function keyword used
```

다음 코드를 보면, 사용자가 호출한 함수가 **존재하지 않을 경우**에 fallback이 호출된다.
```python
contract FallbackTest1 {
    fallback() external {}
}
```

아래 송금하는 경우를 보자. 송금을 하면 receive() 또는 fallback()의 payable 함수가 입금을 하게 된다.
* 송금을 sending하면, Receiving의 receive()가 실행된다.
* Receiving에 receive()가 없는 경우에는 fallback()이 실행된다.

```
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Receiving {
   event PrintLog(string s, address _from, uint amount);
   receive() external payable {
       emit PrintLog("now receiving...", msg.sender, msg.value);
   }
   fallback() external payable {
       emit PrintLog("now fallback...", msg.sender, msg.value);
   }
}

contract Sending {
    Receiving r=new Receiving();
    function deposit(uint amount) payable public {
        require(msg.value==amount);
    }
    function sending() public payable {
        address(r).call{value:11}("");
        //payable(address(r)).transfer(11); //contract address -> payable
    }
    function getBalanceOfThis() public view returns(uint) {
        return address(this).balance;
    }
    function getBalanceOfReceiving() public view returns(uint) {
        return address(r).balance;
    }    
}
```

## 실습문제: fallback

존재하지 않는 함수를 호출해서 강제적으로 fallback 함수를 실행해 본다.

### 단계 1: 컨트랙 개발

```python
[파일명: src/FallbackTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract FallbackTest {
    event PrintLog(string);
    function callA () pure public returns(string memory){
        return "doing callA";
    }
    fallback () external {
        emit PrintLog("fallback called");
    }
}
```

### 단계 2: 컴파일

```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/FallbackTest.sol > src/FallbackTest.json
```

### 단계 3: 컨트랙 배포

```python
[파일명: src/FallbackTestDeployFromFile.js]
var Web3 = require('web3');
var _abiBinJson = require('./FallbackTest.json');      //importing a javascript file
//var fs=require('fs');
//var _str = fs.readFileSync("src/FallbackTest.json");
//var _json=JSON.parse(_str)

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/FallbackTest.sol:FallbackTest']
console.log("- contract name: ", contractName);
_abiArray=_abiBinJson.contracts[contractName].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin=_abiBinJson.contracts[contractName].bin;

//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 1000000}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()

```

```python
pjt_dir> node src/FallbackTestDeployFromFile.js

- contract name:  [ 'src/FallbackTest.sol:FallbackTest' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0xcd89d730a606179c4c36b84bc2ce58719fce84ecac932af8e2fd3000453fb7da
---> The contract deployed to: 0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C
```

### 단계 4: 사용

REMIX에서는 fallback 함수를 호출할 수 있도록 노출하지 않는다.
REMIX 버전 0.9.3부터 지원된 **Low level interactions**라는 기능을 통해 fallback 함수를 호출할 수 있다.
**최소 1바이트 이상을 입력해야 하는데, 16진수로 calldata를 입력**하고 ```Transact``` 버튼을 누르면,
우측 아래 REMIX 콘솔 창에 함수가 호출되었다는 로그가 뜬다.
로그를 열어 보면 이벤트가 발생하였다는 것을 확인할 수 있다.

**아무 데이터도 입력하지 않고** ```Transact``` 버튼을 누르면, ```receive```가 호출된다.

![alt text](figures/10_fallbackREMIX.png "call fallback via low level interactions on REMIX")


```python
[파일명: src/FallbackTestUseFromFile.js]
var Web3=require('web3');
var _abiJson = require('./FallbackTestABI.json');
//var _binJson = require('./FallbackTestBIN.json'); // not needed
//var fs=require('fs');
//var _str = fs.readFileSync("src/FallbackTestABI.json");
//var _json=JSON.parse(_str)

var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
    //web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
}

contractName=Object.keys(_abiJson.contracts); // reading ['src//FallbackTest.sol:FallbackTest']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
//_bin=_binJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    var _instance = new web3.eth.Contract(_abiArray, "0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C");
    var event = _instance.events.PrintLog(function (error, result) {
        if (!error) {
            console.log("Event fired: " + JSON.stringify(result) + "\n---> " + JSON.stringify(result.returnValues));
        }
    });

    _instance.methods.callA().call().then(function(res) { console.log(res); });  //null
    //call without calling any method
    //await _instance.methods.callB().send({from:accounts[0], to: "0x3991e87b71cBFf94aA0718F341d8Ad4bCF969f36"}); //fail
    //await _instance.methods.callA().send({from:accounts[0], data:"0x1234"});  //empty calldata to call fallback -> fail
    web3.eth.sendTransaction({from:accounts[0], to: "0x3991e87b71cBFf94aA0718F341d8Ad4bCF969f36"}); //fallback called
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}

doIt()
```


```python
pjt_dir> type src\FallbackTest.json

{"contracts":{"src/FallbackTest.sol:FallbackTest":{"abi":[{"anonymous":false,"inputs":[{"indexed":false,"internalType":"string","name":"","type":"string"}],"name":"PrintLog","type":"event"},{"stateMutability":"nonpayable","type":"fallback"},{"inputs":[],"name":"callA","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}],"bin":"608060405234801561001057600080fd5b50610157806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063e7f09e0514610062575b7f968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd604051610058906100f8565b60405180910390a1005b61006a610080565b60405161007791906100a5565b60405180910390f35b60408051808201909152600b81526a646f696e672063616c6c4160a81b602082015290565b6000602080835283518082850152825b818110156100d1578581018301518582016040015282016100b5565b818111156100e25783604083870101525b50601f01601f1916929092016040019392505050565b6020808252600f908201526e19985b1b189858dac818d85b1b1959608a1b60408201526060019056fea26469706673582212200d9e1059d2a545eb520d53cf3762599743448a15c33b6509c126eb12fda0589564736f6c63430008010033"}},"version":"0.8.1+commit.df193b15.Windows.msvc"}
```

```python
[파일명: src/FallbackTestUseFromFile.js]
var Web3=require('web3');
var _abiBinJson = require('./FallbackTest.json');      //importing a javascript file

//var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/FallbackTest.sol:FallbackTest']
console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abiArray=_abiBinJson.contracts[contractName].abi; //use just as read from file
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);
//The above ABI from file does not work. So ABI was copied and pasted as below -> It worked!
_abiArray=[{"anonymous":false,"inputs":[{"indexed":false,"internalType":"string","name":"","type":"string"}],"name":"PrintLog","type":"event"},{"stateMutability":"nonpayable","type":"fallback"},{"inputs":[],"name":"callA","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}];
async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    var _instance = new web3.eth.Contract(_abiArray, "0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C");
    var event = _instance.events.PrintLog({fromBlock: 0}, function (error, result) {
        if (!error) {
            console.log("Event fired: " + JSON.stringify(result) + "\n---> " + JSON.stringify(result.returnValues));
        }
    });

    _instance.methods.callA().call().then(function(res) { console.log(res); });  //null
    //call without calling any method
    //await _instance.methods.callB().send({from:accounts[0], to: "0x3991e87b71cBFf94aA0718F341d8Ad4bCF969f36"}); //fail
    //await _instance.methods.callA().send({from:accounts[0], data:"0x1234"});  //empty calldata to call fallback -> fail
    web3.eth.sendTransaction({from:accounts[0], to:"0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C"}); //fallback called
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    process.exit(1); //force exit
}
doIt()
```

#### callA().send({from:accounts[0], data:"0x1234"})는 fallback을 호출하지 못함

REMIX에서는 **임의의 calldata를 입력**하고 (적어도 2자리 1바이트를 입력한다. 예를 들면, 0x11) 또는 아무 것도 입력하지 않은채로 ```Transact``` 버튼을 누르면,
```fallback``` 함수가 호출되었다.
아래 노드 프로그램에서는 ```data``` 필드를 임의로 수정해서 호출되도록 해보자.
```fallback```은 ```payable```함수가 아니라서, 금액이 지급되지 않으며 잘못된 함수호출에 반응하여 호출되어야 하고, 이전 버전의 ```web3.js```에서는 다음과 같은 출력이 발생하였다. 현재 버전에서는 아래에서 보듯이 가능하지 못하다.

```python
{ address: '0x3ab1841c257571342e04b0fc898d2e63050585b2',
  blockNumber: 45484,
  transactionHash: '0x5725425ea888cfcad16298c2e4a1d3fd71cbb2880230b82762ac5be12fee7def',
  transactionIndex: 0,
  blockHash: '0xe4e9f371bb90f67279e9284c17451cba53b5eb850ed416249fe32b64f801e20c',
  logIndex: 0,
  removed: false,
  event: 'PrintLog',
  args: { '': 'fallback called' } }   <--- fallback 호출
```

#### 존재하지 않는 function selector로 fabllback 호출

**존재하지 않는 함수를 호출**하면 fallback함수가 호출된다.
클라이언트에서 web3를 사용하면, 존재하지 않는 함수를 호출할 수 있는 방법은 없다.
가능한 방법은 ```web3.eth.sendTransaction()```의 ```data```, 즉 ```msg.data```를 수정하면 된다. 앞서 설명한 **function selector** (호출할 함수시그니처의 sha3 해시값으로 만들고, 앞 4바이트)가 존재하지 않도록 수정해야 한다.

```python
<address>.call(bytes4(bytes32(sha3("baz(uint32,bool)")))
```

아쉬운대로 다음과 같이 **함수명을 생략하여 강제적으로 존재하지 않는 함수**를 호출한다.
즉 컨트랙 주소로 그냥 트랜잭션을 전송하여 없는 함수가 호출된 것처럼 한다.

```python
web3.eth.sendTransaction({from:web3.eth.accounts[0], to:<<contract address>>});
```

위에서 Http를 사용하면 아래에서 보듯이 Event fired가 출력되지 않는다.

```
- contract name:  src/FallbackTest.sol:FallbackTest
Account: 0x8218Ee4e07eE1a6000ce4542f6DC9532611A18f1
Balance before: 99996604906000000000
doing callA   ----> 출력이 없는데...
Balance after: 99996167646000000000
Balance diff: 437260000002048
```

```python
pjt_dir> node src/FallbackTestUseFromFile.js

- contract name:  [ 'src/FallbackTest.sol:FallbackTest' ]
- ABI: [object Object],[object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999975007733999994122
Balance after: 999975007733999994122
Balance diff: 0
```


```python
pjt_dir> node src/FallbackTestUseFromFile.js

- contract name:  src/FallbackTest.sol:FallbackTest
Account: 0x8218Ee4e07eE1a6000ce4542f6DC9532611A18f1
Balance before: 99996604906000000000
doing callA
Balance after: 99996167646000000000
Balance diff: 437260000002048
```

websocket으로 변경하면 Event fired가 출력된다.
ABI를 파일로 읽지 않고, 수작업으로 복붙하였다.

```python
pjt_dir> node src/FallbackTestUseFromFile.js

- contract name:  [ 'src/FallbackTest.sol:FallbackTest' ]
- ABI: [object Object],[object Object],[object Object]
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999975007733999994122
Event fired: {"address":"0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C","blockHash":"0x465145638b2da03dc570a319703928e6f140d1791a4f91cca36c5aadeac41c78","blockNumber":120,"logIndex":0,"removed":false,"transactionHash":"0x15866c88cd1e6da85e36fa0d6435cb69f44e1e1c28a6e173c5f7e871991b5a64","transactionIndex":0,"id":"log_663c3d49","returnValues":{"0":"fallback called"},"event":"PrintLog","signature":"0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd","raw":{"data":"0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f66616c6c6261636b2063616c6c65640000000000000000000000000000000000","topics":["0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd"]}}
---> {"0":"fallback called"}
Event fired: {"address":"0x305F89e9b9C91B0b242874d77Ef675b0eBAD437C","blockHash":"0x31d9eb58b769e1004aeead29a35f6bc27ce30335cf99c8020e0cc730646171ab","blockNumber":121,"logIndex":0,"removed":false,"transactionHash":"0x5b7a92d8c605ce2153f89f858fa4df3d7509a0abf7e0370bf1ee10763cb184e7","transactionIndex":0,"id":"log_dc4be599","returnValues":{"0":"fallback called"},"event":"PrintLog","signature":"0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd","raw":{"data":"0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f66616c6c6261636b2063616c6c65640000000000000000000000000000000000","topics":["0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd"]}}
---> {"0":"fallback called"}
Balance after: 999975007733999994122
Balance diff: 0
```

```python
pjt_dir> node src/FallbackTestUseFromFile.js

- contract name:  src/FallbackTest.sol:FallbackTest
Account: 0x8218Ee4e07eE1a6000ce4542f6DC9532611A18f1
Balance before: 99995276566000000000
doing callA
Balance after: 99995276566000000000
Balance diff: 0
Event fired: {"logIndex":0,"transactionIndex":0,"transactionHash":"0x988203b9673bb10e9ecfed1d9e4a9aad1f6555607be560c40139625153a8ec0b","blockHash":"0x10bf044ab1a16f74a8fd22436b7cdf4cc6c1e82da6b23c8afb5c3a1a13f446f7","blockNumber":13,"address":"0x3991e87b71cBFf94aA0718F341d8Ad4bCF969f36","type":"mined","id":"log_6023f931","returnValues":{"0":"fallback called"},"event":"PrintLog","signature":"0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd","raw":{"data":"0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000f66616c6c6261636b2063616c6c65640000000000000000000000000000000000","topics":["0x968f0302429ff0e5bd56a45ce3ba1f4fa79f4b822857e438616435f00c3b59fd"]}}
    ---> {"0":"fallback called"}
```

## 실습문제: 컨트랙 결합에서 payable fallback 사용

앞서 존재하지 않는 함수를 호출하는 경우, ```fallback```함수가 호출되었다. 또한 금액없이 송금을 하여도 ```fallback```함수가 호출된다.

송금을 하면서 오류가 발생하여 ```fallback```이 호출된다 하더라도 송금액은 받을 수 없다. 단, 다음과 같이 ```payable```으로 선언하면 송금액을 받을 수 있다.

```python
fallback() payable {}
```

### 단계 1: 컨트랙 개발

```fallback()``` 함수가 호출되려면, web3.js 측에서가 아니라 다른 컨트랙에서 실험하는 편이 좋다.
그렇다면 부득이 컨트랙을 하나 더 만들어야 하고, 두 컨트랙이 연관을 가져야 한다.
아래 코드를 보면:
* 두 컨트랙이 하나의 파일에 존재하면 ```new``` 명령어로 객체를 만들어 사용하거나,
* 이미 배포된 컨트랙을 사용하려고 하면, ```setM7()``` 함수를 통해 그 주소를 할당하여 사용하고 있다.

REMIX에서 버튼을 누르며 테스트 해보자.
* 먼저 Math의 적색버튼 deposit을 누른다. 이 때 인자에 111을, 위 VALUE에도 같은 숫자 111을 적는다.
* 그리고 Math의 적색버튼 ```send11M7```을 누른다. 이때 REMIX의 우측 아래 단말를 보면, 녹색의 성공 체크표시가 뜬다 (실패하면 적색 X 표시). 우측의 청색 Debug를 펼치고, logs로 이동하고 args를 보면, 다음과 같은 로그 출력을 볼 수 있다.
```
"args": {
			"0": "now receiving in Multiply7",
			"1": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4",
			"s": "now receiving in Multiply7",
			"_from": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4"
		}
```
위 출력을 보면, receive() 함수가 호출되고 있음을 알 수 있다. 소스코드의 도움말을 활성화해가면서 (쓰지 않는 부분은 비활성화), 예를 들어 receive()를 비활성화하면 fallback()을 호출하게 된다.

* fallback이나 receive를 호출되는 이유를 잠깐 살펴보자.
```payable(address(m7)).transfer(11)```는 m7에 송금을 하는 것인데, 함수를 통하지 않고 있다. 즉 없는 함수를 호출하면 어떻게 처리를 할 방법이 없다. 이 때 receive가 호출된다. receive 함수가 없으면 fallback이 호출된다.

```python
[파일명: src/MathMultiply7.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Multiply7 {
    //address owner;
   event PrintLog(string s, address _from, uint _amount);
   event PrintLog(string s, address _from);
   event PrintLog(uint);
   //constructor() { owner=msg.owner; }
   receive() external payable {
       //emit PrintLog("now receiving in Multiply7", tx.origin, msg.value); //error
       //emit PrintLog("now receiving in Multiply7", tx.origin, 11);
       emit PrintLog("now receiving in Multiply7", tx.origin);
   }
   fallback() external payable {
       //emit PrintLog("now fallback in Multiply7", tx.origin, msg.value); //error
       //emit PrintLog("now fallback in Multiply7", tx.origin, 111);
       emit PrintLog("now fallback in Multiply7", tx.origin);
   }
   function multiply(uint input) pure public returns (uint) {
      //emit Print(input * 7);
      return input * 7;
   }
   function getAddress() view public returns(address) {
       return address(this);
   }
}

contract Math {
    Multiply7 m7=new Multiply7();
    function deposit(uint amount) payable public {
        require(msg.value==amount);
    }
    function setM7(address payable _addr) public { m7 = Multiply7(_addr); }
    function multiply() view public returns(uint) {
        uint res=m7.multiply(8);
        //this.send11(); It does not send value.
        return res;
    }
    function send11M7() public payable {
        //m7.multiply.value(11)(9);
        //m7.multiply(9);
        payable(address(m7)).transfer(11); //contract address -> payable
    }
    function getBalanceOfThis() public view returns(uint) {
        return address(this).balance;
    }
    function getBalanceOfM7() public view returns(uint) {
        return address(m7).balance;
    }
    function getAddressOfM7() view public returns(address) {
        return address(m7);
   }
}
```

### 단계 2: 컴파일

combined-json 명령어를 abi, bytecode 각 각에 적용해서 해보자.

```python
pjt_dir> solc src/MathMultiply7.sol --combined-json abi > src/MathMultiply7ABI.json
```

파일에 2개의 컨트랙이 포함되어, 아래에서 보듯이 abi는 2개가 생성되고 있다.

```python
pjt_dir> cat src/MathMultiply7ABI.json

{"contracts":{
	"src/MathMultiply7.sol:Math":{"abi":"[...생략...]"},    (1) Math 컨트랙
	"src/MathMultiply7.sol:Multiply7":{"abi"[...생략...]"}  (2) Multiply7 컨트랙
	},"version":"0.6.1+commit.e6f7d5a4.Linux.g++"}
```

이번에는 bytecode를 생성한다.
```python
pjt_dir> solc src/MathMultiply7.sol --combined-json bin > src/MathMultiply7BIN.json
```

bytecode도 역시 2개가 생성되고 있다.

```python
pjt_dir> cat src/MathMultiply7BIN.json

{"contracts":{
	"src/MathMultiply7.sol:Math":{"bin":"608060...생략...10033"},
	"src/MathMultiply7.sol:Multiply7":{"bin":"608060...생략...10033"}
	},"version":"0.6.1+commit.e6f7d5a4.Linux.g++"}
```

### 단계 3: 컨트랙 배포

기억하자. 2개의 컨트랙이 있고 이 가운데 외부에서 사용하게 될 Math 하나만 배포하고 있다. abi, bin 모두 Math만 포함한다.

```python
[파일명: src/MathMultiply7DeployFromFile.js]
var Web3=require('web3');
var _abiJson = require('./MathMultiply7ABI.json');
var _binJson = require('./MathMultiply7BIN.json');

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractNames=Object.keys(_abiJson.contracts); //Math, Multiply7
contractName=contractNames[0]; // -> 'src/MathMultiply7.sol:Math', contractNames[1] -> Multiply7
console.log("- contract name: ", contractName);
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_abiArray=JSON.parse(JSON.stringify(_abiJson.contracts[contractName].abi));    //Unexpected token error
_bin="0x"+_binJson.contracts[contractName].bin;
//_bin=_binJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

```python
pjt_dir> node src/MathMultiply7DeployFromFile.js

- contract name:  src/MathMultiply7.sol:Math
Deploying the contract from 0x83e790081F4117bEfe83d35bcDEC772e066F675D
hash: 0xed1c5059d1f02cf839fc125810d326a34122369da38f7f298c6def3b0cee07db
---> The contract deployed to: 0xcD3F2A7eca9E6cF6dA466E0e011858E05B033C65
```

### 단계 4: 사용

지금은 버전 0.8으로 컴파일해서 실행하고 있다. 혹시 geth@8445에서 실행하면서, 문제가 있으면 버전을 낮추어 적용하여 보자 (이전에 문제가 있어 0.4.21로 낮추어 한 경험이 있다.)

#### 컨트랙 연관관계에서의 주소 설정

```new``` 명령어는 동일한 파일에 연관관계 컨트랙이 존재할 때 주소를 할당하여 객체를 생성한다.
```setM7(address)``` 함수는 ```Multiply7```를 배포하고, 확득한 주소를 할당한다.

#### 입금해서 잔고가 있어야 송금 가능: send11M7()

9를 곱하면서 ```11 wei```를 송금하려면 ```m7.multiply.value(11)(9)```로 호출할 수 있다.
송금하면서 ```payable fabllback```를 호출되게 하기 위해 의도적으로 아래와 같이 존재하지 않는 함수를 호출하였다.

```python
call.value()();  ?????
```

송금하려면 물론 ```deposit()```을 먼저해야 한다.

1. ```deposit(123)```
value를 반드시 함수인자와 동일하게 입력해야 한다.
REMIX에서 할 경우에도 마찬가지이다. ```Run``` 탭 상단의 ```value```, ```Deployed Contracts``` 함수의 인자 두 필드에 동일한 금액을 넣어준다.
2. ```getBalanceOfThis()``` --> 123이 된다. 입금하고 난 후의 잔고이다.
3. ```getBalanceOfM7()``` ---> 0 아직 Multiply7의 잔고는 비어있다.
4. ```send11M7()```
@존재하지 않는 함수를 호출하여 ```m7.call.value(11)()```,
다른 컨트랙 ```Multiply7```의 ```payable fallback``` 함수를 호출하여 ```11 Wei```를 송금한다.
5. ```getBalanceOfThis()``` ---> 112
6. ```getBalanceOfM7()``` ---> 11 ```payable fallback``` **존재하지 않는 함수를 호출하는 오류에도 불구하고 11 wei 송금 성공**

```python
[파일명: src/MathMultiply7UseFromFile.js]
var Web3=require('web3');
var _abiJson = require('./MathMultiply7ABI.json');
//var _binJson = require('./MathMultiply7BIN.json'); // not needed

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));

contractNames=Object.keys(_abiJson.contracts); //Math, Multiply7
contractName=contractNames[0]; // -> 'src/MathMultiply7.sol:Math', contractNames[1] -> Multiply7
console.log("- contract name: ", contractName); //or console.log(contractName);
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_abiArray=JSON.parse(JSON.stringify(_abiJson.contracts[contractName].abi));    //Unexpected token error
//_bin=_binJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    var _instance = new web3.eth.Contract(_abiArray, "0xcD3F2A7eca9E6cF6dA466E0e011858E05B033C65");

    _instance.methods.multiply().call().then(console.log);    //56 출력 (8 곱하기 7 연산)
    _instance.methods.deposit(123).send({from:accounts[0], value:123}); //value, 인자에 동일한 123으로 적음
    _instance.methods.getBalanceOfM7().call().then(console.log); //0 출력
    await _instance.methods.send11M7().send({from:accounts[0]}); //11을 송금
    _instance.methods.getBalanceOfM7().call().then(console.log); //잔고 11을 출력
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}

doIt()
```

```python
pjt_dir> node src/MathMultiply7UseFromFile.js

- contract name:  src/MathMultiply7.sol:Math
Account: 0x83e790081F4117bEfe83d35bcDEC772e066F675D
Balance before: 999997545196000000000
56
0
Balance after: 999997431831999999877
Balance diff: 113364000047104
11
```

한 번 더 실행하고 결과를 보면, 잔고가 증가하고 있다는 것을 알 수 있다.
위에서 11이 송금되었고, 2번째 출력에서 이를 반영해 11을 출력한다.

```python
pjt_dir> node src/MathMultiply7UseFromFile.js

- contract name:  src/MathMultiply7.sol:Math
Account: 0x83e790081F4117bEfe83d35bcDEC772e066F675D
Balance before: 99978782245999999877
56
11
22
Balance after: 99977762925999999754
Balance diff: 1019320000004096
```

## 연습문제 
1. 생성자 문법이 맞는지 또는 틀린지 답하세요.
    (1) constructor() Hello public {} //error
    (2) constructor() internal {} //error
    (3) constructor() private {} //error
    (4) constructor() public {} //ok but warning
    (5) constructor() {} //ok
2. 객체지향에서와 같이 생성자 오버로딩은 가능하다. OX로 답하세요.

3. pure함수는 상태변수를 읽을 수 있다. OX로 답하세요.
4. view함수는 이벤트를 발생할 수 있다. OX로 답하세요.

5. web3 호출하는 측에서 { value: 0 }에 ether를 입력하면 무엇을 통해 전달되는지 답하시오.
전역변수 ```msg.value```

6. public 가시성은 내, 외부에서 모두 사용할 수 있다. OX로 답하세요.
7. extern 가시성은 내부에서 호출할 수 없다. OX로 답하세요.
8. 함수의 제약조건으로 사용할 수 있는 것은?
(1) modifier
(2) underscore
(3) event
(4) returns

9. 블록체인에서의 이벤트는 어디에 기록이 되는가? 여기를 계속 관찰하면서, 이벤트가 발생하는지 포착한다.

10. 이벤트는 오버로딩이 가능하다. OX로 답하세요.

11. 이벤트는 http를 통해서 인식할 수 있다. OX로 답하세요.

12. fallback 문법이 맞는지 또는 틀린지 답하세요.
(1) fallback() {} //error must be extern 
(2) function() external payable {} //오류. 이전 버전 0.6.x 이하에서는 이렇게 선언하였다.
(3) fallback() external payable {} //정상. 송금있는 경우 사용.
(4) fallback() external {} //정상. 송금없는 경우 사용.

13 컨트랙에는 한 개의 fallback 함수만 존재할 수 있다. OX로 답하세요.

14. fallback() 함수가 없으면 receive() 함수가 먼저 호출된다. OX로 답하세요.


15. 블록체인에서 주문을 하고, 그 주문내역을 로컬 파일에 작성하세요.
블록체인으로 주문이 전송되면 이벤트가 발생하도록 프로그램하고,
주문내역은 로컬파일에 저장됩니다.
(강의자료 참조: ```EventTest.sol```에서 이벤트가 발생하고 로그를 파일OrderEvent.txt에 작성)
* 강의자료 ```OrderEvent.sol```을 참조해서 구현하세요.
	* ```order(상품항목, 주문개수, 단가, 배송지)``` 함수를 작성. '개수 x 단가'로 주문금액을 계산하고 이 금액만큼 입금되지 않으면 예외발생.
	* ```getBalance()``` 함수를 작성. 최초 프로그램 생성자 ```owner```만 읽을 수 있게 ```modfier isOwner``` 작성.
(1) REMIX에서 주문하고, 이벤트 발생하는지 보이도록 콘솔 출력 붙여넣으세요.
(2) 파일로 컴파일 하세요.
(3) ```ganache```에서 배포하세요.
(4) 주문을 3건만 하고, 이벤트를 통해 다음과 같이 출력하세요.
항목은 차례대로 주소, 상품항목번호, 주문개수, 단가, 배송지.
```python
전송자주소, 1111, 3, 100, 20 2-gil Hongji-dong Jongro-gu Seoul
전송자주소, 1111, 5, 100, 20 2-gil Hongji-dong Jongro-gu Seoul
전송자주소, 1111, 20, 100, 20 2-gil Hongji-dong Jongro-gu Seoul
```
(5) 파일에 저장된 파일내역을 출력하세요 (주피터 노트북에서 ```!type src/OrderEvent.txt```)

(5) ganache 대신 geth로 해보세요.
(geth는 geth@8445 또는 geth@8446에서 자바스크립트로 한다는 의미)
복수 거래가 발생하므로, 한 건씩 발생하도록 'OrderEventUse.js' 파일을 나누어 코딩해도 된다.

16.  앞서 FallbackTest에 receive()함수를 추가하고, 호출되도록 해보자.

아래와 같이 시도해보자.
```python
web3.eth.sendTransaction({from:web3.eth.accounts[0], to:<<contract address>>});
```

```python
[파일명: src/FallbackTest.sol]
pragma solidity ^0.6.0;
contract FallbackTest {
    event PrintLog(string);
    function callA () pure public returns(string memory){
        return "doing callA";
    }
    fallback () external {
        emit PrintLog("fallback called");
    }
}
```

