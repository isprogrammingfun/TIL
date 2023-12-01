## 아이템 4 - 인스턴스화를 막으려거든 private 생성자를 사용하라

- 정적 메서드만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한 클래스가 아니다.

```java
public class UtilityClass {
	public static String hello() {
		return "hello";
	}

	public static void main(String[] args) {
		UtilityClass.hello();
	}
}
```

- 스태틱한 메서드는 인스턴스 없이 클래스를 통해 바로 호출이 가능하다. 인스턴스를 통해서도 가능은 하나, 권장되는 방법이 아니다.
- 그렇기에 이렇게 인스턴스를 만드는 것을 아예 방지하자는 것이 이번 아이템의 핵심 목표 중 하나이다.

### 추상클래스

```java
public abstract class UtilityClass {
	public UtilityClass() {
		System.out.println("Constructor");
	}

	public static String hello() {
		return "hello";
	}

	public static void main(String[] args) {
		UtilityClass.hello();
	}
}
```

```java
public class DefaultUtilityClass extends UtilityClass {
	public static void main(String[] args) {
		DefaultUtilityClass utilityClass = new DefaultUtilityClass();
	}

}
```

- 그러나 추상클래스로 만들어도 인스턴스는 만들어질 수 있다. ]
- abstract로 인스턴스화가 막힌다면 생성자 부분이 호출이 되지 않아야 한다.
- 그러나 만약 우리가 하위 클래스를 만들면 인스턴스를 만들 수 있게 된다.
- 위에 코드처럼 만들면 자식 클래스의 인스턴스를 만들 때 기본 생성자에서 상위 클래스의 생성자를 기본으로 호출한다.
- 그리고 오히려 abstract라는 키워드 때문에 상속해서 사용하라는 의미로 오해할 수 있다.

### private 생성자

```java
public class UtilityClass {
 //왜 private 생성자를 만들었는지에 대한 주석을 작성해두는것이 좋다. 
	private UtilityClass() {
		throw new AssertionError();
	}

	public static String hello() {
		return "hello";
	}

	public static void main(String[] args) {
		UtilityClass.hello();
	}
}
```

- private 생성자를 추가하면 외부에서는 해당 생성자를 사용할 수 없기 때문에 클래스의 인스턴스화를 막을 수 있다.
- private 생성자를 추가하면 내부에서는 사용할 수 있다. 이를 막기 위해서는 `AssertionError` 를 추가하면 된다.
- 그러나 이 방법을 사용하면 생성자가 보이는데 굳이 생성자를 만들면서까지 생성자를 못 쓰게 하는 형식이라 약간의 아이러니함이 있다.