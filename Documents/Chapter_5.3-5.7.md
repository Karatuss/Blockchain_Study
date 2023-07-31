## 5.3 해싱 기초
**해싱**은 특수하게 정의된 **해시 함수(hash function)** 을 이용해 임의의 길이를 가진 데이터를 고정된 길이를 가진 데이터로 매핑하는 프로세스다.  
``` 해시 = 해시함수(하나 또는 다수의 데이터 아이템) ```  

#### 해시 특징
   - 데이터베이스나 이미지 같은 어떠한 데이터도 고정된 길이의 해시로 간결하게 표현 가능
   - 256비트 데이터 아이템과 강력한 해시 함수를 가지고 **충돌 가능성이 없는** 주소 스페이스를 만들 수 있음
   - *충돌 가능성이 없다*는 말은 두 개의 해시가 **같은 값을 가질 확률이 거의 없다**는 의미
     
![KakaoTalk_20230801_030345322](https://github.com/TwoPair/Blockchain_Study/assets/101495241/b6ee5903-0598-43d8-afdd-8739aee45725)

---

### 5.3.1 문서의 디지털 서명
문서에 디지털 서명을 하기 위해 해시 함수를 이용해 해시값을 계산 -> 해시 값에 디지털 서명을 한 후 원본 문서와 전달 -> 받은 수신자는 원본 문서의 해시값을 송신자의 공개키를 이용해서 디지털 서명을 검증 -> 송신자가 이 문서에 서명했다는 것을 확인

### 5.3.2 분산 장부에 해시 데이터 저장
크기가 큰 문서들로 인해 블록체인에 과부화가 걸리지 않도록 오직 문서의 해시값(문서를 나타내는)만 저장

### 5.3.3 이더리움 블록 헤더의 해시

아래 그림은 이더리움 블록 헤더 다이어그램이다.  

여기에는 트랜잭션 기록, 변화하는 상태, 로그, 리시트 등 여러 정보들이 포함되어 있는데, 이를 해시 함수로 바꿔준 후 블록 헤더에 기록한다.  

또한 이전 블록 헤더의 해시를 포함하여 블록체인의 변조 불가능성을 더욱 강화한다.

![KakaoTalk_20230801_031501576](https://github.com/TwoPair/Blockchain_Study/assets/101495241/fbadce13-22f5-4f00-a54c-1e85f9ba6b88)

### 5.3.4 솔리디티 해싱 함수

솔리디티는 **SHA256**, **Keccak**, **RIPEMD160**이라는 세 가지 해싱 함수를 제공한다.  

Keccak 해시 함수를 위한 솔리디티 코드  
```
pragma ­­solidity >=0.4.22 <=0.6.0;

 

contract Khash {

  bytes32 public hashedValue;

    function hashMe( uint value1, bytes32 password) public {

	    hashedValue = keccak256(abi.encodePacked(value1, password));
    }
}
```

---

## 5.4 해싱 애플리케이션

솔리디티 문서에 나온 블라인드 경매 예제를 검토해 볼 것이다.
```
어떤 예술 작품을 블라인드 경매로 판매해야 한다. 여러 작품을 경매로 판매할 수 있겠지만, 여기서는 오직 한 작품만 판매한다고 가정하자.
이 작품이 팔린 후 다른 작품을 추가할 수는 있다. 수혜자는 경매의 여러 단계{Init, Bidding, Reveal, Done}을 관리한다.
수혜자가 경매를 시작하고 나면, Bidding 단계 동안 입찰자는 한 번에 하나씩 입찰을 한다.
여기서 입찰은 보안이 보장되며 프라이빗하게 이루어져야 한다.
수혜자를 포함한 다른 사람은 각 입찰의 내용을 볼 수 없다. 일정 시간 후, 수혜자는 Reveal 단계로 진행시킨다.
이제 입찰자는 각자의 입찰 내용을 공개하고, 수혜자는 전체 입찰 내용을 공개해서 가장 높은 가격을 제시한 입찰자와 입찰가를 찾아낸다.
수혜자는 단계를 Done으로 변경하고 경매를 종료한다. 수혜자의 어카운트로 최고가 입찰가를 전송한다.
경매에 떨어진 입찰자는 그들의 예치금을 출금할 수 있으나 낙찰받은 입찰자는 입찰한 금액을 출금할 수 없다.
```

### 5.4.1 블라인드 경매 설계

우선 데이터 구조를 정의하는데 설계 원칙 2와 설계 원칙 3을 사용한다.  

컨트랙트 다이어그램(설계 원칙 4)와 유한 상태 머신(FSM, 설계 원칙 5)을 사용해 설계를 형상화한다.  

이후 스마트 컨트랙트에 수정자를 사용한다.

![KakaoTalk_20230801_041640468_01](https://github.com/TwoPair/Blockchain_Study/assets/101495241/aae5a8d9-99cd-4b5a-b787-b0df0af5b333)

위 과정을 통해 나온 컨트랙트 다이어그램과 상태 변화 다이어그램이다.

![KakaoTalk_20230801_041640468](https://github.com/TwoPair/Blockchain_Study/assets/101495241/e87a6250-5c63-472f-95f1-ad8f101b482d)

### 5.4.2 블라인드 경매 스마트 컨트랙트

블라인드 경매를 위한 데이터
```
pragma solidity >=0.4.22 <=0.6.0;

// 입찰 정보
contract BlindAuction { 

    struct Bid {                   
        bytes32 blindedBid;
        uint deposit;
    }

    // 상태는 수혜자에 의해 설정된다(경매 상태 정보)
    enum Phase {Init, Bidding, Reveal, Done}  
    Phase public state = Phase.Init; 

    // 컨트랙트 배포자가 수혜자다
    address payable beneficiary; //owner
    //주소당 오직 1회만 입찰  
    mapping(address => Bid) bids;  


    address public highestBidder; 
    uint public highestBid = 0;
    //최고가 입찰자의 정보   
    
    mapping(address => uint) depositReturns;
    //낙찰 탈락자의 예치금 반환

    //수정자
    //경매 단계를 위한 수정자
    modifier validPhase(Phase reqPhase) 
    { require(state == reqPhase); 
      _; 
    } 
   //수혜자를 확인하는 수정자
   modifier onlyBeneficiary()
   { require(msg.sender == beneficiary); 
      _;
   }
```

### 5.4.3 프라이버시와 보안 측면

블라인드 경매에서 Bidding 단계 동안, 입찰 내용은 시큐어 할 뿐만 아니라 프라이빗해야한다.  

무작위 대입(*brute force*)공격을 피하기 위해 논스(*nonce*)나 시크릿 패스워드를 두 번째 파라미터로 사용하여 방어할 수 있다.  

어떤 값을 Keccak으로 해시할 때 두 번째 시크릿 패스워드를 함께 묶어서 계산하면, 무작위 대입 공격에 대해 안전해진다.

![KakaoTalk_20230801_042822303](https://github.com/TwoPair/Blockchain_Study/assets/101495241/8f139185-b76c-4cd2-a549-9f7aed2636b1)

**:exclamation: 설계 원칙 7**
```
✔ 일회용 시크릿 패스워드를 사용해 함수의 파라미터를 시큐어 해싱을 함으로써 프라이버시와 보안을 확보한다.
```
```
///constructor가 수혜자를 설정
constructor(  ) public {    
        beneficiary = msg.sender;
        state = Phase.Bidding;
    }

    function changeState(Phase x) public onlyBeneficiary {
        if (x < state ) revert();
        state = x;
    }

    ///블라인드 경매 함수
    function bid(bytes32 blindBid) public payable validPhase(Phase.Bidding)
    {  
        bids[msg.sender] = Bid({
            blindedBid: blindBid,
            deposit: msg.value
        });
    }

    ///reveal()이 블라인드 입찰을 확인
    function reveal(uint value, bytes32 secret) public   validPhase(Phase.Reveal){
        uint refund = 0;
            Bid storage bidToCheck = bids[msg.sender];
            if (bidToCheck.blindedBid == keccak256(abi.encodePacked(value, secret))) {
            refund += bidToCheck.deposit;
            if (bidToCheck.deposit >= value) {
                if (placeBid(msg.sender, value))
                    refund -= value;
            }}
            
        msg.sender.transfer(refund);
    }

    ///placeBid()는 내부(internal) 함수다
    function placeBid(address bidder, uint value) internal 
            returns (bool success)
    {
        if (value <= highestBid) {
            return false;
        }
        if (highestBidder != address(0)) {
            // Refund the previously highest bidder.
            depositReturns[highestBidder] += highestBid;
        }
        highestBid = value;
        highestBidder = bidder;
        return true;
    }


    // 낙찰 탈락 입찰 출금
    function withdraw() public {   
        uint amount = depositReturns[msg.sender];
        require (amount > 0);
        depositReturns[msg.sender] = 0;
        msg.sender.transfer(amount);
        }
    
    
    //경매를 종료하고 수혜자에게 최고가 입찰액을 전송
    function auctionEnd() public  validPhase(Phase.Done) 
    {
        beneficiary.transfer(highestBid);
    }
}
```

## 5.6 베스트 프랙티스
  - 개인키 - 콩개키 암호학은 어카운트를 고유하게 식별하는 데 필수적인 역할을 한다. 신용카드를 안전하게 보관해야 하는 것처럼, 블록체인에 있는 자산의 보안을 위해 개인키를 잘 보호해야 한다.
  - 탈중앙화 블록체인 기반 시스템에서 전송되는 데이터(파라미터)의 프라이버시와 보안을 확보하기 위한 해싱 기법을 잘 활용하자.
