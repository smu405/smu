# Chapter  7. 맛보기 웹디앱 프로젝트

Hello 프로젝트를 구현한다. 서버 측을 Solidity로 스마트컨트랙을 만들고, 클라이언트 단에서 웹호출까지 구현한다.
매우 단순한 Greeter, Counter를 구현하면서 '아!  블록체인으로 이렇게 만드는구나'를 이해하도록 하자.

# 1. Hello World 디앱

프로그래밍을 배우면서 보통 처음으로 "Hello World"를 출력하게 된다. 간단한 프로그램이지만 코딩, 컴파일, 실행, 테스트의 전과정을 해보면서 그 언어의 특징을 보다 잘 이해하게 된다.

블록체인 프로그래밍은 다른 프로그래밍과 다를 수 밖에 없지만, 코딩, 컴파일, 테스트 등 이러한 절차는 필수적이고, C또는 자바로 해도 별 다르지 않다. 단계별로 하는 작업을 설명하면:
- 코딩은 자바와 비슷한 Solidity로 하고,
- 소스코드는 컴파일해서 ABI Application Binary Interface와 Byte code를 얻어야 한다. ABI는 컨트랙을 사용하기 위해 인터페이스 표준으로, JSON으로 표현된다. 또한 Byte code는 Solidity 소스코드를 컴파일한 기계어인 바이트코드이다.
- 컴파일한 후 배포를 하는데, 블록체인에 올려야 한다. 이 배포와 같은 작업은 독특하다. 네트워크에 떠 있는 분산 블록체인에 배포하는 작업이 필수적이고, 메모리 번지와 같은 주소가 주어진다, 
- 마지막으로, 비로서 배포주소를 통해 블록체인의 API를 사용할 수 있게 된다. API를 사용하려면, 앞서 설명하고 있는 블록체인의 특성을 이해해야 하는 것이다. 거래에 디지털 서명을 하고, 마이닝되고, 블록체인으로 만들어지게 된다.

단계 | 구분 | 설명
-----|-----|-----
1 | 개발 | 컨트랙 **소스 코드** 개발
2 | 컴파일 | 소스코드를 컴파일해서 **ABI**, **Byte code** 생성
3 | 컨트랙 배포 | 컴파일해서 얻은 ABI, Byte code로 객체를 생성한 후 배포 요청, 마이닝해서 블록체인의 주소 **contract address** 획득
4 | 사용 | **ABI**, **contract address**로 컨트랙 객체를 생성해서, ```call()``` 또는 ```sendTransaction()``` 함수로 API 호출.

## 3.1 단계 1: 컨트랙 개발

물론 그냥 많이 쓰이는 메모장 등의 편집기를 사용해도 되지만, 컨트랙은 REMIX에서 개발해보자.

또 새로운 도구가 등장하여 복잡하겠지만, **REMIX**는 온라인 http://remix.ethereum.org/ 에서 제공되는 Solidity 통합개발환경 (IDE, Integrated Development Environment)을 말한다. 이 도구를 열 때는, HTTP WebProvider를 사용하기 위해서 https 보다는 http연결을 하자.

REMIX는 보통 개발환경에서 제공하는 컴파일, 디버깅, 편집, 배포, 테스트 등의 기능을 제공한다. 온라인에서 사용하는 것이 불편하다면, https://github.com/ethereum/remix-live/tree/gh-pages로 가서 zip파일을 내려받아 자신의 컴퓨터에 설치하여 사용할 수 있다.

또한 github과 연동하여 사용할 수 있다. ```&gist=여기에 id```를 URL에 넣어서 전송하여, gist로 소스코드를 내보내거나 읽을 수 있다.

Solidity가 처음이라 명령어가 낯설겠지만, 객체지향언어이므로 클래스를 코딩하는 것과 같이 시작을 한다.
지금은 한 사이클을 완성하기 위해서 하는 코딩이다. 명령어는 차츰 배워나가게 된다.
REMIX에서 테스트해보고 요구사항대로 작동하면 코드를 복사해서 저장한다.

줄 | 설명
-----|-----
1 | 메이저 버전 0.6이상 가운데, 최신 버전으로 컴파일. 앞의 햇 ```^```표시가 그런 의미이다.
2 | 컨트랙 명칭을 Hello로 정함
3 | ```sayHello()``` 함수, string으로 지역변수 값을 반환하는 단순 조회
4 | 문자열 ```Hello World``` 반환

```python
[파일명: src/Hello.sol]
pragma solidity ^0.8.0;
contract Hello {
    function sayHello() pure public returns(string memory) {
        return "Hello World";
    }
}
```

## 3.2 단계 2: 컴파일

소스코드를 컴파일하는 목적은 (1) ABI와 (2) bytecode를 얻기 위한 것이다.

컴파일은 (1) REMIX를 사용하거나, (2) solc 컴퍼알로를 사용하면 된다.

### REMIX 컴파일

REMIX에서 버전 0.8.0으로 컴파일 해보자.
* ```Hello.sol```을 코딩한 후,
* ```compile``` 버튼을 누른다.
* ```Compilation Details``` 버튼을 누르면, ABI와 bytecode가 생성되어 있다.
또는 ```WEB3DEPLOY```에 ABI, bytecode가 포함된 코드를 발견할 수 있다.
주의할 점은 ```WEB3DEPLOY```가 우리가 사용하는 **web3.js 버전을 지원**하는지의 여부이다.

![alt text](figures/6_helloWeb3Remix.png "geth download page")

### solc compile

설치해 놓은 solc 컴파일러를 실행하여 ABI, bytecode를 생성할 수 있다.

sayHello()의 gas가 ```infinite```라고 하는 것은 반드시 무한대로 필요하다는 의미는 아니다.
(```solc --asm src/Hello.sol```로 컴파일을 하면 backward jump 문이 포함되어 있는 경우 무한대로 계산)

먼저 컴파일러가 설치되었는지 확인하자. 운영체제와 그 버전에 따라 명령어가 다르다. 내려받아 설치하면서 solc, solc-windows인지 분별하여 사용하자. 명령창 프롬프트의 프로젝트 디렉토리 ```pjt_dir> ```에서 확인해보자.

```python
pjt_dir> solc --version         이전의 리눅스, 윈도우 가리지 않고 모두 solc 명령어 사용
solc, the solidity compiler commandline interface
Version: 0.6.1+commit.e6f7d5a4.Windows.msvc

pjt_dir> solc-windows --version  최근 윈도우 최근 버전은 solc-windows 명령어 사용
solc, the solidity compiler commandline interface
Version: 0.8.1+commit.df193b15.Windows.msvc
```

ABI는 컨트랙과 인터페이스하기 위한 표준으로 JSON으로 출력된다. 아래의 abi를 보자. 
```sayHello()``` 함수의 입력 ```inputs```, 출력 ```outputs```를 정의하고 ```stateMutability```가 ```pure```, ```view```, ```non-payable```, ```payable``` 인지 정의하고 있다.
이전에는 ```constant```, ```payable```이 사용되었지만 지금은 제거되어 ```stateMutability```로 대체된다.

```python
pjt_dir> solc --abi --bin --gas src/Hello.sol
    
    ======= src/Hello.sol:Hello =======
    Gas estimation:
    construction:
       105 + 57200 = 57305
    external:
       sayHello():	infinite
    Binary:
    608060405234801561001057600080fd5b5061011e806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063ef5fb05b14602d575b600080fd5b603360ab565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101560715780820151818401526020810190506058565b50505050905090810190601f168015609d5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040518060400160405280600b81526020017f48656c6c6f20576f726c6400000000000000000000000000000000000000000081525090509056fea26469706673582212208ff7c230fefd3d4a66a11d93dd077f556120f8054ee595e3f77349648b8460e064736f6c63430006010033
    Contract JSON ABI
    [{"inputs":[],"name":"sayHello","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}]
```

### 언어에 내장된 solc

geth 1.6부터는 eth_compilers, eth_compileSolidity 지원이 중단되었다.

```python
geth> eth.getCompilers()
Error: The method eth_getCompilers does not exist/is not available
```

다른 언어에서 내장형으로 컴파일 할 수 있다. node solc, py-solc 등이 제공되고 있다. 한 줄씩 실행하는 Solidity REPL 기능도 설치하여 사용할 수 있다. 우리는 사용하지 않기로 한다.
```python
npm install solidity-repl
```

## 3.3 단계 3: 배포

컨트랙 배포의 목적은 주소이다.  컴파일하고 구한 ABI, bytecode를 사용해 배포된 컨트랙의 블록체인 주소를 얻는 것이다.

필요한 절차는:
* (3-1) 앞 2단계의 컴파일을 하고 얻은 **ABI, byteode를 사용해서 contract 객체를 생성**하고,
* (3-2) 그 객체의 배포를 ```deploy()``` 함수로 요청하고 ```transactionHash```를 얻고,
* (3-3) 마이닝을 하고 나면 ```blockchain```에서의 주소 ```contractAddress```를 받게 된다.

### 단계 3-1: contract 생성

```new()``` 명령어로 객체를 생성한다. 이 때 ABI가 필수적으로 필요하고, address, options은 꼭 그렇지 않다.

```python
new web3.eth.Contract(abi [, address] [, options])
```

* abi: 컨트랙에 대한 json 인터페이스
* address: 컨트랙이 배포된 블록체인 주소를 말하며, 최초 배포할 때는 당연히 없으므로 비워놓는다. 배포된 컨트랙을 사용할 경우 해당 주소를 적어주면 된다.
* options: from, gasPrice, gas, data (컨트랙의 바이트코드)

코드로 표현해보자. ABI, 바이트코드를 추가해서 컨트랙을 생성할 수 있다. 아직 _abi, _bin 변수가 정의가 되지 않아서 실행되지 않는다.
```python
node> var myGreeter = new web3.eth.Contract(_abi, {data:'0x'+_bin});
```

### 단계 3-2 배포요청

#### ```deploy()```
컨트랙을 배포하려면 ```deploy()``` 함수를 사용하면 해당 컨트랙을 블록체인에 배포하게 된다.

```python
myGreeter.deploy(options)
```

options에 설정할 수 있는 입력은 data와 arguments를 포함한다.
* data: 컨트랙의 바이트코드
* arguments: 컨트랙 생성자는 인자를 가질 수 있다. 그 인자를 넣고, 없으면 생략한다.

#### ```send()```

컨트랙을 배포할 경우, 물론 블록체인에 전송해야 한다.
```deploy()``` 함수에 연결하여 ```send()``` 함수를 적어준다.

```python
send(options [, callback])
```

* options에는 ```from```, ```gasPrice```, ```gas```, ```value```
* callback함수는 앞의 함수 ```send()```가 실행된 후 실행이벤트 ('transactionHash' 등)에 따라 callback된다. 첫 인자는 'error'를 넣어주어야 한다. callback 함수는 **PromiEvent를 반환**한다. ```PromiEvent```에 ```on```, ```once```, ```off``` 함수를 연결 chaining하여 이벤트를 처리할 수 있다.


### 비동기함수에 대한 Promise Events

web3.js에서의 함수는 promise를 반환한 다음 연결함수 ```on```, ```once```, ```off```를 통해 이벤트 event를 발생할 수 있다. 사용할 수 있는 PromiEvent는:
* ```transactionHash```: 거래가 전송된 직후 transction hash가 발생하는 이벤트 발생
* ```receipt```: transactionReceipt이 만들어지는 이벤트 발생
* ```confirmation``` 매 24번째 confirmation 마다 이벤트 발생
* ```error```: 오류에 대해 이벤트 발생

예를 들어, 이벤트가 발생하면 그 이벤트를 출력하는 명령문이다.
```
web3.eth.sendTransaction({from: '0x123...', data: '0x432...'})
.once('sending', function(payload){ ... })
.once('sent', function(payload){ ... })
.once('transactionHash', function(hash){ ... })
.once('receipt', function(receipt){ ... })
.on('confirmation', function(confNumber, receipt, latestBlockHash){ ... })
.on('error', function(error){ ... })
.then(function(receipt){
    // 마이닝이 끝나면 실행되는 코드
});
```

### 단계 3-3 주소 출력에 필요한 ```then()```

배포를 하는 이유는 무엇일까? 주소를 알아내는 것이다. 블록체인에 배포된 주소가 있어야, 그 주소를 통해 그 컨트랙을 호출할 수 있기 때문이다.

그러나 deploy() 함수는 비동기적인 특징이 있으므로, 배포하고 나면 바로 주소가 주어지는 것이 아니라 대신 앞서 설명한 Promise가 주어진다. 이를 해소해야만 비로서 주소를 얻을 수 있게 된다.

### 3단계 실행에 필요한 코드 종합: Web3 0.20 버전

web3의 어떤 버전을 사용하는지 유의한다. 버전이 다르면 당연히 문제가 되기 때문이다. 사용하는 버전이 0.20 버전이 아니면 아래 내용은 건너뛰어도 된다.

아래 ```HelloDeploy__.js```는 이전 버전의 문법을 사용하기 때문에 web3.js 1.0이상 버전에서는 실행되지 않는다 (앞으로 web3.js 0.20.x에서 실행하는 파일명 뒤에는 underscore를 붙임).

실행하기 전, geth를 띄워놓아서 코드를 실행하면 어떤 변화가 발생하는지 지켜보는 것이 좋다. 포트번호만 다르게 해서 ganache도 같이 띄워 놓자.

```python
[파일명: src/HelloDeploy__.js]  이전 버전 web3 0.20.x의 문법으로 작성
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var helloContract = web3.eth.contract([{"inputs":[],"name":"sayHello","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}]);
var hello = helloContract.new(
   {
     from: web3.eth.accounts[0], 
     data: '0x608060405234801561001057600080fd5b5061011e806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063ef5fb05b14602d575b600080fd5b603360ab565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101560715780820151818401526020810190506058565b50505050905090810190601f168015609d5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040518060400160405280600b81526020017f48656c6c6f20576f726c6400000000000000000000000000000000000000000081525090509056fea2646970667358221220b6877a972a9ca7f396c5cb7c0a7ae50d7532a82ded3789e90afa4b3e108a631364736f6c63430006010033', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

위 프로그램은 web3 0.20.x 버전의 문법으로 작성되었고, 버전업이 되면서 명령어가 부분적으로 변경되어야 한다.
현재 web3 버전으로 실행하면 당연히 오류가 발생하게 된다.

```python
pjt_dir> node src/HelloDeploy__.js
...생략... 
TypeError: web3.eth.contract is not a function 현재 web3 1.0 이상에서 실행하면 오류 발생
...생략...
```

### 3단계 실행에 필요한 코드 종합: Web3 1.0 이상 버전

web3.js 최신버전에서 실행이 되도록 작성해 주자.

#### 계정 읽기

배포하려면 계정 주소가 필요하다. 계정을 읽어 보자. web3 0.20.x에서는 계정배열의 인덱스로 쉽게 얻을 수 있었다.

잠깐! **geth console에서는 배열 인덱스로 가능**하다. 지금은 **노드에서 실행했을 경우는 배열 인덱스가 더 이상 지원되지 않고** 있다.

```python
node> var address0 = web3.eth.accounts[0];  //이렇게 읽어올 수 없다.
```

web3.js 1.0 이상에서는 비동기방식으로 계정을 읽어오며, promise가 반환이 된다. 비동기방식으로 계정 주소를 넣어주기는 불편하므로, 일단 하드코딩해서 넣자.

```python
[파일명: src/getMyAddr.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"))
//var myAddr0;
web3.eth.getAccounts().then(console.log);
```

실행하고 출력된 계정은 자신이 띄어놓은 geth 또는 ganache의 것과 일치하는지 확인하자.
```python
pjt_dir> node src/getMyAddr.js

[ '0xEb670169A5fcD2550035588A2ba3b45509703ABd',
  '0xF724CDbfCf9C94f28547b1111B3A6E3672a9aDCE',
   ...생략...
  '0xe04852Cb018778545e75CC21EEC1849DdCdFa79f' ]
```

#### gas 계산
다음으로 gas도 필요하므로, 계산해보자. 이를 위해서는 data를 입력해 주면 그 바이트 크기와 Opcode에 따라 계산된다. gas를 산정하는 함수는 ```estimateGas()```이다.

컨트랙을 생성하는 코드가 이전 web3 0.20.x 버전과 비교해서 다른데, ```new``` 명령어를 사용하고 있다 (자바의 객체를 생성하는 코드와 비슷하다).

node 창을 열어서 한 줄씩 해도 물론 된다. 그러나 코드를 보라! 한 줄씩 입력하기에는 코드가 복잡하고, 복사-붙여넣기를 상당히 해야 한다.

명령어를 파일로 저장하고, 일괄실행하는 방식으로 하자.

단계 2 컴파일에서 생성된 abi, bytecode와 또한 계정주소도 복사, 붙여넣기를 하고 있다. 불편하지만 현재는 그렇게 하자. 무엇을 어디에서 가져와 어디에 입력하는지 이해하는데 도움이 될 것이다. 나중에 코드로 처리하는 방법을 설명하게 된다.

자바스크립트 도움말을 중간에 적어 놓았다.

```python
[파일명: src/HelloDeployGas.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
// 생성한 abi를 복사-붙여넣기로 입력한다
var shelloContract = new web3.eth.Contract([{"inputs":[],"name":"sayHello","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}]);
// 계정 address와 data도 복사-붙여넣기 한다. 자신의 개발환경에 따라 계정, 데이터는 값이 다를 수 있다.
shelloContract.deploy({
        from: '0x4aae75084f715390Aad4a251DC70327AfEf8a03c',
        data: '0x608060405234801561001057600080fd5b50610139806100206000396000f3fe608060405234801561001057600080fd5b5060043610610048576000357c010000000000000000000000000000000000000000000000000000000090048063ef5fb05b1461004d575b600080fd5b6100556100d0565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009557808201518184015260208101905061007a565b50505050905090810190601f1680156100c25780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040805190810160405280600b81526020017f48656c6c6f20576f726c6400000000000000000000000000000000000000000081525090509056fea165627a7a72305820992c2f4c73a27b9eb53c6aa7b52ba8a5eddba258089eb7d1a3710711703459950029', 
    }).estimateGas().then(function(myGas) {
        console.log("Estimated gas: " + myGas);
        gas = myGas;
    })
    .catch(console.error);
```

실행하면, 컨트랙을 생성, 배포하기 위해 소요되는 gas는 120,415이다.
```python
pjt_dir> node src/HelloDeployGas.js

Estimated gas: 120415
```

실제 금액으로 얼마나 비용이 들게 되는지 계산해 볼 수 있다. 그러려면 gasPrice를 알아야 한다.

getGasPrice()는 사설망에서는 1 gwei로 고정되어 설정되어 있다 (아래 금액을 환산하면 0.000000001 Ether).
```python
node> web3.eth.getGasPrice().then(console.log);
1000000000 출력단위는 wei (1 gwei, 0.000000001 Ether에 해당한다)
```

그러나 실제 거래에 사용되는 평균가격은 보통 이 보다 높고 변동하기 마련이다 (참조: https://ethgasstation.info/)
대략으로 gas, gasPrice를 곱하고, 한화로 환산하면 24,000원이 필요하다. 물론 환전가격은 매일 변동하니까 정확하지 않다. 적지 않은 금액이다. gasPrice가 2배가 되는 경우도 많다. 그렇다면 48,000원으로 껑충 높아진다.

120,000 gas * 0.000000050 gasPrice * 4백만원 (환전가격) = 24,000원

게다가 곧 배포된 컨트랙을 사용하기 위해서 별도로 gas를 지불해야 한다. 이 금액을 더한다면 가격은 더 올라가게 된다.

#### ganache 배포 web3 1.20

자, 이제 배포해보자. ganache로 배포하려면 포트 번호 8345로 해주면 된다.

아래 코드 ```from``` 필드에 적힐 지급계정 주소는 위 ```getMyAddr```실행하고 출력된 목록에서 복사해서 넣어주었다.

```python
[파일명: src/HelloDeploy.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var shelloContract = new web3.eth.Contract([{"inputs":[],"name":"sayHello","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"pure","type":"function"}]);
shelloContract
    .deploy({
            data: '0x608060405234801561001057600080fd5b50610139806100206000396000f3fe608060405234801561001057600080fd5b5060043610610048576000357c010000000000000000000000000000000000000000000000000000000090048063ef5fb05b1461004d575b600080fd5b6100556100d0565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009557808201518184015260208101905061007a565b50505050905090810190601f1680156100c25780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040805190810160405280600b81526020017f48656c6c6f20576f726c6400000000000000000000000000000000000000000081525090509056fea165627a7a72305820992c2f4c73a27b9eb53c6aa7b52ba8a5eddba258089eb7d1a3710711703459950029', 
    })
    .send({
     from: "0xEb670169A5fcD2550035588A2ba3b45509703ABd",
     gas: '4700000'
    }, function (error, transactionHash){ 
            console.log(error, transactionHash); 
    })
    .then(function(newContractInstance){
        console.log(newContractInstance.options.address)
    });
```

실행해보자. 위 프로그램의 ```console.log()``` 명령문의 transactionHash, address가 출력된다.
```python
pjt_dir> node src/HelloDeploy.js

null '0xc255f82ab2150858ce0414f2ceaf0f919a64359f405ebded942bd04610f1a2f4'
0x0d3C29cAD3c40497699e13CC34bA099E1fe426Ba
```

#### geth배포 web3 1.20

사설망 geth에 배포하려면 ganache에 비해 추가 작업이 필요하다.

geth에 배포할 경우에는 지급계정을 해제하고 또한 마이닝도 해주어야 한다. 몇 번 설명하였지만, ganache에서는 계정를 해제해 줄 필요가 없다.

**지급계정을 해제**하면 시간을 설정하지 않는한 1회에 한한다. 또한 **잔고**가 남아 있어, 거래에 필요한 gas비용을 충당할 수 있어야 한다.

**node**에서는 **계정, 비밀번호, 해제기간**을 인자로 넣어주고 계정을 해제한다. 이 코드를 프로그램에 위 배포프로그램에 넣어 실행할 수도 있다 (아래 comment로 비활성화). 비밀번호를 넣어 하드코딩한다는 것은 좋은 생각은 아니다.

물론 **geth**창에서 직접해줄 수 있다. coinbase로 하거나 eth.accounts[0]과 같이 인덱스를 넣어주어도 된다.
```python
geth> personal.unlockAccount(eth.coinbase);  //or eth.accounts[0]
```

앞 ganache 배포 코드와 거의 동일하게 작성한다. 여기서는 몇 가지 이벤트 처리를 추가해서 다르게 보인다.
```python
[파일명: src/HelloDeploy2.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8445"));
//web3.eth.personal.unlockAccount("0x21c70...생략...7451","password",1000).then(console.log('unlocked!'));
var shelloContract = new web3.eth.Contract([{"constant":true,"inputs":[],"name":"sayHello","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}]);
shelloContract
    .deploy({
            data: '0x608060405234801561001057600080fd5b50610139806100206000396000f3fe608060405234801561001057600080fd5b5060043610610048576000357c010000000000000000000000000000000000000000000000000000000090048063ef5fb05b1461004d575b600080fd5b6100556100d0565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009557808201518184015260208101905061007a565b50505050905090810190601f1680156100c25780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b60606040805190810160405280600b81526020017f48656c6c6f20576f726c6400000000000000000000000000000000000000000081525090509056fea165627a7a72305820992c2f4c73a27b9eb53c6aa7b52ba8a5eddba258089eb7d1a3710711703459950029', 
    })
    .send({
     from: "0x21c704354D07f804baB01894e8B4eB4E0EBA7451",
     gas: '4700000'
    }, function (error, transactionHash){ 
            console.log(error, transactionHash); 
    })
    .on('transactionHash', function(error,transactionHash) {
        console.log("hash-- "+transactionHash);
    })
    .on('receipt', function(receipt) {
        console.log('receipt:: '+receipt.contractAddress);
    })
    .then(function(newContractInstance){
        console.log(newContractInstance.options.address)
    });
```

지급계정이 풀려 있다는 것을 확인하고 다름 프로그램을 실행하자.실행해보자. 계정해제를 해주지 않고 거래를 요청하면 '불충분 가스' 오류가 생성된다 (```Error: Returned values aren't valid, did it run Out of Gas?```)
```python
pjt_dir> node src/HelloDeploy2.js

null '0xa40eaf3185812b93c6a5432288a6efe6f6a1d3c644ca50592d850256afa3729c'
hash-- undefined
receipt:: 0xb8770393a75E5B88E7d5cFb5db1068e28B4dA8Ba
0xb8770393a75E5B88E7d5cFb5db1068e28B4dA8Ba
```

node로 ```HelloDeploy2.js```를 실행하고 나서, geth 백그라운드를 확인하자 (그림의 좌측). 그림 좌측에서 보듯이 transaction이 생성되고 그 hash와 contract (컨트랙 주소를 의미한다)이 로그로 출력된다. 그 메시지를 여기 복사해 놓으면 다음과 같다.
```python
INFO [01-07|10:34:47.871] Submitted contract creation
fullhash=0x851d565882f9758d03ef232a5badb6f23629a1f5768db2f3ca2c13e22a1ef40b
contract=0xd29b9e81388F91658a2120587A0e123c29eC6c36
```

![alt text](figures/6_helloWeb3DeployMining.png "deploy hello.sol using web3 with mining")

get 콘솔에서 대기 거래를 아래와 같이 출력하면, 대기하고 있는 transaction hash 값이 서로 일치한다 (그림의 우측 화면을 보자). 마이닝을 하고 나면 컨트랙 주소가 주어진다.

```python
geth> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
    gas: 4700000,
    gasPrice: 1000000000,
    hash: "0x851d565882f9758d03ef232a5badb6f23629a1f5768db2f3ca2c13e22a1ef40b",
    input: "0x60806...생략...50029",
    nonce: 265,
    r: "0xe995ae41b4193839df768ad629a9f2dc09ec7cfc08d5f42ce1207021d3395835",
    s: "0x26c380eb4f60e90cbdb54a45fbf691ce37b18b4b3e8603875f3cf5f6e30a0553",
    to: null,
    transactionIndex: null,
    v: "0x66",
    value: 0
}]
```

### 마이닝

배포를 완성하려면 그 거래가 마이닝되어야 한다. 물론 geth에서 배포할 경우에 그렇고, ganache는 마이닝이 불필요하다.

```python
geth> miner.start(1);admin.sleepBlocks(1);miner.stop()
```

마이닝이 되고 나면 비로서 컨트랙이 블록체인에 배포된다. 따라서 ```blockNumber```가 증가하게 된다.

그리고 컨트랙이 배포된 블록체인의 주소 값이 생성된다. 아래 ```getTransactioinReceipt(해시주소)```를 조회해보면 마이닝이 완료되기 전에는 null이다.

```python
geth> eth.getTransactionReceipt("0x3c58a...50bd5") 괄호 인자는 hash를 적어준다.
> eth.getTransactionReceipt("0x3c58a...50bd5").contractAddress 배포주소를 출력한다.
> eth.getCode(greeter.address) 배포 주소로 deployed code를 출력한다.
```

## 3.4 단계 4: 사용

컨트랙이 배포되었고, 그 주소를 획득하였다. 이제 그 컨트랙의 API를 호출해보자.
* 먼저 컨트랙의 객체를 생성하고,
* API를 호출한다.

### 4-1 주소를 가진 객체생성

배포한 콘트랙을 사용하려면 객체를 먼저 생성해야 한다.

앞서 블록체인에 생성된 객체를 얻어와야 하므로, 거래가 발생하게 된다. 거래가 발생하면, transactionHash가 생성되고, 마이닝이 수행되면 블록체인에 기록된다. 앞에서 얻은 ABI, contractAddress가 필요하다.

web3 0.20.x에서는 ```at()``` 함수를 사용하는데, 앞서 얻은 컨트랙 주소를 넣어준다.

```python
var MyContract = eth.contract(ABI).at(Address); // web3 0.20.x
```

web3 1.20에서는 ```new``` 명령어와 함께, abi와 컨트랙주소를 인자로 넘겨주어야 한다.
```
var MyContract = new web3.eth.Contract(abi, 컨트랙주소);  // web3 1.0 이후
```

### 4-2 methods

```methods```는 함수호출을 가능하게 한다. 즉 ```myContract.methods.myMethod(123)```과 같이 컨트랙의 함수를 호출할 수 있게 한다. 이어서 call 또는 send함수의 호출이 가능하다.

```
myballot.methods.vote.call(1);
myballot.methods.vote.sendTransaction(1, {from: primary})

```
인자가 여러 개인 경우
```
myInstance.methods.myStateChangingMethod('param1', 23, {value: 200, gas: 2000});
```

### call, send

주소를 획득하면 컨트랙 객체를 사용할 수 있게 된다.
로컬 블록체인에서 실행하는 ```call()``` 또는 블록체인에 기록되는 ```send()``` 함수를 사용한다.
거래가 발생하면, ```transactionHash```가 생성된다. 마이닝 전까지 대기 pending 한다.

함수 | 설명 | 블록체인
----------|----------|----------
```call()``` | 로컬에서 실행, 마이닝 불필요 | ```view```, ```pure```, ```constant``` 함수를 호출하는 경우. 블록체인에 기록되지 않는다.
```send()``` | 블럭체인에 기록하는 거래라서 마이닝 필요 | 블록체인에 기록된다.


```python
[파일명: src/HelloUse__.js]  이 파일은 web3 0.20.x 버전으로 작성
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var shelloContract = web3.eth.contract([{"constant":true,"inputs":[],"name":"sayHello","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}]);
var hello = shelloContract.at("0x325bb34f889fa04ada00fa126ed7e8a7600080ba");
console.log(hello.sayHello.call());
```

API를 사용하는 프로그램은 geth, ganache 어느 경우에나 포트번호만 변경해주면 된다. 물론 컨트랙의 주소는 정확하게 복사해서 사용한다.

이번에는 web3 1.20 버전으로 작성한다.

```python
[파일명: src/HelloUse.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
var shelloContract = new web3.eth.Contract([{"constant":true,"inputs":[],"name":"sayHello","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}],
                                      "0x0d3C29cAD3c40497699e13CC34bA099E1fe426Ba");
shelloContract.methods.sayHello().call().then(function(str) {console.log(str);});
```

실행해보자.
```python
pjt_dir> node src/HelloUse.js

Hello World
```

> **버전의 문제**

> ```Hello```를 Geth에서 Deploy는 성공하였지만, API를 사용하는 단계에서 반환 값의 오류가 나는 경우가 있다. 이런 오류를 경험하였다 **ERROR Returned values aren't valid. did it run Ouf of Gas?** 지급계좌도 해제해놓고, gas가 충분했다. 이 경우 **Solidity 버전을 낮추어** 컴파일 한 후 (0.4.25, 0.5.0, 0.5.1, 0.5.3 모두 성공, 0.5 후반대와 0.6은 모두 오류), ABI와 Bytecode로 새로 배포해서 API를 호출해보자.
또는 getPastEvents() 함수를 사용하는 경우 web3 1.0.0 beta에서는 **ERROR Returned values aren't valid. did it run Ouf of Gas?** 오류가 나지만, **web3 버전을 낮추어** 0.20.7에서는 그렇지 않다는 이슈가 있기도 하다.
나중에 배우겠지만 **Library**를 배포하고, API를 사용하면서 Solidity 0.5.x에서는 올바르게 작동하지 않았다. 버전을 낮추어 0.4.25로 하니까 오류가 사라졌다. Solidity 언어가 버전 간의 **후방호환성 backward compatibility**가 보장이 되지 않아서 문제가 발생할 수 있다.

### 3.5 웹디앱

앞서 구현한 Hello 컨트랙을 웹페이지로 만들어 보자. 매우 단순한 기능이지만 웹페이지에서 블록체인과 인터페이스하게 된다.

여기서 만드는 웹페이지는 서비스가 처리하되는 중앙 서버가 없는 **분산 애플리케이션 Distriubted Application, dApp**이다. 서버가 없고, 블록체인에서 서비스가 제공된다는 점이 특이하다.

먼저 웹서버를 띄우자.

```python
pjt_dir> python -m http.server 8045
Serving HTTP on 0.0.0.0 port 8045 (http://0.0.0.0:8045/) ...
127.0.0.1 - - [30/Apr/2020 06:38:09] "GET / HTTP/1.1" 200 -
```

다음으로 Hello 컨트랙으로터 결과를 받아서, "Hello World!"를 출력해보자. 적지 않은 시간과 노력을 들어, 처음으로 만드는 dApp이다.

아래는 web3 1.2에 맞추고 있다. web3는 cdn에서 호출하고 있으니 버전을 올려도 되고, 작동에는 문제가 없을 것으로 보인다. 단 하위 0.20 버전은 문법이 다르므로, 주석처리 해놓았고 필요하면 활성화하면 된다.

```python
[파일명: scripts/hello.html]
<!doctype>
<html>
<head>
<script src="https://cdn.jsdelivr.net/npm/web3@1.2.5/dist/web3.min.js"></script>
<!-- script src="https://cdn.jsdelivr.net/npm/web3@0.20.5/dist/web3.min.js"></script -->
<script type="text/javascript">
    //var Web3 = require('web3');  //can be removed
    var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

    function displayHello() {
        var web3version = web3.version;  //web3.version.api
        //web3@0.20.x
        //var shelloContract = web3.eth.contract([{"constant":true,"inputs":[],"name":"sayHello","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}]);
        //var hello = shelloContract.at("0x0d3C29cAD3c40497699e13CC34bA099E1fe426Ba");
        //var hello=hello.sayHello.call();
        //document.getElementById('sayhello').innerText = "The message from blockchain: " + hello;
        //document.getElementById('web3ver').innerText = 'web3 version 0.20.x: '+web3version.api;
        //document.getElementById('web3coinbase').innerText = 'coinbase: ' + web3.eth.coinbase;

        //web3@1.2.x - can not instanciate a contract
        var shelloContract = new web3.eth.Contract([{"constant":true,"inputs":[],"name":"sayHello","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"pure","type":"function"}],
                                      "0x0d3C29cAD3c40497699e13CC34bA099E1fe426Ba");
        shelloContract.methods.sayHello().call().then(function(str) {
            document.getElementById('sayhello').innerText = "The message from blockchain: " + str;
        });
        document.getElementById('web3ver').innerText = '*web3 version: '+web3version;
        web3.eth.getCoinbase().then(function(coin) {
            document.getElementById('web3coinbase').innerText = 'coinbase: ' + coin;
            });
    }
</script>
</head>
<body>
    <h1>Hello</h1>
    <button type="button" onClick="displayHello();">Display Hello</button>
    <div></div>
    <div id="sayhello"></div>
    <div id="web3ver"></div>
    <div id="web3coinbase"></div>
</body>
</html>
```

이 파일을 로컬호스트에서 띄어보자.

```python
http://localhost:8045/scripts/hello.html
```

# 4. Greeter 디앱

앞서 Hello 컨트랙은 함수 하나만으로 구성되었다. 이를 조금 수정하여, **데이터를 저장하는 컨트랙**으로 개발해 보자. 저장하는 데이터는 단순한 문자열이고, 구현하는 방식은 보통 프로그램과 크게 다르지 않다.

우리가 저장하는 문자열을 생각해보자. string은 동적으로 크기가 할당된다. 반면에 일정한 크기로 제한한다면 bytes4, bytes8 등을 사용하는 편이 필요하다. 최대 32바이트를 사용하다면 bytes32, 크키를 알 수 없다면 string을 사용한다.

그러나 데이터를 저장하면, 그 크기에 따라 gas가 더 필요하다는 것을 주의해야 한다. 256비트 당 20,000 gas가 필요하다.

## 4.1 단계 1. 컨트랙 개발

생성자는 배포하는 시점에 실행이 되기 때문에 'Hello' 문자열이 기본으로 저장된다.

매우 단순한 기능으로, ```set()```, ```get()```함수를 구현했다. ```get()```은 블록체인에서 읽어오는 함수, set()은 블록체인의 값을 변경한다. 따라서 ```set()```함수는 블록체인에 저장된다.

```web3.js```에서 ```sendTransaction()``` 함수로 호출한다.

### REMIX

좌측 메뉴에서 ```+``` 버튼을 눌러 프로그램을 새로 연다. 프로그램의 이름을 정한다. REMIX의 디렉토리 ```browser/```는 기본으로 주어지는 디렉토리이므로 그냥 둔다.

![alt text](figures/6_remixConnectToWebProvider.png "remix connecting to web provider")


```python
[파일명: src/greeter.sol]
pragma solidity ^0.6.0;

contract Greeter {
    string greeting;

    constructor() public {
        greeting = 'Hello';
    }

    function setGreeting(string memory _greeting) public {
        greeting = _greeting;
    }

    function greet() view public returns (string memory) {
        return greeting;
    }
}
```

## 4.2 단계 2. 컴파일

REMIX 또는 Solc로 컴파일해볼 수 있다. 여기서는 ```solc```로 컴파일 해보자.

컴파일해서 abi를 구해보자.
```python
pjt_dir> solc --abi src/greeter.sol

======= src/greeter.sol:Greeter =======
Contract JSON ABI
[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[],"name":"greet","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"string","name":"_greeting","type":"string"}],"name":"setGreeting","outputs":[],"stateMutability":"nonpayable","type":"function"}]
```

이번에는 bytecode를 출력해보자.
```python
pjt_dir> solc --bin src/greeter.sol
   
======= src/greeter.sol:Greeter =======
Binary:
    608060405234801561001057600080fd5b506040518060400160405280600581526020017f48656c6c6f0000000000000000000000000000000000000000000000000000008152506000908051906020019061005c929190610062565b50610107565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b61010491905b808211156101005760008160009055506001016100e8565b5090565b90565b610310806101166000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c8063a41368621461003b578063cfae3217146100f6575b600080fd5b6100f46004803603602081101561005157600080fd5b810190808035906020019064010000000081111561006e57600080fd5b82018360208201111561008057600080fd5b803590602001918460018302840111640100000000831117156100a257600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290505050610179565b005b6100fe610193565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561013e578082015181840152602081019050610123565b50505050905090810190601f16801561016b5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b806000908051906020019061018f929190610235565b5050565b606060008054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561022b5780601f106102005761010080835404028352916020019161022b565b820191906000526020600020905b81548152906001019060200180831161020e57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061027657805160ff19168380011785556102a4565b828001600101855582156102a4579182015b828111156102a3578251825591602001919060010190610288565b5b5090506102b191906102b5565b5090565b6102d791905b808211156102d35760008160009055506001016102bb565b5090565b9056fea2646970667358221220944f93b1d9f9ab2ca4e9ac2c5e6ec4fb4d64c53439729ef8f7856a4f0cdcea1864736f6c63430006010033
```

참고로 gas를 산정하자. gas비용이 무한 infinite로 산정되고 있다. 이 경우는 산정할 수 없다는 표현이 더 적절하다. 프로그램의 string은 배열이고 그 크기를 알 수 없기 때문에 그렇다.

실제 동적배열 또는 반복문을 적절하지 못하게 사용하면 무한의 gas비가 산정될 수 있고 따라서 실행이 되지 않을 수 있다는 점에 유의해야 한다.

```python
pjt_dir> solc --gas src/greeter.sol
    
======= src/greeter.sol:Greeter =======
Gas estimation:
construction:
   infinite + 156800 = infinite
external:
   greet():	infinite
   setGreeting(string):	infinite
```

## 4.3 단계 3. 컨트랙 배포

ABI, Byte code로 컨트랙 객체를 생성하고 배포를 요청한다.
배포하기 전 지급계정은 해제되어야 한다.
배포하기 위해서는 마이닝을 해야 하고, 비로서 콘트랙이 블록체인에 배포된다.
배포가 완성되면 주소 값이 생성된다.

### REMIX

```Run``` 탭을 누르고 ```Deploy``` 버튼을 눌러 배포할 수 있다. 배포할 때는 3가지 옵션 가운데 선택할 수 있다:
- ```JavaScript VM```: 브라우저에서 독립적으로 실행. 콘트랙을 테스트할 때 유용하다.
- ```Injected Web3```: MetaMask, Mist 등을 web3의 provider로 사용할 경우에 선택한다.
    * metamask에서 사설망으로 전송할 경우 ```invalid sender``` 오류가 발생하면 사설망의 ```chainId```가 ```networkId```와 동일한지 확인한다.
- ```Web3 Provider```: RPC를 통하여 ```http://localhost:8305```에 연결한다. ```Web3 provider```에 연결하면 자신의 계정이 나타난다.

* ```Javascript VM```

Deployed Contracts 아래에 해당 컨트랙을 선택하여 실행해 볼 수 있다. 버튼의 색에 따라 함수를 구분할 수 있게 해 놓았다.
**분홍색은 transaction**,
**하늘색은 view, pure 함수**로 블록체인에 기록되지 않는 함수이다. 

함수에 필요한 실제 값을 타입에 맞게 넣고 실행해 보자.

타잎 | 구성 | 예
-----|-----|-----
bytes | 따옴표 | "0x123456"
strings | 따옴표 | "hello"
large numbers | 따옴표 |
array | [] | ["hello",42,"0x123456"]

* ```Web3 provider```

Web3 provider를 선택하면, 해당 IP에서 개설된 자신의 계정으로 거래에 사인을 하게 된다. 그리고 우측 '```Deploy```'버튼을 누르면 하단에 보이듯이 contract address가 주어진다.
단 마이닝이 실행되어야 한다.

![alt text](figures/6_remixConnectToWebProviderKeySign.png "remix connecting to web provider")

### web3.js

아래는 web3 0.20.x 버전을 사용하고 있는 코드다.

```python
[파일명: src/greeterDeploy__.js] 이전 버전의 문법으로 작성되었다.
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8445"));
}
var _abiArray=[{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var _bin="608060405234801561001057600080fd5b506040805190810160405280600581526020017f48656c6c6f0000000000000000000000000000000000000000000000000000008152506000908051906020019061005c929190610062565b50610107565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b61010491905b808211156101005760008160009055506001016100e8565b5090565b90565b610323806101166000396000f3fe608060405234801561001057600080fd5b5060043610610053576000357c010000000000000000000000000000000000000000000000000000000090048063a413686214610058578063cfae321714610113575b600080fd5b6101116004803603602081101561006e57600080fd5b810190808035906020019064010000000081111561008b57600080fd5b82018360208201111561009d57600080fd5b803590602001918460018302840111640100000000831117156100bf57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290505050610196565b005b61011b6101b0565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561015b578082015181840152602081019050610140565b50505050905090810190601f1680156101885780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b80600090805190602001906101ac929190610252565b5050565b606060008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156102485780601f1061021d57610100808354040283529160200191610248565b820191906000526020600020905b81548152906001019060200180831161022b57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061029357805160ff19168380011785556102c1565b828001600101855582156102c1579182015b828111156102c05782518255916020019190600101906102a5565b5b5090506102ce91906102d2565b5090565b6102f491905b808211156102f05760008160009055506001016102d8565b5090565b9056fea165627a7a72305820e8cd5af384936c8c5f80781ebbc5ca63f8fa0e43133353c27accbb02ee216b550029";
var _contract = web3.eth.contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
var _instance=_contract.new("hello world",{data:"0x"+_bin,from:web3.eth.accounts[0],gas:1000000}, function(err, contract) {
    if (!err) {
        console.log("contractAddress: ", contract.address);
        console.log("transactionHash: ", contract.transactionHash);
    }
});
```

web3 1.20으로 배포를 해보자. currentProvider는 현재 설정된 Web3.providers가 있다면, 예를 들어 MetaMask, Mist가 제공하는 것을 쓴다는 뜻이다. 그렇지 않으면 수동으로 설정된 IP를 사용하게 된다.

```python
[파일명: src/greeterDeploy.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
}
var _abiArray=[{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var _bin="608060405234801561001057600080fd5b506040805190810160405280600581526020017f48656c6c6f0000000000000000000000000000000000000000000000000000008152506000908051906020019061005c929190610062565b50610107565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b61010491905b808211156101005760008160009055506001016100e8565b5090565b90565b610323806101166000396000f3fe608060405234801561001057600080fd5b5060043610610053576000357c010000000000000000000000000000000000000000000000000000000090048063a413686214610058578063cfae321714610113575b600080fd5b6101116004803603602081101561006e57600080fd5b810190808035906020019064010000000081111561008b57600080fd5b82018360208201111561009d57600080fd5b803590602001918460018302840111640100000000831117156100bf57600080fd5b91908080601f016020809104026020016040519081016040528093929190818152602001838380828437600081840152601f19601f820116905080830192505050505050509192919290505050610196565b005b61011b6101b0565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561015b578082015181840152602081019050610140565b50505050905090810190601f1680156101885780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b80600090805190602001906101ac929190610252565b5050565b606060008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156102485780601f1061021d57610100808354040283529160200191610248565b820191906000526020600020905b81548152906001019060200180831161022b57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061029357805160ff19168380011785556102c1565b828001600101855582156102c1579182015b828111156102c05782518255916020019190600101906102a5565b5b5090506102ce91906102d2565b5090565b6102f491905b808211156102f05760008160009055506001016102d8565b5090565b9056fea165627a7a72305820e8cd5af384936c8c5f80781ebbc5ca63f8fa0e43133353c27accbb02ee216b550029";
var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
_contract
    .deploy({data:"0x"+_bin})
    .send({from: "0xfA279EEA36550b831ce5734475F67e7d6eC4d607", gas: 364124, gasPrice: '1000000000'})
    .then(function(newContractInstance){
        console.log(newContractInstance.options.address) // instance with the new contract address
    });

```

이제 배포를 해보자.
```python
pjt_dir> node src/greeterDeploy.js

0xE4E640c0BF81Bbe45342589bd0c318D837ebDf2A
```

![alt text](figures/5_greeterDeployPending.png "remix connecting to web provider")

![alt text](figures/5_greeterDeployMining.png "remix connecting to web provider")

## 4.4 단계 4. 사용

위에서 얻은 contract address를 넣어 방금 만든 콘트랙을 사용할 수 있게 된다.

객체를 생성할 경우 ABI, Byte code를 사용한다.

함수를 호출할 경우 로컬 또는 블록체인을 사용할지 판단한다.
* 값을 읽는 경우 ```call()```을 사용하고, gas비용이 발생하지 않는다.
```python
greeter.methods.greet().call()
```

* 값을 변경하는 경우, 함수를 직접 호출하거나 ```send()```을 사용한다. gas비용이 발생하고, 적게 설정하면 오류가 난다. 반환 값이 없다. options에는 from, gasPrice, gas, value를 넣어준다.

```python
myContract.methods.myMethod([param1[, param2[, ...]]]).send(options[, callback])
```

```python
greeter.methods.setGreeting("Hello World").send({from:web3.eth.accounts[0],gas:1000});
```

아래는 web3 0.20.x 버전을 사용하고 있는 코드다.

```python
[파일명: src/greeterUse__.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8445"));
var _abiArray=[{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var _contract = web3.eth.contract(_abiArray);
var greeter = _contract.at("0x9397fbedbcdf9b80bad6805df98d24b6676e19e3");
console.log(greeter.greet.call());
console.log(greeter.setGreeting({from:web3.eth.accounts[0],gas:100000}));
//console.log(greeter.greet());
```

web3 1.20으로 배포를 해보자.

```python
[파일명: src/greeterUse.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var _abiArray=[{"constant":false,"inputs":[{"name":"_greeting","type":"string"}],"name":"setGreeting","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"greet","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"}];
var greeter = new web3.eth.Contract(_abiArray,"0xE4E640c0BF81Bbe45342589bd0c318D837ebDf2A");
greeter.methods.greet().call().then(function(value) {console.log(value);});
greeter.methods.setGreeting("Hello SMU").send({from:"0xfA279EEA36550b831ce5734475F67e7d6eC4d607",gas:100000});
greeter.methods.greet().call().then(function(value) {console.log(value);});
```

노드 함수를 1회 실행하면, 비동기적 특성으로 ```setGreeting()``` 함수가 반영되지 않는다. 처음에는 Hello가 출력된다. 2회 실행시키면, 즉 Hello World로 문자열을 변경한 결과가 출력된다.


```python
pjt_dir> node src/greeterUse.js

Hello SMU
Hello SMU
```

# 5. 간단한 계수기

계수기는 언제나 현재의 계수를 가지고 있다. 0부터 시작해서 하나씩 증가하거나 감소하며, **현재의 계수로부터 증감**한다는 것이다.

블록체인은 **state machine**이라서, 전의 값을 기억하고 그로부터 증가하게 된다. **프로그램을 호출한 횟수**가 좋은 예가 될 수 있다. 여러 사람이 호출했다고 하더라도 총 횟수를 기억할 수 있다. 현재 값을 저장하고 있고, 그 값이 누구에게나 공개된다는 점이 블록체인의 특징을 보여주고 있다.

## 5.1 단계 1. 컨트랙 개발

```python
[파일명: src/Counter.sol]
pragma solidity ^0.6.0;
contract Counter {
    uint256 counter = 0;
    function add() public {
        counter++;
    }
    function subtract() public {
        counter--;
    }
    function getCounter() public view returns (uint256) {
        return counter;
    }
}
```

## 5.2 단계 2. 컴파일

solc 컴파일을 통해 abi, bin, gas를 구한다.


```python
pjt_dir> solc --abi --bin --gas src/Counter.sol
 
======= src/Counter.sol:Counter =======
Gas estimation:
construction:
   5099 + 42200 = 47299
external:
   add():	20959
   getCounter():	1035
   subtract():	20984
Binary:
    60806040526000805534801561001457600080fd5b5060d3806100236000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c80634f2be91f1460415780636deebae31460495780638ada066e146051575b600080fd5b6047606d565b005b604f6080565b005b60576094565b6040518082815260200191505060405180910390f35b6000808154809291906001019190505550565b600080815480929190600190039190505550565b6000805490509056fea2646970667358221220a19d7e0374295c3c6dd75807d6b2bb20a12deb6f736a4ad98c0065f0d9d4bf5764736f6c63430006010033
    Contract JSON ABI
    [{"inputs":[],"name":"add","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"getCounter","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"subtract","outputs":[],"stateMutability":"nonpayable","type":"function"}]
```

## 5.3 단계 3. 배포

web3 0.20.x버전과 1.2.x 버전의 배포 프로그램이다.

```python
[파일명: src/counterDeploy__.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8445"));
}
var _abiArray=[{"constant":false,"inputs":[],"name":"add","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"subtract","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getCounter","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}];
var _bin="60806040526000805534801561001457600080fd5b5060e6806100236000396000f3fe6080604052348015600f57600080fd5b50600436106059576000357c0100000000000000000000000000000000000000000000000000000000900480634f2be91f14605e5780636deebae31460665780638ada066e14606e575b600080fd5b6064608a565b005b606c609d565b005b607460b1565b6040518082815260200191505060405180910390f35b6000808154809291906001019190505550565b600080815480929190600190039190505550565b6000805490509056fea165627a7a723058201fbaa288a76e68fea3b0373a390c6e375e9bb90c0fd24b0660d64ebb408088d60029";
var _contract = web3.eth.contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
var _instance=_contract.new({data:"0x"+_bin,from:web3.eth.accounts[0],gas:100000}, function(err, contract) {
    if (!err) {
        console.log("contractAddress: ", contract.address);
        console.log("transactionHash: ", contract.transactionHash);
    }
});
```

```python
[파일명: src/counterDeploy.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
}
//solc 0.5.0
//var _abiArray=[{"constant":false,"inputs":[],"name":"add","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"subtract","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getCounter","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}];
//var _bin="60806040526000805534801561001457600080fd5b5060e6806100236000396000f3fe6080604052348015600f57600080fd5b50600436106059576000357c0100000000000000000000000000000000000000000000000000000000900480634f2be91f14605e5780636deebae31460665780638ada066e14606e575b600080fd5b6064608a565b005b606c609d565b005b607460b1565b6040518082815260200191505060405180910390f35b6000808154809291906001019190505550565b600080815480929190600190039190505550565b6000805490509056fea165627a7a723058201fbaa288a76e68fea3b0373a390c6e375e9bb90c0fd24b0660d64ebb408088d60029";
//solc 0.6.1
var _abiArray=[{"inputs":[],"name":"add","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"getCounter","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"subtract","outputs":[],"stateMutability":"nonpayable","type":"function"}];
var _bin="60806040526000805534801561001457600080fd5b5060d3806100236000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c80634f2be91f1460415780636deebae31460495780638ada066e146051575b600080fd5b6047606d565b005b604f6080565b005b60576094565b6040518082815260200191505060405180910390f35b6000808154809291906001019190505550565b600080815480929190600190039190505550565b6000805490509056fea2646970667358221220a19d7e0374295c3c6dd75807d6b2bb20a12deb6f736a4ad98c0065f0d9d4bf5764736f6c63430006010033";
var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
_contract
    .deploy({data:"0x"+_bin})
    .send({from: "0x0A815B7818A8e6BC27B430e41Edc8FC455F658c2", gas: 364124, gasPrice: '1000000000'})
    .then(function(newContractInstance){
        console.log(newContractInstance.options.address) // instance with the new contract address
    });
```

```python
pjt_dir> node src/counterDeploy.js

0xb24ab776373e53fFeb9B7298209E195853D2fc8e
```

## 5.4 단계 4. 사용

```python
[파일명: src/counterUse.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var abi =[{"constant":false,"inputs":[],"name":"add","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"subtract","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getCounter","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}];
var addr = "0xb24ab776373e53fFeb9B7298209E195853D2fc8e";
var counter = new web3.eth.Contract(abi,addr);
counter.methods.getCounter().call().then(function(str) {console.log(str);});
//counter.methods.subtract().send({from:"0x0A815B7818A8e6BC27B430e41Edc8FC455F658c2",gas:100000});
counter.methods.add().send({from:"0x0A815B7818A8e6BC27B430e41Edc8FC455F658c2",gas:100000});
counter.methods.add().send({from:"0x0A815B7818A8e6BC27B430e41Edc8FC455F658c2",gas:100000});
counter.methods.getCounter().call().then(function(str) {console.log(str);});
```

geth 개인망에서 ```add()```, ```subtract()``` 함수는 호출하고, Pending Transactions을 처리하고 나면 숫자가 증감한 것을 알 수 있다.

```add()```는 counter에 1을 더하는, ```subtract()```은 1을 빼는 상태변경 이므로 마이닝을 해야 그 결과가 반영된다. 그러고 나면 ```getCounter()```에서 알 수 있다.

주목할 점은 수가 계속 증감한다는 점이다. 세션을 다시 열어도 전의 ```counter``` 값은 남아 있는다.

```python
pjt_dir> node src/counterUse.js

4
4
```


# 실습문제: 타이머 디앱 (파일에서 ABI, bytecode 읽음)

블록체인에서 시간을 측정해보자.

블록체인은 개방되어 있으므로 누구나 타이머를 설정하고 사용할 수 있다. 이러한 점때문에 누구나 자신의 타이머를 가질 수 있지만, 다른 누군가에 의해 재설정될 수도 있다. 이러한 단점을 보완하는 방법은 차츰 학습하기로 하자.

이번 예제에서는 배포, 사용하는 방법을 바꾸어 보았다. 컴파일하고 **abi, bin을 매번 복사하는 것이 불편**하다고 생각할 것이다. 파일로 저장한 후, abi, bin을 읽도록 프로그램을 작성해보자. 프로그램이 조금 복잡해지지만, 복사-붙여넣기 없이 개발할 수 있게 된다.

또한 Geth에서도 배포하고, API를 사용하여 보자.

## 단계 1: 컨트랙 개발

줄 | 설명
-----|-----
1 | Pragma 프로그램 버전. 맨 앞의 ```^``` (caret)은 메이저버전으로 시작하는 최근 버전의 컴파일러를 사용. 따라서 5로 시작하는 최근 버전인 5.21을 사용
4 | public으로 사용권한을 제한하지 않음
5 | ```now```는 현재 시간, 시스템에서 제공하는 전역변수. now는 버전0.7부터 더 이상 지원되지 않는다. ```block.timestamp```를 사용하자.
7 | public이고, pure 또는 view로 읽는 기능으로 한정함.


0.6.0으로 로컬컴파일 후, ganache에서는 문제가 없으나, geth에서 getNow()호출에 ```returned values aren't valid, did it run out of gas?``` 오류가 발생한다. REMIX에서 컴파일하고, 그 abi, bin으로 사용해서 확인하니, 문제가 없다.

* 개선: timePassed가 REMIX에서는 올바르게 출력. 그러나 web3에서는 0으로 출력. 멤버변수로 설정하면?

```python
[파일명: src/Timer.sol]
pragma solidity ^0.6.0;
contract Timer {
    uint256 startTime;
    function start() public {
        //startTime=now; now is deprecated as of v0.7
        startTime=block.timestamp;
    }
    function timePassed() public view returns(uint256) {
        //return now-startTime;
        return block.timestamp-startTime;
    }
    function getNow() view public returns(uint) {
        //return now;
        return block.timestamp;
    }
}
```

## 단계 2: 컴파일

컴파일하고 난 ABI, bytecode를 (1) 자바스크립트, (2) JSON 파일로 내보내게 된다.

abi, bin를 하나씩 생성하지 않고, 한꺼번에 ```json```파일을 생성하여 읽도록 하자. ```combined-json```을 사용하면, 한꺼번에 컴파일 결과를 ```Timer.js``` 파일에 저장한다.  파일의 확장자가 ```.js```라는 것은 자바스크립트를 의미한다.

주의: ```--combined-json abi, bin``` 이렇게 공백이 포함되면 안된다.

```python
pjt_dir> solc --optimize --combined-json abi,bin src/Timer.sol > src/Timer.json
```

```Timer.json```에 저장이 잘 되었는지 출력해보자.  운영체제 별로 명령어와 디렉토리 구분자가 다르므로 주의해야 한다. 윈도우에서는 ```type src\Timer.json``` 리눅스에서는 ```cat src/Timer.json```이다.

출력결과는 이해하기 쉽도록 편집하였다. JSON 형식으로 키, 값이 사상되어 저장되어 있다. 컨트랙의 명칭, abi, bin을 유의해서 보아야, 나중에 배포하면서 이 파일에서 읽어올 수 있게 된다.
```python
pjt_dir> type src\Timer.json

{"contracts":
	{
		"src/Timer.sol:Timer":{
			"abi":"[{\"inputs\":[],...생략...}]",
			"bin":"60806...생략...10033"
		}
	},
"version":"0.6.1+commit.e6f7d5a4.Windows.msvc"}
```

이와 같이 생성된 형식은 다음 명령어로 JSON으로 변환할 수 있다.
```python
node> var fs = require('fs');
> var _abiJson=JSON.parse(fs.readFileSync('./src/TimerABI.json', 'utf8'));
```

반면에 JSON파일은 바로 ```require(JSON파일명)``` 함수로 아래와 같이 읽으면 편하다. Javascript 파일의 ```require(자바스크립트파일명)```와 달리 export 명령어를 사용할 필요가 없다.

```require```에서 읽는 파일경로는 **현재파일의 상대적 경로**로 적어준다. 앞서 컴파일 결과는 JSON 형식이고, 여기서 abi, bin을 읽어올 수 있다.

#### **solc로 컴파일할 경우 파일 경로와 import**

위 파일을 보면, 컨트랙 명칭이 'src/Timer.sol:Timer'로 작명된다. 이 부분은 문법 ```context:prefix=target```에 따라 생성이 된다.

구분 | 설명 | 선택 | 예
-----|-----|-----|-----
context | 읽을 파일 | 선택적 | src/Timer.sol
:prefix |  |  | :Timer
=target | 없으면 prefix와 동일  | 선택적 | 

이 명칭이 꽤 복잡하므로 프로그램으로 얻어보자.
줄2를 보자. 자바스크립트 문법으로 **객체의 키**를 읽어오려면, ```Object.keys(키)``` 함수에 인자 '키'를 입력하면, 그에 해당하는 값을 얻을 수 있다.

```python
[파일명: src/TimerImportTest.js]
const _abiBinJson = require('./Timer.json');     //importing a javascript file
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/Timer.sol:Timer']
console.log("- contract name: ", contractName);
_abi=_abiBinJson.contracts[contractName].abi;
_abiArray=JSON.parse(JSON.stringify(_abi));
_bin=_abiBinJson.contracts[contractName].bin;
console.log("- ABI: ", _abi);
console.log("- ABIArray: ", _abiArray);
console.log("- Bytecode: ", _bin);
```

이 프로그램을 실행하면, abi, bin 정보를 출력한다. 잘 읽었는지 확인해보자.

```python
pjt_dir> node src/TimerImportTest.js

	- contract name:  [ 'src/Timer.sol:Timer' ]
	- ABI:  [{"inputs":[],"name":"getNow","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"start","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"timePassed","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"}]
    - ABIArray:  [{"inputs":[],"name":"getNow","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},{"inputs":[],"name":"start","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"timePassed","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"}]
    - Bytecode:  608060405234801561001057600080fd5b5060af8061001f6000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c8063b4454253146041578063bbe4fd50146059578063be9a655514605f575b600080fd5b60476067565b60408051918252519081900360200190f35b6047606f565b60656073565b005b600054420390565b4290565b4260005556fea2646970667358221220f2dd49ad7ab0645d152793d9df9439e21492b4a9dd91cb1b9fcfd494293709bd64736f6c63430006010033
```

### 노드에서 한 줄씩 코딩해보기

json을 생성하고 그 파일을 읽어오는 과정이 이해하기 어려우면 손코딩이 제일이다.
노드에서 한 줄씩 ABI, BIN을 읽어오는 연습을 해보자.

```python
pjt_dir> node
> var _abiBinJson=require('./src/Timer.json')
undefined
> Object.keys(_abiBinJson.contracts)
[ 'src/Timer.sol:Timer' ]
> 이후에도 한 줄씩 계속 입력하고 결과를 출력해보자.
```

## 단계 3: 컨트랙 배포

컴파일하고 abi, bin는 파일에 저장하였고, 그 파일에서 읽어서 배포해 보자.

Node에서는 앞서 하나씩 생성한 ```Timer.json``` 파일을 읽어서 배포해보자.

주의:
* ```require('./Timer.json')```에 호출 시점의 상대 디렉토리 ```src/Timer.json```를 적는다.
* bytecode는 16진수 표현 "0x"를 붙인다.


줄 | 설명
-----|-----
1 | 사용 계정을 설정한다.
4 | 컴파일하고 생성된 abi, bin이 저장된 js파일을 읽는다
5 | contract명에 path prefix인 디렉토리 구분자 '/'가 포함되어 있다. Object.keys로 읽은 key를 contract명으로 사용한다.
10 | binary data는 '0x'가 빠져 있다. 블록체인에 배포할 때 0x를 붙여서 16진수 문자열이라고 명시해 준다. **gas**를 임의로 정해준다. 가격이 맞지 않으면....

```python
[파일명: src/TimerDeployAbiBinFromFile.js]
var Web3 = require('web3');
var _abiBinJson = require('./Timer.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/Timer.sol:Timer']
console.log("- contract name: ", contractName);
_abi=_abiBinJson.contracts[contractName].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
_bin=_abiBinJson.contracts[contractName].bin;

console.log("- ABI: " + _abiArray);
console.log("- Bytecode: " + _bin);

async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 364124}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance){
        //    console.log(newContractInstance)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()
```

ABI를 출력하면 object가 포함되어 있다. 마지막 줄을 보면, 배포된 컨트랙의 주소를 얻을 수 있다.

```python
pjt_dir> node src/TimerDeployAbiBinFromFile.js

    - contract name:  [ 'src/Timer.sol:Timer' ]
    - ABI: [object Object],[object Object],[object Object]
    - Bytecode: 608060405234801561001057600080fd5b5060af8061001f6000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c8063b4454253146041578063bbe4fd50146059578063be9a655514605f575b600080fd5b60476067565b60408051918252519081900360200190f35b6047606f565b60656073565b005b600054420390565b4290565b4260005556fea2646970667358221220f2dd49ad7ab0645d152793d9df9439e21492b4a9dd91cb1b9fcfd494293709bd64736f6c63430006010033
    Deploying the contract from 0xEb670169A5fcD2550035588A2ba3b45509703ABd
    hash: 0xbd010c6ed017b05579abd42f94d332ece6b4ce21c83eb881d46094fb3572eed4
    ---> The contract deployed to: 0x5dB5E509ff13A760345C0B58591E2a5DA6e1B2DE
```

nodejs example.sol compile, deploy

![alt text](figures/6_nodejsWeb3Example.png "node web3 test run Example.sol")

## 단계 4: 사용

JSON에서 파일을 읽어서 ABI를 가져온다. bytecode는 API를 호출할 때 불필요하다.

우리가 호출한 ```getNow()```, ```timePased()```는 **call 함수**이다. 반면에 ```start()```는 **send 함수**이다.

```python
[파일명: src/timerUse.js]
var Web3=require('web3');
var _abiBinJson = require('./Timer.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/Timer.sol:Timer']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abi=_abiBinJson.contracts[contractName].abi;
//_abiArray=JSON.parse(JSON.stringify(_abi));
_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

var timer = new web3.eth.Contract(_abiArray,"0x5dB5E509ff13A760345C0B58591E2a5DA6e1B2DE");

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Call from: " + accounts[0]);
    timer.methods.getNow().call().then(function(value) {console.log("Now: ", value);});
    await timer.methods.start().send({from:accounts[0],gas:100000});
    //await 4000;
    timer.methods.timePassed().call().then(function(value) {console.log("Passed: ", value);});
}

doIt()
```

일정 시간이 지난 후 몇 번 반복시켜서 지나간 시간이 변동하는지 확인해 보자.

지난 시간은 타임스탬프 signed 32-bit 형식으로 출력된다. 이 형식을 우리가 사용하는 시간 형식으로 변환해야 한다. https://www.epochconverter.com/


> **타임스탬프 timestamp**

> 유닉스에서 사용하는 타임스탬프이고 Long Integer 형식이다. 1970년 1월 1일 00:00:00 기준으로 몇 초나 지났는지 계산한다. 하루는 86400초, 60초 x 24시간 x 60분 기준이다. 윤초는 계산에서 제외해야 한다.

REMIX에서 테스트하면, 올바르게 작동하지만 여기서는 Passed가 0으로 출력되고 있다. (start를 생성자에서 하면 이런 문제가 사라질지 해보자)

```python
pjt_dir> node src/timerUse.js

Call from: 0xEb670169A5fcD2550035588A2ba3b45509703ABd
Now:  1648957889
Passed:  0
```

```python
pjt_dir> node src/timerUse.js

- contract name:  src/Timer.sol:Timer
1619427898
4
```

nodejs example.sol use after deploy 배포

![alt text](figures/6_nodejsWeb3ExampleUse.png "node web3 test run Example.sol")

## 부록: ```geth --exec loadScript()```을 하기 위한 단계 2~4

### 방법 1: 자바스크립트 파일로 내보내기

아래 solc명령어는 앞서 사용했던 명령문과 다르지 않지만, 조금 복잡해졌다. 그 이유는, ```combined-json```을 사용한 컴파일 결과를 ```Timer.js``` 파일에 저장하고 있지만, 자바스크립트 파일로 만들기 위해 몇 가지 명령문을 사용하고 있다.

```python
pjt_dir> echo "exports._compiled=`solc --optimize --combined-json abi,bin,interface src/Timer.sol`" > src/Timer.js
```

위 명령문에 쓰인 echo, exports 그리고 부등호 (>)를 하나씩 배워보자.

명령문 | 의미
-----|-----
리눅스 echo | echo는 부등호와 같이 쓰여, 부등호 우측으로 내용을 보내 파일을 생성하게 된다. 이 경우, solc 컴파일이 되고 --combined-json의 abi, bin, interface가 생성되어 자바스크립트 파일이 만들어지게 된다. 즉 ```src/Timer.sol```을 컴파일하고 그 결과를 자바스크립트 변수 **```_compiled```**로 저장하고, ```src/Timer.js``` 파일에 쓴다.
```exports``` | 변수를 require()할 때 노출시키게 되어 사용할 수 있다.
부등호 | 좌측 결과를 우측으로 보내라는 의미, 그러니까 echo의 결과를 src/Timer.js 파일을 생성하여 쓰게 된다. 참고로 부등호 2개 ">>"는 이미 기존에 있는 파일에 추가한다는 의미이다.

```python
pjt_dir> echo "exports._compiled=`solc --optimize --combined-json abi,bin,interface src/Timer.sol`" > src/Timer.js
```

> 주의: **윈도우 echo 명령어**

> 윈도우는 운영체제가 달라, 조금 다르다는 점에 유의하자.
> 윈도우에서는 echo 다음에 쓰이는 pipe ```"|"```, 변수값을 입력받는 "set /p" 명령어, 파일에 쓰고 ">", 파일에 붙여넣는 ">>" 명령어를 사용한다.

```python
pjt_dir> echo | set /p="exports.compiled=" > src\Timer.js
```

```python
pjt_dir> solc --optimize --combined-json abi,bin,interface src/Timer.sol >> src\Timer.js
```

결과가 적힌 ```Timer.js``` 파일의 내용을 확인해 보자.

그 내용이 아래와 같이 생성이 되어 있지 않으면 오류이다.
cat은 내용을 출력하는 리눅스 명령어이다. 윈도우에서는 type을 대신 사용한다.


```python
pjt_dir> type src\Timer.js
```

    exports.compiled={"contracts":{"src/Timer.sol:Timer":{"abi":"[{\"inputs\":[],\"name\":\"getNow\",\"outputs\":[{\"internalType\":\"uint256\",\"name\":\"\",\"type\":\"uint256\"}],\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"start\",\"outputs\":[],\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"inputs\":[],\"name\":\"timePassed\",\"outputs\":[{\"internalType\":\"uint256\",\"name\":\"\",\"type\":\"uint256\"}],\"stateMutability\":\"view\",\"type\":\"function\"}]","bin":"608060405234801561001057600080fd5b5060af8061001f6000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c8063b4454253146041578063bbe4fd50146059578063be9a655514605f575b600080fd5b60476067565b60408051918252519081900360200190f35b6047606f565b60656073565b005b600054420390565b4290565b4260005556fea2646970667358221220f2dd49ad7ab0645d152793d9df9439e21492b4a9dd91cb1b9fcfd494293709bd64736f6c63430006010033"}},"version":"0.6.1+commit.e6f7d5a4.Windows.msvc"}


파일확장자를 주었듯이, 이렇게 생성된 파일은 자바스크립트 파일이다. json파일이 아니다.

* (1) **node에서 그 파일을 require**해서 사용할 수 있고, 
* (2) 반면에 geth는 자바스크립트를 사용하면서도 불구하고 require() 함수를 사용해서 파일을 읽을 수 없다.

### 단계 3: 자바스크립트 파일을 읽어서 geth 스크립트로 배포

앞서 **--combined-json abi,bin, interface** 스위치로 자바스크립트 파일을 생성해 놓았다. 이 파일을 읽어 Geth에서 Deploy를 해보자.
loadScript()함수는 Geth에서 자바스크립트를 읽어 들일 수 있다. 단 require() 함수를 지원하지 않아서, JSON파일을 읽어올 수 없다.

require()가 아니, loadScript()를 사용하면, export 명령어가 작동하지 않는다.
따라서 ```export``` 대신, ```var _compiled```라고 해주었다.
리눅스에서는 다음과 같이 명령어가 단순하다.


```python
pjt_dir> echo "var _compiled=`solc --optimize --combined-json abi,bin,interface src/Timer.sol`" > src/TimerGeth.js
```

윈도우에서는 다음과 같이 한다.


```python
pjt_dir> echo | set /p="var _compiled=" > src\TimerGeth.js
```


```python
pjt_dir> solc --optimize --combined-json abi,bin,interface src/Timer.sol >> src\TimerGeth.js
```

그리고 나서 Geth에서 읽을 수 있는지 확인해볼 수 있다.
```
pjt_dir> geth attach http://localhost:8345
Welcome to the Geth JavaScript console!
> loadScript('src/TimerGeth.js') <--- 조금 전 생성한 파일을 읽기
true
> _compiled                      <--- 파일의 변수 읽기 (solc 컴파일 결과)
{
  contracts: {
    src/Timer.sol:Timer: {
      abi: "[{\"inputs\":[],\"name\":\"getNow\",\"outputs\":...}]", 중간 생략
      bin: "6080604052...10033"                                     중간 생략
    }
  },
}
> Object.keys(_compiled.contracts)   <--- 컨트랙 명 읽기
["src/Timer.sol:Timer"]
```



```python
[파일명: src/TimerDeployGeth.js]
var primary = eth.accounts[0];
console.log("primary ac: ",primary);
console.log("balance: ",eth.getBalance(primary));
loadScript('src/TimerGeth.js');
contractName=Object.keys(_compiled.contracts); // reading ['src//Timer.sol:Timer']
//_abi=JSON.parse(_compiled.contracts['src/Example.sol:Example'].abi)
//_bin=_compiled.contracts['src/Example.sol:Example'].bin
_abi=JSON.parse(_compiled.contracts[contractName[0]].abi)
_bin=_compiled.contracts[contractName[0]].bin

_class=eth.contract(_abi);
console.log('bin code: ', _bin)
//this async way does not work from the Jupyter Notebook
_object=_class.new({from:primary,data:'0x'+_bin,gas:1000000}, function(err, contract) {
  if (!err && contract.address)
    console.log("contractAddress: ", contract.address);
    console.log("transactionHash: ", contract.transactionHash);
});
```

```python
pjt_dir> geth --exec "loadScript('src/TimerDeployGeth.js')" attach http://localhost:8345


    primary ac:  0xeb670169a5fcd2550035588a2ba3b45509703abd
    balance:  999998252516772701039
    bin code:  608060405234801561001057600080fd5b5060af8061001f6000396000f3fe6080604052348015600f57600080fd5b5060043610603c5760003560e01c8063b4454253146041578063bbe4fd50146059578063be9a655514605f575b600080fd5b60476067565b60408051918252519081900360200190f35b6047606f565b60656073565b005b600054420390565b4290565b4260005556fea2646970667358221220f2dd49ad7ab0645d152793d9df9439e21492b4a9dd91cb1b9fcfd494293709bd64736f6c63430006010033
    transactionHash:  0x1eb79103f4919a5540d19d1fb301468739adf5ad4d94b352c283d06451d76dcb
    true
```

마이닝하고나면 블록체인의 주소가 주어진다. **Jupyter Notebook에서는 geth async가 실행되지 않으므로, 주소가 출력이 되지 않는** 것으로 보인다. ```getTransactionReceipt()```의 결과를 보면 ```contractAddress```가 출력되어 있다.


```python
pjt_dir> geth --exec "eth.getTransactionReceipt('0x1eb79103f4919a5540d19d1fb301468739adf5ad4d94b352c283d06451d76dcb')" attach http://localhost:8345


    {
      blockHash: "0x67717ff2152812529c2171659bea8606dd2b8129d00b130b97fc387d7920bb4f",
      blockNumber: 21,
      contractAddress: "0x41918c2a0b147ccbf4dc1061311c48534722016a",
      cumulativeGasUsed: 91227,
      effectiveGasPrice: "0x49c3b7b",
      from: "0xeb670169a5fcd2550035588a2ba3b45509703abd",
      gasUsed: 91227,
      logs: [],
      logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
      status: "0x1",
      to: null,
      transactionHash: "0x1eb79103f4919a5540d19d1fb301468739adf5ad4d94b352c283d06451d76dcb",
      transactionIndex: 0,
      type: "0x2"
    }
```

### 단계 4: web3.js가 아닌 eth를 사용

위에서 Geth 배포가 성공하였다면, 이어서 API를 호출해보자.

```python
[파일명: src/TimerUseGeth.js]
loadScript('src/TimerGeth.js')
contractName=Object.keys(_compiled.contracts)
_abi=JSON.parse(_compiled.contracts[contractName[0]].abi)
//_abi=JSON.parse(_compiled.contracts['src/Example.sol:Example'].abi)
var _contract=eth.contract(_abi);
var _address="0xb5f064b6cf3d7b7497ab88a846819629d0c05136";
var _instance=eth.contract(_abi).at(_address);
console.log(_instance.getNow.call());
_instance.start.sendTransaction({from:eth.accounts[0]});
console.log(_instance.timePassed.call());
```

Geth 개인망에서 실행하는 경우, 마이닝이 뒤따라야 한다.
여기서는 조금 있다 보게되는 node의 실행결과와 달리 ```timePassed```가 올바르게 출력이 되지 않고 있다.

```python
pjt_dir> geth --exec 'loadScript("src/TimerUseGeth.js")' attach http://localhost:8345
1588492924
0
true
```

## 연습문제: 자바스크립트 전역변수 설정


```python
%%writefile src/hoistingTest.js
var x = 11;

function hoist() {
    x = 22;
}
hoist();
console.log(x);
```

    Overwriting src/hoistingTest.js



```python
!node src/hoistingTest.js
```

    22


## 연습문제: Hello Snowman 출력

```Hello``` 컨트랙을 수정해서 ```sayHello()``` 함수를 호출하면 ```"Hello, Snowman"```이 출력되도록 하세요.

- Solidity로 ```Hello.sol```을 코딩하고
- solc로 컴파일하고
- web3.js를 사용하여 node로 프로그램해서 배포
- web3.js를 사용하여 node로 ```sayHello()``` 출력


```python
pragma solidity ^0.6.0;
contract Hello {
    function sayHello() pure public returns(string memory) {
        return "Hello, Snowman";
    }
}
```

## 연습문제: byte32 타입의 Hello Greeter

```greeter```의 ```greeting```의 데이터타입을 ```string```에서 ```bytes32```로 변경하세요.
- ```string```에서 ```bytes32```로 변경하면 gas의 차이가 있는지 확인
- 노드에서 배포
- 노드에서 인사를 설정하고 ```setGreeting("Hello World!")```, ```greet()```을 출력하세요.


```python
%%writefile src/GreeterB.sol
pragma solidity ^0.6.0;
contract GreeterB {
    bytes32 greeting;
    constructor() public {
        greeting = 'Hello';
    }
    function setGreeting(bytes32 _greeting) public {
        greeting = _greeting;
    }
    function greet() view public returns (bytes32) {
        return greeting;
    }
}
```

    Overwriting src/GreeterB.sol



```python
!solc --abi src/GreeterB.sol
```

    
    ======= src/GreeterB.sol:GreeterB =======
    Contract JSON ABI
    [{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[],"name":"greet","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"bytes32","name":"_greeting","type":"bytes32"}],"name":"setGreeting","outputs":[],"stateMutability":"nonpayable","type":"function"}]



```python
!solc --bin src/GreeterB.sol
```

    
    ======= src/GreeterB.sol:GreeterB =======
    Binary:
    608060405234801561001057600080fd5b507f48656c6c6f00000000000000000000000000000000000000000000000000000060008190555060c7806100466000396000f3fe6080604052348015600f57600080fd5b506004361060325760003560e01c806350513b4f146037578063cfae3217146062575b600080fd5b606060048036036020811015604b57600080fd5b8101908080359060200190929190505050607e565b005b60686088565b6040518082815260200191505060405180910390f35b8060008190555050565b6000805490509056fea2646970667358221220569ff00f36b612b13c5f4906994b92b30bfbfc6ab25826e600a94a44d18ff11a64736f6c63430006010033


```string```을 ```byte32```로 변경하면 글자 수가 제한되므로, gas비가 산정되는 것을 볼 수 있다.
위에서는 infinite로 산정된 것을 기억해보자.


```python
!solc --gas src/GreeterB.sol
```

    
    ======= src/GreeterB.sol:GreeterB =======
    Gas estimation:
    construction:
       20107 + 39800 = 59907
    external:
       greet():	1013
       setGreeting(bytes32):	20220



```python
!node src/getMyAddr.js
```

    [ '0x1a694B14D7e7eFf1F8A788E4F4f535aDfD164378',
      '0xae52FF68C2525c0De82E9cdf73333B7653Ec7C60',
      '0x06112A4dd2C07dF56c8c228fc84249D8401eFdF9',
      '0xF4d70043D4bbC309265AB5050e16cbc234dFba86',
      '0xf43CCed11cdb4978B19eD346eAEF16e5E294903C',
      '0x86938655342a8DA12914e2B07129447B64c71B28',
      '0xC74f7d1BEF7a15ACa7511c1a3dDba2D989b98304',
      '0x5603a1F1e40214c59328d0F6c8d9e19c18f07F0d',
      '0x3E632Cf0Bb1b966f7868Fde61C20CF520141546b',
      '0x89771d784bd8D5aE0A165aa59537CFdAd389d7Ba' ]



```python
%%writefile src/GreeterBDeploy.js
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
}
var _abiArray=[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[],"name":"greet","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"bytes32","name":"_greeting","type":"bytes32"}],"name":"setGreeting","outputs":[],"stateMutability":"nonpayable","type":"function"}];
var _bin="608060405234801561001057600080fd5b507f48656c6c6f00000000000000000000000000000000000000000000000000000060008190555060c7806100466000396000f3fe6080604052348015600f57600080fd5b506004361060325760003560e01c806350513b4f146037578063cfae3217146062575b600080fd5b606060048036036020811015604b57600080fd5b8101908080359060200190929190505050607e565b005b60686088565b6040518082815260200191505060405180910390f35b8060008190555050565b6000805490509056fea2646970667358221220569ff00f36b612b13c5f4906994b92b30bfbfc6ab25826e600a94a44d18ff11a64736f6c63430006010033";
var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
_contract
    .deploy({data:"0x"+_bin})
    .send({from: "0x1a694B14D7e7eFf1F8A788E4F4f535aDfD164378", gas: 364124, gasPrice: '1000000000'})
    .then(function(newContractInstance){
        console.log(newContractInstance.options.address) // instance with the new contract address
    });
```

    Writing src/GreeterBDeploy.js



```python
!node src/GreeterBDeploy.js
```

    0x6A107Ac45F0C516D865aD426aE356B19e967a2C5


"Hello SMU"와 같은 String을 인자로 사용하면, 오류가 발생한다.
반면에 "Hello SMU"에 해당하는 16진수 "0x48656c6c6f20534d55"를 넘겨주면,
16진수 결과가 반환된다.


```python
%%writefile src/GreeterBUse.js
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var _abiArray=[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[],"name":"greet","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],"stateMutability":"view","type":"function"},{"inputs":[{"internalType":"bytes32","name":"_greeting","type":"bytes32"}],"name":"setGreeting","outputs":[],"stateMutability":"nonpayable","type":"function"}];
var greeter = new web3.eth.Contract(_abiArray,"0x6A107Ac45F0C516D865aD426aE356B19e967a2C5");
greeter.methods.greet().call().then(function(value) {console.log(value);});
greeter.methods.setGreeting("0x48656c6c6f20534d55").send({from:"0x1a694B14D7e7eFf1F8A788E4F4f535aDfD164378",gas:100000});
//ERROR greeter.methods.setGreeting("Hello SMU").send({from:"0x1a694B14D7e7eFf1F8A788E4F4f535aDfD164378",gas:100000});
greeter.methods.greet().call().then(function(value) {console.log(value);});
```

    Overwriting src/GreeterBUse.js



```python
!node src/GreeterBUse.js
```

    0x48656c6c6f20534d550000000000000000000000000000000000000000000000
    0x48656c6c6f20534d550000000000000000000000000000000000000000000000


## 연습문제: Timer


```python
pragma solidity ^0.5.0;
contract Timer {
    uint256 startTime;
    uint256 endTime;
    function start() public {
        startTime=now;
    }
    function stop() public {
        endTime=now;
    }
    function timePassed() public view returns(uint256) {
        return endTime-startTime;
    }
    function getNow() view public returns(uint) {
        return now;
    }
}
```

## 연습문제: 7 곱하는 컨트랙

### 문제

어떤 수에 7을 곱하는 스마트 컨트랙을 만든다.


### 해결

컨트랙은 ```Multiply7```, 함수는:

```python
function multiply(uint input) public pure returns (uint)
```

- 컨트랙은 Solidity로, 클라이언트는 노드로 프로그램한다.

- 8을 인자로 너기면, 7을 곱해서 56을 출력하세요.

단계 | 방식 | 결과
-----|-----|-----
개발 | solidity | ok
컴파일 | eth | ok
배포 | _class.new | ok (transactionHash는 배포하지 않아도 생성)
배포 | _class.new, async 방식 | nok  (geth console에서만 ok)
생성 | async방식 | nok (geth console에서만 ok)

* geth console에서의 사용 예.
```
> myMultiply=eth.contract(compiled.test.info.abiDefinition).at(_object.address)
{
  abi: [{
      constant: false,
      inputs: [{...}],
      name: "multiply",
      outputs: [{...}],
      payable: false,
      type: "function"
  }],
  address: "0x050c96a751f86a699a541001223b1ba7f5364d8d",
  transactionHash: null,
  allEvents: function(),
  multiply: function()
}
> myMultiply.multiply.call(6)
'42'
```


```python
%%writefile src/Multiply7.sol
pragma solidity ^0.5.0;

contract Multiply7 {
   function multiply(uint input) public pure returns (uint) {
      return input * 7;
   }
}
```

    Overwriting src/Multiply7.sol



```python
!solc --gas --abi --bin src/Multiply7.sol
```

    
    ======= src/Multiply7.sol:Multiply7 =======
    Gas estimation:
    construction:
       93 + 38800 = 38893
    external:
       multiply(uint256):	291
    Binary: 
    608060405234801561001057600080fd5b5060c28061001f6000396000f3fe6080604052348015600f57600080fd5b50600436106045576000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114604a575b600080fd5b607360048036036020811015605e57600080fd5b81019080803590602001909291905050506089565b6040518082815260200191505060405180910390f35b600060078202905091905056fea165627a7a723058200bedf7a8055c5e72976164e3b7fb22ab8a88ab449168b60b5cf7ba2203eb8a240029
    Contract JSON ABI 
    [{"constant":true,"inputs":[{"name":"input","type":"uint256"}],"name":"multiply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"pure","type":"function"}]


0

The problem here is upon creation of your contract you do not send enough gas. By default it is 90000 ( for Geth ) or the results given after creating it are

Transaction cost: 112321 gas. 
 Execution cost: 44689 gas.


```python
%%writefile src/Multiply7Deploy.js
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8445"));
}
var _abiArray=[{"constant":true,"inputs":[{"name":"input","type":"uint256"}],"name":"multiply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"pure","type":"function"}];
var _bin="608060405234801561001057600080fd5b5060c28061001f6000396000f3fe6080604052348015600f57600080fd5b50600436106045576000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114604a575b600080fd5b607360048036036020811015605e57600080fd5b81019080803590602001909291905050506089565b6040518082815260200191505060405180910390f35b600060078202905091905056fea165627a7a723058200bedf7a8055c5e72976164e3b7fb22ab8a88ab449168b60b5cf7ba2203eb8a240029";
var _contract = web3.eth.contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
var _instance=_contract.new({data:"0x"+_bin,from:web3.eth.accounts[0],gas:1000000}, function(err, contract) {
    if (!err) {
        console.log("contractAddress: ", contract.address);
        console.log("transactionHash: ", contract.transactionHash);
    }
});
```

    Overwriting src/Multiply7Deploy.js


* asynch 방식이라 마이닝이 될 때까지 기다려야 한다.


```python
!node src/Multiply7Deploy.js
```

    contractAddress:  undefined
    transactionHash:  0x875388e6058ba088d323a3056260c55c504faf4d3aa807b45819a49cb976e0fd
    contractAddress:  0x276c3db04f43197e5f67f79b65f7394cd8dd2627
    transactionHash:  0x875388e6058ba088d323a3056260c55c504faf4d3aa807b45819a49cb976e0fd



```python
!node src/Multiply7Deploy.js
```

    contractAddress:  undefined
    transactionHash:  0x7f9393b66a3b6f8a048eb6a6ae1d8ffbfa5c70471c819e252a3e1e83023748ad
    contractAddress:  0xb0858506a4b677e06af14a02be37011bd4196cd5
    transactionHash:  0x7f9393b66a3b6f8a048eb6a6ae1d8ffbfa5c70471c819e252a3e1e83023748ad


* 위에서 만들어진 abi, contract address를 사용한다.
* call은 로컬에서 실행하기 때문에, hash값도 없고 gas도 필요하지 않다. 블록체인에 기록되지 않는 거래이다.
* sendTransaction으로 실행하면 블록체인에 기록된다.

* transactionHash
    * at()으로 생성하면, 거래가 발생하지 않았으므로 (주소에서 컨트랙을 생성했다), transactionHash는 null
    * 계정에서는 여러 거래를 발생하므로, 거래를 확인하려면 id인 transactionHash가 필요.


```python
%%writefile src/Multiply7Use.js
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8445"));
var _abiArray=[{"constant":true,"inputs":[{"name":"input","type":"uint256"}],"name":"multiply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}];
var _contract = web3.eth.contract(_abiArray);
var multiply7 = _contract.at("0x276c3db04f43197e5f67f79b65f7394cd8dd2627");
console.log("multiply7: ", multiply7.multiply.call(6).toNumber());
```

    Overwriting src/Multiply7Use.js



```python
!node src/Multiply7Use.js
```

    multiply7:  42


* geth console이 아니라서 async nok. 따라서 거래가 대기 상태.
    * input은 소스의 bin code
    * from은 전송 계정
    eth.getBlock("pending", true).transactions;
    
```
현재 대기 거래 건수를 확인할 수 있다.
> txpool.status
{
  pending: 1,
  queued: 0
}

대기가 일정 시간 걸리면, 마이닝을 직접한다. 완료되면 주소가 생성된다.
> miner.start(1); admin.sleepBlocks(1); miner.stop();
true
> eth.pendingTransactions 대기 거래 처리됨 (블럭체인으로 넣어짐)
> _object
{
  address: '0x837b3d973f29559e8edab1abd4c0fec6e1445d12',
  transactionHash: '0xf48e7e262bae6566f8592c5a532e389854782130a995c75765829657ce17e6ff',
  allEvents: function (),
  multiply: function ()
}
```


```python
!geth --exec 'eth.pendingTransactions;' attach http://117.16.44.45:8445
```

    [{
        blockHash: [1mnull[0m,
        blockNumber: [1mnull[0m,
        from: [32m"0x2e49e21e708b7d83746ec676a4afda47f1a0d693"[0m,
        gas: [31m90000[0m,
        gasPrice: [31m20000000000[0m,
        hash: [32m"0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"[0m,
        input: [32m"0x6060604052346000575b60458060156000396000f3606060405260e060020a6000350463c6888fa18114601c575b6000565b346000576029600435603b565b60408051918252519081900360200190f35b600781025b91905056"[0m,
        nonce: [31m59[0m,
        r: [32m"0xb35022f42f760331f2d2e0bb5530d05948da599b3b03510c2007a86eecf147b9"[0m,
        s: [32m"0x484e7f295f9999dd20da97c569c99bd34f0023260677ff46a2662c7ef9c8dcce"[0m,
        to: [1mnull[0m,
        transactionIndex: [1mnull[0m,
        v: [32m"0x1c"[0m,
        value: [31m0[0m
    }]


* 위 transactionHash를 가지고 transactionReceipt()를 실행하여 주소를 사용한다.
    * 주소는 마이닝 후 생성된다. 마이닝하지 않으면 주소는 null
    * transactionHash
    "0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"
    * contractAddress: "0x13581bb3c23492b722f230e967a0232741ccd247"
    * 주소의 bin code도 확인할 수 있다.
 


```python
!geth --exec 'eth.getTransactionReceipt("0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9");' attach http://117.16.44.45:8445
```

    {
      blockHash: [32m"0x62c6f4ea5b26f1efbbbfa78dac40b29a6f8a9c3c14510b98301e8c16056a060e"[0m,
      blockNumber: [31m61711[0m,
      contractAddress: [32m"0x13581bb3c23492b722f230e967a0232741ccd247"[0m,
      cumulativeGasUsed: [31m40597[0m,
      from: [32m"0x2e49e21e708b7d83746ec676a4afda47f1a0d693"[0m,
      gasUsed: [31m40597[0m,
      logs: [],
      logsBloom: [32m"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"[0m,
      root: [32m"0xdf63ca69b3ac58bf423229d501dff8897306e2b0c9a821c2fa6b4f19c980c98c"[0m,
      to: [1mnull[0m,
      transactionHash: [32m"0x7ccd5dc3a6498394320201fe09f9553cd5ce4aaa957363a6d2baca4be8309bd9"[0m,
      transactionIndex: [31m0[0m
    }



```python
!geth --exec 'eth.getCode("0x13581bb3c23492b722f230e967a0232741ccd247");' attach http://117.16.44.45:8445
```

    [32m"0x606060405260e060020a6000350463c6888fa18114601c575b6000565b346000576029600435603b565b60408051918252519081900360200190f35b600781025b91905056"[0m




## 연습문제: 계수기와 Timer 콘트랙

* (1) Counter 컨트랙을 구현하세요. add(), subtract(), getCount() 함수를 구현하고, 여기에 호출하는 시점을 출력하기 위해 다음 함수를 추가하세요. 강의자료의 Counter와 Timer를 참조하세요.
```
function setTimeCalled() internal 호출시점을 설정
function getTimeCalled() view public returns(uint256) 호출시점을 조회
```

REMIX에서 각 함수를 실행해보고, 우하단 창에 출력되는 성공로그를 복사해서 붙여넣기 하세요. 아래는 샘플입니다.
```
[vm]from: 0x5B3...eddC4to: Counter.add() 0x0fC...9A836value: 0 weidata: 0x4f2...be91flogs: 0hash: 0x6a7...a14d6
transact to Counter.subtract pending ... 
[vm]from: 0x5B3...eddC4to: Counter.subtract() 0x0fC...9A836value: 0 weidata: 0x6de...ebae3logs: 0hash: 0x5aa...6e62a
call to Counter.getCount
CALL [call]from: 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4to: Counter.getCount()data: 0xa87...d942c
call to Counter.getTimeCalled
CALL [call]from: 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4to: Counter.getTimeCalled()data: 0xc45...7a0a4
```

* (2) 로컬에서 컴파일하세요. 파일로 abi, bytecode를 작성하세요.
* (3) 파일로부터 abi, bytecode를 읽어서 배포하세요. 배포부터 geth 8445로 연결하여 node로 하세요 (ganache 8345로 하면 감점).
* (4-1) 배포된 주소로 부터 객체를 생성하고, 바로 계수와 호출시점을 출력하세요.
* (4-2) 계수를 1회 증가시키세요. 증가한 후, 증가된 계수와 호출시점을 출력하세요.
* (4-3) 위 (4-2)를 1회 반복하세요.

끝


```python
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Counter {
    uint256 count;
    uint256 timeCalled;

    constructor() {
        count=0;
        timeCalled=0;
    }
    function add() public {
        count++;
        setTimeCalled();
    }
    function subtract() public {
        count--;
        setTimeCalled();
    }
    function getCount() public view returns (uint256) {
        return count;
    }
    function setTimeCalled() internal {
        timeCalled=block.timestamp;
    }
    function getTimeCalled() view public returns(uint256) {
        return timeCalled;
    }
}
```

## 연습문제: 거듭제곱 컨트랙

### 문제

아래 (2) ~ (6)은 노드로 프로그램하세요.

(1) 2의 거듭제곱을 계산하는 컨트랙을 프로그램하세요.
8을 입력하면 256, 32는 4294967296을 출력합니다.
컨트랙은 Math.sol로 저장하고, 함수는 powerOf2로 명명해서 프로그램 작성
power를 계산하려면, 반복문으로 하지 않는다. ** 연산자로 계산합니다.
예: 2**2로 코딩하면 4를 얻는다.
	컴파일해서 abi, gas, bin을 화면에 출력한다.
(2) ganache 배포
(3) ganache 사용 (8과 32를 입력해서 결과 출력)
(4) 웹페이지 powerOf2.html을 작성 (32를 입력하고, 결과를 화면에 출력하고 캡쳐해서 별도 제출)
(5) 파일에서 ABI, BIN을 읽고 배포하고 사용
(6) geth에서 배포하고 사용. geth@8446, geth@8445 가운데 선택.


### 결과
uint256 e = uint256(a)**uint256(b);


```python
%%writefile src/Power.sol
pragma solidity ^0.6.0;

contract Math {
   function powerOf2(uint p) public pure returns (uint) {
      return 2 ** p;
   }
}
```

    Overwriting src/Power.sol



```python
!solc --optimize --combined-json abi src/Power.sol > src/PowerABI.json
```


```python
!cat src/PowerABI.json
```

    {"contracts":{"src/Power.sol:Math":{"abi":"[{\"inputs\":[{\"internalType\":\"uint256\",\"name\":\"p\",\"type\":\"uint256\"}],\"name\":\"powerOf2\",\"outputs\":[{\"internalType\":\"uint256\",\"name\":\"\",\"type\":\"uint256\"}],\"stateMutability\":\"pure\",\"type\":\"function\"}]"}},"version":"0.6.1+commit.e6f7d5a4.Linux.g++"}



```python
!solc --optimize --combined-json bin src/Power.sol > src/PowerBIN.json
```


```python
!cat src/PowerBIN.json
```

    {"contracts":{"src/Power.sol:Math":{"bin":"6080604052348015600f57600080fd5b5060958061001e6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c8063bdacc0cf14602d575b600080fd5b604760048036036020811015604157600080fd5b50356059565b60408051918252519081900360200190f35b60020a9056fea264697066735822122012bb3f261b93f67b95de6d9316162d32278824a1da1610ce402da6a4cad966cd64736f6c63430006010033"}},"version":"0.6.1+commit.e6f7d5a4.Linux.g++"}



```python
%%writefile src/PowerDeployAbiBinFromFile.js
var Web3=require('web3');
var _abiJson = require('./PowerABI.json');
var _binJson = require('./PowerBIN.json');

var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
}

contractName=Object.keys(_abiJson.contracts); // reading ['src//Power.sol:Power'];
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
_bin=_binJson.contracts[contractName].bin;
console.log("- ABI: " + _abiArray);
console.log("- Bytecode: " + _bin);


var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
_contract
    .deploy({data:"0x"+_bin})
    .send({from: "0x73A5Abb02eA6857BAeD2CDE51e4678d6f16C9593", gas: 364124, gasPrice: '1000000000'})
    .then(function(newContractInstance){
        console.log("- Contract Address: "+newContractInstance.options.address) // instance with the new contract address
    });
```

    Writing src/PowerDeployAbiBinFromFile.js



```python
!node src/PowerDeployAbiBinFromFile.js
```


```python
%%writefile src/PowerUse.js
var Web3=require('web3');
var _abiJson = require('./PowerABI.json');
var _binJson = require('./PowerBIN.json');

var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));

contractName=Object.keys(_abiJson.contracts); // reading ['src//Power.sol:Power'];
console.log("- contract name: ", contractName[0]); //or console.log(contractName);
_abiArray=JSON.parse(_abiJson.contracts[contractName].abi);    //JSON parsing needed!!
//_bin=_binJson.contracts[contractName].bin;
var power = new web3.eth.Contract(_abiArray,"0x50Ce5765E82A10aB0Fa0f3A13AacbCe0Db1D112A");
power.methods.powerOf2(8).call().then(function(value) { console.log("2^8: "+value); });
power.methods.powerOf2(32).call().then(function(value) { console.log("2^32: "+value); });
```

    Overwriting src/PowerUse.js



```python
!node src/PowerUse.js
```




```python

```

## 연습문제: Counter 짝수, 홀수 판별

Counter 콘트랙의 계수가 홀수인지, 짝수인지 판별하는 함수 ```isEven()```을 추가하세요.
geth 8445에서 실행하세요.

* Counter를 0부터 3까지 증가 (REMIX 우측 하단 함수실행 로그 붙여넣기)
* counter가 짝수일 때, isEven()을 실행해서 결과를 출력 (timestamp 포함)

## 연습문제: Counter 4 판별

REMIX에서 배포하고, 아래를 노드로 프로그램하세요
* Counter를 0부터 3까지 증가
* counter가 4일 때, isEven()을 실행해서 결과를 출력


## 연습문제: Counter 다른 세션에서의 값 확인

노드로 배포하고, 아래를 노드로 프로그램하세요.
* counter를 조회하고, 위 REMIX에서 배포한 콘트랙의 counter의 수와 같은지 확인.
* Counter를 0부터 3까지 증가
* 노드 세션을 새로 열어, Counter를 하나 더 증가시키려고 할 때 0부터 시작하는지 또는 전 세션에서 끝낸 counter의 수가 남아 있는지 출력
* counter가 4일 때, isEven()을 실행해서 결과를 출력

