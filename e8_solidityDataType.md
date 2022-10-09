#  Chapter 8. Solidity 데이터 타입

간단한 스마트 컨트랙을 만들어 보게 된다. 먼저 라이센스를 설정에 필요한 SPDX, 버전을 정하는 Pragma, 다른 코드를 가져오기 위한 import문을 앞 부분에 적어 준다. 그리고 객체지향의 클래스와 비슷하게 멤버변수와 멤버함수를 구성한다. 데이터 타입 불린, 정수, 문자열, struct, 배열, 주소에 대해 배우게 됩니다. 어셈블리 같이 생긴 Opcode에 대해서 설명한다.

# 1. 컨트랙 프로그램

## 1.1 시작하기 두려워? 그렇지 않아!

블록체인에 프로그램을 만들려고 하니까 혹시 두렵지 않아?

새로운 기술이라서 사뭇 다를 것 같고, 인터넷 연결만 되면 누구나 내 프로그램을 호출할 수 있다는 것이 긴장될 수 있다. 자신감을 가지고 시작하자! 블록체인은 화폐를 다루는 플랫폼이라 조심스러워 하는 것이 당연하지만, 매우 안전하기도 하다.

Solidity는 객체지향 언어이고, 다른 프로그래밍 언어와 크게 다르지 않다. 클래스를 만들듯이, 컨트랙도 멤버변수와 멤버함수를 그렇게 프로그래밍하면 된다.

## 1.2 컨트랙은 어떤 모습일까?

프로그램에는 컨트랙 자체와 더불어 SPDX, 버전 pragma, 다른 소스코드를 불러오는 import문, 또한 도움말 comments를 포함하게 된다. 프로그램이 완성되면, 확장자 ```.sol```로 컨트랙을 저장한다.

### SPDX

저작권에 대해 어떤 규정을 따르는지 SPDX Software Package Data Exchange를 적는다. Solidity 버전 0.6.8부터 SPDX 라이센스를 넣어야 한다. (어떤 라이센스가 가능한지 자세한 정보는 https://spdx.org/licenses/ 페이지를 찾아 보자).

GPL을 따르는 경우는 아래와 같이 ```//```를 붙여서 도움말로 적는다.
```
//SPDX-License-Identifier: GPL-3.0-or-later
```

### Pragma

```pragma```는 컴파일러 버전을 선택할 때 사용한다. 아직 발전하고 있는 언어라서 버전업이 활발하게 되고 있고, **후방호환성**이 문제가 되기도 한다. 프로그램의 첫 줄에 적는다.

```python
pragma solidity <버전>;
```

### Import

import는 filename의 다른 컨트랙, 라이브러리를 읽어들여 사용하게 된다. 다른 언어에 비해 **라이브러리가 충분하지 못하여** 불편할 수 있다.

```python
import "filename"
```
그 파일 내 **컨트랙, 전역변수** 등을 그대로 가져오기 때문에 주의를 해야 한다. 이로 인해 현재 파일의 전역변수와 문제를 일으킬 수 있으므로 ```import ... as ...``` 구문을 사용하기도 한다.

```python
import "filename" as symbolName
```

### 도움말

도움말은 C 또는 자바 언어 스타일과 동일하다.

한 줄은 ```//```라고 맨 앞에 적어주어 도움말로 만든다. 여러 줄은 해당 블록 앞 뒤에 ```/*...*/```를 적어준다. Doxygen 스타일의 @태그를 다음과 같이 사용하는 것도 가능하다.
```python
/**
@param p1 The first parameter
@param p2 The second parameter
@returns the returned results
*/
```

### 컨트랙

#### 컨트랙은 멤버변수, 멤버함수로 구성된다.

컨트랙 Contract은 블록체인 상의 프로그램이지만, 그 개념이나 구현하는 객체지향언어에서 사용하는 **클래스와 유사**하다. 컨트랙지향언어로서 컨트랙을 객체지향의 클래스와 같이 구성하면 된다.

클래스가 (1) 멤버 **변수**와 (2) 멤버 **함수**로 구성하듯이 컨트랙도 마찬가지이다.

다른 컨트랙을 불러서 결합할 수도 있다.

또한 Solidity는 객체지향 언어로서, 컨트랙은 상속, 다형성을 사용하여 확장을 할 수도 있다.

#### 컨트랙 예제 프로그램

컨트랙을 구현하면 어떤 모습을 가질지 궁금할 것이다. 다음 ```SimpleChild.sol```은 상속까지 적용된 코드인데, 읽어보면 이런 컨트랙을 만들게 되는구나 하고 느낌이 올 것이다. 어떻게 보면 객체지향 자바와 닮았고, 메모리를 효율적으로 사용하는 관점에서는 C언어하고도 공통점을 발견하게 된다.

코드에서 문득 모르는 명령어가 당연히 있겠지만, 차츰 하나씩 배워가기로 하자. 간단히 설명하면:
* 상속 관계의 컨트랙은 ```is```로 표현한다.
* 컨트랙의 멤버 속성 **state variables**으로 ```int```, ```address```, ```mapping```을 선언하고 있다.
* ```event PrintLog``` 이벤트를 선언한다.
* 함수 function ```constructor```, ```deposit()```, ```queryBalance()``` 또는 ```modifier```를 가질 수 있다.
* Child에서 부모의 함수를 상속받아 ```add()```, ```getCounter()```를 사용할 수 있다.

```python
[파일명: src/SimpleChild.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Parent {
    //state variables
    address owner; //as of 0.8.0 no need to be address payable owner;
    uint private counter;
    
    //constructor. no need for constructor to be public as of 0.7.0
    constructor() {
        owner = msg.sender;
        counter = 0;
    }
    //functions
    function add() public { counter++; }
    function getCounter() public view returns(uint) { return counter; }
}

contract SimpleChild is Parent {
    //state variables
    string nickName;
    mapping(address => uint) private balances;
    //event
    event PrintLog(address, uint);
    
    //constructor
    constructor() {}  // no public as in constructor() public {}  
    //functions
    function setNickName(string memory s) public { nickName = s; }
    function getNickName() public view returns(string memory) { return nickName; }
    function deposit() public payable {
        balances[msg.sender] += msg.value;
        emit PrintLog(msg.sender, msg.value);
    }
    function queryBalance() public view returns (uint) {
        return balances[msg.sender];
    }
    //access non-private members of the parent
    function kill() public {
        if (msg.sender == owner) selfdestruct(payable(owner)); //0.6.x selfdestruct(owner)
    }
}
```

컴파일까지 해보자. 설치된 컴파일러 명령은 ```solc```일 수도 아래와 같은 ```solc-windows.exe```일 수도 있으니 분별하여 사용하자.
```python
pjt_dir> solc-windows.exe src/SimpleChild.sol
```

## 1.3 프로그래밍 스타일

Solidity 프로그래밍은 객체지향 자바와 크게 다르지 않다.
* 자바에서는 파일명과 클래스명이 일치해야 한다. Solidity에서 파일명은 컨트랙명과 달라도 문제가 되지 않지만 일치하도록 한다.
* import문은 프로그램 위, Pragma 다음 줄에 적어준다.
* 컨트랙은 새로운 줄에 적어준다.
* 컨트랙, Event, enum, Struct: 대문자로 시작하고, camel case 스타일로 단어의 첫글자는 대문자로 적어준다
* 함수, 함수인자, 변수, modifier: 소문자로 시작하고 camel case로 적어준다.
* 상수는 모두 대문자, underscore (DATE_OF_BIRTH)로 연결해서 적어준다.
* 컨트랙 간에는 2줄 띄어쓰기를 해준다.
* 들여쓰기를 탭을 사용하지 않고 공백 4칸을 넣어준다.
* 배열은 ```int[] x;```이라고 적어준다 (```int [] x``` 또는 ```int x[]```가 아니라)
* 문자열은 쌍따옴표를 해준다.
* 함수는 새로운 줄에 적어준다
* 한 줄은 최대 79 문자를 넘지 않게 한다.
* 괄호에서는 한 칸 띄어쓰기를 하지 않는다. ```if (x == 1)``` (```if ( x == 1 )```이 아니라)
* 연산자 앞 뒤 1칸을 넣어준다.
* 코드 블록을 적을 때, 바로 이어서 중괄호를 연결한다.
```python
if (x == 1) {
}
```

## 1.4 예약어

Solidity에서 제공하는 예약 명령어를 알파벳 순으로 나열해보자. 명령어는 프로그램을 작성하면서 배워나가기로 하자. 이런 명령어는 컨트랙이나 변수를 명명할 때 사용하지 않도록 주의한다.

"abstract", "after", "alias", "apply", "auto", "case", "catch", "copyof", "default", "define", "final", "immutable", "implements", "in", "inline", "let", "macro", "match", "mutable", "null", "of", "override", "partial", "promise", "reference", "relocatable", "sealed", "sizeof", "static", "supports", "switch", "try", "type", "typedef", "typeof", "unchecked"

# 2. 데이터 타입

Solidity 언어는 블록체인에서 실행이 되는 까닭에 **저장공간을 효율적**으로 사용해야 한다. 저장공간을 많이 사용할수록 비용이 발생하게 된다.

다른 언어에서 지원하는 데이터타입을 큰 차이 없이 사용할 수 있지만 **데이터타입**이 자세하게 나누어져서 불필요한 낭비를 줄이고 있고, **배열을 검색**하거나 **반복문**을 사용할 때도 유의해야 한다.

그리고 소수점은 아직 지원되지 않아 float, double과 같은 자료형이 없다. 또한 암호화폐를 가지고 있어, 계정주소 타입이 있다는 점이 특별나다.

데이터타잎 | 설명
----------|----------
bool | uint8, 0 and 1의 값을 가진다
uint<M> | unsigned integer, M은 8비트 단위로 256비트까지 가능하다 0 < M <= 256, M % 8 == 0
int<M> | signed integer, M은 8비트 단위로 256비트까지 가능하다 0 < M <= 256, M % 8 == 0
address | 주소. 크기가 20바이트이므로 **uint160** 이다.
uint, int | 숫자가 붙지 않은 경우의 타입으로 uint256, int256를 의미
bytes<M> | 바이너리 타잎, M은 32바이트까지 가능하다 0 < M <= 32. **```bytes``` (크기가 없는)는 value type이 아니다**.
string | UTF-8 문자열, **value type이 아니다**.

## 2.1 불린

Boolean은 true, false 둘 중의 하나 값을 가지는 데이터타입이다. 비교연산자 (==, !=, !, >, <), 논리연산자 (&& ||)의 결과가 true, false 와 같이 boolean이 된다.

```python
bool isMarried=true;
```

## 2.2 정수
```int```는 정수를 가지는 타입이고 ```signed```, ```unsigned```로 구분할 수 있다.
* ```signed int```는 양수, 음수 모두 허용되며
* ```unsigned int```는 양수만 사용할 수 있다.

그 크기는 8, 16, 24, 32 이런 식으로 8바이트 단위로 증가하고 **최대 32**바이트까지 사용할 수 있다.
크기를 적지 않으면 256비트로 간주한다. **기본 값 default value은 0**이다.
```+, -, *, /, %, **, ++, --, +=, -=```와 같은 산술연산자를 사용할 수 있다.

```python
int256 x=1;
```

## 2.3 소수

fixed, unfixed는 각 각 signed, unsigned 소수점 숫자를 말한다. 선언은 할 수 있지만, 값을 할당하면 ```UnimplementedFeatureError``` 오류가 발생한다.

## 실습문제: int, boolean 사용

줄 | 설명
-----|-----
1 | 컴파일러 0.8의 최신 버전으로 설정. 다음 명령문 ```^0.8.0```은 메이저버전 8, 마이너버전 0을 의미한다. 맨 앞 ```^``` caret는 메이저버전으로 시작하는 최신버전을 선택한다는 의미이다. 그러나 가급적 정확한 버전을 적는 편이 더 좋다.
7 | fixed 지원여부를 확인하여 보자.
8 | ```update()``` 함수에서 멤버변수의 값을 갱신할 수 있다.
12 | 매개변수 ```int```를 넘겨준다. ```int```와 ```uint```의 형변환이 필요하다.
22 | ```assert()```가 오류이면 ```Exception```이 발생하고, 다음 줄이 실행이 되지 않는다. 즉 ```x==1```인 경우 ```true```가 반환된다.

```python
[파일명: src/IntBool.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract IntBoolTest {
    bool married = true;
    uint256 xAge = 22;
    uint256 yAge = 25;
    //fixed phi; // = 3.14;
    function update() public {
        xAge = yAge;
        yAge = 33;
    }
    function setXAge(int age) public {
        xAge = uint(age);  //type conversion
    }
    function getXAge() public view returns(uint) {
        return xAge;
    }
    function getYAge() public view returns(uint) {
        return yAge;
    }
    function testInt() public view returns(bool) {
        assert(xAge>=20 && yAge>=20);
        return true;
    }
    function isMarried() public view returns(bool) {
        return married;
    }
}
```

컴파일해서 오류가 있는지 확인해보자.
```python
pjt_dir> solc-windows.exe src/IntBool.sol
```

## 2.4 고정크기 바이트 문자열

고정크기의 바이트문자열은 ```bytes1``` ~ ```bytes32``` 사이의 어떤 크기로 선언할 수 있다. **최대 32바이트까지 1바이트 단위**로 크기를 늘릴 수 있다는 의미이다.

byte1, bytes32는 hex 값 뿐만 아니라 정수, 문자, 문자열을 가질 수 있다. ```byte1```은 1 바이트, 8비트의 길이를 가진다. ```byte``` (bytes가 아님)는 0.8.0에서 제거되었고, bytes1을 대신 사용한다.

**bytes literals는 따옴표 없이 0x를 붙여서** 0xFF와 같다. 문자열을 입력할 수 있지만, hex 값이 저장된다.

```python
bytes1 x= 0xFF;
bytes23 place1 = "7 hongji-dong jongro-gu";
bytes8 place2 = "7 hongji"; //결과값은 0x3720686f6e676a69
```

각 문자의 16진수는 다음과 같다.

문자 | ascii | hex
-----|-----|-----
7 | 55 | 37
space | 32 | 20
h | 104 | 68
o | 111 | 6f
n | 110 | 6e
g | 103 | 67
j | 106 | 6a
i | 105 | 69

## 2.5 변동크기 바이트 문자열

```bytes```는 고정크기의 바이트문자열과 혼돈하기 쉬운데, 여기서는 크기를 뜻하는 **숫자가 없는** 경우를 말한다. 또는 ```string```은 그 크기가 정해져 있지 않아서 가변적이다. 길이 제한이 없는 경우 사용한다. string은 UTF8 형식으로 문자열을 저장한다.

크기를 알려주는 ```bytes.length```를 사용할 수 있다. ```string```의 ```length```는 ```bytes(string).length```로 **형변환을 ```bytes```로 한 후**에야만 가능하다.

배열의 인덱스를 넣어 ```bytes(s)[2]```로 3번째 값을 읽을 수 있다.

```python
string s = "hello";
```

상수는 ```constant```로 표현한다.
```python
string constant name="jsl";  //상수는 constant로 표현한다.
```

문자열 합성을 하려면, abi.encodePacked()를 함수를 사용할 수 있다. 다른 언어에서는 쉽게 될 수 있는 "Hello " + "World" 이런 문자열 합성이 되지 않는다.

string(abi.encodePacked("Hello ", "World")) 이렇게 해야 한다. 또는 버전 0.8.10부터 bytes.concat() 또는 string.concat() 함수를 사용하면 된다.

```
string.concat("Hello ", "World");
```

## 실습문제: 문자열 bytes, string 사용

줄 | 설명
-----|-----
1 | 컴파일러 0.8의 최신 버전으로 설정
5 | ```bytes1```은 1 바이트 길이 (전에는 ```byte``` 타입이 있었지만, 0.8.0에서 제거)
6 | ```bytes23```에는 23글자를 입력. 한 글자를 추가하면 오류가 발생된다.
7 | ```bytes8```에는 8글자를 입력.
9 | ```myBytes```의 기본 값은 0이고, 3바이트 이므로 0x000000
10 | ```string```은 utf-8로 저장되므로, 16진수가 아니라 "jsl"
12 | ```byte1```을 ```byte```로 반환하는 경우 형변환이 필요없다.
17 | ```bytes1```로 선언된 b1을 반환한다. (0.8.0 이전까지 ```byte```와 ```bytes1```은 동일한 의미)
18 | ```bytes23```과 같이 정해진 경우, memory를 사용하지 않는다. 그러나 bytes와 같은 ref type은 memory를 사용한다.
22 | ```bytes```는 동적배열이다. ```getBytes()```는 3글자로 설정된 동적배열을 memory로 반환한다. 그 반환 값이 16진수로 출력.
24~27 | ```bytes23.length```는 23이 반환된다.
27~31 | 가변크기 ```bytes.length```는 23이 반환된다. 같은 값을 가지고 있는 ```place23```과 같은 크기.
34-36 | 매개변수는 hex로 넣어준다. a의 hex는 61이다. bytes1은 "0x61",  bytes2 "0x6161"로 입력한다. 2바이트 넘거나, utf-8 문자열을 넣으면 오류이다.
37-39 | ```myBytes```는 3바이트로 할당되었으므로 크기에 맞추어 넣어준다. UTF-8로 넣어주어도 되고 **"smu"로 넣어주면 0x736d75**로 hex값으로 저장된다
40~44 | <string>.length는 오류. 반드시 ```string```을 ```bytes```로 변환해서 ```length```를 구한다.
46~49 | 한글을 입력하려면 utf8을 사용한다.

```python
[파일명: src/ByteStringTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract ByteStringTest {
    //byte b = 0xFF; //use bytes1 as of 0.8.0
    bytes1 b1 = 0xFF;
    bytes2 b2 = 0xFFAA;
    bytes8 place8 = "7 hongji"; //8 characters long
    bytes23 place23 = "7 hongji-dong jongro-gu";
    bytes place = "7 hongji-dong jongro-gu Seoul"; //variable length
    bytes myBytes = new bytes(3);  //0x000000
    string constant name = "jsl"; //utf-8 string "jsl"
    function getB1() public view returns(bytes1) {
        return b1;  //bytes1, so no casting required
    }
    function getB2() public view returns(bytes2) {
        return b2;
    }
    function getB23() public view returns(bytes23) {
        return place23;  //fixed size, value type (no memory)
    }
    /**@return hex bytes. reference type should be set as memory*/
    function getBytes() public view returns(bytes memory) {
        return myBytes;  //smu in hex 0x736d75
    }
    function getLengOfBytes23 () view public returns(uint) {
        return place23.length;  // returns 23
    }
    function getLenOfBytes() pure public returns(uint) {
        bytes memory bm = "7 hongji-dong jongro-gu";
        return bm.length;        // returns 23
    }
    //need the arg in hex e.g. bytes1 0x61 bytes2 0x6161
    //a 61, b 62, ... , y 79
    //try invalid type, e.g. bytes2 0x61 or 0x616161
    function setB2(bytes2 _b2) public {
        b2 = _b2;
    }
    function setBytes() public {
        myBytes = "smu";
    }
    function getLenOfString() pure public returns(uint) {
        string memory nameLocal = "jslLocal";
        //return nameLocal.length;  //error, casting required
        return bytes(nameLocal).length;
    }
    function getString() pure public returns(string memory) {
    //function getString() pure public returns(bytes memory) {
        string memory s = "\xec\x95\x88\xeb\x85\x95"; //"한글";
        //bytes memory s = "\xec\x95\x88\xeb\x85\x95"; //"한글";
        return s;
    }
}
```

```python
pjt_dir> solc-windows.exe src/ByteStringTest.sol
```

## 2.6 struct

struct은 서로 관련있는 데이터를 그에 맞는 데이터타잎으로 구성하여 묶어서 사용할 수 있다. CapWords 스타일로 첫글자는 대문자로 적어준다.

Student를 uint, string, bool을 가지게 만들면 다음과 같다.

```python
struct Student {
        uint num;
        string name;
        bool isEnrolled; 
}
Student s1=Student(201911111,"jslim",true); 
```

## 실습문제: struct 사용

줄 | 설명
-----|-----
1 | 컴파일러 0.8의 최신 버전으로 설정
12 | ```uint```, ```bool```은 value 타입이고 ```string```은 reference 타입이라 ```memory``` 공간을 사용한다.
17~22 | ```struct```은 반환할 수 없다. 항목 하나 하나를 반납한다. **web3에서 반환할 경우에는 문자열로 변환해서 toString() 출력**할 수 있다.
24~26 | 새로운 ```Student```를 지역변수 ```s3```로 만들고, 이름만 반환한다. 주의할 점은 s3를 ```storage```, ```memory``` 선택할 수 있으나 오른쪽에서 ```Student```를 지역에서 생성하고 있으므로 ```memory```를 사용하도록 적는다.

```python
[파일명: src/StructTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract StructTest {
    struct Student {
        uint num;
        string name;
        bool isEnrolled;
    }
    Student s1=Student(201911111,"jslim",true);
    Student s2;
    //memory only for string type
    //201711111,"kim",false
    function setStudent2(uint n, string memory sn, bool e) public {
        s2.num = n;
        s2.name = sn;
        s2.isEnrolled = e;
    }
   function getStudent1() public view returns(uint, string memory, bool){
       return (s1.num, s1.name, s1.isEnrolled);
   }
   function getStudent2() public view returns(uint, string memory, bool){
       return (s2.num, s2.name, s2.isEnrolled);
   }
   function getStudentName() pure public returns(string memory) {
       //the right is locally created, so memory (not storage) is declared
       Student memory s3 = Student(201911112, "jsl3", true);
       return s3.name;
   }
}
```

프로그램을 컴파일 해보자.
```python
pjt_dir> solc-windows.exe src/StructTest.sol
```

## 2.7 열거형

```enum```은 요소로 구성된 데이터 타잎을 정의할 경우 사용한다.
예를 들면, 요일과 같이 월, 화, 수, 목, 금, 토를 '집합'을 구성할 경우 ```enum```을 사용한다. 
요소는 **0부터 시작하여 정수 값**을 가진다.
**비교**에 사용하며, 존재하지 않는 선택으로 인한 오류를 막을 수 있어 유용하다.
아래는 성별을 남, 녀로 구성한 ```enum```이다. 앞 글자는 대문자로 적어준다.

```python
enum Gender {male, female}
```

요일을 ```enum```으로 구성하면 다음과 같다.
```python
enum Day {SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY}
```

자바스크립트나 다른 언어에서 Solidity의 함수를 호출하는 경우가 있다고 하자.
아래 코드에서 보듯이, 요일 enum을 매개변수로 넘겨주어야 하는 Solidity 함수를 자바스크립트에서 호출한다고 하자.
자바스크립트 코드는 다른 언어이고 ```Day``` 타입을 맞추어 줄 수 없다면 정수를 적어주면 된다.
요일 enum은 MONDAY~SUNDAY 사이의 상수 값이므로, 0~6 사이의 정수로 적어줄 수 있고, 그 외 값은 오류가 발생한다.

## 실습문제: enum 사용

줄 | 설명
-----|-----
1 | 컴파일러 0.8의 최신 버전으로 설정
4 | **```enum```이름.변수명**으로 사용하고, 요소는 index로 읽거나 출력한다.
8 | ```getMyDay()```는 정수 값을 반환한다.
11 | 호출하면서 매개변수를 넘겨줄 때, 0~6의 값을 받을 수 있다.
17 | 요소에 해당하는 값을 index로 설정할 수 있다.

```python
[파일명:  src/EnumTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract EnumTest {
    enum Day {MONDAY,TUESDAY,WEDNESDAY,THURSDAY,FRIDAY,SATURDAY,SUNDAY}
    Day myDay = Day.FRIDAY; //index int4
    
    /* @return Day  returning index*/
    function getMyDay() public view returns(Day) {
        return myDay;   //index
    }
    /* @param d  ok to pass an integer (uint8)*/
    function setMyDay(Day d) public {
        myDay = d;
    }
    //uint is converted to uint8, which is default
    function setMyDayInt(uint d) public {
        myDay = Day(d);
    }
}
```

enum이 적힌 코드를 컴파일 해보자.
```python
pjt_dir> solc-windows.exe src/EnumTest.sol
```

## 2.8 배열

1) 고정배열은 그 크기가 사전에 정해지게 된다.
2) 반면에 동적 배열은 그 크기가 사전에 정해지지 않는다.
배열의 크기는 고정, 동적배열 모두 ```length```로 알 수 있다.
동적배열에 대해서 요소를 추가하려면 ```push```, 제거하기 위해서는 ```pop``` 함수를 사용할 수 있다.

* ```length```: 배열의 길이, 즉 몇 개의 요소가 포함되어 있는지 출력한다.
* ```push```: 배열에 요소를 추가하고, 동적배열, ```bytes```에 사용할 수 있고 ```string```에는 쓰지 못한다.
* ```pop```: 배열에서 요소를 제거하는 함수이다. 동적배열, ```bytes```에 사용할 수 있고 ```string```에는 쓰지 못한다.

## 실습문제: 배열 사용

```python
[파일명: src/ArrayTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract ArrayTest {
    uint[3] ages = [15, 25, 35];
    int[] marks; //dynamic ArrayTest
    
    /* @param index  array index
       @param val    value at the index*/
    function updateAges(uint index, uint val) public {
        if(index>=0 && index <=2)
            ages[index] = val;
    }
    function initMarks() public {
        marks = new int[](5);   // default 0
    }
    function appendMark(int mark) public {
        marks.push(mark);
    }
    function popMark() public {
        marks.pop();
    }
    /* @return dynamic array, so memory is used*/
    function getMarks() public view returns(int[] memory) {
        return marks;
    }
    /* @return fixed array, so memory is used*/
    function getAges() public view returns(uint[3] memory) {
        return ages;
    }
    function getLenOfArr() pure public returns(uint) {
        //memory is used because locally created array is assigned
        uint8[3] memory intArr = [0, 1, 2];
        return intArr.length;
    }
}
```

위 프로그램을 컴파일 해보자.
```python
pjt_dir> solc-windows.exe src/ArrayTest.sol
```

## 2.9 계정주소

address는 계정의 주소로서 **20 바이트** 길이를 가진다.
**address 객체**로서 잔고를 조회하거나, 입출금을 할 수 있다.
입금이 되는 계정은 ```address payable```로 선언해야 하지만
버전 0.8.0부터 payable로 선언할 필요가 없다.

### 잔고조회

계정의 잔액 wei를 조회할 수 있고, 반드시 자신이 아니더라도 누구나 읽을 수 있다.
현재 컨트랙의 잔고는 ```address(this).balance```로 구할 수 있다.
```this.balance```와 같이 하면 ```this```는 현재 인스턴스를 의미하기 때문에 잔고를 구할 수 없다.
Solidity 0.5.0부터 ```this.balance```는 사용이 금지되었고, ```address(this).balance```를 사용해야 한다.
this는 contract 자신의 주소를 말한다.

```python
this.myFunction()
```

### 송금

**입금은 어느 계정으로든 가능**하다.
또한 **출금**을 할 수 있다.
은행거래에서도 타인의 계좌에서 임의로 인출이 허용되지 않는 것처럼
**출금은 자신의 주소에서만 가능**하다.
자신의 주소는 스스로 unlock, sign을 할 수 있는 권한이 있고, 그에 따라 출금이 가능한 것이 당연하다.
그러나 임의의 주소에서 누군가 출금을 한다는 것은 비상식적이다.
금액의 단위는 기본 값이 Wei이고, ether 등 다른 단위는 명시하면 된다.

0.5.0 버전부터 주소에서 송금하려면 ```payable```로 선언해야 한다.

차이점 | send() | transfer() |  call.value()
-----|-----|-----|-----
gas 수정가능 | N | N | Y
gas 한도 | 2300 | 2300 | 모든 가용 gas를 넘겨주거나, 일정 gas를 정할 수 있다.
실패하면  | false | throw 예외발생 | false

#### transfer()는 실패하면 원복된다

transfer 뿐만 아니라 어떤 송금이나 실패하는 경우는:
* 송신측에 그만한 잔고가 없거나
* 수신측이 수령을 거절하는 경우이다.
실패하면 throw 예외를 발생시키며, 호출자에게 환급되며 아무런 송금도 없는 것처럼 원래의 상태로 복귀된다 (revert). 따라서 send()를 사용하는 것보다 예외처리가 용이하다.

성공하는 경우, 수신측에서 receive() 또는 fallback() 함수를 통해 수신된다.
또한 수신측에 코드실행에 필요한 **2,300 gas**가 지급된다.

```python
address.transfer(amount) //적은 주소가 수신측이 되고, amount만큼 address로 송금된다.
```

송금이 실패해서 ```throw```가 발생하면:
* 송신자에게 **송금액이 반환**되어 원위치로 돌아가게 된다.
* gas는 당연히 소비되었으므로 반환되지 않는다. gas 잔액이 남아도 반환되지 않고 모두 소비된 것으로 한다.
* 여러 계정에 송금하는 중 throw가 발생하면, 나머지 송금계정에는 송금이 아예 발생하지 않을 수 있다.

#### send()은 성공여부를 반환하므로 처리 로직이 필요하다

send() 역시 transfer와 같이 수신계정으로 송금을 하게 된다.
그러나 송금이 **성공 또는 실패했는지 그 결과를 boolean으로 반환**할 뿐이다.
send가 실패하는 경우는 원래의 상태로 복귀하지 않는다는 점에서 transfer와 차이가 있다.

따라서 송신측에서 실패하는 경우 어떻게 처리되어야 할지,
금액을 출금계정으로 반환하거나, 적절한 처리 로직을 넣어 주어야 한다.
송금이 실패하는 경우는 여러 이유가 있을 수 있겠다.
```call statck error``` 또는 ```out of gas error``` 등의 이유로 실패하는 경우에도 예외 **Exception이 발생하지 않고 false를 반환**하게 된다.
따라서 require, revert, assert와 같은 예외처리를 해주어야 한다 (예외처리는 나중에 배우게 된다)
Solidity 0.4.10 이전에는 ```if(!address.send(amount)) throw```와 같이 했으나,
```require(address.send(amount))```로 코딩하도록 한다.

성공하는 경우, 마찬가지로 수신측에서 receive() 또는 fallback() 함수를 통해 수신된다.
역시 transfer와 마찬가지로 수신측에 코드실행에 필요한 2,300 gas가 지급된다.

* 자신의 잔고가 10 wei이상이면, receiver 주소에 송금하는 코드 예

```python
address _receiver = 0x778ea91cb0d0879c22ca20c5aea6fbf8cbeed480;
if(myAddress.balance>=10) require(_receiver.send(10));
```

#### call() 함수는 gas를 명시할 수 있다

이 함수는 addr.send(x)와 같이 송금하는 기능을 할 수 있다.
그러나 call, callcode, delegatecall은 low-level이므로 주의해서 사용한다.
전과 달리 send, transfer 보다 call이 보다 안전하다고 쓰일 수 있다고 주장한다
(참조: https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/)

앞서 수신측에 지급되는 gas가 2,300에 불과하여 실제 gas에 비해 적다는 것이다.
따라서 악의적인 수신자는 자신의 함수를 재호출하여 송신자의 gas를 소진시킬 우려가 있다는 점이다. 
(이른바 re-entrancy attack)

call함수는 gas를 정해서 처리하는 경우에 사용할 수 있다.
call() 함수는 명시하지 않으면 gas가 실비로 지급된다.
따라서 받는 측에 gas를 사용할 수 있게 하기 때문에 악의적으로 사용될 수 있다.

0.5.7부터 call()와 더불어 delegatecall(), staticcall() 함수는 반환을 한다.
반환은 (bool, bytes memory) 이런 형식으로 반환값을 2개 받는다.
* bool success: 함수 호출이 성공인지
* bytes memory data: 함수가 실제로 반환하는 결과
이전 코딩은 <address>.call.value(20) 이렇게 했지만, 다음과 같이 하자.

```python
(bool success, ) = <address>.call{value:20}(""); //20 Wei를 송금한다. 괄호가 없으면 값을 설정했다는 의미.
require(success, "transfer call failed.");
```

gas를 적을 수도 있다. gas 2, 송금 11111 Wei, 이 경우 gas가 낮으면 실패할 수 있다.
```python
(bool success, ) = <address>.call{gas:2, value:11111}(""); //0.6.4이전에는 call.value().gas()()로 적음
require(success, "transfer call failed.");
```

이미 앞서 call함수를 사용한 적이 있는데, function selector를 사용해서 호출하였다.
송금할 때는 함수를 호출하지 않고, 즉 function selector 없이 호출하고 있다는 점에 주의한다.
```
(bool success, bytes memory data) = <address>.call{gas:10000, value: 20}(bytes4(bytes32(sha3("baz(uint32,bool)")));
```

## 실습문제: 송금

송금하는 프로그램을 작성해보자. 실행할 때는 다음 순서에 따라 해보자.
* (1) 수신측 주소를 설정해야 한다.
setReceiver()에 주소를 넣어 주어야 하는데, address literals은 따옴표 없이 넣어준다. 따옴표를 하면 문자열 타입으로 인식하게 된다. ```deposit()```을 호출할 때는 ```value```에 금액을 넣어주어야 한다.
* (2) 컨트랙에서 출금되므로 입금을 해놓는다.
send(), transfer, callValue()를 성공적으로 실행하기 위해서는 사전에 ```deposit()```을 호출해서 충분한 금액을 입금해야 한다.
* (3) send(), transfer, callValue()를 하나씩 실행해본다.
    * 송금하면서 coinbase의 잔고의 변화가 있다.
    * 수신측에게 gas 2,300의 지급이 없는 것으로 보인다.
    * call() 함수 내 2번째 송금의 경우, gas비를 적게 (10) 설정해도 지급이 된다.
![alt text](figures/9_depositSendValue.png "put amount in value and address without quotes")

```python
[파일명: src/AddressTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract AddressTest {
    address owner;
    address payable receiver; //address receiver;
    uint balanceOfOwner;
    constructor() { // NO! constructor() public {
        owner=msg.sender;
        //myBalance = msg.sender.balance;
        balanceOfOwner = owner.balance;
    }
    function deposit() payable public {
    }
    /* @param addr  set as payable because it will get some gwei*/
    function setReceiver(address payable addr) public {
        receiver=addr;
    }
    function getReceiver() view public returns(address) {
        return receiver;
    }
    function getBalanceOfThis() public view returns(uint) {
        return address(this).balance;  //balance of contract
    }
    function getBalanceOfOwner() public view returns(uint) {
        return owner.balance;
    }
    function getBalanceOfReceiver() public view returns(uint) {
        return receiver.balance;
    }
    function send() public payable {
        require(receiver.send(111)); //send 111 gwei to xAddress
    }
    function transfer() public payable {
        //if !(receiver.transfer(address(this).balance))
        receiver.transfer(11111);
    }
    function callValue() public payable {
        //receiver.call.value(11111)(""); //deprecated
        //receiver.call{value: 11111}("");  //Warning: Return value of low-level calls not used.
        //(bool success, bytes memory data) = receiver.call{value: 11111}(""); //waring persists
        (bool success, ) = receiver.call{value: 11111}("");
        require(success, "transfer call failed.");
        //receiver.call.gas(10).value(11111)("");  //warning
        //receiver.call{gas: 10, value: 11111}(""); //warning
        (success, ) = receiver.call{gas: 10, value: 11111}("");
        require(success, "transfer call failed with gas 10.");
    }
}
```

다음과 같이 컴파일 해보자.
```python
pjt_dir> solc-windows.exe src/AddressTest.sol
```

## 2.10 리터럴

변수가 값을 가지려면 할당연산자 다음에 literals를 적어주어야 한다.
즉 변수는 명칭이고, literals는 변수에 값을 할당하게 된다.
* 정수는 소수점 없는 양수, 음수이고 1, 10, -1, -10 등을 예를 들 수 있다. 또는 16진수 0x14 (십진수 20)도 사용할 수 있다.
* string은 따옴표, 하나이든 쌍따옴표이든 상관없이 "jsl", 'hello' 등이 예가 될 수 있다.
* 주소는 객체로서 20바이트 16진수로 0x로 시작하는 20바이트, 40자리수이다. 예를 들면, 0x307A2Db4424E0F651cfa1Dd7C6C418CDf8f5675b
* bytes는 따옴표 없이 0x로 시작하는 16진수이다. 예: 0x3720686f6e676a69
* 소수점은 1.1 3.14 등이 예이다. 아직 구현되지 않아 값을 할당할 수 없다.
* 배열은 다른 언어와 다르지 않게 값을 할당할 수 있다. 예: [15, 25, 35]

# 3. 데이터의 저장

## 3.1 타입 구분

데이터 타입은 (1) 값을 가지고 있는 value type, (2) 참조 타입 reference type으로 크게 구분할 수 있다.

### 값으로 저장되는 타입
데이터 그 **자체의 값**으로 저장되는 타입이다. 다른 변수에 할당되거나 함수인자로 전달될 경우, 값의 복사본이 되어 전달되어서 쓰이는 **pass by value**의 특성을 가진다.
```bool```, ```int/uint```, ```address```, ```bytes (1~32 바이트)```, ```enum```이 해당된다.
```int```, ```bytes (1~32 바이트)```는 최대 **32바이트**로 그 값을 저장할 수 있다.

### 참조가 저장되는 타입
**32바이트를 초과할 수 있는 크기의 데이터**를 말하며, 그 저장주소를 통해 사용하는 타입이다.
변수에 할당되거나 매개변수로 전달될 경우, 그 저장주소가 복사되어 쓰이기 때문에 원본 데이터가 수정되면 따라서 변경된다.
참조형 타입 reference type은 ```bytes```, ```array```, ```string```, ```Struct```, ```mapping```과 같이 실제 값이 저장되어 있는 주소를 가리키는 참조를 말한다. 이들 데이터는 32바이트 이상의 크기를 가질 수 있고, 저장장소를 정의해 줄 수 있다.

## 4.2 저장장소 구분

Solidity는 데이터를 저장하는 장소를 다음과 같이 구분한다.

### storage는 하드디스크에 저장이 되는 것과 같다.
하드디스크에 저장이 되는 것처럼, 일단 ```storage```에 저장이 되면 영원히 저장된다.
컨트랙의 상태변수는 기본 값이 ```storage```이며, 모든 함수에서 값을 볼 수 있다.
따라서 함수 호출마다 사용되고 없어지지 않고 지속된다.
32바이트 길이의 key-value 형식으로 저장되고 계속 유지되므로, 비용이 다른 저장타입에 비해 비싸다.
```
contract IntBoolTest {
    bool married = true;  storage에 저장된다. bool storage married = true;라고 하지 않는다.
    uint256 xAge = 22;    storage에 저장된다. uint256 storage xAge = 22;라고 하지 않는다.
}  
```

멤버변수 (또는 상태변수라고 함)에 지역변수를 저장하면, 복사본이 저장된다. 복사한 후 지역변수의 값이 변경된다고 하여도, 멤버변수는 복사상태의 값이 유지된다. 참조값이라 할지라도 참조가 복사되지 않기 때문에 멤버변수의 값이 유지된다.

```
pragma solidity ^0.8.0;
  
contract CopyOfStorage {
      uint stateInt = 111; //storage
      string stateStr = "hello";
    
    function copyLocalToStateVar() public {  
        uint localInt = 123;
        string memory localStr = "hello world";
        stateInt = localInt; 복사 후 123을 456으로 변경하여도 123으로 유지
        stateStr = localStr; 참조타입인 경우, 복사 후 hello world를 변경하여도 그대로 유지
        localInt = 456;
        localStr = "hello outer world";
    }
    function getStateInt() public view returns(uint) { return stateInt; } 123이 출력된다.
    function getStateStr() public view returns(string memory ) { return stateStr; } hello world 출력.
}
```

### 메모리는 RAM에 저장이 되는 것과 같다.
```memory```는 단기간 저장하는 경우에 사용된다. 함수 내에서 그 시간 동안 사용되고, 함수를 벗어나면 지워진다. **value type**은 지역변수, 함수의 인자 또는 반환 값도 memory에 저장되지만 memory 수식어를 붙여주지 않는다.

단, struct, array, mapping과 같은 reference type은 지역변수라 하더라도 memory 수식어를 붙여준다.
적어주지 않으면 기본으로 default인 storage에 저장되기 때문에 오류가 발생한다. 또한 storage라고 명시적으로 적어 주어도 안된다.

```
function getLenOfString() pure public returns(uint) { 
    int i;  지역변수 i는 메모리에 저장이 된다.
    string memory nameLocal="jslLocal"; 저장구분을 안 적거나, storage라고 하면 오류
}
```

### 스택은 지역변수의 값을 저장한다.
```stack```은 지역변수 (참조형 타입 제외)의 값을 저장한다. 1,024 수준까지 저장할 수 있고, 그 이상은 예외처리 된다. 비용은 memory와 동일하다.

### callData는 함수의 매개변수를 저장한다.
```calldata```는 **외부**에서 함수를 호출할 때 (extern으로 선언된 함수), 사용하는 함수시그니처와 매개변수를 calldata로 저장한다. 앞서 ABI 명세에서 설명하였듯이 1) function selector부호값과 2) 인자부호값를 합쳐서 data 필드에 적은 "0xcdcd...00001"을 말한다.

```
web3.eth.sendTransaction({
    from: ...,
    data: "0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001",
    gas: ...}
)
```

요약하면,
* 상태변수는 storage에 저장한다.
* value 타입 지역변수, 함수인자/반환은 memory를 기본 default로 사용한다.
단, 함수에서 사용되는 참조타입 (struct, array, mapping, string)은 기본이 storage이고, memory를 사용하려면 선언이 필요하다.

# 5. 가시성

지역변수는 자신이 선언된 함수에 국한되어 사용할 수 있다.
반면에 멤버변수 state varaiable은 사용할 수 있는 범위를 제어할 수 있다.

* 가시성을 적어주지 않으면 **기본 값은 ```internal```**로 정의된다.
```internal```은 블록체인 내부에서만 사용할 수 있고, 상속을 하면 자식이 물려받을 수 있다는 의미이다.
library의 함수는 internal로 선언할 수 있다. 

* ```external```은 ```internal```의 반대, 외부에서 호출하는 경우만 허용된다. 내부에서는 사용할 수 없지만, ```this.f()```로 호출하면 가능하다. fallback 함수는 외부에서만 호출할 수 있도록 extern으로 선언한다.

* 누구나 internal 또는 external 가리지 않고 모두 사용하려면 ```public```으로 선언하면 된다. 객체지향 ```public```과 동일한 의미이다.

* private은 블록체인 내부에서 컨트랙 자신만 사용할 수 있고, 객체지향 ```private```과 동일한 의미이다.

구분 | 설명 | default
-----|-----|-----
```public``` | 누구나 internal 또는 external 모두 사용 | no
```private``` |  컨트랙 자신만 사용 | no
```internal``` | 현재 컨트랙 내부에서만 또는 상속의 경우에 사용하는 경우. java의 ```protected```와 비슷한 의미 | yes
```external``` | 외부에서 호출하는 경우만 허용 | no


# 6. 연산자

다른 프로그래밍 언어에서 사용디는 연산자와 크게 다르지 않아서 어떤 것들이 있는지 구분해보자.

* 산술연산자: +  - ```*```  /  ```%``` (modulus) ++ -- ```**``` (exponentiation)
* 비교연산자: ```==``` ```!=``` ```>``` ```<``` ```>=``` ```<=```
* 논리연산자: && || ! (부정)
* 비트연산자: & | ^ ~
* 할당연산자: ```=``` ```+=``` ```-=``` ```*=``` ```/=``` ```%=``` (```x = x % y```는 ```x %= y```)
* 삼항연산자: ```?:```

## 실습문제: 주소 계정에서 입출금

앞서 geth 단말에서 account[0]에서 account[1]로 전송해보았다. 컨트랙에서 송금하는 프로그램을 개발해 보자.

은행을 경유하지 않고도 송금이 이루어지는 과정을 이해해 보자. 은행없이 송금하는 경우, 서로 어떻게 신뢰할 수 있는지 실패하는 경우에는 어떻게 복구할 수 있는지 생각해 보자.

### 단계 1: 컨트랙 개발

줄 | 설명
-----|-----
10 ~ 13 | 전송자의 ```msg.value```에 입력된 ether를 contract으로 입금하는 함수이다. 주의할 점은 함수인자 amount는 ether가 아니다. 실제 입금되는 금액은 ```msg.value```이다. 단 ```msg.value```와 amount가 동일해야 입금이 실행된다. 그리고 **```payable```**로 명시해 주어야 한다.
14 ~ 17 | ```owner``` 계정으로 컨트랙 자신의 잔고, ```address(this).balance```에서 ammount 만큼을 이체한다. ```msg.sender```는 매번 메시지를 전송할 때마다 변경되는 것이 아니라, 배포하는 시점의 전송자 즉 프로그램을 배포하는 권한을 가진 사람만이 (정확히 말하면 개인키) owner가 된다.
18 ~ 21 | 매개변수로 입력된 주소로 컨트랙 자신의 잔고에서 amount만큼 이체한다.
22 ~ 20 | 컨트랙 자신의 잔고, ```address(this).balance``` 및 onwer의 잔고를 조회한다.

```python
[파일명: src/MyBank.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract MyBank {
    address owner; //address payable owner;
    uint balance;
    constructor() { //constructor() public {
        owner = msg.sender;
        balance = address(this).balance;
    }
    function deposit(uint amount) public payable {
        require(msg.value == amount);
        balance += amount;
    }
    function withdraw(uint amount) public payable {
        balance -= amount;   // deduct before transfer
        payable(owner).transfer(amount); //owner.transfer(amount);
    }
    function transferTo(address payable receiver, uint amount) public payable {
        balance -= amount;   // deduct before transfer
        receiver.transfer(amount);
    }
    function getBalance() public view returns (uint) {
        return balance;
    }
    function getBalanceOfThis() public view returns (uint) {
        return address(this).balance;
    }
    function getBalanceOfOwner() public view returns (uint) {
        return owner.balance;
    }
}
```

### 단계 2: 컴파일

Solidity 버전 0.6대로 컴파일하고 배포할 경우 'Invalid opcode' 오류가 발생한다. 버전을 0.5대로 낮추어서 컴파일해보자.

```python
pjt_dir> solc-windows.exe --optimize --combined-json abi,bin src/MyBank.sol > src/MyBank.json
```

### 단계 3: 컨트랙 배포

```python
[파일명: src/MyBankDeployAbiBinFromFile.js]
var Web3 = require('web3');
var _abiBinJson = require('./MyBank.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));

contractName=Object.keys(_abiBinJson.contracts); // reading ['src/MyBank.sol:MyBank']
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

```python
pjt_dir> node src/MyBankDeployAbiBinFromFile.js

    - contract name:  [ 'src/MyBank.sol:MyBank' ]
    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0x2525f0be96865507f9e4f892c525e4c8b5385694248b8b412cea80cffb22d632
    ---> The contract deployed to: 0x81b928274EC5c4F366fEb3572252A327474C921d
```

### 단계 4: 사용

```deposit()```함수의 ```value:1111``` 필드를 채워주면 ```msg.value```로 전달이 된다.
이 때 함수의 인자도 동일하게 1111 Wei를 넣어준다.

```python
myBank.deposit(1111,{from:web3.eth.accounts[0],gas:80000,value:1111})
```

마이닝을 하고나면 잔고가 1111이 된다. 컨트랙의 ```deposit()``` 함수에서 잔고에 합산을 하지 않아도 된다.

자신의 잔고에서 1111만큼 빠져나간 것을 확인해보자.
```python
geth> eth.getBalance(eth.accounts[0]);
94999999999999998889
```

```python
[파일명: src/MyBankUse.js]
var Web3=require('web3');
var _abiBinJson = require('./MyBank.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/MyBank.sol:MyBank']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abi=_abiBinJson.contracts[contractName].abi;
_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

var myBank = new web3.eth.Contract(_abiArray,"0x81b928274EC5c4F366fEb3572252A327474C921d");

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Call from: " + accounts[0]);
    myBank.methods.getBalance().call().then(console.log);
    myBank.methods.deposit(1111).send({from:accounts[0],gas:80000,value:1111});
    myBank.methods.getBalance().call().then(console.log);
}

doIt()
```

send, call 함수가 섞여 있어서 프로그램을 나누어서 실행해야 하지만, 편의상 그대로 2회 실행을 해보자.
1111을 deposit() 입금하면 잔고에 반영이 되는 것을 볼 수 있다.
또한 withdraw() 출금하면 잔고가 0이 되면서 전송자의 잔고가 증가한다.


```python
pjt_dir> node src/MyBankUse.js

    Call from: 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    1111
    1111
```

# 7. 형변환

## 정수는 크기를 변환
형변환을 할 경우에는 데이터 타잎의 크기에 주의해야 한다.
예를 들어 int8에서 int로 변환할 경우 괄호 안에 써주면 된다.
이 경우 int는 256비트이므로 오류없이 변환이 된다.
```python
int8 i8=20;
uint i256=int(i8);
```

## 정수는 크기가 같은 byte32로 변환
uint는 bytes32로 변환할 수 있다.
```python
uint x=20;
bytes32(x);
```

bytes는 int로 형변환은 가능하지 않다.
예를 들어, bytes4는 uint32와 크기가 같고 형변환이 가능하다.
그러나 크기가 같더라도 int32로 형변환은 허용되지 않는다.
```
bytes4 b4=0x68656c6c;
int32(b4);  //크기가 같더라도  int로 형변환 오류
uint32(b4); //크기가 같은 uint로 형변환 가능
```

uint를 bytes로 바로 형변환을 할 수 없다.
uint는 256비트, 즉 32바이트라서 bytes32로 변환하고, 이를 bytes로 변환해야 한다.
이 경우 바이트마다 옮겨주어야 한다.

## bytes 간의 변환은 크기에 주의
```0x68656c6c```은 16진수, 8자리이다.
한 자리에는 0,1,2...D,E,F까지 $2^4$ 가지를 표현할 수 있고, 4비트가 소요된다.
이 데이터 bytes4를 bytes2로 변환해보자.
**오버플로우는 위 4자리부터 잘려 나가게 된다**.
따라서 ```0x6865```가 저장된다.

```python
bytes4 b4 = 0x68656c6c;
bytes2(b4);
```

## string을 bytes로 변환
```string```을 ```bytes```로 변환하는 경우:

```python
string s = "hello";
bytes b = bytes(s); //0x68656c6c6f
```

hex | char
-----|-----
68 | h
65 | e
6c | l
6c | l
6f | o

## payable 형변환

```address payable```에서 ```address```로의 형변환은 가능하지만, 반대는 허용되지 않는다.
```address``` -> ```uint160``` -> ```address payable```로 20바이트의 정수로 변환한 후 하도록 한다.

## 실습문제: 형변환

```python
[파일명: src/DataConversionTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract DataConversionTest {
    function convertInt8ToInt() pure public returns(int) {
        int8 i8=20;
        return int(i8);  //20
    }
    function convertIntToBytes32() pure public returns(bytes32) {
        uint x=20;
        //0x0000000000000000000000000000000000000000000000000000000000000014
        return bytes32(x);
    }
    function convertInt64ToBytes8() pure public returns(bytes8) {
        uint64 x=10;
        return bytes8(x); //0x000000000000000a
    }
    function convertIntToBytes() pure public returns(bytes memory) {
        // can not convert uint -> bytes
        // convert uint -> bytes32 -> bytes
        uint x = 20;
        bytes32 y = bytes32(x); //uint <=> uint256
        // can not convert from bytes32 -> bytes;
        bytes memory z = new bytes(32);
        for (uint i=0; i < 32; i++) {
            z[i] = y[i];
        }
        //0x0000000000000000000000000000000000000000000000000000000000000014
        return z;
    }
    function convertStringToBytes() pure public returns(bytes memory) {
        string memory s="hello";
        return bytes(s);  //0x68656c6c6f
    }
    function convertBytes4ToBytes2() pure public returns(bytes2) {
        bytes4 b4=0x68656c6c;
        return bytes2(b4);  //0x6865
    }
    function convertBytes4ToInt32() pure public returns(uint32) {    
        bytes4 b4=0x68656c6c;
        //1751477356 = 68656C6C hex = (6 × 16^7) + (8 × 16^6) + ... + (6 × 16^1) + (12 × 16^0) 
        return uint32(b4); //1751477356
    }
}
```

```python
pjt_dir> solc-windows.exe src/DataConversionTest.sol
```

# 8. 전역 변수

* 화폐 단위 ```wei```로 표현되며, 1 ```ether```는 $10^{18}$ wei이다.

```python
1 ether == 1000000000000000000 wei
1 ether == 1000 finney
```

* 시간 ```years```는 지원되지 않으며, ```seconds```로 표시된다. ```now```는 현재 시간, ```block.timestamp```와 동일하다.
```python
1 == 1 seconds, 1 days == 86400 seconds
```

```python
uint mytime=now;
```

* tx 컨트랙을 호출하는 트랜잭션 관련 정보
    * ```tx.origin``` 트랜잭션에 사인한 계정
    * ```tx.gasprice``` 트랜잭션 호출자가 명시한 gas price

* msg 컨트랙의 함수를 호출한 전송 관련 정보
    * ```msg.data```: call 데이터 (bytes 값으로 표현)
    * ```msg.gas```: gas 잔여분
    * ```msg.sender```: 현재 함수를 호출하는 측의 주소
    * ```msg.value```: 컨트랙에 지급되는 ether (단위는 wei)

* ```tx.origin```과 ```msg.sender``` 구분해보자. 거래가 호출자U1 -> 컨트랙C1 -> 컨트랙C2의 순서대로 완성이 될 경우:
    * C2에서 ```msg.sender```는 바로 직전의 호출자C1을 말한다. 즉 컨트랙도 ```msg.sender```가 될 수 있다.
    * 반면에 ```tx.origin```은 U1의 주소를 말한다. 최초로 메시지를 발생한 측을 말한다.
    * 아래 프로그램을 보자. 주소 0x69e9a...0c102에서 ```Order```의 ```getTxOriginMsgSender()``` 함수를 호출하면:
        * ```Customer```의 ```tx.origin``` 은 0x69e9a...0c102 (```Order``` 컨트랙 함수호출자인 ```msg.sender```의 주소와 동일)
        * ```Customer```의 ```msg.sender```는 0x0b878...E7cD4, 즉 ```Order``` 컨트랙 배포주소와 동일하다.


```python
[파일명: src/TxOriginMsgSenderTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract Customer {
    function getTxOriginMsgSender() view public returns(address, address) {
        return(tx.origin, msg.sender);
    }
}

contract Order {
    Customer c;
    constructor() { //constructor() public {
        c = new Customer();
    }
    function getTxOriginMsgSender() view public returns(address, address) {
        return c.getTxOriginMsgSender();
    }
}
```


```python
pjt_dir> solc-windows.exe src/TxOriginMsgSenderTest.sol
```

* block
    * ```block.coinbase```: 현재 블록 마이너의 주소
    * ```block.difficulty```: 현재 블록의 난이도
    * ```block.gaslimit```: 현재 블록의 gaslimit
    * ```block.number```: 현재 블록 수
    * ```block.blockhash```: 현재 블록 해쉬 값
    * ```block.timestamp```: 현재 블록 타임스탬프 epoch (1970년 1월 1일 0시) 이후 지나간 초, uint256.

## 실습문제: 전역변수

```python
[파일명: src/GlobalVarsTest.sol]
//SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity ^0.8.0;

contract GlobalVarsTest {
    function getEther() pure public returns(uint) {
        return 1 ether;  //1000000000000000000
    }
    function getDays() view public returns(uint,uint) {
        //require(block.timestamp==now);  //now deprecated
        return (block.timestamp, 1 days); // 1558816133 86400 ??web3? 1 days?
    }
    //public --> internal, to remove 'payable'
    function getMsgValue() view internal returns(uint) {
        return msg.value;
    }
    function getMsgSender() view public returns(address) {
        return msg.sender;
    }
    function getCoinbase() view public returns(address) {
        return block.coinbase;
    }
    function getBlockNumber() view public returns(uint) {
        return block.number;
    }
    function getBlockTimeStamp() view public returns(uint) {
        return block.timestamp;
    }
}
```

```python
pjt_dir> solc-windows.exe src/GlobalVarsTest.sol
```

# 9. Opcode 읽어보기

Solidity 뿐만 아니라 프로그래밍을 하면서 여러 종류의 오류를 만나게 된다. 이러한 오류는 크게 나누면 (1) 문법적으로 맞지 않거나 (Syntax Errors), (2) 논리적으로 부적합하거나 (Logic Errors), 또는 (3) 문법이나 논리적 오류가 없다고 하더라도 실행시점에 발생하기도 한다 (Runtime Errors).

이러한 오류는 컴파일하고 발생한 오류를 수정하거나, 테스트 실행하면서 기대 값과 다르게 변수 값이 출력이 되면 오류가 발생하는 그 부분을 수정하면서 디버깅을 하게 된다. 우리는 지금까지 이 정도 수준으로 디버깅을 해오고 있다.

난이도가 높지만 Opcode를 실행하면서 오류가 발생하는지 볼 수도 있다. 그렇게 하려면 Opcode, 즉 기계어 형태의 오브젝트 코드인 어셈블러 수준의 이해가 필요하고, 상당한 노력이 필요해서 권하고 싶지는 않다. 여기서는 Opcode가 이런 것이구나, 그리고 REMIX에서 디버깅을 Opcode 수준으로 할 수 있다는 정도까지 살펴보기로 하자.

앞서 ```MyBank.sol```에서 생성된 바이트코드는 다음과 같다.

```python
608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555047600181905550610302806100676000396000f3fe6080604052600436106100555760003560e01c806312065fe01461005a5780632ccb1b30146100855780632e1a7d4d146100.....509056fea2646970667358221220febeb9d97631e86ae2847fb122913ad5400485aa2ea08f75f0f1caa649a1ce8164736f6c63430006010033 (중간 생략)
```

난해해 보이는 이러한 일련의 바이트코드를 Opcode로 변환할 수 있는데, 앞부터 2 자리 씩 읽어보면 된다.
바이트코드는 "6080604052..."로 시작하고 있는데, 이 부분을 2자리 즉 1바이트씩 Opcode로 나타내면 "PUSH1 0x80 PUSH1 0x40 MSTORE..."이다.
이런 변환은 개발자가 직접하기에는 매우 복잡하다.
REMIX에서 컴파일을하면 바이트코드와 함께 Opcode를 생성해주기 때문에 쉽게 알 수 있다.

Opcode를 해석해보자.
* 6080은 PUSH Ox80, PUSH는 stack에 넣는 opcode (POP은 데이터를 stack에서 꺼내는 opcode)
* 6040은 PUSH 0x40
* 52는 mstore이다. mstore는 인자 2개가 필요하다. mstore는 RAM memory에 저장하는 opcode이다 (참고로 disk storage에 저장하는 opcode는 sstore이다). 사용하는 예로 덧셈 결과를 메모리에 저장할 때는 ```mstore(0x10, add(x,y))```라고 해준다.

바이트코드 | Opcode | 설명
-----|-----|-----
6080 | PUSH1 80 | stack에 0x80 넣음.
6040 | PUSH1 40 | stack에 0x40 넣음.
52 |  MSTORE | 

종합적으로 바이트코드 "6080604052..."는 Opcode "PUSH1 0x80 PUSH1 0x40 MSTORE..."이고, 실행결과는 메모리 0x80의 공간을 할당하고, 중간인0x40으로 포인터를 이동. 즉 64바이트 scratch space (사용자데이터를 저장하는 임시저장공간)과 temporary memory storage (단기기억공간)으로 분할하게 된다.

바이트코드 별 Opcode가 있는데, 예를 들면 이렇다. 0x00는 ```STOP```, 0x01은 ```ADD```, 0x02 ```MUL```, ... , 0x52는 ```MSTORE```, 0x55는 ```SSTORE```이다. 이 모든 코드를 어떻게 기억할 필요는 없다. Opcode는 "Ethereum Virtual Machine Opcodes" (https://www.ethervm.io/) 페이지에 상세한 정보가 나와있다. 또한 opcode는 당연히 실행하면 gas 비용이 발생하게 되는데, "eip 150 gas cost"라고 검색하면 구글 스프레드쉬트 문서를 찾을수 있고 상세하게 정리되어 있다.


Operation name | Gas Cost | Function
-----|-----|-----
step	| 1	| Default amount of gas to pay for an execution cycle.
stop 	| 0	| Nothing paid for the SUICIDE operation.
sha3 	| 20 	| Paid for a SHA3 operation.
sload 	| 20 	| Paid for a SLOAD operation.
sstore 	| 100 	| Paid for a normal SSTORE operation (doubled or waived sometimes).
balance 	| 20 	| Paid for a BALANCE operation
create 	| 100 	| Paid for a CREATE operation
call 	| 20 	| Paid for a CALL operation.
memory 	| 1	| Paid for every additional word when expanding memory
txdata 	| 5 	| Paid for every byte of data or code for a transaction
transaction 	| 500 	| Paid for every transaction

컨트랙의 함수버튼을 눌러 실행하고 나서, 디버그 모드로 들어가 보자.
```msg.value```와 ```deposit(uint amount)```의 ```amount```를 다르게 입력하면 ```require()``` 오류가 발생한다.
```step into``` 버튼을 누르면 ```632 REVERT```에서 오류로 인해 중지되고 있다.
이 때 좌측의 소스에 해당 라인이 하이라이트되는 것을 볼 수 있다.
디버깅을 하면서 아래에 지역변수, 스택, 메모리, 스토리지변수에 프로그램에서 쓰이고 있는 변수들이 올바르게 값을 가지고 있는지 확인해보자.

![alt text](figures/9_debug_requireError.png "debug MyBank.sol when runtime error is occured for 'require'")

## 실습문제: 송금 더 해보기

은행 컨트랙을 프로그램하여 아래 기능을 실행하도록 하세요.
* 입금액을 매개변수로 넣어서 입금하세요. amount가 실제 입금액과 일치하지 않으면 오류.
    ```function deposit(uint amount) public payable```
* 전액 인출 함수. 인출 금액을 정하지 않아도 되므로 함수의 인자가 없다.
    ```function widthdrawAll() public```
* 컨트랙 잔고확인 함수 (this를 이용한 잔고, 상태변수 잔고 (this 잔고가 맞는지 확인하는 용도)
	```function getBalance() public view returns(uint, uint)```
* 다른 계정으로 계좌이체 함수
	```function forwardTo(address payable _receiver) public payable```

아래 기능을 실행해 보세요.
- 입금 11111 wei를 하고 컨트랙잔고 11111 wei 출력 (old: 입금 11111 wei, 222 wei를 하고 잔고 11333 wei 출력)
- 자신의 2번째 계정으로 계좌이체 555 wei하고 컨트랙 잔고는 그대로 11111 wei 출력, 자신의 2번째 계정 잔고 증가분 (+555 wei) 출력
- 전액 인출하고 1111 wei, 자신의 잔고 증가분 출력 (+1111 wei)

geth 8445에도 배포하고 동일한 기능을 실행하세요.
주피터 노트북에서 컴파일 (REMIX의 컴파일 결과를 사용해도 됨), 배포, 사용하고 그 결과까지 나오도록 한다.
주피터 노트북에서 사용결과가 나오지 않는 경우에는, 화면을 캡쳐하여 별도 제출해도 된다.

주의:
* 첫 계정은 사설망의 coinbase이기 때문에, 마이닝 보상과 gas비 등이 결제된다. 따라서 증감분이 정확히 반영되지 않는다.
* 일시에 복수의 거래가 발생한다. 따라서 마이닝, 계정해제를 복수거래에 대해 해주어야 한다.
* 이벤트 발생을 위해서는 웹소켓을 사용한다. ganache는 웹소켓이 그대로 가능하다. 반면 geth는 웹소켓을 추가되어 있어야 한다.

### 단계 1: 컨트랙 개발

이벤트는 다음 주에 배우는 내용이다.
이번 문제에서는 이벤트를 제외하고 해도 좋다.

줄 | 설명
-----|-----
4 | ```address```선언
5 ~ 9 | 생성자에 ```msg.sender```를 ```owner```로 설정, balance 초기화
10 | ```event``` 설정하여 송금할 경우 알림
10 ~ 17 | 송금. ```owner```만 송금할 수 있고 ```event``` 발생. msg.value를 넘겨주는 단순 이체라서 잔고를 줄이지 않음.
18 ~ 20 | 잔고는 (1) addres(this).balance와 (2) balance (수작업으로 조정하는 잔고)를 돌려줌. 이들 잔고는 서로 동일해야 함.
25 ~ 29 | ```owner```에게 전액 출금


```python
[파일명: src/BankV2.sol]
pragma solidity ^0.5.0;

contract BankV2 {
    address owner;
    uint balance;
    constructor() public {
        owner = msg.sender;
        balance = 0;
    }
    event Sent(address from, address to, uint amount );  // to learn next week
    //function send(address payable _receiver, uint _amount) public payable {
    function forwardTo(address payable _receiver) public payable {
        //balance -= msg.value;
        require(msg.sender == owner);
        _receiver.transfer(msg.value);
        emit Sent(msg.sender, _receiver, msg.value);  // event
    }
    function getBalance() public view returns(uint, uint) {
        return (balance, address(this).balance);
    }
    function deposit(uint amount) public payable {
        require(msg.value == amount);
        balance += amount;
    }
    function widthdrawAll() public {
        balance -= address(this).balance;
        require(msg.sender == owner);
        msg.sender.transfer(address(this).balance);
    }
}
```

아래는 버전 0.8을 적용한 코드이다.
주의할 점이 있다. 버전 0.8.1로 수정하고, 배포는 문제가 없이 된다. 그러나 API를 사용하면서 ```bank.events.Sent```가 함수가 아니라는 오류가 발생한다. 버전 0.6.1을 적용해도 마찬가지 오류가 지속된다.
**ABI를 버전0.5 컴파일의 것으로 교체**하면 그런 오류가 발생하지 않는 것으로 보인다. 따라서 버전 0.5로 낮추어 진행하기로 한다.
* 줄1 경고: SPDX
* 줄7 경고: public 제거
* 줄27 오류: payable(msg.sender)


```python
[파일명: src/BankV2.sol]
//SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract BankV2 {
    address owner; //from 0.8.0 no need to declare the address as payable explicitly
    uint balance; 
    constructor() public payable { // warning: from 0.7? remove public -> constructor() payable
        owner = msg.sender;
        balance = 0;
    }
    event Sent(address from, address to, uint amount );  // to learn next week
    //function send(address payable _receiver, uint _amount) public payable {
    function forwardTo(address payable _receiver) public payable {
        //balance -= msg.value;
        require(msg.sender == owner);
        _receiver.transfer(msg.value);
        emit Sent(msg.sender, _receiver, msg.value);  // event
    }
    function getBalance() public view returns(uint, uint) {
        return (balance, address(this).balance);
    }
    function deposit(uint amount) public payable {
        require(msg.value == amount);
        balance += amount;
    }
    function widthdrawAll() public {
        balance -= address(this).balance;
        require(msg.sender == owner);
        //owner.transfer(address(this).balance); //error pyable
        //payable(msg.sender).transfer(address(this).balance); //ok
        payable(owner).transfer(address(this).balance); //to use payable from 0.8.0 (but error with 0.5)
    }
}

```

REMIX에서 기능테스트까지 마치고, 컴파일도 성공한다면 컴파일 오류는 없다는 의미이다.
노드에서 배포, 사용하면서 실행오류가 발생할 수 있다.

### 단계 2: 컴파일

아래 로컬 solc를 사용하면 버전 0.5가 아니므로, REMIX의 결과를 복사해서 사용한다.

```python
pjt_dir> solc --optimize --combined-json abi,bin src/BankV2.sol > src/BankV2.json
```

### 단계 3: 컨트랙 배포

web3 버전 1.0 이후로 적용하여 async 함수를 사용하고 있다.
async, await를 사용하면, 비동기로 인한 문제가 해결된다.
계정을 하드코딩하거나, contract을 생성하면서 주소를 기다려 받아올 수 있게 된다.
아래 await를 보면 그러한 비동기 코드가 수정되고 있다.


```python
[파일명: src/BankV2Deploy.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    //web3 = new Web3(new Web3.providers.WebsocketProvider('ws://localhost:8345'));
    web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8345'));
}
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
var _bin="608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555060006001819055506103bb806100686000396000f3fe608060405260043610610062576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806312065fe01461006757806327d8ad88146100995780633c459375146100dd578063b6b55f25146100f4575b600080fd5b34801561007357600080fd5b5061007c610122565b604051808381526020018281526020019250505060405180910390f35b6100db600480360360208110156100af57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610148565b005b3480156100e957600080fd5b506100f261028c565b005b6101206004803603602081101561010a57600080fd5b810190808035906020019092919050505061036e565b005b6000806001543073ffffffffffffffffffffffffffffffffffffffff1631915091509091565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415156101a357600080fd5b8073ffffffffffffffffffffffffffffffffffffffff166108fc349081150290604051600060405180830381858888f193505050501580156101e9573d6000803e3d6000fd5b507f3990db2d31862302a685e8086b5755072a6e2b5b780af1ee81ece35ee3cd3345338234604051808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001828152602001935050505060405180910390a150565b3073ffffffffffffffffffffffffffffffffffffffff16316001600082825403925050819055506000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614151561030e57600080fd5b3373ffffffffffffffffffffffffffffffffffffffff166108fc3073ffffffffffffffffffffffffffffffffffffffff16319081150290604051600060405180830381858888f1935050505015801561036b573d6000803e3d6000fd5b50565b803414151561037c57600080fd5b806001600082825401925050819055505056fea165627a7a72305820ecd3a640093ab47ba909f460ca2bf7e1080c33e50aebc693edea0f112b3c6d4f0029";

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 1000000, gasPrice: '1000000000'}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance) {
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()

//old way - without async, await
//var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
//_contract
//    .deploy({data:"0x"+_bin})
//    .send({from:"0x02e3d1B8b7C63bDB77CdF8B50e4394EAa640E59d",gas:1000000})
//    .then(function(newContractInstance){
//        console.log(newContractInstance.options.address) // instance with the new contract address
//    });
```

결과를 보자.
기대한 바와 같이, 첫 째 계정을 하드코딩하지 않고도 인식하고 있다.
그리고 배포된 주소도 출력하고 있다.

```python
pjt_dir> node src/BankV2Deploy.js

    Deploying the contract from 0x9357f478d86D9222f4413bFd91C8adb0F4c728b7
    hash: 0xc1c1547b5b9662979760caff95ffe1619c17a95eb4e1b327a96ae84f77deb58b
    ---> The contract deployed to: 0x770c4ce87b2B47FA753Ca10ea34e571c0436E7b0
```

### 단계 4: 사용

이벤트의 인자를 잘 살펴보자. 데이터타입이 ```address```로 선언되어 있다.
```address```를 **따옴표 없이 사용**하면 올바르게 처리되지 않는다.
args출력을 비교해 보면 원래 함수 인자에서 주어진 값과 다르게 변경된 것을 알 수 있다.

```python
args: 
   { from: '0x21c704354d07f804bab01894e8b4eb4e0eba7451',
     to: '0x778ea91cb0d08927fa4bf3f90a7ccbb700000000',
          // 원래 함수 인자와 다름 0x778ea91cb0d0879c22ca20c5aea6fbf8cbeed480
     amount: { [String: '555'] s: 1, e: 2, c: [Object] } }
```

#### 이벤트 발생하지 않는 경우

배포를 ganache에서 했으므로 ganache에서 사용하는 코드를 작성해보자.

우선 이벤트를 잡아내는 코드를 제거하고 코딩해보자. 우선 HttpProvider를 생성해서 실행해보자.

await를 언제 적어주는가? * promise가 발생하는 Javascript 함수에 적어준다.
* await는 deposit(), forwardTo(), withdrawAll() 등 거래에만 적어준다.
* bank의 getBalance()에는 await를 붙이지 않고 있다.

```python
[파일명: src/BankV2NoEventUse.js]
var Web3=require('web3');
var _abiBinJson = require('./MyBank.json');      //importing a javascript file

var web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://localhost:8345'));
contractName=Object.keys(_abiBinJson.contracts); // reading ['src/MyBank.sol:MyBank']
//console.log("- contract name: ", contractName); //or console.log(contractName[0]);
_abi=_abiBinJson.contracts[contractName].abi;
_abiArray=JSON.parse(JSON.stringify(_abi));
//_abiArray=JSON.parse(_abi);      //JSON parsing needed!!
//_bin=_abiBinJson.contracts[contractName].bin;
//console.log("- ABI: " + _abiArray);
//console.log("- Bytecode: " + _bin);

//----copied ABI of solidity 0.5 begins
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
//----copied ABI of solidity 0.5 ends

var bank = new web3.eth.Contract(_abiArray,"0x770c4ce87b2B47FA753Ca10ea34e571c0436E7b0");
//var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
var event = bank.events.Sent(function (error, result) {
    if (!error) {
        console.log("Event fired: " + JSON.stringify(result.returnValues));
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account0: " + accounts[0]);
    const balance0Before = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 before: " + balance0Before);
    const balance1Before = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 before: " + balance1Before);

    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal before deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    await bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal after deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    const forward = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
    //const forward = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(forward.events.Sent.returnValues));});
  
    console.log("---> forwardTo called " + JSON.stringify(forward.events.Sent.returnValues));
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after forwardTo: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0After = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after: " + balance0After);
    console.log("Balance0 diff: " + (balance0After - balance0Before));
    const balance1After = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 after: " + balance1After);
    console.log("Balance1 diff: " + (balance1After - balance1Before));

    const withdraw = await bank.methods.widthdrawAll().send({from: accounts[0],gas:100000});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after withdrawAll: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0AfterWithdrawAll = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after withdrawAll: " + balance0AfterWithdrawAll);
    console.log("Balance0 diff after withdrawAll: " + (balance0After - balance0AfterWithdrawAll));
}
doIt()
```

#### 컨트랙 주소를 잘 못 입력하면 Out of Gas 오류가 발생

위 contract의 주소를 잘 못 넣으면, 오류가 발생한다. 왜 오류가 발생할까?
모르는 계정에서 gas를 지급하려고 하니, Out of Gas 오류가 발생하기 마련이다.

#### Http는 이벤트 리스닝 못한다.

앞서 Out of gas 오류에 대해 기억해보니, ganache를 재시작하고 배포하면 컨트랙 주소가 변경이 된다.
다시 배포하고 주소를 가져와 복사한 후 재실행 한다.
이번에는 HttpProvider로 설정하고 실행해 보자.


```python
[파일명: src/BankV2NoEventUse.js]
var Web3=require('web3');
var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
//var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://117.16.44.45:8345'));
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
var bank = new web3.eth.Contract(_abiArray, '0x8f87B1A38380a474f7cdfE2fb3A16e1F856C5A1A');
var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
    if (!error) {
        console.log("Event fired: " + JSON.stringify(result.returnValues));
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account0: " + accounts[0]);
    const balance0Before = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 before: " + balance0Before);
    const balance1Before = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 before: " + balance1Before);

    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal before deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    await bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal after deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    const forward = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
    //const forward = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(forward.events.Sent.returnValues));});
  
    console.log("---> forwardTo called " + JSON.stringify(forward.events.Sent.returnValues));
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after forwardTo: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0After = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after: " + balance0After);
    console.log("Balance0 diff: " + (balance0After - balance0Before));
    const balance1After = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 after: " + balance1After);
    console.log("Balance1 diff: " + (balance1After - balance1Before));

    const withdraw = await bank.methods.widthdrawAll().send({from: accounts[0],gas:100000});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after withdrawAll: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0AfterWithdrawAll = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after withdrawAll: " + balance0AfterWithdrawAll);
    console.log("Balance0 diff after withdrawAll: " + (balance0After - balance0AfterWithdrawAll));
}
doIt()
```


```python
아래 결과에서 보듯이, HttpProvider는 Event를 리스닝하지 못하고 있다.
결과 중간쯤에 보면, 함수를 호출하면서 발생한 로그를 통해 value.events.Sent.returnValues를 출력할 수 있다.
```


```python
pjt_dir> node src/BankV2NoEventUse.js

    Account0: 0xB09867c51FA8A889b3B2706653e58E471f387a79
    Balance0 before: 99979548627999988896
    Balance1 before: 100000000000000004995
    Contract bal before deposit: 0 this.bal:0
    Contract bal after deposit: 1111 this.bal:1111
    ---> forwardTo called {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
    Contract balance after forwardTo: 1111 this.bal:1111
    Balance0 after: 99978061127999987230
    Balance0 diff: -1487500000002048
    Balance1 after: 100000000000000005550
    Balance1 diff: 0
    Contract balance after withdrawAll: 0 this.bal:0
    Balance0 after withdrawAll: 99977655707999988341
    Balance0 diff after withdrawAll: 405419999985664
```

위 출력을 단계별로 설명하고 있다. 송금, 잔고변화 등 어떤 결과가 발생했는지 살펴보자.

```python
Account0: 0xB09867c51FA8A889b3B2706653e58E471f387a79
Balance0 before: 99979548627999988896
Balance1 before: 100000000000000004995
Contract bal before deposit: 0 this.bal:0
Contract bal after deposit: 1111 this.bal:1111        <--- 컨트랙의 잔고가 desposit한 금액만큼 증가
---> forwardTo called {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
Contract balance after forwardTo: 1111 this.bal:1111  <--- 단순 전달이므로 컨트랙의 잔고는 변동이 없다.
Balance0 after: 99978061127999987230
Balance0 diff: -1487500000002048                      <--- gas비, 마이닝보상 등으로 잔고의 변동이 정확히 산정되기 어렵다.
Balance1 after: 100000000000000005550                 <--- 555만큼 증가
Balance1 diff: 0   <--- 미세한 차액이 정밀도로 인해 0으로 출력
Contract balance after withdrawAll: 0 this.bal:0      <--- 컨트랙에서 전액출금한 후 잔고가 0
Balance0 after withdrawAll: 99977655707999988341      <--- gas비, 마이닝보상 등으로 잔고의 변동을 정확히 산정하기 어렵다.
Balance0 diff after withdrawAll: 405419999985664
```

geth 단말에서도 그 변화를 알 수 있다.
```python
geth> eth.getBalance(eth.accounts[1]);
0   <--- 이전 잔고
> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
    gas: 100000,
    gasPrice: 500000000000,
    hash: "0xa4c9e98767fbe60a0722b3d31647ca351d56ef7a960f00d20edbc6e26a98c94f",
    input: "0x3e58c58c000000000000000000000000778ea91cb0d0879c22ca20c5aea6fbf8cbeed480",
    nonce: 32,
    r: "0x733d32c710bc4d4944da811daa0d639b3a0241cd3994039600c73d69f247a6d",
    s: "0x78476837a95b30c7991af36c32eccbe4ee4f375349adb012db3c8f0c919a0213",
    to: "0x53fd2f60c463f19c8c7bcdb5e699cac94b8300b2",
    transactionIndex: 0,
    v: "0x66",
    value: 555 <--- 전송될 금액
}]
> miner.start(1);admin.sleepBlocks(1);miner.stop();
null
> eth.getBalance(eth.accounts[1]);
555 <--- 이후 잔고
```

#### 이벤트가 발생하는 경우: 웹소켓

이번에는 웹소켓으로 Event를 발생해서 해보자.
HttpProvider는 이벤트 subscribe, 리스닝을 지원하지 않는다.
WebsocketProvider를 사용하자.

줄 2에 http 또는 web socket을 사용할 수 있다는 점에 주의하자.
* WebsocketProvider를 사용하면, 주피터 노트북에서는 웹소켓을 열기때문에 프로세스가 지속된다.
```[*]```표로 지속되고 있다. 이는 매우 불편하기 마련이다. 다름 작업을 할 수 없기 때문이다.
* HttpProvider를 사용하면 프로세스가 지속되는 문제는 발생하지 않게 된다.

```python
[파일명: src/BankV2Use.js]
var Web3=require('web3');
//var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://117.16.44.45:8345'));
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
var bank = new web3.eth.Contract(_abiArray, '0x8f87B1A38380a474f7cdfE2fb3A16e1F856C5A1A');
var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
    if (!error) {
        console.log("Event fired: " + JSON.stringify(result.returnValues));
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account0: " + accounts[0]);
    const balance0Before = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 before: " + balance0Before);
    const balance1Before = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 before: " + balance1Before);

    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal before deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    await bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal after deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    const forward = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
    //const forward = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(forward.events.Sent.returnValues));});
  
    console.log("---> forwardTo called " + JSON.stringify(forward.events.Sent.returnValues));
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after forwardTo: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0After = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after: " + balance0After);
    console.log("Balance0 diff: " + (balance0After - balance0Before));
    const balance1After = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 after: " + balance1After);
    console.log("Balance1 diff: " + (balance1After - balance1Before));

    const withdraw = await bank.methods.widthdrawAll().send({from: accounts[0],gas:100000});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after withdrawAll: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0AfterWithdrawAll = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after withdrawAll: " + balance0AfterWithdrawAll);
    console.log("Balance0 diff after withdrawAll: " + (balance0After - balance0AfterWithdrawAll));
}
doIt()
```

프로세스가 완료되지 않고 계속 이벤트를 대기하기 때문에 ```[*]``` 실행중이 표시되고 있다.
그러나 이벤트가 발생(fire)되는 것을 포함하여 다른 결과가 출력되고 있다.

```python
pjt_dir> node src/BankV2Use.js

    Account0: 0xB09867c51FA8A889b3B2706653e58E471f387a79
    Balance0 before: 99976771547999986675
    Balance1 before: 100000000000000005550
    Contract bal before deposit: 0 this.bal:0
    Contract bal after deposit: 1111 this.bal:1111
    Event fired: {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
    ---> forwardTo called {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
    Contract balance after forwardTo: 1111 this.bal:1111
    Balance0 after: 99975284047999985009
    Balance0 diff: -1487500000002048
    Balance1 after: 100000000000000006105
    Balance1 diff: 0
    Contract balance after withdrawAll: 0 this.bal:0
    Balance0 after withdrawAll: 99974878627999986120
    Balance0 diff after withdrawAll: 405420000002048
```

출력메시지를 이해해보자.

```python
Account0: 0xB09867c51FA8A889b3B2706653e58E471f387a79
Balance0 before: 99976771547999986675
Balance1 before: 100000000000000005550
Contract bal before deposit: 0 this.bal:0
Contract bal after deposit: 1111 this.bal:1111
Event fired:   <--- 줄8의 이벤트가 발생. forwardTo() 함수가 호출되면서, Sent 이벤트가 발생. 매개변수의 출력 값을 아래에서 확인. {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
---> forwardTo called {"0":"0xB09867c51FA8A889b3B2706653e58E471f387a79","1":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","2":"555","from":"0xB09867c51FA8A889b3B2706653e58E471f387a79","to":"0x31188410B764b626777ACe8B4aaf5866c6Ce4662","amount":"555"}
Contract balance after forwardTo: 1111 this.bal:1111
Balance0 after: 99975284047999985009
Balance0 diff: -1487500000002048
Balance1 after: 100000000000000006105
Balance1 diff: 0
Contract balance after withdrawAll: 0 this.bal:0
Balance0 after withdrawAll: 99974878627999986120
Balance0 diff after withdrawAll: 405420000002048
```

컨트랙에 대해 발생한 과거 이벤트를 모두 ```getPastEvents()``` 함수로 알아보자. 
이벤트가 발생한 블록을 설정할 수 있다. 이 때 범위를 넓게 하면 로그를 찾다가 부하가 발생할 수 있으니 (예를 들어, main net에 최초블록부터 찾으라고 한다면 당연히 부하가 발생할 것이다), 실제 필요한 범위로 좁혀서 설정한다.

```python
bank.getPastEvents('AllEvents', {
    fromBlock: 0,
    toBlock: 'latest'
    }, function(error, result) {
        if (!error) {
            console.log(result);
        } else {
            console.log(error);
        }
    });
```

```python
bank.getPastEvents('Sent', {
    fromBlock: 0,
    toBlock: 'latest'
    }, function(error, result) {
        if (!error) {
            console.log(result);
        } else {
            console.log(error);
        }
    });
```

반환은 다음과 같기 때문에, ```result.returnValues.amount```와 같이 인자명에 키를 더해서 그 값을 볼 수 있다.


```python
 {
    logIndex: 0,
    transactionIndex: 0,
    transactionHash: '0xfaac636e8c059c2fa2a4e6378c33c04a4884f73c4a5041a66809a2d90f4feec8',
    blockHash: '0x81a5162417a5c1d273764c28340ca77637d97088c79d930f5bdf26fb4d21fbf3',
    blockNumber: 23,
    address: '0xfBe8337DFDcBF32247b392FA1F13326A09FedCC6',
    type: 'mined',
    id: 'log_8240e959',
    returnValues: Result {
      '0': '0xf84B27A6D281Df793C67Dc12A2D69FF003BB5732',
      '1': '0x27F9cb3E30ffa2576d2749aa609FC465f6c00cEE',
      '2': '555',
      from: '0xf84B27A6D281Df793C67Dc12A2D69FF003BB5732',
      to: '0x27F9cb3E30ffa2576d2749aa609FC465f6c00cEE',
      amount: '555'
    },
    event: 'Sent',
    signature: '0x3990db2d31862302a685e8086b5755072a6e2b5b780af1ee81ece35ee3cd3345',
    raw: {
      data: '0x000000000000000000000000f84b27a6d281df793c67dc12a2d69ff003bb573200000000000000000000000027f9cb3e30ffa2576d2749aa609fc465f6c00cee000000000000000000000000000000000000000000000000000000000000022b',
      topics: [Array]
    }
  }
```

### 단계 3: geth@8445 배포

이번에는 geth@8445에서 해보자.

```python
[파일명: src/BankV2Deploy.js]
var Web3=require('web3');
var web3;
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    //web3 = new Web3(new Web3.providers.WebsocketProvider('ws://117.16.44.45:8345'));
    web3 = new Web3(new Web3.providers.HttpProvider('http://117.16.44.45:8445'));
}
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
var _bin="608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555060006001819055506103bb806100686000396000f3fe608060405260043610610062576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806312065fe01461006757806327d8ad88146100995780633c459375146100dd578063b6b55f25146100f4575b600080fd5b34801561007357600080fd5b5061007c610122565b604051808381526020018281526020019250505060405180910390f35b6100db600480360360208110156100af57600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610148565b005b3480156100e957600080fd5b506100f261028c565b005b6101206004803603602081101561010a57600080fd5b810190808035906020019092919050505061036e565b005b6000806001543073ffffffffffffffffffffffffffffffffffffffff1631915091509091565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415156101a357600080fd5b8073ffffffffffffffffffffffffffffffffffffffff166108fc349081150290604051600060405180830381858888f193505050501580156101e9573d6000803e3d6000fd5b507f3990db2d31862302a685e8086b5755072a6e2b5b780af1ee81ece35ee3cd3345338234604051808473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020018373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001828152602001935050505060405180910390a150565b3073ffffffffffffffffffffffffffffffffffffffff16316001600082825403925050819055506000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614151561030e57600080fd5b3373ffffffffffffffffffffffffffffffffffffffff166108fc3073ffffffffffffffffffffffffffffffffffffffff16319081150290604051600060405180830381858888f1935050505015801561036b573d6000803e3d6000fd5b50565b803414151561037c57600080fd5b806001600082825401925050819055505056fea165627a7a72305820ecd3a640093ab47ba909f460ca2bf7e1080c33e50aebc693edea0f112b3c6d4f0029";

//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
async function deploy() {
    const accounts = await web3.eth.getAccounts();
    console.log("Deploying the contract from " + accounts[0]);
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .send({from: accounts[0], gas: 1000000, gasPrice: '1000000000'}, function(err, transactionHash) {
                if(!err) console.log("hash: " + transactionHash); 
        })
        //.then(function(newContractInstance) {
        //    console.log(newContractInstance.options.address)
        //});
    console.log("---> The contract deployed to: " + deployed.options.address)
}
deploy()

//var _contract = new web3.eth.Contract(_abiArray);
//unlock the account with a password provided
//web3.personal.unlockAccount(web3.eth.accounts[0],'password');
//_contract
//    .deploy({data:"0x"+_bin})
//    .send({from:"0x02e3d1B8b7C63bDB77CdF8B50e4394EAa640E59d",gas:1000000})
//    .then(function(newContractInstance){
//        console.log(newContractInstance.options.address) // instance with the new contract address
//    });
```

#### 지급계정 해제

실행하려면 지급계정을 해제해 주어야 한다.
계정을 해제하지 않고 실행하면 authentication 오류가 발생한다: 
```Error: Returned error: authentication needed: password or unlock```
그리고 마이닝을 해주어야 한다.
그렇지 않으면 마이닝이 완료되지 전까지 대기 상태로 지속된다.

#### gas limit 오류

```python
pjt_dir> node src/BankV2Deploy.js
```

```python
Deploying the contract from 0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0
hash: 0x4317dbc15104710db6c49a8b3e3ce0ba19a0f91510896af533351088d6bf5f46
(node:10788) UnhandledPromiseRejectionWarning: Error: The contract code couldn't be stored, please check your gas limit.
    at Object.TransactionError (C:\Users\jsl\Code\201711111\node_modules\web3-core-helpers\src\errors.js:63:21)
    at Object.ContractCodeNotStoredError (C:\Users\jsl\Code\201711111\node_modules\web3-core-helpers\src\errors.js:72:21)
    at C:\Users\jsl\Code\201711111\node_modules\web3-core-method\src\index.js:387:40
    at process._tickCallback (internal/process/next_tick.js:68:7)
(node:10788) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:10788) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
```

gas비를 1,000,000에서 3,000,000으로 수정하였다.
gas비는 REMIX 편집기 밑의 단말 창에서 (우측 하단 ) 테스트하면서 확인할 수 있다.
또는 estimateGas()로 산정할 수도 있다.

```python
async function deploy() {
    var deployed = await new web3.eth.Contract(_abiArray)
        .deploy({data: "0x"+_bin})
        .estimateGas()
        .then(function(gas) {console.log("estimated gas: "+gas)});
}
```

![alt text](figures/9_checkGas4GasLimitError.png "see REMIX terminal for gas limit error")

#### timeout 오류

gaslimit 오류는 사라졌으나, 주피터 노트북에서는 시간초과 오류가 발생하고 있다.
geth 네트워크의 문제, 예를 들어 블록체인이 동기화가 되어 있지 않은 상태라 마이닝이 적체되어 시간이 오래 걸릴 수 있다.
web3에서 transactionPollingTimeout 기본 값이 750초로 한정되어 있다.

timeout이 일어났다고 하더라도, 마이닝이 성공했다면 (miner.start();admin.sleepBlocks(1); miner.stop();의 대기가 해소되었다면)

그리고 log에 commit이 발생했다면, hash값으로 다음과 같이 컨트랙 주소를 얻자.
```python
geth > eth.getTransactionReceipt("0xb3de97ccaeb95a04de838a87dc732d3e19f564306e4790f064a7ac0bb490ff2c")
{
  blockHash: "0xcf4a891a8a779712a2ad570dc84895614c5340d3d7aaf3183133fd616b816cb8",
  blockNumber: 49272,
  contractAddress: "0x0d1fc28d91167110b7728a4f20e07bc4308a78f9",
  cumulativeGasUsed: 336852,
  from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
  gasUsed: 336852,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  root: "0x7ef2e50aa73b17ae2f0d15516cdf8317c1b1619684df51a4333427cb15230c2f",
  to: null,
  transactionHash: "0xb3de97ccaeb95a04de838a87dc732d3e19f564306e4790f064a7ac0bb490ff2c",
  transactionIndex: 0
}
```

위에서 배포하여 얻은 컨트랙 주소 "0x0d1fc28d91167110b7728a4f20e07bc4308a78f9"를 사용한다.

### 단계 4: geth@8446 웹소켓을 사용하여 사용

#### connection not open 오류

geth@8445로 하면 ```connection not open``` 오류가 발생한다.

```
connection not open on send()
connection not open on send()
connection not open on send()
(node:24128) UnhandledPromiseRejectionWarning: Error: connection not open
```

http대신 웹소켓을 사용하면 된다. WebsocketProvider('ws://117.16.44.45:8446') 이렇게 해준다.

#### gethNow.bat에 웹소켓 추가

다음과 같이 ```--ws --wsport 8446 --wsorigins "*"```를 추가한다.
물론 그 다음 gethNow.bat을 재실행한다.

```
geth --identity "jsl" --unlock 0 --datadir .\eth --ws --wsport 8446 --wsorigins "*" --rpc --rpcaddr "localhost" --rpcport "8445" --rpccorsdomain "*" --port "38445" --rpcapi "admin, db, eth, debug, miner, net, shh, txpool, personal, web3" --networkid 33
```


먼저 노드 창에서 웹소켓이 연결되는지 테스트해보자.
* 웹소켓은 new Web3(new Web3.providers.WebsocketProvider('ws://localhost:8446')); 명령어로 생성.
* ```Event fired``` 출력에서 보듯이, 이벤트가 발생하고 있다.
* ```web3._provider.disconnect()``` 하면 끊어짐.


```python
node> var Web3=require('web3');
> var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://localhost:8446'));
undefined
> web3.version
'1.2.6'
> var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
undefined
> async function doIt() {
...     var bank = new web3.eth.Contract(_abiArray, '0xb69A207C9Cb6cA35E2B797A430542D2ec79c06eA');
...     var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
.....         if (!error) {
.......             console.log("Event fired: " + JSON.stringify(result.returnValues));
.......         }
.....     });
...
...     const accounts = await web3.eth.getAccounts();
...     console.log("Account0: " + accounts[0]);
...     const balanceBefore = await web3.eth.getBalance(accounts[0]);
...     console.log("Balance0 before: " + balanceBefore);
...
...
...     bank.methods.getBalance().call().then(console.log);
...     bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
...     bank.methods.getBalance().call().then(console.log);
...
...     const value = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
...     //const value = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
...         //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(value.events.Sent.returnValues));});
...
...     bank.methods.getBalance().call().then(console.log);
...
...     console.log("---> forwardTo called " + JSON.stringify(value.events.Sent.returnValues));
...     const balanceAfter = await web3.eth.getBalance(accounts[0]);
...     console.log("Balance after: " + balanceAfter);
...     console.log("Balance diff: " + (balanceBefore - balanceAfter));
... }
undefined
> doIt()
Promise {
  <pending>,
  domain:
   Domain {
     domain: null,
     _events:
      [Object: null prototype] {
        removeListener: [Function: updateExceptionCapture],
        newListener: [Function: updateExceptionCapture],
        error: [Function: debugDomainError] },
     _eventsCount: 3,
     _maxListeners: undefined,
     members: [] } }
> Account0: 0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0
Balance0 before: 24464013749999999990559
Result { '0': '4444', '1': '4444' }
Result { '0': '4444', '1': '4444' }
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
>

```

앞서 사용했던 파일을 1줄만 수정한다.
포트번호는 설정한 8446을 사용한다.
컨트랙 주소는 앞서 배포하고 얻은 것을 사용한다.

```python
[파일명: src/BankV2Use.js]
var Web3=require('web3');
//var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://117.16.44.45:8446'));
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];
var bank = new web3.eth.Contract(_abiArray, '0x0d1fc28d91167110b7728a4f20e07bc4308a78f9');
var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
    if (!error) {
        console.log("Event fired: " + JSON.stringify(result.returnValues));
    }
});

async function doIt() {
    const accounts = await web3.eth.getAccounts();
    console.log("Account0: " + accounts[0]);
    const balance0Before = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 before: " + balance0Before);
    const balance1Before = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 before: " + balance1Before);

    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal before deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    await bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract bal after deposit: " + bal[0] + " this.bal:" + bal[1]);
    });
    const forward = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
    //const forward = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(forward.events.Sent.returnValues));});
  
    console.log("---> forwardTo called " + JSON.stringify(forward.events.Sent.returnValues));
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after forwardTo: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0After = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after: " + balance0After);
    console.log("Balance0 diff: " + (balance0After - balance0Before));
    const balance1After = await web3.eth.getBalance(accounts[1]);
    console.log("Balance1 after: " + balance1After);
    console.log("Balance1 diff: " + (balance1After - balance1Before));

    const withdraw = await bank.methods.widthdrawAll().send({from: accounts[0],gas:100000});
    bank.methods.getBalance().call().then(function(bal) {
        console.log("Contract balance after withdrawAll: " + bal[0] + " this.bal:" + bal[1]);
    });
    const balance0AfterWithdrawAll = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 after withdrawAll: " + balance0AfterWithdrawAll);
    console.log("Balance0 diff after withdrawAll: " + (balance0After - balance0AfterWithdrawAll));
}
doIt()
```

#### 계정해제 오류

```python
pjt_dir> node src/BankV2Use.js
Account0: 0x21c704354D07f804baB01894e8B4eB4E0EBA7451
Balance0 before: 10329000001089349948334
Balance1 before: 184999843394000029999
Contract bal before deposit: 1111 this.bal:1111
(node:15470) UnhandledPromiseRejectionWarning: Error: Returned error: authentication needed: password or unlock <--- 계정해제 오류
```

단말에서 다음과 같이 해제해주어야 한다. 1회의 거래 동안 유효하다.
```
geth> personal.unlockAccount(eth.accounts[0]);
Unlock account 0x21c704354d07f804bab01894e8b4eb4e0eba7451
Password:
true
```

그러면 다음과 같이 deposit거래가 전송되고, 마이닝되기를 기다린다.


```python
pjt_dir> node src/BankV2Use.js
Account0: 0x21c704354D07f804baB01894e8B4eB4E0EBA7451
Balance0 before: 10329000001089349948334
Balance1 before: 184999843394000029999
Contract bal before deposit: 1111 this.bal:1111
<--- 여기에 커서가 깜빡이며 거래 (forwardTo)의 발생에 따른 마이닝을 대기함
```

단말로 가서 마이닝을 시작한다. 상황에 따라 다르겠지만 짧게는 수 분, 수 시간이 걸리기도 한다.


```python
geth> miner.start();admin.sleepBlocks(1);miner.stop()
```

그리고 geth 단말의 로그를 관찰한다.

```python
9512 INFO [05-20|12:27:04.195] Setting new local account                address=0x21c704354D07f804baB01894e8B4eB4E0EBA7451
9513 INFO [05-20|12:27:04.195] Submitted transaction    <--- 거래가 제출되고, 마이닝을 대기                fullhash=0x6f2bae9a3f6f9793e2ebf7a8ebeff1f82cd3e519de34436056a480     3a2ee74c55 recipient=0x0D1Fc28D91167110B7728a4f20E07BC4308A78f9    <--- 컨트랙 주소도 보인다.
9514 INFO [05-20|12:27:21.217] Updated mining threads                   threads=4
9515 INFO [05-20|12:27:21.217] Transaction pool price threshold updated price=1000000000
9516 INFO [05-20|12:27:21.218] Commit new mining work                   number=49276 sealhash=0e5689…f647df uncles=0 txs=0 gas=0 fees=0      elapsed=87.163µs
9517 INFO [05-20|12:27:22.674] Commit new mining work                   number=49276 sealhash=f00d15…2709ab uncles=0 txs=1 gas=27048 fee     s=2.7048e-05 elapsed=1.456s
```

또 다른 geth attach 창을 열어 pendingTransactions가 있는지, 처리가 되고 있는지 확인한다.
```python
pjt_dir> geth attach http://localhost:8445
Welcome to the Geth JavaScript console!

instance: Geth/jslNode/v1.9.9-stable-01744997/linux-amd64/go1.13.4
coinbase: 0x21c704354d07f804bab01894e8b4eb4e0eba7451
at block: 49275 (Thu, 20 May 2021 11:10:47 KST)
 datadir: /home/jsl/eth
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
    gas: 80000,  <--- gas 80,000인 desposit 거래. 노드가 처리되면서 커서가 대기하고 있는 상태와 동일하다.
    gasPrice: 1000000000,
    hash: "0x6f2bae9a3f6f9793e2ebf7a8ebeff1f82cd3e519de34436056a4803a2ee74c55",
    input: "0xb6b55f250000000000000000000000000000000000000000000000000000000000000457",
    nonce: 275,
    r: "0x5d40aadf45d652929bf8361a776b615ada800de36b600c6fe8d4fdb9676e9533",
    s: "0x5cdc65d9ded8f8c24cb4275c9c87d15bb1ad232528f6ba1f1b52e79832dc2fb7",
    to: "0x0d1fc28d91167110b7728a4f20e07bc4308a78f9",
    transactionIndex: null,
    v: "0x65",
    value: 1111
}]
>
```

마이닝이 완료되었지만, 아래 오류가 발생한다.

jsl@jsl-smu:~/Code/git/bb/jsl/bitcoin$ node src/BankV2Use.js
Account0: 0x21c704354D07f804baB01894e8B4eB4E0EBA7451
Balance0 before: 10329000001089349948334
Balance1 before: 184999843394000029999
Contract bal before deposit: 1111 this.bal:1111
Contract bal after deposit: 2222 this.bal:2222   <--- 1건 거래 deposit하고 종료. 다음 거래 forwardTo를 실행하면서 계정해제 오류
(node:15525) UnhandledPromiseRejectionWarning: Error: Returned error: authentication needed: password or unlock

건수를 제한하지 않고 miner.start()를 실행함.
그러면 계정해제를 기한을 정해서 다음과 같이 한다.
마지막 인자는 해제되는 기간을 의미한다. 그 기간을 초로 설정할 수 있고, 0이면 무기한을 말한다.

```
geth> personal.unlockAccount(eth.accounts[0], "your password", 0);
true
```

#### 순서대로 마이닝

자바스크립트 파일에 여러 거래가 묶여져 있다.
실제 이렇게 처리하는 경우는 많지 않다.
웹에서 구현하면, 기능버튼이 나타나고 클릭에 따라 해당 거래가 실행된다. 그러고 나면 다른 거래가 이어지게 된다.
여기서는 복수 거래를 완성하기 위해
* 마이닝 서버로 실행하고 (miner.start())
* 계정도 무기한 해제해 놓아야 했다.

```python
pjt_dir> node src/BankV2Use.js
Account0: 0x21c704354D07f804baB01894e8B4eB4E0EBA7451
Balance0 before: 10334000001089349947223
Balance1 before: 184999843394000029999
Contract bal before deposit: 2222 this.bal:2222
Contract bal after deposit: 3333 this.bal:3333    <--- deposit 완성되고 출력. 3시간 정도 소요. 다음 거래를 또 마이닝 시작.
```

그러면서 다음 거래가 대기 상태로 들어가게 된다.
```
geth> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
    gas: 100000,
    gasPrice: 1000000000,
    hash: "0xbcc6e0efa1fe95f41f7f8d3bb0faf44b576ebe997d9c137c77ae7debfc417a94",
    input: "0x27d8ad88000000000000000000000000778ea91cb0d0879c22ca20c5aea6fbf8cbeed480",
    nonce: 277,
    r: "0xcbb527b3d5877bac9f9cad5d7732bc473f2877ce72a7500d4b188c3859fb7cd5",
    s: "0x39624c19700bfaae7aac49e905a89ebf39fa15f606bec7761b492ca9a75c174a",
    to: "0x0d1fc28d91167110b7728a4f20e07bc4308a78f9",
    transactionIndex: null,
    v: "0x66",
    value: 555
}]
```

단말에서의 변화이다.
망치모양의 아이콘이 나타난다. 마이닝이 완료되었고, 이 경우 보상이 주어진다. 실제 네트워크에서는 거의 불가능한 상황이겠다.

```
INFO [05-20|16:21:55.522] Successfully sealed new block            number=49277 sealhash=40c4c6…7b42c1 hash=6ed243…5149dc elapsed      =2h49m43.483s
10037 INFO [05-20|16:21:56.054] 🔨 mined potential block                  number=49277 hash=6ed243…5149dc <--- 마이닝 완료. 보상.
10038 INFO [05-20|16:22:00.854] Submitted transaction        <--- 다음 거래 시작            fullhash=0xbcc6e0efa1fe95f41f7f8d3bb0faf44b576ebe997d9c137c77ae7      debfc417a94 recipient=0x0D1Fc28D91167110B7728a4f20E07BC4308A78f9
```

#### 윈도우 웹소켓 

다음은 윈도우에서 실행한다. 차이점은 다음과 같다. 매우 빠르게 1분 정도도 걸리지 않아 완료된다.
- password를 입력해 주지 않아도 ok
- ws는 기본만 gethNow.bat에 ```--ws --wsport 8446 --wsorigins "*"```를 추가한다.

* 왼쪽 윗 화면 geth attach
    - 복수 거래를 처리하기 위해 miner.start()
    - pendingTransactions로 처리가 되고 있는지 확인한다. 현재 gas비가 100000인 forwardTo가 처리되고 있다.
* 좌하화면 gethNow.bat
    - miner.start()를 실행하니 매우 빠르게 comit new mining work -> sealed new block -> mined block 이 실행되고 있다
* 우상화면 BankV2Ws 리스너
    - BankV2WS.js를 실행시켜 놓았다 (우하화면보다 하루 전)
    - 다른 우하화면에서 Event가 발생하자 동시에 같이 이벤트가 발생되고 있다.
* 우하화면 BankV2Ws 이벤트 발생기 (BankV2WS.js 수정본 실행)
    - 우상화면과 동일하게 BankV2WS.js를 실행한다.
    - 이벤트가 성공적으로 발생되고 있다. 우상화면에서도 동시에 Event fired가 발생한다.
    - (BankV2WS.js 수정본 실행되어 출력결과가 우상화면과 다소 차이가 있다.
        - Balance1 차이가 (끝자리만 비교) 7216 - 6661 = 555 (리눅스에서는 0, 여기서는 512가 출력되었다. 동일한 문법인데 차이가 있다)

![alt text](figures/9_BankV2UseWs.png "3 screens: geth_gethAttach_nodeWsListener_nodeWsEventFire")


```python
[파일명: src/BankV2Use.js]
var Web3=require('web3');
//var web3 = new Web3(new Web3.providers.HttpProvider("http://117.16.44.45:8345"));
var web3 = new Web3(new Web3.providers.WebsocketProvider('ws://117.16.44.45:8446'));
var _abiArray=[{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_receiver","type":"address"}],"name":"forwardTo","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[],"name":"widthdrawAll","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"inputs":[],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"to","type":"address"},{"indexed":false,"name":"amount","type":"uint256"}],"name":"Sent","type":"event"}];

async function doIt() {
    var bank = new web3.eth.Contract(_abiArray, '0x0d1fc28d91167110b7728a4f20e07bc4308a78f9');
    var event = bank.events.Sent({fromBlock: 0}, function (error, result) {
        if (!error) {
            console.log("Event fired: " + JSON.stringify(result.returnValues));
        }
    });

    const accounts = await web3.eth.getAccounts();
    console.log("Account0: " + accounts[0]);
    const balanceBefore = await web3.eth.getBalance(accounts[0]);
    console.log("Balance0 before: " + balanceBefore);

    
    bank.methods.getBalance().call().then(console.log);
    bank.methods.deposit(1111).send({from: accounts[0],gas:80000,value:1111});
    bank.methods.getBalance().call().then(console.log);

    const value = await bank.methods.forwardTo(accounts[1]).send({from: accounts[0],gas:100000,value:555});
    //const value = await _test.methods.forwardTo().send({from: accounts[0], gas: 364124, gasPrice: '1000000000'})
        //.then(function(value) {console.log("---> myFunction called " + JSON.stringify(value.events.Sent.returnValues));});

    bank.methods.getBalance().call().then(console.log);  
    
    console.log("---> forwardTo called " + JSON.stringify(value.events.Sent.returnValues));
    const balanceAfter = await web3.eth.getBalance(accounts[0]);
    console.log("Balance after: " + balanceAfter);
    console.log("Balance diff: " + (balanceBefore - balanceAfter));
}
doIt()
```

```python
geth> eth.getTransactionReceipt("0xb3de97ccaeb95a04de838a87dc732d3e19f564306e4790f064a7ac0bb490ff2c")
{
  blockHash: "0xcf4a891a8a779712a2ad570dc84895614c5340d3d7aaf3183133fd616b816cb8",
  blockNumber: 49272,
  contractAddress: "0x0d1fc28d91167110b7728a4f20e07bc4308a78f9",
  cumulativeGasUsed: 336852,
  from: "0x21c704354d07f804bab01894e8b4eb4e0eba7451",
  gasUsed: 336852,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  root: "0x7ef2e50aa73b17ae2f0d15516cdf8317c1b1619684df51a4333427cb15230c2f",
  to: null,
  transactionHash: "0xb3de97ccaeb95a04de838a87dc732d3e19f564306e4790f064a7ac0bb490ff2c",
  transactionIndex: 0
}
> miner.start();admin.sleepBlocks(1);miner.stop()
null
```


```python
pjt_dir> node src/BankV2Use.js
Account0: 0x21c704354D07f804baB01894e8B4eB4E0EBA7451
Balance0 before: 10314000001089349950000
Result { '0': '0', '1': '0' }
Result { '0': '0', '1': '0' }
Event fired: {"0":"0x21c704354D07f804baB01894e8B4eB4E0EBA7451","1":"0x778ea91cb0D0879c22CA20c5AEa6FbF8CBEeD480","2":"555","from":"0x21c704354D07f804baB01894e8B4eB4E0EBA7451","to":"0x778ea91cb0D0879c22CA20c5AEa6FbF8CBEeD480","amount":"555"}
---> forwardTo called {"0":"0x21c704354D07f804baB01894e8B4eB4E0EBA7451","1":"0x778ea91cb0D0879c22CA20c5AEa6FbF8CBEeD480","2":"555","from":"0x21c704354D07f804baB01894e8B4eB4E0EBA7451","to":"0x778ea91cb0D0879c22CA20c5AEa6FbF8CBEeD480","amount":"555"}
Balance after: 10319000001089349948334
Balance diff: -4999999999998689000
Result { '0': '1111', '1': '1111' }

```

```python
pjt_dir> node src\BankV2UseWS.js
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Account0: 0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0
Balance0 before: 24464013749999999990559
Result { '0': '4444', '1': '4444' }
Result { '0': '4444', '1': '4444' }
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Event fired: {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
---> forwardTo called {"0":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","1":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","2":"555","from":"0xC8Ea4C4e655F8152aDC075a649AA7ec35564C7C0","to":"0x4fa2C7caac80A8518264d263BDB5ed74f1A6F398","amount":"555"}
Result { '0': '8888', '1': '8888' }
Balance after: 24469013749999999983895
Balance diff: -4999999999996592000
```

## 연습문제

Hello 컨트랙을 수정하여, 인사하는 대상을 지정해보자.
* 멤버변수에는 toWhom을 지정한다.
* 멤버함수에는 아래 함수를 구현한다. 일부러 함수의 입출력 인자는 생략하니 자신이 생각해서 구현한다.
```
function sayHello() view public returns() 입력: 없다, 출력: Hello smu (bytes 또는 string로 출력)
function setWhom() public 입력: 인사하는 대상 (toWhom을 지정, bytes 또는 string로 입력)
function getWhom() view public returns() 출력: 인사하는 대상 (toWhom을 읽음, bytes 또는 string로 출력)
```

문자열 합성을 하려면, 이전에는 abi.encodePacked()를 함수를 사용했으나 버전 0.8.10부터 bytes.concat() 또는 string.concat() 함수를 사용하면 된다.

각 각 2개의 컨트랙을 구현한다 - HelloInString.sol, HelloInBytes.sol.
* (1) REMIX에서 실행하고, 각 함수의 로그 결과를 붙여넣기한다 (입력, 출력, gas가 보이도록).
* (2) HelloInString과 HelloInBytes의 배포할 때와 sayHello() 함수의 gas를 비교하세요.


Hello - 48 65 6c 6c 6f


```python
for character in "Hello":
    print(character, character.encode('utf-8').hex())
```

    H 48
    e 65
    l 6c
    l 6c
    o 6f



```python
b"\x48\x65\x6c\x6c\x6f".decode('UTF-8')
```




    'Hello'




```python
b"\x68\x65\x6c\x6c\x6f\x20\x20".decode('utf-8')
```




    'hello  '




```python
//SPDX-License-Identifier: GPL3.0
pragma solidity ^0.8.0;

contract HelloInBytes {
    bytes toWhom;
    function sayHello() view public returns(bytes memory) {
        bytes memory hello = bytes("Hello "); // or 0x776f726c64; //hello
        return bytes.concat(hello, toWhom);
    }
    function setWhom(bytes memory whom) public {
        toWhom = whom;
    }
    function getWhom() view public returns(bytes memory) {
        return toWhom;
    }
}
```


```python
//SPDX-License-Identifier: GPL3.0
pragma solidity ^0.8.0;
contract HelloInString {
    string toWhom;
    function sayHello() view public returns(string memory) {
        //return "Hello " + toWhom; //not this way
        //return string(abi.encodePacked("Hello ", toWhom)); //string concat
        return string.concat("Hello ", toWhom); //ok from 0.8.12
    }
    function setWhom(string memory whom) public {
        toWhom = whom;
    }
    function getWhom() view public returns(string memory) {
        return toWhom;
    }
}
```

## 연습문제: 배열

컨트랙 '학생'을 구현하세요.
컨트랙의 이름은 'StudentsCrud.sol'로 명명하세요.
학생들을 배열로 구성하고, 다음 기능이 제공되도록 하자.
* 입력: '학번', '이름', '등록여부'를 입력한다.
* 검색: 조건에 해당하는 항목을 찾기 위해서는 반복문이 필수적이다.
그러나 gas비용이 급증할 수 있으므로, 배열의 인덱스로 검색하도록 하자.
* 삭제: 배열에서는 어떤 항목의 데이터를 지우기 위해서는 검색이 필요하다.
그러나 역시 gas비용이 급증할 수 있으므로, 배열의 인덱스에 해당하는 항목을 지우기로 한다.
해당 항목을 지우고 나서는, 그 항목을 제거하고 배열의 크기도 같이 줄어야 한다.

함수의 시그니처는 다음과 같다.
```python
* 입력함수 - function insert(uint n, string memory sn, bool e) public
* 첫 데이터 조회 - function getFirstStudent() public view returns(uint, string memory, bool)
* 검색함수 - function findBy(uint8 index) view public returns(uint, string memory, bool)
* 삭제함수 - function remove(uint index) public
* 배열크기 조회함수 - function getLength() view public returns(uint)
* 삭제함수 - function pop() public
```

컨트랙을 구현하고, REMIX에서 DEPLOY한 후, 제공되는 함수버튼을 활용하여 다음 작업을 수행해보자.
항목별 답안으로는 수행결과가 출력된 REMIX 단말의 로그를 붙여넣기 하면 된다.
(선택적으로 노드에서 배포하고 실행해도 된다)

(1) 아래 항목을 입력
```python
201711111,"kim",false
201711112, "park", true
201711113, "lee", false
201711114, "lim", false
```

(2) 2번째 데이터 201711112 제거

(3) 배열크기 조회 (4개 항목에서 1개가 제거되었으니 3이 출력)

(4) 2 번째 데이터조회를 조회한다. 20171112는 삭제되어서 출력할 수 없고, 다른 데이터 항목이 출력.

(5) 첫 데이터 조회


### 질문: index로 삭제하는 함수를 index에 맨마지막 배열의 요소를 덮어씌우고 pop을 하였는데, 이 방법보다 좋은 방법이 없을까요?

여러 방법이 존재할 수 있어요. 설계의 문제라는 의미입니다.
- 삭제하고 비워놓고나
- 삭제하고 비우지 않고 삭제표시만
- 삭제하고, 그 자리에 다른 데이터를 이동 <- 한수가 사용한 방법
- 순차적 이동


```python
%%writefile src/StudentsCrud.sol
/**
 * @autor jsl
 * @since 20210520
 * Student array with struct
 */
pragma solidity ^0.6.0;
contract StudentsCrud {
    struct Student {
        uint num;
        string name;
        bool isEnrolled;
    }
    Student[] sDB;
    //memory only for string type
    //201711111,"kim",false
    function insert(uint n, string memory sn, bool e) public {
        Student memory s = Student(n, sn, e);
        sDB.push(s);
    }
    function getFirstStudent() public view returns(uint, string memory, bool) {
        return (sDB[0].num, sDB[0].name, sDB[0].isEnrolled);
    }
    function findBy(uint8 index) view public returns(uint, string memory, bool) {
       return (sDB[index].num, sDB[index].name, sDB[index].isEnrolled);
    }
    function remove(uint index) public {
        if (index < sDB.length) {
            sDB[index]=sDB[sDB.length-1];
            sDB.pop(); //just pop() worked
        }
    }
    function getLength() view public returns(uint) {
        return sDB.length;
    }
    function pop() public {
        sDB.pop();
    }
    //only supported in ABIEncoderV2 (use pragma exprimental ABIEncoderV2)
    /*function getAllStudents()  public view returns (Student[] memory) {
        return sDB;
    }*/
}
```

    Overwriting src/StudentsCrud.sol



```python
%%writefile src/ArrayCRUDTest.sol
pragma solidity ^0.6.0;
contract ArrayCRUDTest{
    uint[] sDB = [1,2,3,4,5];   //sDB.length - 5
    function removeAtIndex(uint index) public {
        if (index < sDB.length) {
            sDB[index]=sDB[sDB.length-1];
            pop();
        }
    }
    function findBy(uint8 index) view public returns(uint) {
       return (sDB[index]);
    }
    function insert(uint s) public {
        //Student memory s = Student(n, sn, e);
        sDB.push(s);
    }
    function getLength() public view returns (uint) {
        return sDB.length;
    }
    function pop() public {
        sDB.pop();
    }
}
```

    Writing src/ArrayCRUDTest.sol



```python

```
