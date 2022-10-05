# Chapter 5. 블록체인의 핵심인 트랜잭션

블록체인에 기록되는 트랜잭션과 기록되지 않는 메시지를 설명할 수 있게 된다. 기계코드 수준으로 함수를 호출하는 방법인 ABI를 작성할 수 있게 된다. 블록들을 해시값으로 서로 연결하고, 어떻게 인증하는지 배우게 된다. 트랜잭션을 처리할 때 발생하는 수수료 gas를 계산할 수 있게 된다. 거래건수를 계산하고, 왜 필요한지 설명할 수 있게 된다. 목차는 다음과 같다.
* 1. 트랜잭션
* 2. ABI 명세
* 3. 트랜잭션 구성
* 4. 트랜잭션 처리 단계
* 5. 블록체인
* 6. 마이닝
* 7. gas
* 8. 거래 건수

# 1. 트랜잭션

블록체인에서의 거래(transaction)는 금액이 이체되는 것을 의미하기 때문에, 입금계정, 출금계정, 금액이 반드시 필요하다.

온라인쇼핑과 블록체인의 거래를 서로 비교해 보자. 온라인 쇼핑에서는 로그인해서 상품 선택, 주문, 지급, 배송까지 하는 일련의 절차로 구성된다. 지급 버튼을 누르는 순간, 발생하는 거래가 블록체인의 거래와 다를 바 없다.

다만, 지급에 필요한 데이터, 금액, 송수신 내용이 블록체인의 것과 다를 수 밖에 없다.

온라인 쇼핑의 거래는 주문내역이 텍스트 형태로 URL 인코딩 되고 (웹에서 인코딩되는 방식이다. 보통 인코딩과 달리 앞에 %가 붙기 때문에 퍼센트 인코딩이라고 부른다), 블록체인에서는 이진수로 인코딩된 바이트코드로 전송된다. 인코딩하는 방식이 다를 뿐이다.

금액은 서로 실물가치가 있는 금액이어야 한다. 블록체인에는 금액을 적는 'value' 필드가 있다. 거기에 적어야 한다. 또한 'gas'라는 비용이 발생한다. 은행의 입출금 수수료와 같다.

거래 금액이 정해지면, 온라인 쇼핑이나 블록체인이나 송수신이 당연히 필요하다. 블록체인에서는 송수신 대상이 'address'라고 하는 암호화되어 해킹이 거의 불가능한 컴퓨터가 이해하는 16진수로 되어 있다는 점이 차이라고 볼 수 있다.

다음 표는 

구성 | 온라인 쇼핑 | 이더리움 거래 | 비교
-----|-----|-----|-----
데이터 | 웹에서 작성된 주문내역이 텍스트 형태로 구성됨 | **바이트코드** | 블록체인은 EVM에서 처리할 수 있는 바이트코드로 작성
value | 은행, 카드, 페이와 연계된 금액 | Ether | 모두 실물가치가 있는 금액, 단 블록체인에서는 'gas' 또는 'value' 필드에 적어서 전송
송수신 | 신용카드, 은행계좌, 페이 | Address | 블록체인에서는 주소 'address'로 송금 (은행 없이)

거래는 블록체인에 기록이 된다. 이 특징이 블록체인을 안전한 매체라고 강조하고 있다고 설명한 바 있다.

기록이 되지 않는 경우 (외부에서 조회 전용 함수를 호출하거나 내부에서 스마트 계약간에 호출되는 경우) "**메시지 호출(Message call)**" 또는 **호출(call)**이라고 한다. 이 경우 **기록되는 거래(Transaction)와 구별**된다. 메시지 호출은 송신, 수신, 데이터, Value, gas 등의 항목을 가질 수 있다는 점에서 거래(트랜잭션)와 비슷하다.

반면에 외부에서 호출되는 경우에는 **sendTransaction함수**를 사용하고, "**거래**"라고 블록체인에 기록된다.

다르게 설명하면, 거래는 계정의 서명이 필요한지에 따라 구분된다. **메시지 호출 call은 서명을 하지 않고**, **거래는 서명**을 해야 한다. 다시 말하면, 거래는 전송자에 의해 서명이 된 메시지를 말한다.

함수 | 설명 | 블록체인에 기록
-----|-----|-----
call | 로컬에서 실행, 블록체인에 전송되지 않는다. 따라서 블록체인을 변경하지 않고, 읽기만 하는 경우, 비용이 발생하지 않음 | 기록되지 않는다.
sendTransaction | 외부계정에 의해 발생하고, 사인해서 블록체인에 전송된다. 블록체인을 변경할 경우 사용 | 기록된다. 반환값은 hash.


## 1.1 call

call() 함수는 로컬 블록체인에서의 호출이라 블록체인에 전송되거나 기록되지 않는다. 예를 들어 거래의 내역을 조회한다고 하자. 이런 것은 call이고, 블록체인에 기록되지 않는다는 의미이다.

이 함수는 읽기 전용 호출이라서 블록체인의 상태에 변화를 줄 수 없다. 그래서 **사인이 필요없고**, gas가 발생하지 않는다.

블록체인에 기록되지 않기 때문에, 네트워크 참여자가 그 함수가 호출되었는지 알지 못한다.

call() 함수는 실행되고 나서 반환 값이 있을 수 있다. 프로그래밍에서 함수를 호출 한 후 값이 반환될 수 있는 것과 비슷하다. 

web3의 API는 **web3.eth.call**이다. 차츰 배우게 된다.

## 1.2 sendTransaction

call과 대조적으로, sendTransaction() 함수는 블록체인에 전송되고 기록된다. **사인**해야 하고, gas가 필요하다.

트랜잭션은 **반환 값이 없다.** 정확하게 얘기하자면 반환값이 있기는 하지만, 그 거래의 해시 값이다. 합산을 한다고 가정하면, 합산 결과를 돌려받지 못하고, 대신 그 거래의 해시값이 반환된다.

프로그래밍을 할 때 반환 값은 함수가 올바르게 실행이 되었는지를 테스트할 때 유용하다. 블록체인에서 이러한 반환값이 없다는 것이 불편할 수 있다. 이럴 때 사용하는 방법은 로그를 이용하는 것이다. 예를 들어 **반환 값을 알려면 event를 발생**시켜서 로그에 적어주고, 나중에 그 로그를 확인해야 한다.

해시 값이 주어지면, 트랙잭션은 마이닝이 되어야 한다. 거래가 인증이 되려면 마이닝이라는 과정이 반드시 필요하다.

web3에서 제공되는 API는 **web3.eth.sendTransaction**이다.

트랙잭션을 코드로 예를 들어보자. 지금은 코드만 확인하기로 하고, 나중에 실제 코딩하고 실행하게 된다.
```python
coin.send(web3.eth.accounts[0],100,{from:web3.eth.accounts[0],gas:100000});
coin.send.sendTransaction(web3.eth.accounts[0],100,{from:web3.eth.accounts[0],gas:100000});
```

수신, 송신, 금액, 수수료 내역을 send() 함수에 적어도 되고, sendTransaction() 함수에 적어도 된다.
- 수신: accounts[0]
- 송신: accounts[0]
- 금액: 100 Wei
- 수수료: 100000 Wei

# 2. ABI 명세

Application Binary Interface (ABI)는 컨트랙의 함수를 호출하기 위해 사용하는 방식이다.

개발자가 하는 것이 아니라, 프로그램 모듈 간의 인터페이스로 쓰이기 때문에 기계코드 수준의 작동방식이다.  프로그래밍에서 모듈의 인터페이스를 호출할 때 자주 사용하는 API, 단 **기계코드 수준의 API**라고 이해하면 된다.

블록체인 외부에서나 또는 블록체인 내부에서 **컨트랙 서로 호출할 때, 함수 및 인자를 ABI 표준에 따라 부호화**해서 호출하게 된다.

이더리움에서 제공하는 예제를 통해 설명해보자 (https://solidity.readthedocs.io/en/develop/abi-spec.html). Foo의 baz()함수를 호출한다고 가정한다. 블록체인에서 함수와 인자를 각각 인코딩하고, 조합해서 16진수 데이터로 만들고, 호출하는 과정을 보면서 ABI를 이해해보자.

```python
contract Foo {
  function bar(fixed[2] xy) {}
  function baz(uint32 x, bool y) returns (bool r) { r = x > 32 || y; }
  function sam(bytes name, bool z, uint[] data) {}
}
```

1) **함수명 부호화**

**함수 시그니처**, 즉 함수명과 인자를 적는다. 이때 인자는 **컴마로 분리하고 공백 없이** 적어 준다. 무심코 컴마 뒤에 공백을 넣지 않도록 한다.

함수 시그니처 ```baz(uint32,bool)```를 **sha3 해시값**으로 만들고, **앞 4 바이트**를 선택한다. 이를 **function selector**라고 한다.

solidity 코드로 앞 4바이트를 구하면:
```python
bytes4(bytes32(sha3("baz(uint32,bool)")))
```

이런 작업은 geth 콘솔 프롬프트 ```> ```에서 하면 바로 결과를 출력할 수 있다. 기억 나겠지만, _gethNow를 작성하면서, 원격접속을 허용하는 API에 web3를 적어 넣은 적이 있다. 그 web3를 사용하게 되는 것이다.

geth 콘솔 프롬프트에서 해보자. 다음에 보인 것처럼 web3.sha3() 함수와 substring() 함수로 앞 4 바이트를 선택할 수 있다. 0x를 포함하므로 10개를 분리하면 된다.

```python
geth> web3.sha3('baz(uint32,bool)').substring(0,10);
"0xcdcd77c0"
```

2) **매개변수 값을 부호화**

매개변수가 69는 16진수로 바꾸면 "0x45", 1은 "0x1"이다.

```python
geth> web3.toHex(69)
"0x45"
```
**32 바이트**로 만들기 위해 **앞에 0으로 채운다**.

```python
0x0000000000000000000000000000000000000000000000000000000000000045 - 인자 uint32 69 (32바이트)
0x0000000000000000000000000000000000000000000000000000000000000001 - 인자 bool 1  (32바이트)
```

3) **데이터 조합**

위 1) function selector부호값과 2) 인자부호값를 합친다. 0x를 제거한다. 아래에서 볼 때 몇 줄로 나누어져 있지만 그것은 공간상의 제약때문에 그러하고, 한 줄로 쭉 연결이 된 문자열이된다.

```python
0xcdcd77c00000000000000000000000000000000000000000000000000000000000000045
0000000000000000000000000000000000000000000000000000000000000001
```

4) **함수 호출**

함수를 호출할 때 ```data``` 항목에 위와 같이 생성된 ABI 부호를 적는다.

```python
geth> web3.eth.sendTransaction({
    from: eth.accounts[0],
    to: "0x672807a8c0f72a52d759942e86cfe33264e73934",
    data: "0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001",
    gas: 400000}
)
```

이 표준은 EVM에서 컨트랙의 함수를 호출할 때 사용하는 방식의 인코딩이기 때문에 복잡하다. 사람이 부호화되는 과정을 이해하고 코딩하기는 어렵다. 그러나 걱정하지 말자. 이더리움 내부에서 알아서 하므로 개발자가 직접 이 방식으로 코딩하지 않는다. 여기서는 내부적으로 이런 형태로 동작한다는 것으로 이해하자.

solidity의 **abi.encodeWithSignature()** 함수를 사용하면 ABI 부호화를 하지 않아도 된다.
```
<address>.call(abi.encodeWithSignature("baz(uint32,bool)",69,1))
```

또한 ABI를 사람이 직접해서 적어 주어도 된다. 인자가 길어서 부분적으로 생략하고 보였다.
```
addr.call("0xcdcd77c0..생략..00001")
```

## 3. 트랜잭션 구성

## 3.1 비트코인의 트랜잭션

비트코인에서의 거래는 한 주소에서 다른 주소로 금액이 이체되는 것으로, 네트워크에 전파되고 모여서 블록으로 만들어지게 된다.

거래는 다음 표에서 보인 것처럼 버전, locktime 타임스탬프, 입금, 출금 관련 항목으로 구성한다.

길이 | 항목 | 설명
-----|-----|-----
4 bytes | Version | 버전정보
1-9 bytes | Input counter | in 개수
variable | in | array로서 이전 거래의 out에서 가져와 본 거래의 out으로 지급된다.사용하게 되는 잔액을 가지고 있는 transactionHash, script를 가진다.
1-9 bytes | output counter | out 개수
variable | out | array로서 in보다 같거나, 적어야 한다 (적으면 처리비용으로). UTXO OpCodes: Scripts - 6...4사용하고 남는 잔액들 unspent transaction output UTXO. script, value를 가진다.
4 bytes | locktime | unix timestamp 또는 블록번호

비트코인의 거래에서 중요한 항목은 (1) 입금 (in), (2) 수신 (out), (3) 금액이다.
- in 항목에는 입금되는 주소와 잔액이 적힌다. 하나의 계정에서 모든 금액이 채워질 수도 있고 또는 여러 개가 필요할 수도 있다.
- out 역시 하나 또는 여러 계정이 필요할 수 있다. 수신계정에 입금하는 거래가 이루어지고 나서, 잔액 역시 하나 이상의 주소로 환급되어야 할 수 있다.

비트코인에서는 송금하고 나서 잔액을 환급하는 특징을 가지고 있다. **하나 이상의 in에서 입금액을 채우고, 하나 이상의 out에 그 잔액을 환급**하게 된다. 따라서 **미사용잔액**이 적힌다.

아래 사례에서 보듯이, 입출금이 하나가 아니고 복수일 수 있다는 점에 주의한다.

거래에서 발생한 미사용 잔액이 수합되어야 하는 경우 input이 여러 개가 된다. ```ins```는 이전 거래 ```outpoint``` 2건에서 미사용 잔액을 수합하여 ```output```으로 지급한다.

출금 또한 수신자가 1명이라도, 그 잔액이 자신에게 돌려지므로 **최소 2개**가 된다. 하나는 수신처로 송금되는 금액, 다른 하나는 거스름돈에 해당하는 잔액이다.

```Value```는 Satoshi 금액이다 (1 BTC = 100,000,000 Satoshi).


```python
>>> import bitcoin, pprint
>>> tx = '0100000002349cbebe5628707553812eb0e591dc49047d743e836006773a65b55cf4444a5a030000006a47304402203f02d209168bb9c4e8fab3b2f2aca19be1f9032eaa85be3a029641eecf69e60d022055d7e22914bb4741ee8c1ab73e23766de034a82dae22dad0c13684d6cb098b9e012102c3a7056ad278bf04f2e5546e29466d8175ec4a052ad8f9901b32d53e442545a8ffffffff0890e18ae1953971b7b64ddedcb75c034e5c16969e498358b517960fd0186238010000006b4830450221009511497e136765037a75f75a126168502bfee73a7c286ea775968a7140368ef002200cfc776ade17b9444bb7ad13e71852974befd1d6fcfa196df677cd72a697c63b012102b8963213667c71471d62361fe27aad7ab2b8b102a6d93f7367634466e04b5422ffffffff022085d0020000000017a914ed91639c578b4c1f526525e6dcbc9fbc505fe7698751bd0d00000000001976a9145f9a3e6e8029dd934a2a00a17aa8254c0975219988ac00000000'
>>> tx_structure = bitcoin.deserialize(tx)
>>> pprint.pprint(tx_structure)

    {'ins': [{'outpoint': {'hash': '5a4a44f45cb5653a770660833e747d0449dc91e5b02e815375702856bebe9c34',
                           'index': 3},
              'script': '47304402203f02d209168bb9c4e8fab3b2f2aca19be1f9032eaa85be3a029641eecf69e60d022055d7e22914bb4741ee8c1ab73e23766de034a82dae22dad0c13684d6cb098b9e012102c3a7056ad278bf04f2e5546e29466d8175ec4a052ad8f9901b32d53e442545a8',
              'sequence': 4294967295},
             {'outpoint': {'hash': '386218d00f9617b55883499e96165c4e035cb7dcde4db6b7713995e18ae19008',
                           'index': 1},
              'script': '4830450221009511497e136765037a75f75a126168502bfee73a7c286ea775968a7140368ef002200cfc776ade17b9444bb7ad13e71852974befd1d6fcfa196df677cd72a697c63b012102b8963213667c71471d62361fe27aad7ab2b8b102a6d93f7367634466e04b5422',
              'sequence': 4294967295}],
     'locktime': 0,
     'outs': [{'script': 'a914ed91639c578b4c1f526525e6dcbc9fbc505fe76987',
               'value': 47220000},
              {'script': '76a9145f9a3e6e8029dd934a2a00a17aa8254c0975219988ac',
               'value': 900433}],
     'version': 1}
```

## 3.2 이더리움의 트랜잭션

이더리움에서는 미사용잔액을 수합해서 사용하는 방식이 아니라, 트랜잭션의 구성항목이 **입금주소 from**, **출금주소 to**와 **금액 value** 로 단순하다. 잔고를 확인하기 위한 비트코인에서의 미사용잔액이 없다.

거래를 수행하려면, 다음과 같은 필요한 속성이 제공되어야 한다. ```from```에 20바이트 주소를 반드시 적어야 한다. ```from```을 제외한 나머지 항목은 선택항목이라 적지 않아도 된다. 단 ```nonce```는 동적으로 생성된다.

속성 | 바이트 | 설명
----|-----|-----
from | 20 바이트 | 전송 주소. 명시하지 않으면 web3.eth.defaultAccount
to | 20 바이트 | (optional) 수신 주소. 컨트랙 생성하는 경우는 당연히 명시하지 않아도 된다.
value | 32 바이트 | (optional) 전송 Wei 금액
gasLimit | 32 바이트 | (optional, default: To-Be-Determined) 거래처리에 허용되는 최대 개스량
gasPrice | 32 바이트 | (optional, default: To-Be-Determined) 거래처리에 필요한 개스가격 (Wei)
data | 제한 없슴 | (optional)  메시지 데이터 (바이트)
nonce | 32 바이트 | (optional) 계정에서 발생한 거래 순서 번호. 동일한 번호가 있으면 하나는 처리 거절, 또는 순서대로 처리 (즉 뒷 순서가 먼저 처리되고 앞 번호를 취소하는 이중거래 편법을 막음)
v, r, s | 1, 32, 32 바이트 | v, r, s는 거래의 signature에서 구하고, 공개키를 회복하기 위해 사용된다.사인에서 구할 수 있다. v는 공개키를 회복하기 위해 필요한 recovery id (27 또는 28을 가진다. 0, 1 또는 2 값을 가질 수도 있다), (r,s)에서 공개키를 회복하는 방법이 2가지가 있고, 그 중 하나를 식별하기 위해 사용된다. (r은 R의 is computed as the X coordinate of a point R, modulo the curve order n.)

예제로 거래를 구성해보면:

```python
transactionObject = {
  from: "0x...생략...A0A0A0A01",
  to: "0x...생략...A0A0A0A02",
  value: web3.eth.getBalance("0x...생략...A0A0A0A01"),
  gas: "----",
  gasPrice: "----",
  data: "0x----",
  nonce: 0,
}
```

# 4. 트랜잭션의 처리 단계

거래가 발생하면, 블록체인에서는 그대로 성립하는 것이 아니다. 그 거래는 마이너에 의해 인증되어야 한다.

갑이 을에게 송금 거래를 한다고 하자. 갑은 거래에 디지털서명을 하고, 그 거래는 참여 노드들에게 전파된다.
노드들은 거래를 받으면:
- 거래의 **디지털서명**을 인증하여 원본이 맞는지, 소유주의 서명이 맞는지, 
- 전송자가 **gas를 지불할 잔고**를 가지고 있는지, 
- gas가 **gas limit**을 초과하지 않는지 검증한다. 

마이너 노드들은 이 거래들을 묶어 블록으로 만들어 가고, 블록이 채워지면 이른바 '합의'를 해야 한다. 합의된 블록은 체인에 연결이 되어 간다.

거래가 블록으로 만들어지는 과정을 단계별로 설명해 보면 다음과 같다.

순서 | 단계 | 설명
-----|-----|-----
1 | 거래 생성 | 거래에 **디지털 서명을**하고, 전송하기 위해 바이트 단위로 변환, **직렬화 serialize**한다.
2 | 거래 전송 | 거래가 p2p 네트워크로 전송되고, 피어 노드들에게 서로 전파된다 (broadcast).
3 | 블럭 생성 | 일정 시간 동안 거래를 묶어서 블록이 될 때까지 **pendingTransaction**에 넣는다. 그러나 아직 블록체인에 연결되기 전이므로 후보 블록 **candidate block**이 된다. 마이너들이 이렇게 묶은 블록 후보는 네트워크로 전파된다. 마이너는 자신들이 거래를 수집하여 블록으로 만들기 때문에 서로 같지 않을 수 있다. 거래가 도착 순서대로 묶어지는 것이 아니라, **우선순위**가 높은 거래부터 시작해서 블록으로 만들어진다 (이더리움에서는 처리비가 높은 거래가 우선순위가 높다. 비트코인에서는 잔고UTXO의 age, 금액이 클수록 높다, Sum (Value of input x Input Age) / Transaction Size). 따라서 부모hash가 없는 **고아 거래orphan transactions**이 있을 수 있다. 포함이 안된 거래는 기다려서 다음 블록에 추가된다.
4 | 블록인증 | 앞서 만들어진 블록을 인증하게 된다. 마이닝은 Proof of Work, 즉 **hash puzzle을 풀어 nonce를 정하는 작업**이다. 블록 해시를 가장 먼저 찾아낸 노드는 이를 네트워크에 알리고, 다른 노드들은 새로 생성된 블록의 유효성을 검사하고 인증한다. **가장 긴 길이를 가진 체인을 선택**하여 (고아 블록은 짧은 길이를 가지게 되므로 선택되지 않는다), 블록을 앞 블록에 **전의 블록 hash를 찾아 체이닝**되어 블록에 추가된다. 한다. 참여한 노드들 중에서 가장 먼저 해시값을 찾아낸 노드가 보상을 받는다: 1) 새롭게 생성된 코인을 획득, 2) 거래비용. 2016개마다 문제의 난이도를 조정한다.
5 | 새로운 블록 전파 | 새로운 블럭이 공지되면 **다른 마이너들은 이 블록 해시값이 올바른지 검사하고 받아들이게 되면서 마이닝은 종료**하게 된다. 거래기록은 중앙 서버가 가지고 있는 것이 아니라, 분산 네트워크에 실시간 공유된다. 공유되고 나면 수정할 수 없다. 로컬 블럭체인 동기화, 마이닝하지 않은 경우 동기화가 뒤떨어질 수 있다.

# 5. 블록체인

## 5.1 블록헤더

### 블록헤더는 해시로 서로 연결된다.

블록은 헤더와 바디로 구분된다.

먼저 블록헤더를 보자. 가장 중요한 것은 현재 블록은 **전 블록의 해시**를 통해 연결된다. 이러한 블록의 연결이 블록체인을 구성하게 되는 것이다.

### 블록헤더에 포함되는 트리의 해시들

이더리움에는 4개의 트리가 있다. 여기서 트리는 trie라고 적히는데, tree와 같은 데이터 구조이다.

tree와 기능에 있어 다소 차이가 있어 발음을 트라이라고 해 구분하기도 한다.

트리를 이해하면서 아래 그림과 같이 보도록 하자.

* (1) 거래 트리 **Transaction Tries**: 거래의 모음으로 트리구조로 만들어 지고, 블록 당 하나씩 존재한다.
블록에 있는 모든 거래 정보는 자료구조에서 학습했던 트리(tree) 구조로 구성된다. 블록체인에서 사용된 트리 구조는 랄프 머클(Ralph Merkle)이라는 사람이 개발했던 특별한 트리 구조로 주로 검증을 목적으로 사용된다. 각 거래정보들을 이용해서 해시값들이 계산되고, 다시 이들을 묶어놓은 최종 해시값이 만들어진다.
* (2) 거래 수령 트리 **Transaction Receipt Tries**: 거래를 받았다는 증명을 모아서 트리구조로 만들어 지고, 각 블록 당 하나씩 존재한다.
* (3) 저장 트리 **Storage Tries**: 모든 컨트랙의 계정이 하나씩 가지고 있다.
계정을 키로 사용하고 (그래서 키는 32바이트), 이러한 키별로 컨트랙의 상태 데이터를 저장하고 있다.
* (4) 상태 트리 **State Trie**: 역시 계정 당 하나씩 존재한다. 앞서 설명한 바와 같이 계정에 대해 balance, nonce, storageRoot, codeHash를 가진다.


### 블록헤더는 다른 State, Transaction, TransactionReceipt Tries의 참조 값을 가진다.

블록은 다른 구조체에 대한 참조를 위해 다음과 같은 해시를 가지고 있다. 이것 역시 다음 그림을 보면서 이해하자.
- 앞서 설명한, 블록을 서로 연결하기 위해 prevHash (또는 parentHash) 전 블록의 해시
- 그리고 거래 트리의 해시 (블록은 **거래를 묶어 이진트리 merkle tree로 구성**해서 가진다)
- 거래수령트리 Transaction Receipt Trie의 해시 ~~(거래 수령 트리라는 것이 receipts trie와 transactions trie를 말하는건가요?)~~
- 상태트리 State Trie의 해시

![alt text](figures/5_blockheaderWithTries.png "block header with transaction, tansaction receipt and state tries")

~~앞에서 설명하는 trie와 그림의 내용이 다릅니다. 이름 수정이 필요합니다~~

### 지난 블록의 해시를 가지고, 이를 통해 블록체인이 된다.

다른 트리와의 연결을 위해 가지고 있는 해시 외에도 블록체인이 되는 핵심인 이전에 생성된 블록해시 값이 다음 블록 헤더의 'parentHash' 항목에 저장된다. 각 블록에 트랜잭션이 지속적으로 쌓이면 새롭게 블록을 생성하여 **블록해시 값을 연결고리로 블록을 연결하여 블록체인이 생성**된다.

블록 뒤에 다른 블록이 추가될 때마다, 이를 **confirmation**이라고 하는데 confirmation이 늘어날 수록 수정이 힘들어 진다. 
**fork**는 **같은 블록높이를 가지는 블록이 여러 개**, 부모가 여러 자식 블록을 가지는 경우 발생한다. 복수의 마이너가 거의 동시에 블록을 생산하는 경우이다.

매 **2016블록마다 difficulty가 조정**된다.

> 블록높이 block height

> 블록체인에서 선행블록의 수를 말한다. 예를 들어 genesis block의 블록높이는 0이다.

### 그 외의 블록헤더 항목들
그 외에도 채굴속도를 조정하는 난이도 (difficulty), 블록이 생성된 시간 (Timestamp), Target Hash를 맞추기 위해 사용하는 난스(nonce), 각 거래의 해시 값을 결합하여 계산하는 머클 루트의 해시값 (TransactionsRoot)으로 구성한다. 가벼운 클라이언트의 spv (simple payment verification)는 이러한 블록헤더를 가지게 된다. spv는 거래를 검증하기 위해서는 full node의 머클트리에 요청하여 거래가 블록에 포함되었는지 확인하게 된다.

항목 | 설명
-----|-----
parentHash | 이전 블록 해시 값으로 블록을 서로 체인으로 연결 (32바이트)
ommersHash | 블록내 ommers (또는 uncles) 의 해시 값. 동시에 블록이 2개 만들어질 수 있다. 이렇게 되면 어느 블록이 포함될 지 혼란스럽게 된다. 여기서 제외되는 블록이 ommer block(줄여서 ommer)이 된다. 부모의 형제를 의미하고 그래서 이전에는 uncle이라고 불리웠다. 
beneficiary | 현재 블록의 마이너 주소, 여기로 채굴수수료가 전송된다.
stateRoot | 계정상태를 가진 State trie의 루트 해시
TransactionsRoot | 블록에 있는 모든 거래 hash의 최상위 루트노드의 해시인 Merkle Root (32바이트). 따라서 어떤 블럭 해시 값이 변경되면 Merkle Root Hash값도 달라지게 되어 수정이 불가능함.
receiptsRoot | 거래영수증의 Root Hash
logsBloom | 로그에 포함된 색인정보
difficulty | 4바이트 난이도를 조정하여 블록생성시간을 유지.
number | 블록의 순번. 참고로 genesis block은 0이고, 그 후 순서대로 번호가 매겨진다.
Gas Limit | 허용된 개스 한도. 이 한도에 따라 몇 건의 거래가 블록에 포함될지 결정된다.예를 들어, 5천의 gas가 필요한 거래A, 6천의 gas가 필요한 거래B, 3천의 gas가 필요한 거래C가 있다고 하자. gas limit이 11,000이라면 거래A와 거래B가 블록에 포함되고, 거래C는 포함될 수 없다. 거래C를 덧붙이려면 '한도초과' 오류가 발생하게 된다.
Gas Used | 블록에 포함된 모든 거래에 소비된 gas의 총량
Timestamp | 마이닝이 끝나고 블록이 생성된 시간 (unix timestamp, 4바이트)
Extra Data | 별도로 저장하려는 데이터
mixHash | 작업증명에 사용되는 해시. 난스 값에서 생성.  Target Hash를 계산할 때, 헤더와 난스만 사용하는 것이 아니라 sha256(sha256(header + nonce + mixhash))에서와 같이 mixhash값이 포함된다.
nonce | 작업증명 PoW에서 Target Hash를 맞추어 나갈 때 조정하는 값 (4바이트)

geth 콘솔 프롬프트에서 특정 블록의 정보를 읽어보자. 위에서 설명한 항목들이 포함된 블록을 확인할 수 있다.
```getBlock()``` 함수에 현재 blockNumber 범위 내 임의의 번호를 적으면 된다.

```python
geth> eth.getBlock(55169)
{
  difficulty: 6314813,
  extraData: "0xd78
  3010504846765746887676f312e372e33856c696e7578",
  gasLimit: 4712388,
  gasUsed: 21000,
  hash: "0xd2c51...생략...94d26",
  logsBloom: "0x00000...생략...00000", 전부 0으로 채워져 있다. 로그에 적힌 내용이 없다는 의미이다.
  miner: "0x2e49e21e708b7d83746ec676a4afda47f1a0d693",
  mixHash: "0x4da21d2c5696ee4d798f6445db0287a73fb2332f089ac43690d302827f52401a",
  nonce: "0x3cd421957745a4b1",
  number: 55169,
  parentHash: "0x6533c71dd0f03108ea2c657896e73253f4f60ed3c4524a2bf0009cdec351d609",
  receiptsRoot: "0x925b88d8f207cf48a9dc303902f5674d239072c1f03493febc311549d820b9ce",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 645,
  stateRoot: "0xdbf466898fd964478d1f606e4a4dc1da9640af35f46d623d5ad4ed277526aaef",
  timestamp: 1481249262,
  totalDifficulty: 273764475970,
  transactions: ["0xd87121b8b0f84f7fa038cd7c1928ca6a222d14228125c90edc2493fdef4fb90b"],
  transactionsRoot: "0x5dbc80ced8364978e61d0c12aef1d7524092a4f7aa6ab30b4fccd6aa8b1282cd",
  uncles: []
}
```

## 5.2 블록바디

블록체인은 블록의 체인이고, 블록은 헤더와 바디로 구성된다고 설명하였다. 그렇다면, 블록 바디에는 무엇이 저장되어 있을까?

아주 간단하다. 블록바디는 발생한 트랙잭션을 저장한다. 그림에서 보듯이, 블록바디에는 **트랜잭션의 목록**, **ommer들의 목록**이 저장된다.

## 5.3 Merkle 증명

이더리움에서 블록은 거래를 트리구조로 연결하여 가지고 있다. Merkle tree는 **hash로 이름 붙여진 가지노드 lead node로 구성된 트리**를 말한다.

최정점의 머클루트는 가지들을 합쳐서 해시값을 구하고, 다른 노드에 대해서도 이 작업을 반복해서 결국 하나의 해시 값을 만드는 것이다. 

예를 들어, 트랜잭션 T1, T2를 머클트리로 구성한다면 각 거래 데이터를 해싱해서 나온 결과 값이 각 가지 노드에 저장된다. 부모 노드를 만들기 위해서 자식노드의 32바이트 크기의 트랜잭션 T1와 T2의 해시 값을 서로 연결해서 64바이트 문자열을 만들고 문자열은 **이중 해시** 처리되어 부모노드의 해시를 생성한다.

최하층 노드에 대해 두 개씩 결합이 끝나면, 이런 식으로 그 다음 계층으로 올라가 반복해서 해시값을 구한다.
이러한 방식으로 상위에 노드가 하나 남을 때까지 계속 계산하고, 각각의 트랜잭션을 이진트리 형태로 만들 경우 가장 최종적으로 남는 해시 값이 머클트리 루트 해시 값이 된다.


블록 내에서 머클 해시 값을 통해서 **트랜잭션의 무결성**을 검증 할 수 있고, 블록 해시 값을 통해서 **해더 값에 대한 무결성**을 검증할 수 있다. 거래 내역의 위변조를 막기 위해서 해시로 만들고 이것을 트리 형태로 만든 것이다.

이렇게 해시로 연결해 놓으면 어떤 장점이 있을까?

특정 transaction이 수정되면, 그 해시가 변경되게 되고, **블록에 변조가 발생하면, merkle tree로 root까지 전달**된다. 연쇄적으로 전체 해시가 다르게 되고, 블록체인 전체가 변조되기 때문에 위변조가 거의 불가능하게 된다.

https://bitcoin.stackexchange.com/questions/67621/checking-the-merkle-root-for-block-100000

```python
                             -------------------------------------------
                             |        바이트교환한 최종 머클루트         |
                             |                                         |
                             |                 머클루트                 |
                             -------------------------------------------
                           /                                            \
                          /                double SHA256                 \
                         /         바이트교환한 Hash 12 + Hash 34          \
                        /                                                  \
                   바이트교환                                           바이트교환   
              ------------------                                   ------------------
              |     Hash 12    |                                   |    Hash 34     |
              ------------------                                   ------------------
             /   double SHA256   \                                /   double SHA256   \
            /                      \                             /                      \ 
           / 바이트교환한 hash1+hash2 \                          / 바이트교환한 hash3+hash4 \
          /                           \                        /                           \
     바이트교환                  바이트교환                  바이트교환                  바이트교환
  ----------------           ----------------           ----------------           ----------------
  |    Hash 1    |           |    Hash 2    |           |    Hash 3    |           |    Hash 4    |
  ----------------           ----------------           ----------------           ----------------
         |                          |                          |                          | double
         | double SHA256            | double SHA256            | double SHA256            | SHA256 
 Transaction 1 Data         Transaction 2 Data         Transaction 3 Data         Transaction 4 Data
```

### 머클루트 구해보기

거래 txA, 거래 txB, txC, txD가 있다고 하자.물론 거래는 문자열만으로 구성되지 않는다. 이들을 묶어 머클루트를 구해보자. 그러려면, 데이터에 대해 해싱을 하고 바이트교환을 해나가야 한다.

먼저 거래 txA, txB, txC, txD를 해싱하고, 바이트교환을 한다. 그리고 노드의 해시를 결합하여 같은 작업을 반복해 나간다.

```python
>>> import hashlib

>>> txA = "Hello"
>>> txB = 'How are you?'
>>> txC = 'This is Thursday'
>>> txD = 'Happy new Year'
```

### 해싱

우선 거래데이타를 해싱하자.

Python 3.0부터는 문자열은 유니코드로 저장된다. ```encode()``` 함수는 문자열을 바이트문자열로 변환한다. 아래에서 보듯이 앞에 ```byte string```을 의미하는 ```b```가 붙는다.


```python
>>> "Hello".encode()
b'Hello'
```

바이트스트링에 대해 해싱을 하고, 또 더블해싱(결과값을 다시 해싱하는 것)을 한다. 더블해싱을 하게되면 해싱기법이 취약해서 공격 받지 않도록 좀 더 보안을 강화한다.~~길이 확장 공격에 대해서 따로 설명하지 않을거라면 이렇게 쓰는 것이 나을 것 같습니

```python
>>> _hashA=hashlib.sha256(hashlib.sha256(txA.encode()).digest()).hexdigest()
>>> _hashB=hashlib.sha256(hashlib.sha256(txB.encode()).digest()).hexdigest()
```

```digest()```는 바이트 형식으로, 반면에 ```hexdigest()```는 16진수로 출력한다.

```python
>>> hashlib.sha256(txA.encode()).digest()
b'\x18_\x8d\xb3"q\xfe%\xf5a\xa6\xfc\x93\x8b.&C\x06\xec0N\xdaQ\x80\x07\xd1vH&8\x19i'
```

```python
>>> hashlib.sha256(txA.encode()).hexdigest()
'185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969'
```

### 바이트 교환

바이트 교환 Byte Swap은 Endianess를 변환하는 것을 의미한다. BE Big-Endian을 LE Little-Endian으로 또는 반대로 변환한다.

문자열 HELLO의 예를 들어보자.

 구분| H | E | L | L | O
---|---|---|---|---|---
16진수 | 48 | 45 | 4C | 4C | 4F
BE 저장주소 (순서대로) | p | p+1 | p+2 | p+3 | p+4
LE 저장주소 (역순으로) | p+4 | p+3 | p+2 | p+1 | p

값이 뒤집히는 것이 아니라, 저장되는 주소만 변경되는 것으로 이해하면 된다.
즉 저장되는 메모리 순서가 BE와 LE 간에 차이가 있다.
* BE: 48-45-4C-4C-4F로 저장, 앞 글자가 먼저 빠른 주소에 저장, 즉 바이트 순서대로 저장 
* LE: 4F-4C-4C-45-48로 저장, 뒷 글자가 먼저 빠른 주소에 저장, 즉 바이트 역순으로 저장

hello 문자열로 할 때는 뭐 그리 큰 차이가 없어 보인다. 그러나 ```00000000000000000000000000000000000000000000000000000000000000045```와 같이 0이 많은 경우, LE에서는 45가 빠른 주소에 앞의 0은 다음에 저장된다. 

이런 변환은 Endianess의 차이때문에 한다. 컴퓨터에서 동일한 CPU에서 작업이 일어날 때는 문제가 되지 않는다.  따라서 우리는 거의 이 차이를 인식하지 않고 데이터를 저장하고 불러오고 한다.

그러나 컴퓨터에서 다른 컴퓨터로 데이터를 반출할 경우 문제가 될 수 있다. **이더리움, 자바와 같은 가상머신에서는 BE**, **인텔 x86등 CPU 하드웨어에서는 LE**를 사용한다. (~~엔디안을 바꾸는 것은 이렇게 역순으로 만드는 것이 아닐텐데요. 이더리움에서 쓰는 방식인가요?~~)

1 바이트, 즉 2 nibbles씩 출력해보자.  전체글자수 (즉 32바이트의 2배)에 대해 1 바이트인 2글자씩 반복을 하면서 출력한다.


```python
>>> print(_hashA)
70bc18bef5ae66b72d1995f8db90a583a60d77b4066e4653f1cead613025861c
```


```python
>>> for i in range(0, hashlib.sha256(txA.encode()).digest_size*2, 2):
...    print(_hashA[i:i+2], end=" ")

70 bc 18 be f5 ae 66 b7 2d 19 95 f8 db 90 a5 83 a6 0d 77 b4 06 6e 46 53 f1 ce ad 61 30 25 86 1c 
```
반복문을 pythonic하게 줄여볼 수 있다. ```digest_size```는 바이트 크기이므로, 2를 곱해서 전체 크기를 구할 수 있다. 그리고 2바이트씩 잘라서 ```reversed()``` 함수로 뒤집는다.


```python
>>> hashlib.sha256(txA.encode()).digest_size
32
```



```python
>>> "".join(reversed([_hashA[i:i+2] for i in range(0, hashlib.sha256(txA.encode()).digest_size*2, 2)]))
'1c86253061adcef153466e06b4770da683a590dbf895192db766aef5be18bc70'
```



```python
>>> hashAswap="".join(reversed([_hashA[i:i+2] for i in range(0, hashlib.sha256(txA.encode()).digest_size*2, 2)]))
>>> hashBswap="".join(reversed([_hashB[i:i+2] for i in range(0, hashlib.sha256(txB.encode()).digest_size*2, 2)]))
```


```python
>>> print("hashAswap: ", hashAswap, "\nhashBswap: ",hashBswap)
hashAswap:  1c86253061adcef153466e06b4770da683a590dbf895192db766aef5be18bc70 
hashBswap:  d0e7719d7633fb945d596090dfa31a95ae81b18d90352d63fc49af7f35ce2710
```

### 다음 단계의 해싱

#### 해시값을 결합한다.

앞서 구한 해시를 결합해서 다음 계층에서의 작업을 반복한다.


```python
>>> #hashAB=hashA.hexdigest()+hashA.hexdigest()
>>> hashAB=hashAswap+hashBswap
```

#### 또 해싱

더블해싱을 한다.

```python
>>> hashlib.sha256(hashlib.sha256(hashAB.encode()).digest()).hexdigest()
'e0c76d87a5a5c18ab29757603c5d1bda709306203b0a44c53fc6c90fba162903'
```

위 값을 변수에 저장한다.

```python
>>> _hashAB=hashlib.sha256(hashlib.sha256(hashAB.encode()).digest()).hexdigest()
```

### 또 바이트교환

바이트교환을 한다.

```python
>>> "".join(reversed([_hashAB[i:i+2] for i in range(0, 32*2, 2)]))
'032916ba0fc9c63fc5440a3b20069370da1b5d3c605797b28ac1a5a5876dc7e0'
```

위 값을 변수에 저장한다.

```python
>>> hashABswap="".join(reversed([_hashAB[i:i+2] for i in range(0, 32*2, 2)]))
```

### 해싱하고 바이트교환하는 함수 만들기

해싱하고 바이트교환을 함수로 만들어 해보자.

```python
def doubleHashByteSwap(raw):
    import hashlib
    size=hashlib.sha256(raw.encode()).digest_size
    _hash=hashlib.sha256(hashlib.sha256(raw.encode()).digest()).hexdigest()
    hashSwap="".join(reversed([_hash[i:i+2] for i in range(0, size*2, 2)]))
    return hashSwap
```

```python
>>> hA=doubleHashByteSwap(txA)
>>> hB=doubleHashByteSwap(txB)
>>> hAB=doubleHashByteSwap(hA+hB)
```

```python
>>> print("hashA: {0}\nhashB: {1}\nhashAB: {2}".format(hA, hB, hAB))

hashA: 1c86253061adcef153466e06b4770da683a590dbf895192db766aef5be18bc70
hashB: d0e7719d7633fb945d596090dfa31a95ae81b18d90352d63fc49af7f35ce2710
hashAB: 032916ba0fc9c63fc5440a3b20069370da1b5d3c605797b28ac1a5a5876dc7e0
```

# 6. 마이닝

## 6.1 Hash 맞추기

마이닝은 거래를 인증하는 과정으로, hash puzzle을 풀고 그 보상으로 새로운 코인을 생성하게 된다.

발생한 모든 거래가 추가되는 것은 아니다. **목표 값(target hash)를 찾아내는 계산**을 하고, 맞추는 경우에 인증되고 블록체인에 추가된다.

정답을 맞추는 작업을 하는 참여자들을 **마이너**라고 하며, **보상은 정답을 맞춘 최초의 경우**에만 해당 마이너에게 주어진다.

이와 같이 일정량의 계산에 따라 거래가 인증되고, 블록체인에 추가될 수 있으므로 이러한 **계산작업증명을 Proof of Work** 이라고 한다.

비트코인의 경우 블록생성시간은 10분으로 설정되어 있으므로 하루 144 블록, 일주일에 1008블록이 생성된다. 
이더리움은 문제의 난이도는 14초 내외에 1건씩 풀도록 정한다.

> 고아(Orphan) 블록, 스테일(Stale) 블록

> * **고아(Orphan) 블록**  블록체인에서 **부모가 없는 블록**을 말한다. 부모로 삼으려 했던 블록이 인증이 실패하면서 부모가 없어지는 경우 발생한다. 이더리움에서는 엉클 블록이라고도 한다. ~~이렇게 얘기가 되던데 맞나요?~~
> * **스테일(Stale) 블록** 블록은 **자식이 없는 블록**을 말한다. 마이닝은 되었지만, 블록높이가 동일한 다른 블록이 먼저 자식이 생기면서 정상적인 블록체인이 되는 경우 발생한다.

이더리움에서의 합의 알고리즘은 GHOST (Greedy Heaviest Observed Subtree, Zohar and Sompolinsky in December 2013)이다. 가장 무거운 트리가 블록체인에 연결되는 방식이다. 열매가 많이 매달려서 무거워진 나무의 가지를 연상하면 된다. 그 가지를 선택한다는 것이다. 비트코인은 가장 긴 블록을, 반면에 이더리움은 가장 무거운 블록을 추가한다. ~~가장 무겁다는 뜻을 설명해야 할까요?~~

> Hash Rate

> Hash Rate는 1초에 Hash를 몇 회하는지 (Number of hashes per second, H/s)
마이닝하는 속도를 말한다. 성능이 좋은 컴퓨터는 당연히 Hash Rate이 높게 마련이다.
H/s는 일반 척도와 같은 단위를 사용하여 나타낼 수 있다.
비트코인이 등장한 초기에는 보통 컴퓨터로 마이닝을 충분히 해낼 수 있었다.
채굴의 난이도가 높아지면서 특별히 마이닝을 하기 위한 **ASIC (Application Specific Integrated Circuit)** 칩을 사용한 고가의 컴퓨터가 사용되고 있다.

구분 | 단위 | 해시 회수
-----|-----|-----
Kilohashes | KH/s | 1000
Megahashes | MH/s | 1,000,000
Gigahashes | GH/s | 1,000,000,000
Terahashes | TH/s | 1,000,000,000,000
Petahashes | PH/s | 1,000,000,000,000,000


> **Ethash**

> 이더리움이 구현한 PoW 알고리즘이다. 이전에 사용하였던 Dagger-Hashimoto 알고리즘을 개선한 버전이다. Ethash에서는 "sha3_256", "sha3_512" 해시함수가 사용된다.

> Ethash는 Ethash DAG라고 하는 1 GB 데이터 분량을 사용한다. 전체를 가지고 있을 필요가 없는 light client는 16 MB 캐시를 사용한다. 이 데이터는 매 **30,000 블록마다 재생성**되는데, 이 시간을 **epoch**이라고 한다. 마이닝할 때 DAG의 일부를 가져다가 블록해시와 nonce를 사용해 target hash를 맞춰 나갈 때 사용한다.

> 마이닝을 하기 전에 보상이 지급되는 계정 coinbase를 정해야 한다. coinbase를 충전할 계정으로 변경하고, reward가 주어질 계정으로 마이닝을 시작한다. **처음 채굴을 시작하면, 기가규모의 블럭체인을 내려받아야 한다. 약 10분 정도 소요되며, DAG가 구성**된다.
```python
> miner.setEtherbase(eth.accounts[2])
true
> eth.coinbase
"0x53cbba17cf9bd0735855809bdcb88e232de96f32"
> miner.start();
```

## 6.2 난이도

**hash puzzle**은 **block hash**값을 결정하기 위해 풀어야 하고, 난이도에 따라 얼마나 어려운지 설정된다.

난이도는 식으로 표현하면, **```sha256(sha256(data+nonce)) < hash_difficulty```**이다.
* 식의 좌측은 데이터와 nonce의 더블해시 값을, 
* 식의 우측 ```hash_difficulty```는 현재의 난이도가 반영된 목표 해시이다.

이 식이 충족되도록 nonce를 찾아야 한다. 그 알고리즘을 적으면 다음과 같다. 즉, 블록헤더의 Hash값이 난이도 목표에 제시된 값 ```hash_difficulty```보다 작은값이 나오게 하는 Nonce값을 찾는 것이다.

```python
loop
    if sha256(sha256(data+nonce)) < hash_difficulty
        stop
    else change the nonce
```

예를 들어, **10분에 문제를 풀기를 기대했는데 5분에 풀었다**. 그러면 **난이도는 $\frac{10}{5}$, 2**가 된다. 그러면 **새로운 난이도 = 현재 난이도 x 2**이 된다. 즉 난이도가 2배로 증가하게 된다. 난이도가 1보다 크게 되면 새로운 난이도는 증가하고, 반대는 감소하게 되는 방식이다.

**새로운 목표값 = 이전 목표값 / 난이도** 즉, 난이도가 높아지면 새로운 목표 값이 낮아지고 맞추기가 더 어려워지게 된다.

예를 들어:
* 1부터 10 범위에 들어가는 수를 생성하는데 1분이 걸린다고 하자.
* 목표 값을 5로 정하고, 그 이내의 값이 나오려면 $60s \times \frac{10}{5}$ 즉 2분이 걸린다 (1~5의 사이이므로 2배의 시간)
* 목표 값을 3으로 정하면, $60s \times \frac{10}{3}$ 즉 3분 20초가 걸린다.

즉:
* 목표 값이 최대값에 가까우면 쉬워진다. 그 보다 작은 수를 찾는 것은 당연히 쉽게 된다. 예를 들어 $2^{255}-2$ (**최대 32바이트 hash값은 $2^{255}-1$**, 이 보다 하나 적은 블록 hash값을 찾는 것은 쉽다.
* 최대 값과 목표 값의 간극이 벌어질수록 어려워진다.
* 비트코인에서 difficulty는 2016개의 평균시간이 1,209,600초 (2주)되도록 정한다 (10분에 1개씩, 1주일에 7일 x 24시간 x 6개 = 1008개).

그 절차를 좀 더 자세하게 설명하면: 
(1) 거래를 수집하여 **블록을 만든다** (비트코인은 1MB 블록). 블록에 있는 모든 거래의 hash 합으로 Merkle Root를 계산한다.

(2) 블록헤더 (Version + Previous Block Hash + Merkle Root + Timestamp + Difficulty Bits + Nonce) 값을 SHA-256 해시하고 또 재해싱를 하여 **해시를 계산**한다.

(3) 2에서 계산된 해시값을 **Target hash와 비교**한다. target hash보다 적으면 정답, 아니면 2번으로 돌아가 nonce값을 증가함.

(4) 문제를 푼 경우, 블럭체인에 승인된 블럭을 맨 뒤에 첨부, 모든 참여자들에게 공지하고 참여자들로 하여금 계산을 하게 하여 검증되면 **합의**한다.

(5) 참여자가 많아져 계산속도가 빨라질 수 있지만 설정된 블럭생성시간에 맞추어 **난이도가 조정**된다. 파이썬에서 ```//```는 정수 나누기 연산이다.

```python
block_diff = parent_diff + parent_diff // 2048 * 
max(1 - (block_timestamp - parent_timestamp) // 10, -99) + int(2**((block.number // 100000) - 2))
```

즉, 부모블록과 현재블록의 생성시간의 차이에 따라 난이도 변경이 이루어진다.  ```parent_diff // 2048```는 난이도 변경범위 (bound divisor of the difficulty)이다.
* **10초 이하**: 난이도 **상향**, ```parent_diff // 2048 * 1``` 만큼 상향
* **10~19초**: 난이도 변동 없슴.
* **20초 이상**: 난이도 **하향** (timestamp 차이에 따라)최소 ```parent_diff // 2048 * -1```에서 최대 ```parent_diff // 2048 * -99```

(6) 블록이 한꺼번에 2개 마이닝 되는 경우,  두 마이너가 동시에 풀면 fork가 발생했다고 하고, 난이도가 높은 블록이 인증되고 체인에 연결된다. 난이도가 낮은 블록은 엉클 블록(고아 블록)이 된다. 

### 마이닝하면서 nonce를 정하는 연습

블록헤더 데이터의 해시 값에 NONCE를 증가시키면서 앞 자리의 0의 개수를 맞출 때까지 반복한다.

찾고자 하는 해시가 ```0000```로 시작한다고 하자. 그러면 최대값은 ```0000FFFF...```가 되겠다 (공간제약으로F를 모두 표현하지 않았다). 즉 그 해시의 최대 값보다 작은 범위 내에 들게 된다. 그러면 멈추고, 그 값을 해시로 정하게 된다.

비교할 Target 해시의 앞 자리 숫자를 정하기 위해서, 숫자에서 1을 제외하고 0만 세어보자. 예를 들어 10000에서 1을 제외하면 0000이 비교할 앞자리 숫자가 된다.

```python
>>> nzeros=4
>>> str(pow(10,nzeros))[1:nzeros+1]
'0000'
```

자릿수에 따라 난이도가 얼마나 영향을 받는지 보자. 함수를 작성해서 하면 편리하겠다.

```python
def mining(nzeros):
    import hashlib
    found=False
    blockNumber=54 # hex
    NONCE=0
    data='Hello'
    previousHash='5d7c7ba21cbbcd75d14800b100252d5b428e5b1213d27c385bc141ca6b47989e'
    while found==False:
        z=str(blockNumber)+str(NONCE)+data+previousHash
        guessHash=hashlib.sha256(z.encode('utf-8')).hexdigest()
        # e.g. nzeros=4 -> str(pow(10,nzeros))[1:nzeros+1] -> "0000"
        if guessHash[:nzeros]==str(pow(10,nzeros))[1:nzeros+1]:
            found=True
        NONCE+=1
        if(NONCE%pow(10,nzeros)==0):   #print guessHash every 10000000
            print("NONCE: ",NONCE, guessHash)
    print("Solved ", "NONCE: ", NONCE, "guessHash: ", guessHash)
    return guessHash
```

* 앞자리 0이 4개일 경우,    94280회 반복 Solved  NONCE:  94280 guessHash:  000043ce4a61d02...7a13f
* 앞자리 0이 5개일 경우,   315753회 반복 Solved  NONCE:  315753 guessHash:  000007f9f69a43f...df4d1
* 앞자리 0이 6개일 경우, 45576417회 반복 Solved  NONCE:  45576417 guessHash:  0000003d02b9560...9994e

즉 Target Hash가 적어질수록 더 어려워지는 것을 알 수가 있다.

```python
>>> mining(4)

    NONCE:  10000 e0458f4777612b5565be6df81b53f76943dac3f77939e886e0dc5a2fb5ad1855
    NONCE:  20000 051a66b1b1f972abb6b1ce04574551d5fea86751e18b1588d556627f11c424da
    NONCE:  30000 931475a41686265ee9f65ac07341ba85c021ca8c426990981bb582d2fbfb2299
    NONCE:  40000 4310482b52d76dea37f4ad6910bfe2bc06b4a8b63ac005b9c784f868d0fe84f2
    NONCE:  50000 8b810c0fe332eeb08db6c982170ce905567fe4a6b1a973b5b3f8db37cce0e415
    NONCE:  60000 7b8d4b1341494a017bd61a42a13ab115b95977705e08e5879d8c4934045e62ca
    NONCE:  70000 93033e94e4570eee1e720b475fec2756eaa564c0fc5beac5638a615ab46ede8c
    NONCE:  80000 2026dca4bf2d9389adc13e50d73e4b2d0f01d41b8352530b37fbf5b5080300f8
    NONCE:  90000 bc9c3c347d9fcff8723440a80815c1100fc3455554ed1dc21b78b06c226c5d64
    Solved  NONCE:  94280 guessHash:  000043ce4a61d02bff0e68ba18a7daf448cb3b93691fdd4850f6cd3f85b7a13f
```

```python
>>> mining(5)
    NONCE:  100000 218b6da50df07b86cb603e3ca2e1928381839996c426298dd89b2c98828eda19
    NONCE:  200000 0ec28e8a713b0d08e115b1f3798d00301b90638e5c524209ee583fd013155140
    NONCE:  300000 55e986ef1a8fae35e8a819561be3ae18491551904afdcde73770b083dc31d4bf
    Solved  NONCE:  315753 guessHash:  000007f9f69a43f1bb6ab92672d873b93d6bafaa2007e44b6151bd19efadf4d1
```


```python
>>> mining(6)
    NONCE:  1000000 83c442aba508ff40f3abcb5395954b2507a514840a07ca6069b126de59aa057c
    NONCE:  2000000 3c5bcb77f2e7e75db377dff4f1d9ab9fffee49f5d1bbb45eb0ebfb9e762d7e0a
    NONCE:  3000000 cbdb66f325cb7559caf902b1461758bfe42528caf424f36f8fa0a0b88ef81888
    NONCE:  4000000 51d35452d169ab30824536d7f77e1c09e1249025272918cea0a9443e2badfed1
    NONCE:  5000000 72058932c8af784e9dc6c0b3c42e92e620e1c81fed8a95f151c7c306ab0753e9
    NONCE:  6000000 44d0314ad69daecfa2cb40e9c3bf64f62ed9de04f3d8ef354e8bcc2d0310c9f2
    NONCE:  7000000 417a07a8f561beca5e25e7985c3683d1173c45298103f0e4a02cb513f59c2586
    NONCE:  8000000 5122d9096c30f83a256f3b66b34edd48468b0f131cddcf0109f16ebf4b612a0d
    NONCE:  9000000 3b79b0b61cbe3cc10b7183a8e19aea023334c7769593621c03d285a6789ded31
    NONCE:  10000000 660b9e057377381579f5c54347901cf462fce656c069a4d0f26bdd4cf1e05e66
    NONCE:  11000000 eee61f78c7cf9b6726dea1bd29f773546abb5499e6b2425ba3264c9999cefbd6
    NONCE:  12000000 c17acc15c35e236b88e257be9e7e152035606ad533d3016473d46e7094ca1f0a
    NONCE:  13000000 d6f097e134a53229cbf4b9ff64f12023a9617136e1caef2058e4e11dada7573a
    NONCE:  14000000 956bfe89432dda6da6899d5c80e697f1c1609b698aeec36cf8d4b4c1f90da57d
    NONCE:  15000000 45a5a393b541e5b06fce29fd772e62b7f921d3db7b81595e883c027085b38cc3
    NONCE:  16000000 ed1ccfd6f34fb0118c55855b01ccc03a141ed6fe6a22e4b34b2e73cbc590d67b
    NONCE:  17000000 fa0f1635522d6227e710d63b63306556ccda556353235554a8b887e88dfabbb5
    NONCE:  18000000 27e763d112bdcc66c1536ecb528a9f945b46c82f406bc08c105139af90f6e301
    NONCE:  19000000 0b63fec4806dcb0d8e363246eeff98925e16c21af8d20abd57881255ac022f23
    NONCE:  20000000 c31d3ddd7bb92312bcc2a88263d92e94c6d17a641ac9e59ac262c775c7f58925
    NONCE:  21000000 20b85245db2f4965a99f4dc9e05548c0abe9de09a44b8ecd6e2aaf278ba8a649
    NONCE:  22000000 352a7bd968096898636b3663f43396f43a3d7968452be442b701e111ce3d73c6
    NONCE:  23000000 ec4fa346631f5ae520468a2f3db8241c4ca2826b3b7554d88962888537c7201e
    NONCE:  24000000 af0792b33165137bba106fecb38ea71e3dba71ecd01012c5dfac957ba44c5ba4
    NONCE:  25000000 771e868e2ae94f67ed5f25c5d85b2fb13165774fdd86de29617ceaddb1151822
    NONCE:  26000000 fd0ebd0fa2a20c9d9c2f0a6b342c691fd8d823b1097c776d676be3983a7044e5
    NONCE:  27000000 97e918dd3fd23f91310f9d651dcc18f535bbb2d6ebf86a7130b51cca0d963e74
    NONCE:  28000000 ada06d09d5a899ed1f0465e11d07d77629a92a2c4d29c6f6752482593b44afeb
    NONCE:  29000000 a3be1ffa6d5afcb482d8e1d80b970f227fd17fe1b53ab9cad6af1d2230ff33ec
    NONCE:  30000000 236077716f4ce428ec83f12bf74fb7ab76450bad60bc3f21d924e200c39a9fa4
    NONCE:  31000000 569f95950b371cc480725c6f8d9740930a95a74349f01471659d652811be12cb
    NONCE:  32000000 a8bdf0fb0bbca3fc65489673e63f91d0934347148dc44683f1b02f0ab26d771a
    NONCE:  33000000 55450a311f273be04af5fe9a7411b9ee51d2a6c52273a99a520b6e7e37bce16f
    NONCE:  34000000 dd801fb3bd328f2939e951cbdd19a5b67057e19d71baffdef223d88122c2ca4a
    NONCE:  35000000 86d866f730dc0de41ecb7bdb737c87e1713d74dc1f3319881090077a4715d1ee
    NONCE:  36000000 4a78696996c02f3ec3794135962eeed434349d0ac6e1b9d3f3209b4804716ae9
    NONCE:  37000000 db672dbd69218223494aaff580c3061c55e65f3c81af2c83e96cac09575eee14
    NONCE:  38000000 ee77e94580954ef8319d549021d0987f66f19c6bd7f996e8ec49f4c874f521c8
    NONCE:  39000000 09a68e6cd624dc2a4408c1efe8a5b2431c37e7a0646abf50a4459d57397f05e3
    NONCE:  40000000 5b07929bd7088f11d6caf0e65a6f44ab3b7f82d32900430cc7c7244b2917e04d
    NONCE:  41000000 ac5b7f8f6471df1a837f043b0327e5f6df63c01636b186efe9b968ae2fcc8015
    NONCE:  42000000 fc156626139e89a0394edbae833cd0f7d9a48bb7df10a6903cb2fbafff271aa4
    NONCE:  43000000 7791f1babe8225ba48ba21bc75b8d4ea8877761d0f8eea5788e6de9d400889e9
    NONCE:  44000000 2a12a9ff792879e7ab58a3e6fd4cb1cb9b1d4b27d736ae44bac4a76213c98e0e
    NONCE:  45000000 975dedd12e4980779c7eb3060b723dae84fbac1c362976b362ab019f65cf1fe2
    Solved  NONCE:  45576417 guessHash:  0000003d02b95604bb1ec436ff20e08168dd339f2ec0f9941bfc58bad039994e
```

마이닝이 안되는 경우가 있을 수 있다.
* 발생한 트랜잭션이 처리되지 않고 여러 건이 적체되어 있는 경우,
* 처리비용이 낮은 경우,
* 또는 트랜잭션 nonce 앞 번호가 처리 되지 않았는데 뒤 번호가 대기하고 있는 경우, 뒤 트랜잭션은 대기하게 된다.

마이닝이 적체되어 안 지워지는 경우에는 바람직하지 않지만, 강제로 정리해야만 하는 순간이 있다. 이럴 경우에는:
- geth를 종료하고,
- datadir 아래 geth디렉토리 밑의 transactions.rlp를 삭제하고,
- geth를 다시 시작한다.

따라서 마이닝을 할 경우, ```txpool.inspect``` 명령어를 실행하여 적체되어 있는 거래가 있는지 확인하는 것이 바람직하다.

```python
> txpool.inspect
{
  pending: {},처리할 수 있는 대기 거래 건수 
  queued: {} nonce가 순서에 맞지않아 처리할 수 없는 거래 건수
}
```

# 7. gas

거래(transaction)을 처리하기 위해서는 비용이 발생한다. 노드들의 컴퓨팅 자원이 투입되기 때문에 이에 대한 비용이다.

비용은 송금 금액에 비례하는 것이 아니라, 데이터 크기와 소요되는 컴퓨팅 자원량에 따라 산정이 된다.
비용은 소모되는 **gas**에 **gasPrice** 단가를 곱하여 계산한다.

```python
거래 비용 = gasPrice * gas
```

주유소에 비유해서 이해할 수 있다. 주행거리에 따라 필요한 연료량과 단가에 따라 주유를 하는 방식이다. 즉 **gas는 주행에 필요한 연료량**에 해당한다. 예를 들어 Kecchak hashing을 하는데 30 단위의 gas가 필요, gasPrice는 일정 분량의 바이트를 실행하는데 몇 gas가 필요한지 나타내는 단위를 의미한다.~~말이 중간에 짤렸는데 어떻게 수정할지 모르겠네요~~

이러한 거래에 비용이 발생하게 되면, DDos 공격 역시 상당한 비용이 수반되므로 쉽게 시도하지 못하게 되는 효과가 있다.

블록체인에 부담이 가는 실행 (예: 무한반복) 역시 gas비용때문에 가능하지 못하게 된다. 일정 gas가 사용되면 gas limit을 초과하게 되므로 정지하게 된다.

## 7.1 gas 가격

gasPrice는 단가로서 사용자가 정할 수 있다. 주유와 비유하면 gasPrice는 시가, 즉 몇 ether가 필요한지에 해당한다.

보통 명시하지 않으면 최선의 가격으로 정해진다. 따라서 gasPrice를 정하지 않고, 거래를 전송하기도 한다.

그러나 gasPrice에 따라 처리속도가 결정된다는 점을 주의한다. gasPrice가 너무 적으면 마이닝되지 않을 수 있으며, 반대로 많으면 빠르게 마이닝 될 수 있다. gasPrice가 높을수록 더 많은 마이너가 처리하려고 들 것이고 거래가 마이닝되는 시간이 짧아지게 된다.

평균적으로 지불해야 하는 가격을 알려주는 사이트가 있다. 메인네트워크에서 gasPrice 단가는 시간에 따라 정해져 있지 않고 변동된다 etherscan.io에서는 지난 수 년간의 gasPrice를 그래프로 제공하고 있는데 (https://etherscan.io/chart/gasprice), 그 가격이 약간씩 감소하고 있지만 이는 이더리움의 실물가치가 상대적으로 증가하면서 생기는 현상일 수 있다.

또한 가격에 따라 채굴속도에 영향을 미치게 되는데, 예를 들어, 2분 이내로 빠르게 처리되려면 ('Fast'), 더 많은 가격을 지불해야 하겠다 (https://ethgasstation.info/)

gas price는 ```web3.eth.getGasPrice()``` 함수를 사용하여 구할 수 있고, 개인망에서 그 값은 1 gwei를 출력하고 있다. 보통 테스트네트워크는 마이닝의 우선순위에 영향을 미치지 않기 때문에, 고정 gas price를 가지고 있다.

~~다음 코드가 다 실행 안됩니다. web3.eth.getGasPrice().then(console.log)를 실행시키면 \"TypeError: Cannot read property 'then' of undefined or null\"이라고 나오고, web3.eth.getGasPrice()만 실행시켜보면 undefined라고 나오네요. web3나 web3.eth는 괜찮은 것 같고요~~

gasPrice()를 계산하기 위해서는
```python
> web3.eth.getGasPrice().then(console.log);
> 1000000000
```

gasPrice변수에 getGasPrice()의 결과를 입력하기 위해서는:
```python
> web3.eth.getGasPrice().then(function(gp) {console.log(gp); gasPrice=gp;});
> 1000000000
```

```python
> web3.eth.getGasPrice().
... then(function(gp) {
...     console.log("Average gas price: " + gp);
...     gasPrice = gp;
... }).catch(console.error);
> Average gas price: 1000000000
```

## 7.2 gas 한도

gas limit은 거래의 처리에 필요한 사용 한도를 말한다. 사용자가 최대로 지불할 용의가 있는 한도를 말한다.

gas limit은 거래의 성격에 따라 다르게 책정된다. 최소 21,000에서, 단순한 거래의 경우 보통 30,000 정도면 충분하다.
복잡한 거래일수록 필요한 gas가 늘어난다.

gas가 소진될 때 까지 마이닝하고, 남으면 반환, 모자라면 **Out of Gas Exception**이 발생한다.

또한 gas limit은 블록에 대한 한도를 의미하며, 이는 블록에 포함될 거래의 갯수를 결정하기 위해 사용된다. 예를 들어, 거래가 6개 있고 gas비가 각 25, 30, 35, 40, 45, 50이라고 하자. 블록의 거래한도는 100이라고 하자. 그러면 거래의 gas가 25 + 30 + 35를 포함할 수도, 40+45가 포함될 수도 있다.

지금 현재 9,994,671 gas이고 (https://ethstats.net/) 이를 21000(거래 gas limit)으로 나누면 475개의 거래가 한 블록에 포함되는 수준이다.

이 속도로 평균 14초 내외로 1블록이 생성되고 있다 (https://ethstats.net/). 블록에 대한 gas limit은 처음에 ```genesis.json```에 설정되어 있고, 블록마다 이 값은 마이너들에 의해 1/1024 만큼씩 조정될 수 있다. 블록을 생성하면서 그만큼의 한도로 조정은 할 수 있으나, 큰 차이로 변경하는 것은 허용되지 않는다. ```eth.getBlock('latest').gasLimit```은 현재의 gasLimit을 알 수 있다.


```python
> eth.getBlock('latest').gasLimit
8000000
```

## 7.3 gas 계산

스마트계약(컨트랙)을 배포하거나 그 함수를 호출하는 경우에는 gas 비용이 발생한다. 배포하면 거래비용 transaction costs이 발생한다. 함수를 호출하는 경우 opcode를 하나 하나 실행하는 비용 execution costs이 발생한다.

### 실행비용

**실행비용(execution costs)**는 사용되는 명령어 opcode를 기준으로 산정이 된다.

코드 | 명령어 | 비용 | 설명
-----|-----|-----|-----
0x00 | stop | | 실행중지
0 01 | ADD | 3 | 더하기
10 | LT | 3 | less than 비교
20 | SHA3 | 30 | Keccak-256 hash 계산
30 | ADDRESS |  | 현재 실행 계정 주소 읽기
40 | BLOCKHASH | 20 | 블록 해시


> Opcodes

> Solidity는 사람이 읽을 수 있는 수준의 코드이다. 반면 기계어는 자신이 사용하는 명령코드로 실행되는데, 이를 **Op**eration **Code**를 말한다 (참조 https://ethervm.io/). 예를 들어, 스택에 ```2 3 ADD```라고 저장이 되어 있고 (push), 하나씩 꺼내 (pull) ```2 + 3``` 연산을 한다.
> * ```2 3 OP_ADD 5 OP_EQUAL``` $2+3==5$
> * ```2 3 OP_ADD 2 OP_MUL 1 OP_ADD 11 OP_EQUAL``` $((2+3) \times 2)+1==1$

> 비트코인도 이러한 스크립트 언어를 사용하는데, 서명과 관련된 다음 스크립트도 Opcode로 실행된다.
> * scriptPubKey는 공개키로 잠그는 opcode 명령문은 ```DUP HASH160 <PubHash> EQAULVERIFY CHECKSIG```
> * scriptSig: 는 반대로 공개키로 푸는 명령문은 ```<sig> <PubK>```
이들 코드 scriptSig, scriptPubKey를 합쳐서 실행하면 ```P2PKH Pay-to-Public-Key-> Hash```:
> * <서명> <공개키> OP_DUP OP_HASH160 <공개키Hash> OP_EQUALVERIFY OP_CHECKSIG

### 거래비용

**거래 비용 transaction costs**는 아래 빨간 색으로 표시된 부분에 해당한다.
* 보통 거래의 처리에는 21,000 ($G_{transaction}$), 컨트랙 배포에는 32,000 gas ($G_{txcreate}$)가 소요된다.
* byte수에 따라 산정된다. Yellow Paper Appendix G를 참조해서 바이트수에 따른 비용을 알 수 있다. 데이터 값이 없는 $G_{txdatazero}$는 4, 데이터 값이 있는 $G_{txdatanonzero}$는 68로 계산된다.


![alt text](figures/5_gasCost.png "gas cost in yellow paper")

아래와 같은 트랜잭션 데이터가 있다고 하자.


```python
txdata = "606060405260405160808061067283398101604090815281516020830151918301516060909301519092905b42811161003457fe5b60008054600181016100468382610100565b916000526020600020900160005b8154600160a060020a038089166101009390930a928302920219161790555060008054600181016100858382610100565b916000526020600020900160005b8154600160a060020a038088166101009390930a928302920219161790555060008054600181016100c48382610100565b916000526020600020900160005b8154600160a060020a038087166101009390930a928302920219161790555060028190555b5050505061014b565b8154818355818115116101245760008381526020902061012491810190830161012a565b5b505050565b61014891905b808211156101445760008155600101610130565b5090565b90565b6105188061015a6000396000f300606060405236156100a15763ffffffff7c0100000000000000000000000000000000000000000000000000000000600035041663120b205381146100a35780634ecb35c4146100c557806366d003ac146100f557806367e404ce146101215780636cc6cde11461014d5780637022b58e1461017957806393c0e83a1461018b578063a79a3cee146101ba578063ac4c25b2146101de578063c7fe23a3146101f0575bfe5b34156100ab57fe5b6100b3610220565b60408051918252519081900360200190f35b34156100cd57fe5b6100e1600160a060020a0360043516610226565b604080519115158252519081900360200190f35b34156100fd57fe5b61010561023b565b60408051600160a060020a039092168252519081900360200190f35b341561012957fe5b610105610273565b60408051600160a060020a039092168252519081900360200190f35b341561015557fe5b6101056102ab565b60408051600160a060020a039092168252519081900360200190f35b341561018157fe5b6101896102e3565b005b341561019357fe5b610105600435610364565b60408051600160a060020a039092168252519081900360200190f35b34156101c257fe5b6100e1610396565b604080519115158252519081900360200190f35b34156101e657fe5b610189610410565b005b34156101f857fe5b6100e1600160a060020a0360043516610479565b604080519115158252519081900360200190f35b60025481565b60016020526000908152604090205460ff1681565b60006000600181548110151561024d57fe5b906000526020600020900160005b9054906101000a9004600160a060020a031690505b90565b60006000600081548110151561024d57fe5b906000526020600020900160005b9054906101000a9004600160a060020a031690505b90565b60006000600281548110151561024d57fe5b906000526020600020900160005b9054906101000a9004600160a060020a031690505b90565b6102ec33610479565b15156102f85760006000fd5b600160a060020a0333166000908152600160208190526040909120805460ff19169091179055610326610396565b1561035f5761033361023b565b604051600160a060020a039182169130163190600081818185876187965a03f192505050151561035f57fe5b5b5b5b565b600080548290811061037257fe5b906000526020600020900160005b915054906101000a9004600160a060020a031681565b600080805b60005481101561040357600160006000838154811015156103b857fe5b906000526020600020900160005b9054600160a060020a036101009290920a900416815260208101919091526040016000205460ff16156103fa576001909101905b5b60010161039b565b600282101592505b505090565b33600160a060020a0316610422610273565b600160a060020a0316146104365760006000fd5b600254421161044157fe5b610333610273565b604051600160a060020a039182169130163190600081818185876187965a03f192505050151561035f57fe5b5b5b565b6000805b6000548110156104e15782600160a060020a03166000828154811015156104a057fe5b906000526020600020900160005b9054906101000a9004600160a060020a0316600160a060020a031614156104d857600191506104e6565b5b60010161047d565b600091505b509190505600a165627a7a72305820b02ae8d25b238d57e8363bfc5f30351753ca38bde86d49729841a827fb4e9d660029000000000000000000000000a4be607be0a6cceff581ef35c0cdc47a7532ff540000000000000000000000007c6b393dbb1152691b02a61378bc731662cdd9f1000000000000000000000000748ed23daa18226d872b5bd4d48ff2594fd6901c00000000000000000000000000000000000000000000000000000000597fc480"
```

2자리씩, 즉 1바이트씩 읽어서 0인지 아닌지 그 갯수를 산정한다. count_zero_bytes() 는 데이터가 없는, 즉 0인 바이트의 수를 세는 함수이다.

```python
def count_zero_bytes(data):
  count = 0
  for i in range(0, len(data), 2):
    byte = data[i:i+2]
    if byte == "00":
      count += 1
  return count
```

다음은 데이터가 있는, 0이 아닌 바이트 개수를 세는 함수이다.

```python
def count_non_zero_bytes(data):
  return (len(data) // 2) - count_zero_bytes(data)
```

그 결과를 출력하면 0은 184바이트, 아닌 경우는 1594 바이트가 된다. 따라서 거래비용은 $1594 \times 68 + 184 \times 4 = 109128$로 계산한다.


```python
>>> print("zero-bytes: {0}".format(count_zero_bytes(txdata)))
>>> print("non-zero-bytes: {0}".format(count_non_zero_bytes(txdata)))
zero-bytes: 184
non-zero-bytes: 1594.0
```

### gas 산정 함수

거래를 실행하기 위해, 꼭 gas를 산정해야 하는 것은 아니다. 개스비용을 적지 않으면, ```web3.eth.estimateGas()``` 함수로 산정한 값으로 적용하게 된다. ~~byteCode가 defined되어 있지 않다고 오류 납니다.~~ ~~추가로 estimateGas()에 인자로 전달되는 부분에서 무엇을 하는 건지 설명이 안되어 있습니다. {data: byteCode}가 어떤 의미로 사용되는건지요?~~

```python
var gas = web3.eth.estimateGas({
    data: byteCode
});
console.log(gas);
```

### 사용된 gas 알아보기

트랜잭션 처리 비용은 ```eth.getTransactionReceipt(hash)```로 알 수 있다. 아래에서 보듯이 gasUsed는 21000이고, 이는 보통의 거래에 소요되는 gas이다. 참고로 eth.getTransactionReceipt(hash)에 전달되는 인자값은 앞에서 확인했던 블록(eth.getBlock(55169)를 실행시킨 코드)에서 transaction값으로 기록되었던 것이다. 


```python
> eth.getTransactionReceipt("0xd87121b8b0f84f7fa038cd7c1928ca6a222d14228125c90edc2493fdef4fb90b")
{
  blockHash: "0xd2c51ae5dea10e50c915e9d7ccc6c117c2d14d0f38da936b62a1c38fd0494d26",
  blockNumber: 55169,
  contractAddress: null,
  cumulativeGasUsed: 21000,
  from: "0x2e49e21e708b7d83746ec676a4afda47f1a0d693",
  gasUsed: 21000,
  logs: [],
  logsBloom: "0x00000...생략...00000",
  root: "0x1817a1775db945025d4a67a0bfcb633b5fed6a1fa76804d152db410c9140237d",
  to: "0xe36104ad419c719e356e86f94b5a7ca47a83f9e7",
  transactionHash: "0xd87121b8b0f84f7fa038cd7c1928ca6a222d14228125c90edc2493fdef4fb90b",
  transactionIndex: 0
}
```

### 실제

실제 hash를 가지고 gas 상세를 살펴보자 (참조: https://etherscan.io/tx/0xcb1e3530950cf2c43a307bcb5645ae71a12c76a60831617badd04aea3efe68aa)

* Transaction Fee: 0.000284248 Ether ($0.05) = gas used x gas price = 35531 x 8 (아래를 참조)
* Gas Limit: 136,500
* Gas Used by Transaction: 35,531 (26.03%)
* Gas Price: 0.000000008 Ether (8 Gwei) 이 수준은 'Fast' 기준

![alt text](figures/5_transactionFee.png "transaction fee in a real case")

# 8. 거래 건수

주소에서 전송된 거래건수는 ```getTransactionCount``` 명령어를 통해서 알 수 있다. Nonce는 0부터 계산되므로, 거래건수가 nonce보다 1만큼 크게 된다.

옵션을 주어 ```getTransactionCount(address, 'pending')``` 이런 식으로 거래건수를 구할 수도 있다.
* "latest" - 최근 블록을 의미
* "pending" - 현재 채굴된 블록 (Transaction Pool 에 남아있는 대기중 상태)


현재 거래건수를 계산해보면 11이다. ~~0x8078e6bc8e02e5853d3191f9b921c5aea8d7f631는 갑자기 어디서 나타난 건가요?~~


```python
> eth.getTransactionCount('0x8078e6bc8e02e5853d3191f9b921c5aea8d7f631')
11
```

최근 블록을 살펴보자. 'transactions' 필드를 보면, 포함된 hash가 있다. 하나를 구해서 nonce 값을 확인해 보자.


```python
> eth.getBlock('latest')
    {
      difficulty: 0,
      extraData: "0x",
      gasLimit: 6721975,
      gasUsed: 58533,
      hash: "0x5a247e574a2ba56a74eb5f5d5201365563dad61eb141e333b59dfa393b50476b",
      logsBloom: "0x00000000000000000000000000000000000000000000040000000000000000000200000000000000000000080000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000",
      miner: "0x0000000000000000000000000000000000000000",
      mixHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
      nonce: "0x0000000000000000",
      number: 11,
      parentHash: "0x36af17737106912bc85078ad63133143802d824ad36207912d267960028695d7",
      receiptsRoot: "0xee5bc87f490e552952ea8f769e6cb2d0a84509e3a57c6975cd3c67c022cd35d9",
      sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
      size: 1000,
      stateRoot: "0xb4549f2c55629a06a09cad796a95c38db14aa58aa76cb33e34f3966a45e3e66d",
      timestamp: 1646129233,
      totalDifficulty: 0,
      transactions: ["0x2a33ec94c133f92d6aea65db36805dac8131d19ae32a875006e281e128938f53"],
      transactionsRoot: "0x3373d83bc8440c98a4fd82681546eed7240693b20fa2ae9f16083a1ea066b30c",
      uncles: []
    }
```

최근 블록에 포함된 거래의 명세를 출력해 보면, nonce 값을 구할 수 있다. 어떤가? transactonCount와 비교해서 1이 작다. getTransaction()에 전달되는 값은 eth.getBlock('latest')를 실행시켜 얻은 결과에서 transactions에 있는 값이다. 


```python
> eth.getTransaction('0x2a33ec94c133f92d6aea65db36805dac8131d19ae32a875006e281e128938f53')
    {
      blockHash: "0x5a247e574a2ba56a74eb5f5d5201365563dad61eb141e333b59dfa393b50476b",
      blockNumber: 11,
      from: "0x8078e6bc8e02e5853d3191f9b921c5aea8d7f631",
      gas: 800000,
      gasPrice: 20000000000,
      hash: "0x2a33ec94c133f92d6aea65db36805dac8131d19ae32a875006e281e128938f53",
      input: "0x5f74bbde000000000000000000000000cc16c60a1fb054d2594ad06c6f8323afad7290650000000000000000000000000000000000000000000000000000000000000000",
      nonce: 10,
      r: "0x66e212fdd9500d61058b0c7f2dd19592ae2bf84afef7e332e2e1175980393494",
      s: "0x33f444713f2e94e332a4b35864dc167fd7fcdde62f3672e431dacb42395caaaa",
      to: "0x9250411625df0124335018c83521b91bde59ed5f",
      transactionIndex: 0,
      v: "0x25",
      value: 0
    }
```

## 문제 5-1: 다른 계정으로 송금 거래

송금거래를 하면, 계정A에서 계정B로 자금의 이체가 발생한다. 은행으로 말하면, 동일 은행이거나 타 은행으로 송금할 수 있다.

블록체인에서도 동일한 기계 또는 다른 기계로의 송금이 가능할 것이다.

물론 송금이 가능하려면, 고정IP의 노드와 geth를 설치해서 계정을 발급받아 놓아야 가능하다는 점에 유의하자.

### 1-1 같은 기계, 다른 계정으로 송금하기

타계정으로 송금을 해보자. 송금하기 위해서는 비용이 수반된다. 송금 거래를 네트워크에 전송하기 위해서는 gas가 필요하고, 계정에 이를 충당할 잔고가 있어야 한다.

gas가 모자라면 거래가 처리되지 않거나, 지연될 수 있다. 잔고를 충전하기 위해서는 Ether를 구매하거나, 마이닝을 해서 충전을 해두어야 한다.

우선:
* 지급계정을 선택하고, 잔고를 확인한다. 송금액과 gas비를 충당할 금액이 있어야 한다.
* 현재 blockNumber를 확인하고, 거래가 완성된 후 blockNumber와 잔고의 변화가 있는지 살펴보자.
* 개인망이라서 자신이 거래를 발생하여 스스로 마이닝을 하자.
* eth와 web3를 섞어서 사용하는 것은 피하도록 한다.


```python
> eth.getBalance(eth.accounts[1])
184999843394000029444
> eth.sendTransaction({from:eth.accounts[1], to:eth.accounts[0],value:10000})
Error: authentication needed: password or unlock
        at web3.js:3143:20
        at web3.js:6347:15
        at web3.js:5081:36
        at <anonymous>:1:1
```

물론 geth client에서 한 줄씩 실행해도 좋지만, 여기서는 프로그램을 만들어 일괄실행해보자.

그러기 위해서는, 즉 거래에 필요한 gas또는 송금액이 지급되려면, 계정이 해제되어 있어야 한다. geth를 시작할 때 --unlock 하거나, console에서 직접 할 수 있다. 다음과 같이 프로그램을 실행하기 전에 다음과 같이 unlock을 하자.
```python
> personal.unlockAccount(eth.accounts[0]) 0번째 계정을 해제한다.
```

```eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1],value:10000})```는 coinbase에서 2번째 계정으로 송금하는 명령어이다. ```value```에 송금액을 기입하였다. ```eth.gasPrice()``` 지난 몇 건의 블록의 gasPrice의 중위값을 알려준다.

실행하고 나면, 거래가 발생하고, 마이닝을 하고, 잔고와 bloack number가 변화하게 된다. coinbase에는 마이닝 보상까지 더해지므로, 잔고가 송금액만큼 줄어들지는 않을 것이다.


```python
[프로그램: src/e_testTran.js]
miner.setEtherbase(eth.accounts[0]);
console.log('coinbase: ', eth.coinbase);
var bal1=eth.getBalance(eth.coinbase);
var bal2=eth.getBalance(eth.accounts[1]);
console.log('sender balance in ether: ', web3.fromWei(bal1,"ether"));
console.log('receiver balance in ether: ', web3.fromWei(bal2,"ether"));
console.log('median gas price: ', eth.gasPrice);
console.log('block number: ', eth.blockNumber);
console.log('transaction count: ', eth.getTransactionCount(eth.coinbase));
eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1],value:10000});
console.log('...mining start');
miner.start(1);admin.sleepBlocks(1);miner.stop();
console.log('mining done...');
var bal1new=eth.getBalance(eth.coinbase);
var bal2new=eth.getBalance(eth.accounts[1]);
console.log('- new sender balance in ether: ', web3.fromWei(bal1new,"ether"));
console.log('- new receiver balance in ether: ', web3.fromWei(bal2new,"ether"));
console.log('- block number: ', eth.blockNumber);
console.log('- transaction count: ', eth.getTransactionCount(eth.coinbase));
```

파일을 저장하고 나서, 일괄실행을 해보자.

```python
geth --exec "loadScript('src/e_testTran.js')" attach http://localhost:8445

    coinbase:  0x21c704354d07f804bab01894e8b4eb4e0eba7451
    sender balance in ether:  60.000021786999979076
    receiver balance in ether:  184.999843394000019444
    median gas price:  1000000000
    block number:  47206
    tranaction count:  264
    ...mining start
    mining done...
    - new sender balance in ether:  65.000021786999969076
    - new receiver balance in ether:  184.999843394000029444
    - block number:  47207
    - tranaction count:  265
true
```
![alt text](figures/2_sendTransactionToAnotherAccount.png "send transaction to another account")

![alt text](figures/2_transactionReceipt.png "transaction recepit after send transaction")

### 1-2 다른 기계, 다른 계정으로 송금하기

은행이라고 생각하면, 타행이체라고 간주할 수 있는 거래이다.

블록체인으로 보면, 노드A에서 노드B ("0x519775cc61e4c9b3f19b75426a7a3696a3c85035") 으로 송금하는 사례이다. 노드A와 노드B는 동일한 사설망의 멀티노드들이다. 


노드A에서 노드B의 계정을 안다면, 그 잔고를 다음과 같이 알 수 있다.
```python
> eth.getBalance("0x519775cc61e4c9b3f19b75426a7a3696a3c85035") 
10000
```

```python
[프로그램: src/e_testTran2.js]
var hqaccount="0x519775cc61e4c9b3f19b75426a7a3696a3c85035";
console.log('hq account balance: ', eth.getBalance(hqaccount));
console.log('block number: ', eth.blockNumber);
var t=eth.sendTransaction({from:eth.accounts[0], to:hqaccount, value:10000});
console.log('transactionHash: ',t);
console.log('...mining start');
miner.start(1);admin.sleepBlocks(1);miner.stop();
console.log('mining done...');
console.log('block number: ', eth.blockNumber);
console.log('hq account balance: ', eth.getBalance(hqaccount));
```

노드A에서 프로그램을 일괄실행해보자.
```python
geth --exec "loadScript('src/e_testTran2.js')" attach http://localhost:8445

    hq account balance:  20000
    block number:  70296
    transactionHash:  0x15b17f04f20322cc8ee14aef9a9b617ec77935f29ce313a78ad1388241607e6c
    ...mining start
    mining done...
    block number:  70297
    hq account balance:  30000
true
```


## 연습문제 5-1: 간단하게 설명하기

* 1. 거래는 2종류가 있다고 하는데, 이를 설명하세요.
(1) 블록체인에 기록되는 거래
(2) 블록체인에 기록되지 않는 거래
(1)번이 당연히 중요하고 이것을 sendTransaction
(2)번은 로컬에서 발생하고 블록체인에는 기록되지 않아요. call
(1)번은 수수료 gas가 발생하고 사인이 필요하지만, (2)는 그렇지 않아요.

* 2. ABI는 왜 필요한지? 어떤 함수를 사용하면 알아서 해주는가?
ABI는 함수호출을 머신코드 수준으로 표현한 것이고, API와 같은 역할을 한다.
abi.encodeWithSignature() 함수에 호출하려는 함수와 인자를 적어주면 된다.

* 3. ABI에서 함수명, 인자는 어떻게 부호화하는가?
함수시그니처를 sha3 해싱을 하고 앞 4바이트를 선택.
매개변수는 16진수 32바이트로 만든다.

* 4. 이더리움 트랜잭션의 필수 항목은?
from에 20바이트 주소를 반드시 적어야 한다.

* 5. 이더리움 트랜잭션의 항목이 아닌 것은?
- from
- to
- value
- gas
- gasPrce
- data
- nonce
- balance
from을 제외한 나머지 항목은 선택항목이라 적지 않아도 된다.
balance는 적지 않는다.

* 6. 이더리움과 비트코인의 거래를 구성하는 항목에 차이가 있는가?
비트코인에서는 송금하고 나서 잔액을 환급하는 특징을 가지고 있다.
따라서 하나 이상의 in에서 입금액을 채우고, 하나 이상의 out에 그 잔액을 환급하게 된다.
따라서 미사용잔액이 적힌다.
반면에 이더리움에는 잔액을 적지 않는다.
이더리움에서는 미사용잔액을 수합해서 사용하는 방식이 아니라, 입금주소, 출금주소와 금액으로 단순해지게 된다.

* 7. 블록에 포함되지 않는 거래는 버려지는지 설명하세요.
블록에 포함되지 않는 거래는 ommers 목록에 포함되고, 기다려서 다음 블록에 추가될 수 있다.

* 8. 블록은 어떻게 서로 연결되는지 설명하세요.
현재 블록은 **전 블록의 해시**를 통해 연결된다.

* 9. 블록헤더에 포함되지 않는 해시는?
- 거래트리의 해시
- 거래수령트리의 해시
- 저장트리의 해시
- 상태트리의 해시
3번 저장트리의 해시 storageRoot는 상태트리에 저장되고 블록헤더에는 포함되지 않는다.

* 10. 블록헤더에 포함되는 gas 관련 항목은 어떤 것들이 있는지?
gas limit, gas used가 있다.

* 11. 최근 블록의 명세를 출력에서, 어떤 항목들이 실제 포함되었는지 확인하고 그 출력을 설명하세요.
eth.getBlock('latest')로 출력하면 그 항목을 출력할 수 있다.

* 12. 머클증명이란 무엇인지 설명하세요. 어떤 거래도 수정될 수 없다는 의미는 무엇인지?
거래를 해싱해서 맨 위 루트의 해시값이 됩니다.
머클루트는 가지들을 합쳐서 결국 하나의 해시 값을 만드는 것. a와 b를 합쳐서 해시값을 구하고, b와 c를 합쳐서 또 해시값을 구했듯이 이 작업을 반복하는 것이예요. 이런 식으로 ab와 cd를 합쳐서 해시값을 구하면 됩니다.
어떤 거래라도 위변조가 일어나면 머클루트의 해시값이 변동하기 때문에 위변조가 불가능해요.
특정 transaction이 수정되면, 그 해시가 변경되게 되고, 연쇄적으로 전체 해시가 다르게 되기 때문에 위변조가 거의 불가능하게 된다.

* 13. gas 단가는 마이닝 속도에 영향을 미치는지 설명하세요.
gas비 = gas량 x gasPrice 단가로 계산
gasPrice가 너무 적으면 마이닝되지 않을 수 있으며, 반대로 많으면 빠르게 마이닝 될 수 있다. gasPrice가 높을수록 더 많은 마이너가 처리하려고 들 것이고 트랜잭션이 마이닝되는 시간이 짧아지게 된다.
개인망에서는 1 gwei로 정해져 있어요.

* 14. 거래가 6개 있고 gas비가 각 25, 30, 35, 40, 45, 50이라고 하자. 블록의 거래한도는 100이라고 하자. 그렇다면 블록에는 어떤 거래가 포함될 수 있는지 설명하세요
gas limit은 블록에 대한 한도를 의미하며, 이는 블록에 포함될 거래의 갯수를 결정하기 위해 사용된다. gas가 25 + 30 + 35를 포함할 수도, 40+45가 포함될 수도 있다.

* 15. 컨트랙 배포에는 얼마나 gas가 필요한지?
32,000

* 16. 거래건수와 nonce의 관계를 설명하세요.
주소에서 전송된 거래 건수는 getTransactionCount 명령어를 통해서 알 수 있다. Nonce는 0부터 계산되므로, 거래건수가 nonce보다 1만큼 크게 된다.

* 17. gas는 누가 지불하는지 설명하세요.
gas는 계정의 balance에서
거래를 발생하는 자신이 지불해야 해요.

* 18. 블록이 너무 빨리 또는 느리게 만들어지는 것을 막기 위해 무엇을 한다?
난이도 difficulty를 조정해요.
- 10초 이하이면: 난이도 상향
- 10~19초 이면: 난이도 변동 없슴
- 20초 이상이면 난이도 하향

* 19. gas의 실행비용과 거래비용을 설명하세요.
gas 산정은 실행비용 execution costs, 거래비용 transaction costs으로 구분된다.
실행비용은 명령어 기준으로 산정한다.
Opcodes라고 하는 것이 있어요. 더하기, 곱하기, 나누기 이런 연산이 비용이 됨.
그래서 반복문은 비싼 연산
프로그래밍할 때 가급적이면 반복문 무심코 사용하면 안돼요
배열로 데이터 넣어놓고, 반복문으로 검색하는 거 비싸게 됩니다.
거래비용은 (1) 보통거래의 경우 21,000. 컨트랙배포에는 32,000 gas가 소요됩닌다.
거래비용은 바이트 수에 따라 계산
실행비용과 거래비용에 따라 gas가 산정
이런 내용을 모두 기억하거나 외울 필요는 없어요.
보통 대략 넣으면, 차액을 반환해줍니디다.

## 연습문제 5-2: Merkle Root 계산

다음 4건의 데이터에 대해 Merkle Root 값을 계산하고 출력하세요.
중간 노드의 AB, CD의 해시도 출력하세요.
* txA = 'Hello'
* txB = 'How are you?'
* txC = 'This is Thursday'
* txD = 'Happy new Year'

## 연습문제 5-3: 다른 계정으로 송금거래

친구에게 주소를 구하고, 그 주소로 송금해 보자. 송금이 되지 않으면 왜 안되는지 이유를 알아보자.
안되면 친구의 주소를 만들고 전송한다.
잔고의 증가분을 출력하세요.
소요된 gas비용 출력하세요.

### 답

우선 genesis block를 동일하게 설정해 놓은 상대 컴퓨터의 enode를 구해서, Peer로 추가해야 한다 (참조 3장 3-3).
enode에는 IP주소, 포트번호가 포함되어 있어, 상대 컴퓨터를 노드로 추가할 수 있다.

```python
admin.addPeer("enode://719b29cc599ee5964...011155676a3cb@117.16.44.46:30446?discport=0")
```

그리고 송금거래를 발생하면 된다 (참조 5-1) (아래 실행하지 않았슴 (NotYet))


```python
%%writefile src/exercise5_2.js
var hqaccount="0x519775cc61e4c9b3f19b75426a7a3696a3c85035";
console.log('hq account balance: ', eth.getBalance(hqaccount));
var bal=eth.getBalance(eth.coinbase);
console.log('- balance in ether: ', web3.fromWei(bal, "ether"));
var t=eth.sendTransaction({from:eth.coinbase, to:hqaccount, value:10000});
console.log('transactionHash: ',t);
console.log('...mining start');
miner.start(1);admin.sleepBlocks(1);miner.stop();
console.log('mining done...');
console.log('hq account balance: ', eth.getBalance(hqaccount));
```

## 연습문제 5-4: ABI 명세

다음 ```sayHello()```함수의 ABI 명세를 생성한다.

```python
contract Hello {
    function sayHello(bytes toWhom) pure public returns(string memory) {}
}
```

## 연습문제 5-5: gas비 계산

"Let's meet in my office at 10 AM."의 거래비용 gas를 계산하시오.

## 연습문제 5-6: 해시 맞추기 1

**해시는 100미만의 양수**로 정해진다고 하자.
해시 값은 별도로 해싱을 할 필요없이 십진수로 생각하자.
**NONCE는 반복회수**로만 쓰이고 무작위 수를 생성하는데 입력되지는 않는다.
암호화폐에서는 마이닝이라고 하는 작업, 즉 목표 해시를 찾아보자.
**목표해시 값이 100에 얼마나 가까운지에 따라**, 즉 90은 가깝고 10은 먼 값이다.
**몇 회만에 그 값을 찾는지 비교**해 보자.
여기서 난이도를 3회만에 찾게 되면 1이라고 하자.
3회 보다 횟수가 많이 걸리면 이 경우는 어렵다는 의미이므로 난이도를 낮추어야 한다.
반대의 경우 3회보다 적은 횟수가 필요하면 난이도를 높이게 된다.
문제는 난이도를 계산해서 조정하지는 않고, 다순하게 목표 값을 몇 회에 출력하는지만 알아보자.
- 90을 목표해시로 정하고 몇 회만에 마이닝에 성공하는지 출력
- 10을 목표해시로 정하고 몇 회만에 마이닝에 성공하는지 출력

### random 함수
```random()```은 0 ~ 1사이의 무작위 수를 생성한다.
이 함수에 100을 곱하면 0 ~ 100 사이의 수를 생성한다 (100은 제외).
```print```문의 ```end```는 출력을 이어서 하게 만든다.


```python
from random import randint
for i in range(1,20):
    print(int(random.random()*100), end=" ")
```

    83 18 99 61 2 74 4 51 12 63 46 64 67 72 7 70 73 49 86 

### 답

문제의 답에서 난이도는 수정하지 않는다. 그리고 해시를 생성하지도 않는다 (문제가 어렵다고 해서, 쉽게 하려고)
목표해시를 정하는 과정을 이해하면, 불과 강의코드 몇 줄만 수정하면 풀린다.
- targetHash는 90 (또는 10)으로 정했다고 가정
- guessHash는 random으로 1~100 사이의 수를 생성
```int(random.random()*100)```로 0~100 까지의 수를 생성하고  (100 포함하지 않음) (random() 함수가 0 이상~1.0 미만 사이의 수를 생성)
- guessHash가 targetHash보다 작으면 성공
목표로 하는 90 이내 (또는 10) 이내의 수가 생성이 되면 반복을 종료하게 된다.

아래 6번째 줄을 90 또는 10으로 해서 실행을 해보자.
TargetHash 한도를 90으로 하면 보통 1회, 10으로 하면 훨씬 더 많은 횟수가 필요하였다.
직접 10 이하의 수를 4번 시도해 보니 5, 22, 7, 16번 회만에 생성되었다.
그러니까 90으로 하면 난이도가 낮고, 10으로 하면 난이도가 높다는 것을 알 수 있다.


```python
import random
found=False
NONCE=0
while found==False:
    guessHash=int(random.random()*100)
    if guessHash<90:
        found=True
    NONCE+=1
    print("NONCE: ", NONCE, "guessHash: ", guessHash)
print("Solved ", "NONCE: ", NONCE, "guessHash: ", guessHash)
```

    NONCE:  1 guessHash:  66
    Solved  NONCE:  1 guessHash:  66


## 연습문제 5-7: 해시 맞추기 2

블록헤더 데이터의 해시 값에 NONCE를 증가시키면서
앞 자리의 0의 개수를 맞출 때까지 반복한다.
(1) 찾고자 하는 해시가 ```0000```로 시작한다고 하자. 몇 회만에 찾는지 출력하세요.
(2) 찾고자 하는 해시가 ```00000```로 시작한다고 하자. 몇 회만에 찾는지 출력하세요.
(3) 찾고자 하는 해시가 ```000000```로 시작한다고 하자. 몇 회만에 찾는지 출력하세요.

난이도가 어떤 경우가 높았으며, 난이도에 따라 찾는 회수의 차이가 있는지 설명하세요.

## 연습문제 5-8: 계좌이체 스크립트 작성 1

geth서버를 포트번호 8446에 하나 더 띄우세요 (geth@8446이라고 명명).
geth@8446의 chainid, nteworkid는 36번으로 설정한다.
geth@8446에서 계정을 2개 만들고, 충전을 해 놓는다 (coinbase에 5 ether 이상).
아래 문제는 eth 스크립트로 작성하여 풀고, geth --exec 'loadScript()'로 실행한다.
스크립트는 하나 이상 작성해도 된다. 3개까지 허용되고, 4개 이상은 감점하게 됩니다.

* 1-1 geth@8446에서 admin.nodeInfo 출력
* 1-2 geth@8446에서 계정, ether 잔고 출력 (잔고가 5 ether 이상 있어야 함)
* 1-3 geth@8446에서 블록번호를 출력
* 1-3 geth@8446 coinbase에서 geth@8446 2번째 계정으로 1.1111 ether 계좌이체
* 1-4 계좌이체의 hash값을 사용해 getTransactionReceipt 출력
* 1-5 계좌이체가 성공했다면 geth@8446의 수신측 계정잔고, 수신측 잔고변화 ether, 블록번호를 출력하세요.

## 연습문제 5-9: 계좌이체 스크립트 작성 2

geth서버를 포트번호 8446에 하나 더 띄우세요 (geth@8446이라고 명명)
현재 8445 포트를 쓰고 있는 geth(geth@8445라고 명명)를 포함해서, 총 2개의 geth 서버가 실행된다.
geth가 2개가 뜨지 않으면, geth 띄울 때 --ipcdisable 스위치를 추가한다.
아래 문제는 eth 스크립트로 작성하여 풀고, geth --exec 'loadScript()'로 실행한다.
스크립트는 하나 이상 작성해도 된다. 3개까지 는 허용되고, 4개 이상은 감점하게 됩니다.

* 1-1 geth@8446 coinbase에서 geth@8445으로 coinbase로 1.11 ether 계좌이체
* 1-2 계좌이체의 hash값을 사용해 getTransactionReceipt 출력
* 1-3 계좌이체가 성공했는지, 실패했는지 적으세요.
    성공했다면 수신측 geth@8445의 계정, 잔고 ether, 블록번호를 출력하고 실패했다면 그 이유를 적으세요.

## 연습문제 5-10: 계좌이체 스크립트 작성 3
geth서버를 포트번호 8446에 하나 더 띄우세요 (이하 geth@8446이라고 함).
geth@8446의 nteworkid는 36번으로 설정한다.
geth@8446에서 계정을 2개 만들고 (이하 계정A, 계정B라고 함), 충전을 해 놓는다
아래 문제는 eth 스크립트로 작성하여 풀고, geth --exec 'loadScript()'로 실행한다.
스크립트 파일은 하나 이상 작성해도 된다.
아래 문제는 모두 geth@8446에서 실행한다.
참고로 JSON 형식이라서, "Object" 내용을 볼 수 없는 경우
JSON.stringify()함수를 사용하면 문자열로 변환되어 출력된다.

* 1-1 admin.nodeInfo를 살펴보면, ip와 chainId가 있다.
	이 두 정보를 출력하자.
	부가적으로 if문을 사용하여 자신이 설정한 chainId 36이 private network인지 아닌지 출력한다.
* 1-2 계정A, 계정B의 wei, ether 잔고를 출력한다 (5 ether 이상). 그리고 transaction count를 출력한다.
* 1-3 계정A -> 계정B로 0.00000000000010101 ether를 계좌이체하고, 해시를 구한다.
	이 거래는 마이닝이 필요하다.
	마이닝을 시작할 때, 해시값과 더불어 시작한다는 출력을 한다.
	끝나면 끝나면 "mining done" 출력하세요.
* 1-4 계좌이체 이후, 잔고와 transaction count를 출력한다. 차이는 계산하여 출력한다.
	계좌이체의 hash값을 사용해 getTransactionReceipt의 gasUsed를 출력한다.
	gasUsed를 한화로 출력한다. 1 ether에 2,500,000원으로 가정한다.

대략 다음과 같이 출력하도록 프로그램한다.
```python
1
- Your chainId 33@192.168.25.36 is a private network
2
- Before
	- coinbase balance in Wei: 2.4449013749999999291819e+22 ether: 24449.013749999999291819
	- account1 balance in Wei: 1110000000000708181 ether: 1.110000000000708181
	- transaction count:  47
3
...mining start 0x96d7de4f91d594de0e2ce640da685f2dc5c58030367e36ff282cf7008f9a62db
mining done...
4
- After
	- coinbase balance in ether: 24454.013749999999190809
	- account1 balance in ether: 1.110000000000809191 increased by 100992
	- transaction count:  48 increased by 1
	- gas used: 0.000000000000021 won (1 ether = 2500000): 5.25e-8
```


```python
!cat _gethNow.sh
```

    _dir=$HOME/eth/
    _log=$HOME/eth/my.log
    geth --identity "jslNode" \
    --allow-insecure-unlock \
    --rpc --rpcaddr "117.16.44.45" --rpcport "8445" --rpccorsdomain "*" \
    --datadir $_dir \
    --port "38445" \
    --rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
    --networkid 33 


```python
%%writefile src/ethEx1.js
console.log("1");
console.log("- nodeInfo: ", JSON.stringify(admin.nodeInfo.protocols.eth.config));
var chainId = admin.nodeInfo.protocols.eth.config.chainId;
var ip = admin.nodeInfo.ip
if (chainId>=5) { console.log("\n- Your chainId "+ chainId + "@" + ip + " is a private network"); }
else if (chainId==1) { console.log("Your chainId "+ chainId + " is NOT a private network"); }
else  { console.log("[ERROR] Your chainId "+ chainId); }
console.log("2");
console.log('- coinbase: ', eth.coinbase);
var acc0 = eth.accounts[0];
var acc1 = eth.accounts[1];
var bal0 = eth.getBalance(acc0);
var bal1 = eth.getBalance(acc1);
console.log("- Before")
console.log('\t- coinbase balance in Wei: ' + bal0 + " ether: " + web3.fromWei(bal0, "ether"));
console.log('\t- account1 balance in Wei: ' + bal1 + " ether: " + web3.fromWei(bal1, "ether"));
console.log('\t- blocknumber: ', eth.blockNumber);
var tCountBefore=eth.getTransactionCount(eth.coinbase);
console.log('\t- transaction count: ', tCountBefore);
console.log("3");
//101010 wei = 0.00000000000010101 ether
//eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1], value:101010});
//personal.unlock() is needed before transaction
var thash = eth.sendTransaction({from:eth.coinbase, to:eth.accounts[1], value:101010});
console.log('...mining start '+thash);
miner.start(1);admin.sleepBlocks(1);miner.stop();
console.log('mining done...');
console.log("4");
console.log("- After")
var bal0new=eth.getBalance(acc0);
var bal1new=eth.getBalance(acc1);
console.log('\t- coinbase balance in ether: ' + web3.fromWei(bal0new,"ether"));
console.log('\t- account1 balance in ether: ' + web3.fromWei(bal1new,"ether") + " increased by " + (bal1new - bal1));
console.log('\t- block number: ', eth.blockNumber);
var tCountAfter=eth.getTransactionCount(eth.coinbase);
console.log('\t- transaction count: ', tCountAfter + " increased by " + (tCountAfter - tCountBefore));
console.log('\t- median gas price: ', eth.gasPrice);

var tr=eth.getTransactionReceipt(thash);
gasUsedEther = web3.fromWei(tr.gasUsed,"ether");
console.log("\t- gas used: " + gasUsedEther + " won (1 ether = 2500000): " + gasUsedEther * 2500000);

//var my = eth.getBlock('latest')
//console.log(JSON.stringify(my))
//console.log('gasLimit: ', my.gasLimit)
```

    Overwriting src/ethEx1.js


thash는 비동기 문제로 그 값이 저장이 안될 수 있다.


```python
!geth --exec "loadScript('src/ethEx1.js')" attach http://117.16.44.45:8445
```

    1
    - nodeInfo:  {"chainId":33,"eip150Hash":"0x0000000000000000000000000000000000000000000000000000000000000000","eip155Block":0,"eip158Block":0,"homesteadBlock":0}
    
    - Your chainId 33@117.16.44.45 is a private network
    2
    - coinbase:  0x21c704354d07f804bab01894e8b4eb4e0eba7451
    - Before
    	- coinbase balance in Wei: 80000021786999967965 ether: 80.000021786999967965
    	- account1 balance in Wei: 184999843394000029444 ether: 184.999843394000029444
    	- blocknumber:  47210
    	- tranaction count:  268
    3
    mining done...
    4
    - After
    	- coinbase balance in ether: 9.00000108934995
    	- account1 balance in ether: 184.999843394000029444 increased by 0
    	- block number:  47211
    	- tranaction count:  269 increased by 1
    	- median gas price:  1000000000
    err: ReferenceError: 'thash' is not defined
    false

