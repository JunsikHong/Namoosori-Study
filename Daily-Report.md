# 2024.9.23. (월)

> 현재 Stage 상태 : stage1 step3 (1회)

## Study & Think

---

### 요약
**각 레이어 사이의 완충작용을 하는 로직을 어떻게 구현할 것인가**에 대한 고민을 할 수 있었고 실제 어떻게 예외처리를 하는 지 알 수 있던 프로젝트였다.

**레이어 간 완충작용을 해 줄 수 있는 예외처리 및 확인사항들**

1. UI -> Controller : 행동의 예외사항 처리
2. Controller -> Service : 입력 값 존재 여부, 예외처리
3. Service -> DB : DB의 상태 체크

---

### Study (과정)
클라이언트와 비즈니스 로직 사이에 서비스 발행레이어를 추가하고 비즈니스 로직 레이어에 데이터 접근 레이어를 추가할 수 있다는 것을 알았다. 처음엔 이 단어들이 확 와닿진 않았지만 `TravelClub의 step3` 부분을 직접 코딩해보면서 힌트를 얻을 수 있었다.

#### ClubConsole Class
`ClubConsole`은 아래와 같이 구성 되어 있다.

1. 사용자에게 메뉴 항목을 보여주는 메서드
2. 사용자가 선택한 메뉴에 따라 `ClubCoordinator`와 맵핑하는 메서드

이로 미루어 보아 3레이어 중 UI 레이어와 비즈니스로직 레이어 역할을 겸하고 있다고 볼 수 있다.

아래는 `ClubConsole Class`과 `ClubCoordinator Class`의 `register메서드`가 맵핑되는 사이에 완충작용을 해 줄 수 있는 `서비스 발행 레이어` 코드만 모아 놓은 것이다.

```java
public class ClubConsole {
    private void register() {    	
    	...
        //사용자가 입력한 clubName 변수가 null이거나 공백일 경우 서비스 로직과 연결되지 않도록 한다
    	if(clubName == null || clubName.equals("")) return;
        
        //사용자가 의도치 않은 앞뒤 공백을 넣었을 경우 의도한 데이터만 서비즈 로직에 넘겨줌
        clubName = clubName.trim();
        
        //사용자가 입력한 clubName이 데이터베이스에 이미 존재하는 지 판단하고 서비스 로직에 넘겨줌
        if (clubCoordinator.exist(clubName)) return;
    }
}
```
ClubCoordinator로 요청을 위임하는 것 외에도, UI에서 컨트롤러로 요청이 들어올 때, 필요한 예외처리나 에러처리 등을 수행하고 있는 것을 알 수 있다.

#### ClubCoordinator Class
`ClubCoordinator`는 아래와 같이 구성 되어 있다.

1. 사용자가 선택 한 메뉴의 실질적인 비즈니스 로직을 수행하는 메서드
2. 비즈니스 로직에 따라 `DB`에 반영하는 메서드

이로 미루어 보아 3레이어 중 비즈니스로직 레이어 역할과 DB 역할을 겸하고 있다고 볼 수 있다.

아래는`ClubCoordinator Class`의 `register메서드`와 `DB` 사이에 완충작용을 해 줄 수 있는 `데이터 접근 레이어` 코드만 모아 놓은 것이다.

```java
public class ClubCoordinator {
	public String register() {
    	...
        //사용자가 입력한 clubName이 데이터베이스에 이미 존재하는 지 판단
        if(this.exist(newClub.getName())) return;
       
        //포인터 인덱스와 DB의 현재 max길이가 같다면 추가 공간을 만들어야 하므로 DB의 공간 먼저 체크
        if(nextIndex == clubs.length) {
           clubs = Arrays.copyOf(clubs, nextIndex + capacity);
        }
	    ...
	}
}

```
적절한 비즈니스 로직처리만 해서 DB에 결과를 반영하는 것 외에도, ClubConsole에서 넘어오는 값의 확인과 DB의 상태를 체크하고 반영하는 것을 알 수 있다.

---

### Think (결과)
현업 비즈니스 로직을 짜는 것은 단순한 `CRUD` 메서드 외에 **사용자의 입력 값 체크**, **서버 내에서 일어날 수 있는 각종 예외처리들**, 또한 **`DB`의 상태를 체크**해서 대부분의 상황에 대응할 수 있는 단단한 소프트웨어를 만들어야 한다는 생각이 들었다.

---

# 2024.9.24. (화) 

> 현재 Stage 상태 : stage1 step3 (1회) -> stage1 step4 (2회)
  
## Study & Think

---

### 요약

**실생활의 이야기를 정보로 표현하기 위한 과정**을 정리 해 보면서 줄거리를 시스템으로 옮기는 과정을 체계화 하였다.

<img src="/img/flow1.png" width="400px" height="400px">

---

### Study (과정)
 
**줄거리** : 친구들과 여행에 관해 이야기를 나눌 공간이 필요하다

1. 이야기의 줄거리를 이해한다 (사용자의 시점과 시스템의 시점에서)
- **사용자의 시점**에서 **어떤 사용자**가 **어떤 일**을 하려고 하는 지를 본다.
ex 클럽 관리자가 여행클럽 정보를 저장하고 찾으려고 한다.

- **시스템의 시점**에서 **어떤 정보**를 **어떻게 처리** 할 것인 지를 고민한다.
ex 여행클럽 정보를 사용자에게 보여주고 처리하고 저장한다.

2. 이야기의 인물과 소재를 이해한다 (시스템의 시점에서)
- 등장인물은 **이야기 도우미**이고, 소재는 **입력한 정보**이다.
ex 이야기 도우미가 여행클럽에게 여행클럽을 만들어 달라고 요청한다.

3. 이야기를 토대로 시스템을 구축한다.
- 이야기 도우미의 역할을 설계한다 (정보의 설계서를 바탕으로 정보를 만들라는 요청을 한다)
- 정보를 설계한다 (소재의 구성요소는 **속성**으로, 행동은 **기능**으로)
- 이를 코드로 나타내면 아래와 같다

```java

public class TravelClub {

	//속성 : TravelClub의 이름과 내용
    private String name;
    private String intro;

	//기능 : TravelClub의 이름과 내용을 표시하는 기능
    public String tellMeAboutYou() {
        return "Name : " + name + ", Intro : " + intro;
    }
    
    ...
    
}
```

---

### Think (결과)
기업의 요구사항을 시스템을 통해 문제를 해결하는 것이 개발자의 본질이다. 현업에서 시스템을 이해하지 못하고 있는 사람의 요구사항을 시스템에 바로 적용하지는 못한다. 개발자가 그 사람의 요구사항을 듣고 시스템에 적용시킬 수 있도록 재정의하고 다듬는 과정을 거쳐야 한다. 오늘 공부한 내용은 그러기 위한 초석이라 할 수 있겠다!


# 2024.9.25. (수) 

> 현재 Stage 상태 : stage1 step4 3번째 반복 코딩을 끝내고 stage2로 넘어와 step1 코딩까지 완료했습니다.
  
## Study & Think

---

### 요약 (알게 된 것)

Travel Club Stage2 Step1을 코딩하며 **복잡한 구조의 객체를 짜임새 있게 구성하는 방법**에 대해서 알 수 있었다.

1. 입력값에 대해 유효성 검사 또는 검증이 필요 할 때에는 `setter`를 구현 할 때 검증 절차를 진행 한 후 변수가 초기화 되도록 한다.
2. 변수가 가질 수 있는 값이 몇 가지의 한정된 값이어야 한다면 `enum`타입으로 선언하여 이용하여 안정적인 값을 가지도록 한다.
3. 제약조건이 변경되지 않으며 전역에서 사용가능하여야 한다면 `static` `final`로 선언한다.

---

### Study (알게 된 과정)

이야기의 줄거리를 파악하며 정보가 어떤 구조로 구성되어 있는지 파악했다.

```
정보의 구조
    여행클럽 type
        - 이름 (최소길이=3)
        - 소개말 (최소길이=10)
        - 설립일자 (날짜형식)
        - 회원들 (Collection)

    회원 type
        - 이메일 (유효성검사)
        - 이름
        - 닉네임
        - 휴대폰번호
        - 생일
        - 역할 (Memeber or President)

    역할 enum
        -Member
        -President
```

#### How?
회원 타입에서 중요한 요소는 이메일과 역할이다. `ClubMember`의 이메일 변수는 **이메일 패턴**을 정확히 지키도록 해야한다. 그 방법으로 생성자는 생성 시에 전역변수 이메일을 바로 초기화 시키지 않고, `setter`에게 위임하여 처리한다. 이 때, `setter`는 이메일 입력값에 대해 유효성 검사를 실시하고 부적합하다고 판단 될 경우 예외를 발생시켜 객체의 생성을 막는다. 만약 외부에서 `setter`로 직접 접근 하더라도 유효성 검사는 무조건 실행 되기에 **조건에 맞지 않는 값으로 초기화 시킬 수 있는 방법은 없다.**

#### 잘못된 이메일이 전달될 때 ClubMember의 흐름
StoryAssistant -> ClubMember (잘못된 이메일) {setter에게 값 처리 위임} -> setter() {이메일 유효성 검사 후 예외클래스에게 위임} -> CustomException() {최종 부모인 Throwable의 DetailMessage에 message를 전달한다} -> 예외 발생

```java
//StoryAssistant가 ClubMember 호출 -> ClubMembeer
class StoryAssistant {}

class ClubMember {
    //생성 시 매개변수로 잘못된 이메일 주입 -> setter에게 값 처리 위임 -> setter
    public ClubMember() {}
    //이메일 유효성 검사 후 예외클래스에게 위임 -> CustomException
    public void setter() {}
}

class CustomException extend Exception{
    //생성 시 최종 부모인 Throwable에게 위임 -> Throwable
    public CustomException() {}
}

class Throwable {
    //예외 발생
}
```

#### How?
한 편 `ClubMember`의 역할 변수는 **멤버와 관리자** 두 가지의 값만 가질 수 있도록 해야한다. 그 방법으로 `RoleInClub`을 `enum` 타입으로 선언하여 `Member`, `President` 두 상수만 열거한다. 이렇게 생성된 `ClubMember`의 전역변수 `role`은 **열거된 상수로만 값을 가질 수 있기에 많은 예외 상황들을 사전에 예방할 수 있다.** 보통 관리자보다는 멤버의 비율이 더 많기에 기본으로 생성되는 값은 `Member`로 설정해 둔다.

#### enum정리
- 정의 : 서로 연관된 상수(바뀌지 않는 값)들의 집합.
- 배경 : 기존에 상수를 관리하기 위한 static final 문법의 한계 (코드 복잡, 타입으로 인스턴스 생성시 switch문 비교 불가)
- 원리 : **상수 열거 기능 + 클래스 역할** => 필드값과 메서드 생성 가능
- 문법 : `상수, 상수` 형태로 상수를 열거. `상수(값)` 형태로 생성자의 매개변수로 값을 전달.
- 주의 : 생성자를 private으로 만들지 않으면 newInstance 메서드를 통해 값 변경이 가능하다.

```java
enum Fruit {
    //상수 APPLE의 실제 값은 public static final Fruit APPLE = new Fruit(1);
    //상수 MANGO의 실제 값은 public static final Fruit MANGO = new MANGO(2);
    APPLE(1), MANGO(2); 

    //APPLE(1) 생성 시에 1값이 생성자 매개변수로 주입되어 생성자 실행
    private Fruit(int i) {
        this.setCount(i);
    }

    //필드변수는 private으로 생성해서 외부 접근 불가
    private int count;

    //대신 메서드를 통해 간접적으로 필드변수 초기화
    public void setCount(int i) {
        this.count = count;
    }
}

```

#### How?
여행클럽 타입에서 중요한 요소는 이름과 소개말이다. `TravelClub`의 이름과 소개말은 각각 **최소 3글자, 10글자 이상**어야 한다. 앞서 이메일 값을 `setter`에서 검증한 것과 같은 원리이다. 다만 이 제약조건이 전역에서 참조되면서 바뀌지 않는 값으로 지정하기 위해 전역변수에 `static` `final`로 선언한다. 이로써 외부에서 이름과 소개말의 **제약조건과을 변경시킬 수 없고**, 이름과 이메일 값 자체도 **조건에 맞지 않는 값으로 초기화 시킬 수도 없다.**

#### 이름과 이메일 제약조건
```java
private static final int MINIMUM_NAME_LENGTH = 3;
private static final int MINIMUM_INTRO_LENGTH = 10;
```
---

### Think (생각)
이전엔 비즈니스 로직 레이어를 통해서 필요할 때마다 입력값에 대한 검증을 실행했다. 이런 방식이 비효율적이라는 것을 느끼고 있었지만 어떤 방식으로 접근해야 할 지 몰랐다. 그런데 객체 자체가 생성될 때 값이 조건을 만족시키지 못하면 객체를 생성시키지 못하도록 짜여진 코드가 인상적이었다. 큰 프로젝트 개발 시에 입력값에 대해 일일히 검증하다보면 효율성 측면과 실수 가능성이 있기에 이 방법으로 효율적이고 안정적인 개발에 도움이 될 것 같다는 생각이 들었다.




# 2024.9.26. (목) 

> 현재 Stage 상태 : stage2 step1 1회 째 반복에서 시작해서 stage2 step4 1회 째 반복 진행중에 있습니다.
  
## Study & Think

---

### 요약 (알게 된 것)

자바에서 반복문을 수행하는 여러가지 방법들과 문법에 대해서 알아보았다.

1. `향상된 for문`은 반복을 수행하는 대상 컬렉션에 `iterator()` 메서드를 호출하고 `Iterator` 인터페이스타입을 반환받는다.
2. `Collection` 프레임워크에서 `Stream` 인터페이스를 이용해 반복을 순회할 때 파이프라인 형식으로 구성하는데 중간연산은 `stream`을 반환하지만 최종연산은 `stream`을 반환하지 않고 종료하기 때문에 파이프라인 구성 시 이를 고려해야 한다.
3. 람다 표현식의 `->` (화살표) 로 반복으로 나온 값을 활용할 수 있고, `::` (메서드 참조)은 정적메서드를 참조하여 해당 클래스의 메서드를 사용할 수 있다.

---

### Study (알게 된 과정)

step2 stage3 를 구현하는 와중에 `MemberHelper` `modify` 메서드에서 `Iterator`가 쓰이는 것을 보고 궁금증이 생겼다. IDE에서 `Iterator`를 `향상된 for문`으로 리팩토링하라는 안내문을 보고 두 코드가 어떻게 대치될 수 있는 지 알아봤다.

```java
    public void modify(ClubMember member, Map<String, String> newValueMap) {
        Iterator<String> nameIter = newValueMap.keySet().iterator();
        while(nameIter.hasNext()) {
            String name = nameIter.next();
            String value = newValueMap.get(name);
        }
    }

    public void modify(ClubMember member, Map<String, String> newValueMap) {
        for (String name : newValueMap.keySet()) {
            String value = newValueMap.get(name);
        }
    }
```

#### Iterator와 향상된 for문의 관계
* `향상된 for문`이 실행되는 원리를 살펴보니 **반복을 도는 대상 컬렉션에 `iterator()` 메서드를 호출하고 `Iterator` 인터페이스타입을 반환 받는다**는 사실을 알았다. 이제 반복을 시작하면 `Iterator`의 `hasNext()`를 호출하고 `true`를 반환할 시 `next()` 메서드를 호출하여 대상 컬렉션을 순회하는 것이다.

#### Stream
* 이를 알아보다 보니 반복을 수행하는 역할의 `Stream` 인터페이스에 대해서도 궁금해져 추가로 알아보았다. `Stream` 인터페이스를 생성하는 방법은 `StreamBuilder`를 생성해 직접 생성하는 방법과 `Collection` 프레임워크로 구현된 객체에서 반환받는 방법이 있다. 기본적으로 많이 쓰는 방법은 `Collection` 프레임워크로 구현된 객체에서 반환받는 방법이니 이것에 대해서 알아봤다.

* `Collection` 프레임워크로 구현된 객체는 `Stream`을 반환하는데 **중간연산을 수행한 이후에는 `stream`을 그대로 반환하지만, 최종연산을 수행한 이후에는 `stream`을 반환하지 않는다.** 최종연산을 수행하면 `stream`은 닫혀서 재사용이 불가하기 때문에 **파이프라인 구성 시 이를 고려해야 한다.**

* 중간연산과 최종연산은 각각 아래와 같은 메서드들을 가지고 있다.
- 중간연산 : filter, map, limit, sorted, distinct, peak, skip ...
- 최종연산 : forEach, collect, count, sum, reduce ...

#### lambda
* `Stream`을 통해 `Collection`의 반복문을 수행할 때 람다 표현식을 사용하니 람다 표현식에 대해서도 더 알아보았다. 반복을 돌리려는 대상 뒤에 `.stream()`으로 `stream`을 열고 중간연산자로 파이프라인을 구성한다. **이 때 `filter`의 화살표 이전에 있는 것은 반복으로 나온 값, 화살표 이후에는 반복으로 나온 값을 어떻게 활용할 것인지 정의**하는 부분이다. `map`의 `::`은 왼쪽에 있는 **정적메서드를 참조하여 오른쪽에 해당 클래스의 메서드를 참조한다.** `collect`는 최종연산자로 아래처럼 `Collectors.toList()` 와 같은 형태로 **최종 반환 타입을 지정할 수 있다.**

```java
List<String> names = ...
List<String> upperCaseNames = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

### Think (생각)

자바에 대해 모르는 것이 많다고 느껴졌다. 많은 사람들과 협업하여 코드를 작성할 때 간결한 문장을 통해 다른 사람들로 하여금 이해가 쉽도록 하는 것이 좋은 개발자다. **수많은 주석보다 코드 자체로 이해가 간편하도록 작성하는 연습과 고민들**을 많이 할 필요가 있어보인다. 그러기 위해 오늘 배운 `iterator`, `stream` `lambda` 과 같은 문법들을 잘 알아 놓을 필요가 있다. 앞으로 자바에 대한 깊은 이해를 통해 품질좋은 코드를 생산해내는 개발자가 되어야겠다.



# 2024.9.27. (금) 

> 현재 Stage 상태 : stage2 step4 1회 반복을 끝내고 stage2 step3 2회 반복 진행중에 있습니다.
  
## Study & Think

---

### 요약 (알게 된 것)

프로젝트의 구조를 더 자세히 이해하게 되었다.
1. step3에서 클래스들은 전역에서 단 하나씩만 존재하게 설계되었다. StoryAssistant가 ClubMenu를 호출할 때 ClubMenu가 모든 클래스를 다 생성하고 전달하는 방식으로 설계 되어있다.
2. step4에서는 MapStorage가 프로그램에서 단 하나만 존재하도록 설계되었기 때문에 모든 클래스들이 하나씩만 존재할 필요가 없다. 그래서 짐꾼 역할을 하는 ClubMemberStore와 TravelClubStore는 여러 객체가 생성 될 수 있다.

---

### Study (알게 된 과정)

#### 의문
step3 ClubCoordinatoor 클래스에서 MemberHelper를 생성하고 있는 것을 보고 의문점이 들었다. MemberHelper객체의 입장에서 보면 ClubCoordinator에서 최초로 객체가 생성되고 MemberWindow에서 ClubCoordinator를 호출하면서 그 안에 있는 MemberHelper객체를 꺼내온다. MemberWindow에서 바로 MemberHelper객체를 생성하면 되는데 왜 이렇게 복잡하게 구현했을까 고민했다. 

#### 결론(정리)
교육생들과 의논해 보면서 깨달았는데 결론적으로 MemberHelper 뿐만 아니라 **모든 객체들이 전역에서 단 하나만 존재**해야 하기 때문에 이렇게 구현된 것이었다. 전역에서 단 하나만 존재해야 하는 이유는 프로그램에서 **사용자가 바라보고 있는 currentClub 객체가 전역에서 단 하나만 존재**해야 하기 때문이다. 다시 한 번 더, 그래야 하는 이유는 최종적으로 **DB역할을 하는 클래스가 전역에서 단 하나만 존재**하도록 설계 되지 않았기 때문이다.

#### 과정(검증)
맨처음에 프로그램을 실행하면 StoryAssistant가 ClubMenu(Club UI)를 호출하면서 MemberMenu(Member UI), ClubWindow(Club Controller), ClubCoordinator(Club Service), MemberHelper(Member Service) 이렇게 4객체를 만들어낸다. 이제 사용자가 최초로 클럽을 등록하려고 1번을 누르고 입력값을 입력하면 이미 생성된 ClubWindow객체의 메서드를 호출하고, 이어서 ClubCoordinator까지 호출한다. 호출 할 때도 이미 생성한 객체를 전달한 것이기 때문에 새로운 객체는 생겨나지 않는다. 그리고 앞서 생성된 객체들의 currentClub에는 현재 조회한 클럽의 정보가 똑같이 저장되어 있다. 이제 MemberMenu로 들어가서 멤버를 조회하면 이미 ClubCoordinator가 생성한 MemberHelper를 호출하는 것이기 때문에 이 때도 새로운 객체 생성은 없다. 디테일한 검증을 하기 위해 자세히 설명했지만, 결론적으로는 프로그램을 실행할 때 모든 객체가 한 번씩만 생성되도록 설계되었다는 것이다.

---

### Think (생각)

Spring을 쓰면서는 미처 생각하지 못했던 부분이다. 애시당초 구조 자체가 최대한 불필요한 객체를 최소화 할 수 있었고, DB Connection을 이용하여 외부에 있는 안정된 DB와 연결하여 사용할 수 있기 때문이다. 그러나 이것을 알고 개발을 하는 것과 모르고 개발을 하는 것은 다를 것이다. 




# 2024.9.30. (월) 

> 현재 Stage 상태 : stage2 step3 2회 반복부터 시작해서 stage3 step1 1회반복까지 완료했습니다.
  
## Study & Think

---

### 요약 (알게 된 것)
* 다뤄야할 정보가 많을 때, 단순히 클래스를 포함시키는 형식으로 1:N, N:M관계를 구현하려고 하면 한계점이 많다는 것을 알았습니다.
* 대신 각 객체에 고유한 값을 가지도록 하고, 관계를 맺는 객체끼리 서로의 값을 공유하면 된다는 것을 알았습니다. 이로써 1:N관계는 1쪽의 PK를 N쪽의 객체가 FK로 가지면 관계가 맺어집니다. 또한 N:M관계는 N쪽과 M쪽의 PK를 모두 갖는 객체를 만들고, N과 M은 각각 새로운 객체의 PK를 갖는것으로 관계가 맺어집니다.

---

### Study (알게 된 과정)

* Stage2에서 관계는 아래와 같은 논리와 구현의 관점을 가지고 있었습니다.
#### 논리적인 관점
* 클럽과 회원의 관계는 1:N

#### 구현적인 관점
* 클럽 클래스에 회원 클래스를 리스트 형태로 갖도록 구현

* Stage3에서 관계는 아래와 같은 논리와 구현의 관점을 가지도록 변경되었습니다.
#### 논리적인 관점
1. 게시판과 게시글의 관계는 1:N
2. 클럽클래스와 회원클래스의 관계는 N:M

#### 구현적인 관점
1. 클럽클래스에 회원클래스를 직접 포함시키지 않고, 회원 클래스가 클럽클래스의 PK를 갖도록 구현
2. 클럽 클래스와 회원 클래스를 이어주는 클럽멤버쉽 클래스를 생성. 클럽 멤버쉽클래스는 클럽클래스의 PK와 회원 클래스의 PK를 갖는다. 클럽클래스와 회원클래스는 멤버쉽 클래스의 PK만을 갖는다.

* 클래스들이 PK의 역할을 하는 변수를 갖는 것의 의미 : PK라고 간단히 말하긴 했지만, 사실 자바의 관점에서 보면 객체가 생성될 때마다 서로 다른 변수의 값을 생성하는 메서드를 통해 각 객체들이 서로 다른 값을 가지게 되는 것이다. 즉 PK의 특성(NULL이어서는 안되고 유일해야 한다)을 가지는 변수들이 각 객체마다 있다는 것이다.

---

### Think (생각)
무엇인가를 결과로 증명해내지 못하면 한 것이 아니라는 것을 알았습니다.



# 2024.10.1. (화) 

> 현재 Stage 상태 : stage3 step1 1회반복부터 시작해서 stage3 step4 1회반복까지 완료했습니다.
  
## Study & Think

---

### 요약 (알게 된 것)

* stage3의 전체적인 흐름을 파악했습니다.
* Menu클래스는 사용자에게 메뉴를 보여주고 입력값을 받고, Console클래스는 받은 입력값과 요청행위를 분기합니다.
* 이제 Service 인터페이스는 ServiceLogic 클래스가 구현해야 할 메서드들을 정의하고 ServiceLogic 클래스는 적절한 예외상황을 처리하며, Dto형태로 받은 입력값을 Entity로 바꾸고 Store 인터페이스에게 나머지 업무를 위임합니다.
* Store 인터페이스도 마찬가지로 MapStore 클래스가 구현해야 할 메서드들을 정의하고 MapStore 클래스는 DB가 해야 할 일들을 정의합니다. 
* MapMemory 클래스는 DB의 역할을 하며 실제 데이터들을 저장하고 있습니다. 
* 아래의 그림은 해당 설명을 그림으로 표현한 것입니다.

<img src="/img/flow2.png" width="400px" height="400px">

---

### Study (알게 된 과정)

#### Service
* 우선 Service인터페이스를 실제로 구현하고 있는 ServiceLogic 클래스의 메서드들이 무엇을 하고 있는 지를 파악했다.

1. register
* 입력값 중 이름이 중복인지 확인하도록 Store에게 업무 위임
* Null값 대응 + 예외 처리
* 저장 가능하다면 Dto 객체를 Entity 객체로 전환
* Entity 객체를 저장소에 저장하도록 Store에게 업무 위임

2. find
* 입력값이 저장소에 있는지 확인하도록 Store에게 업무 위임
* Null값 대응 + 예외 처리
* 찾은 Entity 객체를 Dto 객체로 전환
* Dto 객체를 반환

3. modify
* 입력값 중 이름이 중복인지 확인하도록 Store에게 업무 위임
* Null값 대응 + 예외처리
* 수정 가능하다면 해당 객체를 가져오도록 Store에게 업무 위임
* 새로 입력된 값 검증 후에 Dto 객체를 Entity 객체로 전환
* Entity 객체를 수정하도록 Store에게 업무 위임

4. remove
* 입력값이 저장소에 존재하는지 확인하도록 Store에게 업무 위임
* Null값 대응 + 예외처리
* 존재한다면 해당 객체를 제거하도록 Store에게 업무 위임

#### Store
* 이제 Store인터페이스를 실제로 구현하고 있는 MapStore 클래스의 메서드들이 무엇을 하고 있는 지를 파악했다.

1. create
* 입력값 중 이름이 중복인지 확인
* Null값 대응 + 예외 처리
* 키값 생성
* 저장소에 저장

2. retrieve
* 조회 조건에 맞는 값 혹은 값들을 반복자를 통해 조회

3. update
* 수정하려고 하는 객체의 키값과 내용을 DB에 저장

4. delete
* 삭제하려고 하는 객체의 키값을 토대로 DB에서 삭제

5. exists
* 조회하려는 객체의 키값을 토대로 조회한 후 Null인지 아닌지 판별

---

### Think (생각)
* 하나의 클래스가 여러 역할을 맡았을 때보다, 하나의 클래스가 하나의 역할만 수행하는 구조가 훨씬 파악하기 편했다.
* 클래스의 메서드들이 하는 역할에 대해서도 예측이 가능했고, 객체의 유일성을 보장하기 위해 불필요한 객체 이동도 없었기 때문이다.




# 2024.10.2. (수) 

> 현재 Stage 상태 : stage3 step4 1회반복부터 시작해서 stage3 step3 2회반복까지 완료했습니다.

### 요약 (알게 된 것)
자바에 구현되어 있는 객체가 서로 어떻게 상호작용하는지 정리하고 알게 되었습니다.

<img src="/img/flow3.png" width="400px" height="400px">

* club의 pk값의 역할
1. club의 주 식별자 
2. membership의 외래키
3. board의 주 식별자

* member의 pk값의 역할
1. member의 주 식별자
2. membership의 외래키

* board의 pk값의 역할
1. club의 pk값
2. posting의 외래키

* posting의 pk값의 역할
1. posting의 주 식별자

---

### Study (알게 된 과정)

#### Club과 Member의 관계
* 요구 : club은 member를 여러명 등록할 수 있고 member는 club을 여러개 가입할 수 있어야 합니다.
* 구현 : 이를 구현하기 위해 member객체와 club객체는 각각 membershipList를 가지고, membership객체는 club객체의 키값과 member객체의 키값을 가집니다.
* 결과 : 그러면 클럽 객체 하나는 자신의 클럽에 등록한 멤버들의 정보를 가지고, 멤버 객체 하나는 자신이 가입 클럽의 정보를 가질 수 있습니다.

```java
public class TravelClub implements AutoIdEntity {
    private String usid; //club의 pk
    private List<ClubMembership> membershipList; //membershipList
}

public class CommunityMember implements Entity {
    private String email; //member의 pk
    private List<ClubMembership> membershipList; //membershipList
}

public class ClubMembership {
    private String clubId; //club의 pk -> membership의 복합키
    private String memberEmail; //member의 pk -> membership의 복합키
}
```

#### Club과 Board의 관계
* 요구 : club 하나 당 board는 하나만 생성할 수 있어야 하고, board는 하나는 club 한개에만 속해야 합니다.
* 구현 : 이를 구현하기 위해 board객체는 club객체의 키값을 본인의 키값으로 지정합니다.
* 결과 : 그러면 board를 생성하기 위해서는 club 객체의 키값이 필요하고 club 객체 하나당 한개의 키값만 가지고 있으니 요구를 만족할 수 있습니다.

```java
public class TravelClub implements AutoIdEntity {
    private String usid; //club의 pk
    private List<ClubMembership> membershipList; //membershipList
}

public class SocialBoard implements Entity {
    private String clubId; //club의 pk이면서 board의 pk
}
```

#### Board와 Posting의 관계
* 요구 : board 하나 당 posting은 여러개 생성할 수 있어야 합니다.
* 구현 : 이를 구현하기 위해 posting 객체는 board객체의 키값을 가져야 합니다. 
* 결과 : 그런데 이 때, board의 키값은 club의 키값입니다. 즉, club하나는 board하나를 가지고 board는 여러개의 posting을 갖게 되는데, 이 세 객체를 연결해주는 것은 club의 키값이 됩니다.

```java
public class TravelClub implements AutoIdEntity {
    private String usid; //club의 pk
}

public class SocialBoard implements Entity {
    private String clubId; //club의 pk이면서 board의 pk
}

public class Posting implements Entity {
    private String usid; //posting의 pk
    private String boardId; //club의 pk이면서 board의 pk
}
```
---

### Think (생각)
* 객체 간 관계를 어떻게 정의하느냐에 따라 메서드의 구현 방법이 달라질 수 있음을 알게 되었습니다.



# 2024.10.3. (목) 

> 현재 Stage 상태 : stage3 step3 2회반복부터 시작해서 3회반복째의 stage3 step4 작성중에 있습니다.

### 요약 (알게 된 것)

* 사용자의 modify요청 시에 console 레이어에서 findOne()메서드가 수정될 객체를 불러온 이후에 변경된 값을 셋팅한다.
* service 로직 레이어에서는 새로 가져온 targetClub에 변경된 값을 셋팅하는 것보다 사용자가 입력한 값이 주입되어있는 객체를 그대로 다시 저장소로 저장하는 흐름이 더 자연스럽다.
---

### Study (알게 된 과정)


```java
    @Override
    public void modify(TravelClubDto clubDto) {
        //이름 중복 검사
        Optional.ofNullable(clubStore.retrieveClubByName(clubDto.getName()))
                .ifPresent(club -> {
                    throw new ClubDuplictionException("club already exists" + club.getName());
                });

        //수정하고자 하는 클럽 가져오기
        TravelClub targetClub = Optional.ofNullable(clubStore.retrieveClub(clubDto.getClubId()))
                .orElseThrow(() -> new NoSuchClubException("no such club" + clubDto.getClubId()));

        //입력값에 이름이 비어있다면 가져온 클럽으로 set
        if(StringUtil.isEmpty(clubDto.getName())) {
            clubDto.setName(targetClub.getName());
        }

        //입력값에 소개가 비어있다면 가져온 클럽으로 set
        if(StringUtil.isEmpty(clubDto.getIntro())) {
            clubDto.setIntro(targetClub.getIntro());
        }

        //입력값을 entity로 바꿔서 수정
        clubStore.update(clubDto.toTravelClub());
    }
```
* 클럽 수정 서비스 로직의 코드가 조금 이상해 보여서 분석해보았습니다.
* 사용자가 CLI를 통해 수정할 클럽의 이름과 소개를 입력합니다. 서비스 로직은 해당 값을 받아 입력된 이름값이 중복하지는 않는지 검사하고, 중복이 아니라면 이제 수정하고자 하는 클럽의 정보를 가져오게 됩니다. 이 때, 입력받은 값이 비어있다면 가져온 클럽의 정보를 주입합니다.
* 이러한 흐름에서 입력받은 dto 값에는 membership정보가 누락되어있을 것이라고 생각했습니다. 그래서 아래의 코드로 바꾸는 것이 누락하지 않는 방법이라고 생각했습니다.
* 하지만 이미 console 레이어에서는 findOne() 메서드를 통해 기존의 등록된 객체를 가져오면서 membership정보를 가지고 있는 상태입니다. findOne()을 통해 가져온 객체를 수정해서 다시 저장하는 것이 자연스럽기 때문에 기존의 코드가 맞는 것을 재확인했습니다.


```java
        //입력값에 이름이 비어있지 않다면 수정된 것이므로 수정된 이름으로 set
        if(!StringUtil.isEmpty(clubDto.getName())) {
            targetClub.setName(clubDto.getName());
        }

        //입력값에 소개가 비어있지 않다면 수정된 것이므로 수정된 소개로 set
        if(!StringUtil.isEmpty(clubDto.getIntro())) {
            targetClub.setIntro(clubDto.getIntro());
        }

        //가져온 클럽entity (멤버쉽 포함되어 있는) 를 수정하도록 요청
        clubStore.update(targetClub);
```



---

### Think (생각)
이야기의 흐름을 파악할 때 객체가 어떤 상태인지 파악하는 것이 중요하다는 것을 알았습니다.





# 2024.10.3. (목) 

> 현재 Stage 상태 : stage3 step4 3회반복 완료했습니다.

### 요약 (알게 된 것)

* 서비스 발행 레이어와 데이터 접근레이어는 각각 입력값 자체를 잘 가공해서 비즈니스 로직에 전달하는 것과, 저장소에 데이터를 건네주고 받을 때 고려해야 하는 것들만 생각하면 되지만, 비즈니스 서비스 로직에서는 각 CRUD마다 예외사항들을 처리하고 객체간의 관계에 따라 행동을 다르게 정의해줘야 됩니다.

* 비즈니스 서비스 로직에서 중요한 것
* CRUD에서 고려해야 할 예외사항
1. C create 생성시에는 키값의 중복이 없는지 확인해야 합니다.
2. R read 조회한 값이 있는지 없는지 확인해야 합니다.(NullPointerException)
3. U update 업데이트 하려는 객체가 존재하는지 확인하고, 업데이트 하는 값이 비어있지 않도록 확인해야 합니다.
4. D delete 삭제하려는 객체가 존재하는지 확인해야 합니다.

* 클럽과 멤버의 관계 : 멤버쉽에 클럽과 멤버쉽의 키값이 있으므로, 키값이 바뀌는 경우인 삭제하는 경우에는 각각의 멤버쉽에서 삭제해줘야 합니다.
* 클럽과 게시판의 관계 : 클럽 당 게시판은 하나이고, 게시판의 키값은 클럽의 키값과 동일합니다.
* 게시판과 게시글의 관계 : 게시판 하나에 게시글 여러개가 생성될 수 있고, 게시판 삭제시에는 게시판에 있는 게시글 전체를 삭제해줘야 합니다.
* 멤버와 게시글의 관계 : 게시글을 작성하기 위해서는 멤버가 클럽에 가입되어있어야 하니, 게시글 생성 시 멤버쉽을 확인해야 합니다.
---

### Study (알게 된 과정)

#### 클럽 서비스 로직
1. 접근해야 하는 저장소 : 클럽 스토어, 멤버 스토어
2. 고려해야할 예외사항 처리 및 흐름
- register() : clubName 중복확인, 생성
- findClub() : 입력한 clubId가 없는 경우 확인, 조회
- findClubByName() : 입력한 clubName이 없는 경우 확인, 조회
- modify() : 수정할 clubName 중복확인, 수정할 클럽이 있는지 확인, 수정할 입력값 있는지 확인, 클럽 정보 업데이트
- remove() : 삭제할 클럽이 있는지 확인, 클럽에 가입한 멤버들에게서 클럽 정보 삭제
- addMembership() : 추가할 멤버가 있는지 확인, 추가할 멤버가 클럽에 가입되어있는지 확인, 클럽과 멤버의 멤버쉽 모두 업데이트
- findMembership() : 클럽에 멤버쉽이 있는지 확인, 조회
- modifyMembership() : 바꿀 역할을 클럽과 멤버의 멤버쉽 모두 업데이트
- removeMembership() : 클럽과 멤버의 멤버쉽 모두 삭제

#### 멤버 서비스 로직
1. 접근해야 하는 저장소 : 클럽 스토어, 멤버 스토어
2. 고려해야할 예외사항 처리 및 흐름
- register() : memberEmail 중복확인
- find() : 해당 멤버 이메일이 없는 경우
- findByName() : 해당 멤버 이름이 없는 경우
- modify() : 수정하려는 멤버가 없는 경우 (이메일은 바꿀 수 없기 때문에 멤버쉽은 고려하지 않는다)
- remove() : 삭제하려는 멤버가 존재하는지 확인, 해당 멤버가 가입한 멤버쉽 확인

#### 게시판 서비스 로직
1. 접근해야 하는 저장소 : 클럽 스토어, 멤버 스토어
2. 고려해야할 예외사항 처리 및 흐름
- register() : 게시판 이름이 중복인지 확인하고, 게시판을 생성하려는 클럽이 존재하는지 확인, 관리자 이메일이 해당 클럽에 가입되어있는지 확인 후에 생성
- find() : 조회하려는 게시판이 존재하는지 확인
- findBoardByName() : 조회하려는 게시판이 존재하는지 확인하고 입력값의 boardName과 일치하는지 확인
- findBoardByClubName() : 해당 클럽 이름으로 클럽을 조회하고 클럽의 키값으로 게시판을 조회한 후에 dto객체로 전환
- modify() : 수정하려는 게시판이 존재하는지 확인하고, adminEmail이 바뀌었을땐 해당 이메일이 클럽에 가입되어있는지 확인 후에 업데이트
- remove() : 해당 게시판이 존재하는지 확인하고 게시판에 있는 게시글들을 모두 삭제한 후에 게시판 삭제

#### 게시글 서비스 로직
1. 접근해야 하는 저장소 : 클럽 스토어, 멤버 스토어
2. 고려해야할 예외사항 처리 및 흐름
- register() : 멤버가 존재하는지 확인하고 게시판이 존재하는지 확인 한 후에 생성
- find() : 조회하려는 게시글이 존재하는지 확인
- findByBoardId() : 조회하려는 게시판이 존재하는 지 확인하고 게시판의 전체글을 조회
- modify() : 수정하려는 게시글이 존재하는지 확인하고 입력값들을 알맞게 setting 한 후에 업데이트
- remove() : 삭제하려는 게시글이 존재하는지 확인하고 삭제

---

### Think (생각)
게시판 서비스 로직에서 객체간의 관계가 행위로 정의된다는 것을 알았습니다.