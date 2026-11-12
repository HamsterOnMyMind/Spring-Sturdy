<div align="center">

# 🌱 스프링 핵심 원리 — 실무 요약 노트

**4. 컨테이너와 빈 · 5. 싱글톤 · 6. 컴포넌트 스캔 · 7. 자동 의존관계 주입 · 8. 빈 생명주기 · 9. 빈 스코프**


</div>

---

## 📑 목차

| # | 주제 | 실무 핵심 한 줄 |
|:-:|------|----------------|
| 4 | [스프링 컨테이너와 빈](#4-스프링-컨테이너와-빈) | `ApplicationContext` = 빈 관리 컨테이너. 타입 조회 시 중복이면 예외 |
| 5 | [싱글톤 컨테이너](#5-싱글톤-컨테이너) | 빈은 기본 싱글톤 → **무상태(stateless)** 로 설계 |
| 6 | [컴포넌트 스캔](#6-컴포넌트-스캔) | `@Component` 계열 + 스캔 위치는 **프로젝트 최상단** |
| 7 | [의존관계 자동 주입](#7-의존관계-자동-주입) | **생성자 주입 + `final` + `@RequiredArgsConstructor`** |
| 8 | [빈 생명주기 콜백](#8-빈-생명주기-콜백) | `@PostConstruct` / `@PreDestroy` 만 기억 |
| 9 | [빈 스코프](#9-빈-스코프) | 싱글톤이 기본, 필요 시 `ObjectProvider` / 프록시 |

---

## 4. 스프링 컨테이너와 빈

### ✅ 핵심

```java
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
MemberService memberService = ac.getBean(MemberService.class); // 타입으로 조회
```

| 개념 | 실무 포인트 |
|------|------------|
| **ApplicationContext** | 스프링 컨테이너. `BeanFactory` 기능 + 메시지/환경변수/이벤트/리소스 등 부가기능. **실무에선 항상 이걸 씀** |
| **빈 이름** | 기본은 메서드명(@Bean) / 클래스명 첫 글자 소문자(@Component). **이름 중복 금지** |
| **타입 조회** | 같은 타입이 2개 이상이면 `NoUniqueBeanDefinitionException` |
| **상속 관계** | 부모 타입으로 조회하면 **자식 타입 빈까지 전부** 조회됨 (`Object`로 조회 시 모든 빈) |

> 💡 실무에서 `getBean()`을 직접 호출할 일은 거의 없음. 대부분 **DI로 주입**받는다. 다만 "타입 중복 시 예외" 원리는 7장 `@Autowired` 충돌 해결의 기반이 되므로 중요.

<details>
<summary>🗑️ 생략한 내용</summary>

XML 설정(`GenericXmlApplicationContext`), `BeanDefinition` 메타정보 — 스프링 부트 환경에서 직접 다룰 일 거의 없음.
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> BeanFactory 대신 ApplicationContext를 쓰는 이유는?</summary>

> `BeanFactory`는 빈 조회·관리만 제공. `ApplicationContext`는 여기에 **국제화(MessageSource), 환경변수(Environment: local/dev/prod 프로필), 이벤트 발행, 리소스 조회**를 추가로 제공한다. 실무 애플리케이션에 필요한 기능이 전부 들어있어서 사실상 항상 ApplicationContext를 쓴다.
</details>

<details>
<summary><b>Q2.</b> 같은 타입의 빈이 2개일 때 타입으로 조회하면? 어떻게 해결?</summary>

> `NoUniqueBeanDefinitionException` 발생. 조회 시엔 **빈 이름을 같이 지정**하거나, 주입 시엔 **`@Primary` / `@Qualifier`** 로 해결한다(7장).
</details>

<details>
<summary><b>Q3.</b> 부모 타입으로 조회하면 무엇이 조회되나? 이게 실무에서 왜 중요?</summary>

> 부모 타입으로 조회하면 **자식 빈들까지 모두** 조회된다. 그래서 인터페이스 하나에 구현체가 여러 개면 주입 시 충돌이 나고, 반대로 `List<인터페이스>` / `Map<String, 인터페이스>` 로 **구현체를 한 번에 주입**받는 전략 패턴이 가능해진다.
</details>

---

## 5. 싱글톤 컨테이너

### ✅ 핵심

- 웹 애플리케이션은 요청이 동시에 많이 들어옴 → 요청마다 객체 생성하면 메모리 낭비
- 스프링 컨테이너는 **빈을 싱글톤으로 관리** (싱글톤 패턴 코드 없이도)
- `@Configuration` 은 **CGLIB** 바이트코드 조작으로 `@Bean` 메서드가 여러 번 호출돼도 **같은 객체**를 반환하게 보장

### ⚠️ 실무에서 제일 중요한 것: **무상태(stateless) 설계**

```java
// ❌ 절대 금지 — 공유 필드에 상태 저장
@Service
public class StatefulService {
    private int price;                       // 모든 요청이 공유!
    public void order(String name, int price) {
        this.price = price;                  // 사용자A 값을 사용자B가 덮어씀
    }
}

// ✅ 지역변수 / 파라미터 / 반환값으로 처리
@Service
public class StatelessService {
    public int order(String name, int price) {
        return price;
    }
}
```

| 규칙 | 설명 |
|------|------|
| 🚫 특정 클라이언트에 의존적인 필드 X | 사용자별 데이터 저장 금지 |
| 🚫 클라이언트가 값을 변경할 수 있는 필드 X | setter로 바꾸는 필드 금지 |
| ✅ 읽기 전용 / 지역변수 / 파라미터 / `ThreadLocal` 사용 | 필드는 주입받은 의존성(final)만 |

> 🔥 이 실수는 **운영에서 다른 고객 데이터가 섞이는** 대형 장애로 이어진다. 테스트로는 잘 안 잡힘.

<details>
<summary>🗑️ 생략한 내용</summary>

싱글톤 패턴 직접 구현(`private` 생성자 + `static getInstance()`) — 스프링이 대신 해주므로 직접 작성할 일 없음.
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> 스프링 빈에 상태(필드)를 두면 어떤 문제가 생기나?</summary>

> 빈은 하나의 인스턴스를 모든 스레드(요청)가 공유한다. A 요청이 저장한 값을 B 요청이 덮어써서 **다른 사용자의 데이터가 노출/저장**되는 동시성 버그가 생긴다. 필드는 `final`로 주입받은 의존성만 두고, 요청 데이터는 지역변수·파라미터로 다룬다.
</details>

<details>
<summary><b>Q2.</b> AppConfig에서 memberRepository()를 여러 번 호출하는데 왜 객체가 1개인가?</summary>

> `@Configuration` 클래스는 스프링이 CGLIB으로 만든 **자식 프록시 클래스**로 등록된다. 이 프록시가 `@Bean` 메서드 호출을 가로채서 컨테이너에 이미 있으면 그 빈을 반환한다. 그래서 싱글톤이 보장된다.
</details>

<details>
<summary><b>Q3.</b> @Configuration 없이 @Bean만 쓰면?</summary>

> 빈 등록은 되지만 CGLIB 프록시가 없어서 **메서드 직접 호출 시 매번 new 객체**가 생성 → 싱글톤 깨짐. 설정 클래스에는 **항상 `@Configuration`** 을 붙인다.
</details>

<details>
<summary><b>Q4.</b> 요청별로 상태를 꼭 유지해야 하면?</summary>

> 파라미터로 넘기거나, `ThreadLocal`을 쓰거나(요청 끝에 반드시 `remove()`), **request 스코프 빈**(9장)을 사용한다.
</details>

---

## 6. 컴포넌트 스캔

### ✅ 핵심

```java
@SpringBootApplication   // 안에 @ComponentScan 포함
public class CoreApplication { }   // ← 프로젝트 최상단 패키지에 위치
```

| 애노테이션 | 용도 | 스프링 부가기능 |
|-----------|------|----------------|
| `@Component` | 일반 컴포넌트 | - |
| `@Controller` | 웹 MVC 컨트롤러 | MVC 컨트롤러로 인식 |
| `@Service` | 비즈니스 로직 | 특별 기능 없음 (계층 표시용) |
| `@Repository` | 데이터 접근 | **DB 예외 → 스프링 예외로 변환** |
| `@Configuration` | 설정 | CGLIB 싱글톤 보장 |

### 📌 실무 규칙

- **스캔 위치**: `basePackages` 지정하지 말고, **메인 클래스를 프로젝트 최상단**에 두는 게 관례 (부트 기본)
- **이름 충돌**
  - 자동 vs 자동 → `ConflictingBeanDefinitionException`
  - 수동 vs 자동 → 과거엔 수동 우선, **스프링 부트는 기본적으로 오류로 막음** (`spring.main.allow-bean-definition-overriding=false`)

<details>
<summary>🗑️ 생략한 내용</summary>

`includeFilters` / `excludeFilters` / `FilterType` — `@Component`로 충분하고 필터는 거의 안 씀. (excludeFilters 정도만 가끔)
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> 새로 만든 @Service가 빈으로 등록되지 않는다. 원인은?</summary>

> 대부분 **메인 클래스(@SpringBootApplication)보다 상위/다른 패키지**에 클래스를 만들어서 스캔 범위 밖에 있는 경우. 메인 클래스 패키지 하위로 옮긴다.
</details>

<details>
<summary><b>Q2.</b> @Repository를 붙이면 @Component와 무엇이 다른가?</summary>

> 데이터 접근 계층에서 발생한 **DB 벤더별 예외를 스프링 공통 예외(`DataAccessException`)로 변환**해준다. 그래서 DB를 바꿔도 서비스 계층 예외 처리 코드가 안 바뀐다.
</details>

<details>
<summary><b>Q3.</b> 수동 빈과 자동 빈 이름이 같으면?</summary>

> 순수 스프링은 수동 빈이 덮어쓰지만(경고 로그), **스프링 부트는 기본값으로 애플리케이션 실행 시 오류**를 낸다. 의도치 않은 덮어쓰기 버그를 막기 위한 것. 이름 충돌은 만들지 않는 게 원칙.
</details>

---

## 7. 의존관계 자동 주입

### ✅ 결론: **생성자 주입 + final + Lombok**

```java
@Service
@RequiredArgsConstructor               // final 필드로 생성자 자동 생성
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

| 주입 방식 | 실무 사용 | 이유 |
|-----------|:--------:|------|
| **생성자 주입** | ⭐ **기본** | 불변(`final`), 누락 시 **컴파일 오류**, 순수 자바 테스트 가능 |
| 수정자(setter) 주입 | 드묾 | 선택적·변경 가능한 의존관계에만 |
| 필드 주입 | ❌ | 테스트 불가, DI 프레임워크 없으면 동작 X (테스트 코드에서만 허용) |
| 일반 메서드 주입 | ❌ | 거의 안 씀 |

> 생성자가 **1개면 `@Autowired` 생략 가능**.

### 🔀 같은 타입 빈이 2개 이상일 때

```java
@Component @Primary                       // 기본으로 쓸 빈
public class RateDiscountPolicy implements DiscountPolicy {}

@Component @Qualifier("fixDiscountPolicy") // 특별히 지정해서 쓸 빈
public class FixDiscountPolicy implements DiscountPolicy {}
```

| 방법 | 언제 |
|------|------|
| `@Primary` | **메인으로 쓰는 빈** (예: 메인 DB 커넥션) |
| `@Qualifier` | **가끔 쓰는 서브 빈** 지정 (예: 서브 DB). 우선순위 `@Qualifier` > `@Primary` |
| `List` / `Map` 주입 | **모든 구현체를 받아 동적 선택** (전략 패턴) |

```java
// 전략 패턴 — 할인 정책을 코드로 동적 선택
@RequiredArgsConstructor
public class DiscountService {
    private final Map<String, DiscountPolicy> policyMap; // key = 빈 이름
    public int discount(Member m, int price, String code) {
        return policyMap.get(code).discount(m, price);
    }
}
```

### 📌 자동 vs 수동 빈 등록 기준

| 구분 | 방식 |
|------|------|
| **업무 로직** (Controller / Service / Repository) | **자동** (컴포넌트 스캔) |
| **기술 지원 로직** (AOP, DB 설정, 공통 로그 등) | **수동** `@Configuration` + `@Bean` 으로 명확히 드러내기 |
| 다형성 활용 비즈니스 로직 (구현체 여러 개) | 수동 등록 or **같은 패키지에 묶기** — 한눈에 파악되도록 |

<details>
<summary>🗑️ 생략한 내용</summary>

옵션 처리(`required=false`, `@Nullable`, `Optional<>`), 애노테이션 직접 만들기(`@MainDiscountPolicy`), 수정자/필드/메서드 주입 상세.
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> 왜 생성자 주입을 써야 하나? (면접 단골)</summary>

> ① **불변**: 의존관계는 앱 실행 중 바뀔 일이 없으므로 `final`로 고정
> ② **누락 방지**: 의존성 빠지면 컴파일 오류
> ③ **테스트 용이**: 스프링 없이 `new`로 Mock 주입 가능
> ④ **순환참조를 앱 시작 시점에 감지**
</details>

<details>
<summary><b>Q2.</b> 같은 타입 빈 2개 → NoUniqueBeanDefinitionException. 어떻게 해결?</summary>

> 자주 쓰는 쪽에 `@Primary`, 가끔 쓰는 쪽은 주입 지점에서 `@Qualifier("이름")` 으로 지정. 둘 다 있으면 `@Qualifier`가 우선. 모두 필요하면 `List`/`Map`으로 주입.
</details>

<details>
<summary><b>Q3.</b> @RequiredArgsConstructor 쓸 때 @Qualifier가 안 먹히는 문제?</summary>

> Lombok이 생성한 생성자 파라미터에는 필드의 `@Qualifier`가 기본으로 복사되지 않는다. `lombok.config`에 `lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier` 를 추가하거나, 그 클래스만 생성자를 직접 작성한다.
</details>

<details>
<summary><b>Q4.</b> 구현체가 여러 개인 정책(할인/결제수단 등)을 런타임에 고르려면?</summary>

> `Map<String, 인터페이스>` 로 주입받아 **빈 이름을 키로** 꺼내 쓴다. if-else 분기 없이 구현체만 추가하면 확장되는 전략 패턴.
</details>

---

## 8. 빈 생명주기 콜백

### ✅ 이것만 쓰면 됨

```java
@Component
public class NetworkClient {

    @PostConstruct          // 의존관계 주입 완료 후 호출
    public void init() {
        connect();          // 커넥션 연결, 캐시 로딩 등
    }

    @PreDestroy             // 컨테이너 종료 직전 호출
    public void close() {
        disconnect();       // 커넥션 종료, 자원 반납
    }
}
```

> **생명주기**: 컨테이너 생성 → 빈 생성 → **의존관계 주입** → `@PostConstruct` → 사용 → `@PreDestroy` → 종료

| 상황 | 방법 |
|------|------|
| 내가 작성한 클래스 | ⭐ `@PostConstruct` / `@PreDestroy` |
| **외부 라이브러리** (코드 수정 불가) | `@Bean(initMethod = "init", destroyMethod = "close")` |

> 💡 `@Bean`의 `destroyMethod` 기본값은 **추론(inferred)** → `close`, `shutdown` 메서드를 자동 호출. 외부 라이브러리 대부분 그냥 동작함.

<details>
<summary>🗑️ 생략한 내용</summary>

`InitializingBean` / `DisposableBean` 인터페이스 — 스프링 전용 인터페이스에 의존, 외부 라이브러리 적용 불가. 지금은 거의 안 씀.
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> 초기화 로직을 생성자에 넣으면 안 되나?</summary>

> 생성자 시점엔 아직 **의존관계 주입이 끝나지 않았을 수 있다**(수정자 주입 등). 또 생성자는 필수값 세팅이라는 단일 책임만 갖고, 커넥션 연결 같은 **무거운 작업은 분리**하는 게 유지보수에 좋다. 그래서 `@PostConstruct`에서 한다.
</details>

<details>
<summary><b>Q2.</b> 외부 라이브러리(HikariCP 등)의 초기화/종료는?</summary>

> 코드에 애노테이션을 붙일 수 없으므로 `@Bean(initMethod=..., destroyMethod=...)` 를 사용. destroyMethod는 기본 추론이라 `close()`가 있으면 따로 안 적어도 호출된다.
</details>

<details>
<summary><b>Q3.</b> 프로토타입 빈에서 @PreDestroy가 호출되나?</summary>

> **호출되지 않는다.** 스프링은 프로토타입 빈을 생성·주입·초기화까지만 하고 관리에서 손을 뗀다. 종료 처리는 받은 클라이언트가 직접 해야 한다(9장).
</details>

---

## 9. 빈 스코프

### ✅ 스코프 종류

| 스코프 | 생존 범위 | 실무 사용 |
|--------|----------|:--------:|
| **singleton** | 컨테이너 시작 ~ 종료 (기본값) | ⭐⭐⭐ 99% |
| prototype | 조회할 때마다 새로 생성, **종료 메서드 호출 X** | 드묾 |
| **request** | HTTP 요청 하나 들어와서 나갈 때까지 | 가끔 (요청별 로그 추적 등) |
| session / application | 세션 / 서블릿 컨텍스트 범위 | 드묾 |

### ⚠️ 함정: 싱글톤이 짧은 스코프 빈을 주입받으면?

- 싱글톤은 **생성 시점에 1번만 주입** → 프로토타입이 매번 새로 안 만들어짐
- request 스코프는 앱 시작 시점에 요청이 없어서 **주입 자체가 실패**

### 🛠️ 해결 1 — `ObjectProvider` (필요한 시점에 꺼내기)

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject(); // 이 시점에 조회
        myLogger.log("service id = " + id);
    }
}
```

### 🛠️ 해결 2 — 프록시 (코드가 싱글톤처럼 깔끔)

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger { ... }

// 사용 쪽은 그냥 주입
private final MyLogger myLogger;   // 실제로는 CGLIB 가짜 프록시가 주입됨
```

> 💡 프록시는 호출 시점에 진짜 request 빈을 찾아 위임한다. **편하지만 싱글톤처럼 보여서 착각하기 쉬움** → 꼭 필요한 곳에만, 최소한으로.

<details>
<summary>🗑️ 생략한 내용</summary>

프로토타입 상세 테스트, `ApplicationContext` 직접 조회(DL), JSR-330 `Provider`(`javax.inject`), `@Lookup` — `ObjectProvider`로 충분.
</details>

### 💬 꼬리질문

<details>
<summary><b>Q1.</b> 싱글톤 빈 안에서 프로토타입 빈을 주입받아 쓰면 매번 새 객체인가?</summary>

> 아니다. 싱글톤 생성 시점에 **딱 1번 주입**된 프로토타입을 계속 재사용한다. 매번 새로 받으려면 `ObjectProvider.getObject()`로 사용 시점에 조회한다.
</details>

<details>
<summary><b>Q2.</b> request 스코프 빈을 싱글톤에 주입했더니 앱이 안 뜬다. 왜?</summary>

> 앱 시작 시점엔 HTTP 요청이 없어서 request 빈을 만들 수 없기 때문(`ScopeNotActiveException`). `ObjectProvider`로 지연 조회하거나 `proxyMode = ScopedProxyMode.TARGET_CLASS`로 프록시를 주입한다.
</details>

<details>
<summary><b>Q3.</b> request 스코프는 실무에서 어디에 쓰나?</summary>

> 동시 요청이 섞인 로그에서 **요청별 UUID를 붙여 추적**하는 용도가 대표적. 요청 단위로 공유해야 하는 정보(요청 ID, 사용자 컨텍스트 등)를 파라미터로 계속 넘기지 않아도 된다. (실무에선 MDC + 필터/인터셉터 방식도 많이 씀)
</details>

<details>
<summary><b>Q4.</b> 프록시 방식의 주의점은?</summary>

> 겉보기엔 싱글톤 필드처럼 보이지만 실제로는 **요청마다 다른 객체**에 위임된다. 동작 방식을 모르면 디버깅이 어렵고, 요청이 없는 곳(배치·스케줄러)에서 호출하면 예외가 난다. 남용 금지.
</details>

---

<div align="center">

## 🧭 한 장 요약

| 질문 | 답 |
|------|----|
| 빈은 기본적으로? | **싱글톤** → 필드에 상태 저장 금지 |
| 설정 클래스엔? | **`@Configuration`** (CGLIB 싱글톤 보장) |
| 빈 등록은? | 업무 로직 **자동**, 기술 지원 **수동** |
| 주입은? | **생성자 + `final` + `@RequiredArgsConstructor`** |
| 빈이 2개 이상? | **`@Primary`**(메인) / **`@Qualifier`**(서브) / **`Map`**(전략) |
| 초기화·종료? | **`@PostConstruct` / `@PreDestroy`**, 외부 라이브러리는 `@Bean(initMethod, destroyMethod)` |
| 짧은 스코프 주입? | **`ObjectProvider`** or **프록시** |
</div>
