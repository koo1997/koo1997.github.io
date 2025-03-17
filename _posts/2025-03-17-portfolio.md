---
layout: post
title: 포트폴리오
date: 2025-03-17
categories: [Spring, SNS, 프로젝트,이력서]
tags: [Spring Boot, Security, JWT, JPA]
---
##  프로젝트 개요

**프로젝트명:** SNS 회원 관리 시스템  (개인)
**사용 기술:** Java, Spring Boot, JPA (Hibernate), Spring Security, JWT, MySQL, Lombok, AOP, Async

###  주요 기능

- 회원 가입 및 로그인 (비밀번호 암호화 적용)
- JWT 기반 인증 및 보안 처리
- 트랜잭션 관리 및 낙관적 락(Optimistic Lock) 재시도 로직 구현
- 비동기 처리(Async)를 활용한 팔로우 기능
- 예외 처리 및 글로벌 핸들러 적용
- 사용자 정보 조회 및 수정 기능
- 팔로우 및 언팔로우 기능 구현
- 데이터 정합성을 위한 동시성 제어

---

##  기술 스택

| 카테고리        | 사용 기술                                               |
| --| --|
| **백엔드**     | Spring Boot, Spring Security, JPA (Hibernate)       |
| **DB**      | MySQL                                               |
| **보안**      | JWT, BCrypt 비밀번호 해싱                                 |
| **트랜잭션**    | Spring Transaction (Optimistic Lock, REQUIRES\_NEW) |
| **비동기 처리**  | @Async (Thread Pool 활용)                             |
| **API 문서화** | Swagger                                             |
| **로그 관리**   | SLF4J + Logback                                     |

---

##  프로젝트 상세 기능

###  회원 가입 및 로그인 기능

- `BCryptPasswordEncoder`를 활용하여 비밀번호 암호화 저장
- 로그인 시 `JWT` 발급 및 사용자 인증
- 예외 발생 시 `GlobalExceptionHandler`를 통한 공통 예외 처리
- 로그인 ID 중복 검사 기능 추가
- 회원 정보 업데이트 기능 구현

```java
@Autowired
private BCryptPasswordEncoder passwordEncoder;

public void createUserLogin(User user, String loginId, String rawPassword) {
    if (userLoginRepository.findByLoginId(loginId).isPresent()) {
        throw new IllegalArgumentException("이미 사용 중인 로그인 ID입니다.");
    }
    
    UserLogin userLogin = new UserLogin();
    userLogin.setUser(user);
    userLogin.setLoginId(loginId);
    userLogin.setPassword(passwordEncoder.encode(rawPassword));
    userLoginRepository.save(userLogin);
}
```

###  JWT 기반 보안 처리

- `io.jsonwebtoken`을 사용하여 JWT 토큰 발급 및 검증
- 만료된 토큰 및 위조된 토큰 예외 처리
- 인증 필터 적용하여 Spring Security와 연동
- 사용자 권한(Role)에 따라 접근 제어 구현

```java
@ExceptionHandler(SignatureException.class)
public ResponseEntity<Map<String, String>> handleSignatureException(SignatureException e) {
    Map<String, String> response = new HashMap<>();
    response.put("error", "Invalid JWT signature");
    return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(response);
}
```

###  트랜잭션 관리 및 낙관적 락(Optimistic Lock) 처리

- `@Transactional`을 활용한 데이터 일관성 보장
- `OptimisticLockingFailureException` 발생 시 **재시도 로직 적용**
- `@Version` 어노테이션을 사용하여 버전 관리
- `REQUIRES_NEW`를 활용한 독립적인 트랜잭션 처리

```java
public void doInNewTransactionOptimisticRetry(Runnable runnable, int retryCount, long backOff) {
    int tryCount = 0;

    while (tryCount < retryCount) {
        DefaultTransactionDefinition def = new DefaultTransactionDefinition();
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRES_NEW);
        TransactionStatus txStatus = txManager.getTransaction(def);

        try {
            runnable.run();
            txManager.commit(txStatus);
            return;
        } catch (OptimisticLockingFailureException olfe) {
            txManager.rollback(txStatus);
            log.info("[Retrying Transaction] count: {}", tryCount);
            tryCount++;
            Thread.sleep(backOff);
        } catch (Exception e) {
            txManager.rollback(txStatus);
            throw new IllegalStateException("Transaction failed", e);
        }
    }
    throw new IllegalStateException("Transaction retry limit exceeded");
}
```

###  비동기 처리 (@Async 활용)

- 팔로우 기능 비동기 처리로 **성능 최적화**

```java
@Async
public void increaseFolloweeCount(User user) {
    transactionService.doInNewTransactionOptimisticRetry(() ->
        user.setFolloweeCount(user.getFolloweeCount() + 1), 3, 100);
}
```

---

## 성과 및 개선점

 **트랜잭션 안정성 강화** → Optimistic Lock을 활용한 충돌 해결  
 **비밀번호 보안 강화** → `BCryptPasswordEncoder` 적용  
 **비동기 처리로 성능 개선** → `@Async` 적용하여 팔로우 기능 최적화  
 **JWT 인증 적용** → Spring Security + JWT로 인증 강화  
 **사용자 경험 향상** → 회원 정보 수정 및 중복 ID 검사 기능 추가  

---

##  프로젝트를 통해 배운 점

- **Spring Security와 JWT를 활용한 보안 처리 방법**
- **트랜잭션 관리 및 낙관적 락을 이용한 동시성 제어 기법**
- **비동기 처리를 통해 API 응답 속도를 개선하는 방법**
- **Spring Boot의 예외 처리 및 로깅 기법 적용**
- **JPA와 MySQL을 활용한 데이터 모델링 및 성능 최적화**

---

##  GitHub Repository

- GitHub: [바로가기](https://github.com/koo1997/SNS-)
