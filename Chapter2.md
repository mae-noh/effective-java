
---

## 2장. 객체 생성과 파괴.

- 객체를 만들어야 할 때 vs 객체를 만들어야하지 말아야 할 때
- 올바른 객체 생성 방법
- 불필요한 생성을 피하는 방법
- 제때 파괴됨을 보장하고, 파괴 전 수행해야 할 정리 작업을 관리

> item1. 생성자 대신 `정적 팩터리 메서드`를 고려하라.

- 정적 팩터리 메서드?
public static, 클래스의 인스턴스를 반환하는 단순한 정적 메서드
- 클래스가 인스턴스를 얻는 방법 2가지
1. public 생성자
2. 정적 팩터리 메서드
- 장점
1. 이름을 통해 반환될 객체의 특성을 쉽게 묘사 할 수 있음.
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 됨. (인스턴스 재활용)
3.
4. 입력 매개 변수에 따라 다른 클래스 객체를 반환 가능.
5.
- 단점
1. 상속을 하려면 public, protected 생성자가 필요, 정적 팩터리 메서드만 만들며 하위 클래스를 만들 수 없음.
2.
- 정적 팩터리 메서드 명명 방식

> item2. 생성자에 매개변수가 많다면 `빌더`를 고려하라.

- 빌더?
필수 매개변수만으로 생성자를 호출하여 빌더 객체를 얻은 후, 원하는 선택 매개변수들을 설정
- 사용 코드

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
		.calories(100).sodium(35).carbohydrate(27).build();
```

- 빌더 패턴

```java
public class NutritionFacts{
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;

	public static class Builder{
		// 필수 매개변수
		private final int servingSize;
		private final int servings;

		// 선택 매개변수, 기본 값으로 초기화
		private final int calories = 0;
		private final int fat = 0;
		private final int sodium = 0;
		private final int carbohydrate = 0;

		public Builder(int servingSize, int servings){
			this.servingSize = servingSize;
			this.servings = servings;
		}

		public Builder calories(int val){
			calories = val;
			return this;
		}

		public Builder fat(int val){
			fat = val;
			return this;
		}

		...
	}
}
```

> item3. `private 생성자`나 `열거타입`으로 싱글턴임을 보증하라.

- 싱글턴이란?
인스턴스를 오직 하나만 생성할 수 있는 클래스

- 싱글턴을 만드는 3가지 방법

    1. public static final
    2. 정적 팩터리 메서드 방식
    3. 열거 타입 방식

- public static final

```java
public class Elvis{
	**public static final Elvis INSTANCE = Elvis();**
	private Elvis() { ... } // Elvis.INSTANCE 초기화 시 딱 한번 호출

	public void leaveTheBuilding() { ... }
}
```

- 정적 팩터리 메서드 방식

```java
public class Elvis{
	public static final Elvis INSTANCE = Elvis();
	private Elvis() { ... }
	**public static Elvise getInstance() { return INSTANCE; }**

	public void leaveTheBuilding() { ... }
}
```

- 열거 타입 방식, Enum

```java
public enum Elvis{
	INSTANCE;

	public void leaveTheBuilding() { ... }
}
```

> item4. `인스턴스화`를 막으려거든 `private 생성자`를 사용하라.

- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아님, 
컴파일러가 자동으로 기본 생성자를 만들어 주기 때문에 private 생성자를 추가하면 인스턴스화를 막을 수 있음.

> item5. 자원을 직접 명시하지 말고 `의존 객체 주입`을 사용하라.

- 클래스가 내부적으로 하나 이상의 자원에 의존하면 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음.
- 인스턴스 생성 시 생성자에 필요한 자원을 넘겨줌

```java
public class SpellChecker{
	private final Lexicon dictionary;

	public SpellChecker(Lexicon dictionary){
		this.dictionary = Objects.requireNonNull(dictionar);
	}

	public boolean isValid(String word){ ... }
	public List<String> suggestions(String typo) { ... }
}
```

> item6. 불필요한 객체 생성을 피하라.

- 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라.
- 정적 팩터리 메서드를 사용하여 불필요한 객체 생성을 피할 수 있다.
Boolean(String) 대신 Boolean.valueOf(String) 팩터리 메서드를 사용하는 것이 좋음.
- 박싱된 타입보다 기본 타입을 사용하자.

> item7. 다 쓴 객체 참조를 해제하라.

- 자바는 가비지 컬렉터를 갖춘 언어로 메모리 관리에 신경쓰지 않아도 된다? X
- 메모리 누수

> item8. finalizer와 cleaner 사용을 피하라.

- 자바의 2가지 객체 소멸자
1. finalizer
2. cleaner
- 예측 불가능하고 상황에 따라 위험할 수 있어 기본적으로 사용하지 말아야 한다.

> item9. try-finally 보다는 `try-with-resources`를 사용하라.

- 자바 라이브러리에서 close 메서드로 직접 닫아줘야 하는 자원
InputStream, OutputStream, java.sql.Connection
- try-with-resources를 사용하면 코드가 더 짧고 분명해진다.
- try-finally

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try{
		OutputStream out = new FileOutputStream(dst);
		try{
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally{
		in.close();
	}
}
```

- try-with-resources

```java
static void copy(String src, String dst) throws IOException{
	try(InputStream in = new FileInputStream(src);
			OutputStream out = new FileOutputStream(dst)) {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while((n = in.read(buf)) >= 0)
				out.write(buf, 0, n);
	}
}
```