## SSE(Server Sent Event), JPA
`2022.04.06. -`

## Introduction
기존 지반침하 프로젝트에서 과도한 트랜젝션이 발생하여 HikariCP Connection Timeout(Deadlock)이 발생하는 이슈를 확인하고, 원인을 해결한다.

---

## Task
* [x] HikariCP Connection Timeout 원인 파악
* [x] SSE Emitter의 발송 완료/에러 발생한 emit close 처리
* [ ] JPA 로직 전반 불필요한 트랜젝션 발생 로직 리팩토링
---

## Details
### Task #1, #2
프로젝트에 주기적으로 HikariCP Connection Timeout이 발생하는 것을 확인하였다.  
먼저, 해당 이슈의 원인을 파악하도록 한다.

레퍼런스로 참고한 [HikariCP Dead lock에서 벗어나기 (실전편)](https://techblog.woowahan.com/2663/) 페이지에서는 원인을 찾기 위해, 
`hikari:leak-detection-threshold`를 선언할 수 있다고 명시 해 두었다. 직접 한번 작성해보자 :  

```properties
spring.datasource.hikari.leak-detection-threshold=2000
spring.datasource.hikari.connection-test-query=SELECT 1 FROM DUAL
```

![](../../../Assets/images/leak-detection.png)  

돌려보니 바로 잡힌다.   
**명백한 누수가 감지됨(Apparent connection leak detected).**  
원인은 SSE(Server Sent Event) 실시간 알림 메시지의 발송이 완료한 후에도 complete(close) 처리를 하지 않았기 때문이었다.  


아래의 코드에 다음과 같이 2줄을 추가해 준다.

```java
final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

@Override
@Transactional
public void notifyStatusChanged(HttpServletRequest httpRequest, StatusChangeRequest request) {
    request.setMessage(i18nMessage(httpRequest, request));
    List<SseEmitter> deadEmitters = new ArrayList<>();

    if(request.getPaidDataId() != null){
        notificationRepository.save(notificationBuilder(httpRequest, request));
        emitters.forEach(emitter -> {
            try {
                emitter.send(SseEmitter.event().data(request.getMessage()));
                emitter.complete(); // 발송 후 complete 처리
            }
            catch (Exception e) {
                deadEmitters.add(emitter);
                emitter.completeWithError(e);  // 에러 발생시 complete 처리 및 throw error
            } 
        });
    }
    emitters.removeAll(deadEmitters);
}
```

SSE initialize를 위해 작성한 controller 내 구문에도 다음과 같이 complete 구문을 추가해준다 :  

```java
private final ExecutorService executor = Executors.newCachedThreadPool();

@GetMapping(value = "/notification", produces = "text/event-stream")
public ResponseEntity<SseEmitter> doNotify(HttpServletRequest httpRequest, StatusChangeRequest request) throws Exception {
    notificationService.notifyStatusChanged(httpRequest, request);
    return new ResponseEntity<>(initializeSSE(), HttpStatus.OK);
}

public SseEmitter initializeSSE() throws Exception {
    SseEmitter emitter = new SseEmitter(Long.MAX_VALUE);
    executor.execute(() -> {
        try{
            emitter.send(SseEmitter.event().name("connected"));
            notificationService.addEmitter(emitter);
            emitter.onCompletion(() -> notificationService.removeEmitter(emitter));
        } catch(Exception e){
            log.error("Error while sending SSE for connection : caused by : {}", e.getMessage());
            emitter.completeWithError(e); // 에러 발생시 complete 처리 및 throw error
        }
        emitter.onTimeout(() -> notificationService.removeEmitter(emitter));
    });
    return emitter;
}
```

더이상 누수가 발생하지 않는지 확인해 보자.  
hikari maximum-pool-size를 1로 잡고 서버를 재시작 해 보면 connection 누수가 해결되었음을 확인할 수 있다.  

```properties
spring.datasource.hikari.maximum-pool-size=1
```


---

### Task #2



---

## Remark

---

## Reference

[HikariCP Dead lock에서 벗어나기 (실전편)](https://techblog.woowahan.com/2663/)

---
