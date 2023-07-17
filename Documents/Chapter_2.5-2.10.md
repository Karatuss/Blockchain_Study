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
 <img width="364" alt="그림2 8" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/f91db6e4-6af2-4679-84c0-31cbe5c5718c">

## 2.6 탈중앙화 항공사 시스템 유스 케이스
**항공사 시스템 컨소시엄 ASK(airline system consortium) 블록체인**은 참여 항공사 간에 좌석의 P2P트랜잭션을 가능하게 해주는 시스템
즉, ASK는 서로 코드를 공유하지 않는 항공사들이 좌석을 P2P로 거래할 수 있는 시장

### 2.6.1 ASK 정의
- 각 항공사는 ASK 트랜잭션에서 좌석의 정산에 사용될, 미리 설정된 최소 escrow금액을 예치함으로써 ASK에 합류할 수 있음.
- ASK의 문제 설정, 이슈, 블록체인 기반 솔루션, 결과물 등을 다음 사분면 차트로 요약

<img width="574" alt="그림2 9" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/2a5cf210-2c6d-47ce-bd45-b272202442ad">

- 이 유스 케이스는 기본적인 항공사 간의 P2P 좌석 거래로 허용 범위를 한정함.
- **이 조회는 소프트웨어 어플리케이션에 의해 프로그래밍적으로 호출되며, 중개자가 필요없다.**
- 애플리케이션의 요청은 직접 항공사로 보내진다.
  
### 2.6.2 오퍼레이션 단계
- 두 항공사 A와 B는 반드시 서로 알고 있는 관계일 필요가 없고, 서로 간에 전통적인 신뢰범위를 벗어나서 운영하고 있다고 가정함.
- 즉, 이들은 code-sharing과 alliances같은 통상적인 비즈니스 파트너십을 가지고 있지 않다고 가정함.
- 그럼 어떻게 서로를 신뢰하며 트랜잭션을 처리할 것인가?
- 탈중앙화 시스템에서 확인자, 검증자, 보관자로서의 역할을 하는 블록체인을 이해하기 위해 다음 그림의 오퍼레이션을 분석하자.
<img width="578" alt="그림2 10" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/52511a65-d1b2-4a9d-b0dd-55593e3fac32">

1. 고객이 항공사 A에 예약해 두었던 좌석을 바꾸어 달라는 요청을 한다.
2. 항공사 A의 에이전트나 애플리케이션은 ASK 컨소시엄 회원들에게 공유된 스마트 컨트랙트 로직을 사용해 이 요청을 확인하고 검증한다.
3. 확인 후에 이 요청 Tx는 컨펌되고 변조 불가능한 분산 장부에 기록된다. 이제 컨소시엄에 속해 있는 모든 주체는 정당한 요청이 생성되었음    을 알게 된다.
4. 가장 단순한 설계에서는 항공사 A의 에이전트가 확인 및 검증된 요청(VVRequest)을 항공사 B에게 보낸다.
5. 항공사 B의 에이전트 또는 애플리케이션이 자사의 데이터 베이스를 검색해 가용성을 확인한다.
6. 항공사 B의 에이전트는 컨소시엄의 공동 이해관계와 공유된 규칙을 확인하고 검증하는 공유 스마트 컨트랙트 로직을 통해서 응답을 한다.
  

