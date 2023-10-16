## 아이템 3 - private 생성자나 열거 타입으로 싱글턴임을 보증하라

애플리케이션을 만들다 보면 어떤 인스턴스가 애플리케이션 내에서 여러 인스턴스가 필요하지 않은 경우가 있다. 어떤 타입의 인스턴스가 오로지 **하나만 있어도 되는** 경우, 또는 **하나만 유지를 해야 하는** 경우가 존재한다. 즉, **하나만 유지해도 좋은 경우**가 있고, 반드시 **하나만 유지해야 하는 경우**도 있다. 

하나만 유지해도 좋은 경우는 여러 개를 만들어서 사용해도 크게 시스템적으로 문제는 없지만, 하나만 만들었을 때 리소스를 조금 덜 사용한다거나 메모리를 덜 사용한다거나 또는 객체를 만드는 과정을 좀 더 생략할 수 있다거나 그런 장점이 있을 것이다. 

반면에 반드시 하나만 유지해야 하는 경우라면 여러 개의 인스턴스가 있을 때 오히려 애플리케이션에 문제가 생긴다. 예를 들어 설정을 생각해보면 화면의 크기나 밝기, 사운드 크기 등이 있다. 이런 설정에 대한 인스턴스는 하나만 있으면 된다. 오히려 여러개가 존재하면 헷갈리게 된다. 이러한 인스턴스를 우린 **싱글턴**이라 부른다. 

### private 생성자 + public static final 필드

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() {}

	public void leaveTheBuilding() {
		System.out.println("Whoa baby, I'm outta here!");
	}
	public void sing() {
		System.out.println("I'll have a blue~ Christmas without you~");
	}

	public static void main (String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
	}
}
```

위에 코드를 보면 Elvis 생성자를 Private으로 만들면 외부에서는 Elvis 생성자를 호출할 수가 없게 된다. 오로지 해당하는 클래스 안에서만 Elvis라는 생성자를 호출할 수가 있게 된다. 

여기서 싱글턴을 생하는 부분은 

```java
public static final Elvis INSTANCE = new Elvis();
```

이 부분이다. `public static` 필드로 INSTANCE라는 본인의 타입으로 인스턴스를 미리 만들어두고, 인스턴스 메서드들을 제공한다. 

Elvis를 사용하는 클라리언트들은 다음과 같은 방식으로 사용할 것이다.

```java
public static void main (String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
	}
```

Elvis.INSTANCE를 통해 인스턴스를 가져온 다음에 인스턴스 메서드를 호출할 것이다. 

private 생성자를 사용하면 좋은 점은 간편하고 보기 좋다는 점이다. 생성자가 private이기 때문에 인스턴스를 직접 만들지 못하게 막았음을 알 수 있고, static final로 선언을 해두면 자바 도큐먼트를 만들 때 별도의 필드로 보여준다. 

이 방식의 첫번째 단점은 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트 코드를 테스트하기 어려워진다는 점이다. 특히나 인터페이스 없이 정의한 경우에는 싱글톤을 사용하는 클라이언트 코드를 테스트하기가 어려워진다. 

```java
public class Concert {
	private boolean lightsOn;

	private boolean mainStateOpen;

	private Elvis elvis;
	
	public Concert(Elvis elvis) {
		this.elvis = elvis;
	}

	public void perform() {
		mainStateOpen = true;
		lightsOn = true;
		elvis.sing();
	}

	public boolean isLightsOn() { return lightsOn; }
	public boolean isMainStateOpen() { return mainStateOpen; }
}

class ConcertTest {

	@Test
	void perform() {
		Concert concert = new Concert(Elvis.INSTANCE);
		concert.perform();

		assertTrue(concert.isLightsOn());
		assertTrue(concert.isMainStateOpen());
	}
}
```

Concert 라는 클래스가 있고, 이 Concert가 Elvis를 사용하고 있다. 즉, 이 Concert라는 인스턴스 자체가 Elvis의 클라이언트 코드이다. 현재 이 클라언트 코드는 테스트하기가 어렵다. 만약 오퍼레이션(객체의 기능) 자체가 외부 네트워크를 거쳐서 어떤 다른 외부 API를 호출하거나 연산이 굉장히 오래 걸리는 작업일 수도 있다. 테스트를 실행할 때마다 그런 오퍼레이션들을 호출하는 것은 비효율적이다. 또한 코드에선 인터페이스를 구현해서 만든 싱글턴이 아니기 때문에 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없는 상황이다. 

```java
public interface IElvis {
	void leaveTheBuilding();
	void sing();
}

public class MockElvis implements IElvis{
	@Override
	public void leaveTheBuilding() {
	}

	@Override
	public void sing() { System.out.println("You ain't nothin' but a hound dog.");
}

public class Concert {
	private boolean lightsOn;

	private boolean mainStateOpen;

	private IElvis elvis;
	
	public Concert(IElvis elvis) {
		this.elvis = elvis;
	}

	public void perform() {
		mainStateOpen = true;
		lightsOn = true;
		elvis.sing();
	}

	public boolean isLightsOn() { return lightsOn; }
	public boolean isMainStateOpen() { return mainStateOpen; }
}

class ConcertTest {

	@Test
	void perform() {
		Concert concert = new Concert(new MockElvis());
		concert.perform();

		assertTrue(concert.isLightsOn());
		assertTrue(concert.isMainStateOpen());
	}
}
```

만약 인터페이스 기반으로 코딩을 바꾸면 테스트를 할 때 가짜 Elvis를 사용할 수 있다. 이렇게 되면 매번 리소스가 드는 오퍼레이션이 아니라 Mock을 사용해 원하는대로 테스트가 가능하다.  

두번째 단점으로는 리플렉션을 사용하면 싱글턴이 깨진다는 점이다. 즉, 같은 타입의 인스턴스를 여러 개 만들 수 있다. 

```java
public class ElvisReflection {
	public static void main(String[] args) {
		try {
				// 기본 생성자에 접근 getDeclaredConstructor로는 private한 생성자에도 접근이 가능하다
				Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
				// setAccessible(true) 해줘야 private 생성자 호출이 가능
				defaultConstructor.setAccessible(true);
				Elvis elvis1 = defaultConstructor.newInstance();
				Elvis elvis2 = defaultConstructor.newInstance();
				System.out.println(elvis1 == elvis2); // 결과 : false
		}
		catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException) {
			e.printStackTrace();
		}
	}
}
```

결과는 false가 나온다. 생성자를 여러 번 호출해 각기 다른 인스턴스가 여러 번 만들어지기 때문이다. 이런 문제점을 막기 위해선 생성자가 두 번째 호출 되었을 때는 인스턴스 생성을 막아야 한다. private static한 Boolean으로 어떤 일정의 Flag를 하나 만들어 이미 생성이 되었으면 예외를 던지게 해야 한다. 

```java
	public static final Elvis INSTANCE = new Elvis();
	private static boolean created;
	private Elvis() {
		if (created) {
				throw new UnsupportedOperationException("can't be created by constructor.");
		}
		created = true;
}

public class ElvisReflection {
	public static void main(String[] args) {
		try {
				// 기본 생성자에 접근 getDeclaredConstructor로는 private한 생성자에도 접근이 가능하다
				Constructor<Elvis> defaultConstructor = Elvis.class.getDeclaredConstructor();
				// setAccessible(true) 해줘야 private 생성자 호출이 가능
				defaultConstructor.setAccessible(true);
				Elvis.INSTANCE.sing(); 
				Elvis elvis1 = defaultConstructor.newInstance(); //여기서 부턴 에러 발생
				Elvis elvis2 = defaultConstructor.newInstance(); 
				System.out.println(elvis1 == elvis2); // 결과 : false
		}
		catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException) {
			e.printStackTrace();
		}
	}
}
```

마지막 단점으로는 역직렬화 할 때 새로운 인스턴스가 생길 수 있다는 것이다. 

- 직렬화를 통해 객체 정보를 어딘가에 저장해 놓을 수 있고, 역직렬화로 어딘가에 저장되어 있는 객체 정보를 읽어올 수 있다.

```java
public class ElvisSerialization {
	public static void main(String[] args) {
		try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
				out.writeObject(Elvis.INSTANCE);
			} catch (IOException e) {
					e.printStackTrace();
			}
			
			try (ObjectInput in = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
					Elvis elvis3 = (Elvis) in.readObject();
					System.out.println(elvis3 = Elvis.INSTANCE); // 결과 : false
			} catch (IOException | ClassNotFoundException e) {
					e.printStackTrace();
			}
	}
}
```

이 문제를 해결하려면 readResolve 메서드를 통해 기존에 있는 인스턴스를 리턴해줘야 한다. 

```java
// Elvis elvis3 = (Elvis) in.readObject(); 실행하면 실행
private Object readResolve() {
	return INSTANCE;
}
```

### private 생성자 + 정적 팩터리 메서드

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	public static Elvis getInstance() { return INSTANCE; }
	
	public void leaveTheBuilding() {
		System.out.println("whoa baby, I'm outta here!");
	}

	public static void main(String[] args) {
		Elvis elvis = Elvis.getInstance();
		elvis.leaveTheBuilding();
	}
}
```

첫번째 방법과 다른 점은 public static final 대신에 private으로 선언을 해놓고 get instance라는 public static method를 통해서 가져가도록 하는 것이다. 이 방법 또한 첫번째 방법과 같은 단점을 가지고 있다. 이 방법 또한 private 생성자를 사용하기 때문이다. 

public한 static method를 통해서 인스턴스를 제공할 때의 장점으로는 

첫번째로 API를 바꾸지 않고 `getInstance`를 그대로 쓰면서 행위를 바꿀 수 있다. 원래는 `getInstance` 를 쓰면 계속해서 싱글턴 인스턴스를 가져왔었다. 그러나 매번 새로운 인스턴스를 만들고 싶다면 아래와 같이 안에 동작만 수정하 된다.

```java
	public static Elvis getInstance() { return new Elvis()}
```

이렇게 하면 클라이언트 코드는 바뀌지 않지만 매번 다른 인스턴스를 만들어 줄 수 있게 된다. 

두번째 장점으로는 정적 팩토리를 Generic Singleton Factory로 만들 수 있다는 점이다.  

```java
public class MetaElvis<T> {
	private static final MetaElvis<Object> INSTANCE = new MetaElvis<>();
	
	private MetaElvis() {}
	
// 이미 클래스에 T가 선언되어 있는데 앞에 T가 또 붙은 이유는? 
// 둘다 T타입으로 이름은 같지만 스콥이 다르기 때문이다 
// 클래스에 붙은 T는 인스턴스 스콥이고, 여기에 붙은 T는 스태틱한 스콥이다 
	@SuppressWarnings("unchecked")
	public static <T> MetaElvis<T> getInstance() { return (MetaElvis<T> INSTANCE; }

// 이건 클래스에 선언한 T가 맞다 
	public void say(T t) {
		System.out.println(t);
	}

	public void leaveTheBuilding() {
		System.out.println(t);
	}

	public static void main (String[] args) {
		MetaElvis<String> elvis1 = MetaElvis.getInstance();
		MetaElvis<Integer> elvis2 = MetaElvis.getInstance();
		System.out.println(Objects.equals(elvis1, elvis2); // 결과 : true
	}
}
```

이렇게 Generic한 타입으로 동일한 Singleton Instance를 사용하고 싶을 때 Generic Singleton Factory를 만들어 사용할 수 있다. Generic한 타입을 쓸 때 인스턴스는 동일하지만 Generic Singleton Factory를 사용해 각각이 원하는 타입으로 바꿔서 사용할 수 있다. 

마지막 장점으로는 정적 팩토리의 메서드 참조를 공급자 사용할 수 있다는 점이다. 

공급자(supplier)는 Java 8에 들어있는 functional 인터페이스를 이야기 한다. Java 8에서는 functional 인터페이스라는 어노테이션이 붙어 있는 기본적인 function들을 제공한다. 

Supplier interface만 만족하면, 즉 어떤 타입이든 리턴하기만 한다면 어떤 메서드도 Supplier를 직접 구현하지 않더라도 Supplier Functional 타입으로 사용할 수 있다. 관련 내용은 아래 코드에 있다. 

```java
public class Concert {
	//공급
	public void start(Supplier<Singer> singerSupplier) {
		Singer singer = sungerSuppliser.get();
		singer.sing();
	}

	public static void main(String[] args) {
		Concert concert = new Concert();
		// Singer를 제공해주는 supplier한테 Singer 인스턴스를 만들어주는 오퍼레이션이
		// Supplier Interface에 준하는 기능이기 때문에 인자 없는 메서드 이름은 중요하지 않다.
		// 호출해서 무언가를 하나 리턴해주는 이 인자 없는 메서드가 정확하게 Supplier에
		// 준하는 메서드가 된다. Supplier를 구현하지 않았지만 여기에 준하기 때문에 
		// 메서드 레퍼런스로 Elvis의 GET 인스턴스를 참조해서 GET 인스턴스가 하는 메서드 자체를
		// Supplier의 구현체처럼 파라미터에 넘기고 Supplier에 있는 GET으로 호출하면 GET 인스턴스에서
	  // 리턴하는 Elvis가 넘어오게 된다. Elvis라는 것이 Singer 타입이니까 인터페이스를 통해
		// 싱어를 호출하면 결국에는 Elvis가 가지고 있는 sing이 호출된다.  
		concert.start(Elvis::getInstance);
	}
}

public interface Singer {
	void sing();
}

public class Elvis implements Singer{
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() {}
	public static Elvis getInstance() { return INSTANCE; }
	
	public void leaveTheBuilding() {
		System.out.println("whoa baby, I'm outta here!");
	}

	public static void main(String[] args) {
		Elvis elvis = Elvis.getInstance();
		elvis.leaveTheBuilding();
	}

	// singer 인터페이스 구현 -> Elvis가 Singer가 된다. 
	@Override
	public void sing() {
		System.out.println("my way~~");
	}
}
```

### 열거 타입

```java
public enum Elvis {
	INSTANCE;
	
	public void leaveTheBuilding() { System.out.println("comming~~"); }

	public static void main (String[] args) {
		Elvis elvis = Elvis.INSTANCE;
		elvis.leaveTheBuilding();
	}
}
```

Enum을 사용하면 매우 간단하다. 생성자를 private으로 만들 필요도 없고 public static한 메서드를 만들 필요도 없고, 필드를 정의할 필요도 없다. 그리고 사용할 인스턴스 메서드들을 Enum 안에 정의하고 사용할 수 있다. 이 방법은 리플렉션과, 직렬화, 역직렬화에 굉장히 안전하다.