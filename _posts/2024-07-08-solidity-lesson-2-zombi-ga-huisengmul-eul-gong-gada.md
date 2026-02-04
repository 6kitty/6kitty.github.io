---
layout: post
title: "Solidity Lesson 2: 좀비가 희생물을 공격하다"
categories: [Cryptography & Blockchain]
tags: [Solidity, Smart Contract, Blockchain]
last_modified_at: 2024-07-08
---

1. 매핑과 주소  
address: 각 계정마다 가주고 있는 고유 식별자  
주소는 특정 유저(혹은 스마트 컨트랙트)가 소유한다 ???  
mapping: 키-값 저장소  

```javascript
// 금융 앱용으로, 유저의 계좌 잔액을 보유하는 uint를 저장한다: 
mapping (address => uint) public accountBalance;
// 혹은 userID로 유저 이름을 저장/검색하는 데 매핑을 쓸 수도 있다 
mapping (uint => string) userIdToName;
```

2. msg.sender  
솔라디티에 있는 특정 전역 변수 중 하나  
함수를 호출한 유저(혹은 스마트 컨트랙트)의 주소를 가리키는 변수 msg.sender  

3. require  
특정 조건이 참이 아니면 에러 메세지 출력 후 실행 중단  

```javascript
function sayHiToVitalik(string _name) public returns (string) {
  // _name이 "Vitalik"인지 비교한다. 참이 아닐 경우 에러 메시지를 발생하고 함수를 벗어난다
  // (참고: 솔리디티는 고유의 스트링 비교 기능을 가지고 있지 않기 때문에 
  // 스트링의 keccak256 해시값을 비교하여 스트링 값이 같은지 판단한다)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 참이면 함수 실행을 진행한다:
  return "Hi!";
}
```

4. 상속  
그냥 우리가 아는 그 상속...  

```javascript
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

5. import  
다른 파일을 import...  

```javascript
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

./는 동일한 폴더(알죠?)  

6. Storage vs. Memory  
변수를 저장할 수 있는 공간으로 위 두 개가 있음  
Storage는 영구적임 -> 보통 상태 변수  
Memory는 임시적임 -> 보통 함수 내 선언한 변수  
컴퓨터의 하드 디스크와 RAM으로 생각하면 굳  

자동 선언되는데 구조체와 배열의 경우에 지정(명시)  

```javascript
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 꽤 간단해 보이나, 솔리디티는 여기서 
    // `storage`나 `memory`를 명시적으로 선언해야 한다는 경고 메시지를 발생한다. 
    // 그러므로 `storage` 키워드를 활용하여 다음과 같이 선언해야 한다:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...이 경우, `mySandwich`는 저장된 `sandwiches[_index]`를 가리키는 포인터이다.
    // 그리고 
    mySandwich.status = "Eaten!";
    // ...이 코드는 블록체인 상에서 `sandwiches[_index]`을 영구적으로 변경한다. 

    // 단순히 복사를 하고자 한다면 `memory`를 이용하면 된다: 
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...이 경우, `anotherSandwich`는 단순히 메모리에 데이터를 복사하는 것이 된다. 
    // 그리고 
    anotherSandwich.status = "Eaten!";
    // ...이 코드는 임시 변수인 `anotherSandwich`를 변경하는 것으로 
    // `sandwiches[_index + 1]`에는 아무런 영향을 끼치지 않는다. 그러나 다음과 같이 코드를 작성할 수 있다: 
    sandwiches[_index + 1] = anotherSandwich;
    // ...이는 임시 변경한 내용을 블록체인 저장소에 저장하고자 하는 경우이다.
  }
}
```

7. 함수 접근 제어자  
private는 상속하는 컨트랙트가 접근할 수 없음  
internal로 수정~  
internal: private에서 상속 컨트랙트에서도 접근이 가능하다는 점만 다름  
external: 컨트랙트 바깥에서만 호출될 수 있는 public  

```javascript
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // eat 함수가 internal로 선언되었기 때문에 여기서 호출이 가능하다 
    eat();
  }
}
```

8. 인터페이스  
우리가 소유하지 않은 컨트랙트와 상호작용하려면 인터페이스 정의  

```javascript
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

여기서 getNum 함수를 이용하여 해당 컨트랙트의 데이터를 읽고자하는 external 함수..  
LuckyNumber이라는 인터페이스 먼저 정의  

```javascript
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

컨트랙트 뼈대 느낌..  
다른 컨트랙트에서  

```javascript
contract MyContract {
  address NumberInterfaceAddress = 0xab38...
  // ^ 이더리움상의 FavoriteNumber 컨트랙트 주소이다
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 이제 `numberContract`는 다른 컨트랙트를 가리키고 있다.

  function someFunction() public {
    // 이제 `numberContract`가 가리키고 있는 컨트랙트에서 `getNum` 함수를 호출할 수 있다:
    uint num = numberContract.getNum(msg.sender);
    // ...그리고 여기서 `num`으로 무언가를 할 수 있다
  }
}
```

위와 같이 활용, 이때 활용하는 함수는 public 또는 external로 선언되어야 함  

9. 다수의 반환값  

```javascript
function multipleReturns() internal returns (uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 다음과 같이 다수 값을 할당한다:
  (a, b, c) = multipleReturns();
}

// 혹은 단 하나의 값에만 관심이 있을 경우: 
function getLastReturnValue() external {
  uint c;
  // 다른 필드는 빈칸으로 놓기만 하면 된다: 
  (,,c) = multipleReturns();
}
```

저기 저 쉼표로 반환값만큼 체크해주면 됨