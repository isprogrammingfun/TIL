## 아이템 1 - 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 public 생성자다. 하지만 꼭 알아야 할 기법이 하나 더 있다. 클래스는 생성자와 별도로 정적 팩터리 메서드를 제공할 수 있다. 여기서 정적 팩터리 메서드는 그 클래스의 인스턴스를 반환하는 단순한 정적 메서드를 말한다. 

> 이 정적 팩터리 메서드는 디자인 패턴에서의 팩터리 메서드와는 다르다. 디자인 패턴 중에는 이와 일치하는 패턴은 없다.
> 

다음 코드는 boolean 기본 타입의 박싱 클래스인 Boolean에서 발췌한 간단한 예시다. 이 메서드는 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해준다.

```java
public static Boolean valueOf(boolean b) {
		return b ? Boolean.TRUE : Boolean.FALSE;
}
```

클래스는 클라이언트에 public 생성자 대신 (혹은 생성자와 함께) 정적 팩터리 메서드를 제공할 수 있다. 이 방식에는 장점과 단점 모두 존재한다. 정적 팩터리 메서드가 생성자보다 좋은 장점은 다음과 같다.

**이름을 가질 수 있다**  

생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다. 반면 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다. 

예를 들어 생성자인 **BigInteger(int, int, Random)** 과 정적 팩터리 메서드인 **BigInteger.probablePrime** 중 어느 쪽이 값이 소수인 BigInteger를 반환한다는 의미를 더 잘 설명할 것 같은가?

하나의 시그니처로는 생성자를 하나만 만들 수 있다. 입력 매개변수들의 순서를 다르게 한 생성자를 새로 추가하는 식으로 이 제한을 피해볼 수 있으나, 좋지 않은 발상이다. 앞서 말한 방법대로 하면 각 생성자가 어떤 역할을 하는지 정확히 기억하기 어려워 엉뚱한 것을 호출하는 실수를 할 수 있다. 코드를 읽는 사람도 클래스 설명 문서를 찾아보지 않고는 의미를 알지 못할 것이다.

이름을 가질 수 있는 정적 팩터리 메서드에는 이런 제약이 없다. 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어줘야 한다. 

**호출될 때마다 인스턴스를 새로 생성하지 않아도 된다**

이 덕분에 불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다. 대표적인 예인 Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다. 따라서 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다. 플라이웨이트 패턴도 이와 비슷한 기법이라 할 수 있다.

반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어느 인스턴스를 살아 있게 할지를 철저히 통제할 수 있다. 이런 클래스를 인스턴스 통제 클래스라고 한다. 

왜 인스턴스를 통제할까? 인스턴스를 통제하면 클래스를 싱글턴으로 만들 수도, 인스턴스화 불가로 만들 수도 있다. 또한 불변 값 클래스에서 동치인 인스턴스가 단 하나 뿐임을 보장할 수 있다. 즉, a==b 일 때만 a.equals(b)가 성립한다. 인스턴스 통제는 플라이웨이트 패턴의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장한다. 

**반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다**

이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성’을 선물한다. API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 해당 객체를 반환할 수 있어 API를 작게 유지할 수 있다. 이는 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도 하다.

자바 8 전에는 인터페이스에 정적 메서드를 선언할 수 없었다. 그렇기 때문에 이름이 “Type”인 인터페이스를 반환하는 정적 메서드가 필요하다면, “Types” 라는 인스턴스화가 불가능한 동반 클래스를 만들어 그 안에 정의하는 것이 관례였다. 자바 컬렉션 프레임워크는 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙인 총 45개의 유틸리티 구현체를 제공하는데, 이 구현체 대부분을 단 하나의 인스턴스화 불가 클래스인 java.util.Collections에서 정적 팩터리 메서드를 통해 얻도록 했다. 

컬렉션 프레임워크는 이 45개의 클래스를 공개하지 않기 때문에 API 외견을 훨씬 작게 만들 수 있었다. API가 작아진 것은 프로그래머가 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도도 낮췄다. 프로그래머는 명시한 인터페이스대로 동작하는 객체를 얻을 것임을 알기에 굳이 별도로 문서를 찾아가며 실제 구현 클래스가 무엇인지 알아보지 않아도 된다. 나아가 정적 팩터리 메서드를 사용하는 클라이언트는 얻은 객체를 그 구현 클래스가 아닌 인터페이스만으로 다루게 된다. 

자바 8부터는 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸기 때문에 인스턴스화 불가 동반 클래스를 둘 이유가 별로 없다. 동반 클래스에 두었던 public 정적 멤버들 상당수를 그냥 인터페이스 자체에 두면 되는 것이다. 

하지만 정적 메서드들을 구현하기 위한 코드 중 많은 부분은 여전히 별도의 package-private 클래스에 두어야 할 수 있다. 자바 8에서도 인터페이스에는 public 정적 멤버만 허용하기 때문이다. 자바 9에서는 private 정적 메서드까지 허락하지만 정적 필드와 정적 멤버 클래스는 여전히 public이어야 한다.

**입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**

예를 들어 EnumSet 클래스는 public 생성자 없이 오직 정적 팩터리만 제공하는데, OpenJDK에서는 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다. 원소가 64개 이하면 원소들을 long 변수 하나로 관리하는 RegularEnumSet의 인스턴스를, 65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스를 반환한다.

클라이언트는 이 두 클래스의 존재를 모른다. 즉, 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고, 알 필요도 없다. EnumSet의 하위 클래스이기만 하면 된다.

**정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**

이런 유연함은 서비스 제공자 프레임워크를 만드는 근간이 된다. 대표적인 서비스 제공자 프레임워크로는 JDBC가 있다. 서비스 제공자 프레임워크에서의 제공자는 서비스의 구현체이다. 그리고 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여, 클라이언트를 구현체로부터 분리해준다.

서비스 제공자 프레임워크는 3개의 핵심 컴포넌트로 다음과 같이 이뤄진다.

- 서비스 인터페이스 - 구현체의 동작을 정의
- 제공자 등록 API - 제공자가 구현체를 등록할 때 사용
- 서비스 접근 API - 클라이언트가 서비스의 인스턴스를 얻을 때 사용

클라이언트는 서비스 접근 API를 사용할 때 원하는 구현체의 조건을 명시할 수 있다. 조건을 명시하지 않으면 기본 구현체를 반환하거나 지원하는 구현체들을 하나씩 돌아가며 반환한다. 이 서비스 접근 API가 서비스 제공자 프레임워크의 근간이라고 한 유연한 정적 팩터리의 실체다. 

3개의 핵심 컴포넌트와 더불어 종종 서비스 제공자 인터페이스라는 네 번째 컴포넌트가 쓰이기도 한다. 이 컴포넌트는 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명해준다. 서비스 제공자 인터페이스가 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 한다. 

- JDBC에서는 Connection이 서비스의 인터페이스 역할을, DriverManager.registerDriver가 제공자 등록 API 역할을, DriverManager.getConnection이 서비스 접근 API 역할을, Driver가 서비스 제공자 인터페이스 역할을 수행한다.

서비스 제공자 프레임워크 패턴에는 여러 변형이 있다. 예컨대 서비스 접근 API는 공급자가 제공하는 것보다 더 풍부한 서비스 인터페이스를 클라이언트에 반환할 수 있다. 이를 브리지 패턴이라 한다. 의존 객체 주입 프레임워크도 강력한 서비스 제공자라고 생각할 수 있다. 자바 6부터는 java.utill.ServiceLoader라는 범용 서비스 제공자 프레임워크가 제공되어 프레임워크를 직접 만들 필요가 없어졌다. 

정적 팩터리 메서드에도 단점은 있다. 단점은 다음과 같다.

**상속을 하려면 public이나 protected 생성자가 필요하기 때문에 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다**

앞서 이야기한 컬렉션 프레임워크의 유틸리티 구현 클래스들을 상속할 수 없다는 이야기이다. 어찌 보면 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.

**정적 팩터리 메서드는 프로그래머가 찾기 어렵다**

생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다. 

다음은 정적 팩터리 메서드에 흔히 사용하는 명명 방식들이다. 

- from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
    - ex) Date d = Date.from(instant);
- of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
    - ex) Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
- valueOf : from과 of의 더 자세한 버전
    - ex) BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
- instance 혹은 getInstance : (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.
    - ex) StackWalker luke = StackWalker.getInstance(options);
- create 혹은 newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
    - ex) Object newArray = Array.newInstance(classObject, arrayLen);
- getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”는 팩터리 메서드가 반환할 객체의 타입이다.
    - ex) FileStore fs = Files.getFileStore(path)
- newType : newInstace와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”는 팩터리 메서드가 반환할 객체의 타입이다.
    - ex) BufferedReader br = Files.newBufferedReader(path);
- type : getType와 newType의 간결한 버전
    - ex) List<Complaint> litany = Collections.list(legacyLitany);