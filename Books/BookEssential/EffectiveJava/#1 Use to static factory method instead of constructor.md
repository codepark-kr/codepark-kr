### \#1 생성자 대신 정적 팩터리 메서드를 고려하라

[내용 추가 예정]

### Static Factory Method 명명 방식
> **from** : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드  
> e.g. `Date d = Date.from(instant);`  
>
> **of** : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드  
> e.g. `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`  
> 
> **valueof** : from과 of의 더 자세한 버전  
> e.g. `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`  
>
> **instance** | **getInstance** : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.  
> e.g. `StackWalker luke = StackWalker.getInstance(options);`  
> 
> **create** | **newInstance** : instance 혹은 getInstance와 같지만, 매 번 새로운 인스턴스를 생성해 반환함을 보장한다.  
> e.g. `Object newArray = Array.newInstance(classObject, arrayLen);`
> 
> **getType** : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.  
> e.g. `FileStore fs = Files.getFileStore(path);`
> 
> **newType** : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. "Type"은 팩터리 메서드가 반환할 객체의 타입이다.  
> e.g. `BufferedReader br = Files.newBufferedReader(path);`  
> 
> **type** : getType과 newType의 간결한 버전  
> e.g. `List<Complaint> litany = Collections.list(legacyLitany);`

---

### 핵심 정리
정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.
그렇다고 하더라도 **정적 팩터리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.**

---
