~~이 내용은 책에 들어갈  내용은 아닌 것 같습니다. 코드 등을 올려놓을 온라인에 넣는 것이 좋을 것 같습니다. 강의 관련 질문들도 있고, 또 이런 내용들은 지속적으로 업데이트 될 수 있는데, 굳이 지면에 넣어야 할 필요는 없을 것 같습니다.~~

# 부록: 질의 응답

블록체인 프로그래밍을 배우다 보면, 어렵다는 소리가 여기 저기 들린다. 발생하는 오류도 매우 다양하다. 왜 그럴까?

첫째, 멀티언어를 사용하기 때문이다. 블록체인을 완성하려면 네트워크, 블록체인 자체 또는 호출하는 클라이언트 측에서도 오류가 발생하기도 한다. 게다가 이를 끊김 없이 이어주는 통합개발도구 IDE가 아직 충분하지 못하다.

둘째, 새로운 방식이라 그렇다. 블록체인 프로그래밍은 지금 배운다 해도 꽤 빠른 편이라서, 서적, 강좌 또는 온라인 도움말이 충분하지 않다.

세째, 발전하는 언어라서, Solidity의 문법이 자주 변경된다. 자바도 버전업이 잦지만, 블록체인 프로그래밍은 버전업이 되고 기존 버전과의 호환성이 취약하다. 

네째, 블록체인에 연결하기 위해서 사용했던 web3.js도 변화가 있었고 역시 호환되지 않는 부분이 발생한다.

지난 몇 년 강좌를 운영하면서, 실로 다양한 질문을 받게 된다. 질문을 취합, 정리하는 편이 좋겠다는 판단을 하였다. 때로는 어렵고, 오류가 난감할 수 있는데, 도움되기를 바란다. 

가장 일반적인 질문에 대한 답부터 시작해보자.

## 질문: 오류가 발생해요...

답: 원인을 아는 것이 중요해요.
블록체인은 멀티언어를 사용하기 때문에 어렵고, 오류가 발생하기 쉽고 파악이 어려워요
또 네트워크가 개입되니까 어려워요.

오류가 발생하면 당연히 질문해야죠. 그러나 가장 중요한 것은 무엇일까요?
원인을 아는 것이 중요. 원인을 알아야 해결할 수 있게, 대응을 체계적인 지식으로 습득할 수 있어요.
예를 들어, 배열이 오류. 원인을 알아보니 배열의 크기가 범위를 넘어서 IndexOutOfBounds라는 것을 알았어요. 그러면 해결이 쉽고, 다음에 유사한 오류에 대해 헤메지 않고 대응할 수 있어요.

즉, "어떻게 해야 하나...?" 이런 접근은 지양해야. "무엇을 해야 하지" 이런 식으로 대응해야 해요.
'무엇'을 생각한다는 것은 오류의 원인을 알고, 고치려면 대안이 무엇인지 알아야 가능하거든요.
그 대안을 하나씩 테스트하는 것이 매우 매우 중요해요.

프로그램을 실행하고, 오류가 발생한다고 바로 질문하는 것은 좋은 습관이 아니예요.
몇 째줄에서 오류인지, 그 원인이 무엇인지 먼저 생각하세요.

우리는 geth, node의 경우 일괄 실행하고 있지만, 단말을 열어 인터랙티브하게 오류발생 확인하면 어느 줄에서 발생한 오류인지 알 수 있어요.
Solidity의 경우도 REMIX에서 어느 줄이 오류가 발생하는지 확인하고, 해결하고 다음 단계로 진행하도록 한다.

## 질문: geth attach 오류

geth attach 실행을 했는데 안돼요ㅠㅠ 오류를 붙여 놓여 놓습니다.
```
C:\Users\KH\Code\201815053>geth attach http://192.168.0.2:8345
Fatal: Failed to start the JavaScript console: api modules: Post "http://192.168.0.2:8345": dial tcp 192.168.0.2:8345: connectex: No connection could be made because the target machine actively refused it.
```
답:
접속하려면 자신의 IP를 적어야 해요. 오류 메시지를 잘 보세요. IP가 ```192.168```로 시작하는 것을 보니, 공유기에서 부여한 것으로 보입니다. ```geth attach http://localhost:8345``` 또는 ```geth attach http://127.0.0.1:8345```로 해보세요. 참고로, localhost의 IP 주소. 127.0.0.1 입니다.

## 질문: chainID 1
충전하기 전에 _gethNow.bat 부분에서 unlock이 안되고 사설망을 접속한 후 admin.nodeInfo를 해보았더니 chainid가 33이 아닌 1로 나옵니다ㅠㅠ 어디서 잘못된 건가요??

답:
보통 디렉토리 명이 잘 못되면 그런 오류가 발생해요. 강의자료를 따라 동일한 디렉토리를 사용한 것은 아닌지, 디렉토리가 실제로 존재하는지, 오타가 있는지 확인해보세요.

답변 후 회신, "해결하였습니다!. ethash 가 아니라 Clique 를 사용하는 genesis 파일을 사용했었습니다." 제공된 파일의 chainID를 수정하지 않고 사용해서 오류가 발생한 것으로 판단합니다.

## 질문: testnet 접속 오류
```geth --syncmode fast --cache=1024```를 했는데도 testnet이 생기지 않습니다. 그래서 ```geth --testnet```하면 Incorrect Usage. flag provided but not defined: -testnet 에러가 뜹니다.

답: 버전이 변경되면서 --testnet이 아니라 --ropsten을 사용합니다 ```geth --ropsten```. 그리고 스위치에는 더블 하이픈을 사용해요. 오류 메시지를 보니, 하이픈이 하나입니다.

## 질문: 이더리움 디렉토리
...그리고 appdata 와 그 하위 디렉토리가 만들어지지 않습니다. Roaming이 아니라 Local에 이더리움이 생깁니다.

답: 생성되는 디렉토리는 운영체제 별로 차이가 있을 수 있어요.
다르다고 해도 생성이 되었다면 문제가 없어요.
공중망 경우:

- 공중망 윈도우: ```C:\Users\사용자\AppData\Roaming\Ethereum\geth```
- 리눅스: ```/home/사용자/.ethereum```
사설망은 당연히 ```genesis.json```을 생성할 때 명시한 디렉토리 (예를 들어, ```\eth```)에 생성이 됩니다

답변 후 회신: "AppData및 Roaming이 아닌 AppData 및 Local에 생기는 듯 합니다"라고 했다.

## 질문: 모듈 ```--rpcapi``` 오류
```--rpcapi```에서 shh를 제거했지만 db에 또 문제가 있는 경우에는 어떻게 해야할까요? 버전은 1.9.23입니다."

답: 최신 버전을 사용할 때 ```--rpcapi``` 작동이 안된다는 질문이 좀 있었어요.
명령어에 오타가 없다면, 이전 버전에는 그런 질문이 없었으므로 조심스럽지만 최신 버전이 이유가 아니가 추측합니다.
rpcapi는 원격접속해서 사용하는 기능을 말해요.
가장 간단한 대처방법은 안되는 부분은 제거하면 됩니다. 원격접속하면서 그 기능들을 다 안씁니다.
```--rpcapi```에서 shh를 제거하고 실행하라는 의미.

계정 생성하면서 personal을 사용할 수 없다는 질문이 있어요. 이 경우 rpcapi에서 personal을 제거해 놓았는지 확인하세요.
personal을 사용하려면, gethNow를 만들어야 해요. 그러나 새로운 계정을 만드는 geth account new 명령어로 대체될 수 있어요. 그렇게 잘 되었다는 "gethNow를 새로 만들어서 해결했습니다 감사합니다" 회신이 있었다.

## 질문: "INFO looking for peers"
답:
INFO이므로 오류가 아닙니다. 설정에 따라 Peer를 찾고 있다는 의미.

## 질문: peer가 없다고 합니다.
admin.addPeer()를 이용해서 결과값 True를 반환했는데 admin.peers에서 아무것도 나오지 않습니다. 어떻게 해야 하나요
admin.nodeInfo 이용해서 나온 enode로 admin.addPeer(enode@IP:port)하면 true는 나오는데 net.peerCount를 하면 0이 출력됩니다.

답:
찾으려고 하는 네트워크 주소를 내부 IP로 변경할 때 nat 네트워크 주소 변환 테이블을 사용해요.
기본으로 geth는 외부 IP를 사용한다. 여기에 혼돈이 있어 그럴 수 있으니 "_gethNow.bat"의 스위치에 --nat none을 넣어서 해보거나 --nat extip:<IP address> 명시를 해주세요.

분산시스템의 특성 상 peer는 정적으로 항상 고정되어 있지를 않아요. 
net.peerCount는 고정되지 않고 변동한다.
```
> net.peerCount
1
> net.peerCount
2
> net.peerCount
1
```

외부 노드에서 무작위로 접속하려고 시도하는데, 허용되면 증가한다. 무작위 접속을 끄고 싶으면 --nodiscover 스위치를 끈다.

## 질문: "Access is denied"
_gethNow.bat를 실행했더니 Fatal: Error protocol stack: Access is denied라고 오류가 생깁니다. 어떻게 해야하나요ㅠㅠ

답:
단말을 관리자 권한으로 열어보세요.

## 질문: 경로설정오류
단말에서 solc --version이 안됩니다.

답:
C:\"Program Files"\solidity-windows\solc --version 윈도우 단말에 타이핑해보세요.
버전이 출력된다면, C:\Program Files\solidity-windows\ 경로가 맞다는 것이고
그렇다면 경로설정에 오타가 있거나, 저 경로대로 인식이 안되었다는 의미입니다.
윈도우 단말에 path 타이핑, 엔터하고 거기서 C:\"Program Files"\solidity-windows\가 포함되어 있는지 확인해보세요.

solc 경로는 PATH 변수에 추가해야 하고, 경로가 올바르게 설정되어 있는지 보려면, 화면에 다음과 같이 해보세요.
C:\Users\jsl>path | find "solidity"
- path는 설정경로를 화면에 출력
- find는 해당 문자열 찾음
- 두 명령어를 연결하는 것은 |

## 질문: 지급해제 불가 오류
지급해제가 금지되었다는 오류 Error: account unlock with HTTP access is forbidden

답: 계정의 잔고를 사용할 수 있게 잠금해제할 경우 발생하는 오류입니다.
```--unlock``` 대신 ```--allow-insecure-unlock```을 사용하세요.

강의에서 설명한대로, unlock이 http rpc에서 문제가 될 수 있어요.
1.9인가 최근 버전에서 계정을 풀 수 없도록 했어요. 그 이유는 http는 공중망이라, 즉 누구나 접근 가능할 수 있기 때문에 보안이 취약할 수 있기 때문이예요. 계정의 인덱스로 특정 계정의 지급을 해제하려고 하지 말고, allow-insecure-unlock로 하면 됩니다.

## 질문 Snapshot extension registration failed.
peer connected on snap without compatible eth support

답: 오류이지만, 심각하지 않고 작동에 문제는 될만한 오류가 아닙니다.
peer가 서로 접속하려고 하지만, 거절되는 경우입니다 (참조: "질문: peer가 없다고 합니다")
동기화는 문제없이 될 것이니 걱정하지 않아도 됩니다.
사용하는 Geth버전이 서로 다르기 때문인 것으로 보입니다.
```geth --snapshot=false```을 꺼놓아도 발생한다면 그냥 두세요.

또는 snapshot extension registration failed는 현 v1.10에서 발생하는 오류. v1.10.2-unstable-91726e8a-20210316 으로 업데이트하거나, 버전을 낮추어서 하세요.

## 질문: Error starting protocol stack: Access is denied. 

답: Geth가 동일한 포트를 또 사용하려고 시도하기 때문에 발생하는 오류입니다.
아마 Geth를 여러 번 실행하면서 --port로 명시한 포트가 점유되고 또 사용하려고 하기 때문인 것으로 보입니다.
윈도우에서만 발생하는 오류이고, 리눅스에서는 발생하지 않아요.
(1) 해당하는 포트를 사용하는 프로세스를 죽이거나
(2) --ipcdisable 스위치로 Geth를 실행하세요 (복수의 Geth를 실행하려고 할 경우에)

## 질문: "Error: Invalid JSON RPC response"
UnhandledPromiseRejectionWarning: Error: Invalid JSON RPC response: ""
답: 발생한 상황의 설명이 없어 정확한 답변이 어려워요.
그러나 RPC 측의 반환 JSON이 오류가 있다는 것이예요.
geth 네트워크가 떠 있는지 확인하세요. 혹시 띄우는 것을 잊었는지? 또는 떠 있지만, contract address 잘 못 넣었어도 그래요. 사용단계에서 자신의 coinbase를 넣지 않았는지 확인하세요. contract의 address를 넣어야 해요.

## 질문: 마이닝 해시를 맞추는 5주차 문제
마이닝 해시를 맞추는 5주차 3번 문제가 이해가 되지가 않습니다.
해시를 100미만 양수로 정하는 법도 모르겠고,
nonce 값을 사용하지 않는 다는 것도 어떤 식으로 코드를 짜라는 것인지 감이 잡히지를 않습니다.
또한, 난이도에 따라 목표해시가 결정된다고 하셨는데 강의자료에서 본 block_diff = parent_diff + parent_diff..... 이 연산에서 parent_diff가 없어서 난이도를 구하는 법도 모르겠습니다...

답: 강의자료 '난이도' 부분에 나와있는 소스코드를 잘 이해하고, 수정하면 됩니다.
문제의 답에서 난이도는 수정하지 않아요. 그리고 해시를 생성하지도 않아요 (학생들이 어렵다고 해서, 문제를 쉽게 하려고)
목표해시를 정하는 과정을 이해하면, 불과 강의코드 몇 줄만 수정하면 풀립니다.
- targetHash는 90 (또는 10)으로 정했다고 가정
- guessHash는 random으로 1~100 사이의 수를 생성
- guessHash가 targetHash보다 작으면 성공

## 질문: echo 명령어가 되지 않아요

답: 강의자료 확인하세요.
강의자료에서 윈도우에서는 어떻게 명령어를 사용해야 하는지 설명하고 있어요.
"윈도우에서는 pipe "|", 변수값을 입력받는 "set /p" 명령어, 파일에 쓰고 ">", 파일에 붙여넣는 ">>" 명령어를 사용한다."
다음 줄에 명령어:
```
echo | set /p="exports.compiled=" > src\Timer.js
```

그리고다음 줄에 다음과 같이 파일 내용을 출력할 수 있어요.
```
type src\Timer.js
```

## 질문: "type src/Hello.sol" 오류
윈도우에서 되지 않아요.
```
pjt_dir>type src/Hello.sol
The syntax of the command is incorrect.
```

답: ```type src\Hello.sol```
강의 자료에서는 윈도우, 리눅스 섞여 있어서 그래요. 리눅스, 맥, 윈도우 운영체제에 따라 여러 가지 차이가 있기 마련입니다.
* 윈도우: ```type```, 디렉토리 구분자 ```\```, ```dir```
* 리눅스, 맥: ```cat```, 디렉토리 구분자 ```/```, ```ls```

그렇다고 어느 운영체제가 우수한지 단언할 수는 없어요.
강의에서 설명하였듯이, 개발을 한다면 리눅스가 보다 적합한 운영체제 입니다.
수 많은 회사들이 유닉스 서버를 사용하기 때문에, 윈도우 보다는 리눅스를 사용하는 것이 좋습니다.
맥도 유닉스 운영체제 변형이라 좋습니다.

## 질문 js Object 출력 오류
답: stringify() 해주세요.
예를 들어, 이러한 JSON이 있다고 하자.
var obj = { name: "John", age: 30, city: "New York" };

var myJSON = JSON.stringify(obj);
이렇게 하면 JSON.stringify()은 문자열로 변환해준다.

## 질문: 오류 디버깅
답:
* (1) REMIX에서 기능이 올바르게 실행되는지 먼저 확인해야 합니다.
* (2) REMIX에서 테스트가 완료되면, 비로서 배포해야 합니다.
* (3) 배포하거나 사용하면서 오류가 발생하면 노드에서 인터랙티브하게 한 줄씩 해보세요.
* (4) 질문하려면, 오류가 발생하는 ```노드화면을 캡쳐해서 올려보세요.```

## 질문: forwardTo() 함수에 금액?
forwardTo() 함수에 금액이 없어도 되는지?
답: 함수명이 말해주듯이, 전달하는 기능입니다. 금액은 전송하는 사람이 정하고 그 금액만큼 전달되는 기능.
amount 매개변수를 추가해도 좋아요.
그러나 amount로 금액이 전달되는 것이 아닙니다. msg.value로 전달됩니다.

## 질문: ```deposit(1111)```을 해도 잔고가 증가하지 않아요
답
```deposit()``` 함수는 블록체인에 기록되는 함수이므로 ```send()```를 해야 하고, ```transaction``` 데이터를  구성해 주어야 해요.
 ```send({from:"0xbA606f...d9e9",gas:...,value:1111});```

## 질문: Source file requires different compiler version
pragma solidity 0.5.0이라고 설정하고 컴파일 하면 오류가 발생합니다.

답: REMIX에서 컴파일러 버전을 동일하게 선택하여 컴파일 하세요.

## 질문: 마이닝이 너무 오래 걸려요.
답
(1) 마이닝 difficulty가 높아서 그럴 수 있어요. 예를 들어, 0x00010 이 정도로 낮추어 보세요. 또는 0x0으로 설정해도 됩니다.
이 난이도는 한 번 genesis.json에서 설정되고 나면, 알고리듬에 따라 증가하거나 감소하게 됩니다.
따라서 블록번호가 증가할수록 증가할 수 있어요.
설치할 때 다음 go함수 수정하면 난이도를 낮춰 마이닝 시간을 줄일 수 있다고 합니다.
func CalcDifficulty(config *params.ChainConfig, time, parentTime uint64, parentNumber, parentDiff *big.Int)

또한 하드웨어의 CPU, RAM이 낮아서 그럴 수 있어요.
An intel i5 quad core 2.6Ghz with 16Gb RAM it takes less than 3 minutes
geth 마이닝에 메모리 할당을 늘려보세요.

## 질문: 왜 노드 hangs??
답:
모든 것이 잘 설정되어 작동한다면, 동기화 때문에 그래요. 메인넷은 원래 시간이 오래 걸려요.
사설망은 바로 시작했다면 동기화할 블록이 없어요. 다만 peer enode를 추가한다면 필요할 수 있어요.

## 질문: "Switch sync mode from fast sync to full sync"
답: "Switch sync mode from fast sync to full sync"는 오류가 아니라 경고.
빠른 동기화 (fast)가 거의 끝나가기 때문에 그래요. 그 시점에는 완전동기화로 진행해야 되거든요.

## 질문: .ethash가 아주 커요. 삭제해도 되나요?

답: 삭제해도 됩니다. ethash는 해싱알고리듬입니다.

## 질문: 잔고가 정확하지 않아요
첫번째 계좌에서 두번째 계좌로 333wei송금시, 두번째 계좌의 잔액은 333만큼 증가하는데, 첫번째 계좌의 잔액은 그대로인것으로 출력해요.

답: 첫번째 계정은 마이닝을 하기 때문에 잔고 변동이 송금액만큼 감소하지 않을 수 있어요.

## 질문: "newContractInstance.options.address" 오류
배포할때 "newContractInstance.options.address" 이 부분에서 에러가 뜹니다. 주소도 geth 계정으로 바꿨고, 포트번호도 이상 없는데 배포 주소를 얻을 수가 없습니다.

답: deploy().send() 오류가 의심됩니다.
정확한 상황을 파악할 수 없어요. 노드 화면 캡쳐로 질문하세요.
"newContractInstance.options.address" 이 부분 오류가 맞나요?
그 명령어가 속한 deploy().send() 오류로 보입니다.
"newContractInstance.options.address" 오류가 맞다면 하나씩 줄여서 올바르게 출력되는지 확인해보세요.
"newContractInstance.options" 이렇게. [Object]로 출력되면 맞고 그 내용을 보려면 stringify() 함수를 이용하세요.

## 질문: gaslimit 오류
마이닝을 하다가, gaslimit오류가 납니다.
답:
* 실제 오류 출력과 같이 gaslimit오류이면, genesis.json에 설정한 gasLimit을 수정해야 겠지만 그럴 가능성은 적습니다.
* 오류가 발생하면 원인을 알아내는 것이 중요합니다. 그 원인은 국지적이지 않을 수 있습니다.
* 일단 geth창의 변화를 관찰해보세요. 배포 자바스크립트를 실행했을 경우, 그 거래가 접수되는지.
아무 변화도 없다면 명령문 자체의 오류일 가능성이 높아요.
* 접수가 되었다는 것을 확인하면, geth attach창으로 이동해서 eth.pendingTransactions를 출력하면 그 거래가 출력되는지.
* 마이닝을 시작하고 실패한다면 geth창과 geth attach 창의 eth.pendingTransactions가 어떤 변화가 있는지 보세요.

gaslimit은 수정하기 어렵습니다. 꼭 gaslimit과 관련된 오류가 아닐 가능성이 높기 때문입니다.
solidity 로직 오류로 인해 gas비가 많이 산정되는 것도 원인이 될 수 있어요.
* REMIX로 가서 배포한 후 기능을 실행하면서, 우측 하단의 로그창을 크게해서 오류가 발생하는지, 정상적으로 처리된다면 gas비가 얼마나 나오는지 확인해보세요.
debug옆 펼침 화살표버튼을 누르면 거래상세에서 gas비용을 알 수 있어요.
그 gas 비용과 내가 입력한 트랜잭션의 gas비와 비교하고, 실제 genesis.json에 설정된 gasLimit 0x8000000 보다 클 것인지 산정해야 합니다.
* gas를 넉넉하게 산정해도 환불되니까 괜찮다고 생각할 수 있어요. 그러나 너무 많은 gas비를 산정해도 이런 오류가 발생하게 됩니다.
달리 말하면 프로그램의 로직이 너무 복잡하여, 많은 gas가 필요할 수 있어요. 달리 말하면 컨트랙이 오류는 없지만 논리적으로 손 봐야 한다는 의미입니다.
* gaslimit을 수정할 수 있지만 사설망이라고 하더라도 권고하고 싶지는 않습니다.
geth --targetgaslimit '10000000'

버전을 0.5로 낮추어서 해결이 된 경우도 있습니다.
또는 컴파일할 때 (0.8.2) evm을 petersberg로 해서 풀린 경우도 있습니다.

## 질문: Out of gas오류

답: gas비 산정이 적어 그래요. 
계정 주소를 다르게 적어도, 당연히 지급할 수 없으니 Out of gas 오류가 발생할 수 있어요.
ArrayTest2.sol 동적배열 배포하면서 1백만 이상이 나오고, estimateGas보다 1적게 나오면 오류가 발생해요.

## 질문: at 명령어 오류
답: 객체를 생성할 경우, 사용하는 at 명령어는 하위버전의 것이예요. 강의에서 설명한 바 있어요. 강의자료 잘 확인해 보세요.

## 질문: 웹소켓 오류
웹소켓이 안돼요
답: 
웹소켓을 사용하려면 관련 스위치를 (1) _gethNow.bat에 추가하세요.
포트 8446에 웹소켓을 사용한다는 의미
geth --identity "jslNode" \
--datadir $_dir \
--allow-insecure-unlock \
--rpc --rpcaddr "117.16.xxx.xxx" --rpcport "8445" --rpccorsdomain "*" \
--rpcapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
--ws --wsaddr="117.16.xxx.xxx" --wsport 8446 --wsorigins="*" \  웹소켓을 사용하려면 이 부분을 추가하세요.
--wsapi "admin,db,eth,debug,miner,net,shh,txpool,personal,web3" \
--port "38445" \
--networkid 33

## 질문: "the tx doesn't have the correct nonce" 오류 
답:
```
> eth.getTransactionCount(eth.accounts[0])
66
> eth.getTransactionCount(eth.accounts[0], 'pending')
66
```
이 경우 count는 66이고, nonce는 0부터 시작하므로 65가 되겠죠.
getTransactionReceipt에도 nonce가 있어요.
트랜잭션 거래를 구성할 때 {from: , gas: , value: , nonce: <숫자>}를 설정해 줄 수 있어요.
그러나 손코딩으로 nonce를 추적하는 것은 바람직하지 않아요. 
nonce는 일련번호입니다.
클라이언트 측에서 nonce를 거래 당 1씩 자동으로 증가시켜 갑니다.
nonce가 올바른 순서로 차례대로 처리되지 않는 경우 문제가 발생할 수 있습니다.
예를 들어, 최근의 nonce가 65라고 할께요.
* 동일 계정에서 65 또는 그 이하 nonce로 거래를 전송하는 경우, 당연히 오류
* 동일 계정에서 한 칸 띄어 67로 전송하는 경우, 66이 처리되지 않는 한 67은 대기

질문한 오류는 복수의 트랙잭션을 하나의 파일에 넣는 경우, 이런 가능성이 높겠죠.
await를 사용하면 그런 오류가 발생할 확률이 줄어들게 됩니다.

## 질문: 여러 건의 order() 등 복수 거래가 발생되는 경우
복수거래 deposit(), forwardTo(), withdraw() 또는 여러 건의 order() 등 복수 거래가 발생되는 경우 오류가 나요.
어떻게 하죠?

답: geth 네트워크를 사용할 경우, 복수 거래는 오류가 발생하기 마련입니다.
ganache는 그런 문제가 발생할 까닭이 없어요.
왜 그럴까요? geth는 하나씩 단건 거래를 처리해야 해요.
반면에 ganache는 여러 건을 잘 처리해요 (마이닝을 하지 않아도 되니까)

건수를 제한하지 않고 miner.start()를 실행해요. 다 끝나면, miner.stop() 해도 좋아요.
마이닝을 걸어놓아도 계정을 지급해제 해놓아야겠죠!!
계정해제를 기한을 정해서 다음과 같이 해요.
마지막 인자는 해제되는 기간을 의미한다. 그 기간을 초로 설정할 수 있고, 0이면 무기한을 말한다.
```
geth> personal.unlockAccount(eth.accounts[0], "your password", 0)
> true
```
web3.js에서 비밀번호를 해제하는 방법이 있나요?
```
node> web3.eth.personal.unlockAccount("0x..", "<passs>", 1000)
```
## 질문: 마이닝하면서 timeout 오류
Error: Transaction was not mined within 750 seconds.
geth 8445에서 deploy를 할 때마다 실행이 오래 걸리다가 이런 오류가 뜹니다. 

답: ```transactionPollingTimeout``` 기본 값이 750초로 한정
사설망 geth@8445에서 deploy하면 timeout 오류가 발생할 수 있어요.
블록체인이 최신화가 되어 있지 않으면 그런 오류가 잘 발생합니다.
eth.pendingTransaction 명령어로 현재 적체되어 있는 transaction이 있는지 확인하세요.

nounce가 올바르지 않은 경우 실행이 되지 않는 경우가 있어요.
어떤 거래의 gas price가 낮게 책정되어 실행되지 못하면, 그 이후 거래가 모두 막히게 되어요.
nounce가 차례대로 실행되기 때문입니다.
(1) 첫 째 방법은 eth.resend(eth.pendingTransactions[0], web3.toWei(20, 'gwei'), 2000000)
(2) 다음 방법은 그냥 삭제하는 방법은 transactions.rlp을 삭제하고 사설망을 재시작하세요.
자신의 사설망 디렉토리 geth밑에 transaction.rlp 파일이 있어요.

## 질문: Cannot read property 'returnValues' 오류
이벤트발생을 확인하려는데, Cannot read property 'returnValues' 오류가 뜹니다.

답:
returnValues 오타일 가능성이 큽니다.
JSON.stringify(result); 이런 식으로 한 계단씩 올려서 출력하고, 확인해보세요.
json 출력을 살펴보면, 계층식의 구조가 보일 거예요. 거기의 필드명이니 오타없이 적으면 됩니다.

## 질문: "invalid opcode" 오류.
답:
assert문이 false가 되면 이런 오류가 발생합니다. 그러니까 
또한 이 밖에도:
* 배열의 범위를 벗어나는 인덱스를 사용하거나
* 0으로 나누거나
* enum의 상수 범위를 벗어나는 경우

## 질문: invalid opcode: SHR
답:
Opcode 0x1c 또는 SHR 오류가 발생하는 경우를 말합니다.
shr은 right shift 오른쪽으로 한 비트 이동 (bitwise shift right.)
shr은 비트연산자입니다. Solidity를 컴파일하면 opcode로 만들어지게 되는데, 어셈블리 코드같은 것이 있어요.
opcode에 나오는 비트연산자 shr에 문제가 있다는 의미인거죠.

SHR opcode은 2019년 2월 Constantinople 버전으로 업데이트하면서 에 등장하였어요. 
오른쪽 왼쪽 비트 연산을 하는 shr, shl이 gas비용을 35 -> 3으로 줄여주기 때문에 적용되었어요 
(참조: EIP 145: Bitwise shifting instructions in EVM)
Constantinople의 1단계 업그레이드는 블록 4,370,000에서 실행되었고 이를 Byzantium이라고 합니다.
2019년 2월 블록 7,280,000에서 2단계 업그레이드 되었어요.
따라서 이 코드를 인식하지 못해서 이런 문제가 발생하기 때문에
아래와 같이, Constantinople, St. Petersburg이 지원되도록 genesis.json을 업그레이드 해줍니다.
이때 byzantiumBlock도 같이 설정해줍니다.
"byzantiumBlock": 0,
"constantinopleBlock": 0
참고로, St. Petersburg 업그레이드 (EIP 1283)는 Constantinople과 같은 블록에서 적용되었어요.

그렇게 되면, genesis.json은 다음과 같이 되겠지요.
https://ethereum.stackexchange.com/questions/74068/genesis-json-in-private-geth-network/74103#74103
```
{
"config": {
"chainId": 9731,
"homesteadBlock": 0,
"eip150Block": 0,
"eip150Hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
"eip155Block": 0,
"eip158Block": 0,
"byzantiumBlock": 0,
"constantinopleBlock": 0,
"petersburgBlock": 0,
"ethash": {}
},
"nonce": "0x0",
"timestamp": "0x5d5cdc87",
"extraData": "0x0000000000000000000000000000000000000000000000000000000000000000",
"gasLimit": "0x47b760",
"difficulty": "0x80000",   -> 거의 0으로???
"mixHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
"coinbase": "0x0000000000000000000000000000000000000000",
"alloc": {
"0000000000000000000000000000000000000000": {
  "balance": "0x1"
}
},
"number": "0x0",
"gasUsed": "0x0",
"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000"
}
```

또는 사용하는 버전을 낮추면 되겠죠. 그러면 SHR 비트연산자를 만나지 않겠죠.
어느 경우에서는 SHR이 발생하지 않고,  ```error: returned values aren't valid, did it run out of gas?```이 발생했고 버전을 낮추어서 해결했어요.

## 질문: "error: returned values aren't valid, did it run out of gas?"
답
먼저 gas 부족이라고 하니, 충분한지 확인해야 한다. gas가 충분하다면 gas 문제라고 말하고 있지만, 실제로는 아닐 수도 있다. 예를 들어, 컨트랙에 존재하지 않는 함수를 호출하거나, 다른 주소를 사용해도 gas비가 부족하다는 오류가 발생하기 때문이다.

이와 같이 오류가 진짜 원인을 잘 못 추정하면, 디버깅이 어렵기 마련이다.
먼저 이런 오류가 발생한 상황을 잘 파악하고 있어야 디버깅이 가능할 것이다.
ganache에서는 멀쩡하게 잘 실행되다가, geth@8445에 배포하면 문제가 되지 않는가?
그렇다면 solc version을 낮추어서 해보자.
나 자신도, 0.5.17에서 오류였으나, 0.5.0 정상 또는 0.6.12에서 오류였으나, 0.4.2에서 정상인 적이 있었어요.
REMIX에서 버전을 낮추어, 예를 들어 0.4.2로 컴파일하고, 그 abi, bin을 복사해서 사용하면 됩니다.
또는 web3 버전을 변경. 1.0.0-beta.55 -> 2.0.0-alpha 변경해서 문제가 해결되었다는 경우도 있다.

## 질문: msg.sender가 0x0000000000000000000000000000000000000000
genesis.json에 "byzantiumBlock": 0으로 되어있어요.

답: 버전문제
(1) genesis.json에서 "byzantiumBlock": 0을 제거하거나
(2) 1.9.4에서 발생한문제, 그 다음 날 바로 hotfix 1.9.5가 발표되었으니 업그레이드 하세요.

## 질문: ```require('./RspABI.json')``` 파일을 읽을 수 없어요.
노드단말에서 ```require('./RspABI.json')```하면 파일을 읽을 수 없어요.

답: 노드를 실행한 경로에 맞추어서 해보세요.
예를 들어:
C:\Users\jsl\Code\201711111> node 이렇게 실행하고, TimerABI.json을 읽는다 해요.
```파일 위치는 C:\Users\jsl\Code\201711111\src\TimerABI.json```

아래와 같이 상대경로를 잡아줘야 합니다.
```
> require('./TimerABI.json'); //Error: Cannot find module './TimerABI.json'
> require('src/TimerABI.json'); //Error: Cannot find module 'src/TimerABI.json'
> require('./src/TimerABI.json'); //정상
{ contracts:
   { 'src/Timer.sol:Timer':
      { abi:
         '[{"inputs":[],"name":"getNow",...}]' } },
  version: '0.6.1+commit.e6f7d5a4.Windows.msvc' }
```

위와 같이 대화형 방식의 경로 설정은, 배치 방식에서 다릅니다.
```
C:\Users\jsl\Code\201711111> node src/TimerDeployAbiBinFromFile.js 이렇게 일괄실행방식으로 하면
var _abiJson = require('./TimerABI.json'); //정상
```

## 질문: REMIX에서는 정상, node에서 호출하면 다른 결과?
REMIX에서는 모든 함수가 제대로 동작하여 원하는 결과가 나오지만, 똑같은 sol파일로 ganache에서 USE한 결과는 다르게 나온다면 어떻게 해야 하나요? 혹시 거래가 한꺼번에 발생하는게 문제인가 싶어서 USE파일을 나누어서 해봤는데도 여전히 REMIX에서의 결과와 다르게 출력됩니다ㅠㅠ

답:
REMIX에서 정상 작동하면 ganache에서도 문제가 없을 거예요.
여러 거래들이 하나의 파일에 있다고 하더라도, 실행하면 ganache에서는 문제가 없어요. node 코드에 버그가 있을 겁니다.
특히 REMIX에서 오렌지, 빨간 버튼인 send함수는 REMIX 우하단 콘솔 단말창을 열어서 녹색 정상 처리를 확인해야 해요.
REMIX에서 했던 절차에 따라 함수를 호출하지 않았거나, 함수에 await가 없어도 그럴 수 있어요.

## 질문: "Error: Returned error: Number can only safely store up to 53 bits"

## 답:
자바스크립트 Number 자료형의 최대값이 2^53^입니다. 자바스크립트 측에서 너무 큰 수가 발생해서 그래요. 그렇다 하더라도 이렇게 큰 수가 발생할 개연성은 매우 낮아요. solidity 코드에서 자바스크립트로 넘겨주는 값에 이상이 있는지 확인해 보세요.

