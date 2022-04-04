## UI-UX Development(2), Open Layers
`2022.04.04. -`

## Introduction
지반침하 모니터링 서비스 개발 프로젝트의 폴리곤 그리기 기능을 별도의 버튼으로 생성, 바인딩하고 표출 시스템의 UI/UX를 개선한다.

---

## Task
* [x] welcome 페이지 접근 핸들링 기능 추가
* [ ] 폴리곤 그리기 기능을 별도의 버튼으로 생성 후 토글 형식으로 그리기 기능 바인딩
* [ ] 표출 시스템, 차트의 UI/UX 전반 개선

---

## Details
### Task #1
작성한 임의의 welcome 페이지의 1회성 접근을 쿠키를 통해 핸들링한다. 현재 사용자가 회원 계정으로 로그인하지 않은 경우, 
동시에 welcome 페이지 접근을 통해 생성된 쿠키 `visitedWelcome`를 브라우저에 가지고 있지 않은 경우에 한 해 표출하도록 한다.
사용자가 welcome 페이지에 재접근을 원하는 경우는 header의 info 메뉴를 클릭해서 접근할 수 있도록 한다.  


1. 먼저, 아래와 같이 HTML 내 버튼을 생성하고, 쿠키 생성 및 경로 이동 함수를 생성해 onclick 이벤트로 붙여준다.
2. welcome 페이지에서 해당 버튼을 클릭하면 쿠키를 생성하고, 메인 페이지로 이동한다.
3. `/` 경로의 controller 매핑 메소드에서 현재 로그인 중인 사용자이거나, welcome 페이지를 경유해 생성한 쿠키가 있는지 검사한다.
4. 3의 조건문이 충족되면: 즉, 로그인 중인 사용자이거나 welcome 페이지에서 go to main as guest 버튼을 클릭한 적 있다면 index로 보낸다.
5. 3의 조건문이 충족되지 않으면: 즉, 비회원이며 welcome 페이지에서 go to main as guest 버튼을 클릭하지 않았다면 welcome 페이지로 보낸다.
6. 추후 재방문을 원하는 회원 또는 welcome 페이지의 방문 기록이 있는 비회원을 위해 navbar의 info 탭에 welcome 경로를 매핑해 둔다.
7. 마지막으로, 비로그인 사용자의 방문을 spring security가 막지 않도록 `/welcome` 경로를 permit 처리하는 것을 잊지 않는다.

정리하면:  
1. 다음과 같은 HTML 버튼을 생성하고,  
```html
<button class="btn btn-primary" onclick="goMainAsGuest()">
    <span th:text="#{go-to-main}"></span>
</button>
```

2. 쿠키 생성 및 경로 이동 함수를 생성 후 onclick으로 붙이고,  
```javascript
goMainAsGuest=()=>{
    generateCookie("visitedWelcome", "1");
    window.location.href = "/";
}

generateCookie=(cookieName, cookieValue)=>{ 
    document.cookie = encodeURIComponent(cookieName) + "=" + encodeURIComponent(cookieValue); 
}
```

3. 쿠키 존재 및 로그인 여부에 따라 `/` 경로의 매핑 페이지를 다르게 하며,  
```java
@RequestMapping("/")
public String index(
        Model model, HttpServletResponse response, @CurrentUser UserPrincipal currentUser, HttpServletRequest httpRequest,
        @RequestParam(value = "page", required = false, defaultValue = DEFAULT_PAGE_NUMBER) Integer page,
        @RequestParam(value = "size", required = false, defaultValue = "3") Integer size
    ) {
        ...
        return dependOnIsVisitedWelcome(currentUser, httpRequest);
        }

    public String dependOnIsVisitedWelcome(@CurrentUser UserPrincipal currentUser, HttpServletRequest httpRequest) {
        if(currentUser != null || getCookieValue(httpRequest, "visitedWelcome").equals("1")){
            return "index";
        } else { return "welcome"; }
    }
```

4. Navbar의 info 탭 dropdown에 경로를 매핑 해 둔다.
```html
<li class="dropdown" style="height: 60px;">
    <a th:href="@{/welcome}" class="nav-link" style="padding: 20px 14px; height: 54px; color: var(--light);">
        <i class="fas fa-info-circle" style="font-size: 1.4em"></i>
        <p th:text="#{info.title}"></p>
    </a>
</li>
```

5. Spring security에 `/welcome` 경로의 방문을 비인증 사용자도 가능하도록 허용해 둔다.
```java
    private static final String[] AUTH_WHITELIST = {
        ... 
        "/welcome"
    };

@Override
protected void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests()
        .antMatchers(AUTH_WHITELIST).permitAll()
        ...
}
```


완료 결과는 다음과 같다:  

![](../../../Assets/images/info-welcome.gif)

---

### Task #2

완료 결과는 다음과 같다:  

---

## Remark

---

## Reference

---
