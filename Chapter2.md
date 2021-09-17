
## 2장. 객체 생성과 파괴

- 객체를 만들어야 할 때 vs 객체를 만들어야하지 말아야 할 때
- 올바른 객체 생성 방법
- 불필요한 생성을 피하는 방법
- 제때 파괴 됨을 보장하고, 파괴 전 수행해야 할 정리 작업을 관리

### item1. 생성자 대신 `정적 팩터리 메서드`를 고려하라.

- 정적 팩터리 메서드?
`public static`, 클래스의 인스턴스를 반환하는 단순한 정적 메서드

- 클래스가 인스턴스를 얻는 방법 2가지

```java
1. public 생성자
2. 정적 팩터리 메서드(또는 public 생성자와 함께)
```

- 장점

```java
1. 이름을 통해 반환될 객체의 특성을 쉽게 묘사 할 수 있음.
2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 됨. (cf. 플라이웨이트 패턴)
3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력.
4. 입력 매개 변수에 따라 다른 클래스 객체를 반환 가능.
5. 정적 팩터리 메서드 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 됨. (cf. 퓨처 패턴)
```

- 단점

```java
1. 상속을 하려면 public, protected 생성자가 필요, 정적 팩터리 메서드만 만들며 하위 클래스를 만들 수 없음.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어려움.
```

### item2. 생성자에 매개변수가 많다면 `빌더`를 고려하라.

- 빌더?
필수 매개변수만으로 생성자를 호출하여 빌더 객체를 얻은 후, 세터 메서드로 원하는 선택 매개변수들을 설정

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

### item3. `private 생성자`나 `열거타입`으로 싱글턴임을 보증하라.

- 싱글턴이란?
인스턴스를 오직 하나만 생성할 수 있는 클래스

- 싱글턴을 만드는 3가지 방법

    ```java
    1. public static final
    2. 정적 팩터리 메서드 방식
    3. 열거 타입 방식
    ```

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

 

- 열거 타입 방식, Enum (권장)
리플렉션 공격, 복잡한 직렬화 상황에서 제 2의 인스턴스가 생기는 것을 완벽하게 막아줌.
원소가 하나뿐인 열거 타입에 가장 적합.

    ```java
    public enum Elvis{
    	INSTANCE;

    	public void leaveTheBuilding() { ... }
    }
    ```

### item4. 인스턴스화를 막으려거든 `private 생성자`를 사용하라.


- 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계한게 아님, 
컴파일러가 자동으로 기본 생성자를 만들어 주기 때문에 private 생성자를 추가하면 인스턴스화를 막을 수 있음.

### item5. 자원을 직접 명시하지 말고 `의존 객체 주입`을 사용하라.


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

### item6. 불필요한 객체 생성을 피하라.


기존 객체를 `재사용`해야 한다면 새로운 객체를 만들지 마라.

- 불변 객체 String의 재사용

    ```java
    String s = new String("apple"); // 새로 만듦
    String s = "apple"; // 기존 객체 생성 (권장)
    ```

- `정적 팩터리 메서드`를 사용하여 불필요한 객체 생성을 피할 수 있다.
Boolean(String) 대신 **Boolean.valueOf(String)** 팩터리 메서드를 사용하는 것이 좋음.

- 박싱된 타입보다 `기본 타입`을 사용하자.

    ```java
    private static long sum(){
    	Long sum = 0L;
    	for (long i=0; i<=Integer.MAX_VALUE; i++)
    		sum += i; // 불필요한 Long 인스턴스 생성.

    	return sum;
    }
    ```

### item7. 다 쓴 객체 참조를 해제하라.


가비지 컬렉션 언어에서 `메모리 누수`를 찾는 것은 아주 까다로워서 미리 예방하는 것이 중요하다.

**메모리 누수에 주의해야 하는 경우와 해결 방법**

- 자기 **메모리를** 직접 관리하는 클래스 (ex. Stack)

    ```java
    원소를 다 사용한 즉시 원소가 참조한 객체들을 null 처리
    ```

- 객체 참조를 **캐시**에 넣어두고 잊은 경우

    ```java
    외부에서 키를 참조하는 동안만 캐시가 필요한 상황이라면 WeakHashMap을 사용하여 캐시를 만듦
    다 쓴 엔트리는 즉시 자동으로 제거 가능
    ```

- 리스너, **콜백**을 등록하고 해지 않는 경우

    ```java
    콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해감
    WeakHashMap에 키로 저장
    ```

**메모리 누수 Example : Stack**


- Stack, pop (좋지 않은 예)

```java
public class Stack{
	private Object[] elements;
	...

	public Object pop(){
		if(size == 0)
			throw new EmptyStackException();
		return elements[--size]; // 발생
	}

}
```

- 이 코드에서는 스택이 커졌다가 줄어들 때, 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수 하지 않는다.
스택이 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다.

- 제대로 구현한 Stack, pop (좋은 예)

```java
public class Stack{
	...

	public Object pop(){
		if(size == 0)
			throw new EmptyStackException();
		elements[--size];
		**return elements[size] = null; // 다 쓴 참조 해제**
	}

}
```

### item8. finalizer와 cleaner 사용을 피하라.


자바의 2가지 객체 소멸자 : finalizer, cleaner
예측 불가능하고 상황에 따라 위험할 수 있어, 특정 상황을 제외하고는 기본적으로 사용하지 말아야 한다.

### item9. try-finally 보다는 `try-with-resources`를 사용하라.


try-with-resources를 사용하면 코드가 더 짧고 분명해진다.

**자원이 2개 이상인 경우, 코드의 복잡성 증가**


- try-finally (좋지 않은 예)

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

- try-with-resources (좋은 예)

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