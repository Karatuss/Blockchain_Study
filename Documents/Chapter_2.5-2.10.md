## 2.5 블록체인 컨트랙트는 왜 스마트한가?

### 스마트 컨트랙트가 갖는 속성
  - 이름
  - 주소
  - 암호 화폐(이더) 잔액
  - 암호 화폐(이더)를 송금하고 수신할 수 있는 내장 기능
  - 데이터와 함수
  - 메시지를 수신하고 함수를 호출할 수 있는 내장 기능
  - 함수 실행을 계산할 수 있는 능력
  
### 스마트 컨트랙트 기능 : 어카운트 번호(주소)로 참여자를 식별함.
  - 이더리움 : EOA(externally owned accounts) 외부 소유 어카운트
              SCA(smart contract accounts) 스마트 컨트랙트 어카운트
              라는 두 가지 종류의 어카운트를 가짐.
  - EOA와 SCA 모두 이더 밸런스를 가짐. 즉, 모든 어카운트는 address와 balance라는 묵시적 속성을 가짐.
  - EOA와 SCA는 스마트 컨트랙트에 메시지를 보내서 함수를 호출할 수 있음.
    메시지는 msg.sender와 msg.value라는 두 가지의 묵시적 속성을 가짐.
    이렇게 금액을 전송받기 위해선느 함수에 payable modifier를 포함해서 선언해야 함.
- **전통적인 컴퓨팅 시스템**에서는 사용자 이름과 암호로 참여자 식별함.
  <사용자 이름, 암호> 조합은 탈중앙화 시스템에서 작동하지 않음.
  피어가 통상적인 신뢰 범위 밖에 있기 때문.

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;
contract AccountsDemo{
    address public  whoDeposited;
    uint public depositAmt;
    uint public accountBalance;

    function deposit() public payable {
        whoDeposited = msg.sender;
        depositAmt = msg.value;
        accountBalance = address(this).balance;

    }
}
```
 <img width="182" alt="그림2 8" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/f91db6e4-6af2-4679-84c0-31cbe5c5718c">


