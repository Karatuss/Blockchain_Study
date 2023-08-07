# Chapter 6. 온체인과 오프체인 데이터

일반적으로 블록체인상에 저장하는 모든 데이터는 온체인 데이터이고, 그 이외의 것은 오프체인 데이터다.

> 💡 **온체인 데이터(on-chain data)란?**<br>
> 블록체인 기반 애플리케이션이 구동한 트랜잭션이 생생한 정보와 블록체인의 구축(materialization)에 쓰인 아이템의 집합이다.


## 6.1 온체인 데이터

**이더리움 프로토콜이 가진 요소들**
- 블록체인 헤더(6A)는 블록의 속송을 저장한다.
- 트랜잭션(6B)은 블록에 저장된 Tx의 내용을 저장한다.
- 리시트(6C)는 블록에 기록된 Tx의 실행 결과를 저장한다. 모든 트랜잭션은 리시트를 가지고 있고, 그림 6.2는 이 일대일 관계를 보여준다.
- 총합 글로벌 스테이트(6D)는 모든 블록체인상의 스마트 컨트랙트와 다른 일반 어카운트의 데이터값 또는 현재 스테이트를 저장하고, 이것을 이용하는 Tx를 컴펌할 때 갱신한다.

![KakaoTalk_20230807_141149402](https://github.com/TwoPair/Blockchain_Study/assets/101495241/a43d6aee-b9a1-428f-bdb4-8c3b1ebadfc3)

위 그림에서 볼 수 있듯 6A 가 6B, 6C, 6D의 해시를 모두 포함한다.  

또한 블록 헤더, Tx 트리, 리시트 트리는 블록 단위 데이터 구조이며, *새로운 블록을 추가할 때마다 새로운 블록 헤더, Tx 트리, 리시트 집합의 인스턴트를 생성*한다.   

그에 반해 스테이트 트리는 블록체인 단위 구조이다. 스테이트 트리는 블록체인에 어떤것이 어떻게 일어났는지에 대한 히스토리 전체를 기록한다. 이 스테이트는 *Tx를 실행할 때마다 계속 변한다.*


## 6.2 블라인드 경매 유스 케이스

블라인드 경매 유스 케이스를 만들기 위해 검토해야 할 것들
- 이벤트는 무엇인가?
- 어떻게 이벤트를 정의하는가?
- 이벤트를 어떻게 생성하는가?
- UI를 통해 사용자에게 알림을 주기 위해  Tx 리시트에 있는 이벤트 로그를 어떻게 사용하는가?

### 6.2.1 온체인 이벤트 데이터

솔리디티는 이벤트를 정의하고 발생시키는 기능을 제공하는데, 이벤트는 파라미터를 가질 수도 아닐 수도 있다.

> 💡 **이벤트(events)란?**<br>
> 스마트 컨트랙트 함수의 실행에서 어떤 조건이나 플래그(flag)가 발생했다는 것을 알리기 위해 함수에서 발생시키는 알림이다.



이벤트 정의 문구
```
event NameOfEvent (parameters);
```

솔리디티에서 이벤트 이름은 **대문자**로 시작하고 캐멀(camel)케이스를 사용한다.  

**emit** 명령어를 이용하여 이벤트를 발생시킨다
```
event BiddingStarted();
event RevealStarted();

emit BiddingStarted();
emit RevealStarted();
```


### 6.2.2 이벤트를 가진 블라인드 경매

- 이벤트는 블록의 리시트 트리에 기록하는 전형적인 온체인 데이터이다.
- 이벤트 로그는 거의 **리얼타임 수준의 응답**에도 사용할 수 있지만, 오프라인에서 **인덱싱해서 조회**하거나 온체인 **데이터를 분석**할 때도 사용한다.
- 이벤트를 호출하면 로그를 발생시키는데, 이 로그에 온체인 인덱스를 걸고 토픽으로 조회할 수 있다.

#### 이벤트에 접근하기 위한 방법
- 리스너를 이용 (push 방법)
- 리시트 로그를 사용 (pull 방법)
- 트랜잭션 리시트를 사용하는 기법

*우리는 트랜잭션 리시트를 이용한다*

**블라인드 경매 컨트랙트 다이어그램**

![KakaoTalk_20230807_170122807](https://github.com/TwoPair/Blockchain_Study/assets/101495241/66832f52-c9dd-44b6-a586-d45e054d4f9c)

블라인드 경매 스마트 컨트랙트는 세 개의 이벤트를 가지고 있다.
- 경매의 종료를 알리는 AuctionEnded
- 경매의 Bidding 단계를 알리는 BiddingStarted
- 경매의 Reveal 단계를 알리는 RevealStarted


> BlindAuction.sol
```
pragma solidity >=0.4.22 <=0.6.0;


contract BlindAuction {
    struct Bid {
        bytes32 blindedBid;
        uint deposit;
    }

    // 단계는 오직 수혜자(경매 운영자)만이 설정할 수 있다
    // Enum-uint mapping:
    // Init - 0; Bidding - 1; Reveal - 2; Done - 3
    enum Phase {Init, Bidding, Reveal, Done}
    // Owner
    address payable public beneficiary;
    // Keep track of the highest bid,bidder
    address public highestBidder;
    uint public highestBid = 0;
    // Only one bid allowed per address
    mapping(address => Bid) public bids;
    mapping(address => uint) pendingReturns;

    Phase public currentPhase = Phase.Init;
    // Events
    event AuctionEnded(address winner, uint highestBid);
    event BiddingStarted();
    event RevealStarted();
    event AuctionInit();
    // Modifiers
    modifier validPhase(Phase phase) {
        require(currentPhase == phase, "phaseError");
        _;
    }
    modifier onlyBeneficiary() {
        require(msg.sender == beneficiary, "onlyBeneficiary");
        _;
    }

    constructor() public {
        beneficiary = msg.sender;
        // advancePhase();
    }

    function advancePhase() public onlyBeneficiary {
        // If already in done phase, reset to init phase
        if (currentPhase == Phase.Done) {
            currentPhase = Phase.Init;
        } else {
            // else, increment the phase
            // Conversion to uint needed as enums are internally uints
            uint nextPhase = uint(currentPhase) + 1;
            currentPhase = Phase(nextPhase);
        }

        // Emit appropriate events for the new phase
        if (currentPhase == Phase.Reveal) emit RevealStarted();
        if (currentPhase == Phase.Bidding) emit BiddingStarted();
        if (currentPhase == Phase.Init) emit AuctionInit();
    }

    function bid(bytes32 blindBid) public payable validPhase(Phase.Bidding) {
        require(msg.sender != beneficiary,'beneficiaryBid');    // Beneficiary should not be allowed to place bids
        bids[msg.sender] = Bid({blindedBid: blindBid, deposit: msg.value});
    }

    function reveal(uint value, bytes32 secret) public validPhase(Phase.Reveal) {
        require(msg.sender != beneficiary,'beneficiaryReveal');
        uint refund = 0;
        Bid storage bidToCheck = bids[msg.sender];

        if (bidToCheck.blindedBid == keccak256(abi.encodePacked(value, secret))) {
            refund += bidToCheck.deposit;
            if (bidToCheck.deposit >= value*1000000000000000000) {
                if (placeBid(msg.sender, value*1000000000000000000))
                    refund -= value * 1000000000000000000;
            }
        }
        msg.sender.transfer(refund);
    }

    // This is an "internal" function which means that it
    // can only be called from the contract itself (or from
    // derived contracts).
    function placeBid(address bidder, uint value) internal returns (bool success)
    {
        if (value <= highestBid) {
            return false;
        }
        if (highestBidder != address(0)) {
            // Refund the previously highest bidder.
            pendingReturns[highestBidder] += highestBid;
        }

        highestBid = value;
        highestBidder = bidder;
        return true;
    }

    // Withdraw a non-winning bid
    function withdraw() public {
        uint amount = pendingReturns[msg.sender];
        if (amount > 0) {
            pendingReturns[msg.sender] = 0;
            msg.sender.transfer(amount);
        }
    }

    // Send the highest bid to the beneficiary and
    // end the auction
    function auctionEnd() public validPhase(Phase.Done) {
        if(address(this).balance >= highestBid){
            beneficiary.transfer(highestBid);
        }
        emit AuctionEnded(highestBidder, highestBid);
    }

    function closeAuction() public onlyBeneficiary {
        selfdestruct(beneficiary);
    }
}
```

위 코드는 전체 코드이다. 우리는 이번장에서 이벤트와 관련된 코드만 살펴볼 것이다.   

#### advancePhase 함수

advancePhase() 함수는 현재 경매 진행상태를 바꿔주는 함수이다.  

이전 사용한 changeState() 함수는 변조 불가성 요구 사항으로 인해 경매가 끝나게 된다면 다시 경매를 시작할 수 없다.  

> changeState()
```
function changeState(Phase x) public only Beneficiary {
    if (x < state) revert();
        state = x;
}
```

advancePhase() 함수는 Done 단계 이후에 다시 다음 경매를 위해 Init 상태로 순환시켜 changeState()의 문제를 해결한다.  

또한, BiddingStarted와 RevealStarted 이벤트를 발생시킨다.

#### 이벤트 로그 온체인 데이터

다음 사진은 advancePhase 함수를 실행했을 때 생성되는 로그의 내용이다.
![KakaoTalk_20230807_172847240](https://github.com/TwoPair/Blockchain_Study/assets/101495241/591acbae-4eed-43f0-889f-a1cd9dba7a0a)

- "from"주소는 스마트 컨트랙트를 배포한 주소를 나타낸다.
- "topic"은 호출 함수(이 경우에는 advancePhase()) 시그너처(헤더)의 16진수 값이다.
- "event"는 발생한 이벤트명(BiddingStarted)이고, "args"는 그 파라미터값을 보여준다.

*advancePhase 버튼을 여러 번 클릭하면 상태가 다시 Init으로 돌아간 것을 확인할 수 있을 것이다.*


### 6.2.3 웹 UI를 가진 테스팅


## 6.3 오프체인 데이터: 외부 데이터 소스

- 오프체인 데이터는 온체인 데이터와 다르게 오프체인 데이터의 타입과 사용은 블록체인 프로토콜이 결정하지 않는다.
- 오프체인 데이터는 테이터 소스의 타입과 포맷의 제한이 없으며 애플리케이션 종속적이다.

![KakaoTalk_20230807_234128770](https://github.com/TwoPair/Blockchain_Study/assets/101495241/c487cf37-5f74-44d1-b2b9-cb04f76616fb)\


전통적인 데이터베이스를 스마트 컨트랙트로 정의하지 않는다.  
하지만 스마트 컨트랙트는 온체인 저장을 쉽게 하므로 오프체인에서 무엇이 일어났는지를 쉽게 파악하기 위한 함수와 데이터만을 스마트 컨트랙트에 담도록 설계해야 한다.

> 💡 **설계원칙 8**<br>
> 스마트 컨트랙트는 규칙의 강제, 준수, 규정, 출처, 리얼타임 알림을 위한 로그, 타임스탬프 활동 정보와 오프라인 오퍼레이션에 대한 메시지에 필요한 함수와 데이터만을 포함하도록 설계한다.
