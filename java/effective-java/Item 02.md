## 아이템 2 - 생성자에 매개변수가 많다면 빌더를 고려하라

다음과 같은 코드가 있다고 해보자.

```java
public class NutritionFacts {
	private final int servingSize; // (mL, 1회 제공량) 필수
	private final int servings; // (회, 총 n회 제공량) 필수
	private final int calories; // (1회 제공량당) 선택
	private final int fat; // (g/1회 제공량) 선택
	private final int sodium; // (mg/1회 제공량) 선택
	private final int carbohydrate; // (g/1회 제공량) 선택

	public NutritionFacts(int servingSize, int servings) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = 0;
		this.fat = 0;
		this.sodium = 0;
		this.carbohydrate = 0;
}

	public NutritionFacts(int servingSize, int servings, int calories) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = 0;
		this.sodium = 0;
		this.carbohydrate = 0;
}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
		this.servingSize = servingSize;
		this.servings = servings;
		this.calories = calories;
		this.fat = fat;
		this.sodium = 0;
		this.carbohydrate = 0;
}
...

}
```

이런 식으로 `NutritionFacts` 클래스에 생성자도 많지만, 생성자에 넘겨주는 파라미터도 많다. 보통 이런 경우들이 흔히 있을 것이다. 클래스가 가지고 있는 여러 필드 중에 일부는 필수적으로 가져야 하는 필드가 있고, 그렇지 않은 필드도 있을 것이다.  

필수로 있어야 하는 필드들 같은 경우는 생성자로 받는게 좋다. 왜냐하면 객체를 만들 때 반드시 생성자를 통해서 받아서 만들 수 있게끔 강제할 수 있어 즉, 필수적인 필드 값을 넘겨받도록 강제할 수 있기 때문이다.

선택적인 필드도 같이 생성자를 통해 넘겨주면 좋으나, 코드가 매우 지저분해진다. 이런 코드를 그나마 줄이기 위해 사용하는 것이 점층적 생성자 패턴이다. 

**점층적 생성자 패턴**

위에 코드를 보면 생성자 코드 중에 대부분 중복이 발생하고 있다. 

이를 해결하기 위해선 매개변수가 적은 쪽에서 매개변수가 많은 쪽에 생성자를 this를 통해 호출한다. 따라서 위에 코드를 다음과 같이 수정할 수 있다. 

```java
public class NutritionFacts {
	private final int servingSize; // (mL, 1회 제공량) 필수
	private final int servings; // (회, 총 n회 제공량) 필수
	private final int calories; // (1회 제공량당) 선택
	private final int fat; // (g/1회 제공량) 선택
	private final int sodium; // (mg/1회 제공량) 선택
	private final int carbohydrate; // (g/1회 제공량) 선택

	public NutritionFacts(int servingSize, int servings) {
			this(servingSize, servings, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories) {
			this(servingSize, servings, calories, 0);
	}

	public NutritionFacts(int servingSize, int servings, int calories, int fat) {
			this(servingSize, servings, calories, fat, 0);
	}
...

public static void main(String[] args) {
		//이 방식의 문제점 : 인스턴스를 만들때 어떤 parameter를 줘야하는지 모른다. 
		new NutritionFacts(240,8,100,0,35,27); 
	}

}
```

보통 이런 생성자는 사용자가 설정하길 원치 않는 매개변수까지 포함하기 쉬운데, 어쩔 수 없이 그런 매개변수에도 값을 지정해줘야 한다. 위에 코드에서는 매개변수가 6개 뿐이라 괜찮아 보일 수 있으나, 매개변수 개수가 늘어나면 금세 걷잡을 수 없게 된다. 

요약하면, 점층적 생성자 패턴을 사용할 수 있으나, 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다. 

두 번째 대안으로는 자바빈즈 패턴이 있다.

**자바빈즈(JavaBeans Pattern) 패턴**

자바빈즈는 자바 표준 스펙 중 하나이다. 매개변수가 없는 생성자로 객체를 만든 후, 세터 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다. 

```java
public class NutritionFacts {
	//필드들 기본값이 있다면 기본값으로 초기화된다.
	private int servingSize = -1; // 필수; 기본값 없음
	private int servings = -1; // 필수; 기본값 없음
	private int calories = 0;
	private int fat = 0;
	private int sodium = 0;
	private int carbohydrate = 0;

	public NutritionFacts() {}
	//세터 메서드들
	public void setServingSize(int servingSize) {
		this.servingSize = servingSize;
	}
	public void setServings(int servings) {
		this.servings = servings;
	}
	public void setCalories(int calories) {
		this.calories = calories;
	}
	...

	public static void main(String[] args) {
		NutritionFacts cocaCola = new NutritionFacts();
		cocaCola.setServingSize(240);
		
		cocaCola.setServings(0);
		cocaCola.setCalories(100);
		cocaCola.setSodium(35);
		cocaCola.setCarbohydrate(27);
	}
}
```

이 방식을 통해 얻을 수 있는 장점은 객체 생성이 간단해진다는 것이다. 그러나 문제는 필수적으로 필요로 하는 필수 값들이 세팅이 안된 상태 그대로 사용이 될 수 있다는 점이다. 이를 일관성이 무너진 상태라고 표현한다.

일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며(setter를 사용하고 있기 때문에 값이 변경될 수 있음) 스레드 안전성을 얻으려면 프로그래머가 추가 작업을 해줘야만 한다. 

> 이런 단점을 완화하고자 생성이 끝난 객체를 수동으로 얼리고(freezing) 얼리기 전에는 사용할 수 없도록 할 수 있으나, 이 방법은 다루기 어려워 실전에서는 거의 사용하지 않는다.
> 

세 번째 대안으로는 빌더 패턴이 있다.

**빌더 패턴 (Builder Pattern)**

```java
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;
	
	public static class Builder {
		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수
		private int calories = 0;
		private int fat = 0;
		private int sodium = 0;
		private int carbohydrate = 0;

			public Builder (int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			} // 생성자에는 필수로 받아야 하는 값들 

			//Setter와 동일한 역할을 하는 메서드에는
			//부가적으로 세팅해야 하는 값들 
			public Builder calories(int val)
			{ calories = val; return this; }
			public Builder fat(int val)
			{ fat = val; return this; }
			public Builder sodium(int val)
			{ sodium = val; return this; }
			public Builder carbohydrate(int val)
			{ carbohydrate = val; return this; }

			// 최종적으로 빌드하는 메서드 제
			public NutritionFacts build() { return new NutritionFacts(this);}
	}

	private NutritionFacts(Builder builder) {
		servingSize = builder.servingSize;
		servings = builder.servings;
		calories = builder.calories;
		fat = builder.fat;
	  sodium = builder.sodium;
		carbohydrate = builder.carbohydrate;
	}

	// 플루언트 API
	public static void main(String[] args) {
		NutritionFacts cocaCola = new Builder(240, 0)
							.calories(100).sodium(35).carbohydrate(27).build();
	}
}
```

`NutritionFacts` 클래스는 불변이며, 모든 매개변수의 기본값들을 한곳에 모아뒀다. 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 이런 방식을 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API 혹은 메서드 연쇄라 한다. 

이 방법의 장점은 인스턴스를 만들 때 필수 속성에 해당하는 값들을 설정하는 것을 강제한다는 점이다. 따라서 자바 빈즈보다 훨씬 안전하게 만들 수 있다. 

> Optional한 값은 (선택 매개변수) 설정을 해도 되고 안 해도 된다.
> 

최종적으로는 `build()`만 호출하면 된다. `build()`를 호출하지 않은 상태에서는 객체를 만들 수 없다. `build()`를 호출하지 않으면 type이 `NutritionFacts` 가 아닌 `Builder` 이기 때문이다. 

정리하자면, 반드시 `build()` 를 호출한 다음에만 사용할 수 있게 되어서 생성자에 매개변수도 줄어들어 필수적인 매개변수만 남고, Optional한 값들은 부가적으로 사용할 수 있고, 객체도 안전하게 사용할 수 있다. 빌더 패턴은 점층적 생성자 패턴과 자바빈즈 패턴의 장점만 취했다. 

> 그렇다고 모든 경우에 빌더가 적절한 것은 아니다. 빌더는 오히려 코드를 이해하기 어렵게 만든다. 
빌더 패턴은 필수적인 필드가 있고, 필수적이지 않은 필드들이 있어서 생성자의 매개변수가 너무 많이 늘어난다거나, 클래스를 불변하게 만들고 싶을 때 사용하면 좋다.
> 

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다. 각 계층의 클래스에 관련 빌더를 멤버로 정의한다. 추상 클래스는 추상 빌더를 , 구체 클래스는 구체 빌더를 갖게 한다. 

```java
public abstract class Pizza {
	public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
	final Set<Topping> toppings;
	
	abstract static class Builder<T extedns Builder<T>> {
			EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
			public T addTopping(Topping topping) {
					toppings.add(Objects.requireNonNull(topping));
					return self();
			}
			abstract Pizza build();

			// 하위 클래스는 이 메서드를 재정의(overriding) 하여
			// "this"를 반환하도록 해야 한다.
			protected abstract T self();
		}
		
		Pizza(Builder<?> builder) {
			toppings = builder.toppings.clone(); 
		}
}
```

```java
public class NyPizza extends Pizza {
		public enum Size { SMALL, MEDIUM, LARGE }
		private final Size size;
		
		// 뒤에 Builder와 앞에 Builder가 같음 (재귀)
		public static class Builder extends Pizza.Builder<Builder> {
				private final Size size;

				public Builder (Size size) {
							this.size = Objects.requireNonNull(size);
				}

				@Override public NyPizza build() {
						return new NyPizza(this);
				}

				@Override protected Builder self() { return this; }
				}
		private NyPizza(Builder builder) {
			super(builder);
			size = builder.size;
		}
}
```

```java
public class Calzone extends Pizza {
		private final boolean sauceInside;
		
		public static class Builder extends Pizza.Builder<Builder> {
				private boolean sauceInside = false; // 기본값

				public Builder sauceInside() {
							sauceInside = true;
							return this;
				}

				@Override public Calzone build() {
						return new Calzone(this);
				}

				@Override protected Builder self() { return this; }
				}
		private Calzone(Builder builder) {
			super(builder);
			size = builder.sauceInside;
		}
}
```

Pizza 추상 클래스에선 추상 빌더를 만들면서 추상 빌더가 받을 수 있는 제네릭 타입에는 Builder 자신의 하위 클래스 타입을 받도록, 즉 재귀적인 타입 제한을 사용했다. 여기에 추상 메서드인 self를 더해 하위 클래스에는 형변환하지 않고도 메서드 연쇄를 지원할 수 있다. 

각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다. NyPizza.Builder는 NyPizza를 반환하고, Calzone.Builder는 Calzone를 반환한다는 뜻이다. 

하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능은 공변환 타이핑이라 한다. 이 기능을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다.

생성자로는 누릴 수 없는 사소한 이점으로, 빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있다. 각각을 적절한 메서드로 나눠 선언하거나 메서드를 여러 번 호출하도록 하고 각 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다. Pizza 클래스의 addToping 메서드가 이렇게 구현한 예다. 

빌더 패턴은 상당히 유연하다. 빌더 하나로 여러 객체를 순회하면서 만들 수 있고, 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다. 객체마다 부여되는 일련번호와 같은 특정 빌드는 빌더가 알아서 채우도록 할 수 있다.

빌더 패턴에 장점만 있는 것은 아니다. 객체를 만드려면, 그에 앞서 빌더부터 만들어야 한다. 빌더 생성 비용이 큰 건 아니나, 성능에 민감한 경우에는 문제가 될 수 있다. 또한 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다. 하지만 API는 시간이 지날수록 매개변수가 많아지는 경향이 있음을 명시하자.