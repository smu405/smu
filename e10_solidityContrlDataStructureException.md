#  Chapter 10. Solidity 제어, 데이터구조, 예외
 
Solidity의 반복문, 조건문에 대해서 배운다. 단 반복하면서 그 횟수와 멤버변수를 갱신하는 것에 주의한다. 데이터를 저장하기 위해서는 배열 또는 매핑을 사용하다. 예외에 대해서 배우게 된다. 블록체인 특성에 따라, 키보드, 마우스 등에서 입력을 받거나 또는 하드디스크에 쓰거나 하는 등의 입출력 예외는 발생할 수 없다. 반면 입출금 관련 예외를 주의해야 한다.

# 1. 제어

## 1.1 반복문

### for 문

반복문은 다른 언어와 비교해서 그 사용법이 대동소이하다. for 문은 증감연산자를 0부터 시작하여 하나씩 증가하며 반복적으로 수행한다. 반복을 언제 시작하고, 언제 끝나는지 명시하는 것이 필요하므로, 괄호 안에 ```for(증감연산자 초기화; 종료조건; 증감 연산)```과 같이 작성한다. 그리고 반복할 명령문을 다음 줄에 작성하게 된다.

for 문을 적용하여 합계를 계산해보자.

```python
uint sum=0;
for(uint i=0;i<10;i++)
    sum+=i;
```

### while 문
while 문은 조건이 충족할 때까지 계속 반복한다. for문과 달리 증감연산자를 반드시 사용할 필요는 없다. 단 while(조건) 문이 먼저 나오기 때문에, 조건에 따라 1회도 실행이 되지 않을 수 있다.

```python
uint sum=0;
uint i=0;
while(i<10) {
    sum+=i;
    i++;
}
```

### do-while 문

while문과 의미는 비슷하지만, do { } 로 시작하기 때문에, 최소한 1회는 명령문이 실행된다.
```python
uint sum=0;
uint i=0;
do {
    sum+=i;
    i++;
} while(i<10);
```

## 1.2 반복의 비용

블록체인에서의 프로그램은 명령문마다 비용이 발생하는 특성을 가지고 있기 때문에, 반복을 하게되면 비용이 발생하게 된다.

반복이 늘어날수록 gas가 대칭적으로 증가하고, 한도 gasLimit를 초과하게 되면 실행이 중지될 수 밖에 없다. 10에서 100, 1000으로 반복횟수가 증가할 수록, 비용이 점진적으로 증가하고 있다.

특히 sum 변수를 메모리에 두는 경우에 비해, storage에 (state variable) 두는 경우 비용이 매우 높았다. 메모리에 비해 Storage를 사용하는 비용이 많이 든다. 10회일 경우 약 2배, 그러나 1000회는 15배 이상으로 급증하고 있다.

게다가 10,000회를 실행하게 되면, REMIX가 메모리 부족으로 재부팅하는 현상이 발생하고 있다.

특히 
반복 횟수 | sum을 로컬에 두는 경우 | sum을 state variable에 두는 경우
-----|-----|-----
10 | 약 30,000 (23845 + 6773)  |  약 64,000 (43045 + 21773)
100 | 약 78,000 (49345 + 28073)  |  약 368,000 (192881 + 175809)
1000 | 약 203,000 (112345 + 91073) | 약 3,408,000 (1712981 + 1695909)
10000 | REMIX가 죽고, 다시 실행 | 좌동

## 실습: 반복문, 조건문 사용

* 아래 forLoopCostly() 함수는 멤버변수 sum을 반복해서 수정하고 있다. 그렇지 않은 forLoop()에서 발생하는 gas비용과 서로 비교해보자.
* 아래 함수 whileLoop은 반복문을 실행하지만, whileLoopBreak는 break문으로 종료하고 있다.

```python
[파일명: src/LoopTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract LoopTest {
    uint sum;
    constructor() { //constructor() public {
        init();
    }
    function forLoop() public {
        //init();
        uint sumLocal = 0;
        for(uint i = 0; i < 101; i++) {
            sumLocal += i;
        }
        sum = sumLocal;
    }
    function forLoopCostly() public {
        init();
        //uint sumLocal = 0;
        for(uint i = 0; i < 101; i++) {
            //sumLocal += i;
            sum += i;
        }
        //sum = sumLocal;
    }
    function whileLoop() public {
        uint i = 0;
        uint sumLocal = 0;
        while(i <= 100) {
            sumLocal += i;
            i++;
        }
        sum = sumLocal;
    }
    function whileLoopBreak() public {
        uint sumLocal = 0;
        uint i = 0;
        while(true) {
            sumLocal += i;
            if(i==100) break;
            i++;
        }
        sum = sumLocal;
    }   
    function doWhileLoop() public {
        uint i = 0;
        uint sumLocal = 0;
        do {
            sumLocal += i;
            i++;
        } while(i <= 100);
        sum = sumLocal;
    }
    function init() public { sum = 0; }
    function getSum() view public returns(uint) { return sum; }
}
```

컴파일해서 오류가 발생하는지 확인하자.

```python
pjt_dir> solc-windows.exe src/LoopTest.sol
```


## 1.3 조건문

조건문 역시 다른 언어와 크게 다르지 않다. if, else if, else 문으로 조건을 평가할 수 있다.

```python
if(condition) {
    ...
} else if {
    ...
}
```

삼항 연산자 ternary operator는 간단한 if문을 대신해서 사용한다. 물음표 (?) 앞에 조건, 콜론 (:)을 기준으로 '참일 경우 값':'거짓일 경우' 값을 적는다.
```python
betAmount=betAmount>amount ? amount : betAmount;
```

```python
[파일명: src/ConditionalTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract ConditionalTest {
    function toLowerCase(string memory str) pure public returns (string memory) {
        bytes memory bIn = bytes(str);
        bytes memory bOut = new bytes(bIn.length);
        uint8 _iIn = 0;
        for (uint i = 0; i < bIn.length; i++) {
            _iIn = uint8(bIn[i]);
            if (_iIn >= 65 && _iIn <= 90)    // ascii A:65 ~ Z:90                
                bOut[i] = bytes1(_iIn + 32); // ascii a:97 ~ z:122
            else 
                bOut[i] = bIn[i];
        }
        return string(bOut);
    }
    function abs(int i) pure public returns (int) {
        return (i > 0) ? i : -i;
    }
}
```

실행해보자.
```python
pjt_dir> solc-windows.exe src/ConditionalTest.sol
```

## 실습: 인사하는 컨트랙

"Hello"를 출력하는 프로그램을 수정해서 if문을 사용해 보자. modifier를 사용하면 조건문을 사용하지 않을 수 있다.
조건문에서 비교문을 사용하게 되는데, ```string```은 참조타입이라 비교를 하면 주소를 비교하게 된다. 어떻게 해야 하는지 생각해보자.

```python
contract Hello1 {
    function sayHello() 
    modifier isOwner()
    function setHello()
    function compareTo(string memory _str)
    function getBalance()
}
```

###  단계 1: 컨트랙 개발

Line | 설명
-----|-----
1 | major version 6 이상 최신으로 컴파일
2 | contract 명. 파일명과 반드시 일치할 필요가 없다.
3 | 속성 ```hello```. 누구나 사용할 수 없도록 ```private```
4 | 속성 ```address```. ```private```으로 자신만 사용할 수 있게 선언.
5 | ```event```로서 주소, 문자열 인자를 출력한다. Solidity에는 print문이 별도로 없다. 문자열을 반환하거나 ```event```문으로 출력할 수 있다.
6 ~ 9 | 생성자로서 외부에서 인자를 받아 속성을 초기화. ```constructor```는 ```public``` 또는 ```internal```로 선언해야 한다. 속성 값을 넣어준다. ```msg```는 전역변수로서 ```msg.sender```는 전송자의 계정주소를 말한다. 생성자의 ```msg.sender```를 컨트랙의 소유주 ```owner```로 등록한다.
10 ~ 12 | ```sayHello()``` 함수는 속성 값을 읽는다. 따라서 ```view```로 선언한다. 인자는 함수 내에서만 잠시 사용하므로 ```memory```로 선언
13 ~ 18 | ```modifier``` 함수. ```msg.sender```와 ```owner```가 일치하지 않으면 예외처리한다. 아니면 계속 실행.
19 ~ 29 | ```setHello()``` 함수는 ```event```함수를 가지고 있으므로 ```view```로 선언하지 않는다. ```view```와 ```event```는 같이 사용할 수 없다. ```event```는 상태를 변경하기 때문이다.
30 ~ 37 | ```compareTo()```함수는 입력문자열과 hello를 비교한다. 문자열은 참조값이기 때문에 비교할 경우 참조를 비교하게 되므로 주의를 해야한다. hash로 비교하고 있다.
38 ~ 40 | 잔고를 구한다.


```python
[파일명: src/Hello1.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Hello1 {
    string private hello;
    address private owner;
    event PrintLog(address addr, string s);

    constructor(string memory _hello) { //constructor(string memory _hello) public {
        hello = _hello;
        owner = msg.sender;
    }
    function sayHello() view public returns(string memory) {
        return hello;
    }
    modifier isOwner() {
        if (msg.sender != owner) {
            revert();
        }
        _; //insert here the calling function
    }
    function setHello() public {
        string memory s = "";
        if (msg.sender == owner) {
            s = "Hello";
        } else {
            s = "Olleh";
        }
        emit PrintLog(msg.sender, s);
        hello = s;
    }
    function compareTo(string memory _str) view public returns(bool) {
        bool isEqual = false;
        //if (hello == _str) {
        if (keccak256(bytes(hello)) == keccak256(bytes(_str))) {
            isEqual = true;
        }
        return isEqual;
    }
    function getBalance() view public isOwner returns(uint) {
        return address(this).balance;
    }
}

```

### 단계 2: 컴파일

이번에는 abi, bin을 따로 만들어본다.

```python
pjt_dir> solc-windows.exe src/Hello1.sol --combined-json abi > src/Hello1ABI.json
```

```python
pjt_dir> solc-windows.exe src/Hello1.sol --combined-json bin > src/Hello1BIN.json
```

### 단계 3: 배포

bytecode는 16진수이므로 앞에 ```0x```를 붙여도 또는 붙여주지 않아도 된다.
gas를 지급해야 하는 등 비용이 발생하는 경우에는 반드시 web3.personal.unlockAccount()를 해주어야 한다.
REMIX를 사용하면, Web3 deploy를 복사해서 사용할 수 있다. abi, bin을 넣은 javascript명령문을 발견할 수 있다.

```python
[파일명: src/Hello1DeployFromFile.js]
var Web3=require('web3');
var _abiJson = require('./Hello1ABI.json');
var _binJson = require('./Hello1BIN.json');
//var fs=require('fs');
//var _str = fs.readFileSync("src/Hello1ABI.json");
//var _json=JSON.parse(_str)

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiJson.contracts); // reading ['src/Hello1.sol:Hello1']
//console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=_abiJson.contracts[contractName].abi;    //JSON parsing needed!!
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_bin="0x"+_binJson.contracts[contractName].bin;  // ok with "0x"
//_bin=_binJson.contracts[contractName].bin;  // ok without"0x"
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin, arguments: ["Hello from web3"]})
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
pjt_dir> node src/Hello1DeployFromFile.js

    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0xb677bc893937465d2487e7ea36b1077eb6053b21263581f1cf68105a61b8e567
    ---> The contract deployed to: 0xea7a5B548aBA1e6fE7532E4D1a7Ad9C32b8fE105
```

개인망을 띄우고, node로 Hello1DeployFromFile.js를 실행하면 화면좌측에 보이듯이 transaction이 생성되고 그 hash값 이 주어진다.

우측 geth 단말에서 보듯이 대기중 transaction hash 값이 서로 일치한다. 마이닝을 하고 나면 contract 주소가 주어진다.

```
INFO [01-08|11:37:07.548] Submitted contract creation
fullhash=0x2ef00671e310f594dca636acbc352ce61463e78110e8098b0d483c8374643248
contract=0x96d8b65B4586B090840d6587C216D4A69071F405
```

![alt text](figures/sol_helloV1Web3DeployMining.png "deploy hello.sol using web3 with mining")

### REMIX에서 이벤트 확인하기

REMIX에서 다음을 하나씩 해보자.
* Environment를 Javascript VM으로
* Deploy버튼 옆에 생성자 함수를 호출하기 위해 문자열 "Hello from web3"을 넣는다. 문자열에 따옴표를 잊지 않는다.
* Deploy버튼을 누르면 "Deployed Contracts" 아래, Hello1 객체가 나타난다. 괄호안 메모리는 실제 블록체인에 배포되지 않았다는 의미 (Javascript VM이므로)
* 속성 및 함수 버튼들을 눌러 결과가 출력되는지 보자.
* setHello() 함수 버튼을 누르면 화면 우측 하단을 잘 살펴보자. 특히 굵은 글씨 "logs"를 누르면 하단에 표가 펼쳐지고 이벤트 로그를 볼 수 있다.

![alt text](figures/sol_eventREMIX.png "geth download page")

### 단계 4: 사용

이벤트는 transaction log에 저장이 된다.
이벤트는 거래가 마이닝되어 블럭에 포함되어야만 발생된다.
트랙잭션이 pending되어 완료가 되지 않으면, 로그에 포함되지 않기 때문에 여기서 이벤틑가 발생하는지 지켜보는 경우 대기하게 된다.
단 이벤트가 발생하는 순서는 지켜진다.
단순하게 블록체인의 transaction이 아닌 ```call()```은 이벤트를 발생하지 않게 된다.

```python
[파일명: src/Hello1UseFromFile.js]
var Web3=require('web3');
var _abiJson = require('./Hello1ABI.json');
//var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://localhost:8345"));  //ok
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));  //ok
//var fs=require('fs');
//var _str = fs.readFileSync("src/Hello1.json");
//var _json = JSON.parse(_str)

contractName=Object.keys(_abiJson.contracts); // reading ['src/Hello1.sol:Hello1']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!

async function doIt() {
    var hello = new web3.eth.Contract(_abiArray, "0x5A9D02844aAdfb407A1AD0E0d6fA14627672B026");
    var event = hello.events.PrintLog(function (error, event) {
        console.log(">>> Event fired: " + JSON.stringify(event.returnValues));
    })
    .on('>> data', function(event) {
        console.log(event);
    })
    .on('>> changed', function(event) {
        console.log(event);
    })
    .on('>> error', console.error);
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    hello.methods.sayHello().call().then(console.log);  //null
    await hello.methods.setHello().send({from: accounts[0]});
    hello.methods.sayHello().call().then(console.log);
    hello.methods.compareTo("Hello").call().then(console.log);
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    
}
doIt()

```

웹소켓을 사용하면, 프로세스가 계속되고 있다. 프로세스를 인위적으로 해제해주어야 한다.

```python
pjt_dir> node src/Hello1UseFromFile.js


    - contract name:  src/Hello1.sol:Hello1
    Account: 0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1
    Balance before: 99964578880000000000
    Hello from web3
    >>> Event fired: {"0":"0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1","1":"Hello","addr":"0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1","s":"Hello"}
    Hello
    true
    Balance after: 99963981860000000000
    Balance diff: 597019999993856
```

## 실습: 블록체인에서 컨트랙 삭제

컨트랙의 소스코드는 만들어지면 영구히 남게된다.
보통 프로그램은 요구사항이 변경되면 수정하고 다시 배포하는 것이 가능하지만
블록체인의 프로그램은 그렇게 유지보수 할 수 없다.
오류가 발생하거나, 쓸모가 없어지면 삭제를 해야 한다.

이 때 쓰는 명령어가 selfdestruct()이고,
코드와 관련 메모리가 블록체인으로부터 제거된다.
당연히 이 명령어를 사용할 때는 매우 신중해야 한다.
보관되어 있는 잔고 역시 전액 정리되어야 한다.

잔고를 전액 반환하기 때문에, selfdestruct()는 지급가능한 주소가 필요하다.
Solidity 0.6.x 부터는 payable이라고 명시해야 한다.
```
selfdestruct(payable(owner));
```

###  단계 1: 컨트랙 개발


```python
[파일명: src/Hello2.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Hello2 {
    string hello;
    address owner; //address payable owner;

    event PrintLog(string _s);
    constructor() { //constructor() public {
        owner = msg.sender;
    }
    function sayHello() view public returns(string memory) {
        return hello;
    }
    function setHello(string memory _hello) public payable {
        hello = _hello;
        emit PrintLog(_hello);
    }
    function getBalance() view public returns(uint) {
        return address(this).balance;
    }
    function kill() public {
        if (msg.sender == owner) selfdestruct(payable(owner));
    }
}
```


### 단계 2: 컴파일


```python
pjt_dir> solc-windows.exe src/Hello2.sol --combined-json abi > src/Hello2ABI.json
```


```python
pjt_dir> type src\Hello2ABI.json


    {"contracts":{"src/Hello2.sol:Hello2":{"abi":[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"internalType":"string","name":"_s","type":"string"}],"name":"PrintLog","type":"event"},{"inputs":[],"name":"getBalance","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"kill","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"sayHello","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"string","name":"_hello","type":"string"}],"name":"setHello","outputs":[],"stateMutability":"payable","type":"function"}]}},"version":"0.8.1+commit.df193b15.Windows.msvc"}
```


```python
pjt_dir> solc-windows.exe src/Hello2.sol --combined-json bin > src/Hello2BIN.json
```

### 단계 3: 배포

컴파일한 후 abi, bin을 저장한 파일에서 읽어서 배포해보자.


```python
[파일명: src/Hello2DeployFromFile.js]
var Web3=require('web3');
var _abiJson = require('./Hello2ABI.json');
var _binJson = require('./Hello2BIN.json');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiJson.contracts); // reading ['src/Hello2.sol:Hello2']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=_abiJson.contracts[contractName].abi;
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_bin="0x"+_binJson.contracts[contractName].bin; //ok without "0x"

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
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
pjt_dir> node src/Hello2DeployFromFile.js

    - contract name:  src/Hello2.sol:Hello2
    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0x5b375a9b43ce502536c1d9ab67201d635401b339cbb636147b785efc2a4f4c48
    ---> The contract deployed to: 0x0de24099235b8CB671f3Ed2771F69B2049F679AF
```

### 단계 4: 사용


```python
[파일명: src/Hello2UseFromFile.js]
var Web3=require('web3');
var _abiJson = require('./Hello2ABI.json');
//var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://localhost:8345"));  //ok
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));  //ok

contractName=Object.keys(_abiJson.contracts); // reading ['src/Hello2.sol:Hello2']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=_abiJson.contracts[contractName].abi;
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!

async function doIt() {
    var hello = new web3.eth.Contract(_abiArray, "0x0de24099235b8CB671f3Ed2771F69B2049F679AF");
    var event = hello.events.PrintLog(function (error, result) {
        if (!error) {
            console.log("Event fired: " + JSON.stringify(result.returnValues));
        }
    });
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    hello.methods.sayHello().call().then(console.log);  //null
    await hello.methods.setHello("Hello World!").send({from: accounts[0], value: 1111});
    hello.methods.sayHello().call().then(console.log);
    //hello.methods.getBalance().call(function(err, bal) { console.log("Contract Balance: "+bal) });
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    process.exit(1);  //force exit
    //hello.methods.kill().send({from: accounts[0]})
}

doIt()

```

kill 함수를 마지막에 호출하기 때문에 한 번 실행해서 결과를 출력한 후, 재호출하면 ```Error: Returned values aren't valid, did it run Out of Gas?```가 호출된다.

```python
pjt_dir> node src/Hello2UseFromFile.js

    - contract name:  src/Hello2.sol:Hello2
    Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    Balance before: 999969369785999991900
    Hello World!
    Event fired: {"0":"Hello World!","_s":"Hello World!"}
    Balance after: 999969310753999990789
    Balance diff: 59032000004096
```

## 1.4 무작위 수의 생성

블록체인에서 무작위 수를 생성하기 위해서는 전역변수를 활용해야 한다.
전역변수를 256비트의 해시값으로 만들고, modulus를 이용해 일정 범위의 수가 생성되도록 한다.

일련의 무작위 수를 생성하기 위해서는 전역변수 값이 짧은 시간에 변동하지 않으므로,
매 번 다른 수가 생성되도록 하려면, 약간의 트릭을 써서 카운터 값을 더해주도록 한다.

abi.encodedPacked는 encoding 방식을 말한다. 예를 들면 앞서 배웠던 ABI 인터페이스에서 zero padding을 했으나 abi.encodedPacked 방식은 이를 하지 않는다.

```python
[파일명: src/Random.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Random {
    function rand() public view returns(bytes32) {
        return keccak256(abi.encodePacked(block.timestamp, block.difficulty));
    }
    function rand0and250() public view returns(uint8) {
        return uint8(uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty)))%251);
    }
    function rand0and9() public view returns(uint8) {
        return uint8(uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty)))%10);
    }
    function rand0and2() public view returns(uint8) {
        return uint8(uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty)))%3);
    }
    function genRandomInteger() view public returns(uint8[] memory) {
        uint8[] memory r = new uint8[](10);
        for(uint i = 0; i<r.length; i++)
            //r[i] = uint8(uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty)))%10);
            //change timestamp by adding i, otherwise all the same numbers generated
            r[i] = uint8(uint256(keccak256(abi.encodePacked(block.timestamp + i, block.difficulty)))%10);
        return r;
    }
}

```

```python
pjt_dir> solc-windows.exe src/Random.sol
```

# 2. 데이터구조

## 2.1 배열

배열은 관련이 있는 데이터를 하나의 변수로 묶어 놓은 구조이다.
순서가 있기 때문에 인덱스를 사용하여 요소를 특정할 수 있다.
컴파일 시점에 메모리가 할당되는 고정배열과 실행시점에 메모리가 배정되는 동적배열로 구분할 수 있다.

### 2.1.1 고정배열

#### 고정배열의 선언

고정배열은 배열이 저장하는 데이터의 개수를 사전에 정할 수 있는 경우에 사용한다.
다음은 ```uint balance```를 배열, 크기 5개로 선언하는 코드이다.
```
uint[5] balance;
```

#### 고정배열은 인덱스를 사용하여 입력, 수정

우측은 요소가 3개로 정해져 있는 배열이므로 좌측의 고정배열에 할당할 수 있다.
또는 인덱스를 사용하여 입력하고 수정할 수 있다. 그러나 push() 함수를 사용할 수 없다.

```python
string[3] cities1=["Seoul", "Sydney", "Tokyo"];
```

고정배열에 대해 push는 사용할 수 없다.
```
int[5] mathMarks;
mathMarks.push(100); //오류. 고정배열에 push 사용하지 못함.
```

#### 데이터타입의 범위를 벗어나는 값을 지정하지 않는다

배열의 타입에 허용되는 값의 범위를 넘어서면 오류가 발생한다.
예를 들어, uint에 음수 값을 지정하거나, uint8에 256을 지정하거나, uint16에 65536을 넣으면 오류가 발생한다.
```
uint8[5] balance = [255, 255, 95, 50, -1];  //오류: -1이 음수
uint8[5] balance = [255, 256, 95, 50, 1];  //오류: 256이 8비트 범위 밖 -> uint16
uint16[5] balance = [255, 65536, 95, 50, 1];  //오류: 65536이 16비트 범위 밖 -> uint24
```

#### 함수에서 고정배열을 초기화

* 함수 내에서 배열을 초기화하려고 하면 형변환의 문제가 발생한다.
mathMarks는 정수배열 ```int[]```로서 storage에 선언되어 있다.
이 storage 배열을 아래와 같이 그룹으로 초기화하면 오류가 발생한다.
```
mathMarks=[100,60,95,50,80]; //오류: 함수 내 우측 배열은 uint8 memory -> mathMarks의 int로 변환 오류 
mathMarks=uint8([100,60,95,50,80]); //오류: 우측 배열 uint8 memory -> uint8() 형변환 오류
```

위 코드의 오류를 설명해보자.
* 함수 내에서 ```[100, 60, 95, 50, 80]```이라고 우측 값을 넣으면, ```int[] memory```가 된다. 게다가 값의 범위가 256을 넘지 않으므로 uint8[] memory가 된다.
* 따라서 uint8[] memory를 int[]로 넣지 못한다. 서로 타입이 서로 맞지 않는다.
* 반복문으로 하나씩 읽어서 ```uint8 -> int8 -> int```로 변환해 주어야 한다.
```
contrat ArrayTest2 {
    int[5] mathMarks;  //storage 변수
    function setMathMarks() public {
        uint8[5] memory temp = [100, 60, 95, 50, 80];
        for (uint i = 0; i < temp.length; i++)
            mathMarks[i] = int(int8(temp[i])); //ok: uint8 -> int8 -> int256
    }
}
```

#### 배열 storage의 읽기

* 함수에서 storage 배열을 반환하려면, 값을 하나씩 돌려주도록 한다.
* 함수에서 storage 배열의 전체를 반환하는 것은 쉽지 않다.
그 이유는 함수 내에서 반환하려면 memory로 해야 하고, storage와 타입이 맞지 않게 되기 때문이다.

* 우선 메모리배열을 선언하여 공간을 확보한다. 
```
int[] memory _arr256 = new int[](mathMarks.length);
```

* 반복문으로 값을 하나씩 stoarage -> memory로 옮기고 반환한다.
```
contrat ArrayTest2 {
    int[5] mathMarks;  //storage 변수
    function getMathMarks() view public returns (int[] memory) {
        //return mathMarks;  //storage를 memory로 반환하면 오류 발생: int[] storage -> int[] memory
        int[] memory _arr256 = new int[](mathMarks.length);
        for (uint i = 0; i < mathMarks.length; i++)
            _arr256[i] = int256(mathMarks[i]);
        return _arr256;
    }
}
```

#### 배열의 삭제

배열을 삭제하는 경우 특정 인덱스를 정해서 ```delete array[index]```.
그러나 그 데이터 항목은 그대로 유지된다는 점에 주의해야 한다.
예를 들어, 배열 ```data``` ```[1, 2, 3, 4]```에서 ```delete data[2]```하면 ```[1, 2, 0, 4]```가 된다.

#### 배열의 크기

```length```를 사용하여 배열 크기를 알 수 있다.


### 2.1.2 동적배열

고정배열은 크기가 사전에 정해져 있는데 반해, 동적배열은 크기가 정해져 있지 않아서 실행시점에 메모리가 할당되며 크기가 수정될 수도 있다.
동적배열의 선언은 크기없이 다음과 같이 한다.

* 초기화

```python
int[] arr; //동적배열 선언
```

#### **동적 배열은 그 크기를 알 수 없으므로 반환을 할 수 없다**.

배열을 반환하는 경우 사전에  크기를 알아야 한다.
또는 ```pragma experimental ABIEncoderV2;```라고 프로그램 앞에 선언해주어야 한다.

#### 동적배열은 storage에서만 가능하다. memory 영역에서는 사용할 수 없다.

함수 내에서 동적배열 선언을 하기 위해서는 memory 키워드를 사용할 수 있지만,
memory의 배열은 크기를 동적으로 재할당할 수 없다.
memory 영역에서는 동적배열을 사용하는 필요성이 없어지게 된다.

동적배열을 사용하려면, storage 영역을 사용하는 상태변수를 이용해야 한다.
```python
contract my { 
    uint[] myArr; //동적배열 가능.
    function setArr() public {
       myArr.push(11); //동적배열에 대해 push 함수를 사용한다.
    }
}
```

```python
contract my { 
    function setArr() public {
       uint[] myArr; //오류. 로컬에서 동적배열 사용할 수 없다.
    }
}
```

#### ```push```는 ```storage``` 배열에만 사용하고 (즉 상태변수의 배열), ```memory``` 배열에는 사용할 수 없다.

동적배열은 storage에서만 가능하고, 그 배열에는 push() 함수를 사용하여 데이터를 추가할 수 있다
동적배열로 선언했다 하더라도 memory 배열에 대해서는 push 함수를 사용할 수 없다.

#### 동적배열을 memory에서 사용할 때 new, 괄호 안에 개수 적는다.

동적배열을 memory에서 사용하면, 그 크기를 정해서 초기화해주어야 한다.
이 겨우 ```new``` 명령어로 생성할 수 있고, 괄호 안에 개수를 적어서 초기화할 수 있다.
갯수가 정해지면 고정배열이 되어 버린다.
 
```python
function setLocalDynamicArr() view public returns(uint) {
	//uint[] myArr; //오류. storage 또는 memory 선언 필요
	//uint[] storage myArr; //오류. uninitialized storage
	//uint[] memory myArr; //오류. 동적배열 선언 가능. 그러나 push 못함.
	//uint[] myArr = new uint[](3); //오류. storage 또는 memory 필요.
	//uint[] storage myArr = new uint[](3); //오류. 좌측은 memory. 우측 stoarge로 변환 오류
	uint[] memory myArr = new uint[](3); //정상. memory 동적배열은 크기를 할당해야 한다.
	//myArr.push(11); //오류. myArr 크기가 정해져서, 고정배열에 push 사용하지 못함.
	myArr[0]=11; //정상. 인덱스를 사용해서 입력
	myArr[1]=12; //정상.
    //myArr.push(13); //오류.
	//myArr[5]=15; //오류.
    return myArr.length; //반환 3
}
```

또는 다음과 같이 초기화하지 않으면, 컴파일 오류는 없지만 실행오류가 발생한다.
크기를 정해주어야 한다는 것을 재확인하게 된다.
```
function setArr() pure public {
		uint[] memory myArr;
	    myArr[0]=11;
	    myArr[1]=12;
	    myArr[5]=15;
```

```string```은 그 자체로 동적배열이다.

```python
string[] cities2;
cities2.push("New York");
cities2.push("Beijing");
```

```
function setCities() public {
    //string my = "Seoul"; //오류. 우측이 로컬이므로 좌측 string은 memory
    string memory my = "seoul"; //정상
    //string[2] storage places = ["9000", "Sydney"]; //오류
    string[2] memory places = ["9000", "Sydney"];  //정상
}
```


### 2.1.3 다차원배열

정수의 동적 2차원배열이다.
앞의 3은 요소의 개수를, 뒤는 배열의 개수를 의미한다.
다음 예는 3개의 요소를 가진 배열이 2개라는 선언이다.

```python
uint[3][] marks=[[100, 80, 95],[20,30,40]];
```

## 실습: 배열

### 단계 1: 컨트랙 개발

함수 명 | 설명
-----|-----
```getDynamicArrMemory()``` | 메모리에 선언된 동적배열이지만 데이터를 채우고 크기를 알게 되면 반환가능 ```uint[] memory```
```getStringDynamicArrMemory()``` | ```string```은 그 자체로 동적배열이므로 반환 불가능 ```string[] memory```
```getCities1_()``` | ```string```은 그 자체로 동적배열이므로 반환 불가능 ```string[] memory```
```getCities1()``` | ```string``` 1개씩은 반환 가능
```getCities2()``` | ```pragma experimental ABIEncoderV2```를 사용하면 ```string``` 동적 배열 반환가능
```getMathAbove70()``` | 크기를 결정하지 않고, 값을 할당할 수 없슴
```function setMathMarks()``` | storage 변수를 함수 내에서 초기화하려면, 요소 하나씩 형변환한다.
```function getMathMarks()``` | storage 변수를 함수 내에서 반환하려면, 요소별로 형변환하여 memory변수로 반환한다.


```python
[파일명: src/ArrayTest2.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
pragma experimental ABIEncoderV2;

contract ArrayTest2 {
    string[3] cities1=["Seoul", "Sydney", "Tokyo"];
    string[] cities2 = new string[](2);
    string[] cities3;
    uint24[5] balance = [255, 65536, 95, 50, 1];
    int[5] mathMarks;
    uint[3][] marks=[[100, 80, 95],[20,30,40]];

    function setLocalDynamicArr() pure public returns(uint) {
        //uint[] myArr; //error
        //uint[] storage myArr; //error uninitialized storage
        //uint[] memory myArr; //ok. but cannot push
        //uint[] myArr = new uint[](3); //error
        //uint[] storage myArr = new uint[](3); //error conversion from memory to stoarge
        uint[] memory myArr = new uint[](3);
        //myArr.push(11); //error
        myArr[0]=11;
        myArr[1]=12;
        //myArr.push(13); //error
        //myArr[5]=15;
        return myArr.length;
    }
    function getDynamicArrMemory() pure public returns(uint[] memory) {
        uint[] memory num=new uint[](3);  //dynamic
        for (uint i=0; i<num.length; i++)
            num[i]=i;       //push() not allowed for array memeory
        return num;
    }
    //string is a dynamic array itself, which can not be returned.
    //function setArrMemory() view public returns(string[] memory) {
    function getStringDynamicArrMemory() pure public {
        //array storage not allowed
        //error: string[2] storage places = ["9000", "Sydney"];
        string[2] memory places = ["9000", "Sydney"];
        //array memory push not allowed
        //places.push("Seoul");
        places[0]="Seoul";
        //return places;
    }
    /*returning 'string[] storage' is not allowed
    function getCities1_() view public returns(string[] memory) {
        return cities1;  //can not return stoarge var. 
    }*/
    function getCities1() view public returns(string memory) {
        return cities1[0];
    }
    function getCities1Length() view public returns(uint) { return cities1.length; }
    function setCities23() public {
        //string my = "Seoul"; //error
        string memory my = "seoul";
        cities2[0]="New York";
        cities2.push(my);
        cities2.push("Busan");
        cities3.push("New York");
        cities3.push("Beijing");
    }
    //dynamic array return needs "pragma experimental ABIEncoderV2;"
    function getCities2() view public returns(string[] memory){
        return cities2;
    }
    function setMathMarks() public {
        //mathMarks.push(100); //push for fixed array not allowed
        //mathMarks=uint8([100,60,95,50,80]); //error: uint8 memory -> uint8
        uint8[5] memory temp = [100, 60, 95, 50, 80];
        //mathMarks = int(temp); //error: uint8[5] memory -> int256
        for (uint i = 0; i < temp.length; i++) {
            //mathMarks[i] = int(temp[i]); //error: uint8 -> int256
            mathMarks[i] = int(int8(temp[i])); //ok: uint8 -> int8 -> int256
        }
    }
    function getMathMarks() view public returns (int[] memory) {
        //return mathMarks;  //can not return storage array: error: int[] storage -> int[] memory
        int[] memory _arr256 = new int[](mathMarks.length);
        for (uint i = 0; i < mathMarks.length; i++) {
            _arr256[i] = int256(mathMarks[i]);
        }
        return _arr256;
    }
    //run setMathMarks() beforehand
    function getMathAbove70_() view public returns(int[] memory) {
        // size is not allocated yet -> invalid opcode error
        int[] memory mathAbove70;
        uint counter = 0;
        for(uint8 i=0;i<mathMarks.length;i++)
            if(mathMarks[i]>70) {
                mathAbove70[counter] = mathMarks[i];
                //mathAbove70.push(mathMarks[i]);
                counter++;
            }
        return mathAbove70;
    }
    //run setMathMarks() beforehand
    function getMathAbove70() view public returns(int[] memory) {
        //compute lengthOfMathAbove70
        uint8 counter=0;
        uint8 lengthOfMathAbove70=0;
        for(uint8 i=0;i<mathMarks.length;i++)
            if(mathMarks[i]>70) counter++;
        lengthOfMathAbove70=counter;
        //move math marks above 70
        int[] memory mathAbove70=new int[](lengthOfMathAbove70);
        counter=0;
        for(uint i=0;i<mathMarks.length;i++) {
            if(mathMarks[i]>70) {
                mathAbove70[counter]=mathMarks[i];
                counter++;
            }
        }
        return mathAbove70;
    }
    function updateMarks() public returns(uint[3][] memory){
        marks[0][0]=90;
        return marks;
    }
    function getMarksArr() view public returns(uint[3][] memory) {
        return marks;
    }
    function getMarksLength() view public returns(uint) {
        return marks.length;
    }
}
```


### 단계 2: 컴파일


```python
pjt_dir> solc-windows.exe src/ArrayTest2.sol --combined-json abi > src/ArrayTest2ABI.json
```


```python
pjt_dir> solc-windows.exe src/ArrayTest2.sol --combined-json bin > src/ArrayTest2BIN.json
```

### 단계 3: 컨트랙 배포

#### estimateGas

배열은 ```gas```를 많이 필요로 한다. ```Gas```를 산정해보니, 앞서 컨트랙보다 많이 1,226,648가 발생하고 있다.
1,226,647로 1을 줄여서 하면 ```out of gas``` 오류가 발생한다.

```python
new web3.eth.Contract(_abiArray).deploy({data: _bin}).estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);});
```


```python
[파일명: src/ArrayTest2Deploy.js]
var Web3=require('web3');
var _abiJson = require('./ArrayTest2ABI.json');
var _binJson = require('./ArrayTest2BIN.json');

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
contractName=Object.keys(_abiJson.contracts); // reading ['src/ArrayTest2.sol:ArrayTest2']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=_abiJson.contracts[contractName].abi;
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_bin="0x"+_binJson.contracts[contractName].bin; //ok without "0x"

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);

    new web3.eth.Contract(_abiArray).deploy({data: _bin}).estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);
    });

    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1621137}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address);
}

deploy()

```

gas를 산정한 수치보다 적게 적으면 gas limit오류가 발생하는 것을 확인해보자.
함수가 추가될수록 gas는 증가하는 것이 당연할 수 밖에 없다.
함수 안에서 일어나는 연산, storage 변수 갱신, 반복문 등이 비용을 증가시키는 요인이 되고 있다.


```python
pjt_dir> node src/ArrayTest2Deploy.js


    - contract name:  src/ArrayTest2.sol:ArrayTest2
    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0x0cd55fad15ed2616cb165718f22b207843e57167007a92d3607f8433545e410f
    ---> The contract deployed to: 0x23293b04d95B80b7aFbcb20a70258203ad040448
    >> gas: 1621137
```

### 단계 4: 사용

#### estimateGas

gas를 없이 트랙잭션을 발생하면, ```out of gas``` 오류가 발생한다.
```python
await arr.methods.setCities23().send({from: accounts[0]});  //out of gas error
```

```estimateGas()``` 함수를 사용하여 ```gas```를 산정하면 130312가 소요된다.
그 값을 넣어주면 오류가 없어지게 된다.

```python
await arr.methods.setCities23().send({from: accounts[0], gas:130312});
```

arr.methods.setMathMarks().send({from: accounts[0], gas: 162354});


```python
[파일명: src/ArrayTest2Use.js]
var Web3=require('web3');
var _abiJson = require('./ArrayTest2ABI.json');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //ok

contractName=Object.keys(_abiJson.contracts); // reading ['src/ArrayTest2.sol:ArrayTest2']
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=_abiJson.contracts[contractName].abi;
//_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!

async function doIt() {
    var arr = new web3.eth.Contract(_abiArray, "0x23293b04d95B80b7aFbcb20a70258203ad040448");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    
    arr.methods.setCities23().estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);
    });

    //await arr.methods.setCities23().send({from: accounts[0]});   //out of gas error
    await arr.methods.setCities23().send({from: accounts[0], gas:133310});
    arr.methods.getCities2().call().then(console.log);

    arr.methods.setMathMarks().estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);
    });

    await arr.methods.setMathMarks().send({from: accounts[0], gas: 162354});
    //ERROR invalid opcode arr.methods.getMathAbove70().call().then(console.log);
    arr.methods.getMarksArr().call().then(console.log);

    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}

doIt()

```


```python
pjt_dir> node src/ArrayTest2Use.js


    - contract name:  src/ArrayTest2.sol:ArrayTest2
    Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    Balance before: 999941007891999987456
    >> gas: 127611
    [ 'New York',
      '',
      'seoul',
      'Busan',
      'seoul',
      'Busan',
      'seoul',
      'Busan',
      'seoul',
      'Busan' ]
    >> gas: 33810
    Balance after: 999940685049999987456
    Balance diff: 322841999966208
    [ [ '100', '80', '95' ], [ '20', '30', '40' ] ]
```

```gas``` 비용은 ```ArrayTest2Use.js```를 **다시 실행하면 많이 낮아진다**.
첫 회의 실행에서는 동적배열을 생성하고, 다음 실행에서는 그 비용이 줄어들기 때문에 ```gas```비가 낮아지고 있다.


```python
pjt_dir> node src/ArrayTest2Use.js


    - contract name:  src/ArrayTest2.sol:ArrayTest2
    Account: 0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1
    Balance before: 99924320779999997778
    >> gas: 95769
    [ 'New York', '', 'Busan', 'Busan' ]
    >> gas: 23209
    [ [ '100', '80', '95' ], [ '20', '30', '40' ] ]
    Balance after: 99921941219999997778
    Balance diff: 2379560000012288
```

#### 배열의 출력

```web3.js``` 현재 버전에서는 Solidity -> node로 배열의 전달에 문제가 없어서 출력이 올바르게 되고 있다.
이전 버전에서는 배열을 올바르게 출력하려면 ```toString()``` 함수를 사용해야 한다.
아래 BigNumber의 표현에서 s(ignal), e(xponent), c(oefficient)는 각 각 부호, 지수 및 계수를 의미한다.

```python
> arraytest.getMarksArr()                      출력하기 위해서는 toString()
[ [ BigNumber { s: 1, e: 2, c: [Array] },
    BigNumber { s: 1, e: 1, c: [Array] },
    BigNumber { s: 1, e: 1, c: [Array] } ],
  [ BigNumber { s: 1, e: 1, c: [Array] },
    BigNumber { s: 1, e: 1, c: [Array] },
    BigNumber { s: 1, e: 1, c: [Array] } ] ]
> arraytest.getMarksLength()
BigNumber { s: 1, e: 0, c: [ 2 ] }
> arraytest.getMarksLength().toString()
'2'
> arraytest.getMarksArr().toString()
'100,80,95,20,30,40'
```

## 실습: Members 배열

배열을 검색하는 것은 비용이 많이 발생한다. 
mapping을 사용하거나, 클라이언트 측에서 배열 데이터를 검색하는 편으로 하자.

문자열비교는```keccak256()``` 해싱을 한 후 비교해야 한다.
아래와 같이 문자열을 해싱한후 ==로 비교를 해야 한다.
```
keccak256(abi.encodePacked(문자열))==keccak256(abi.encodePacked(찾는 문자열))
```


```python
[파일명: src/Members.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Members{
    address owner;
    event printAddr(address arg);
    struct Member{
        uint id;
        string name;
    }
    Member[] public memberArr;
    constructor() { //constructor() public {
        owner=msg.sender;
    }
    function del() public {
        delete memberArr;
    }
    function delOne(uint i) public{
        delete memberArr[i];  //try pop()
    }
    function add(uint id,string memory name) public {
        memberArr.push(Member(id,name));
    }
    //return Member
    function getMember(string memory who) view public returns(uint, string memory) {
        uint _id;
        string memory _name;
        for(uint i=0;i<memberArr.length;i++) {
            _name=memberArr[i].name;
            if(keccak256(abi.encodePacked(_name))==keccak256(abi.encodePacked(who))) {
                _id=memberArr[i].id;
                _name=memberArr[i].name;
            }
        }
        return (_id,_name);
    }
    function compareStr(string memory s1, string memory s2) pure public returns(bool) {
        return keccak256(abi.encodePacked(s1))==keccak256(abi.encodePacked(s2));
    }
    function compareBytes(bytes memory b1, bytes memory b2) pure public returns(bool) {
        return keccak256(b1) == keccak256(b2);
    }
    function getLength() view public returns(uint) {
        return memberArr.length;
    }
}
```


```python
pjt_dir> solc-windows.exe src/Members.sol
```

## 2.2 mapping

### 2.2.1 키와 값 쌍으로 저장

```mapping```은 `키`와 `값`을 쌍으로 저장한다. **```=>```** 기호를 사용하여, 좌측에 키를 오른쪽에 값을 적어준다.
다음 ```mapping``` 코드를 보면:
* ```(string => uint)```에서 보면 키는 ```string```, 값은 ```uint```로 정의 한다는 의미이다.
* ```mapping (string => uint) balances```는 mapping 변수 명을 ```balances```로 선언하고 있다.

이렇게 선언된 변수 balance에, ```string```으로 선언된 키에 ```jsl```, ```uint```로 선언된 값에 잔고 100을 저장한다.

```python
mapping (string => uint) public balances;
balances["jsl"] = 100;
```

struct에 데이터를 넣을 경우, 아래 모두 가능한 코드이다.
```python
contract Customer {
    struct CustomerDetails {
        uint id;
        string name;
        string phNumber;
        string homeAddress;
    }
    mapping(address=>CustomerDetails) customers;
    function addCustomer(uint _id, string memory _name, string memory _ph, string memory _home) public {
        //ok
		//customers[tx.origin].id = _id;
        //customers[tx.origin].name = _name;
        //customers[tx.origin].phNumber = _ph;
        //customers[tx.origin].homeAddress = _home;
		//ok
        CustomerDetails memory cd = CustomerDetails(_id, _name, _ph, _home);
        customers[tx.origin] = cd;
	}
}
```

## 실습: 은행 잔고의 mapping

* 처음 ```_mint()``` 함수로 잔고를 채워 놓을 수 있다.

* 앞서 은행에서는 한 계정에서의 입금, 출금을 실행해 보았다.
여러 계정이 있는 경우, mapping을 이용해 잔고를 관리할 수 있다.

```mapping(address=>uint) balanceOf;``` 라고 선언하고
```balanceOf[<address>]=amount;```로 잔고를 저장할 수 있다.

증액, 감액을 하려면:
```python
uint balanceToAdd=111;
balanceOf[<address>] += balanceToAdd;
```

* adddress 자신의 주소를 따옴표 없이, 금액도 따옴표 없이 정수

입출금은 기능을 어떻게 설계하느냐에 따라 금액변동에 영향을 미칠 수 있기 마련이다.
여기서는 입출금 및 계좌이체를 하면 자신의 계좌, owner 계좌 그리고 컨트랙의 계좌에 아래와 같이 변동이 있도록 해보았다.

함수 | 관련 잔고 | owner잔고 | 컨트랙 잔고
-----|-----|-----|-----
deposit(111)을 하면 | owner잔고에 입금함 | owner 잔고 111 증가 | 컨트랙잔고 111 증가
forwardTo(0x1234..., 11)을 하면| 계좌이체이므로 Ox1234... 잔고 11 증가 | owner잔고 11감소 | 컨트랙 잔고 변동 없슴
withdraw(0x1234..., 11)을 하면 | Ox1234... 잔고 11 감소 | owner잔고 변동 없슴 | 컨트랙 잔고 11감소


```python
[파일명: src/BankV4.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

//check for Coin.sol (ch12) and BankV3.sol 201906
contract BankV4 {
    address owner;
    mapping (address => uint) balanceOf;
    constructor() {//constructor() public {
        owner = msg.sender;
        _mint(1000000000000000000000000);
    }    
    // save to individual addresses
    function deposit(uint amount) payable public onlyOwner {
        require(msg.value == amount);
        balanceOf[msg.sender] += amount;
    }
    // forward from owner to another
    function forwardTo(address receiver, uint amount) payable public onlyOwner {
        require(balanceOf[msg.sender] >= amount);
        balanceOf[msg.sender] -= amount; // Subtract from the sender
        balanceOf[receiver] += amount;   // Add the same to the recipient
    }
    function withdraw(address payable receiver, uint amount) public onlyOwner {
        require(balanceOf[receiver] >= amount && address(this).balance >= amount);
        balanceOf[receiver] -= amount;
        receiver.transfer(amount);
    }
    function getBalance() public view returns(uint, uint) {
        return (address(this).balance, balanceOf[owner]);
    }
    function getBalanceOf(address addr) public view returns (uint) {
        return balanceOf[addr];
    }
    function _mint(uint amount) onlyOwner {
        balances[msg.sender] += amount;
    }
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}
```


```python
pjt_dir> solc-windows.exe src/BankV4.sol
```


### 2.2.2 중복키 확인

맵은 그 특성상 중복키는 허용되지 않는다. 동일한 키로 당연히 데이터를 입력할 수가 없다.
맵에 이미 키가 존재하는지 확인할 수 있는 ```containsKey()```와 같은 지원 함수가 없다.
필드 ```isMember```를 설정해놓고 그 값을 확인하는 방법으로 하자.
```
mapping(address=>Member) memberMap;
struct Member {
	uint id;
	string name;
	bool isMember;
}
if (memberMap(<address>).isMember) 중복키 존재함
```

* 삭제

키삭제는 쉽지 않다. 다른 언어에서는 ```remove()```함수가 지원되어, 간편하게 키를 지울 수 있는 것과 비교된다.

```
delete pendingAccessRequest[key];
```

### 2.2.3 bidirectional map

주소에 대해 회원(아이디, 이름)을 저장하는 ```map```이다.
이 경우 'id' 또는 '이름'으로 '주소'를 검색하는 기능은 ```bidiretional map```으로 구현하였다.
이렇게 하면, 반복문을 사용하지 않고 검색하는 기능이 가능해 진다.
특정 id에 해당하는 주소를 찾고, 그 주소에 해당하는 값을 읽어올 수 있게 된다.

줄 | 설명
-----|-----
9 | 주소의  ```Member strcut ```을 조회하는 map
10 | 이름의 주소를 조회하는 map
11~19 ```addMember() ``` |  ```Member struct```을 추가하고 ```getMemberByName()``` 함수에서 이름을 입력하면 주소를 조회할 수 있도록 저장. ```name```이 중복되는 경우, 엎어쓰기 때문에 문제가 될 수 있다. 실제는 중복이 없는 ```id```를 사용하여 ```bidirectional map```을 구현해야 한다.
20~24 ```getMemberById()``` | bidirectional map에서 ```id```에 해당하는 주소를 조회하고, 그 주소키를 이용 Member를 조회
25~28 ```getMemberByName()``` | bidirectional map에서 ```name```에 해당하는 주소를 조회
29~32 ```getMember()``` | 주소에 해당하는 ```Member struct```을 조회. (1) addressById[정수]를 하면 주소를 얻을 수 있다. (2) 그러면 memberMap[주소]는 Member라는 값을 얻을 수 있다.


```python
[파일명: src/MembersMap.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract MembersMap {
    struct Member {
        uint id;
        string name;
    }
    //bidrectional map
    mapping(address=>Member) memberMap;
    mapping(uint=>address) addressById;
    mapping(string=>address) addressByName;
    function addMember (uint _id, string memory _name) public {
       Member memory m=Member(_id, _name);
       memberMap[msg.sender]=m;
       //saving into a bidiretional map to get address by id
       addressById[_id]=msg.sender;
       //saving into a bidiretional map to get address by name
       addressByName[_name]=msg.sender;
    }
    //get Member by id
    function getMemberById(uint _id) view public returns(uint, string memory) {
        Member memory my = memberMap[addressById[_id]];
        return (my.id, my.name);
    }
    // get address (not Member) by name
    function getMemberAddressByName(string memory _name) public view returns(address) {
        return addressByName[_name];
    }
    function getMember(address addr) view public returns (uint, string memory) {
        Member memory m=memberMap[addr];
        return (m.id, m.name);
    }
}
```

```python
pjt_dir> solc-windows.exe src/MembersMap.sol
```

# 3. 예외

오류는 발생하기 마련이다. 컴파일 오류는 수정하기 비교적 용이하다. 그러나 실행시점에 발생하는 오류는 잡아내기 어렵다.
다른 언어에서 보통 제공하는 ```try..catch``` 구문도 제공되지 않는다.
버전 0.4.10 이전에는 ```throw``` 명령문이 사용되었다.

```python
if (msg.sender != owner) { throw; }
```

버전 4.10이후에는 ```throw```구문은 더 이상 사용되지 않고 require, revert, assert 구문이 사용되어 예외가 발생한다.
* if (msg.sender != owner) { revert(); } throw 명령문을 revert()로 교체해도 동일한 기능이 수행된다
* require(msg.sender == owner) 조건이 부정에서 긍정으로 변경된다
* assert(msg.sender == owner) 조건이 부정에서 긍정으로 변경된다

발생하는 예외는 여러 경우가 있을 수 있다. 실행 시점에 발생하는 예외는 배열이 범위를 넘어서거나 ```out-of-gas errors```, ```divide by zero```와 같이 어떤 특정 조건에서 예외를 발생시킬 수 있다.
이 경우 실행이 중지되고, 원상복구 (상태변수와 잔고)된다.

## 3.1 require
```require()```문은 자주 쓰이는 명령문으로 실행한 결과를 ```true```, ```false```로 반환된다. ```false```인 경우 예외가 발생하고 실행이 중지된다. 미사용 gas를 호출자에게 반환하고 상태를 원래대로 복원한다.
* 송금이 되었는지 확인하는 경우 ```require(<address>.transfer(amount))```
* 조건이 충족되었는지 확인하는 경우 ```require(msg.value == amount)```, ```require(address(this).balance >= amount)```
* 함수의 진행에 필요한 사전조건을 확인하는 방식으로, 보통 함수의 맨 앞에 위치하여 사용된다.

## 3.2 revert

revert() 문은 예외가 발생하는 이유를 설명하는 **문자열만 매개변수로 받을 수** 있다. 조건을 매개변수로 받지 않는다.
```revert()```문은 ```require()```함수와 동일한 의미지만, ```revert()```문은 조건이 없기 때문에 참인지, 거짓인지 어떤 평가를 하지 않는다는 점이 다르다.
따라서 조건을 매개변수로 받는 ```require()``` 문과 달리, 조건을 외부에 두어 **```if/else```**문과 사용되어 예외를 발생하는 방식으로 사용될 수 있고 보다 복잡한 조건을 사용하기 편리하다.
```require()```와 마찬가지로 ```revert()``` 명령문은 미사용 gas를 호출자에게 반환하고 상태를 원래대로 복원한다.

## 3.3 assert
```assert()``` 함수는 내부조건 또는 제약조건이 충족되었는지 확인하는 경우에 사용된다.
이러한 오류가 발생하였는지 디버깅 용도로 사용된다.
미사용 gas는 반환되지 않고, 모두 소모된다. 상태는 원래대로 복원된다.
* overflow/underflow ```assert(age > 20)```
* 항상 충족해야 하는 조건 ```assert(address(this).balance >= 0)```
* 함수가 끝나고 나서 사후조건을 확인하는 방식으로, 보통 함수의 맨 뒤에 위치하여 사용된다. ```require```에 비해 자주 사용되지 않는다.

명령문 | gas 반환 | 원복 | 사용하는 경우
-----|-----|-----|-----
```assert()``` | NO | YES | overflow/underflow, invariants 등 어떤 오류가 발생하는 것을 예방하는 경우. 실패하면 중단되며, 다음으로 진행되지 않는다.
```revert()``` | YES | YES | ```require()```는 참, 거짓을 평가하지만, revert()는 조건문 없이 수행된다. ```revert```되면 중단되며, 다음으로 진행되지 않는다.
```require()``` | YES | YES | 사용자 입력 , 상태변수 값, 반환 값 등 조건을 검증하는 경우. 실패하면 중단되며, 다음으로 진행되지 않는다.

## 3.4 재진입 공격

재진입 공격 Reentrancy Attack이란 예외가 발생하여 중도에 멈추어야 하지만, 그렇지 않게 실행에 재진입하여 재실행을 시도하는 것을 말한다.
예를 들면 인출이 실패하면 예외가 발생해야 하지만, receive() 또는 fallback() 함수에 인출 코드를 넣어 놓아서 재인출을 시도하는 경우이다.

이런 재진입 공격을 막으려면 몇 가지 방법이 있다.
(1) 우선 잔고를 사전에 초기화 하면 된다.
이른바 Checks-Effects-Interactions 패턴을 적용하여 재진입공격을 무력화하는 것이다.
즉 송금이라는 실행 (Interactions) 이전에 잔고 차감을 적용한다 (Effects). 잔고 초기화를 나중에 하게 되면, 잔고가 남아 있어 재송금공격을 할 수 있기 때문이다.
공격자가 receive() 또는 fallback() 함수로 인출을 가능하게 코딩해 놓으면, 인출이 실패하더라도 인출 재시도가 가능하다.
아예 잔고를 0으로 만들어 놓으면 송금이 불가능해진다.

(2) Flag를 사용
성공적으로 송금되면 flag를 false로 만들고, false일 경우에는 송금시도가 아예 불가능하도록 한다.
아래 ```withdrawFlag()``` 함수를 참조하자.

(3) gas limit
send(), transfer()의 gas limit은 2,300으로 제한되어 있어서 이런 재진입 시도는 어렵다.
사용할 gas가 없으면 재진입시도가 아예 불가능해 진다.


```python
[파일명: src/ReentrancyAttack.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Withdrawing {
    mapping(address => uint) internal balances;
    bool internal locked = false;
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }
    function withdrawInsecure() public {
        uint _amount=balances[msg.sender]; //to send all total
        (bool success, ) = msg.sender.call{value: _amount}("");
        require(success, "Failed to send");
        balances[msg.sender] = 0;
    }
    function withdrawSecure() public {
        uint _amount=balances[msg.sender]; //to send all total
        (bool success, ) = msg.sender.call{value: _amount}("");
        require(success, "Failed to send");
        balances[msg.sender] = 0;
    }
    function withdrawFlag() public {
        require(!locked);
        locked = true;
        withdrawInsecure();
        locked = false;
    }
    function getBalanceOfMsgSender() view public returns(uint) { return balances[msg.sender]; }
}

contract ReentrancyAttack {
    Withdrawing internal w;
    constructor(address _w) { //provide the adderss of Withdrawing contract
        w = Withdrawing(_w);
    }
    receive() external payable {
        if (address(w).balance >= 1 ether) {
            w.withdrawInsecure();
        }
    }
    function withdrawAttack() external payable {
        require(msg.value >= 1 ether);
        w.deposit{value: 1 ether}();
        w.withdrawInsecure();
    }
    function getBalance() view public returns(uint) { return address(this).balance; }
}

```


## 실습: 예외


```python
[파일명: src/ExceptionTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract ExceptionTest {
    address owner;
    constructor() {//constructor() public {
        owner=msg.sender;
    }
    function requireException() view public returns(string memory) {
        //if (msg.sender != owner) { throw; } // not this way to throw an exception
        require(msg.sender != owner, "Sorry! You are owner. Require failed...");
        return "require condition met";
    }
    function assertException() view public returns(string memory) {
        assert(msg.sender == owner);
        return "asserted";  //do this line only 'assert' is succeeded
    }
    function revertException() view public returns(string memory) {
        if(msg.sender != owner) {
            revert("Sorry! You are NOT owner. Reverted...");
            //return "reverted";  // this line can't be reached if revert is executed 
        } else {
            return "not reverted";
        }
    }
    function raiseException() pure public {
        int x=0;
        int y=0;
        x/y;    //divide by zero
    }
}
```


```python
pjt_dir> solc-windows.exe src/ExceptionTest.sol
```

## 실습: 주문에 예외 적용

* 제한시간: 220분

```Customer```, ```Order``` 컨트랙을 개발하고, 블록체인에서 주문 거래 프로그램을 작성하세요.
### ```Customer``` 컨트랙의 멤버 속성과 함수:
필요한 상태변수와 함수를 구성하고, 명칭과 입출력과 수식어도 의미있게 결정한다.
- 상태변수에는:
	- 고객ID, 이름, 전화번호, 주소
    - 계정주소에 대한 ```mapping``` 등 그 외 필요한 상태변수를 추가.
- 고객정보 입력 함수 addCustomer(uint _id, string memory _name, string memory _ph, string memory _home)
- 배송지 주소 조회 함수 getHomeAddress()
- 고객id 조회 함수 getId()

### ```Order``` 컨트랙의 멤버 속성과 함수:
필요한 상태변수와 함수를 구성하고, 명칭과 입출력과 수식어도 의미있게 결정한다.
주의 할 점은, ```Order``` 컨트랙을 통해 고객정보를 입력하거나 배송지를 조회하지 않을 수 있지만,
```Customer``` 컨트랙을 따로 배포해야 가능하다. 이 문제에서는 ```Customer``` 객체를 통해서 이들 함수를 호출한다(new Customer()).
주문을 삭제하는 기능은 구현하지 않는다.
- 상태변수에는:
	- 주문ID, 상품명, 개수, 금액, 시간, 상태, 배송지
	- 계정주소에 대한 ```mapping``` 등 그 외 필요한 상태변수를 추가.
- 주문 함수 placeOrder (uint _id, string memory _p, uint _n, uint _amount)
	- 주문을 받으면 상태는 "Ordered"로, 시간은 현재시간으로 설정한다.
	- 주문 금액은 컨트랙에게 입금이 필요하다.
	- bidirectional map을 설정하여, 주문ID를 통하여 주문자 주소키를 알 수 있도록 힌다.
- 고객정보 입력 함수 addCustomer(uint _id, string memory _name, string memory _ph, string memory _home)
	- ```Customer``` 컨트랙을 통해 설정
- 배송지 주소 조회 함수 getHomeAddress(). ```Customer``` 컨트랙을 통해 조회
- 주문처리 상황 조회 함수 getStatus() 주문처리 상황 조회 함수
- 주문처리 상황 갱신 함수 updateStatus(uint _id, string memory _s)
- 주문내역 출력 함수 getOrderItem(): 주문했던 고객의 계정에 해당하는 주문ID, 상품명, 상태, 배송지 출력
- 주문ID로 주문내역 조회 함수 getOrderById(uint _id) : 주문ID를 입력하면 주문ID, 상품명, 상태, 배송지 출력
- 주문 갯수 조회 함수 getNOrder()
- 주문 총액 조회 함수 getTotalOrderAmount()
- 컨트랙 잔고 확인 함수 queryBalance() 
- 그 외 필요한 함수를 추가할 수 있다.

아래 문항별 10점씩 배점된다.
1~2는 REMIX에서 실행하고, 해당 화면캡쳐를 제출,
3~8은 ganache@8345에서 노드로 실행
9~10은 geth@8445에서 노드로 실행 (노드 단말에서 실행하면 해당 화면 제출)

### 1) REMIX Customer 화면
Customer의 addCustomer, getHomeAddress, getId 등 기능버튼 결과 보이도록 화면캡쳐 제출.

### 2) REMIX Order 화면
Order 버튼 결과 모두 출력, 특히 placeOrder할 때, 오른쪽 단말창이 열려서 녹색표시가 나오도록 화면캡쳐 제출.
REMIX에서 테스트하면서 콘솔 창에서 출력되는 필요한 gas를 확인하자. 

### 3) 계정, 잔고, gas 출력
온라인 주문에서는 많은 주문자가 있을 수 있다는 점에서 계정1, 계정2, 계정3 사용한다. 계정이 없으면 3개를 만들어 놓는다.
계정1, 계정2, 계정3과 각 잔고를 출력한다. 현재의 블록번호도 출력한다. 
```Order``` 컨트랙 생성에 필요한 gas 출력. ```Order``` 컨트랙은 바이트코드량과 그 기능이 많아 배포하기 위해서는 gas가 상당히 필요하다.

### 4) 고객정보 3건 입력
* 계정1을 사용하여 고객정보 입력 -> 111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul"
* 계정2를 사용하여 고객정보 입력 -> 112, "lee", "010-2017-1112", "112 hongji-dong jongro-gu seoul"
* 계정3을 사용하여 고객정보 입력 -> 113, "lim", "010-2017-1113", "113 hongji-dong jongro-gu seoul"
* 아래 ```tx.origin```와 ```msg.sender``` 관련 주의를 참조해서 ```msg.sender``` 대신 ```tx.orgin```을 사용하자.

### 5) 모든 고객의 배송지 출력
* 계정1을 사용하여 고객배송지 출력, "111 hongji-dong jongro-gu seoul"
* 계정2를 사용하여 고객배송지 출력, "112 hongji-dong jongro-gu seoul"
* 계정3을 사용하여 고객배송지 출력, "113 hongji-dong jongro-gu seoul"
* 고객배송지를 출력할 때, ```call({from: accounts[1]})```과 같이 해당 계정주소를 적어주고 호출한다.

### 6) 주문
* 주문하면서, 주문상태를 "Ordered"로 설정하고, 주문시간을 지금으로 설정한다.
* 계정1에서 주문내역 입력 -> 555, "T-Shirt", 2, 1115
* 계정2에서 주문내역 입력 -> 556, "T-Shirt", 3, 1116
* 계정3에서 주문내역 입력 -> 557, "T-Shirt", 4, 1117

### 7) 주문개수, 주문금액 합계 및 잔고 출력
* 주문입력을 잘 했으면, 주문개수 3건, 주문금액 합계 3348, 컨트랙잔고를 출력한다. 잘 풀었으면 컨트랙잔고는 주문총액과 동일하다.
* 주문ID 556으로 주문내역 출력. 계정을 입력해서 출력하지 않는다. -> 556, "T-Shirt", "On delivery", "112 hongji-dong jongro-gu seoul"

### 8) 모든 고객의 주문 내역 출력
556번 주문배송중 갱신해서 출력, 관리자만 할 수 있게 제한한다.
* 계정1에서 주문내역 출력 -> 555, "T-Shirt", "Ordered", "111 hongji-dong jongro-gu seoul"
* 계정2에서 주문내역 출력 -> 556, "T-Shirt", "On delivery", "112 hongji-dong jongro-gu seoul"
* 계정3에서 주문내역 출력 -> 557, "T-Shirt", "Ordered", "113 hongji-dong jongro-gu seoul"

### 9) geth@8445에서 배포하고, 고객정보 입력
* 계정1을 사용하여 고객정보 입력 -> 111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul"

### 10) geth@8445에서 앞 문항 9)에 이어서 실행
* 계정1에서 주문내역 입력 -> 555, "T-Shirt", 2, 1115
* 주문ID 555으로 주문내역 출력 -> 555, "T-Shirt", "Ordered", "111 hongji-dong jongro-gu seoul"

끝

## 연습문제 주문

사용자별 주문은 1:1 관계로 구현하고 있다 (1:n 관계로 구현하려면 복잡해진다). 주문id로 조회할 수 있도록 하기 위해 bidiectional map을 사용하였다.

줄 | 설명
-----|-----
4~12 | 주문 데이터, ```isOrdered```는 중복키 확인용도로 쓰인다.
14 | 주문 갯수. 반복문으로 갯수를 세는 것을 피하기 위해서 계속 추적한다.
15 | 주문 총액 역시 반복문을 피하기 위해 추적한다.
18 | 주문번호에서 검색을 하기 위해 설정한다.
22~30 ```placeOrder()``` | 주문금액 확인, 전송자 주소로 주문데이터 확인, 주문갯수 및 주문총액을 추적. bidirectional map을 이용해서 주문id 별로 주문내역을 검색할 수 있게 함. 주문id는 중복되지 않게 생성을 하도록 함.
32~35 ```updateStatus()``` | 주문번호를 이용해 주문의 상태를 갱신한다. 관리자만 갱신권한을 가진다.


## 연습문제

* 제한시간: 24시간

```Customer```, ```Order``` 컨트랙을 개발하세요.
컨트랙을 개발하면서 필요한 상태변수와 함수를 구성하고, 그 명칭과 입출력과 수식어도 의미있게 결정한다.

```Customer``` 컨트랙은 별도로 배포하면, ```Order``` 컨트랙을 통해서 데이터를 입력하지 않을 수 있지만,
이 문제에서는 ```new``` 명령어로 ```Customer``` 객체를 만들어 사용하자.
함수를 호출할 때 gas는 넉넉히 배정하도록 하자 (REMIX에서 테스트하면서 콘솔에서 출력되는 gas를 확인).
주문을 삭제하는 기능은 구현하지 않는다.
또한 온라인 주문에서와 같이 주문자는 여러 명이 있을 수 있다는 점에서 계정1, 계정2, 계정3 사용하고,
이런 계정별로 복수의 주문은 가능하지 않도록 구현한다. 현재는 1:1 관계, 즉 계정별로 주문은 1건만 가능하다.

### ```Customer``` 컨트랙의 멤버 속성과 함수:
- 상태변수
	- 고객ID, 이름, 전화번호, 주소, 마일리지, 고객여부 (고객이면 true)
    - 계정주소에 대한 ```mapping``` 등 그 외 필요한 상태변수를 추가할 수 있다.
- addMileage(uint amount) 마일리지 추가 함수: 주문금액의 1%를 마일리지로 부여
- getMileage() 마일리지 조회함수
- 고객정보 입력 함수 addCustomer(uint _id, string memory _name, string memory _ph, string memory _home)
    - 고객정보 입력은 고객ID가 같으면 입력이 되지 않게 한다.
- 배송지 주소 조회 함수 getHomeAddress()
- 고객id 조회 함수 getId()
- 고객여부 확인 함수 isCustomer()
- 그 외 필요한 함수를 추가할 수 있다.

### ```Order``` 컨트랙의 멤버 속성과 함수:
- 상태변수
	- 주문ID, 상품명, 개수, 금액, 시간, 상태, 배송지, 주문여부 (주문하면 true)
	- 계정주소에 대한 ```mapping``` 등 그 외 필요한 상태변수를 추가할 수 있다.
- 주문 함수 placeOrder (uint _id, string memory _p, uint _n, uint _amount)
    - 위 상태변수를 설정한다. 주문을 받으면서 상태는 "Ordered"로, 시간은 현재시간으로 설정한다.
    - 주문 금액 반드시 입금되어야 한다.
    - 마일리지는 ```Customer``` 컨트랙을 통하여 설정 (주문금액의 1%)
    - bidirectional map을 설정하여 주문ID를 통하여 주문자 계정주소를 알 수 있도록 힌다.
- 환불처리 함수: 상품ID에 대해 주문상태를 "refunded"로, 주문금액도 0으로 설정하고 마일리지도 차감하고, 그에 따라 잔고도 감해준다.
- 고객정보 입력 함수 addCustomer(uint _id, string memory _name, string memory _ph, string memory _home)
	- ```Customer``` 컨트랙을 통해 설정
- 배송지 주소 조회 함수 getHomeAddress(). ```Customer``` 컨트랙을 통해 조회
- 주문처리 상황 조회 함수 getStatus() 주문처리 상황 조회 함수
- 주문처리 상황 갱신 함수 updateStatus(uint _id, string memory _s)
- 주문내역 출력 함수 getOrderItem(): 주문했던 고객의 계정에 해당하는 주문ID, 상품명, 상태, 배송지 출력
- 주문ID로 주문내역 조회 함수 getOrderById(uint _id) : 주문ID를 입력하면 주문ID, 상품명, 상태, 배송지 출력
- 주문 갯수 조회 함수 getNOrder()
- 주문 총액 조회 함수 getTotalOrderAmount()
- 컨트랙 잔고 확인 함수 queryBalance() 
- 그 외 필요한 함수를 추가할 수 있다.

### 사용사례

아래 사용사례 항목별 10점씩 100점, geth로 하면 가점 10점이 배점된다.

#### 1-1 계정 및 잔고 출력
계정은 3개 미리 만들어져 있어야 한다. 계정1, 계정2, 계정3과 각 잔고를 출력한다. 현재의 블록번호도 출력한다.

#### 1-2 ```Order``` 컨트랙 생성에 필요한 gas 출력
```Order``` 컨트랙은 bytecode가 많아 배포에 gas가 적지 않게 지불해야 하는데, 이를 산정해서 출력한다. 

#### 1-3 고객정보 3건 입력
* 계정1을 사용하여 고객정보 입력 -> 111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul"
* 계정2를 사용하여 고객정보 입력 -> 112, "lee", "010-2017-1112", "112 hongji-dong jongro-gu seoul"
* 계정3을 사용하여 고객정보 입력 -> 113, "lim", "010-2017-1113", "113 hongji-dong jongro-gu seoul"
(* 중복된 ID, 예를 들어 111, "kim", "010-2017-1111", "7 hongji-dong jongro-gu seoul"는 입력이 불가능하고, 오류 발생해야 한다.
또한 중복ID를 허용하지 않으므로 테스트하면서 고객정보를 여러 번 입력하려면 오류가 발생하게 된다. 이 경우 팁은 배포를 새로 하고, 고객정보를 입력하면 중복ID가 해소되어 가능하다는 점에 주의하자.)
* 아래 ```tx.origin```와 ```msg.sender``` 관련 주의를 참조해서 ```msg.sender``` 대신 ```tx.orgin```을 사용하자.

#### 1-4 모든 고객의 배송지 출력
* 계정1을 사용하여 고객배송지 출력, "111 hongji-dong jongro-gu seoul"
* 계정2를사용하여 고객배송지 출력, "112 hongji-dong jongro-gu seoul"
* 계정3을 사용하여 고객배송지 출력, "113 hongji-dong jongro-gu seoul"
* 배송지 주소를 출력할 때, 다른 계정을 사용한다면, ```call({from: accounts[1]})```과 같이 계정주소를 적어주고 호출한다.

#### 1-5 주문
* 주문하면서, 주문금액을 전송해야 하고, 주문금액의 1% 마일리지를 주문자에게 부여
* 주문하면서, 주문상태를 "Ordered"로 설정하고 시간은 현재시간을 넣는다.
* 계정1에서 주문내역 입력 -> 555, "T-Shirt", 2, 1115
* 계정2에서 주문내역 입력 -> 556, "T-Shirt", 3, 1116
* 계정3에서 주문내역 입력 -> 557, "T-Shirt", 4, 1117

#### 1-6 주문개수, 주문금액 합계 및 잔고 출력
* 주문입력을 잘 했으면, 주문개수 3건, 주문금액 합계 3348, 컨트랙잔고를 출력한다.
* 잘 풀었으면 컨트랙잔고는 주문총액과 동일하다.
* 한글로 출력하지 않아도 된다.

#### 1-7 모든 고객의 주문 내역 출력
* 계정1에서 주문내역 출력 -> 555, "T-Shirt", "Ordered", "111 hongji-dong jongro-gu seoul"
* 계정2에서 주문내역 출력 -> 556, "T-Shirt", "Ordered", "112 hongji-dong jongro-gu seoul"
* 계정3에서 주문내역 출력 -> 557, "T-Shirt", "Ordered", "113 hongji-dong jongro-gu seoul"

#### 1-8 주문ID 556으로 주문내역 출력
* 주문ID 556으로 주문내역 출력 -> 556, "T-Shirt", "Ordered", "112 hongji-dong jongro-gu seoul"
* 계정을 입력해서 출력하지 않는다.

#### 1-9 주문ID 556으로 환불처리
* 주문ID 556으로 환불처리, 단 주문ID 556을 생성한 동일한 주문ID로 해야 처리된다.
* 환불 후, 상품ID에 대해 주문상태를 "refunded"로, 주문금액도 0으로 설정하고 그에 따라 컨트랙 잔고도 감해준다.
* 계정2 잔고변동 출력. 주문ID 556은 계정2에서 발생했다. 계정2는 환불을 받았으니 1116이 환입되어 변동이 없어야 한다 (gas 비용 증감은 있을 수 있다.)

#### 1-10 환불처리가 반영된 출력
* 주문ID 556으로 주문내역 출력 -> 556, "T-Shirt", 0, "refunded", "112 hongji-dong jongro-gu seoul"
* 잘 풀었으면, 주문개수 2건, 주문금액 2232, 컨트랙 잔고 출력 (주문금액 합계와 동일한 잔고)

### 주의
거래가 호출자U1 -> 컨트랙C1 -> 컨트랙C2의 순서대로 완성이 될 경우:
* C2에서 ```msg.sender```는 바로 직전의 호출자C1을 말한다.
* 반면에 ```tx.origin```은 U1의 주소를 말한다.

아래 프로그램을 보자. 주소 0x69e9a...0c102에서 ```Order```의 ```getTxOriginMsgSender()``` 함수를 호출하면:
* ```Customer```의 ```tx.origin``` 은 0x69e9a...0c102 (```Order``` 컨트랙 함수호출자인 ```msg.sender```의 주소와 동일)
* ```Customer```의 ```msg.sender```는 0x0b878...E7cD4, 즉 ```Order``` 컨트랙 배포주소와 동일하다.

```python
pragma solidity ^0.6.0;

contract Customer {
    function getTxOriginMsgSender() view public returns(address, address) {
        return(tx.origin, msg.sender);
    }
}

contract Order {
    Customer c;
    constructor() public {
        c = new Customer();
    }
    function getTxOriginMsgSender() view public returns(address, address) {
        return c.getTxOriginMsgSender();
    }
}
```

### 단계 1: 컨트랙 개발

- 마일리지 차감 아직 안넣었슴.
- 마일리지 1%는 `곱하기 0.01`은 소숫점이라 안되고, `나누기 100`을 구해서 더하는 방식으로 풀어야 한다.


```python
[파일명: src/CustomerOrder.sol]
pragma solidity ^0.6.0;

contract Customer {
    struct CustomerDetails {
        uint id;
        string name;
        string phNumber;
        string homeAddress;
        uint mileage;
        bool isCustomer; // check if exists
    }
    mapping(address=>CustomerDetails) customers;
    function addMileage(uint amount) public {
        require(customers[tx.origin].isCustomer == true); //only if the customer exists
        require(amount > 0);
        uint _mileage = uint(amount / 100);
        customers[tx.origin].mileage += _mileage; //1 percent mileage
    }
    //function subtractMileage(uint amount)
    function addCustomer(uint _id, string memory _name, string memory _ph, string memory _home) public {
        require(customers[tx.origin].isCustomer == false); //only if the customer does not exist
        customers[tx.origin].id = _id;
        customers[tx.origin].name = _name;
        customers[tx.origin].phNumber = _ph;
        customers[tx.origin].homeAddress = _home;
        customers[tx.origin].mileage = 0;
        customers[tx.origin].isCustomer = true;
    }
    function getHomeAddress() view public returns(string memory) {
        return customers[tx.origin].homeAddress;
    }
    function getMileage() view public returns(uint) {
        return customers[tx.origin].mileage;
    }
    function getId() view public returns(uint) {
        return customers[tx.origin].id;
    }
    function isCustomer() view public returns(bool) {
        return customers[tx.origin].isCustomer;
    }
    function getTxOriginMsgSender() view public returns(address, address) {
        return(tx.origin, msg.sender);
    }
}

contract Order {
    struct OrderItem {
        uint id;
        string product;
        uint n; //number of items
        uint amount;
        uint time;
        string status;
        string deliveryTo;
        bool isOrdered; // check if exists
    }
    address manager;
    uint nOrder;    //track number of orderItems (to avoid for loop)
    uint totalOrderAmount; //track number of orderItems (to avoid for loop)
    //bidrectional map
    mapping(address=>OrderItem) orderItems;
    mapping(uint=>address) addressById;
    Customer c;
    constructor() public {
        manager=msg.sender;
        c = new Customer(); //c = Customer(0xB57ee...89dF30); 
    }
    function addCustomer(uint _id, string memory _name, string memory _ph, string memory _home) public {
        c.addCustomer(_id, _name, _ph, _home);
    }
    function getTxOriginMsgSender() view public returns(address, address) {
        return c.getTxOriginMsgSender();
    }
    function getHomeAddress() view public returns(string memory) {
        return c.getHomeAddress();
    }
    function placeOrder (uint _id, string memory _p, uint _n, uint _amount) public payable isCustomer {
        require(msg.value == _amount);
        //customer.id is not added in as msg.sender is used as key to find the customer
        OrderItem memory o=OrderItem(_id, _p, _n, _amount, now, "Ordered", getHomeAddress(), true);
        orderItems[msg.sender]=o;
        //addMileage
        c.addMileage(_amount);
        //saving into a bidiretional map to get address by id
        addressById[_id]=msg.sender;
        nOrder++;
        totalOrderAmount+=_amount;
    }
    /*function isHomeDelivery(bool _home, string memory _deliveryTo) public {
        string memeory deliveryTo;
        if(_home == true) {
            deliveryTo = c.getHomeAddress();
        } else {
            deliveryTo = _deliveryTo;
        }
        return deliveryTo;
    }*/
    //only manager allowed
    function updateStatus(uint _id, string memory _s) public isOwner returns(string memory) {
        orderItems[addressById[_id]].status=_s;
        return (orderItems[msg.sender].status);
    }
    function getStatus() view public returns(string memory) {
        return orderItems[msg.sender].status;
    }
    function refund(uint _id) public payable isCustomer isOrdered{
        uint _amountToRefund = orderItems[msg.sender].amount;
        msg.sender.transfer(_amountToRefund);
        orderItems[msg.sender].amount -= _amountToRefund; // amount nullify
        //subtractMileage
        updateStatus(_id, "refund");
        nOrder--;
        totalOrderAmount -= _amountToRefund;
    }
    function getOrderById(uint _id) public view returns(uint, string memory, uint, string memory, string memory) {
        OrderItem memory o = orderItems[addressById[_id]];
        return (o.id, o.product, o.amount, o.status, o.deliveryTo);
    }
    function getOrderItem() public view returns(uint, string memory, uint, string memory, string memory) {
        OrderItem memory o = orderItems[msg.sender];
        return (o.id, o.product, o.amount, o.status, o.deliveryTo);
    }
    function getNOrder() view public returns(uint) {
        return nOrder;
    }
    function getTotalOrderAmount() view public returns(uint) {
        return totalOrderAmount;
    }
    function queryBalance() view public returns(uint) {
        return address(this).balance;
    }
    modifier isOwner() {
        require(msg.sender == manager);
        _;
    }
    modifier isCustomer() {
        require(c.isCustomer() == true);
        _;
    }
    modifier isOrdered() {
        require(orderItems[msg.sender].isOrdered == true);
        _;
    }
}
```


### 단계 2: 컴파일


```python
pjt_dir> solc src/CustomerOrder.sol --combined-json abi > src/CustomerOrderABI.json
```


```python
pjt_dir> solc src/CustomerOrder.sol --combined-json bin > src/CustomerOrderBIN.json
```

### 단계 3: 컨트랙 배포

배포하는 Gas가 3백만 가까이 계산되었다.


```python
[파일명: src/CustomerOrderDeploy.js]
var Web3 = require('web3');
var _abiJson = require('./CustomerOrderABI.json');
var _binJson = require('./CustomerOrderBIN.json');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractNames=Object.keys(_abiJson.contracts); // Customer, Order
contractName=contractNames[1]; // contractNames[0] -> 'src/CustomerOrder.sol:Customer', contractNames[1] -> Order
console.log("- contract name: ", contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_bin=_binJson.contracts[contractName].bin;  //ok without "0x"

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    
    new web3.eth.Contract(_abiArray).deploy({data: _bin}).estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);
    });

    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 3000000}, function(err, transactionHash) {
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
pjt_dir> node src/CustomerOrderDeploy.js


    - contract name:  src/CustomerOrder.sol:Order
    Deploying the contract from 0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1
    >> gas: 2832346
    hash: 0xc4f3f11b6908ddb0fa3685be0527872ca9aa241903d1ccb230bdc903cc1f10c2
    ---> The contract deployed to: 0xc4404cC5E73462415BFba4592195b8Dde21384c5
```

### 단계 4: 사용


```python
[파일명: src/CustomerOrderUse.js]
var Web3=require('web3');
var _abiJson = require('./CustomerOrderABI.json');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //ok
contractNames=Object.keys(_abiJson.contracts); // Customer, Order
contractName=contractNames[1]; // contractNames[0] -> 'src/CustomerOrder.sol:Customer', contractNames[1] -> Order
console.log("- contract name: ", contractName); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!

async function doIt() {
    var ord = new web3.eth.Contract(_abiArray, "0xc4404cC5E73462415BFba4592195b8Dde21384c5");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    
    ord.methods.addCustomer(111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul")
        .estimateGas(function(err, gas) {
        if(!err) console.log(">> gas: "+ gas);
    });

    await ord.methods
        .addCustomer(111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul")
        .send({from: accounts[0], gas: 140000});
    await ord.methods
        .addCustomer(112, "lee", "010-2017-1112", "112 hongji-dong jongro-gu seoul")
        .send({from: accounts[1], gas: 140000});
    await ord.methods
        .addCustomer(113, "lim", "010-2017-1113", "113 hongji-dong jongro-gu seoul")
        .send({from: accounts[2], gas: 140000});
    //await ord.methods
    //    .addCustomer(111, "kim", "010-2017-1111", "111 hongji-dong jongro-gu seoul")
    //    .send({from: accounts[0], gas: 140000});
    ord.methods.getHomeAddress().call().then(console.log);

    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    
}

doIt()

```

```python
pjt_dir> node src/CustomerOrderUse.js


    - contract name:  src/CustomerOrder.sol:Order
    Account: 0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1
    Balance before: 99865294299999997778
    >> gas: 134584
    111 hongji-dong jongro-gu seoul
    Balance after: 99862602619999997778
    Balance diff: 2691680000000000
```


```python
[파일명: src/CustomerOrderUse2.js]
var Web3=require('web3');
var _abiJson = require('./CustomerOrderABI.json');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //ok
contractNames=Object.keys(_abiJson.contracts); // Customer, Order
contractName=contractNames[1]; // contractNames[0] -> 'src/CustomerOrder.sol:Customer', contractNames[1] -> Order
console.log("- contract name: ", contractName); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!

async function doIt() {
    var ord = new web3.eth.Contract(_abiArray, "0xc4404cC5E73462415BFba4592195b8Dde21384c5");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);

    ord.methods.getHomeAddress().call({from: accounts[0]}).then(console.log);
    ord.methods.getHomeAddress().call({from: accounts[1]}).then(console.log);
    ord.methods.getHomeAddress().call({from: accounts[2]}).then(console.log);

    // without value filed??
    //ord.methods.placeOrder(555, "T-Shirt", 2, 1115)
    //    .estimateGas(function(err, gas) {
    //    if(!err) console.log(">> gas: "+ gas);
    //});
    
    //set enough gas
    await ord.methods.placeOrder(555, "T-Shirt", 2, 1115).send({from: accounts[0], gas: 300000, value:1115});
    await ord.methods.placeOrder(556, "T-Shirt", 2, 1116).send({from: accounts[1], gas: 300000, value:1116});
    await ord.methods.placeOrder(557, "T-Shirt", 2, 1117).send({from: accounts[2], gas: 300000, value:1117});

    ord.methods.getNOrder().call().then(console.log);
    ord.methods.getTotalOrderAmount().call().then(console.log);
    //ord.methods.queryBalance().call().then(console.log);    //error! why?
    ord.methods.getOrderItem().call({from: accounts[0]}).then(console.log);
    ord.methods.getOrderItem().call({from: accounts[1]}).then(console.log);
    ord.methods.getOrderItem().call({from: accounts[2]}).then(console.log);
    
    ord.methods.getOrderById(555).call().then(console.log);
    //await ord.methods.refund(556).send({from: accounts[1], gas: 300000}); //call from the same account. why error????
    ord.methods.getOrderById(556).call().then(console.log);
    ord.methods.getNOrder().call().then(console.log);
    ord.methods.getTotalOrderAmount().call().then(console.log);    

    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));  
}

doIt()

```


```python
pjt_dir> node src/CustomerOrderUse2.js


    - contract name:  src/CustomerOrder.sol:Order
    Account: 0x42E5468FC83A0c5F6D4d2E1E632bd8864Dd87bd1
    Balance before: 99862143899999996663
    111 hongji-dong jongro-gu seoul
    112 hongji-dong jongro-gu seoul
    113 hongji-dong jongro-gu seoul
    3
    3348
    Result {
      '0': '555',
      '1': 'T-Shirt',
      '2': '1115',
      '3': 'Ordered',
      '4': '111 hongji-dong jongro-gu seoul' }
    Result {
      '0': '556',
      '1': 'T-Shirt',
      '2': '1116',
      '3': 'Ordered',
      '4': '112 hongji-dong jongro-gu seoul' }
    Result {
      '0': '557',
      '1': 'T-Shirt',
      '2': '1117',
      '3': 'Ordered',
      '4': '113 hongji-dong jongro-gu seoul' }
    Result {
      '0': '555',
      '1': 'T-Shirt',
      '2': '1115',
      '3': 'Ordered',
      '4': '111 hongji-dong jongro-gu seoul' }
    Result {
      '0': '556',
      '1': 'T-Shirt',
      '2': '1116',
      '3': 'Ordered',
      '4': '112 hongji-dong jongro-gu seoul' }
    3
    3348
    Balance after: 99856617999999995548
    Balance diff: 5525900000002048
```

## 연습문제


```python
[파일명: src/Order.sol]
pragma solidity ^0.5.0;

contract Order {
    struct OrderItem {
        uint id;
        string product;
        uint n; //number of items
        uint amount;
        uint time;
        string status;
        bool isOrdered; // check if exists
    }
    address manager;
    uint nOrder;    //track number of orderItems (to avoid for loop)
    uint totalOrderAmount; //track number of orderItems (to avoid for loop)
    //bidrectional map
    mapping(address=>OrderItem) orderItems;
    mapping(uint=>address) addressById;
    constructor() public {
        manager=msg.sender;
    }
    function placeOrder (uint _id, string memory _p, uint _n, uint _amount) public payable {
        require(msg.value==_amount);
        OrderItem memory o=OrderItem(_id, _p, _n, _amount, now, "Ordered", true);
        orderItems[msg.sender]=o;
        //saving into a bidiretional map to get address by id
        addressById[_id]=msg.sender;
        nOrder++;
        totalOrderAmount+=_amount;
    }
    //only manager allowed
    function updateStatus(uint _id, string memory _s) public isOwner returns(string memory) {
        orderItems[addressById[_id]].status=_s;
        return (orderItems[msg.sender].status);
    }
    function getStatus() view public returns(string memory) {
        return orderItems[msg.sender].status;
    }
    function getOrderById(uint _id) public view returns(uint,string memory,string memory) {
        OrderItem memory o=orderItems[addressById[_id]];
        return (o.id,o.product,o.status);
    }
    function getOrderItem() view public returns(uint,string memory,string memory) {
        OrderItem memory o=orderItems[msg.sender];
        return (o.id,o.product,o.status);
    }
    function getNOrder() view public returns(uint) {
        return nOrder;
    }
    function getTotalOrderAmount() view public returns(uint) {
        return totalOrderAmount;
    }
    function queryBalance() view public returns(uint) {
        return address(this).balance;
    }
    modifier isOwner() {
        require(msg.sender == manager);
        _;
    }
}
```

## 연습문제

A, B 2인의 가위바위보 후 내기금액을 이긴 사람에게 지급하는 게임을 블록체인에 개발하세요.
게임 컨트랙은 Rsp, 구현할 함수는 다음과 같다. 그 외 필요로 하는 함수는 추가할 수 있다.

함수 | 설명
-----|-----
setA | 직접 입력하지 않고, 가위, 바위, 보를 무작위로 생성하여 내고 ```1000 wei```를 베팅한다.
setB | 게임 플레이어가 직접 입력하고, ```1000 wei```를 베팅한다.
play | setA, setB 입력이 끝나고 컴퓨터가 실행하는 것으로 하고, 승패가 결정짓는다.
distributeBetAmount | 승패에 따라 분배한다. 승자가 패자의 내기 금액 ```1000 wei```를 가지게 된다.
getMatchResult() | 누가 이겼는지 A, B 승자를 포함한 문자열 "A wins", "B wins", "tie" 결과를 출력한다.

노드에서 게임을 다음과 같이 진행하세요.
- 게임 전의 A잔고, B잔고, 컨트랙 잔고 출력 (컨트랙 상의 잔고를 말한다)
- setA() 실행. 플레이어 A는 컴퓨터가 대행하는 것으로 하고, 내기금액 걸고 가위바위보 중 하나를 무작위로 선택
- setB() 실행. B는 자신이 직접 내기금액 걸고, 가위바위보 중 하나를 선택하여 입력
- play() 실행해서, 승부를 결정
- getMatchResult() 실행해서, 승부의 결과를 출력
- 게임 끝나고 A잔고, B잔고, 컨트랙 잔고 출력 (컨트랙 상의 잔고를 말한다)



```python
[파일명: src/Rsp.sol]
pragma solidity ^0.5.0;

contract Rsp {
    uint8 betA; //1 scissor, 2 rock or 3 paper
    uint8 betB; //1 scissor, 2 rock or 3 paper
    string matchResult; //win or lose message
    bytes32 winnerAorB;  //bytes32 enables the '==' comparison 
    //string winnerAorB;  //A or B
    uint betAmount;     //smaller bet between A and B
    mapping(address => uint256) public balanceOf;
    mapping(string => address) playersAddr;
    //mapping(bytes32 => address) public playersAddr;
    
    event PrintLog(string);
    //event PrintLog(bytes32);
    function setA(uint8 n, uint amount) public payable {
        // ADD: 'n' is randomly generated to be either 1,2 or 3
        betA=n;
        betAmount=amount;
        depositBalance(betAmount);
        addPlayersAddr("A");
    }
    function setB(uint8 n, uint amount) public payable {
        betB=n;
        betAmount=amount;   // bets of A and B to be the same
        depositBalance(betAmount);
        addPlayersAddr("B");
    }
    function play() public {
        if(betA==betB) {
            matchResult="tie";
            winnerAorB="N";
        } else if(betA==1 && betB==2) {
            matchResult="B wins";
            winnerAorB="B";
        } else if(betA==1 && betB==3) {
            matchResult="A wins";
            winnerAorB="A";
        } else if(betA==2 && betB==1) {
            matchResult="A wins";
            winnerAorB="A";
        } else if(betA==2 && betB==3) {
            matchResult="B wins";
            winnerAorB="B";
        } else if(betA==3 && betB==1) {
            matchResult="B wins";
            winnerAorB="B";
        } else if(betA==3 && betB==2) {
            matchResult="A wins";
            winnerAorB="A";
        }
    }
    function getMatchResult() view public returns(string memory) {
        return matchResult;
    }
    function distributeBetAmount() public {
        if (winnerAorB=="A") {
            // deduct from the loser and give all to ther winner
            balanceOf[playersAddr["A"]]+=betAmount;
            balanceOf[playersAddr["B"]]-=betAmount;
        } else if (winnerAorB=="B") {
            // deduct from the loser and betAmountgive all to ther winner
            balanceOf[playersAddr["B"]]+=betAmount;
            balanceOf[playersAddr["A"]]-=betAmount;
        }
    }
    function queryBalance() view public returns(uint) {
        return address(this).balance;
    }
    function queryBalanceOf(string memory player) view public returns(uint) {
        //ADD: check a to be 'A' or 'B'
        return balanceOf[playersAddr[player]];
    }
    function depositBalance(uint amount) public payable {
        require(msg.value==amount);
        // ADD: check if msg.sender exists
        balanceOf[msg.sender]+=amount;        
    }
    function addPlayersAddr(string memory p) internal {
        // ADD: check if p exists
        playersAddr[p]=msg.sender;
    }
}
```

## 연습문제

Observer 패턴을 구현해보자.


```python
pragma solidity ^0.6.0;

contract Observer {
    event PrintLog(int _value, uint _time, bytes32 info);
    function update(int _value, uint _time, bytes32 info) public {
        emit PrintLog(_value, _time, info);
    }
}

contract Observable {
    Observer[] obs; // array of all subscribers

    // register subscribers
    function addObserver(Observer o) public {
        obs.push(o);
    }

    function notifyObservers(int value, uint time, bytes32 info) public {
        for(uint i = 0;i < obs.length; i++) {
            obs[i].update(value, time, info);
        }
    }
}
```


