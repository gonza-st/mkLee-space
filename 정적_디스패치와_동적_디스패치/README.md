## 문제 1.

```java
public class StaticDispatch {
		static abstract class Human {
		}

		static class Man extends Human {
		}

		static class Woman extends Human {
		}

		// 오버로딩된 메서드 (1)
		public void sayHello(Human guy) {
				System.out.println("Hello, guy!");
		}

		// 오버로딩된 메서드 (2)
		public void sayHello(Man guy) {
				System.out.println("Hello, gentleman!");
		}

		// 오버로딩된 메서드 (3)
		public void sayHello(Woman guy) {
				System.out.println("Hello, lady!");
		}

		public static void main(String[] args) {
				Human man = new Man();
				Human woman = new Woman();
				StaticDispatch sr = new StaticDispatch();

				sr.sayHello(man);
				sr.sayHello(woman);
		}
}
```

</br>

## 문제 2.

```java
public class DynamicDispatch {
		static abstract class Human {
				protected abstract void sayHello();
		}

		static class Man extends Human {
				@Override
				protected void sayHello() {
						System.out.println("Man said hello");
				}
		}

		static class Woman extends Human {
				@Override
				protected void sayHello() {
						System.out.println("Woman said hello");
				}
		}

		public static void main(String[] args) {
				Human man = new Man();
				Human woman = new Woman();

				man.sayHello();
				woman.sayHello();

				man = new Woman();
				man.sayHello();
		}
}
```

</br>

## 문제 3.

```java
import java.io.Serializable;

public class Overload {
		public static void sayHello(Object arg) {
				System.out.println("Hello Object");
		}

		public static void sayHello(int arg) {
				System.out.println("Hello int");
		}

		public static void sayHello(long arg) {
				System.out.println("Hello long");
		}

		public static void sayHello(Character arg) {
				System.out.println("Hello Character");
		}

		public static void sayHello(char arg) {
				System.out.println("Hello char");
		}

		public static void sayHello(char... arg) {
				System.out.println("Hello char ...");
		}

		public static void sayHello(Serializable arg) {
				System.out.println("Hello Serializable");
		}

		public static void main(String[] args) {
				sayHello('a');
		}
}
```

- Q. `sayHello(char arg)` 메서드 주석처리시 어떻게 되는가 ?
- Q.`sayHello(int arg)` 메서드 주석처리시 어떻게 되는가 ?
- Q.`sayHello(long arg)` 메서드 주석처리시 어떻게 되는가 ?
- Q. `sayHello(Character arg)` 메서드 주석처리시 어떻게 되는가 ?
- Q. 만약 `sayHello(Comparable<Character> arg)` 메서드가 같이 있다면 ?
- Q. `sayHello(Serializable arg)` 메서드 주석처리시 어떻게 되는가 ?
- Q. `sayHello(Object arg)` 메서드 주석처리시 어떻게 되는가 ?

</br>

## 문제 4.

```java
public class FieldHasNoPolymorphic {
		static class Father {
				public int money = 1;

				public Father() {
						money = 2;
						showMeTheMoney();
				}

				public void showMeTheMoney() {
						System.out.println("I am a Father, I have $" + money);
				}
		}

		static class Son extends Father {
				public int money = 3;

				public Son() {
						money = 4;
						showMeTheMoney();
				}

				public void showMeTheMoney() {
						System.out.println("I am a Son, I have $" + money);
				}
		}

		public static void main(String[] args) {
				Father guy = new Son();
				System.out.println("This guy has $" + guy.money);
		}
}
```

</br>

# 문제 해석 전 사전 지식

</br>
</br>

## 자바 소스 코드

- 자바 소스 코드는 `javac` 라는 컴파일러를 거치고 나면 자바 바이트 코드가 된다
- 바이트 코드 (`*.class`) → 이후 JVM 위에서 실행됨
- 자바 바이트 코드는 JVM 이 설치된 곳이 어디든 JVM 만 설치되어 있으면 실행됨 → 플랫폼 독립적

</br>

## 자바 바이트 코드

클래스 파일 구조

```
ClassFile {
		u4                 magic;
		u2                 minor_version;
		u2                 major_version;
		u2                 constant_pool_count;
		...
		u2                 fields_count;
		field_info         fields[fields_count];
		u2                 methods_count;
		method_info        methods[methods_count];
		u2                 attributes_count;
		attribute_info     attributes[attributes_count];
		...
}
```

- 클래스 파일의 구조는 부호 없는 숫자 와 테이블로 구성되어 있다
  - 부호없는 숫자 : `u1` (1 바이트) , `u2` (2 바이트) , `u4` (4 바이트) , `u8` (8 바이트)
  - 테이블 : 클래스 파일 구조 형태 → 테이블 이름은 관례적으로 `_info` 로 끝남

 </br>
 
 클래스 파일을 십육진수 편집기 (HxD) 로 열어본 모습 
 
![](https://velog.velcdn.com/images/dlaudrb09-/post/6140b25c-97e8-4b06-9c17-21cf66e12583/image.png)

- 자바 코드를 `javac` 로 컴파일할 때는 C, C++ 와 달리 링크 단계가 없다
  - C, C++ 링크 단계 : 컴파일러가 생성한 obj (목적) 파일은 라이브러리 (Library) 와 연결하여 실행 파일(exe) 로 생성하는데 이 과정을 링크라고 함
  - 동적링크 즉, 런타임에서의 링크단계는 존재함
- 자바에서 링크는 JVM 이 클래스 파일을 로그할 때 동적으로 이루어진다
  - 필드와 메서드가 메모리에서 어떤 구조로 표현되는가에 관한 정보는 클래스 파일에 저장되지 않는다는 뜻.
  - 따라서 JVM 이 필드와 메서드의 참조를 런타임에 실제 심벌 참조 로 변환하지 않으면 각 항목의 실제 메모리 위치를 알 수 없다
    - 심벌 참조 : 몇가지 정해진 심벌로 참조 대상을 정함. 특정 리터럴 형태
    - 직접 참조 : 오프셋 혹은 대상의 위치를 가리키는 핸들
- 필드 테이블
  - 클래스 변수와 인스턴스 변수를 뜻함
  - 메서드안에 선언된 지역 변수는 필드가 아님 → 스택 프레임안에 로컬 변수 테이블에 있음
- 메서드 테이블
  - 메서드 메타 데이터 (정의) 정보들이 들어있음
  - 메서드 본문의 코드는 여기에 있지않고 메서드 속성 테이블 컬렉션의 Code 속성에 따로 저장된다

```
Code_attribute {
    u2            attribute_name_index;
    ...
    u4            code_length;
    u1.           code[code_length];  // 바이트 코드 명령어들 (명령어 각각이 1바이트임)
}
```

</br>

## 클래스 로더 준비

- 준비

  - 준비는 클래스 변수를 메모리에 할당하고 초기값을 설정하는 단계이다
    - 정적 변수
    - 클래스 변수가 메서드 영역에 존재한다 라는 개념은 논리적으로 그렇다는 의미이며 JDK 8 부터는 클래스 변수가 클래스 객체와 함께 자바 힙에 저장된다
  - 첫째. 인스턴스 변수가 아닌 클래스 변수만 할당된다. 인스턴스 변수는 객체가 인스턴스화할 때 객체와 함께 자바 힙에 할당된다.
  - 둘째. 준비 단계에서 클래스 변수에 할당하는 초기값은 해당 데이터 타입의 제로 값이다.
    - public static int value = 123; → 준비 단계를 마친 직후 value 변수에 할당되어 있는 초기값은 123 이 아닌 0 이다.
    - 준비 단계에서는 어떠한 자바 메서드도 아직 실행되지 않은 상태이기 때문이다 → 클래스 초기화 단계에서 실행
    - 123 을 할당하는 putstatic 명령어는 클래스 생성자인 <clinit>() 메서드에 포함됨
  - 셋째. 지역변수는 준비단계가 없다. 즉 아래 int a 가 컴파일되지 않는 것은 시스템 초기값 (0) 이 할당되지 않는 것이다

  ```java
  public static void main(String[] args) {
  	int a;  // 컴파일 에러 발생
  	System.out.println(a);
  }
  ```

</br>

## 클래스 로더 초기화

- 초기화
  - 클래스 로더의 마지막 단계.
  - 초기화 단계에서 초기화 되지 않은 것들은 모두 초기화를 거친다
  - new 키워드로 인스턴스 생성
  - 클래스를 초기화할 때 상위 클래스가 초기화되어 있지 않다면 상위 클래스 초기화
  - 총 6가지의 경우가 있음 (초기화를 능동적으로 실행하는 6가지 시나리오 → 능동참조)
  - 앞서 준비 단계에서 모든 변수에 시스템이 정의한 초기값인 0을 할당했다, 초기화 단계에서는 개발자가 직접 기술한대로 초기화를 한다 → `<clinit>()` 메서드를 실행하는 단계
    - 부모 클래스의 생서자를 명시적으로 호출하지 않아도 자바 가상 머신은 하위 클래스의 <clinit>() 가 실행되기 전에 부모 클래스의 <clinit>() 부터 실행한다
    - 그러므로 첫번째 <clinit>() 는 바로 java.lang.Object 의 <clinit>() 이다
- 로딩

  - JVM 은 이름을 보고 해당 클래스를 정의하는 바이너리 바이트 스트림을 가져옴
    - 이 규칙들은 JVM 명세에서 구체적으로 어떻게 하라는 구체적인 룰이 명시되어 있지 않음
    - 완전한 이름 (= 패키지 명을 포함한 클래스의 전체 이름) (Fully Qualified Name = FQN)
  - 바이트 스트림으로 표현된 정적인 저장 구조를 메서드 영역에서 사용하는 런타임 데이터 구조로 변환
  - 로딩 대상 클래스를 표현하는 java.lang.Class 객체를 힙 메모리에 생성한다
  - 이 Class 객체는 애플리케이션이 메서드 영역에 저장된 타입 데이터를 활용할 수 있게 하는 통로가 된다

  </br>

## 동적 링크

- 런타임 상수 풀
  - 런타임 상수 풀은 메서드 영역의 일부이다
  - 상수 풀 테이블에는 클래스 버전, 필드, 메서드, 인터페이스 등 클래스 파일에 포함된 설명 정보에 더해 컴파일 타임에 생성된 다양한 리터럴과 심벌참조가 저장된다
  - JVM 이 클래스를 로드할 때 이러한 정보를 메서드 영역의 런타임 상수 풀에 저장한다
- 동적 링크

  - 메서드에서 이용하는 외부 객체를 가리키는 참조는 런타임 상수 풀에 담겨 있으며 각 메서드의 스택 프레임에서 런타임 상수 풀 내의 원소를 참조하는 식으로 구성된다
  - 이 참조가 동적 링크를 가능하게 하는 매개다

  </br>

## 메서드 호출

- 메서드 호출은 메서드 본문 코드를 실행하는 일과 다르다
- 메서드 호출 단계에서 수행하는 유일한 일은 호출할 메서드의 버전을 선택하는 것이다
  - 상속 구조에서 같은 메서드가 2개 존재할때 이를 총 2개의 버전이 있다고 이야기한다 → 어디에 정의된 메서드를 실행해야하는지 “해석” 해야한다
- 링킹 단계가 존재하지 않기 때문에 클래스 파일에 저장된 모든 메서드 호출은 심벌 참조일 뿐 메서드 주소를 담은 직접 참조가 아니다
- 그러나 그 중 일부는 직접 참조로 변환하는데 즉, 컴파일러가 프로그램 코드를 컴파일하는 시점에 호출 대상이 정해진다

  - 이 처럼 호출 대상이 미리 특정되는 경우를 정적 해석이라고 한다
  - 주로 정적 메서드와 private 메서드, final 인스턴스 메서드다.
  - 정적 메서드는 특정 클래스에 고정되어 있고, private 메서드는 인스턴스 바깥에서는 접근할 수 없다
  - final 인스턴스 메서드는 오버라이딩이 불가능하므로 다른 버전이 만들어질 가능성이 없다
  - 따라서 두 유형의 메서드 모두 상속등을 통해 다른 버전을 만들 수 없으므로 클래스 로딩 단계에서 정적 해석을 한다 → 바이트 코드 (명령어가 다름) `invokestatic` , `invokespecial`
  - 정적 해석을 해야하는 메서드 외의 메서드들은 가상 메서드라고 한다

  </br>

## 디스패치

- 한편 메서드 호출의 또 다른 형태로 디스패치가 있다
- 정적일 수도 있고 동적일 수도 있고 단일 디스패치 일수도 있고 다중 디스패치 일 수도 있다
- 총 네가지 유형이 생겨난다
  - 정적 단일 디스패치
  - 정적 다중 디스패치
  - 동적 단일 디스패치
  - 동적 다중 디스패치

</br>

## 문제 1 해석

- `sayHello()` 메서드 중 JVM 이 Human 타입을 받는 버전을 선택하는 이유는 무엇일까 ?
- 문제의 이유를 알기전에 개념을 알고가자.

```java
Human man = new Man();
```

- 이 코드에서 Human 을 변수의 정적 타입 또는 겉보기 타입 이라고 한다
- Man 을 변수의 실제 타입 또는 런타임 타입이라고 한다

```java
// 실제 타입 변경
Human human = new Random() ? new Man() : new Woman(); // 랜덤으로 둘중 하나의 타입이 런타임에 들어갔다고 가정

// 정적 타입 변경
sr.sayHello((Man) human);
sr.sayHello((Woman) human);
```

- human 객체는 랜덤으로 실제 타입이 변경될 수 있으니 알수없다 → 슈뢰딩거의 고양이
- 실제 타입이 `Man` 인지 `Woman` 인지는 프로그램이 해당 코드 라인을 실행할때에 마침내 선택된다
- 반면 `human` 객체의 정적 타입은 `Human` 이며 사용중에는 변경가능하다
- 하지만.
- 이 변경은 컴파일타임에 알 수 있다
- 위 코드에서 `sayHello()` 메서드를 호출할 때 강제로 변환했기 때문에 변환 결과가 `Man` 인지 `Woman` 인지 컴파일 타임에 명확해진다
- 문제 1 코드

```java
Human man = new Man();      // 정적 타입 = Human, 실제 타입 = Man
Human woman = new Woman();  // 정적 타입 = Human, 실제 타입 = Woman

sr.sayHello(man);           // 메서드 버전 선택 필요
sr.sayHello(woman);         // 메서드 버전 선택 필요
```

- 보다시피 `sayHello()` 메서드를 두 번 호출한다.
- 이때 메서드 수신자인 `sr` 객체의 `sayHello()` 메서드를 호출시, **오버로딩된 메서드 중 어느 버전을 호출할지는 전적으로 매개변수의 수와 타입이 기준이다**
- 이 코드에서 `man` 과 `woman` 은 정적 타입이 같지만 실제 타입은 다르다.
- 하지만.
- JVM (정확히 말하자면 컴파일러) 은 호출할 `sayHello()` 를 선택할 때 매개변수의 실제 타입이 아닌 정적 타입을 참고한다.
- 정적 타입은 컴파일타임에 알려지기 때문에 `javac` 컴파일러는 매개변수의 정적 타입을 보고 어떤 오버로딩 버전을 호출할지 선택한다. 따라서 `sayHello(Human)` 이 호출 대상으로 선택된다
- 메서드 버전 선택에 정적 타입을 참고하는 모든 디스패치 작업을 정적 디스패치라고한다.
- 정적 디스패치의 가장 일반적인 응용 예가 메서드 오버로딩이다
- 정적 디스패치는 컴파일타임에 이루어지므로 지금 설명한 선택 작업은 JVM 에서는 이루어지지 않는다

</br>

## 문제 3 해석

- `javac` 컴파일러가 오버로딩된 메서드 중 적절한 버전을 선택할 수 있지만, 어느 하나를 꼭 집어내지 못하여 “비교적 더 적합한” 버전으로 선택하는 경우도 많다
- 모호함의 주된 원인은 바로 “리터럴” 이다.
- 리터럴에는 명시적인 정적 타입이 없으므로 언어와 문법 규칙을 바탕으로 이해하고 유추할 수 있을 뿐이다.

- 문제 3번의 1번째 답은 `“Hello char”` 이다
- 리터럴 `‘a’` 의 타입은 `char` 이므로 자연스럽게 매개변수 타입인 `char` 인 버전을 선택했다

- 문제 3번의 2번째 답은 `“Hello int”` 이다
- 자동 형 변환이 이루어진 것 이다.
  - 문제 `a` 는 숫자 97을 나타낼 수도 있다, 따라서 매개 변수 타입인 int 인 메서드도 적합하다.
- 문제 3번의 3번째 답은 `“Hello long”` 이다
- 이번에는 자동 형 변환이 두번 일어난다
  - `a` 를 정수 97 로 변환한 후 매개 변수 타입인 long 과 일치시키기 위해 long 타입 정수인 97L 로 다시 변환한다
  - `char` → `int` → `long` → `float` → `double` 순서로 이루어진다.
  - char 를 byte 나 short 로 변환하는 것은 안전하지 않기 때문에 후보에 들지 않는다
- 문제 3번의 4번째 답은 “Hello Character” 이다
- 오토박싱이 일어났다
  - 즉 `a` 의 래퍼 타입인 `java.lang.Character` 로 박싱하여 매개 변수 타입인 `Character` 인 메서드와 일치시켰다
- 문제 3번의 5번째 답은 `"Hello Serializable"` 이다
- 문자와 직렬화가 무슨 관련이 있을까 ?
- 원인은 `Character` 가 `java.lang.Serializable` 을 구현했기 때문이다
  - 오토박싱 이후에도 매개 변수를 찾지 못하여 대신에 래퍼 클래스가 구현한 인터페이스 타입을 받는 메서드를 선택했다
- 문제 3번의 6번째 답은 컴파일 오류이다
- `Character` 는 또 다른 인터페이스인 `java.lang.Comparable<Character>` 도 구현한다
- 그러나 이는 `Serializable` 을 받는 메서드와 우선순위가 동일하다
  - 그러므로 컴파일러는 어느 타입으로 변환할지 선택할 수 없기에 컴파일 오류가 발생하게 되는것이다
- 문제 3번의 7번째 답은 `"Hello Object"` 이다
  - 오토 박싱 이후 부모 클래스로 변환된 것 이다
- 문제 3번의 8번째 답은 `"Hello char ..."` 이다
  - 7개의 오버로딩된 메서드 중 가변 길이 매개 변수를 받는 메서드의 우선순위가 가장낮다

</br>

## 문제 2 해석

- 오버라이딩과 밀접하게 관련된 또 다른 주제인 동적 디스패치의 작동 과정을 살펴보자.
- 문제 2의 실행 결과는 다음과 같다

```java
Man said hello
Woman said hello
Woman said hello
```

- 자연스러운 결과이다
- 그러나 위와 같이 JVM 이 어떤 메서드 버전을 선택하는지를 살펴보자
- 이번 문제는 호출할 메서드의 버전을 정적 타입만으로 결정하기가 불가능하다.
- 변수 `man` 과 `woman` 의 정적 타입은 모두 `Human` 으로 똑같지만, `sayHello()` 를 호출하면 서로 다른 메서드를 호출하기 때문이다
- 바이트 코드에서 메서드 호출은 `invokevirtual` 이고 실제로 심벌 참조 (`Human.sayHello()`) 도 모두 동일하다
- 그러나 실제 실행된 메서드는 다르다
- `invokevirtual` 명령어는 실행의 첫 단계에서 런타임 수신 객체의 실제 타입을 해석한다는 점이 중요하다
- 이런 이유로 앞에 두 메서드는 상수 풀에 있는 메서드의 심벌 참조를 직접 참조로 변환하는데서 그치지 않고 메서드 수신 객체의 실제 타입을 보고 메서드 버전을 선택한다
- 런타임에 실제 타입을 보고 메서드 버전을 선택하는 이러한 디스패치 방식을 동적 디스패치라고 한다
- 다형성의 뿌리는 가상 메서드 호출 명령어인 `invokevirtual` 의 실행 로직에 있다

</br>

## 문제 4 해석

- 실행 결과는 다음과 같다

```java
I am a Son, I have $0
I am a Son, I have $4
This guy has $2
```

- “I am a Son” 문장이 두 번 출력되었다
- 첫 번째 줄은 부모 클래스인 `Father` 의 생성자에서 출력하고 두 번째 줄은 `Son` 클래스의 생성자에 출력한 결과이다
- 그런데 왜 둘 다 `“Son”` 이라는 결과가 나왔을까 ?
- 코드는 `Father` 타입의 `guy` 는 `Son` 의 객체로 초기화 된다. 따라서 `guy` 는 `Father` 타입을 참조하지만 실제 인스턴스는 `Son` 이며 이 과정에서 `Father` 와 `Son` 클래스의 생성자가 모두 호출된다
- `Son` 클래스가 생성될 때 암묵적으로 `Father` 의 생성자를 먼저 호출하는데, `Father` 의 생성자에서 `money = 2` 가 수행되고 `showMeTheMoney()` 가 호출된다
- 하지만 해당 메서드는 `Son` 자식클래스에서 오버라이딩 되었으므로 실제 실행되는 것은 `Son` 의 `showMeTheMoney` , 이는 가상 메서드 호출을 실행합니다
- 이때 부모 클래스의 `money` 필드는 `2` 로 초기화되었지만 `Son.showMeTheMoney()` 메서드는 자식 클래스의 `money` 필드를 이용한다
- 자식 클래스의 `money` 필드는 자식 클래스의 생성자가 실행될 때에 초기화되기 때문에 아직은 값이 `0`인 상태이다
  - 즉 `Son` 생성자가 실행되어야지만 값을 할당함
- 마지막은 정적 타입을 통해 부모 클래스인 `money` 로부터 값을 직접 가져왔기 때문에 `2` 를 출력한다
  - 필드 접근은 정적 타입에 의해 결정되며 필드는 오버라이드 되지 않는다.
  - 필드는 메서드와 다르게 다형성을 지원하지 않기 때문에 동적 디스패치는 오직 메서드에만 적용된다
    - 즉 필드에는 `invokevirtual` 이라는 명령어를 사용하지 않음
  - 가상 필드라는 개념은 존재하지 않음
  - 그러므로 객체의 정적 타입을 기준으로 접근한다.
  - 추가로 필드의 이름은 클래스의 메서드가 그 필드에 담겨있는 값에 접근할 수 있는 수단이다

</br>

## 단일 디스패치 와 다중 디스패치

- 메서드의 수신 객체 (호출된 메서드의 주인) 와 매개 변수를 합쳐 메서드 볼륨이라고 한다
- 디스패치의 기준이 되는 볼륨 수에 따라 디스패치는 단일 디스패치와 다중 디스패치로 나뉜다

```java
public class Dispatch {
		static class QQ {}
		static class _360 {}

		public static class Father {
				public void choice(QQ arg) {
						System.out.println("Father chose a qq");
				}

				public void choice(_360 arg) {
						System.out.println("Father chose a 360");
				}
		}

		public static class Son extends Father {
				public void choice(QQ arg) {
						System.out.println("Son chose a qq");
				}

				public void choice(_360 arg) {
						System.out.println("Son chose a 360");
				}
		}

		public static void main(String[] args) {
				Father father = new Father();
				Father son = new Son();

				father.choice(new _360());
				son.choice(new QQ());
		}
}
```

- 가장 먼저 집중할 부분은 컴파일 단계에서 컴파일러의 선택 과정 즉, 정적 디스패치 과정이다
- 이때 대상 메서드를 선택하는데는 두 가지를 고려한다
- 하나는 변수는 정적 타입이 `Father` 이냐 `Son` 이냐이고 다른 하나는 매개 변수 타입이 `QQ` 이나 `_360` 이냐이다
- 두 가지를 조합해 내린 결론이 두 개의 invokevirtual 명령어를 생성하는데 이용된다
  - `Father` 의 두 개의 메서드를 실행할 명령어를 생성하는데 사용된다.
- 두 명령어의 매개 변수는 각각 상수 풀에 있는 `Father.choice(_360)` 과 `Father.choice(QQ)` 메서드이다
  - 이 처럼 선택에 이용된 볼륨이 두 개라서 자바의 정적 디스패치는 다중 디스패치다
- 이어서 동적 디스패치, 실행단계의 메서드 선택을 살펴보자
- `son.choice(QQ)` 가 실행될 때 컴파일 타임에 이미 `choice(QQ)` 로 결정되었다
- 따라서 매개 변수 `QQ` 로 전달되는 인수의 실제 타입이 무엇인지는 상관이 없다
  - 이시점에서 매개 변수 `QQ` 로 전달되는 인수의 실제 타입이 무엇인지는 상관하지 않는다. 이처럼 정적 타입과 실제 타입은 메서드 선택에 관여하지 않는다
  - 그렇다면 JVM 선택에 영향을 주는 유일한 요소는 메서드 수신 객체의 실제 타입이 `Father` 이냐 `Son` 이냐 뿐.
  - 즉, 선택 기준 볼륨이 하나뿐이므로 자바의 동적 디스패치는 단일 디스패치이다

```java
출력.

Father chose a 360
Son chose a qq
```

- 위 내용을 좀 더 간단히 이야기하자면,
  - 결정하는 시점이 다르다는 것에 초점을 맞춰야한다
  - 정적 디스패치는 컴파일 시점에 결정을 하는 것 이고 동적 디스패치는 실행 시점에 결정을 하는 것이다
  - 위에서 이야기했듯 볼륨 즉 결정 (선택) 할 때 기준이 되는 요소는 두가지
    1. 메서드 매개변수 타입
    1. 메서드 수신 객체 타입 → 호출한 메서드의 주인 타입
  - 정적 디스패치 가 왜 다중 디스패치 인지는 컴파일 시점의 결정을 생각해야 한다
    - 정적 디스패치는 컴파일 시점에 위에서의 두 가지 기준을 모두 고려해야 한다
    - 이 때문에 다중 디스패치라고 하고
    - 실행시점에는 이미 메서드 호출이 명확해 졌기 때문에 선택하지 않는다
    - 동적 디스패치는 컴파일 시점에 두 가지 기준이 아닌 한 가지 기준 즉, 메서드 매개변수 타입만을 고려해야 한다
      - 이유는 수신 객체의 타입이 컴파일 시점에 알 수 없는 슈뢰딩거의 고양이 이니 말이다
    - 그렇게 동적 디스패치는 컴파일 시점에 한 가지 기준을 두고 선택을 한다 이후
    - 실행 시점에서 선택을 해야한다 이때는 메서드 수신 객체의 타입을 가지고 메서드 선택을 해야한다
    - 이를 단일 디스패치라고 한다

</br>

## JVM 의 동적 디스패치 구현

- 동적 디스패치는 매우 자주 일어난다
- 또한 동적 디스패치 중 메서드 버전 선택시에는 런타임에 수신 객체 타입의 메서드 메타데이터를 보고 적절한 대상 메서드를 찾는 작업이 이루어진다
- 그래서 실행 성능을 중요시하는 JVM 구현에는 일반적으로 타입 메타데이터를 그리 자주 검색하지 않는다
  - 타입 메타데이터는 메서드 정보외의 객체의 복잡한 정보가 담겨있음 (클래스 이름, 필드 목록, 상속관계 등)
  - 계층적으로 구성되어 있으며 여러 단계를 거쳐서 특정 메서드를 찾는 과정에 많은 검색 작업이 필요함
- 해당 타입에 대한 가상 메서드 테이블 (메서드 영역에 존재하는 vtable) 을 만들어 최적화하는 것이다
- 이처럼 일반적으로 메타데이터 조회 대신 가상 메서드 테이블 인덱스를 사용해 성능 향상을 꾀한다.
  ![](https://velog.velcdn.com/images/dlaudrb09-/post/d2111dc5-7d3c-473c-a70e-a8d2fc66f521/image.png)
- 가상 메서드 테이블에는 각 메서드의 실제 시작 주소가 담긴다
- 하위 클래스에서 메서드가 오버라이딩되지 않으면 하위 클래스 가상 메서드 테이블의 주소 항목은 부모 클래스에 있는 동일한 메서드의 주소 항목과 같다
- 즉, 둘 다 부모 클래스의 구현 시작점을 가리킨다
- 하위 클래스에서 메서드를 오버라이딩하면 하위 클래스 가상 메서드 테이블의 주소 항목은 하위 클래스의 구현 시작점을 가리키게 바뀐다
- 자바에서 메서드는 final 로 선언하지 않는 이상 기본적으로 가상 메서드이다

</br>
</br>

참고 )

- JDK 10 에서 추가된 `var` 라는 키워드는 오른쪽 표현식의 타입을 보고 컴파일 타임에 추론하기 때문에 정적 디스패치를 한다

</br>
</br>

답.

문제1

```
Hello, guy!
Hello, guy!
```

문제2

```
Man said hello
Woman said hello
Woman said hello
```

문제3

```
Hello char
```

문제3-1

```
Hello int
```

문제 3-2

```
Hello long
```

문제 3-3

```
Hello Character
```

문제 3-4

```
Hello Serializable
```

문제 3-5

```java
"The method sayHello(Object) is ambiguous for the type overload"

라는 메시지를 뿌리며 컴파일을 거부한다.
```

문제 3-6

```
Hello Object
```

문제 3-7

```
Hello char ...
```

문제 4

```
I am a Son, I have $0
I am a Son, I have $4
This guy has $2
```
