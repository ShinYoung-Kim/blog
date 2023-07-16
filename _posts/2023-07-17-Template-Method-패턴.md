---
title: Template Method 패턴 (템플릿 메서드 패턴)
author: lukey0515
date: 2023-07-16 22:10:00 +0800
categories: [Design Pattern]
tags: [Design Pattern]
render_with_liquid: false
---

# 1. 정의

![[Pasted image 20230716155401.png]]

`템플릿`의 사전적 의미는 무언갈 찍어내는 틀을 의미한다.
`템플릿 메서드`는 이러한 템플릿 기능을 가진 패턴으로 상위 클래스 쪽에 템플릿이 될 메서드가 정의되어 있고, 하위 클래스에서 메서드를 구현하여 구체적인 처리 방식을 결정한다.

# 2. 템플릿 메서드 예시

![[Pasted image 20230716161700.png]]

템플릿 메서드 패턴에는 상위 클래스이자 일종의 '틀'이 되어줄 AbstractClass와 해당 클래스를 상속받아 추상 메서드를 구체적으로 구현해줄 ConcreteClass가 필요하다.

예시의 상황은 문자나 문자열을 print하는 프로그램의 구현에 템플릿 메서드 패턴을 사용하는 상황이다.
<< + 문자 5번 + >> 을 출력하는 방식과,
+--...--+ + | 문자열 | + +--...--+ 을 출력하는 방식을 만들어야 하는데,
두 방식 모두 원하는 문자/문자열의 출력이 있고, 해당 출력의 전후로 출력의 시작과 끝을 알리는 출력이 존재한다는 공통점이 있다.

```
public abstract class AbstractDisplay {
	public abstract void open(); // 출력의 시작
	public abstract void print(); // 문자/문자열 출력
	public abstract void close(); // 출력의 끝

	public final void display() { // 알고리즘의 정의
		open();
		for (int i = 0; i < 5; i++) {
			print();
		}
		close();
	}
}
```

이 예제에서 AbstractClass역할인 AbstractDisplay 클래스에서는 open은 출력의 시작, print는 해당 문자나 문자열의 출력, close는 출력의 끝을 나타내는 데에 쓰였다. 또한, 이 메서드들이 어떤 순서나 반복 등을 지니는지 일종의 알고리즘을 보여주는 display라는 함수가 정의되어 있다.

open, print, close 추상 메서드들은 이후 하위 클래스인 CharDisplay, StringDisplay에서 각자의 방식으로 구현된다.

```
public class CharDisplay extends AbstractDisplay {
	private char ch;

	public CharDisplay(char ch) {
		this.ch = ch;
	}

	@Override
	public void open() {
		Sytem.out.print("<<");
	}

	@Override
	public void print() {
		System.out.print(ch);
	}

	@Override
	public void close() {
		System.out.print(">>");
	}
}
```

이제 이 ConcreteClass들을 활용을 해보면 다음과 같이 활용해볼 수 있다.

```
public class Main {
	public static void main(String[] args) {
		AbstractDisplay d1 = new CharDisplay('H');
		AbstractDisplay d2 = new StringDisplay("Hello, world.");

		d1.display();
		d2.display();
	}
}
```

여기서 중요한 점은 CharDisplay의 인스턴스인 d1과 StringDisplay의 인스턴스인 d2 모두 AbstractDisplay형 변수에 대입하여 display 메서드를 호출하고 있다는 것이다. 이는 리스코프 치환 원칙과 깊은 연관이 있다.

### 리스코프 치환 원칙(LSP)

리스코프 치환 원칙은 1988년 올바른 상속 관계를 정의하기 위해 발표한 원칙으로 서브 타입은 언제나 기반 타입으로 교체될 수 있어야 한다는 원칙이다.
이는 부모 클래스와 자식 클래스 사이 일관성을 위해 부모 클래스의 인스턴스를 사용하는 자리에 자식 클래스의 인스턴스를 대신 사용했을 때 코드가 원래 의도대로 동작해야 함을 의미한다.

위 코드는 리스코프 치환 원칙을 준수하고 있기 때문에 다음과 같은 코드가 가능하다.

```
public class Main {
	public static void main(String[] args) {
		AbstractDisplay d1 = new CharDisplay('H');
		AbstractDisplay d2 = new StringDisplay("Hello, world.");

		d1.display();
		d1 = new StringDisplay("Hi");
		d1.display();
	}
}
```

### printLine 메서드는 LSP에 위배되는 것이 아닌가?

private로 선언된 메서드 -> 외부 호출될 가능성이 없어서 LSP에 위배될 상황이 막힌 상황이다.

# 3. 자바 언어

> display 메서드에 사용된 `final` 키워드는 무엇인가?
> final 키워드는 불변성을 위해 JAVA에서 사용되는 키워드로, class, method, 변수에 붙을 수 있다.
>
> - 변수에 붙는다면, 초기화 후 변경될 수 없다.
> - 클래스에 붙는다면, 다른 클래스가 상속할 수 없는 클래스가 된다.
> - 메서드에 붙는다면, Override가 불가능하도록 한다.
>
> display 메서드에서 final 키워드를 사용한 이유는 AbstractClass를 상속받는 하위 클래스들이 display 메서드를 Override하여 상위 클래스의 의도와는 다른 방식으로 사용하는 것을 막기 위함이다.

# 4. hook 메서드

템플릿 메서드 패턴을 구현하는 AbstractClass에는 하위 클래스들이 공통적으로 가질 메서드들을 모아 일종의 알고리즘을 만든다. 예제에서는 display라는 메서드를 통해 시작할 때의 출력, 문자나 문자열 출력의 반복, 종료할 때의 출력을 정해두었다. 해당 예제는 간단한 예제라 두 ConcreteClass의 알고리즘이 완벽히 일치했지만, 만약 두 ConcreteClass가 조금 다른 알고리즘을 가진다면 어떻게 해야 할까?
이를 위해 hook 메서드가 존재한다. 이 메서드는 부모 템플릿 메서드(예제에선 display 메서드)의 영향이나 순서를 제어하기 위해 사용된다. hook 메서드를 통해 다음과 같이 사용할 수 있다.

```
public final void display() {
	open();
	for (int i = 0; i < 5; i++) {
		print();
	}
	close();

	if (hook()) {
		//원하는 동작 추가
	}
}
```

# 5. 템플릿 메서드 사용하는 이유?

- 확장의 범위를 좁히고 수정에 닫혀있는 패턴
- 동시에 확장을 못하게 강제 / 확장 하도록 강제
- 책임의 분리와 구현의 분리 = 관심사의 분리
- Abstract class는 알고리즘에 집중, Concrete class는 기능에 집중
- abstract class는 로직이 동작할 수 있는 범위만 가지고, 하위 클래스는 하나에만 집중해서 만들면 된다.
- 범위에 대해 분명한 상황일 때 관심사를 분리하기 위해서 템플릿 메서드를 사용한다.
- 제한되고 견고한 알고리즘이 존재하는 경우 템플릿 메서드를 사용한다.
- 템플릿 메서드를 확장 / 변경하겠다는 의도가 담겨있다면 다른 패턴으로 변경해야 한다. (ex. 전략 패턴)
- 공정에 대한 상세 내용을 하위 클래스에게 넘김
