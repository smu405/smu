#  Chapter 11. 컨트랙의 결합

컨트랙이 늘어날수록 당연히 결합이 필요하다. import문을 사용해서 가져오고, new 명령어를 통해 객체를 생성하게 된다. Solidity는 객체지향의 상속도 지원한다. 다른 언어에서 제공되는 라이브러리를 만들 수도 있다.

# 1. 컨트랙 간의 관련

## 1.1 관련의 구분

지금까지는 주로 하나의 컨트랙을 개발하였다. 컨트랙이 2개 이상이 되는 경우, 이들 간의 결합이 필요하다. Solidity는 객체지향언어이므로 ```is-a```, ```has-a``` 관계를 구현할 수 있다. 객체지향에서의 ```has-a```는 (1) 의존 dependency, (2) 연관 association, (3) 포함 aggregation, (4) 조립 composition로 구분하고 있다. 상속은 ```is-a``` 관계로서 곧 뒤 이어 설명하기로 한다.

어느 경우에 어떤 결합을 사용해야 하는지의 결정은 객체지향에 있어서는 성능에 영향이 있는만큼 중요하다. 사용하는 기한에 따라, 서로 집합의 관계인지, 사용하는 시점에 따라 결합하는 형태가 영향을 받는다.

반면에 컨트랙의 경우, 블록체인에 객체가 항상 상주하면서 어떤 관련을 적용하는지 덜 중요하다. 언제 관련이 맺고, 끊고 등의 메모리를 효율적으로 사용하려는 노력은 무의미할 수 있다. Solidity에서는 코드 재사용과 더불어 개스 비용의 관점에서 어떤 관련을 적용하게 되는 것인지의 선택이 합리적일 수 있다.

### 1.1.1 의존은 제한된 시간 동안의 has-a 관계이다.

의존관계 Dependency는 객체 간의 관계가 있지만, 그 관계가 오래가지 않고 한시적이다. 주문의 경우 고객정보를 매개변수로 받거나, 반환하는 것이 한 예가 된다. 그 관계가 함수 내에서는 제한적으로 존재하고, 함수 밖에서는 끊어진다.
```
Contract Order {
    function placeOrder(Customer c, Product p) //컨트랙이 함수의 인자로 쓰이고, 함수 내에서 제한적 관련
}
```

### 1.1.2 연관은 장기간의 has-a 관계이다.

연관 Association은 의존관계에서 보는 has-a의 관계이지만, 그 관계가 오래 지속된다는 점에서 차이가 있다. 객체 간 관계가 있고, 그 관계가 거의 항상 지속되는 경우이다. 주문의 예를 다시 들어보자. 고객과 주문, 배송, 문의, 결재 등 지속적으로 상호작용하는 경우에는 의존관계 보다는 연관관계가 적합하다. 고객과의 관계는 주문, 배송, 문의, 결재 등 여러 함수에서 필요하기 때문에 오랫동안 그 연관을 가져간다.
```
contract Order {
    Customer customer //멤버변수 자리의 컨트랙이고, 장기적 관계
    function placeOrder()
    function pay()
    function deliver()
}
```

### 1.1.3 포함은 전체와 부분의 관계이다.

포함은 Aggregation이라고 불리우고, has-a의 관계이면서 전체-부분로 구성된다. 장기적인 has-a관계라는 점에서 연관과 유사하지만, 전체-부분이라는 차이가 있다. 그러나 부분은 그 자체로 전체를 구성하지 않고 존속할 수 있다. 전체는 부분을 배열과 같은 구성으로 가지게 된다.
```
contract Course {
    Student[] students; //학생은 강의를 구성하는 부분, 그러나 그 자체로 존재할 수 있다.
}
```

### 1.1.4 조립은 전체와 부분이고 전체가 부분을 소유한다. 

영어로는 composition, 한글은 조립관계로 말하지만, 객체지향의 그 개념을 온전하게 나타내는 단어는 아니라고 생각한다. 포함 aggregation과 유사하지만, 전체-부분의 관계가 강하다. 부분은 전체와 생성-제거의 동일한 시간 범위를 가진다. 즉 전체가 부분을 생성하고, 소유하고 있어 전체가 부분을 해제하면 그 자체로 존재하지 못하는 것으로 이해하자.

```
contract Vehicle {
    Engine engine;
    constructor Vehicle() {
       engine = new Engine();  //생성될 때부터 전체의 부분이고, 자체 존재하지 못한다.
    }
}
```

## 1.2 상대측 컨트랙 객체의 생성

컨트랙을 결합할 경우, 그 대상 컨트랙이 (1) 배포되어 있지 않은 경우, (2) 배포되어 있는 경우로 나누어 구분할 필요가 있다. 배포되어 있지 않는 경우에는 단순하다. 반면에 배포되어 있는 컨트랙을 결합하려면 주소를 알아야 한다. 이와 같이 구분하여 그 객체를 어떻게 가져와 사용하는지 설명해 보자.

### 1.2.1 동일한 파일의 컨트랙과 결합

한 파일에 컨트랙의 소스코드가 포함되는 경우, ```new``` 명령어로 **인스턴스**를 만들어서 함수를 호출한다. 멤버변수를 선언하면서, 아예 객체를 생성해 놓을 수 있다.

```
contract Customer {
}
contract Order {
    Customer c = new Customer(); //객체를 처음부터 생성
    Order() public {
	}
}
```

또는 객체생성을 연기할 수 있다. 생성자 함수를 호출할 때 비로서 객체를 생성하므로 메모리가 나중에 할당이 된다. 생성자가 아닌 함수에서 호출하면 시점이 더욱 대조적으로 늦춰지는 효과가 있다.
```
contract Customer {
}
contract Order {
	Customer c;
    Order() public {
	    c = new Customer(); //생성자 함수를 호출할 때 비로서 객체를 생성
	}
}
```

### 1.2.2 다른 파일의 컨트랙과 결합

상대 컨트랙을 한 파일에 포함하지 않는 경우라면, ```import``` 문으로 그 컨트랙을 포함하여야 한다. ```import```문 다음에는 파일명을 적고 (컨트랙 명이 아니라), **현재 컨트랙을 기준으로 상대경로**를 적어준다.

```python
import <<filename>>
```

상대 컨트랙을 import하게 되면 코드가 주입되기 때문에, 그 컨트랙은 별도로 배포하지 않아도 된다.

### 1.2.3 이미 배포된 컨트랙과 결합

이미 배포된 컨트랙을 포함되는 경우, **주소**를 구해서 전달한다. new 명령어는 필요없다.
```
C1 c1 = C1(_addressOfC1) // 배포주소를 인자로 넣어서 결합한다
```

## 1.3. 함수의 호출

### 1.3.1 컨트랙에서 호출

상대측 객체를 만들고 나면, 함수를 호출한다. 함수는 객체지향에서 하는 방식으로 점연산자 dot operator를 사용하면 된다.
```python
<instance>.functionMethod()
```

### 1.3.2 ```web3.js```에서 호출

```web3.js```에서는 ```<instance>.methods.functionMethod()```라고 호출한다.

굳이 좀 더 자세히 설명하면, 앞서 ```5장의 ABI 명세```에서 설명하였던 ```function selector```를 사용하여 함수가 호출된다. ```function setCounter(uint n){ counter = n; }``` 함수를 컨트랙에서 호출해 보자. 함수명을 sha3 해싱한 후, 처음 4바이트로 함수를 호출하게 된다.

데이터가 없는 경우 (```calldata```가 없는 경우), ```receive``` 또는 ```fallback``` 함수가 호출된다.

힘수 호출 | 설명
-----|-----
<address>.call.gas(200000).value(msg.value)("") | fallback 함수 호출
<address>.call.foo{value: msg.value, gas: 5000}(bytes4(bytes32(sha3("setCounter(uint)")))) | ```setCounter``` 함수 호출. 우리는 이런 어려운 방식으로 호출하지 않는다.

다른 컨트랙을 호출하는 경우 gas 비용이 infinite라고 계산된다. 그 이유는 다른 컨트랙이 얼마나 gas를 사용하게 될지 모르기 때문이다. gas비용은 전송측에서 차감이 된다는 점에 주의하자.

## 실습: new 명령어로 컨트랙 조립 composition

```new()``` 명령어로 컨트랙을 생성하려면 컴파일 시점에 그 소스코드를 가져올 수 있어야 한다.

즉 **상대 컨트랙이 동일한 파일에 존재**하거나 또는 **```import``` 문으로 상대 컨트랙이 포함**되는 경우가 해당이 된다.
그러면 상대 컨트랙이 컴파일되어 바이트코드가 포함되게 된다.

### 단계 1: 컨트랙 개발

매우 단순한 컨트랙을 만들어 보자. 컨트랙 C1과 C2는 서로 강한 관계를 가지고 있다. C2 생성자에서 C1을 가지고 있다. 또는 set() 함수를 만들어 생성 후에 필요한 시점에 C1과의 연관을 만들고 있다.

Solidity 결합을 할 경우, 버전의 문제가 있을 수 있다. 일부 0.5버전에서 컴파일은 문제가 없었으나, 실행하면 일부 기능이 적절하게 수행되지 않는 경험을 했다. 일부 개발자들 사이에서 0.4.21이하에서는 문제가 없으나 그 이후에는 문제가 있다는 지적이 있다 (2018년 작성된 글 https://github.com/ethereum/solidity/issues/3969).

```python
[파일명: src/C1C2.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6;
//pragma solidity 0.4.21;

contract C1 {
    uint128 v1;
    function set(uint128 _v1) public {
        v1=_v1;
    }
    function get() public view returns(uint128) {
        return v1;
    }
    function get7() public pure returns(uint128) {
        return 7;
    }
}

contract C2 {
    C1 c1;
    //function C2() public {  //0.4.21 constructor
    constructor() { //constructor() public {    //0.6 constructor
        c1=new C1();     //C2에서 C1의 객체를 생성, 저장
    }
    function set(uint128 _v1) public {
        c1.set(_v1);
    }
    function get() public view returns(uint128) {
        return c1.get();
    }
    function get7() public view returns(uint128) {
        return c1.get7();
    }
    function getC1Address() public view returns(address) {
        return address(c1);   //C2에서 C1의 주소를 읽는다
    }
}
```

### 단계 2: 컴파일

다음과 같이 컴파일할 수 있다. 파일에 컨트랙이 2개 있으면, 컴파일한 ABI, bin도 2개가 생성된다. 로컬에서 설치된 solc보다 낮은 버전으로 컴파일 하려면 REMIX에서 버전을 낮추어 컴파일하고 ABI, bin을 가져온다.

```python
pjt_dir> solc-windows.exe src/C1C2.sol --combined-json abi,bin > src/C1C2.json
```

### 단계 3: 배포

컴파일 하면 C1, C2 모두 abi, bin이 생성된다 (앞서 생성된 ```src/C1C2.json``` 파일을 열어 확인하고 진행하는 편이 좋다). 그 중 C2의 abi, bin을 가져와서 배포를 한다.

여기서는 자바스크립트의 파일 ```fs``` 라이브러리에서 제공하는 ```fs.readFileSync()``` 함수를 사용해서 JSON파일을 읽어온다.

```python
[파일명: src/C1C2Deploy.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C1C2.json");
var _json=JSON.parse(_str)
//var _abiArray=JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray=JSON.parse(_json.contracts["src/C1C2.sol:C2"].abi);
var _abiArray=_json.contracts["src/C1C2.sol:C2"].abi;
//var _bin=_json.contracts.sHello2.bin;
var _bin="0x"+_json.contracts["src/C1C2.sol:C2"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000})
        .on('transactionHash', function(hash){
            console.log(">>> transactionHash"+hash);
        })
        .on('receipt', function(receipt){
            console.log(">>> RECEPIT hash: " + receipt.transactionHash + "\n>>> address:" + receipt.contractAddress);
        })
        .on('error', function(error, receipt) {
            console.log(">>> ERROR "+error);
        });
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

배포하고 나면, hash 및 컨트랙의 주소가 생성되고 있다.
```python
pjt_dir> node src/C1C2Deploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
>>> transactionHash0x8475ca0bb427e0acf37d9213a85c88eef795701788d9f327b8835d6ac8c1ac03
>>> RECEPIT hash: 0x8475ca0bb427e0acf37d9213a85c88eef795701788d9f327b8835d6ac8c1ac03
>>> address:0xE0b20Cf5684a8Aa9Bac4F7c1A186237Abe18026B
---> The contract deployed to: 0xE0b20Cf5684a8Aa9Bac4F7c1A186237Abe18026B
```

### 단계 4: 사용

위에서 생성된 abi와 컨트랙 주소를 넣어 프로그램을 작성한다.
 
```python
[파일명: src/C1C2Use.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C1C2.json");
var _json=JSON.parse(_str)
//var _abiArray=JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray=JSON.parse(_json.contracts["src/C1C2.sol:C2"].abi);
var _abiArray=_json.contracts["src/C1C2.sol:C2"].abi;

var c2 = new web3.eth.Contract(_abiArray, "0xE0b20Cf5684a8Aa9Bac4F7c1A186237Abe18026B");

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    c2.methods.get7().call().then(function(res) {console.log("C1 get7(): " + res)});
    await c2.methods.set(9).send({from: accounts[0],gas:50000}, function(err,res) {
        console.log("setting 9..."+res);
    });
    c2.methods.get().call().then(function(res) {console.log("C1 get(): " + res)});
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    //hello.methods.kill().send({from: accounts[0]})
}

doIt()
```


```python
pjt_dir> node src/C1C2Use.js

    Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    Balance before: 999939404231999987456
    C1 get7(): 7
    setting 9...0x8866ede6c1a1bd88e937f8e70b5a1e9fcf19da5629cbda9fe946fef1fcc457c8
    Balance after: 999939345375999987456
    Balance diff: 58856000061440
    C1 get(): 9
```

## 실습: 컨트랙의 주소를 사용하여 컨트랙 결합 association

앞서 ```new()``` 명령어는 소스코드를 포함할 수 있는 경우에 사용하였다. 이번에는 이미 배포된 컨트랙을 결합하여 보자. 그렇다면 C2에 C1의 주소를 넘겨주어야 한다. 즉 C1을 배포하고 그 주소를 알야야 한다.

### C1을 배포하고 주소를 구하기

* 컨트랙 개발 (C1)

```python
[파일명: src/C1.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6;
//pragma solidity 0.4.21;

contract C1 {
    uint128 v1;
    function set(uint128 _v1) public {
        v1=_v1;
    }
    function get() public view returns(uint128) {
        return v1;
    }
    function get7() public pure returns(uint128) {
        return 7;
    }
}
```


* 컴파일 (C1)

참고로 다른 버전으로 컴파일하려면, REMIX에서 0.4.21, 0.6이든 어떤 버전이든 선택하여 컴파일한다. 거의 사용하지 않겠지만, 참고로 특정 evm에서, 예를 들어 "homestead"로 컴파일 하려면 (byzantium이 기본) ```solc --evm-version "homestead" --gas C1.sol```로 적어준다.

```python
pjt_dir> solc-windows.exe src/C1.sol --combined-json abi,bin > src/C1.json
```

* 배포 (C1)

```python
[파일명: src/C1Deploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C1.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/C1.sol:C1"].abi);
var _abiArray = _json.contracts["src/C1.sol:C1"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x"+_json.contracts["src/C1.sol:C1"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000})
        .on('transactionHash', function(hash){
            console.log(">>> transactionHash" + hash);
        })
        .on('receipt', function(receipt){
            console.log(">>> RECEPIT hash: " + receipt.transactionHash + "\n>>> address:" + receipt.contractAddress);
        })
        .on('error', function(error, receipt) {
            console.log(">>> ERROR " + error);
        });
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

C1Deploy.js를 일괄 실행해보자.

```python
pjt_dir> node src/C1Deploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
>>> transactionHash0x1e68c99b948f52b7940f4ff02c4e62b0e60ddb5fadecebe07975f55ae6da9bcd
>>> RECEPIT hash: 0x1e68c99b948f52b7940f4ff02c4e62b0e60ddb5fadecebe07975f55ae6da9bcd
>>> address:0xc84381615C183B74ba3cd0cb9Ae9455d5236d586
---> The contract deployed to: 0xc84381615C183B74ba3cd0cb9Ae9455d5236d586
```

* 사용 (C1)

위 주소를 얻어서 복사해 넣고 아래를 실행하면 결과를 얻을 수 있다.

```python
[파일명: src/C1Use.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C1.json");
var _json=JSON.parse(_str)
//var _abiArray=JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray=JSON.parse(_json.contracts["src/C1.sol:C1"].abi);
var _abiArray = _json.contracts["src/C1.sol:C1"].abi;

var c1 = new web3.eth.Contract(_abiArray, "0xc84381615C183B74ba3cd0cb9Ae9455d5236d586");
async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    c1.methods.get7().call().then(console.log);
    await c1.methods.set(9).send({from: accounts[0],gas:50000});
    c1.methods.get().call().then(console.log);
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    //hello.methods.kill().send({from: accounts[0]})
}

doIt()
```

```python
pjt_dir> node src/C1Use.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999939035101999987456
7
Balance after: 999938947535999987456
Balance diff: 87565999931392
9
```

### 단계 1: 컨트랙 개발 (C2)

자, 이제 C2에서 C1의 주소를 사용하여 결합하는 단계이다.

```C1```과 ```C2```를 결합하는 소스코드를 구현해보자.
```C1```은 이미 배포가 되었고, 그 주소를 받을 수 있는 기능이 필요하다.
이 경우 ```C1```의 ABI를 모르면 ```C2```를 컴파일을 할 수 없다.
ABI는 함수의 호출방식을 정의하고 있어서 예를 들어 ```C1```의 함수 ```c1.get7()```의 ABI를 모르면, C2의 ```get7()```을 컴파일할 수 없게 된다.

앞서 ```C1.sol```, ```C2.sol```을 한 파일 안에 적어주지 않고,
```import```문으로 ```C1```을 포함한다.
이 경우 ```import C1.sol```이라고 해주지 않고, 현재 파일의 상대 디렉토리 ```import "./C1.sol"```로 적어준다.

구분 | 사용 예 | 이유
-----|-----|-----
올바른 사용 | ```import "./C1.sol"``` | 상대 디렉토리로 적어주면 ```C1.sol```을 못 찾는다.
올바르지 않은 사용 | ```import "C1.sol"``` | 상대 디렉토리가 아니라서 오류가 발생

```python
[파일명: src/C2.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity 0.6;
//pragma solidity 0.4.21;
import "./C1.sol";

contract C2 {
    C1 c1;
    //function C2() public {  //0.4.21 constructor
    constructor() {    //constructor() public {    //0.6 constructor
        c1=new C1();
    }
    function setC1(address _addressOfC1) public {
        c1 = C1(_addressOfC1);
    }
    function set(uint128 _v1) public {
        c1.set(_v1);
    }
    function get() public view returns(uint128) {
        return c1.get();
    }
    function get7() public view returns(uint128) {
        return c1.get7();
    }
    function getC1Address() public view returns(address) {
        return address(c1);
    }
}
```

### 단계 2: 컴파일 (C2)

C2를 컴파일한다. C2가 C1을 사용하고 있고, C1의 ABI, Bytecode는 포함된다.

```python
pjt_dir> solc-windows.exe src/C2.sol --combined-json abi,bin > src/C2.json
```

### 단계 3: 배포

```python
[파일명: src/C2Deploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C2.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/C2.sol:C2"].abi);
var _abiArray = _json.contracts["src/C1.sol:C1"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x"+_json.contracts["src/C2.sol:C2"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000})
        .on('transactionHash', function(hash){
            console.log(">>> transactionHash" + hash);
        })
        .on('receipt', function(receipt){
            console.log(">>> RECEPIT hash: " + receipt.transactionHash + "\n>>> address:" + receipt.contractAddress);
        })
        .on('error', function(error, receipt) {
            console.log(">>> ERROR " + error);
        });
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

배포 단계에서는 앞의 C1을 지정하지 않고 있다. 실행하면 컨트랙 주소 등을 출력하고 있다.
```python
pjt_dir> node src/C2Deploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
>>> transactionHash0xe6be9e256fa1f82f7aa305b05579662d916abd0b9223ce1c6b9ff991852cf5c3
>>> RECEPIT hash: 0xe6be9e256fa1f82f7aa305b05579662d916abd0b9223ce1c6b9ff991852cf5c3
>>> address:0x3B7A1c3e7C223eDd963eE045DaCf4A6860164Cf8
---> The contract deployed to: 0x3B7A1c3e7C223eDd963eE045DaCf4A6860164Cf8
```

### 단계 4: 사용

자, 먼 길을 돌아 이제 C2의 API를 호출해보자. 기억하자! C1의 주소를 설정하고, 확인하기 위해 이렇게 돌아오고 있다. 코드를 설명하고 있으니, ```setC1()```, ```getC1Address()```를 잘 살펴보자.

줄 | 함수 | 설명
-----|-----|-----
16 | ```c2.methods.getC1Address().call()``` | 생성자에서 ```new``` 명령어로 생성된 ```C1```의 주소를 출력. 단, ```C2```를 배포하고 첫 회 실행할 때만 유효하고, 2회부터는 이전에 실행된 ```setC1()```의 결과인 이전 ```C1```의 주소가 출력된다.
19 | ```c2.methods.get7().call()``` | ```await```로 하지 않으면 실행 순서가 늦춰질 수 있다.
21 | c2.methods.setC1() | 생성자에서 설정한 C1을 제거하고, 위에서 블록체인에 배포한 C1의 주소를 사용하여 교체한다.

```python
[파일명: src/C2Use.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/C2.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/C2.sol:C2"].abi);
var _abiArray = _json.contracts["src/C2.sol:C2"].abi;

var c2 = new web3.eth.Contract(_abiArray, "0x3B7A1c3e7C223eDd963eE045DaCf4A6860164Cf8");
async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    console.log("--- new C1 ---");
    await c2.methods.getC1Address().call(function(err, c1addr) {
        if(!err) console.log("c1 address by 'new': "+c1addr);
    });
    c2.methods.get7().call().then(function(res) { console.log("get7(): "+res) });
    console.log("--- set the above deployed address of C1 ---");
    await c2.methods.setC1("0xc84381615C183B74ba3cd0cb9Ae9455d5236d586").send({from:accounts[0], gas:50000});
    await c2.methods.getC1Address().call(function(err, c1addr) {
        if(!err) console.log("c1 address by 'setC1()': "+c1addr);
    });
    c2.methods.get7().call().then(console.log);
    await c2.methods.set(222).send({from: accounts[0],gas:50000});
    c2.methods.get().call().then(console.log);
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    //hello.methods.kill().send({from: accounts[0]})
}

doIt()
```

new명령어를 사용하거나, 주소를 사용해서 설정을 하거나 원하는 기능이 올바르게 수행되고 있다.


```python
pjt_dir> node src/C2Use.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999937537163999987456
--- new C1 ---
c1 address by 'new': 0xc84381615C183B74ba3cd0cb9Ae9455d5236d586
--- set the above deployed address of C1 ---
get7(): 7
c1 address by 'setC1()': 0xc84381615C183B74ba3cd0cb9Ae9455d5236d586
7
Balance after: 999937429911999987456
Balance diff: 107251999965184
222
```

편의상 하나의 파일에서 모두 다 하고 있어, 그냥 재호출하기로 한다.

```python
pjt_dir> node src/C2Use.js

Account: 0xAD4c0912D2562b7072780A2F0FB2749D749B14fB
Balance before: 99945866560000000000
--- new C1 ---
c1 address by 'new': 0x6BA98F59D5E3dDd9a3A38727e04dFDC4C4bE9D16
7
--- set the above deployed address of C1 ---
c1 address by 'setC1()': 0xc71cC556bf5C9c4694062BF4752768F65d9349Dc
7
222
Balance after: 99944817720000000000
Balance diff: 1048839999995904
```

2회를 실행하면 아래에서 보는 것처럼 C1의 주소가 같아진다.
2회부터는 이전에 실행된 ```setC1()```의 결과인 이전 ```C1```의 주소로 설정되어 출력된다.
생성자가 최초에만 호출되기 때문이다.

다시 생성자를 호출하여 ```C1```의 주소를 설정하려면, C2를 다시 배포한 후 그 주소로 ```C2Use.js```를 실행하면 된다.

```python
pjt_dir> node src/C2Use.js

Account: 0xAD4c0912D2562b7072780A2F0FB2749D749B14fB
Balance before: 99944817720000000000
--- new C1 ---
c1 address by 'new': 0xc71cC556bf5C9c4694062BF4752768F65d9349Dc
7
--- set the above deployed address of C1 ---
c1 address by 'setC1()': 0xc71cC556bf5C9c4694062BF4752768F65d9349Dc
7
222
Balance after: 99943864880000000000
Balance diff: 952839999995904
```

## 실습: 자동차와 엔진의 조립- new 명령어 사용

동일한 파일에 2개의 컨트랙을 넣어서 개발해 보자.

### 단계 1: 컨트랙 개발

Line | 설명
-----|-----
1 | major version 6 이상 최신으로 컴파일
2 | contract 명. 컨트랙이 2개 포함된 경우, 파일명과 반드시 일치할 필요가 없다.
4 | ```Engine```을 포함하고, ```constructor```에서 할당한다.
26 ~ 31 | ```start()```은 ```engineObj.on()``` 엔진을 켠다. 그리고 엔진상태를 이벤트로 발생한다.
52 ~ 60 | ```speed```가 0 ~ 최대속도 범위 내에서 속도를 증감하도록 한다. 감속을 위해 음수를 사용할 수 있도록 ```int```를 적용하면 ```uint```로 형변환이 필요함.

```python
[파일명: src/Car.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;
//pragma solidity 0.4.21;

contract Car {
    Engine engineObj;
    string private color;
    event PrintLog(address sender, string msg);

    constructor() { //constructor() public {
        engineObj = new Engine();
    }
    function setColor(string memory _color) public {
        color=_color;
    }
    function getColor() public view returns(string memory) {
        return color;
    }
    function getSpeed() public view returns(uint) {
        return engineObj.getSpeed();
    }
    function speedUpBy10() public {
        engineObj.setSpeedUpBy(10);
    }
    function speedDownBy10() public {
        engineObj.setSpeedDownBy(10);   
    }
    function start() public {
        engineObj.on();
        //convert bool to string
        string memory engineStateStr=engineObj.getEngineState()? "on" : "off";
        emit PrintLog(msg.sender, engineStateStr);
    }
}

contract Engine {
    uint constant private MAXSPEED = 200;
    uint private speed;
    bool private engineState;

    constructor() {//constructor() public {
        speed = 0;
        off();
    }
    function on() public {
        engineState = true;
    }
    function off() public {
        engineState = false;
    }
    function getEngineState() public view returns(bool){
        return engineState;
    }
    function setSpeedUpBy(uint _speed) public {
        if(speed < (MAXSPEED - 10) && engineState == true)
            speed += _speed;
    }
    // int _speed -> needs casting to uint (speed)
    function setSpeedDownBy(uint _speed) public {
        if((speed - 10) > 0  && engineState == true)
            speed -= _speed;
    }
    function getSpeed() public view returns(uint) {
        return speed;
    }
}
```

### 단계 2: 컴파일

현재 설치되어 있는 버전으로 컴파일 해서, abi, bin을 구한다.
버전이 다른 경우, REMIX에서 컴파일한다.

```python
pjt_dir> solc-windows.exe src/Car.sol --combined-json abi,bin > src/Car.json
```

### 단계 3: 배포

```python
[파일명: src/carDeploy.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/Car.json");
var _json=JSON.parse(_str)
//var _abiArray=JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray=JSON.parse(_json.contracts["src/Car.sol:Car"].abi);
var _abiArray=_json.contracts["src/Car.sol:Car"].abi;
//var _bin=_json.contracts.sHello2.bin;
var _bin="0x"+_json.contracts["src/Car.sol:Car"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000})
        .on('transactionHash', function(hash){
            console.log(">>> transactionHash"+hash);
        })
        .on('receipt', function(receipt){
            console.log(">>> RECEPIT hash: " + receipt.transactionHash + "\n>>> address:" + receipt.contractAddress);
        })
        .on('error', function(error, receipt) {
            console.log(">>> ERROR "+error);
        });
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

```python
pjt_dir> node src/carDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
>>> transactionHash0x1bb2b96b66d599658b3f88b25b48ac14c07ccbf970f876cd675bda7f77ac5de4
>>> RECEPIT hash: 0x1bb2b96b66d599658b3f88b25b48ac14c07ccbf970f876cd675bda7f77ac5de4
>>> address:0x44B2A4352a50866da46FDB39AFbcA12Cf6277506
---> The contract deployed to: 0x44B2A4352a50866da46FDB39AFbcA12Cf6277506
```

### 단계 4: 사용
아래를 실행하면 다음 결과

```python
> car.getColor.call();
'RED'
```

```python
[파일명: src/carUse.js]
var Web3=require('web3');
//var web3=new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));         //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://117.16.44.45:8345"));  //ok
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/Car.json");
var _json=JSON.parse(_str)
//var _abiArray=JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray=JSON.parse(_json.contracts["src/Car.sol:Car"].abi);
var _abiArray=_json.contracts["src/Car.sol:Car"].abi;

async function doIt() {
    var car = new web3.eth.Contract(_abiArray, "0x44B2A4352a50866da46FDB39AFbcA12Cf6277506");
    car.events.PrintLog({fromBlock: 'latest', toBlock:'pending'}, function (error, event) {
            console.log(">>> Event fired: " + JSON.stringify(event.returnValues));
        }).on('>> data', function(event){
            console.log(event); // same results as the optional callback above
        }).on('>> changed', function(event){
            console.log(event); // remove event from local database
        }).on('>> error', console.error);

    var speed;
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    car.methods.setColor("RED").send({from: accounts[0], gas:100000});
    car.methods.getColor().call().then(console.log);
    await car.methods.start().send({from: accounts[0], gas:100000})
    await car.methods.speedUpBy10().send({from: accounts[0], gas:100000})
    car.methods.getSpeed().call().then(function(speed) { console.log("-> speed: " + speed) });
    await car.methods.start().send({from: accounts[0], gas:100000})
    await car.methods.speedUpBy10().send({from: accounts[0], gas:100000})
    car.methods.getSpeed().call().then(function(speed) { console.log("-> speed: " + speed) });
    await car.methods.speedDownBy10().send({from: accounts[0], gas:100000})
    car.methods.getSpeed().call().then(function(speed) { console.log("-> speed: " + speed) });
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    process.exit(1); //force exit -> may terminate some functions (speedDownBy10, getSpeed)
}

doIt()
```

웹소켓은 계속 이벤트가 발생하는지 지켜보기 때문에 프로세스가 종료하지 않을 수 있다.
마지막에 적은 ```process.exit(1)```로 인해 3번째 speedDownBy10() 호출이 실행하지 않고 강제 종료되고 있다.

```python
pjt_dir> node src/carUse.js


Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999934797147999987456
RED
>>> Event fired: {"0":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","1":"on","sender":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","msg":"on"}
-> speed: 30
>>> Event fired: {"0":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","1":"on","sender":"0x9357f478d86D9222f4413bFd91C8adb0F4c728b7","msg":"on"}
-> speed: 40
Balance after: 999934405075999987456
Balance diff: 392071999913984
```

## 실습: 사각형과 면적의 조립 - new 명령으로 사각형 생성하고 주소 출력

정사각형은 한 변의 길이만 가지면 충분하다. 그리고 각도는 당연하지만 90도를 반환하도록 구현한다.
그리고 별도의 '면적' 컨트랙을 구현하여, 이 정사각형을 조립관계로 가지고, 면적을 계산한다.

### 단계 1: 컨트랙 개발

* ```getAddressOfSquare()``` 정사각형의 주소를 출력하는 함수를 구현한다. Square, Area 두 개의 컨트랙이 한 파일에 있고 우리는 Area 컨트랙만 배포한다. 그럼에도 불구하고 Square의 주소를 획득할 수 있다는 점을 유의하자.
* ```changeSquare(address _addressOfSquare)``` 별도로 사각형을 배포한 후, 그 사각형의 주소로 변경할 수 있다.


```python
[파일명: src/SquareArea.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;
//pragma solidity 0.4.21;

contract Square {
    uint128 length;
    function getLength() public view returns(uint128) {
        return length;
    }
    function setLength(uint128 _length) public {
        length=_length;
    }
    function getDegree() public pure returns(uint128) {
        return 90;
    }
}

contract Area {
    Square s;
    address owner;
    //event PrintLog(uint128);
    //function Area() public {
    constructor() { //constructor() public {
        s = new Square();
        owner = msg.sender;
    }
    function changeSquare(address _addressOfSquare) public {
        s=Square(_addressOfSquare);
    }
    function calcArea() view public returns(uint128) {
        uint128 length = s.getLength();
        uint128 area = length*length;
        //emit PrintLog(area);
        return area;
    }
    function setLength(uint128 _length) public {
        s.setLength(_length);
    }
    function getLength() public view returns(uint128) {
        return s.getLength();
    }
    function getDegree() public view returns(uint128) {
        return s.getDegree();
    }
    function getAddressOfSquare() public view returns(address) {
        return address(s);
    }
}

```

### 단계 2: 컴파일



```python
pjt_dir> solc-windows.exe src/SquareArea.sol --combined-json abi,bin > src/SquareArea.json
```

### 단계 3: 컨트랙 배포


```python
[파일명: src/SquareAreaDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/SquareArea.json");
var _json=JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/SquareArea.sol:Area"].abi);
var _abiArray = _json.contracts["src/SquareArea.sol:Area"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/SquareArea.sol:Area"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin})
        .send({from: accounts[0], gas: 1000000})
        .on('transactionHash', function(hash){
            console.log(">>> transactionHash"+hash);
        })
        .on('receipt', function(receipt){
            console.log(">>> RECEPIT hash: " + receipt.transactionHash + "\n>>> address:" + receipt.contractAddress);
        })
        .on('error', function(error, receipt) {
            console.log(">>> ERROR "+error);
        });
        //.then(function(newContractInstance){
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}

deploy()

```


```python
pjt_dir> node src/SquareAreaDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
>>> transactionHash0xaf846e338979d50ed1d133004dc4dedea16dc81b37afdc72538d07f3a7409925
>>> RECEPIT hash: 0xaf846e338979d50ed1d133004dc4dedea16dc81b37afdc72538d07f3a7409925
>>> address:0xf9B59A87fF00Eb38E59A4E065c63b2C991C8C11A
---> The contract deployed to: 0xf9B59A87fF00Eb38E59A4E065c63b2C991C8C11A
```

### 단계 4: 사용

노드에서 한 줄씩 입력하면서 그 결과를 출력할 수 있다.
```geth 8445```를 띄워놓고 어떤 로그가 발생하는지 관찰하는 것도 좋은 방법이다.
마이닝이 ganache에서는 필요하지 않지만, geth에서는 필요하고 아래 중간에 마이닝을 커멘트로 적어놓았는데, 그 send()하는 시점에 마이닝을 실행해야 한다.
```python
> area.methods.getDegree().call().then(console.log);
> 90
> area.methods.setLength(9).send({from:"0x9cd639aac746b3bbc85e386c05cf1ac74925112e"});
> //miner.start(1);admin.sleepBlocks(1);miner.stop();
undefined
> area.methods.getLength().call().then(console.log);
> 9
> area.methods.calcArea().send({from:"0x9cd639aac746b3bbc85e386c05cf1ac74925112e"});
> //miner.start(1);admin.sleepBlocks(1);miner.stop();
undefined
> area.methods.calcArea().call().then(console.log);           로컬에서 처리되기 때문에 이전 면적계산 값 81을 반환
> 81
> area.methods.getAddressOfSquare().call().then(console.log);
> 0x9f75D1A80D5715EED0962dd72d1d4E409DaCF037
```

처음 실행하면 길이는 0이다. 그 다음 실행하면 9가 된다. 그 이유는 async방식이라 마이닝하지 않고 연달아 시행하면 마이닝 전의 결과가 나오기 때문에 그렇다.

```python
[파일명: src/SquareAreaUse.js]
var Web3 = require('web3');
//var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));         //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://117.16.44.45:8345"));  //ok
var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://localhost:8345"));
var fs=require('fs');
var _str = fs.readFileSync("src/SquareArea.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/SquareArea.sol:Area"].abi);
var _abiArray = _json.contracts["src/SquareArea.sol:Area"].abi;

async function doIt() {
    var area = new web3.eth.Contract(_abiArray, "0xf9B59A87fF00Eb38E59A4E065c63b2C991C8C11A");
    var speed;
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    area.methods.getDegree().call().then(console.log);
    await area.methods.setLength(9).send({from: accounts[0]});
    area.methods.getLength().call().then(console.log);
    //area.methods.calcArea().send({from: accounts[0]});
    area.methods.calcArea().call().then(console.log);
    area.methods.getAddressOfSquare().call().then(function(address) {
        console.log("Square Address: " + address);
    });
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
    //process.exit(1); //force exit -> may terminate some functions (speedDownBy10, getSpeed)  
}

doIt()

```

아래에서 구한 ```Square```의 주소는 다음 문제에서 ```changeSquare(address _addressOfSquare)```에 설정하여 사용해보자. ```process.exit(1)```으로 일부 명령문이 실행되지 못하고 중단될 수 있다. 명령창에서 실행하고 모든 명령이 실행되면, ```Ctrl-C```를 눌러 강제로 종료하도록 한다.
```
pjt_dir> node .\src\SquareAreaUse.js
Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
Balance before: 999932747743999987456
90
Balance after: 999932688843999987456
Balance diff: 58900000014336
Square Address: 0xFD21931acdCccA516682cC853eD03257515302d9
9
81
```

```python
pjt_dir> node src/SquareAreaUse.js

    Account: 0x0A2aca05EB30707F09C883A4b1881F775ACA4Fa8
    Balance before: 99973924640000000000
    90
    9
    81
    Square Address: 0x748929714418AaF173Cc14cB269BA246573a71eD
    Balance after: 99973043860000000000
    Balance diff: 880780000002048
```

## 실습: 사각형과 면적의 조립 - import문으로 사각형 포함, 이미 배포된 사각형 컨트랙으로 교체해보기

* ```Square.sol```을 구현
* ```Area.sol```을 구현
    * ```import```문을 사용하여 ```Square.sol```을 포함
    * ```changeSquare(address _addressOfSquare)``` 함수를 호출하여, 앞서 획득한 Square.sol의 주소로 '중도에' 교체해보자.

### 단계 1: 컨트랙 개발

#### Square

```Square.sol```은 별도로 로컬 파일에 저장해서, ```Area.sol```에서 포함하도록 한다.
```Square.sol```은 컴파일하지 않는다.


```python
[파일명: src/Square.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;
//pragma solidity 0.4.21;

contract Square {
    uint128 length;
    function getLength() public view returns(uint128) {
        return length;
    }
    function setLength(uint128 _length) public {
        length=_length;
    }
    function getDegree() public pure returns(uint128) {
        return 90;
    }
}
```


#### Area

Square를 import할 때는 컴파일하는 Area.sol 기준으로 상대 경로 ```import "./Square.sol"```로 적어준다.


```python
[파일명: src/Area.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;
//pragma solidity 0.4.21;

import "./Square.sol";

contract Area {
    Square s;
    address owner;
    //event PrintLog(uint128);
    //function Area() public {  //0.4.21
    constructor() { //constructor() public {
        s = new Square();
        owner = msg.sender;
    }
    function changeSquare(address _addressOfSquare) public {
        s=Square(_addressOfSquare);
    }
    function calcArea() view public returns(uint128) {
        uint128 length = s.getLength();
        uint128 area = length*length;
        //emit PrintLog(area);
        return area;
    }
    function setLength(uint128 _length) public {
        s.setLength(_length);
    }
    function getLength() public view returns(uint128) {
        return s.getLength();
    }
    function getDegree() public view returns(uint128) {
        return s.getDegree();
    }
    function getAddressOfSquare() public view returns(address) {
        return address(s);
    }
}
```

### 단계 2: 컴파일

import 문으로 포함했으니, Square를 컴파일해서, abi, bin를 필요하지 않는다.

```python
pjt_dir> solc-windows.exe src/Area.sol --combined-json abi,bin > src/Area.json
```

### 단계 3: 컨트랙 배포

컴파일하고 Area.sol의 abi, bin만을 넣어준다. Square.sol의 abi,bin은 무시한다.


```python
[파일명: src/AreaDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/Area.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/Area.sol:Area"].abi);
var _abiArray = _json.contracts["src/Area.sol:Area"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/Area.sol:Area"].bin;

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
pjt_dir> node src/AreaDeploy.js

    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0x029823d5e240701347231ab1774afc4348f337ef0f9b483979392aa48b76af69
    ---> The contract deployed to: 0x01Dc10B774D545926CAC4595aF91Cd4Df11923aB
```

### 단계 4: 사용

함수 | 설명
-----|-----
HttpProvider("http://localhost:8345")) | 이벤트를 출력하는 것은 이미 해보았고, 여기서는 사각형 주소 교체가 주목적이므로 그냥 Http를 사용하자.
```getAddressOfSquare()``` | 생성자에서 할당된 Square 주소, 즉 ```import ./Square.sol```에서 가져온 ```new Square()```의 주소. 주소 없이 ```new Square()```라고 해도 문제없이 실행이 된다. 즉, ```import```문을 사용하면 주소없이 컨트랙을 생성해서 사용할 수 있다는 의미이다.
```setLength()``` | 길이 설정하고
```calcArea()``` | 설정된 길이로 면적을 계산
```changeSquare('0xFD21931acdCccA516682cC853eD03257515302d9')``` | 주소를 재설정. 위에서 주소를 넣는다. 즉, 앞서 ```SquareArea.sol```을 컴파일하고 ```getAddressOfSquare()```에서 구한 ```Square```의 주소를 넣어 ```Square```를 교체하여 보자.  단 주소에 따옴표를 해야 한다. 그러고 나서 기능이 적절히 수행되는지 확인해 보자.


```python
[파일명: src/AreaUse.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://117.16.44.45:8345"));  //ok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://117.16.44.45:8345"));  //ok
var fs=require('fs');
var _str = fs.readFileSync("src/Area.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.Area.abi);
//var _abiArray = JSON.parse(_json.contracts["src/Area.sol:Area"].abi);
var _abiArray = _json.contracts["src/Area.sol:Area"].abi;

async function doIt() {
    var hello = new web3.eth.Contract(_abiArray, "0x01Dc10B774D545926CAC4595aF91Cd4Df11923aB");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance before: " + balanceBefore);
    // do by the Square address as set in the constructor
    hello.methods.getAddressOfSquare().call(function(err, c1addr) {
        if(!err) console.log(">> Square address by 'new': "+c1addr);
    });
    await hello.methods.setLength(10).send({from: accounts[0]});
    hello.methods.getLength().call().then(console.log);
    hello.methods.calcArea().call().then(console.log);
    hello.methods.getDegree().call().then(console.log);
    hello.methods.getAddressOfSquare().call().then(console.log);
    //redo by the Square address as changed by changeSquare()
    await hello.methods.changeSquare('0xFD21931acdCccA516682cC853eD03257515302d9').send({from: accounts[0]});
    hello.methods.getAddressOfSquare().call(function(err, c1addr) {
        if(!err) console.log(">> Square address by 'changeSquare: "+c1addr);
    });
    await hello.methods.setLength(10).send({from: accounts[0]});
    hello.methods.getLength().call().then(console.log);
    hello.methods.calcArea().call().then(console.log);
    hello.methods.getDegree().call().then(console.log);
    hello.methods.getAddressOfSquare().call().then(console.log);
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}

doIt()

```


* 첫 번째 ```getAddressOfSquare()``` 함수는 ```new``` 명령어로 생성된 사각형의 주소를 출력한다.
* ```changeSquare()``` 후 두 번째 ```getAddressOfSquare()```는 web3에서 주입한 사각형의 주소(앞의 예제에서 획득)를 출력한다.


```python
pjt_dir> node src/AreaUse.js

    Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    Balance before: 999931522283999987456
    >> Square address by 'new': 0x42ec439A7A362D2BD4FFfd14302fF65741C15Db9
    0x42ec439A7A362D2BD4FFfd14302fF65741C15Db9
    90
    10
    100
    >> Square address by 'changeSquare: 0xFD21931acdCccA516682cC853eD03257515302d9
    Balance after: 999931304999999987456
    Balance diff: 217283999956992
    0xFD21931acdCccA516682cC853eD03257515302d9
    90
    10
    100
```

다시 한 번 실행해보자. 사각형의 주소에 어떤 변화가 있는지 살펴보자.
생성자는 최초에만 실행되므로, ```Square```의 주소가 변경된 상태로 실행된다.


```python
pjt_dir> node src/AreaUse.js

    Account: 0x0A2aca05EB30707F09C883A4b1881F775ACA4Fa8
    Balance before: 99954709760000000000
    >> Square address by 'new': 0x07cE6901B343abA0Fabe1098E360F7503De920cD
    10
    100
    90
    0x07cE6901B343abA0Fabe1098E360F7503De920cD
    >> Square address by 'changeSquare: 0x748929714418AaF173Cc14cB269BA246573a71eD
    10
    100
    90
    0x748929714418AaF173Cc14cB269BA246573a71eD
    Balance after: 99952778820000000000
    Balance diff: 1930939999993856
```

###  registrar

```python
> primary = eth.accounts[0];
'0xa5cd6f89bb6100aa6c1f634fa5937f0b201ff43c'
> globalRegistrarAddr = admin.setGlobalRegistrar("", primary);
Please unlock account a5cd6f89bb6100aa6c1f634fa5937f0b201ff43c.
Passphrase: 
Account is now unlocked for this session.
'0x89101a18ba7d97c35bcff99e8f2d290ecd9e9faabc8b4b36200b237d9974545d'
> globalRegistrarAddr
'0x89101a18ba7d97c35bcff99e8f2d290ecd9e9faabc8b4b36200b237d9974545d'
>
```

# 2. 상속

## 2.1 상속 ```is-a``` 관계

Solidity는 객체지향의 상속을 사용할 수 있다. 상속에서와 같이, 자식 클래스가 부모 클래스의 ```private```을 제외한 멤버속성과 멤버함수를 물려받으면서, 이를 확장하거나 개선하게 된다.

단일, 다중 상속을 모두 지원한다.

자바에서 사용하는 ```super```, ```this``` 명령어는 사용할 수 없다. ```this```는 다른 용도로 사용되며, ```external``` 함수를 내부에서 호출할 때 사용된다.

## 실습: 점과 사각형의 상속

### 단계 1: 컨트랙 개발

#### 부모 컨트랙 Point

점은 좌표 x, y를 가진다. 이 좌표를 출력하는 getX(), getY()를 멤버함수로 가진다.

```python
[파일명: src/Point.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Point {
    int x;
    int y;
    constructor(int _x, int _y) { //constructor(int _x, int _y) public {
        x = _x;
        y = _y;
    }
    function getX() view public returns(int) { return x; }
    function getY() view  public returns(int) { return y; }
}
```

#### 자식 컨트랙 Rectangle

* 상속할 부모 컨트랙을 import 한다. 주의, 컨트랙 명이 아니라, 그 파일명을 경로와 함께 적어준다. REMIX에서도 같은 파일명을 잘 찾는다.

* 사각형은 점을 상속받는다. 이러한 상속관계는 ```contract Rectangle is Point```로 나타낸다.

* 생성자는 좌표와 너비, 높이를 가진다. 부모의 생성자를 직접 적어준다.

```
constructor(int _w, int _h, int _x, int _y) Point(_x, _y)
```

생성자에 매개변수를 가지고 있으므로, 배포할 때 그 값을 넣어주어야 한다.

```super``` 명령어로 부모생성자를 호출하지 못한다.
```
constructor(int _w, int _h, int _x, int _y)  {
        super(_x, _y); //부모 생성자 호출 못함.
```

* 부모 함수의 호출은 물려받은 ```getX()```, ```getY()``` 함수를 그냥 호출하면 된다.


```python
[파일명: src/Rectangle.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
import "./Point.sol";  //컨트랙의 경로포함 파일명을 적는다

contract Rectangle is Point {
    int w;
    int h;
    constructor(int _w, int _h, int _x, int _y) Point(_x, _y) {
        //super(_x, _y); //nok
        w = _w;  //this.w --> nok
        h = _h;
    }
    function getPerimeter() view public returns(int) {
        return 2*(w+h);
    }
    function getXOpposite() view public returns(int) { return getX() + w; }
    function getYOpposite() view public returns(int) { return getY() + h; }
}
```

### 단계 2: 컴파일

부모, 자식 가운데 어떤 컨트랙을 컴파일하면 될지 생각해보자.
부모는 자식을 import하므로 컴파일할 때 그 코드도 포함하게 된다.
따라서 부모 Rectangle 컨트랙만 다음과 같이 컴파일하면 된다.


```python
pjt_dir> solc-windows.exe src/Rectangle.sol --combined-json abi,bin > src/Rectangle.json
```

### 단계 3: 컨트랙 배포

배포할 경우 부모 컨트랙은 이미 포함되므로 자식의 abi, bin만 넣어서 배포한다.

줄 (1)을 보면, constrcutor가 문자열을 인자로 받으므로 4개의 인자를 다음과 같이 넣어준다.
```
.deploy({data: _bin, arguments: [10, 20, 30, 40]})
```

```python
[파일명: src/RectangleDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/Rectangle.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/Rectangle.sol:Rectangle"].abi);
var _abiArray = _json.contracts["src/Rectangle.sol:Rectangle"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/Rectangle.sol:Rectangle"].bin;

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin, arguments: [10, 20, 30, 40]})       // (1) 생성자의 인자를 넣어준다.
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
pjt_dir> node src/RectangleDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x6e3c0f430b2f745c1cf785f9f28a490b4358660866dbccdbe721d7389496b151
---> The contract deployed to: 0xA6D9e53c4E10ae1Af52F647C417E1F41eBEe9B1A
```

### 단계 4: 사용

```python
[파일명: src/RectangleUse.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));       //nok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://117.16.44.45:8345"));  //ok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://117.16.44.45:8345"));  //ok
var fs = require('fs');
var _str = fs.readFileSync("src/Rectangle.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/Rectangle.sol:Rectangle"].abi);
var _abiArray = _json.contracts["src/Rectangle.sol:Rectangle"].abi;

async function doIt() {
    var rect = new web3.eth.Contract(_abiArray, "0xA6D9e53c4E10ae1Af52F647C417E1F41eBEe9B1A");
    const accounts = await web3.eth.getAccounts();
    console.log("(1) Account: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("(2) Balance before: " + balanceBefore);
    rect.methods.getPerimeter().call().then(function(res) {console.log("(3) Perimeter: "+res)});
    rect.methods.getXOpposite().call().then(function(res) {console.log("(4) X opp: "+res)});
    rect.methods.getYOpposite().call().then(function(res) {console.log("(5) Y opp: "+res)});
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("(6) Balance after: " + balanceAfter);
    console.log("(7) Balance diff: " + (balanceBefore - balanceAfter));
    
}

doIt()
```

자바스크립트의 비동기적 특성으로 출력 순서가 예상과 다르다는 점에 주의한다.
배포할 때 생성자에 주었던 인자의 값으로 올바르게 출력되고 있다.
* 너비는 2 * (10 + 20) = 60
* XOpposite는 30 + 10 = 40
* YOpposite는 40 + 20 = 60

```python
pjt_dir> node src/RectangleUse.js

(1) Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
(2) Balance before: 999929177179999987456
(6) Balance after: 999929177179999987456
(7) Balance diff: 0
(3) Perimeter: 60
(5) Y opp: 60
(4) X opp: 40
```

## 실습: greeter.sol

### 단계 1: 컨트랙 개발

* 부모 ```mortal``` 컨트랙은 자신을 제거할 수 있도록 ```kill()``` 함수를 구현하고 있다.
```selfdestruct```는 컨트랙을 삭제하는 경우, 잔액을 반환하는 주소를 필요로 한다.
* 이를 상속받은 자식 ```greeter```는 ```greet()``` 함수로 인사할 수 있다.
* 자식 컨트랙은 부모에서 설정된 ```owner```를 읽어오는 ```getOwner()```를 구현해 놓았다.
* 출처 https://www.ethereum.org/greeter

```python
[파일명: src/greeter.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.5.0;

contract mortal {
    address owner;  //address payable owner;
    constructor() { //constructor() public {
        owner = msg.sender;
    }
    function kill() public {
        if (msg.sender == owner) selfdestruct(payable(owner)); }
}

contract greeter is mortal {
    string greeting;
    constructor(string memory _greeting) {
        greeting = _greeting;
    }
    function greet() public view returns (string memory) {
        return greeting;
    }
    function getOwner() view public returns(address) { return owner; }
}
```


## 단계 2: 컴파일

파일에 2개의 컨트랙이 있으므로, 2개의 abi, bytecode가 작성된다.

```python
pjt_dir> solc-windows.exe src/greeter.sol --combined-json abi,bin > src/greeter.json
```

### 단계 3: 컨트랙 배포

* 2개의 컨트랙이 포함되어 있으므로, 배포할 컨트랙을 인덱스로 식별하여 지정한다.
* 인자는 배열로 넣어준다 ```arguments: ['Hello jsl']```
* 상속까지 구현되어 있으므로, gas는 필요한 정도로 적어준다.


```python
[파일명: src/greeterDeploy.js]
var Web3 = require('web3');
var _abiBinJson = require('./greeter.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); 
console.log("- contract name: ", contractName);
//console.log(contractName[0], contractName[1]); // index 0 -> ['src/greeter.sol:greeter']
_abiArray=_abiBinJson.contracts[contractName[0]].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
_bin="0x"+_abiBinJson.contracts[contractName[0]].bin;
//var _abiArray=[{"constant":false,"inputs":[],"name":"kill","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
//var _bin="0x"+"608060405234801561001057600080fd5b506040516103db3803806103db8339810180604052602081101561003357600080fd5b81019080805164010000000081111561004b57600080fd5b8281019050602081018481111561006157600080fd5b815185600182028301116401000000008211171561007e57600080fd5b5050929190505050336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555080600190805190602001906100dc9291906100e3565b5050610188565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061012457805160ff1916838001178555610152565b82800160010185558215610152579182015b82811115610151578251825591602001919060010190610136565b5b50905061015f9190610163565b5090565b61018591905b80821115610181576000816000905550600101610169565b5090565b90565b610244806101976000396000f3fe608060405234801561001057600080fd5b5060043610610053576000357c01000000000000000000000000000000000000000000000000000000009004806341c0e1b514610058578063cfae321714610062575b600080fd5b6100606100e5565b005b61006a610176565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100aa57808201518184015260208101905061008f565b50505050905090810190601f1680156100d75780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415610174576000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b565b606060018054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561020e5780601f106101e35761010080835404028352916020019161020e565b820191906000526020600020905b8154815290600101906020018083116101f157829003601f168201915b505050505090509056fea165627a7a72305820baad8d78174d9288c9240f9d06b886a4e3cb19c1fa71fc14517e70cd520978180029";

var _contract = new web3.eth.Contract(_abiArray);
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: _bin, arguments: ['Hello jsl']})
        .send({from: accounts[0], gas: 400000}, function(err, transactionHash) {
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
pjt_dir> node src/greeterDeploy.js

- contract name:  [ 'src/greeter.sol:greeter', 'src/greeter.sol:mortal' ]
Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x958fb30c284549ab887df94ac1c1a6cc3aa496ebad9e42cc2862a98846548e39
---> The contract deployed to: 0xAFa3e557548a5cD0d6Dd2ae8e5Fce5757F079047
```

### 단계 4: 사용

상속받은 부모의 멤버변수 owner를 읽어서 getOwner() 함수가 출력을 하고 있다.

```python
[파일명: src/greeterUse.js]
var Web3 = require('web3');
var _abiBinJson = require('./greeter.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); 
//console.log("- contract name: ", contractName);
//console.log(contractName[0], contractName[1]); // index 0 -> ['src/greeter.sol:greeter']
_abiArray=_abiBinJson.contracts[contractName[0]].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!! //SyntaxError: Unexpected token o in JSON at position 1
var greet = new web3.eth.Contract(_abiArray, '0xAFa3e557548a5cD0d6Dd2ae8e5Fce5757F079047');

async function doIt() {
    greet.methods.greet().call().then(function(res) {console.log("greet: "+res);});
    greet.methods.getOwner().call().then(function(res) {console.log("owner: "+res);});
}
doIt()
```


```python
pjt_dir> node src/greeterUse.js

owner: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
greet: Hello jsl
```

## 2.2 추상컨트랙

추상컨트랙은 최소한 한 개의 멤버함수를 구현해 놓지 않는 추상함수를 가지게 된다.
자식클래스가 상속받아서 구현이 생략된 부분을 채워넣는다.
자식클래스 역시 모든 멤버함수를 구현하지 않고, 추상함수를 가질 수 있다.

* 추상컨트랙은 abstract이라고 적고, 추상함수는 virtual이라고 적는다.
abstract은 콘트랙에, virtual은 함수에 적용을 한다는 차이가 있다.
* 부모의 virtual 함수는 자식컨트랙에서 override로 적고 부모의 함수를 재정의할 수 있다.
* 미구현 함수가 있는 경우에 즉 추상컨트랙은 객체를 만들지 못한다.
* private은 virtual이 될 수 없다.
* override와 virtual을 같이 한 함수에 동시에 적용할 수 있다.
자신이 부모의 virtual 함수를 override 재정의하면서, 동시에 자신을 상속하는 자식들이 재정의 하도록 허용하는 의미가 있다.

다음에 배우게 되는 DeFi, NFT를 만들 때 활용하게 되므로, 여기서는 예제 코드만 보기로 하자.

```python
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

abstract contract Hello {
    function sayHello() public virtual returns (bytes32);
}

contract HelloMorning is Hello {
    function sayHello() pure public virtual override returns (bytes32) { return "Hello good morning"; }
}
```

## 2.3 Inteface

인터페이스는 함수의 선언과 구현을 분리할 수 있는 방법으로 사용된다.
인터페이스는 '스위치'와 같이 연결하면 그 기능이 제공되고 분리하면 해제되는 것으로 이해하자.

contract 명령어 자리에 interface를 적어주면 된다.
인터페이스는 모든 함수가 virtual이고, 함수는 구현하지 않는다.
* virtual로 선언하지 않는다. 인터페이스의 함수는 기본적으로 ```virtual```이다.
* external로 가시성을 선언한다. internal 또는 private이라고 선언하면 TypeError가 발생한다. public은 버전 0.5까지 가능했으나, 역시 선언하지 못한다. 
* constructor를 가지지 못한다
* state variable을 가지지 못한다
* enum, structs를 가질 수 있다.

구현할 때는 오버라이드하더라도 굳이 override를 적어주지 않아도 된다 (버전 0.8.8부터)

다음 장의 DeFi, NFT를 만들 때 인터페이스를 적용하게 된다.

```python
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

interface Hello {
    function sayHello() external returns (bytes32);
}

contract HelloImpl is Hello {
    function sayHello() pure public virtual override returns (bytes32) { return "Hello"; }
}
```

# 3.  라이브러리

## 3.1 라이브러리는 무엇일까?

라이브러리는 빈번하게 사용되는 부분을, 인터페이스를 잘 정의해서 구현해 놓은 소스코드 또는 실행파일을 말한다. 빈번하게 사용되는 부분이라서 재사용하지 않고 동일한 코드를 여러 번 배포해야 한다면, 비용이 그 만큼 발생하므로 라이브러리가 좋은 대안이 된다.

라이브러리는 다른 형태의 컨트랙으로 보아도 된다. 단 library라는 명령어를 적어서 선언한다는 차이가 있다.

```python
library LibrayName {
}
```

컨트랙과 달리 라이브러리는 storage 공간을 가질 수 없어서 멤버 속성을 생성하여 그 어떤 값을 가질 수 없다.
라이브러리는 ether를 가질 수 없고, payable을 사용할 수 없고, fallback 함수 역시 가질 수 없다. 여러 사람이 사용할 수 있으므로 당연히 그렇겠다.
또한 상속을 사용할 수 없다.


## 3.2 라이브러리 import

라이브러리는 ```import```문으로 가져와 사용할 수 있다.

```python
import LibraryName from "./myLib.sol";
```

```import```문 다음에 괄호로 라이브러리 명을 ```from``` 뒤에 파일 명을 적는다.
예를 들어 다음과 myLib.sol이 구성되어 있다고 하자.

```python
library Library1 {
 // Code from library 1
}
library Library2 {
 // Code from library 2
}
library Library3 {
 // Code fom library 3
}
```

그러면 Libray2를 제외하고 나머지를 사용하려면 다음과 같이 적어주면 된다.

```python
import {Library1, Library3} from "./library-file.sol";
contract MyContract {
    // Your contract code here
}
```

## 3.3 using ... for

```Using MyLib for <<Type>>```는 MyLib을 <<Type>>에 붙여서 사용하겠다는 의미이다.
```Using MyLib for uint```는 uint타잎에 ```Using MyLib for *```는 어떤 타잎이든 붙여 사용하겠다는 의미이다.

첫째 인자는 Python의 self와 같이 자신이 된다. 즉 첫째인자를 객체와 같이 간주하고 자신에게 함수를 붙여 사용한다.

```python
using myLib for uint;
uint result = num.multiply7();
```

## 3.4 라이브러리 배포

라이브러리는 구현해서 배포해 놓으면, 필요로 할 때 그대로 재사용할 수 있게 된다.
내부 또는 외부에서 사용할 수 있는지에 따라 (1) 내장형과 (2) 연결형으로 구분될 수 있다.

### 3.4.1 내부에서만 사용하는 '내장형 라이브러리'

내장형 라이브러리 **embedded library**는 라이브러리 함수가 **internal**로 선언된 경우를 말한다.
내장형 라이브러리는 컨트랙이 배포될 때도 내장, 즉 포함되기 때문에 주소를 적어야 하는 **placeholder가 없다**. 그냥 **library 코드가 콘트랙에 포함**되기 때문에 별도의 link 작업이 필요없다는 의미이다. 따라서 library를 호출할 때 JUMP문을 사용하여 보통 함수를 부르는 것처럼 처리가 된다.


## 실습: 내장형 라이브러리 internal multiply7

라이브러리가 내부용으로 선언되었기 때문에 주소 없이 그냥 사용하면 된다.

### 단계 1: 컨트랙 개발

* multiply7()은 internal로 선언한다.
* 라이브러리는 ```myLib.multiply7()``` 직접 라이브러리명을 통해 함수를 호출한다.


```python
[파일명: src/LibraryTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;
//pragma solidity ^0.5.0;

library myLib {
    //function multiply7(uint num) public pure returns (uint) {
    function multiply7(uint num) internal pure returns (uint) {
      return num * 7;
   }
}
    
contract LibrayTest {
    //event PrintLog(uint);
    function multiply7By(uint num) pure public returns(uint) {
        uint n=myLib.multiply7(num);
        //emit PrintLog(n);
        return n;
    }
}
```


### 단계 2: 컴파일

internal로 선언한 까닭에 myLib은 ABI가 없다. 생성된 JSON 파일을 열어 확인해보자.

```python
pjt_dir> solc-windows.exe src/LibraryTest.sol --combined-json abi,bin > src/LibraryTest.json
```

### 단계 3: 컨트랙 배포


```python
[파일명: src/LibraryTestDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/LibraryTest.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/LibraryTest.sol:LibrayTest"].abi);
var _abiArray = _json.contracts["src/LibraryTest.sol:LibrayTest"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/LibraryTest.sol:LibrayTest"].bin;

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
pjt_dir> node src/LibraryTestDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0xf539552ec42e24125b4a21e77c1e4ea15c5fd0e3eecd9fb35c1502005077104c
---> The contract deployed to: 0x073AC32fCAffe303BF39D15Bad575Eb882E6262A
```

### 단계 4: 사용

```multiply7By(5).call()```는 이벤트를 발생하지 않고 결과만 출력한다.
```lib.multiply7By.send(5,{from: accounts[0]})```는 이벤트를 출력해서 결과를 볼 수 있다. 단 마이닝을 해야 한다.


```python
[파일명: src/LibraryTestUse.js]
var Web3=require('web3');
var web3=new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider("http://117.16.44.45:8345"));  //ok
//var web3 = new Web3(new Web3.providers.WebsocketProvider("ws://117.16.44.45:8345"));  //ok
var fs = require('fs');
var _str = fs.readFileSync("src/LibraryTest.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/LibraryTest.sol:LibrayTest"].abi);
var _abiArray = _json.contracts["src/LibraryTest.sol:LibrayTest"].abi;

async function doIt() {
    var lib = new web3.eth.Contract(_abiArray, "0x073AC32fCAffe303BF39D15Bad575Eb882E6262A");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    lib.methods.multiply7By(5).estimateGas(function(err,gas) {
        if(!err) console.log(">> gas: "+ gas);
    });
    lib.methods.multiply7By(5).call().then(console.log);
}

doIt()

```


```python
pjt_dir> node src/LibraryTestUse.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
35
>> gas: 22050
```

### 3.4.2 외부에서 사용하는 '연결 라이브러리'

연결 라이브러리 **linked library**는 외부에서 사용할 수 있다.
라이브러리 함수가 **public 또는 external**로 선언하면 외부에서 사용할 수 있다는 의미이다.
라이브러리를 사전에  **별도로 배포해서 link**해야 한다 (Library linking).
이는 연결부에 **placeholder**를 표시해 놓게 되는데,
그 연결부에 placeholder를 표시해놓게 되는데, 그 곳에 실제 **라이브러리의 배포주소를 대체**해서 넣은 바이트코드를 배포하게 된다.

## 실습: 연결 라이브러리 public multiply7

외부 라이브러리를 사용하는 컨트랙을 컴파일한다고 하자.
그러면 바이트코드에 라이브러리의 주소를 넣을 수 있는 자리 placeholder를 표시해 놓는다.
외부라이브리러가 이미 배포되어 있거나, 동시에 작업하더라도 자리를 표시해 놓는 것은 동일하다.
이를 Library linking 작업을 통해 해야 한다.
```solc --libraries``` 명령어를 사용하여 라이브러리를 링크할 수 있다.

### 단계 1: 컨트랙 개발

```library```는 별도로 저장한다.


```python
[파일명: src/myLib.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;

library myLib {
    //function multiply7(uint num) internal pure returns (uint) {
    function multiply7(uint num) public pure returns (uint) {
        return num * 7;
    }
}
```


library를 사용하기 위해서는 ```import "./myLib.sol"```로 포함한다.
그리고 그 사용범위를 uint256에 대해서 적용한다.
```using myLib for uint256```로 사용한다.


```python
[파일명: src/LibraryTestPublic.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;

import "./myLib.sol";

contract LibrayTestPublic {
    using myLib for uint256;
    function multiply7By(uint num) public pure returns(uint) {
        return num.multiply7();
    }
}
```

### 단계 2: 컴파일

아래 코드를 보면 **placeholder**를 찾을 수 있다.

#### 2-1 전체 컴파일

* 컨트랙의 placeholder
    * 컨트랙이 public library를 포함하고 있다면, 바이트코드에  ```__$2eb0f...473de$__``` 이런 식으로 라이브러리주소 넣을 자리 placeholder를 마련해 놓는다. 이 문자열은 34자리, 앞 뒤 underbar 4자리, 두자리 ampersand를 포함하여 총 **40 자리**로 구성되어 있다. 그 placeholder는 **library 명의 keccak256 hash의 hex 부호 34자리**이다.
* 라이브러리의 abi, bin
    * import 문을 사용했지만, abi와 bin을 출력하고 있다.
    * 컴파일 한 결과를 위 ```internal library```와 비교해 보자. public library를 컴파일하면 ABI를 생성하고 있다는 점에서 차이가 있다.



```python
pjt_dir> solc-windows.exe src/LibraryTestPublic.sol --combined-json abi,bin > src/LibraryTestPublic.json
```


```python
pjt_dir> type src\LibraryTestPublic.json

{"contracts":{"src/LibraryTestPublic.sol:LibrayTestPublic":{"abi":[{"inputs":[{"internalType":"uint256","name":"num","type":"uint256"}],"name":"multiply7By","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}],"bin":"608060405234801561001057600080fd5b50610219806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c80633e65296914610030575b600080fd5b61004a6004803603810190610045919061011c565b610060565b60405161005791906101a7565b60405180910390f35b60008173__$2eb0fc2ba9c1b20950fd715a17616473de$__6350785a4b90916040518263ffffffff1660e01b815260040161009b919061018c565b60206040518083038186803b1580156100b357600080fd5b505af41580156100c7573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906100eb9190610145565b9050919050565b600081359050610101816101cc565b92915050565b600081519050610116816101cc565b92915050565b60006020828403121561012e57600080fd5b600061013c848285016100f2565b91505092915050565b60006020828403121561015757600080fd5b600061016584828501610107565b91505092915050565b610177816101c2565b82525050565b610186816101c2565b82525050565b60006020820190506101a1600083018461017d565b92915050565b60006020820190506101bc600083018461016e565b92915050565b6000819050919050565b6101d5816101c2565b81146101e057600080fd5b5056fea264697066735822122096c86b02c81bcd6936204cee57ed71b16788850cc4ee45802a7fb6f200e3a4b164736f6c63430008010033"},"src/myLib.sol:myLib":{"abi":[{"inputs":[{"internalType":"uint256","name":"num","type":"uint256"}],"name":"multiply7","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}],"bin":"6101c8610053600b82828239805160001a607314610046577f4e487b7100000000000000000000000000000000000000000000000000000000600052600060045260246000fd5b30600052607381538281f3fe73000000000000000000000000000000000000000030146080604052600436106100355760003560e01c806350785a4b1461003a575b600080fd5b610054600480360381019061004f9190610095565b61006a565b60405161006191906100cd565b60405180910390f35b600060078261007991906100e8565b9050919050565b60008135905061008f8161017b565b92915050565b6000602082840312156100a757600080fd5b60006100b584828501610080565b91505092915050565b6100c781610142565b82525050565b60006020820190506100e260008301846100be565b92915050565b60006100f382610142565b91506100fe83610142565b9250817fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff04831182151516156101375761013661014c565b5b828202905092915050565b6000819050919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b61018481610142565b811461018f57600080fd5b5056fea26469706673582212209d23de3599abd1b52e054ee01e02c804af15e5aeb456144bfa21c589d1c75a1464736f6c63430008010033"}},"version":"0.8.1+commit.df193b15.Windows.msvc"}
```

#### 2-2 library 주소 구하기

컴파일하고 나면 ```import```한 ```myLib.sol``` 라이브러리 abi, bin을 뒷부분에서 발견할 수 있는데, 이를 이용해서 배포해보자.


```python
[파일명: src/myLibDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/LibraryTestPublic.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/myLib.sol:myLib"].abi);
var _abiArray = _json.contracts["src/myLib.sol:myLib"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/myLib.sol:myLib"].bin;

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
!node src/myLibDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x100335e84a7dafb3c5b31253b1e775e694aa9301ce987cc6e71e3afb04f269cf
---> The contract deployed to: 0x19307aB5C14062A109C8cb91B2e742F15cA7707a
```

#### 2.3 링크

라이브러리를 링크해서 컴파일한다. solc 명령어로 링크를 하면 placeholder 그 자리에 라이브러리 주소를 넣어 연결하게 된다.

컴파일할 때 solc에 ```--libraries "file.sol:Math:0x1234567890123456789012345678901234567890```


#### 자동

```myLib```은 간편하게 라이브러리 명만 넣어도 되고, 아래와 같이 경로 ```src/myLib.sol```와 같이 라이브러리명 ```myLib```을 같이 적어도 된다.
```python
pjt_dir> solc src/LibraryTestPublic.sol --libraries "src/myLib.sol:myLib:0x92b3CA97A7cD6aED69D88304D6a98D8e39cFb988" --combined-json abi,bin > src/LibraryTestPublicLink__.json
```

```python
pjt_dir> solc-windows.exe src/LibraryTestPublic.sol --libraries "myLib:0x19307aB5C14062A109C8cb91B2e742F15cA7707a" --combined-json abi,bin > src/LibraryTestPublicLink.json
```

아래에서 보듯이 (일부러 ```__```로 구분해 놓았다)  앞서 배포한 ```myLib``` 주소를 placeholder에 넣고 있다.
```
"608060405234801561001057600080fd5b50610219806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c80633e65296914610030575b600080fd5b61004a6004803603810190610045919061011c565b610060565b60405161005791906101a7565b60405180910390f35b60008173__19307ab5c14062a109c8cb91b2e742f15ca7707a__6350785a..."
```


```python
pjt_dir> type src\LibraryTestPublicLink.json

{"contracts":{"src/LibraryTestPublic.sol:LibrayTestPublic":{"abi":[{"inputs":[{"internalType":"uint256","name":"num","type":"uint256"}],"name":"multiply7By","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}],"bin":"608060405234801561001057600080fd5b50610219806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c80633e65296914610030575b600080fd5b61004a6004803603810190610045919061011c565b610060565b60405161005791906101a7565b60405180910390f35b6000817319307ab5c14062a109c8cb91b2e742f15ca7707a6350785a4b90916040518263ffffffff1660e01b815260040161009b919061018c565b60206040518083038186803b1580156100b357600080fd5b505af41580156100c7573d6000803e3d6000fd5b505050506040513d601f19601f820116820180604052508101906100eb9190610145565b9050919050565b600081359050610101816101cc565b92915050565b600081519050610116816101cc565b92915050565b60006020828403121561012e57600080fd5b600061013c848285016100f2565b91505092915050565b60006020828403121561015757600080fd5b600061016584828501610107565b91505092915050565b610177816101c2565b82525050565b610186816101c2565b82525050565b60006020820190506101a1600083018461017d565b92915050565b60006020820190506101bc600083018461016e565b92915050565b6000819050919050565b6101d5816101c2565b81146101e057600080fd5b5056fea2646970667358221220d43b0df1e69e638684ba3f950f5f4137e82e79e4a152683d8df039e47fc7d96564736f6c63430008010033"},"src/myLib.sol:myLib":{"abi":[{"inputs":[{"internalType":"uint256","name":"num","type":"uint256"}],"name":"multiply7","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}],"bin":"6101c8610053600b82828239805160001a607314610046577f4e487b7100000000000000000000000000000000000000000000000000000000600052600060045260246000fd5b30600052607381538281f3fe73000000000000000000000000000000000000000030146080604052600436106100355760003560e01c806350785a4b1461003a575b600080fd5b610054600480360381019061004f9190610095565b61006a565b60405161006191906100cd565b60405180910390f35b600060078261007991906100e8565b9050919050565b60008135905061008f8161017b565b92915050565b6000602082840312156100a757600080fd5b60006100b584828501610080565b91505092915050565b6100c781610142565b82525050565b60006020820190506100e260008301846100be565b92915050565b60006100f382610142565b91506100fe83610142565b9250817fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff04831182151516156101375761013661014c565b5b828202905092915050565b6000819050919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b61018481610142565b811461018f57600080fd5b5056fea2646970667358221220b8a27c1352db3c8ab8699e4aa70868b6ee80532c1bcdd8dfa5cbd838ab641e2d64736f6c63430008010033"}},"version":"0.8.1+commit.df193b15.Windows.msvc"}
```

#### 수동

컴파일을 하고 나면, myLib 라이브러리가 public함수라서 binary에 placeholder가 위치하고 있다는 것을 알 수 있다.
작업순서:

* 1. 위 2.1에서 생성한 bytecode를 복사해오자.

```python
608060405234801561001057600080fd5b50610134806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633e65296914602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b60008173__$2eb0fc2ba9c1b20950fd715a17616473de$__6350785a4b90916040518263ffffffff1660e01b81526004018082815260200191505060206040518083038186803b15801560be57600080fd5b505af415801560d1573d6000803e3d6000fd5b505050506040513d602081101560e657600080fd5b8101908080519060200190929190505050905091905056fea2646970667358221220f42ce135fbefe1ffc4eefc7bc0320c33dbdf257910573c611e2adf7c738edb8b64736f6c63430006010033
```

* 2. 위 2.2에서 library myLib 배포 후 얻은 주소를 복사해오자.

```python
0xFFd1f58a808ab6001133eA3a60e07f4991E6012e
```

* 3. 수작업으로 placeholder에 myLib주소 복사

위 주소에서 ```0x```를 제거하고, placeholder에 복사해 넣자.
아래는 예시로 일부러 underbar를 남겨 놓았지만 당연히 제거되어야 한다.

```python
608060405234801561001057600080fd5b50610134806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633e65296914602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b60008173__FFd1f58a808ab6001133eA3a60e07f4991E6012e__6350785a4b90916040518263ffffffff1660e01b81526004018082815260200191505060206040518083038186803b15801560be57600080fd5b505af415801560d1573d6000803e3d6000fd5b505050506040513d602081101560e657600080fd5b8101908080519060200190929190505050905091905056fea2646970667358221220f42ce135fbefe1ffc4eefc7bc0320c33dbdf257910573c611e2adf7c738edb8b64736f6c63430006010033
```

underbar를 제거하고 실제로 사용할 (library의 주소가 포함된) bytecode이다.
```python
608060405234801561001057600080fd5b50610134806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633e65296914602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b60008173FFd1f58a808ab6001133eA3a60e07f4991E6012e6350785a4b90916040518263ffffffff1660e01b81526004018082815260200191505060206040518083038186803b15801560be57600080fd5b505af415801560d1573d6000803e3d6000fd5b505050506040513d602081101560e657600080fd5b8101908080519060200190929190505050905091905056fea2646970667358221220f42ce135fbefe1ffc4eefc7bc0320c33dbdf257910573c611e2adf7c738edb8b64736f6c63430006010033
```

### 단계 3: 배포

위에서 작성된 ```LibraryTestPublicLink.json```의 abi, bin을 사용하여 배포하자.


```python
[파일명: src/LibraryTestPublicDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
//var _str = fs.readFileSync("src/LibraryTestPublicLink__.json");
var _str = fs.readFileSync("src/LibraryTestPublicLink.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/LibraryTestPublic.sol:LibrayTestPublic"].abi);
var _abiArray = _json.contracts["src/LibraryTestPublic.sol:LibrayTestPublic"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/LibraryTestPublic.sol:LibrayTestPublic"].bin;

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
pjt_dir> node src/LibraryTestPublicDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x23d4988819361344fa12399a5f89ee778ae437a17abd39222564f1017b2ff593
---> The contract deployed to: 0x27B4E66C995AB06bF7cB20F26AfCacA654C3914B
```

### 단계 4: 사용


```python
[파일명: src/LibraryTestPublicUse.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
//var _str = fs.readFileSync("src/LibraryTestPublicLink__.json");
var _str = fs.readFileSync("src/LibraryTestPublicLink.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/LibraryTestPublic.sol:LibrayTestPublic"].abi);
var _abiArray = _json.contracts["src/LibraryTestPublic.sol:LibrayTestPublic"].abi;

async function doIt() {
    var lib = new web3.eth.Contract(_abiArray, "0x27B4E66C995AB06bF7cB20F26AfCacA654C3914B");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    lib.methods.multiply7By(5).estimateGas(function(err,gas) {
        if(!err) console.log(">> gas: "+ gas);
    });
    lib.methods.multiply7By(5).call().then(console.log);
}

doIt()

```


```python
pjt_dir> node src/LibraryTestPublicUse.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
35
>> gas: 25832
```

## 3.5 외부 라이브러리

외부의 제3자가 제공하는 라이브러리 당연히 사용할 수 있다.
인터넷을 찾아 보면 꽤 많은 라이브러리가 제공되고 있다는 것을 알 수 있다.
그 중에서 OpenZeppelin을 꼽을 수 있는데 상당한 라이브러리를 제공하고 있다.
* 그리고 Modular network (https://github.com/modular-network/ethereum-libraries)에서도 ArrayUtils, BasicMath, CrowdSale, LinkedList, StringUtils, Token, Vesting, Wallet 관련 라이브러리가 제공되고 있다.
* 또한 Dapp-bin by Ethereum https://github.com/ethereum/dapp-bin: IterableMapping, DoublyLinkedList, StringUtils


## 실습: OpenZeppelin의 SafeMath를 사용해서 정수 연산

https://github.com/OpenZeppelin/openzeppelin-contracts
사이트를 가보면, 유용한 라이브러리를 만나게 된다.
이 가운데 연산 기능을 제공하는 SafeMath.sol을 사용해보자.
https://github.com/OpenZeppelin/openzeppelin-contracts 의 ```contracts/utils/math```로 이동해보자.

### 단계 1: 컨트랙 개발

컴파일은 0.6 이상에서 하고, 함수는 ```internal```로 선언되어 있다.


```python
[파일명: src/SafeMath.sol]
pragma solidity ^0.6.0;
library SafeMath {
    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
       if (a == 0) {
            return 0;
        }
        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");
        return c;
    }

    function div(uint256 a, uint256 b) internal pure returns (uint256) {
        return div(a, b, "SafeMath: division by zero");
    }

    function div(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b > 0, errorMessage);
        uint256 c = a / b;
        return c;
    }

    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        return sub(a, b, "SafeMath: subtraction overflow");
    }

    function sub(uint256 a, uint256 b, string memory errorMessage) internal pure returns (uint256) {
        require(b <= a, errorMessage);
        uint256 c = a - b;
        return c;
    }

    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
}
```

```python
[파일명: src/TestSafeMath.sol]
pragma solidity ^0.6.0;

import "./SafeMath.sol";

contract TestSafeMath {   
    using SafeMath for uint256;    
    function add(uint256 x, uint256 y) public pure returns (uint256) {
        uint256 z = x.add(y);
        return z;
    }
}
```



### 단계 2: 컴파일


```python
pjt_dir> solc src/TestSafeMath.sol --combined-json abi,bin > src/TestSafeMath.json
```

```python
pjt_dir> cat src/TestSafeMath.json

{"contracts":{"src/SafeMath.sol:SafeMath":{"abi":"[]","bin":"60566023600b82828239805160001a607314601657fe5b30600052607381538281f3fe73000000000000000000000000000000000000000030146080604052600080fdfea26469706673582212207b5b2523e13fcae7ff82d2c9fb39b280a5193c67b625b428e8431011c83852c764736f6c63430006010033"},"src/TestSafeMath.sol:TestSafeMath":{"abi":"[{\"inputs\":[{\"internalType\":\"uint256\",\"name\":\"x\",\"type\":\"uint256\"},{\"internalType\":\"uint256\",\"name\":\"y\",\"type\":\"uint256\"}],\"name\":\"add\",\"outputs\":[{\"internalType\":\"uint256\",\"name\":\"\",\"type\":\"uint256\"}],\"stateMutability\":\"pure\",\"type\":\"function\"}]","bin":"608060405234801561001057600080fd5b5061015c806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063771602f714610030575b600080fd5b6100666004803603604081101561004657600080fd5b81019080803590602001909291908035906020019092919050505061007c565b6040518082815260200191505060405180910390f35b600080610092838561009e90919063ffffffff16565b90508091505092915050565b60008082840190508381101561011c576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601b8152602001807f536166654d6174683a206164646974696f6e206f766572666c6f77000000000081525060200191505060405180910390fd5b809150509291505056fea264697066735822122029d9995da2da645f9948d5c17455f769ea56dabb7ec2c7cc98a55ff6dfa47bc164736f6c63430006010033"}},"version":"0.6.1+commit.e6f7d5a4.Linux.g++"}
```

### 단계 3: 배포

TestSafeMath.sol의 abi, bin을 생성하고, 객체를 생성하는데 넣어준다.


```python
[파일명: src/TestSafeMathDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/TestSafeMath.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
var _abiArray = JSON.parse(_json.contracts["src/TestSafeMath.sol:TestSafeMath"].abi);
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/TestSafeMath.sol:TestSafeMath"].bin;

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
pjt_dir> node src/TestSafeMathDeploy.js

Deploying the contract from 0xA87e9e49512f32c345e34686a9897e4c06C13A87
hash: 0x1b14656d5b0a98c049d4ee285a574f9abde07b0b78be6ce13722c43640b53673
---> The contract deployed to: 0x520E0d4e4254518600F027de3439f756e74E5809
```

### 단계 4: 사용


```python
[파일명: src/TestSafeMathUse.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/TestSafeMath.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
var _abiArray = JSON.parse(_json.contracts["src/TestSafeMath.sol:TestSafeMath"].abi);

async function doIt() {
    var lib = new web3.eth.Contract(_abiArray, "0x520E0d4e4254518600F027de3439f756e74E5809");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    lib.methods.add(11,20).estimateGas(function(err,gas) {
        if(!err) console.log(">> gas: "+ gas);
    });
    lib.methods.add(11,20).call().then(console.log);
}

doIt()

```


```python
pjt_dir> node src/TestSafeMathUse.js

Account: 0xA87e9e49512f32c345e34686a9897e4c06C13A87
>> gas: 22084
31
```

## 실습: OpenZeppelin를 설치하고 SafeMath 사용

라이브러리 소스코드가 github에 있는 경우는 다운로드 받아서, 노드에서 설치할 수 있으면 설치해서 사용하면 된다.

###  단계 1: 컨트랙 개발

* **설치**
노드에서 ```npm install @openzeppelin/contracts```를 설치하면,
```node_modules/@openzeppelin/contracts/math``` 아래 디렉토리에 설치되어 있다.

* **import 문**
```import "@openzeppelin/contracts/math/SafeMath.sol";```
이 파일은 ```pragma solidity ^0.5.0;``` 버전이 다르기 때문에 맞추어 주어야 한다.
위 ```@openzeppelin/contracts/math/SafeMath.sol";```는 인식이 되지 않으므로
현재 컴파일하는 파일 기준의 **상대경로**를 넣어준다.

* **경로 재지정 및 접근경로 제한 풀어주기**
상대경로로 지정하였으므로, solc에서는 보통 해주던 것처럼 하면된다.
단 그 경로를 사용하는 것이 보안상 제한되므로 풀어준다.
solc 컴파일하는 소스코드가 있는 경로, **재지정된 경로 외에는 접근 할 수 없도록 보안 제한**을 하게 된다.
```--allow-paths /sample/path,/another/sample/path``` 스위치를 넣어준다.


```python
[파일명: src/TestSafeMathZeppelin.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;
//pragma solidity ^0.6.0;

import "../node_modules/@openzeppelin/contracts/utils/math/SafeMath.sol";
//import "github.com/OpenZeppelin/openzeppelin-contracts/contracts/math/SafeMath.sol";

contract TestSafeMath {   
    using SafeMath for uint256;
    function add(uint256 x, uint256 y) public pure returns (uint256) {
        uint256 z = x.add(y);
        return z;
    }
}

```


### 단계 2: 컴파일
보안의 이유로 Solidity는 디렉토리 접근 권한을 제한하고 있다. ```Error: Source "@openzeppelin/contracts/math/SafeMath.sol" not foundFile outside of allowed directories.```. 이를 허용하기 위해 **```--allow-paths```**를 넣어준다. 뒤에 절대경로를 넣어주면 된다.

```SafeMath``` 경로로 ```import```문에 사용되었던 경로 ```node_modules/@openzeppelin/contracts/math/SafeMath.sol:SafeMath```가 쓰이고 있다는 것을 발견할 수 있다.


```python
pjt_dir> solc-windows.exe src/TestSafeMathZeppelin.sol --allow-paths . --combined-json abi,bin > src/TestSafeMathZeppelin.json
```


```python
pjt_dir> type src\TestSafeMathZeppelin.json

{"contracts":{"node_modules/@openzeppelin/contracts/utils/math/SafeMath.sol:SafeMath":{"abi":[],"bin":"60566050600b82828239805160001a6073146043577f4e487b7100000000000000000000000000000000000000000000000000000000600052600060045260246000fd5b30600052607381538281f3fe73000000000000000000000000000000000000000030146080604052600080fdfea264697066735822122068115c4a0549c3053713c9a89288ba4e8cc1583d887f1810d41b7081d5e630df64736f6c63430008010033"},"src/TestSafeMathZeppelin.sol:TestSafeMath":{"abi":[{"inputs":[{"internalType":"uint256","name":"x","type":"uint256"},{"internalType":"uint256","name":"y","type":"uint256"}],"name":"add","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}],"bin":"608060405234801561001057600080fd5b506101ef806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063771602f714610030575b600080fd5b61004a600480360381019061004591906100ad565b610060565b60405161005791906100f8565b60405180910390f35b600080610076838561008290919063ffffffff16565b90508091505092915050565b600081836100909190610113565b905092915050565b6000813590506100a7816101a2565b92915050565b600080604083850312156100c057600080fd5b60006100ce85828601610098565b92505060206100df85828601610098565b9150509250929050565b6100f281610169565b82525050565b600060208201905061010d60008301846100e9565b92915050565b600061011e82610169565b915061012983610169565b9250827fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff0382111561015e5761015d610173565b5b828201905092915050565b6000819050919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b6101ab81610169565b81146101b657600080fd5b5056fea264697066735822122026ae744faf917dbc92a530ca49764b2943c5261816e8b6f0f38173d35167bdbe64736f6c63430008010033"}},"version":"0.8.1+commit.df193b15.Windows.msvc"}
```

### 단계 3: 배포

위 ```TestSafeMathZeppelin```의 abi, bin을 사용해서, 객체를 생성하는데 넣어준다.


```python
[파일명: src/TestSafeMathZeppelinDeploy.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/TestSafeMathZeppelin.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//var _abiArray = JSON.parse(_json.contracts["src/TestSafeMathZeppelin.sol:TestSafeMath"].abi);
var _abiArray = _json.contracts["src/TestSafeMathZeppelin.sol:TestSafeMath"].abi;
//var _bin = _json.contracts.sHello2.bin;
var _bin = "0x" + _json.contracts["src/TestSafeMathZeppelin.sol:TestSafeMath"].bin;

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
pjt_dir> node src/TestSafeMathZeppelinDeploy.js

Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
hash: 0x3c212353dc911c8a84d0f053f6f64d4cb04f34c64c5ebab78944b6919be07d51
---> The contract deployed to: 0x235C5eb4C4ec5A0FB30e4CdAA8AA28f9812fF67D
```

### 단계 4: 사용


```python
[파일명: src/TestSafeMathZeppelinUse.js]
var Web3 = require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var fs = require('fs');
var _str = fs.readFileSync("src/TestSafeMathZeppelin.json");
var _json = JSON.parse(_str)
//var _abiArray = JSON.parse(_json.contracts.sHello2.abi);
//|var _abiArray = JSON.parse(_json.contracts["src/TestSafeMathZeppelin.sol:TestSafeMath"].abi);
var _abiArray = _json.contracts["src/TestSafeMathZeppelin.sol:TestSafeMath"].abi;

async function doIt() {
    var lib = new web3.eth.Contract(_abiArray, "0x235C5eb4C4ec5A0FB30e4CdAA8AA28f9812fF67D");
    const accounts = await web3.eth.getAccounts();
    console.log("Account: " + accounts[0]);
    lib.methods.add(11,20).estimateGas(function(err,gas) {
        if(!err) console.log(">> gas: "+ gas);
    });
    lib.methods.add(11,20).call().then(console.log);
}

doIt()

```



```python
pjt_dir> node src/TestSafeMathZeppelinUse.js

Account: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
31
>> gas: 22367
```

## 실습: 제삼자가 제공하는 github 라이브러리

```github.com``` 소스를 직접 컴파일하는 것은 안된다. REMIX는 소스를 다운로드 받아서 사용하므로, 아래 같은 매핑을 사용하지 않아도 된다.
```solc``` 컴파일할 할 경우, ```github``` 소스코드를 ```import```하고 컴파일할 경우에는 내려받아서 사용하고 그 디렉토리에 사용허가 권한을 부여해야 한다. 

예를 들어, ```github.com/ethereum/dapp-bin/``` 를 사용한다고 하자.
* **단계 1. import 문**
소스에는 다음과 같이 ```github.com```까지 포함해서 직접 적어준다. ```https://``` 또는 ```blob/master``` 같은 용어는 적지 않는다.
```python
import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;
```
원래의 URL은 ```https://github.com/ethereum/dapp-bin/blob/master/library/stringUtils.sol```이다. 여기서 ```https://```와 ```blob/master/``` 등을 제거하고 실제 ```github.com/ethereum/dapp-bin/library/stringUtils.sol```을 적어준다.


* **단계 2. 경로 재지정 remapping**

```solc``` 컴파일할 때 ```github```에서 직접 파일을 가져오지 못한다. 해당 github repo를 clone하고 재지정해주어야 한다.
* 단계 2-1: 소스를 git clone한다. 예를 들어, ```github.com/ethereum/dapp-bin/``` 를 ```/usr/local/dapp-bin```으로 clone한다
* 단계 2-2: 소스의 URL을 로컬디렉토리로 대체해 주고 컴파일할 때는 다음과 같이 적어준다.

```python
solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol
```
```import``` 문에 쓰인 경로를 재지정하여 파일을 읽을 수 있게 하는 것으로,
```github.com/ethereum/dapp-bin/``` 으로 **시작하는 선행 경로 ```prefix```**를 ```/usr/local/lib/dapp-bin/```로 **로컬 경로 ```path```**로 변경하고,
단순히 해당 파일을 찾는 디렉토리를 재지정하는 것이다. 즉 ```prefix=path```로 재지정 remapping한다.


## 연습문제: 문자열 라이브러리

Solidity는 문자열 라이브러리를 제공하지 않아서, 문자열을 자르고, 비교하는 작업을 하기 불편하다.
github에 배포한 라이브러리를 사용해서, 문자열 비교를 해보자.
또는 NPM에서도 제공하고 있으니 설치해도 된다 ```npm intall solidity-stringutils```


```python
[파일명: src/TestStringUtils.sol]
pragma solidity 0.6.1;

//import "github.com/ethereum/dapp-bin/library/stringUtils.sol";
//import "./StringUtils.sol";
import "github.com/Arachnid/solidity-stringutils/strings.sol";

contract TestStringUtils {
    using strings for *;
    function foo() public returns (bool){
        string memory a = "hello";
        string memory b = "world";
        return a.equal(a);
    }
}
```


### import

Remix IDE 에서는 직접 github.com에 배포된 제3자 라이브러리를 사용할 수 있다.

또는 로컬에 다운로드 한 후 사용해도 된다.

```python
git clone https://github.com/Arachnid/solidity-stringutils.git
```


```python
[파일명: src/StringUtils.sol]
library StringUtils {
    /// @dev Does a byte-by-byte lexicographical comparison of two strings.
    /// @return a negative number if `_a` is smaller, zero if they are equal
    /// and a positive numbe if `_b` is smaller.
    function compare(string _a, string _b) returns (int) {
        bytes memory a = bytes(_a);
        bytes memory b = bytes(_b);
        uint minLength = a.length;
        if (b.length < minLength) minLength = b.length;
        //@todo unroll the loop into increments of 32 and do full 32 byte comparisons
        for (uint i = 0; i < minLength; i ++)
            if (a[i] < b[i])
                return -1;
            else if (a[i] > b[i])
                return 1;
        if (a.length < b.length)
            return -1;
        else if (a.length > b.length)
            return 1;
        else
            return 0;
    }
    /// @dev Compares two strings and returns true iff they are equal.
    function equal(string _a, string _b) returns (bool) {
        return compare(_a, _b) == 0;
    }
    /// @dev Finds the index of the first occurrence of _needle in _haystack
    function indexOf(string _haystack, string _needle) returns (int)
    {
    	bytes memory h = bytes(_haystack);
    	bytes memory n = bytes(_needle);
    	if(h.length < 1 || n.length < 1 || (n.length > h.length)) 
    		return -1;
    	else if(h.length > (2**128 -1)) // since we have to be able to return -1 (if the char isn't found or input error), this function must return an "int" type with a max length of (2^128 - 1)
    		return -1;									
    	else
    	{
    		uint subindex = 0;
    		for (uint i = 0; i < h.length; i ++)
    		{
    			if (h[i] == n[0]) // found the first char of b
    			{
    				subindex = 1;
    				while(subindex < n.length && (i + subindex) < h.length && h[i + subindex] == n[subindex]) // search until the chars don't match or until we reach the end of a or b
    				{
    					subindex++;
    				}	
    				if(subindex == n.length)
    					return int(i);
    			}
    		}
    		return -1;
    	}	
    }
}
```


```python
pjt_dir> solc --abi --bin github.com/Arachnid/solidity-stringutils/=/home/jsl/Code/git/gh/solidity-stringutils/ src/TestStringUtils.sol --allow-paths .

Error: Source file requires different compiler version (current compiler is 0.6.1+commit.e6f7d5a4.Linux.g++ - note that nightly builds are considered to be strictly less than the released version
 --> /home/jsl/Code/git/gh/solidity-stringutils/strings.sol:37:1:
  | 37 | pragma solidity ^0.4.14;
  | ^^^^^^^^^^^^^^^^^^^^^^^^
```

 버전을 변경해서 ```pragma solidity ^0.4.14 -->  pragma solidity ^0.6;``` 컴파일해보자.

```python
[파일명: src/TestStringUtils.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
}
var _abiArray=[{"inputs":[{"internalType":"uint256","name":"num","type":"uint256"}],"name":"multiply7By","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"pure","type":"function"}];
var _bin="608060405234801561001057600080fd5b50610134806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633e65296914602d575b600080fd5b605660048036036020811015604157600080fd5b8101908080359060200190929190505050606c565b6040518082815260200191505060405180910390f35b6000817357b9ebe354dcb1263237d3fd5c87ab262b2944456350785a4b90916040518263ffffffff1660e01b81526004018082815260200191505060206040518083038186803b15801560be57600080fd5b505af415801560d1573d6000803e3d6000fd5b505050506040513d602081101560e657600080fd5b8101908080519060200190929190505050905091905056fea26469706673582212204db82013c693a29f1d853cd0f12f599ed38bc943b4ae865891c580aa85679daa64736f6c63430006010033";
var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
_contract
    .deploy({data:"0x"+_bin})
    .send({from:'0x051cbe7561112cbe9a357cd246d9da343a6115c4',gas:1000000}, function(err, transactionHash) {
        if (!err) {
            console.log(">>> transactionHash: ", transactionHash);
        } else { console.log(err);}
    })
    .then(function(contract) {
        console.log(">>> Deployed address: "+contract._address);
    })
    .catch(function(e) {
        console.log("error",e);
    });
```

