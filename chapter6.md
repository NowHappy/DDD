# 들어가면서
## 학습 목표
모델 주도 설계의 3가지 관리 패턴을 이해하고 활용할 수 있다.
 
## 도메인 객체(= 모델) 관리의 범주
1. 생명주기 동안 모델의 무결성 유지
2. 생명주기 관리의 복잡성으로 모델이 난해해지는 것 방지
  = (문제의 복잡도를 낮추기 위해) 단순한 모델 유지

**MODEL-DRIVEN DESIGN을 실천하기 위해서**는 

> 오래 지속되며，활성 메모리 안에서만 시간을 보내지 않는

>다른 객체와 복잡한 상호의존성을 맺는

>여러 가지 상태의 변화를 겪기도 하고，갖가지 불변식이 적용되는

![image.png](/files/2586250807818343361) 
이러한 객체들의 생명 주기를 잘 관리하는 것이 중요하다.

### 무결성?
* 생명주기 동안 데이터의 정확성과 일관성을 유지하고 보증하는 것

뒤에서 불변식 유지라는 표현이 자주 나옴
**불변식이란 데이터가 변경되어도 유지되는 일관적인 규칙**
불변식이 지켜지지 않는다, 유지되지 않는다 -> 무결성 유지 X

# 관리를 위한 패턴(= 도메인 설계 구조물) 3가지
    1. AGGREGATE
    2. FACTORY
    3. REPOSITORY

패턴 -> 귀감이 되는 본 -> 지키면 좋은 규칙이라고 이해함

## 관리 범주에 따른 패턴 분류
* 무결성 유지 - AGGREGATE 모델링
* 단순한 모델 유지 - FACTORY, REPOSITORY를 설계에 추가

## 생명주기 영향 단계에 따른 패턴 분류
(중요 X, 각각 패턴에 대한 이해가 더 중요)
* 생명주기 전체 - AGGREGATE
* 생명주기의 초기 - FACTORY
* 생명주기의 중간과 마지막 - REPOSITORY

## 각 패턴의 간략한 특징(뒤에서 더 자세하게 다룸)

### AGGREGATE : 모델 형상에 대한 패턴
> 데이터 변경의 단위로 사용하는 연관 객체의 묶음

* 루트(root)와 경계(boundary)로 구성
* 소유권과 경계를 명확히 정의할 수 있음
* 모델을 엄격하게 만듬
* 객체 간의 연관관계를 단순,깔끔하게 유지
* 생명주기의 전 단계에서 무결성 유지돼야 할 범위를 표시

### FACTORY : 모델 객체 생성과 재구성에 대한 패턴
> 다른 객체를 생성하는 것인 책임인 프로그램 요소

* 복잡한 객체나 AGGREGATE를 생성(이하 '생성 객체')
* 생성 객체를 재구성
* 생성 객체를 생성하는데 필요한 지식을 캡슐화
* 생성 객체를 생성하는 일이 복잡하거나,	내부 구조를 감추고자하는 경우 사용
* 생성 객체의 불변식 유지 보장
* 생성 객체와 가장 밀접한 관계가 있는 위치(객체)에 존재해야 함
	
### REPOSITORY : 모델 객체 상태 변이(저장, 보관 또는 삭제)에 대한 패턴
> 모델에 집중하게 해주는 단순한 개념적 틀

* 저장소 관련 인프라스트럭처(기술적 복합성)를 캡슐화
* 영속 객체를 찾아 저장, 조회, 삭제 등 질의 수단을 제공

#### cf) FACTORY & REPOSITORY
앞 장에서도 보면 도메인 로직을 UML 등으로 그대로 표현하기 위해
모델을 난해하게 수정한다던가 하는 오류를 범하는 경우가 있을 수 있음.
이처럼 객체의 생명주기 관리의 복잡성으로 모델이 난해해지는 것을 방지하고
객체의 생명주기 동안 객체를 체계적이고 의미있는 단위로 조작, 관리하기 위해 사용
- AGGREGATE를 대상으로 연산을 수행
- 특정 생명주기로 옮겨가는데 따르는 복잡성을 캡술화

# 패턴 1) AGGREGATE
> 데이터 변경의 단위로 사용하는 연관 객체의 묶음

## AGGREGATE 설명에 들어가기 앞서..
### 무결성 유지가 어려운 이유
객체들 간의 연관 관계를 명확히 하는 것이 어렵기 때문

**무결성 유지가 안되는 모델 예)**
 주소를 참조하는 Person 객체(여러 객체로 구성된 한 객체)의 삭제
	-> 참조된 주소 객체 삭제시 문제점 : 같은 주소를 쓰는 사람이 있을 경우 삭제된 객체를 참조
	-> 참조된 주소 객체를 삭제하지 않을 시 문제점 : 쓰레기 주소 쌓임

### 해결책?
자동화된 가비지 컬렉션이나 DB에서 제공해주는 잠금 기법 등의 기술적인 해법을 시도할 수는 있지만
모두 임시 방편일 뿐 근본적인 원인은 모델링 설계에 있다.
이런 문제는 동일한 객체에 여러 클라이언트가 동시에 접근하는(= 경합이 높은) 시스템에서 더 심각해진다.
=> 경합이 높은 지점을 느슨하게 만드는,
불변식이 잘 지켜지는(= 무결성이 유지되는) 모델을 설계해야 한다.
* high-contention points looser
* strict invariants tighter

contention 언쟁, 논쟁
**ex)** 상품과 주문의 관계에서 상품은 여러 주문에 사용된다. => 경합이 높은 지점
(뒤 예제에 나오지만 교착 상태 등 사용에 문제가 발생할 가능성이 높은 포인트를 의미하는 것 같음)

## AGGREGATE 모델링
> 각 AGGREGATE에는 루트(root)와 경계(boundary)가 있다. 경계는 AGGREGATE에 무엇이 포함되고 포함되지 않는지를 정의한다 루트는 단 하나만 존재하며，AGGREGATE에 포함된 특정 ENTITY를 가리킨다. 경계 안의 객체는 서로 참조할 수 있지만, 경계 바깥의 객체는 해당 AGGREGATE의 구성요소 가운데 루트만 참조할 수 있다. 루트 이외의 ENTITY는 지역 식별성(local identity)을 지니며，지역 식별성은 AGGREGATE 내에서만 구분되면 된다. 이는 해당 AGGREGATE의 경계 밖에 위치한 객체는 루트 ENTITY의 컨텍스트 말고는 AGGREGATE의 내부를 볼 수 없기 때문이다.
### 예시1)  자동차 수리점에서 사용하는 소프트웨어 모델
* 사용자가 알고 싶어하는 정보
    * 네 개의 바퀴 위치를 토대로 타이어 로테이션 이력
    * 각 타이어의 주행거리와 마모 정도

한 자동차는 세상의 다른 모든 자동차와 구별되야 함 -> 자동차는 전역 식별성을 지닌 ENTITY
타이어도 식별 가능해야 함 -> 타이어도 ENTITY
해당 특정 자동차 외부의 컨텍스트에서는 타이어의 식별성에 관심 X(교체해서 재활용 센터로 타이어를 보낸 경우 추적 불가능한 쓰레기 정보가 됨) -> 타이어는 해당 특정 자동차에 종속적임
사람들은 데이터베이스에 조회해서 자동차를 찾은 다음 타이어를 일시적으로 참조할 것이다. 
따라서 자동차는 타이어도 포함하는 경계를 지닌 AGGREGATE의 루트 ENTITY로 볼 수 있다. 
![image.png](/files/2586678498206325046)

* 모델의 불변식
    * 동일한 바퀴에 대해서는 시간대가 겹쳐서는 안된다
    * 주행거리 = sum(Position.mileage)

![image.png](/files/2586686045573231620)
타이어 위치를 변경하기 위해서는 외부에서만 접근이 가능하고 자동차 entity의 rotate 메서드에서 불변식 규칙을 이행한다.

>ENTITY 와 VALUE OBJECT를 AGGREATE로 모으고 각각에 대해 경계를 정의하라. 한 ENTITY를 골라 AGGREATE의 루트로 만들고 AGGREATE경계 내부의 객체에 대 해서는 루트를 거쳐 접근할 수 있게 하라. AGGREATE 밖의 객체는 루트만 참조할 수 있게 하라. 내부 구성요소에 대한 일시적인 참조는 단일 연산에서만 사용할 목적에 한해 외부로 전달될 수 있다. 루트를 경유하지 않고는 AGGREATE의 내부를 변경할 수 없다. 이런 식으로 AGGREATE의 각 요소를 배치하면 AGGREATE 안의 객체와 전체로서의 AGGREATE의 상태를 변경할 때 모든 불변식을 효과적으로 이행할 수 있다.

### 예시2) 구매주문
* 모델의 불변식
    * 주문품목의 수량에 따른 품목 금액의 총 합 <= 구매주문 허용한도
* 기존 모델
![image.png](/files/2586694617614696852)

기존 모델에서 단순하게 lock 전략을 사용하여 무결성을 유지해보자 
![image.png](/files/2586693583546296513)
-> 우선 주문품목만 lock
![image.png](/files/2586697663663261331)
![image.png](/files/2586696721602891372)
-> 구매주문의 무결성 유지 X
-> 구매주문도 lock
![image.png](/files/2586698388767094050)
-> (또 다른 문제점)구매주문을 commit 하기 전 품목의 가격을 변경해도 불변식 위반이 이러남
-> 품목도 같이 lock
![image.png](/files/2586699110589524940)
-> 그러면 이전과 같은 문제는 해결되겠지만 사용자 편의를 방해할 수 있음
![image.png](/files/2586700956311399049)

#### 모델 개선(feat. AGGREGATE)
*  업무 지식을 반영해 모델을 개선 
    * 품목은 여러 구매주문에서 사용된다(경합이 높음).
    * 구매주문보다는 품목의 변경이 더 적다.
    * 품목 가격에 대한 변경 내역이 반드시 기존 구매주문에 전해질 필요는 없다. 이는 구매주 문의 상태와관련된 가격 변경 시점에 좌우된다.

![image.png](/files/2586690524689548402)
=> 위 문제들이 해결되고 불변식 보장

# 패턴 2) FACTORY
> 자신의 책임이 다른 객체를 생성하는 것인 책임인 프로그램 요소

객체 본연의 책임은 객체가 생성된 후 객체가 수행하는 것들임
>어떤 객체를 생성하는 것이 그 자체로도 주요한 연산이 될 수 있지만 복잡한 조립 연산은 생성 된 객체의 책임으로 어울리지 않는다. 이런 책임을 클라이언트에 두면 이해하기 힘든 볼품없는 설계가 만들어질 수 있다. 클라이언트에서 직접 필요로 하는 객체를 생성하면 클라이언트 설계 가 지저분해지고 조립되는 객체나 AGGREGATE의 캡슐화를 위반하며，클라이언트와 생성된 객체의 구현이 지나치게 결합된다.

![image.png](/files/2586703671487779145)

>복잡한 객체와 AGGREGATE의 인스턴스를 생성하는 책임을 별도의 객체로 옮겨라. 이 객체 자체는 도메인 모델에서 아무런 책임도 맡지 않을 수도 있지만 여전히 도메인 설계의 일부를 구성한다. 모든 복잡한 객체 조립 과정을 캡슐화하는 동시에 클라이언트가 인스턴스화되는 객체 의 구상 클래스를 참조할 필요가 없는 인터페이스를 제공하라. 전체 AGGREGATE를 하나의 단 위로 생성해서 그것의 불변식이 이행되게 하라.

## FACTORY와 FACTORY의 위치 선정
1. 이미 존재하는 AGGREGATE에 요소를 추가하는 경우
추천위치 : 해당 AGGREGATE의 루트에 METHOD로 만들어라
이유 : AGGREGATE의 무결성을 보장하는 책임을 루트(엔티티)가 담당하고，동시에 모든 외부 클라이언 트에게서 AGGREGATE의 내부 구현을 감출 수 있다.

![image.png](/files/2586705433818634256)

2. 한 객체의 데이터나 규칙이 객체를 생성하는데 매우 크게 영향을 주는 경우 
추천위치 : 생성된 객체를 소유하지는 않지만 다른 객체를 만들어내는 것과 밀접한 관련이 있는 특정 객체에 METHOD로 만들어라
이유 : 어떤 다른 곳에서 해당 객체를 생성할 때 생산자의 정보를 필요로 하는 것을 줄일 수 있다. 

![image.png](/files/2586705577626908867)

3. 구상 구현체나 생성 과정의 복잡성과 같은 것을 감춰야 하거나,
특정 AGGREGATE안의 어떤 객체가 FACTORY를 필요로 하는데
 AGGREGATE 루트가 해당 FACTORY가 있기에 적절한 곳이 아닌 경우
추천위치 : 독립형(standalone) FACTORY로 만들어라

![image.png](/files/2586705674714112746)

## 생성자만으로 충분한 경우
FACTORY는 실제로 다형성을 활용하지 않는 간단한 객체를 이해하기 어렵게 만들 수 있다.
타협점을 고려해봤을 때 다음과 같은 상황에서는 공개 생성자(public constructor)를 사용하는 편이 좋다.
* 클래스가 타입인 경우. 클래스가 어떤 계층구조의 일부를 구성하지 않으며，인터페이스를 구현하는 식으로 다형적으로 사용되지 않는 경우.
* 클라이언트가 STRATEGY를 선택하는 한 방법으로서 구현체에 관심이 있는 경우
* 클라이언트가 객체의 속성을 모두 이용할 수 있어서 클라이언트에게 노출된 생성자 내에서 객체 생성이 중첩되지 않는 경우
* 생성자가 복잡하지 않은 경우
* 공개 생성자가 FACTORY와 동일한 규칙을 반드시 준수해야 하는 경우. 이때 해당 규칙은 생성된 객체의 모든 불변식을 중족하는 원자적인 연산이어야 한다.

## 인터페이스 설계
* FACTORY를 잘 설계하기 위한 두 가지 기본 요건
	* 각 생성 방법은 원자적(atomic)이어야 한다.
	*  생성된 클래스보다는 생성하고자 하는 타입으로 추상화돼야 한다. 

## 불변식 로직의 위치
FACTORY는 자신이 생성한 객체의 불변식 유지가 보장되도록 설계되어야 함.
따라서 불변식 로직 위치가 FACTORY인 경우가 많음
### 불변식 로직 위치가 FACTORY인 경우
* 생성 객체에 들어 있는 복잡한 요소를 줄이는 이점이 있음
예 ) ENTITY의 식별 속성을 할당하는데 적용되는 불변식
=> 객체 생성 후 해당 로직을 쓸 일이 없으므로 생성될 때, FACTORY에 위치하는게 좋음

### 불변식 로직 위치가 FACTORY의 생성 객체인 경우
* 간혹 생성 객체 자체에 불변식 검사를 위임하는 것이 최선일 때가 있으므로
불변식 로직이 어디에 존재하는게 좋을지 한 번 더 생각해 봐야 한다.
예 ) 다른 도메인 객체에 속한 FACTORY METHOD의 경우

## ENTITY FACTORY와 VALUE OBJECT FACTORY 
* 차이점 3가지
    1. VALUE OBJECT FACTORY의 연산은 생성 객체에 대해 풍부한 설명을 곁들여야 한다. (이유 : 생성 객체가 완전히 최종적인 형태임)
    2. ENTITY FACTORY는 유효한 AGGREGATE 를 만들어 내는 데 필요한 필수 속성만 받아들이는 경향이 있다. 불변식에서 세부사항을 필요로 하지 않는다면 그와 같은 세부사항은 나중에 추가해도 된다.
    3. ENTITY FACTORY는 식별성 할당에 대해 반드시 신경써야 한다.

## 저장된 객체의 재구성
> 지금까지 FACTORY는 특정 객체의 생명주기의 초반에 해당하는 부분을 다뤘다. 언젠가 객체 는 대부분 데이터베이스에 저장되거나 네트워크상으로 전송될 것이며，현재의 데이터베이스 기 술 가운데 객체에 들어 있는 객체의 특성이나 내용을 그대로 유지해주는 것은 거의 없다. 대부 분의 전송 방법은 객체를 납작하게 만들어 훨씬 더 제한적인 형태로 만든다. 따라서 그와 같은 객체를 검색하려면 각 부분을 하나의 살아 있는 객체로 재구성하는 잠재적으로 복잡한 과정이 필요하다. 

* 생성과 재구성의 차이점
    * 재구성에 사용된 ENTITY FACTORY는 새로운 ID를 할당하지 않는다.
    * 불변식 위반에 좀더 탄력적으로 대응한다.

* 재구성 예시
    1. JPA 쓰면 편리하다~
![image.png](/files/2586715428031148139)
    2. 
![image.png](/files/2586715576656675344)

## 요약 정리
인스턴스 생성을 생성자로도 할 수 있지만 FACTORY가 필요할 때도 있다. FACTORY는 모델의 어떤 부분도 표현하지는 않지만 해당 모델을 나타내는 객체를 뚜렷하게 드러내는 데 일조하는 도메인 설계의 일부로 볼 수 있다.
* 객체의 생성과 재구성이라는 생명주기 전이(transition)를 캡슐화

# 패턴 3) REPOSITORY
> 모델에 집중하게 해주는 단순한 개념적 틀

> 객체의 상태 변이(저장소에 들어갈 때와 저장소에서 나올 때)의 기술적 복잡성을 책임지는 인터페이스

![image.png](/files/2586792667674240573)

JPA repository 설명인줄...

>전역적인 접근이 필요한 각 객체 타입에 대해 메모리상에 해당 타입의 객체로 구성된 컬렉션 이 있다는 착각을 불러 일으키는 객체를 만든다. 잘 알려진 전역 인터페이스를 토대로 한 접근 방 법을 마련하라. 객체를 추가하고 제거하는 메서드를 제공하고，이 메서드가 실제로 데이터 저장 소에 데이터를 삽입하고 데이터 저장소에서 제거하는 연산을 캡슐화하게 하라. 특정한 기준으로 객체를 선택하고 속성값이 특정 기준을 만족하는 완전히 인스턴스화된 객체나 객체 컬렉션을 반환하는 메서드를 제공함으로써 실제 저장소와 질의 기술을 캡슐화하라. 실질적으로 직접 접 근해야 하는 AGGREGATE의 루트에 대해서만 REPOSITORY를 제공하고，모든 객체 저장과 접근은 REPOSITORY에 위임해서 클라이언트가 모델에 집중하게 하라.

* 전역적인 접근이 필요한 각 객체 타입에 대해 메모리상에 해당 타입의 객체로 구성된 컬렉션 이 있다는 착각을 불러 일으키는 객체를 만들어라
![image.png](/files/2586798222084686426)
* 잘 알려진 전역 인터페이스를 토대로 한 접근 방법을 마련하라
`@Repository` `@Autowired`
* 객체를 추가하고 제거하는 메서드를 제공하고，이 메서드가 실제로 데이터 저장소에 데이터를 삽입하고 데이터 저장소에서 제거하는 연산을 캡슐화하게 하라.
![image.png](/files/2586798517483468849)
* 특정한 기준으로 객체를 선택하고 속성값이 특정 기준을 만족하는 완전히 인스턴스화된 객체나 객체 컬렉션을 반환하는 메서드를 제공함으로써 실제 저장소와 질의 기술을 캡슐화하라. 
![image.png](/files/2586798222084686426)
* Provide REPOSITORIES only for AGGREGATE roots that actually need direct access.
AGREGATE 루트에 대해서만 REPOSITORIES를 제공해라(AGGREGATE 내부에 존재하는 모든 객체는 루트에서부터 탐색을 토대로 접근하는 것 말고는 접근이 금지돼 있음)
* Keep the client focused on the model, delegating all object storage and access to the REPOSITORIES.
모든 객체 저장 및 접근 권한을 REPOSITORIES에 위임하여 클라이언트가 모델에 집중하도록 하십시오.

### REPOSITORY 패턴 설계의 장점
*  영속화된 객체를 획득하고 해당 객체의 생명주기를 관리하기 위한 단순 한 모델을 클라이언트에게 제시
* 영속화 기술과 다수의 데이터베이스 전략，또는 심지어 다수의 데이터 소스로부터 애플리케이션과 도메인 설계를 분리
* 객체 접근에 관한 설계 결정 전달
* 테스트에서 사용할 가짜 구현을 손쉽게 대체 가능(보통 메모리상의 컬렉션을 이용)

**=> 클라이언트가 모델에 집중할 수 있게 한다. 이는 도메인 주도 설계를 더 잘 할 수 있도록 보조하는 것이고 궁극적으로는 더 나은 소프트웨어를 설계하도록 한다.**
 
##  REPOSITORY에 질의하기
* REPOSITORY(인터페이스) 설계하는 방법
    * 여러가지가 존재할 수 있지만 책에서는 2가지 예시를 듬
        * 질의에 구체적인 매개변수를 직접 입력하는 방법
        * SPECIFICATION(명세)에 기반을 둔 질의를 사용하는 방법(이 경우에도 구체적인 매개변수를 직접 입력하는 방법도 추가 가능해야함)

사용 가능한 인프라스트럭처에 따라SPECIFICATION 기반 질의는 가장 적당한 프레임워크가 되거나 아니면 굉장히 사용하기 힘든 것이 될지도 모른다. => JPA는 책에서 예시든 방법 2개 외에도 example 과 같은 여러가지 다양한 방법을 제공하므로 ㅊㅊ


## 클라이언트 코드가 REPOSITORY 구현을 무시한다(개발자는 그렇지 않지만)

 > 개발자들은 캡슐화된 행위를 활용하는 것에 내포된 의미를 알아야 한다.
 
 개발자가 캡슐화된 행위를 활용하는 것에 내포된 의미를 모르고 사용했을때 시스템 메모리 부족 현상을 야기할 줄 몰라 장애를 발생시켰던 일화가 나옴
 
 >구현상의 상세한 내용까지 잘 알고 있어야 한다는 의미는 아니다. 
 REPOSITORY를 이용하는 개발자와 해당 REPOSITORY의 질의를 구현하는 개발 자는 서로 피드백을 주고받아야 한다.


## REPOSITORY 구현
* REPOSITORY 구현의 가장 기본적인 기능 : 저장 • 조회 • 질의 메커니즘을 캡슐화하는 것

![image.png](/files/2586812865206268756)
* 구현할때 명심해야할 사항
    * 타입 추상화 
    ![image.png](/files/2586815896917124091)
    * 클라이언트와의 분리를 활용 (ex - 자유롭게 REPOSITORY의 구현을 교체하면서 질의 기법을 다양하게 하거나 메모리상에 객체를 캐싱해 성능을 최적화)
    * 트랜잭션 제어를 클라이언트에 위임

## 프레임워크의 활용
REPOSITORY와 같은 것을 구현하기 전에 먼저 현재 사용 중인 인프라스트럭처，특히 모든 아키 텍처 프레임워크에 관해 곰곰이 생각해봐야 한다. 
즉，프레임워크에서 REPOSITORY를 만드는 데 손쉽게 사용할 수 있는 서비스가 제공된다는 사실을 알게 되거나，아니면 줄곧 프레임워크 때 문에 고생하고 있다는 사실을 알게 되거나 아키텍처 프레임워크에 이미 영속화 객체 획득에 대 응하는 패턴이 정의돼 있다는 사실을 알게 될지도 모른다.
할 수만 있다면 자신이 사용하고자 하는 설계 형식과 조화되는 프레임워크나 프레임워크 요소를 선택해라.

**요약 : 도메인 주도 설계를 구현하기 좋은 프레임워크를 활용하면 좋다.**

## FACTORY와의 관계
REPOSITORY가 데이터를 근거로 객체를 생성하므로 많은 이들이 REPOSITORY를 FACTORY로 생각하는데，사실 기술적 관점에서는 그렇다고 볼 수 있다. 
 도메인 주도 관점에서 보면 FACTORY와 REPOSITORY의 책임이 뚜렷이 구분되는데，FACTORY가 새로운 객체를 만들어 내는 데 반해 REPOSITORY는 기존 객체를 찾아낸다. 
![image.png](/files/2586819285956876591)
![image.png](/files/2586819373817477287)

**요약 : 새로운 객체와 이미 존재하는 객체를 구분하여 FACTORY와 REPOSITORY의 책임을 명확하게 분리해야한다.**