# 3 신뢰와 무결성을 위한 기법

:exclamation: 신뢰와 무결성은 탈중앙화 시스템에서 특히 중요함.

:bulb: 블록체인이 탈중앙화 시스템에서의 신뢰와 무결성 문제를 해결 가능함.

---

## 3.1 신뢰와 무결성의 핵심
  ### 3.1.1 신뢰

  - 신뢰는 시스템에 참여하는 피어 참여자의 신용에 대한 척도


    <img width="281" alt="그림3-1" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/6136b9ac-38c7-46f7-bbd9-2f8149f7e027">


  - 그림 3.1의 첫 번째 사분면 차트에서 신뢰의 근본적 요소인 확인(la)과 검증(lb)에 의해 신뢰를 확립한다.

  ✔ 확인과 검증의 차이
  - 확인(la)은 일반적인 규칙에 관한 것으로 문제 영역의 일반적인 또는 전역적인 요구 조건을 다루는 것
  - 검증(lb)은 애플리케이션 또는 데이터에 특정한 규칙에 관한 것


  ### 3.1.2 무결성
  :bulb: 무결성이란 대상 시스템의 참여자와 그들이 보낸 메시지, 데이터, 오퍼레이션의 정확성에 관한 것

  :bulb: **블록체인에서 무결성**이란 데이터의 보안과 프라이버시, 트랜잭션의 기밀성을 보장하는 것

  - 무결성은 그림 3.1의 두 번째 사분면 차트처럼 노드에 있는 피어 참여자를 고유하게 식별하는 방법부터 다룸.
  - 탈중앙화 시스템인 블록체인에서는 블록체인 어카운트 주소를 사용해 참여자를 고유하게 식별함.
  - 무결성의 요소들(아이덴티티, 보안, 프라이버시, 기밀성)은 프라이빗-퍼블릭 키 쌍 개념을 주로 기반하고 있음.
  
---


## 3.2 전자 민주주의 문제
:exclamation: **전자 민주주의**란 인도의 간단한 디지털 신분 카드에서부터 에스토니아의 전자 영주권에 이르기까지 매우 다양한 것을 포함함.

:large_orange_diamond: 온라인 투표 시스템 기획
- 사람들은 다수의 제안 중 하나에만 투표
- 의장은 투표할 수 있는 사람을 등록하고, 등록된 사람만 제안된 선택지 중 하나에만 투표함.
- 의장의 표는 가중치를 두어서 두 표로 계산함.
- 투표 과정은 네 개의 상태(Init, Regs, Vote, Done)를 거치며, 이 상태에 따라 다른 각각 다른 오퍼레이션(initialize, register, vote, count votes)을 수행한다.
  
  ### 3.2.1 솔루션 설계
  1. **유스 케이스 다이어그램**을 설계하기 위해 설계 원칙 1, 2, 3을 적용하자. 이 다이어그램을 이용해 사용자, 데이터 애셋, 트랜잭션을 식별하자.
  2. 설계 원칙 4를 사용해 데이터, 수정자, 확인과 검증을 위한 규칙, 함수를 정의하는 **컨트랙트 다이어그램**을 설계하자.
  3. 컨트랙트 다이어그램을 사용해 솔리디티로 **스마트 컨트랙트**를 개발하자.
  4. 리믹스 IDE에서 스마트 컨트랙트를 컴파일하고 배포, 테스트하자.
   
  ### 3.2.2 유스 케이스 다이어그램
  투표 문제를 UML 유스 케이스 다이어그램을 이용해 분석하자.

  
  <img width="269" alt="그림3-3" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/55a70809-b4bc-4e15-a850-f5bc67e05cd6">


  **✔ 주요 행위자와 역할**
  - **의장** _chairperson_ 은 투표자를 등록하며, 자신도 스스로 등록하고 투표 할 수 있다.
  - **투표자** _voters_ 는 투표를 한다.
  - **불특정 다수** _anybody_ 누구나 투표 과정의 승자나 결과를 요청 할 수 있다.
  
  ### 3.2.3 코드의 점진적 개발
  - 스마트 컨트랙트의 개발 과정을 잘 보여줄 수 있도록 투표 문제를 해결하기 위한 코드를 네 개의 점진적 단계를 거쳐 개발할 것임.
   
    1. **BallotV1** - 스마트 컨트랙트의 데이터 구조를 정의하고 테스트
    2. **BallotV2** - constructor와 투표 상태를 변화시키기 위한 함수를 추가
    3. **BallotV3** - 스마트 컨트랙트의 다름 함수와 신뢰 구축을 위한 솔리디티 기능을 보여주기 위한 수정자 추가
    4. **BallotV4** - 신뢰 요소인 require(), revert(), assert()와 함수 접근 수정자를 추가

   ### 3.2.4 사용자, 애셋, 트랜잭션
   - 사용자 : 의장, 투표자, 불특정 다수
   - 데이터 애셋 : 투표자들이 투표할 제안들
    
      **BallotV1.sol**
      ``` 
      pragma solidity >=0.4.2 <=0.6.0;

      contract BallotV1{
          struct Voter{
              uint weight;
              bool voted;
              uint vote;
          }
          struct Proposal{
              uint voteCount;
          }

          address chairperson;
          mapping(address => Voter) voters;
          Proposal[] proposals;

          enum Phase {Init, Regs, Vote, Done}
          
          Phase public state = Phase.Init;

      }
      ```


   ### 3.2.5 유한 상태 머신 다이어그램
    - 그림 3.3 투표 유스 케이스 다이어그램은 오직 정적인 정보만을 보여줘서 투표 과정에서 일어나는 다이내믹한 타이밍과 상태 변화를 보여줄 방법이 없음.
    - 시스템 역동성을 표현할 수 있는 또 다른 다이어 그램이 필요함.
    - 시스템 다이내믹스를 보여주기 위해 UML 유한 상태 머신 또는 FSM 다이어그램을 사용함.
    
      **:exclamation: 설계 원칙 5**

      ```✔ 스마트 컨트랙트 내에서 일어나는 상태 변화와 같은 시슽메 역동성을 표현하기 위해 유한 상태 머신 UML 다이어그램을 활용한다.```
      
    - **FSM의 구성 요소**
       - **states** - 시작 상태와 하나 이상의 종료 상태를 가지고 있는데, 이 두 상태는 관례적으로 이중 원으로 표시한다.
       - **transitions** - 하나의 상태에서 다른 상태로 변화하는 것을 말한다.
       - **inputs** - 상태 변화를 일으키는 입력값을 의미한다.(T=0, T+10일, T+11일)
       - **outputs** - 상태 변화 동안 출력되는 것. 없을 수도 있고 하나 이상일 수도 있다. 예를 들면 표시된 상태에 따라 등록(Regs), 투표(Vote), 집계(Done) 등이 일어난다.
      - 투표 과정을 나타내는 네 개의 단계 또는 상태를 정의함.

        <img width="260" alt="그림3-5" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/a3608961-754a-4f2f-92ca-f7b3739d1a28">

      
      **BallotV2.sol**
      ```
      pragma solidity >=0.4.2 <=0.6.0;
      contract BallotV2{
          struct Voter{
              uint weight;
              bool voted;
              uint vote;
          }
          struct Proposal{
              uint voteCount;
          }
          
          constructor (uint numProposals) public{
              chairperson = msg.sender;
              voters[chairperson].weight = 2;
              for(uint prop = 0; prop < numProposals; prop++){
                  proposals.push(Proposal(0));
              }
          }
          function changeState(Phase x) public{
              if (msg.sender != chairperson) revert();
              if (x < state) revert();
              state = x;
          }

          address chairperson;
          mapping(address => Voter) voters;
          Proposal[] proposals;

          enum Phase {Init, Regs, Vote, Done}  
          Phase public state = Phase.Init;
      }
      ```

   ### 3.2.6 신뢰 중개
   - 솔리디티가 제공하는 신뢰 요구 조건을 다루는 언어적 기능
     - modifier는 확인해야 할 접근 통제 규칙을 명시하고 신뢰와 프라이버시를 구축하기 위해 누가 데이터와 함수에 대한 통제권을 갖는지를 관리함.
     - require(condition) 선언 은 파라미터로 전달된 조건을 검증하고 만일 실패할 경우 함수를 중단함.
     - revert() 선언은 트랜잭션을 중단시키고, 블록체인에 기록되는 것을 막아줌.
     - assert(condition) 선언문은 변수의 조건이나 함수의 실행 과정에서 데이터를 검증하는데, 만일 실패할 경우 트랜잭션을 되돌림.
  
   ### 3.2.7 수정자를 정의하고 사용하기
   - 수정자 구문
     - 이름과 파라미터를 가진 헤더 라인이 있다.
     - require 문 안에 체크해야 할 조건들을 명시하는 본문이 있다.
     - 마지막으로 _; 라인이 있는데, 이것은 수정자의 조건을 통과하고 난 후 실행해야 할 코드를 나타낸다. 즉, 수정자가 보호하고 있는 코드 내용을 상징한다.
      ```
      modifier validPhase(Phase reqPhase){
          require(state == reqPhase);
          _;
      }
      ```
    - 수정자 정의를 왜 함수 정의로부터 분리해 내는가?
  
      ▶ 신뢰와 무결성 구축을 위해 스마트 컨트랙트가 강제하는 규칙을 명확하게 보여줄 수 있도록 확인, 검증, 예외를 다루는 부분을 분리하는 것
    - **modifier 키워드**는 스마트 컨트랙트 감사인이 모든 규칙이 미리 잘 설정되어 있고, 기대한 바대로 사용된느지 확인하는데도 도움이 된다. 
    
  - 수정자 사용
    ```
    function register(address voter) public validPhase(Phase.Regs){
          if(msg.sender != chairperson || voters[voter].voted) revert();
          voters[voter].weight = 1;
          voters[voter].voted = false;
      }
    ```
    - 신뢰 구현자(중개)로서 수장자의 사용은 **설계 원칙 6**이 된다.
    
  **:exclamation: 설계 원칙 6**
      
    ```  ✔ 스마트 컨트랙트에서 규칙과 조건을 명시하는 수정자를 사용함으로써 신뢰 중개를 위한 확인과 검증을 구현한다. 통상적으로 확인은 참여자에 대한 일반적인 규칙을 담당하고, 검증은 애플리케이션에 특정한 데이터를 체크하는 역할을 맡는다.```

   
   ### 3.2.8 수정자를 포함한 컨트랙트 다이어그램
   :exclamation: 투표 스마트 컨트랙트를 코딩하기 위해 필요한 데이터 구조와 함수를 정의하는 컨트랙트 다이어그램을 만들어 보자.

  <img width="284" alt="그림3-6" src="https://github.com/TwoPair/Blockchain_Study/assets/51403372/603b0eba-064e-4ac5-a985-209c9bd5bee4">

  - 데이터 정의 아래에 **modifier validPhase** 라는 수정자 정의가 있음
  - 수정자 validPhase가 Phase reqPhase 파라미터를 가지고 있다는 것에 주목하자.
  - 다른 함수에서 validPhase 수정자가 세 개의 다른 파라미터(Regs, Vote, Done)를 가지고 호출되었다는 것에 주목하자.
  
    **▶ 수정자의 유연성과 재활용성을 보여줌**



   ### 3.2.9 완성한 전체 코드
    ```
    pragma solidity >=0.4.2 <=0.6.0;
    contract BallotV3{
        struct Voter{
            uint weight;
            bool voted;
            uint vote;
        }
        struct Proposal{
            uint voteCount;
        }

        address chairperson;
        mapping(address => Voter) voters;
        Proposal[] proposals;

        enum Phase {Init, Regs, Vote, Done}
        Phase public state = Phase.Init;
        
        modifier validPhase(Phase reqPhase){
            require(state == reqPhase);
            _;
        }

        constructor(uint numProposals) public{
            chairperson = msg.sender;
            voters[chairperson].weight = 2;
            for(uint prop = 0; prop < numProposals; prop++){
                proposals.push(Proposal(0));
            }
            state = Phase.Regs;
        }

        function changeState(Phase x) public {
            if(msg.sender != chairperson) {revert();}
            if(x < state) revert();
            state = x;
        }

        function register(address voter) public validPhase(Phase.Regs){
            if(msg.sender != chairperson || voters[voter].voted) revert();
            voters[voter].weight = 1;
            voters[voter].voted = false;
        }
        
        function vote(uint toProposal) public validPhase(Phase.Vote){
            Voter memory sender = voters[msg.sender];

            if(sender.voted || toProposal >= proposals.length) revert();
            sender.voted = true;
            sender.vote = toProposal;
            proposals[toProposal].voteCount += sender.weight;
        }
        function reqWinner() public validPhase(Phase.Done) view returns(uint winningProposal){
            uint winningVoteCount = 0;
            for(uint prop = 0; prop < proposals.length; prop++){
                if (proposals[prop].voteCount > winningVoteCount){
                    winningVoteCount = proposals[prop].voteCount;
                    winningProposal = prop;
                }
            }
        }

    }
    ```
    
    - 스토리지 vs 메모리 변수
      - Voter sturct : 함수 안에서 로컬 변수로 구조체를 정의할 때, memory 타입인지 storage 타입인지를 명시적으로 선언해 주어야 함. 
      - 솔리디티에서 변수는 **storage(지속되고 블록고에 보관됨)** 또는 **memory(일시적이고 블록에 저장되지 않음)** 로 정의할 수 있음.

    - 함수 상세 설명
      - constructor() - 스마트 컨트랙트를 배포할 때 constructor 함수를 호출함. 이 함수는 파라미터로 투표할 제안수를 받는다. 데이터 요소들과 투표 단계의 상태를 초기화한다.
      - changeState() - 이 함수는 투표 상태를 매칭하는 단계로 변화시킨다. 오직 의장만 실행할 수 잇고 파라미터는 올바른 순서가 되어야한다.
      - register() - 이 함수는 오직 의장만 실행할 수 있다. 만일 voted 불 변수가 true이거나 state가 Phase.Regs가 아니면 함수는 중단된다.
      - vote() - 이 함수는 오직 Phase.Vote에서만 실행할 수 있다. '1인 1표' 규칙의 검증과 제안 수를 확인할 수 있다. 
      - reqWinner() - 이 함수는 투표를 집계하고 투표수로 어느 제안이 승리했는지 식별한다. 실제 프로덕션 환경에서는 최적화해야 한다. 체인에 기록되지 않는 'view' 함수임에 주목하자.
   ---

  ## 3.3 테스팅
   - **긍정 테스트** - 유효한 입력값이 주어졌을 때 스마트 컨트랙트는 기대한 바대로 올바르게 작동한다.
   - **부정 테스트** - 유효하지 않은 입력값이 주어졌을 때 스마트 컨트랙트는 확인과 검증을 통해 오류를 잡아내고 함수를 중단시킨다.

  ---

  ## 3.4 수정자, require(), revert() 사용하기
  - 함수를 실행하는 데 다수의 규칙이 필요한 경우
    - 함수 호출에 다수의 규칙(액세스 수정자)을 적용할 수 있다.
  - 하나의 함수 내에서 구문의 실행 과정 중이나 후에 어떤 조건을 검사해야 하는 경우
    - require() 조항을 이용해 조건을 충족하지 못했을 때 함수를 중단시킬 수 있다.
    - 투표자가 중복 투표를 하지 못하도록 검증하기 위해 revert()를 사용했다.
  - 수정자는 표시된 순서대로 실행하기 때문에 만일 앞 수정자의 실행이 뒤쪽에 있는 수정자의 판단에 영향을 미칠 수 있다면, 순서가 뒤바뀌지 않도록 주의해야 한다.
  ---

  ## 3.5 assert() 선언
  - **assert()** - 특수 내장 함수, 어떤 함수 내에서의 연산 과정에서 특정한 조건을 충족했는지 여부를 확인해 줌.
  **BallotV4.sol**
  ```
  pragma solidity >=0.4.2 <=0.6.0;
  contract BallotV3{
      struct Voter{
          uint weight;
          bool voted;
          uint vote;
      }
      struct Proposal{
          uint voteCount;
      }

      address chairperson;
      mapping(address => Voter) voters;
      Proposal[] proposals;

      enum Phase {Init, Regs, Vote, Done}
      Phase public state = Phase.Init;
      
      modifier validPhase(Phase reqPhase){
          require(state == reqPhase);
          _;
      }

      modifier onlyChair(){
          require(msg.sender == chairperson);
          _;
      }

      constructor(uint numProposals) public{
          chairperson = msg.sender;
          voters[chairperson].weight = 2;
          for(uint prop = 0; prop < numProposals; prop++){
              proposals.push(Proposal(0));
              state = Phase.Regs;
          }
      }

      function changeState(Phase x) public {
          require(x > state);
          state = x;
      }

      function register(address voter) public validPhase(Phase.Regs) onlyChair{
        require(! voters[voter].voted);
          voters[voter].weight = 1;
      }
      
      function vote(uint toProposal) public validPhase(Phase.Vote){
          Voter memory sender = voters[msg.sender];
          require(!sender.voted);
          require(toProposal < proposals.length);
          sender.voted = true;
          HYPERLINK "http//sender.vote/"sender.vote = toProposal;
          proposals[toProposal].voteCount += sender.weight;
      }
      function reqWinner() public validPhase(Phase.Done) view returns(uint winningProposal){
          uint winningVoteCount = 0;
          for(uint prop = 0; prop < proposals.length; prop++){
              if (proposals[prop].voteCount > winningVoteCount){
                  winningVoteCount = proposals[prop].voteCount;
                  winningProposal = prop;
              }
          }
          assert(winningVoteCount>=3)
      }

  }
  ```
  - 함수 **assert()**와 **require()**는 조건을 체크한다는 점과 체크에 실패할 경우 트랜잭션을 되돌린다는 점에서도 유사함.
  - **require()**는 변숫값의 리미트를 체크하는 것과 같은 일반적인 검증에 사용하고 체크에 실패할 가능성이 있다고 가정한다.
  - **assert()**는 예외를 다루기 위한 것으로 보통 실패하지 말아야 한다고 가정한다.
