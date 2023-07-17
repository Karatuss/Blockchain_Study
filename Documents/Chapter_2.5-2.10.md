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

```Remix
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
 3. 확인 후에 이 요청 Tx는 컨펌되고 변조 불가능한 분산 장부에 기록된다. 이제 컨소시엄에 속해 있는 모든 주체는 정당한 요청이 생성되었     음을 알게 된다.
 4. 가장 단순한 설계에서는 항공사 A의 에이전트가 확인 및 검증된 요청(VVRequest)을 항공사 B에게 보낸다.
 5. 항공사 B의 에이전트 또는 애플리케이션이 자사의 데이터 베이스를 검색해 가용성을 확인한다.
 6. 항공사 B의 에이전트는 컨소시엄의 공동 이해관계와 공유된 규칙을 확인하고 검증하는 공유 스마트 컨트랙트 로직을 통해서 응답을 한다.
 7. 확인 후에 응답 Tx는 컨펌되고 변조 불가능한 분산 장부에 기록된다. 이제 컨소시엄의 모든 참여자가 응답이 보내졌음을 알 수 있다.
 8. 항공사 B는 이 응답(VVResponse)을 항공사 A의 에이전트에게 보낸다.
 9. 항공사 A는 자기의 데이터베이스를 업데이트하고 변화된 것을 기록한다.
10. 항공사 B의 에이전트는 고객에게 좌석과 기타 상세 정보를 보낸다.
    *여기서 항공사 B는 항공사 A에게 자신이 가진 정보를 보내는 것이 아니고 직접 고객에게 보낸다는 점에 주목*
12. 지급은 참여 항공사들이 공유하고 있는 스마트 컨트랙트에 에스크로해 두었거나, 예치했던 금액을 이용한 P2P 트랜잭션을 통해 정산한다.      지급 정산은 시스템 내의 다른 오퍼레이션에서도 포함할 수 있지만, 모든 공유 스마트 컨트랙트로 처리하여 장부에 기록한다. 스마트 컨트     랙트의 로직이 자동으로 정산을 수행하는 것이다.


## 2.7 항공사 스마트 컨트랙트
**설계원칙**
1. 테스트 체인에서 스마트 컨트랙트를 코딩, 개발, 배포하기 전에 우선 설계부터 한다. 또한, 프로덕션 블록체인에 배포하기 전에 철저한 테     스트를 거쳐야 한다. 왜냐하면 스마트 컨트랙트는 변조 불가능하기 때문이다.
2. 시스템 사용자와 유스 케이스를 정의한다. 사용자란 행위와 입력값을 발생시키고, 설계하고 있는 해당 시스템으로부터 그 출격값을 받는 주    체다.
3. 데이터 에셋, 피어 참여자, 그들의 역할, 강제할 규칙, 설계하고 있는 시스템에 기록해야 할 트랜잭션을 정의힌다.
4. 컨트랙트 이름, 데이터 애셋, 함수, 함수의 실행과 데이터 접근을 위한 규칙을 정의하는 컨트랙트 다이어그램을 작성한다.

다음 그림은 설계원칙 1,2,3을 이용해서 ASK 유스 케이스와 컨트랙트 다이어그램을 보여줌.

<img width="586 alt="그림2 11" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/b33fa683-f6b4-44d5-a986-0d37f9f606e5">

### 2.7.1 피어 참여자, 데이터 애셋, 역할, 트랜잭션, 규칙
- 참여자, 그들의 역할, 그들이 컨트롤하고 있는 애셋, 관련 트랜잭션을 구분하는 것은 표준적인 설계 과정의 공통된 첫 번째 단계다.

#### 피어 참여자
- **피어 참여자**라는 말은 중개자 없이 P2P 형식으로 참여자들이 상호작용 한다는 점을 강조하기 위함.
- ASK 유스 케이스에서는 항공사를 대신에서 활동하는 에이전트가 피어 참여자임.
- ASK 컨소시엄 관리 기관은 **컨소시엄 의장(chairperson)** 으로 불리는 감시자를 둔다.
- 감시자/의장은 참여사들 간에 정기적으로 순환하여 임명됨.
- 공유 데이터는 어떤 중앙화된 데이터 베이스가 아닌 공유된 블록체인의 분산원장에 저장함.

#### 데이터 애셋
- ASK 유스 케이스에서 데이터 애셋은 피어 참여자들에 의해 처리되고 있는 비행기 좌석과 자금임.
- 탈중앙화 시스템의 가장 근본적 원리는 그들의 애셋을 피어 참여자 자신들이 보유하고 있다는 것.
  
#### 역할
- 항공사를 대신해서 에이전트는 에스크로/예치와 register() 함수를 이용해 직접 ASK 회원으로 등록할 수 있음.
- 에이전트(회원만)는 비행기 좌석을 요청할 수 있음.
- 항공사를 대신해서 에이전트는 좌석 가용성을 확인하기 위해 자신들의 중앙화 데이터 베이스를 조회하고 응답할 수 있음.
- 피어 에이전트는 만일 좌석 이용이 가능할 경우 서로 지급 정산을 할 수 있음.
- 컨소시엄의 의장은 회원을 탈퇴시키고 남은 예치금을 반환할 수 있는 단독 권한을 가지고 있음.

#### 트랜잭션
- 여러 오퍼레이션과 데이터베이스 같은 다양한 하부 시스템 간의 상호작용이 이루어지는데 여기서 확인, 검증, 컨펌하고 모든 참여자에 의해    저장되어야 할 오퍼레이션을 **트랜잭션(Txs)** 라고 부름.

#### 규칙
- 그림 2.11의 컨트랙트 다이어그램에는 **수정자(modifier)** 라는 카운터 유스 케이스에서 없었던 새 요소가 있음.
- 수정자는 스마트 컨트랙트의 특수한 요소로서, 데이터와 함수에 대한 접근을 통제하기 위한 규칙을 나타냄.
- 오직 **유효한 회원(onlyMember)** 만이 시스템에서 트랜잭션을 생성할 수 있고,
  오직 **의장(onlyChairperson)** 만이 항공사를 회원에서 탈퇴시킬 수 있다.
  
:heavy_exclamation_mark: **수정자(modifier)** 는 검증과 확인을 위한 규칙을 명시적으로 표현하는 것을 가능하게 하는 언어적 기능이다. 이 수정자는 검증과 확인을 하는 게이트키퍼이기 때문에 신뢰 확보를 위해 특히 중요하다.

### 2.7.2 항공사 스마트 컨트랙트 코드
```Remix
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.18;
contract Airlines{
    address chairperson;
    struct details{
        uint escrow;
        uint status;
        uint hashOfDetails;
    }

    mapping (address=>details) public balanceDetails;
    mapping (address=>uint) membership;

    modifier  onlyChairperson{
        require(msg.sender==chairperson);
        _;
    }
    modifier onlyMember{
        require(membership[msg.sender]==1);
        _;
    }

    constructor() payable {
        chairperson==msg.sender;
        membership[msg.sender]=1;
        balanceDetails[msg.sender].escrow=msg.value;
    }

    function register() public payable {
        address AirlineA = msg.sender;
        membership[AirlineA]=1;
        balanceDetails[msg.sender].escrow=msg.value;
    }

    function unregister (address payable AirlineZ) onlyChairperson public{
        membership[AirlineZ]=0;
        AirlineZ.transfer(balanceDetails[AirlineZ].escrow);
        balanceDetails[AirlineZ].escrow=0;
    }

    function request (address toAirline, uint hashOfDetails) onlyMember public{
        if(membership[toAirline]!=1){
            revert();
        }
        balanceDetails[msg.sender].status=0;
        balanceDetails[msg.sender].hashOfDetails=hashOfDetails;
    }
    function response (address fromAirline, uint hashOfDetails, uint done) onlyMember public{
        if(membership[fromAirline]!=1){
            revert();
        }
        balanceDetails[msg.sender].status=done;
        balanceDetails[msg.sender].hashOfDetails=hashOfDetails;
    }
    function settlePayment (address payable toAirline) onlyMember payable public{
        address fromAirline=msg.sender;
        uint amt=msg.value;

        balanceDetails[toAirline].escrow=balanceDetails[toAirline].escrow+amt;
        balanceDetails[fromAirline].escrow=balanceDetails[fromAirline].escrow-amt;

        toAirline.transfer(amt);
    }


}
```

### 2.7.3 ASK 스마트 컨트랙트의 배포와 테스팅

#### 테스트 계획 설명

#### 테스트 설명


## 2.8 스마트 컨트랙트 설계 고려 사항

## 2.9 베스트 프랙티스

## 2.10 요약



