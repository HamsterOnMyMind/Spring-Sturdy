# 스프링 핵심 원리 - 기본편 정리 (1~3강)

---

# 1. 객체 지향 설계와 스프링

### 스프링의 핵심
> **스프링은 좋은 객체 지향 애플리케이션을 개발할 수 있게 도와주는 프레임워크**

- 객체 지향 특징: 추상화, 캡슐화, 상속, **다형성**
- 객체 지향의 목적: 프로그램을 **유연하고 변경이 용이**하게

### 다형성 = 역할과 구현의 분리
- 실세계 비유: 운전자–자동차(K3, 아반떼, 테슬라), 공연–배우(로미오 역할 / 장동건, 원빈)
- **역할 = 인터페이스, 구현 = 인터페이스를 구현한 클래스**
- 클라이언트는 역할만 알면 됨 → 구현 내부 구조나 구현 자체가 바뀌어도 영향 없음
- 설계 순서: **역할(인터페이스)을 먼저 정하고 → 구현 객체를 만든다**
- 객체는 혼자 있지 않고 **협력**함 (클라이언트: 요청, 서버: 응답)
- **다형성의 본질**: 클라이언트를 변경하지 않고 서버의 구현 기능을 실행 시점에 유연하게 변경
- **한계**: 역할(인터페이스) 자체가 바뀌면 클라이언트·서버 모두 크게 변경 → **인터페이스를 안정적으로 설계하는 게 중요**

### SOLID (로버트 마틴)
| 원칙 | 한 줄 정의 | 핵심 포인트 |
|---|---|---|
| **SRP** 단일 책임 | 한 클래스는 하나의 책임만 | 기준은 **변경**. 변경 시 파급효과가 적으면 잘 지킨 것 |
| **OCP** 개방-폐쇄 | 확장에는 열려 있고 변경에는 닫혀 있어야 | 다형성 활용. 단, 클라이언트가 구현체를 직접 `new` 하면 못 지킴 |
| **LSP** 리스코프 치환 | 하위 타입으로 바꿔도 프로그램 정확성 유지 | 컴파일 성공을 넘어 **인터페이스 규약**을 지켜야 함 (엑셀=전진) |
| **ISP** 인터페이스 분리 | 범용 하나보다 특정 클라이언트용 여러 개 | 자동차 → 운전 / 정비 인터페이스 분리 |
| **DIP** 의존관계 역전 | 추상화에 의존, 구체화에 의존 X | 구현 클래스 말고 인터페이스(역할)에 의존 |

### 결론: 다형성만으로는 부족하다
```java
public class MemberService {
//  private MemberRepository memberRepository = new MemoryMemberRepository();
    private MemberRepository memberRepository = new JdbcMemberRepository();
}
```
- 인터페이스에 의존하지만 **구현 클래스도 직접 선택(new)** → 구현을 바꾸면 클라이언트 코드 수정 → **OCP·DIP 위반**
- 해결: 객체를 생성하고 연결해주는 **별도의 조립·설정자** 필요 → 스프링의 **DI + DI 컨테이너**
- 순수 자바로 OCP, DIP를 지키며 개발하다 보면 결국 DI 컨테이너(=스프링)를 만들게 됨
- **실무 고민**: 인터페이스 도입 = 추상화 비용. 확장 가능성이 없으면 구체 클래스를 쓰다가 필요할 때 리팩터링해서 인터페이스 도입

## 꼬리물기 Q&A

**Q1. 다형성 = 오버라이딩인가?**
- A. 자바에서 다형성을 구현하는 주된 문법이 오버라이딩. 인터페이스 타입 변수에 어떤 구현 객체가 들어 있느냐에 따라 **오버라이딩된 메서드가 실행**됨. 클래스 상속으로도 가능.
- ↳ **Q1-1. 오버로딩은 다형성이 아닌가?**
  - A. 넓게는 "정적(컴파일 타임) 다형성"이라 부르기도 하지만, 이 강의에서 말하는 건 **실행 시점에 구현체를 갈아끼우는 동적 다형성**. 오버로딩은 컴파일 시점에 호출 메서드가 정해지므로 교체 유연성과는 무관.
- ↳ **Q1-2. 역할을 인터페이스 대신 추상 클래스로 만들면?**
  - A. 가능하지만, 자바는 단일 상속이라 구현체가 다른 클래스를 상속할 수 없게 됨. 공통 구현 코드가 필요한 게 아니라면 **인터페이스가 역할 표현에 더 적합**.

**Q2. "인터페이스를 안정적으로 설계"한다는 건 구체적으로 뭔가?**
- A. 구현이 바뀌어도 인터페이스(메서드 시그니처, 의미)는 바뀌지 않도록 **클라이언트가 필요로 하는 것 기준으로** 설계. 예) `DiscountPolicy.discount(member, price)` 는 정액이든 정률이든 "할인 금액을 돌려준다"는 계약만 지킴.
- ↳ **Q2-1. 그래도 인터페이스가 바뀌어야 하면?**
  - A. 그게 다형성의 한계(자동차→비행기, USB 규격 변경). ISP로 인터페이스를 잘게 나눠두면 변경 영향 범위를 줄일 수 있음.

**Q3. SRP의 "하나의 책임"은 어떻게 판단하나?**
- A. 크기가 아니라 **변경의 이유**가 하나인가로 판단. 예) UI가 바뀔 때 비즈니스 로직 클래스까지 고쳐야 하면 책임이 섞인 것.
- ↳ **Q3-1. 이 강의에서의 SRP 사례는?**
  - A. "객체의 생성과 사용 분리". 3강에서 클라이언트(`OrderServiceImpl`)가 가진 생성·연결·실행 책임 중 생성·연결을 `AppConfig`로 떼어냄.

**Q4. OCP는 "확장하면서 변경하지 않는다"인데 말이 되나?**
- A. 다형성으로 **새 구현 클래스를 추가**하는 건 확장, 기존 클라이언트 코드는 건드리지 않는 게 변경 닫힘. 단, 누군가 `new 새구현체()`로 바꿔줘야 하므로 그 역할을 **클라이언트 밖으로 빼야** 비로소 OCP 성립.

**Q5. DIP의 "역전"은 무엇이 뒤집힌다는 건가?**
- A. 전통적으로 상위 모듈(서비스)이 하위 모듈(구체 저장소)을 직접 의존. DIP는 둘 다 **추상(인터페이스)에 의존**하게 해서, 의존 방향이 구체 → 추상으로 뒤집힘.
- ↳ **Q5-1. DIP, DI, IoC는 다 같은 말인가?**
  - A. 다름.
    - **DIP**: 설계 원칙(추상에 의존하라)
    - **DI**: 그 원칙을 지키는 **방법** 중 하나(외부에서 의존 객체를 넣어줌)
    - **IoC**: 더 넓은 개념(프로그램 제어 흐름을 외부가 가짐). DI는 IoC의 한 형태.

---

# 2. 스프링 핵심 원리 이해1 - 예제 만들기

## 핵심 요약

## 꼬리물기 Q&A

**Q1. 왜 `MemberService` / `MemberServiceImpl` 처럼 인터페이스 + Impl 구조로 만드나?**
- A. 요구사항이 미확정이라 **역할을 먼저 고정**하고 구현은 나중에 바꾸기 위해. 구현체가 하나뿐이면 관례적으로 `Impl` 접미사를 붙임.
- ↳ **Q1-1. 서비스는 구현이 바뀔 일이 거의 없는데 굳이?**
  - A. 맞음. 1강 "실무 고민"처럼, 실제로 교체 가능성이 큰 건 `MemberRepository`, `DiscountPolicy`. 서비스 인터페이스는 팀 규칙이나 테스트 전략에 따라 생략하기도 함.

**Q2. `HashMap` 대신 `ConcurrentHashMap`을 쓰라는 이유는?**
- A. 웹 애플리케이션은 여러 요청이 동시에 같은 저장소에 접근. `HashMap`은 thread-safe하지 않아 동시 쓰기 시 데이터 유실·손상 가능. `ConcurrentHashMap`은 동시 접근에 안전.
- ↳ **Q2-1. id 생성도 같이 신경 써야 하나?**
  - A. 실무라면 그렇음(여러 스레드가 시퀀스를 올리면 `AtomicLong` 등 사용). 이 예제는 id를 직접 넣으므로 생략.

**Q3. 주문 서비스가 존재하지 않는 회원 id로 호출되면?**
- A. `findById`가 null 반환 → `FixDiscountPolicy`에서 `member.getGrade()` 호출 시 **NPE**. 예제라 생략된 예외 처리.
- ↳ **Q3-1. 실무에선 어디서 막아야 하나?**
  - A. 서비스 레이어에서 회원 존재를 검증하고 의미 있는 예외(예: `IllegalArgumentException`, 커스텀 예외)를 던지는 게 일반적. 저장소가 `Optional<Member>`를 반환하게 설계하기도 함.

**Q4. 금액을 `int`로 처리해도 되나?**
- A. 학습용으로는 충분. 실무에선 큰 금액(약 21억 초과) 오버플로, 소수점 통화, 반올림 정책 때문에 `long`이나 `BigDecimal`을 씀.

**Q5. 테스트에서 `isEqualTo`로 비교했는데, 같은 객체인지 비교하려면?**
- A. `isEqualTo`는 `equals()` 비교, `isSameAs`는 참조(==) 비교. `Member`가 `equals`를 재정의하지 않았으므로 지금은 둘 다 참조 비교와 같고, 메모리 저장소가 **같은 객체를 그대로 반환**하므로 통과. DB 저장소라면 조회 시 새 객체가 만들어질 수 있어 결과가 달라질 수 있음.

**Q6. 그래서 이 코드의 근본 문제는 정확히 뭔가?**
- A. `OrderServiceImpl`이 `DiscountPolicy`(추상)와 `FixDiscountPolicy`(구체)에 **동시에 의존**. 정책을 바꾸려면 클라이언트 코드를 수정해야 함 → 다음 강의의 출발점.

---

# 3. 스프링 핵심 원리 이해2 - 객체 지향 원리 적용

## 핵심 요약

### ① 새로운 할인 정책 개발
- 기획 변경: 고정 1000원 → **정률 10%** (10,000원 → 1,000원, 20,000원 → 2,000원)
- `RateDiscountPolicy` 추가 + 테스트(VIP 10% 적용 / BASIC 미적용, `@DisplayName`)
- 다형성 덕분에 **새 정책을 개발하는 것 자체는 문제 없음**

### ② 적용 시 문제점
```java
// private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
```
- 정책을 바꾸려면 **클라이언트(`OrderServiceImpl`) 코드 수정** 필요
- **DIP 위반**: 인터페이스(`DiscountPolicy`)와 구체 클래스(`FixDiscountPolicy`)에 동시 의존
- **OCP 위반**: 기능 확장 시 클라이언트 코드 변경
- 인터페이스에만 의존하도록 `private DiscountPolicy discountPolicy;` 로 바꾸면 → 구현체가 없어서 **NPE**
- 해결: 누군가 **대신 구현 객체를 생성하고 주입**해야 함

### ③ 관심사의 분리 → AppConfig 등장
- 비유: 로미오 배우(디카프리오)가 줄리엣 배우를 직접 섭외 = 공연 + 섭외라는 **다양한 책임**
- 배우는 연기에만 집중, 섭외는 **공연 기획자**가 담당
- **AppConfig = 공연 기획자**: 구현 객체를 **생성**하고 **생성자를 통해 주입(연결)**

```java
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```
- `OrderServiceImpl`은 이제 **추상에만 의존**(DIP 완성), 어떤 구현이 들어올지 모름, **실행에만 집중**
- 클라이언트 입장에서 의존관계를 외부에서 넣어주는 것 같아서 **DI(Dependency Injection, 의존관계 주입)**
- 테스트도 `@BeforeEach`에서 `AppConfig`로 객체를 받아오도록 수정

### ④ AppConfig 리팩터링
```java
public class AppConfig {
    public MemberService memberService() { return new MemberServiceImpl(memberRepository()); }
    public OrderService orderService() { return new OrderServiceImpl(memberRepository(), discountPolicy()); }
    public MemberRepository memberRepository() { return new MemoryMemberRepository(); }
    public DiscountPolicy discountPolicy() { return new FixDiscountPolicy(); }
}
```
- `new MemoryMemberRepository()` **중복 제거**
- **역할(메서드 반환 타입)과 구현(new 대상)이 한눈에** 보임 → 애플리케이션 전체 구성 파악이 쉬움

### ⑤ 새 구조에서 할인 정책 변경
- 애플리케이션이 **사용 영역**과 **구성 영역**으로 분리
- `discountPolicy()`의 `new FixDiscountPolicy()` → `new RateDiscountPolicy()` **한 줄만 변경**
- 사용 영역(`OrderServiceImpl` 등)은 변경 없음. 구성 영역은 당연히 변경됨(기획자는 모든 배우를 알아야 함)

### ⑥ SOLID 적용 정리
- **SRP**: 생성·연결 책임은 AppConfig, 실행 책임은 클라이언트
- **DIP**: 클라이언트는 추상에만 의존, 구현 객체는 AppConfig가 주입
- **OCP**: 정책을 확장해도 **사용 영역의 변경은 닫혀 있음**

### ⑦ IoC, DI, 컨테이너
- **IoC(제어의 역전)**: 객체가 스스로 의존 객체를 생성·연결·실행하던 제어 흐름을 **외부(AppConfig)가 가져감**. `OrderServiceImpl`조차 AppConfig가 생성
- **프레임워크 vs 라이브러리**: 프레임워크가 내 코드를 제어·실행하면 프레임워크(JUnit), 내 코드가 흐름을 제어하면 라이브러리
- **DI**: 의존관계는 두 가지로 분리해서 봐야 함
  - **정적 클래스 의존관계**: import만 보고 판단, 실행 없이 분석 가능(클래스 다이어그램)
  - **동적 객체 인스턴스 의존관계**: 런타임에 실제 생성된 객체 참조로 연결(객체 다이어그램)
  - DI를 쓰면 **정적 의존관계는 그대로 두고 동적 의존관계만 바꿀 수 있음**
- **IoC 컨테이너 / DI 컨테이너**: AppConfig처럼 객체를 생성·관리하고 의존관계를 연결해주는 것 (어셈블러, 오브젝트 팩토리)

### ⑧ 스프링으로 전환
```java
@Configuration
public class AppConfig {
    @Bean public MemberService memberService() { ... }
    @Bean public OrderService orderService() { ... }
    @Bean public MemberRepository memberRepository() { ... }
    @Bean public DiscountPolicy discountPolicy() { return new RateDiscountPolicy(); }
}
```
```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
MemberService memberService = ac.getBean("memberService", MemberService.class);
```
- `ApplicationContext` = **스프링 컨테이너**
- `@Configuration` 클래스를 설정 정보로 사용, `@Bean` 메서드를 모두 호출해 반환 객체를 컨테이너에 등록 = **스프링 빈**
- 빈 이름 = **메서드 이름** (memberService, orderService)
- 이제 객체를 AppConfig에서 직접 꺼내지 않고 **컨테이너에서 `getBean()`으로 조회**

### 전체 흐름 한 장 요약
| 단계 | 상황 | 결과 |
|---|---|---|
| 새 정책 개발 | 다형성으로 `RateDiscountPolicy` 추가 | 문제 없음 |
| 정책 적용 | 클라이언트가 구체 클래스를 `new` | DIP·OCP 위반 |
| 관심사 분리 | AppConfig가 생성·주입 담당 | DIP 해결, SRP 준수 |
| 리팩터링 | 중복 제거, 역할/구현 명시 | 구성 한눈에 파악 |
| 정책 변경 | AppConfig 한 줄 수정 | OCP 준수 |
| 스프링 전환 | `@Configuration` + `@Bean` | 스프링 컨테이너가 관리 |

## 꼬리물기 Q&A

**Q1. `RateDiscountPolicy`의 `price * discountPercent / 100`에 함정은 없나?**
- A. 두 가지.
  - **정수 나눗셈 절사**: 15원 주문 시 15*10/100 = 1(1.5가 아님). 반올림/올림 정책이 필요하면 별도 처리
  - **연산 순서**: `price / 100 * 10` 으로 쓰면 99원 이하는 할인이 0이 됨. 곱하기를 먼저 하는 이유
- ↳ **Q1-1. 그럼 곱하기 먼저면 안전한가?**
  - A. `int` 기준 약 2억 1천만 원 이상이면 `price * 10`에서 **오버플로**. 실무는 `long`이나 `BigDecimal` + `RoundingMode` 사용.


**Q2. 왜 setter나 필드에 바로 넣지 않고 "생성자"로 주입하나?**
- A. ① 필드를 `final`로 둘 수 있어 **불변 보장**, ② 필수 의존성이 빠지면 **객체 생성 자체가 안 됨**(NPE를 컴파일·생성 시점에 차단), ③ 테스트에서 가짜 구현을 넣기 쉬움.
- ↳ **Q2-1. 아까 `private DiscountPolicy discountPolicy;`만 두고 실행하면 NPE가 났던 것과 연결하면?**
  - A. 그 NPE가 바로 "주입하는 사람이 없음" 문제. 생성자 주입은 주입 없이는 객체를 못 만들게 강제해서 같은 실수를 원천 차단.

**Q3. AppConfig에서 `memberRepository()`가 `memberService()`와 `orderService()`에서 각각 호출되는데, 그럼 저장소 객체가 2개 생기는 거 아닌가?**
- A. 순수 자바 AppConfig에서는 **맞음, 2개 생성**됨. 그래도 동작하는 건 2강의 `static Map store` 덕분에 데이터가 공유되기 때문.
- ↳ **Q3-1. `@Configuration` + `@Bean`으로 바꾸면 달라지나?**
  - A. 달라짐. 스프링은 `@Configuration` 클래스를 CGLIB로 조작해 `@Bean` 메서드를 여러 번 호출해도 **같은 인스턴스(싱글톤)** 를 돌려줌. 강의 로그에서도 `memberRepository`가 한 번만 생성됨. 이게 "스프링 컨테이너를 쓰는 장점" 중 하나이고, 이후 강의(싱글톤 컨테이너)의 주제.

**Q4. 구성 영역(AppConfig)은 정책 바뀔 때마다 수정되는데, 그럼 OCP 위반 아닌가?**
- A. OCP의 대상은 **사용 영역(비즈니스 로직)**. 구성 영역은 "어떤 배우를 쓸지 정하는 곳"이라 바뀌는 게 정상. 변경 지점을 **한 곳으로 모은 것** 자체가 목적.

**Q5. IoC와 DI는 같은 말 아닌가?**
- A. IoC는 "제어 흐름을 외부가 가진다"는 넓은 개념, DI는 그중 **의존 객체를 외부에서 넣어주는 구체적 방식**. 예) JUnit이 내 테스트 메서드를 대신 호출하는 것도 IoC지만 DI는 아님.
- ↳ **Q5-1. 프레임워크 vs 라이브러리를 ERP·실무에 대입하면?**
  - A. 내가 정의한 훅·컨트롤러를 프레임워크가 알아서 호출 → 프레임워크. 내가 필요할 때 함수를 import해서 호출 → 라이브러리. 판별 기준은 "**누가 호출 흐름을 쥐고 있나**".

**Q6. 정적 의존관계와 동적 의존관계를 왜 굳이 나눠서 보나?**
- A. DI의 효과가 정확히 그 둘을 분리하는 것이기 때문. 클래스 다이어그램(`OrderServiceImpl` → `DiscountPolicy`)은 **고정**인 채로, 런타임 객체 다이어그램(→ Fix 또는 Rate)만 바꿀 수 있음. 코드 수정 없이 동작 교체가 가능한 이유.

**Q7. `@Bean` 이름을 메서드명으로 쓴다면, 메서드명이 같으면?**
- A. 빈 이름 충돌. `@Bean(name = "...")`으로 직접 지정 가능하지만, **빈 이름은 항상 다르게** 주는 게 원칙. 같은 이름이면 설정에 따라 덮어쓰기되거나 오류가 남(스프링 부트는 기본적으로 오류).
- ↳ **Q7-1. `getBean()`에 이름 없이 타입만 줘도 되나?**
  - A. 가능(`getBean(MemberService.class)`). 단, 같은 타입 빈이 2개 이상이면 어떤 걸 줄지 몰라 예외. → 이후 "빈 조회" 강의 내용.

**Q8. 스프링 전환 후 코드가 오히려 길어졌는데, 왜 쓰나?**
- A. 지금 단계에선 이득이 안 보임. 앞으로 나올 장점: 싱글톤 관리, 컴포넌트 스캔과 자동 주입(`@Autowired`)으로 AppConfig 수동 작성 감소, 빈 생명주기 관리, AOP 적용, 다양한 설정 형식 지원 등.
- ↳ **Q8-1. `ApplicationContext`와 `BeanFactory`는 무슨 차이?**
  - A. `BeanFactory`가 빈 생성·조회의 최상위 인터페이스, `ApplicationContext`는 이를 상속하고 메시지 국제화, 이벤트, 환경 변수, 리소스 조회 같은 부가 기능을 더한 것. 실무에선 거의 `ApplicationContext`만 씀. 1강에서 로드 존슨 책에 이미 있었다던 그 두 개념.

**Q9. 테스트에서 매번 `new AppConfig()`를 하는데, 테스트끼리 독립적인가?**
- A. 객체는 매번 새로 만들어지지만 저장소의 `static store`는 **공유**되므로 완전 독립은 아님. 테스트 데이터 정리 로직이 있어야 안전

---

# 한 줄 결론

> **다형성(역할/구현 분리)만으로는 OCP·DIP를 지킬 수 없다 → 객체 생성·연결 책임을 외부(AppConfig)로 분리(DI/IoC) → 이걸 프레임워크로 만든 게 스프링 DI 컨테이너.**
