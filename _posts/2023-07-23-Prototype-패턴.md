---
title: Prototype 패턴(프로토타입 패턴)
author: lukey0515
date: 2023-07-22 20:00:00 +0800
categories: [Design Pattern]
tags: [Design Pattern]
render_with_liquid: false
---

# 프로토타입 패턴이 필요한 경우

> 프로토타입 패턴은 클래스를 통해 인스턴스를 만드는 것이 아니라, 인스턴스로부터 `복사`를 통해 인스턴스를 만드는 패턴이다. 해당 패턴이 필요한 경우는 다음과 같다.

## 1. 종류가 너무 많아 클래스로 정리할 수가 없는 경우

클래스로 정리하려던 원형이 너무 다양하여 각각의 클래스를 만들면 오히려 클래스 수를 증가시켜 관리의 어려움만 생기는 경우를 의미한다.

## 2. 클래스로부터 인스턴스 생성이 어려운 경우

동일한 인스턴스를 만들고 싶은데 해당 인스턴스가 복잡한 과정을 거쳐 만들어지는 경우를 의미한다. 대표적인 예로 사용자가 마우스를 조작하여 그린 도형을 만드려고 하는 경우가 있다.

## 3. 프레임워크와 생성하는 인스턴스를 분리하고 싶은 경우

인스턴스를 복제하는 부분을 다른 패키지로 분리하여 사용할 수 있다. 이후, Client 클래스의 create 메서드에서 클래스 이름 대신 문자열을 통해 인스턴스를 생성함으로써 클래스 이름의 속박으로부터 프레임워크를 분리하였다.(?)

# Prototype 패턴 다이어그램과 실습 다이어그램

프로토타입 패턴은 Prototype, ConcretePrototype, Client를 필요로 한다.

![프로토타입 패턴 다이어그램과 실습 다이어그램](/assets/img/image/prototype_패턴.png)

## Prototype 인터페이스

```
public interface Product extends Cloneable{
    public abstract void use(String s);
    public abstract Product createCopy();
}
```

Prototype 인터페이스는 인스터스를 복사하여 새로운 인스턴스를 만드는 데에 사용될 메서드를 결정한다. 해당 예제에서는 createCopy라는 메서드가 사용되었다.

## ConcretePrototype 클래스

```
public class MessageBox implements Product{
    private char decochar;

    public MessageBox(char decochar) {
        this.decochar = decochar;
    }
    @Override
    public void use(String s) {
        int decolen = 1 + s.length() + 1;
        for (int i = 0; i < decolen; i++) {
            System.out.println(decochar);
        }
        System.out.println();
        System.out.println(decochar + s + decochar);
        for (int i = 0; i < decolen; i++) {
            System.out.println(decochar);
        }
        System.out.println();
    }

    @Override
    public Product createCopy() {
        Product p = null;
        try {
            p = (Product) clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return p;
    }
}

```

ConcretePrototype 클래스에서는 Prototype 클래스에서 결정한 메서드를 실제로 구현하는 역할을 한다.

## Client 클래스

```
public class Manager {
    private Map<String, Product> showcase = new HashMap<>();

    public void register(String name, Product prototype) {
        showcase.put(name, prototype);
    }

    public Product create(String prototypeName) {
        Product p = showcase.get(prototypeName);
        return p.createCopy();
    }
}

```

Client 클래스는 인스턴스를 복사하는 메서드를 이용해 새로운 인스턴스를 만든다. 즉 실제 인스턴스의 복제가 일어나는 클래스이다.

## Prototype 메서드의 사용

```
public class Main {
    public static void main(String[] args) {
        Manager manager = new Manager();
        MessageBox mbox = new MessageBox('*');
        MessageBox sbox = new MessageBox('/');

        manager.register("warning box", mbox);
        manager.register("slash box", sbox);

        Product p2 = manager.create("warning box");
        p2.use("Hello, world.");
        Product p3 = manager.create("slash box");
        p3.use("Hello, world.");
    }
}

```

# 핵심

## 1. 독립적인 관계

Prototype 인터페이스나 Client 클래스에서 ConcretePrototype 클래스의 이름이 등장하지 않는다. Prototype 인터페이스가 Client 클래스와 CocnretePrototype 클래스 사이의 연결다리가 되어 Client 클래스에서는 Prototype 인터페이스만으로 코드를 구성할 수 있게 되었다. 이런 구조를 만듦으로써 Prototype이나 Client는 ConcretePrototype 클래스와 독립적으로 수정할 수 있는 장점이 생긴다.

## 2. Cloneable

clone 메서드를 실행하기 위해서는 `Cloneable` 인터페이스를 구현해야만 한다. 만약 Cloneable 인터페이스를 구현하지 않고 clone 메서드를 호출하면 CloneNot-SupportedException이라는 예외가 호출된다.
이 사실만 보면 Cloneable 인터페이스 내부에 clone 메서드가 선언되어 있을 것 같지만, Cloneable 인터페이스 내부에는 어떤 메서드도 선언되어 있지 않다. 해당 인터페이스는 마커 인터페이스로 clone 메서드로 복제를 허용한다는 표시를 위해 사용된다.

❓Cloneable 인터페이스 내부에 어떤 메서드도 없다면 왜 clone 메서드를 실행할 때 cloneable 인터페이스를 구현하지 않으면 예외가 발생하지?

## 3. clone 메서드의 어려움

- 얕은 복사

  clone 메서드에서는 얕은 복사가 일어난다. 얕은 복사란 필드 내용을 그대로 복사하기 때문에 필드가 가리키는 인스턴스의 내용까지는 고려하지 않는 것이다. 따라서 만약 하고자 하는 구현이 얕은 복사로는 부족하다면 clone 메서드를 오버라이드하여 필요로 하는 복사를 재정의할 수 있다.

- protected

  clone 메서드는 protected로 지정되어 있어 상속 관계가 없는 클래스의 clone 메서드를 호출하기 어렵다. 따라서 여러 필드를 가진 클래스의 인스턴스를 복제하기 위해 clone 메서드를 이용하는 것이 부적합할 수 있다. 실제로 prototype 패턴을 사용하려할 때, clone 메서드에 의존하기 보단 복사 생성자나 복사 팩토리를 사용하는 편이 좋다. (?)

### 복사 생성자

```
public interface Product {
    public abstract void use(String s);
    public abstract Product createCopy();
}
```

```
public class MessageBox implements Product{
    private char decochar;

    public MessageBox(char decochar) {
        this.decochar = decochar;
    }

    public MessageBox(MessageBox prototype) {
        this.decochar = prototype.decochar;
    }

    @Override
    public void use(String s) {
        int decolen = 1 + s.length() + 1;
        for (int i = 0; i < decolen; i++) {
            System.out.println(decochar);
        }
        System.out.println();
        System.out.println(decochar + s + decochar);
        for (int i = 0; i < decolen; i++) {
            System.out.println(decochar);
        }
        System.out.println();
    }

    @Override
    public Product createCopy() {
        return new MessageBox(this);
    }
}

```

> 이전과 달리 Cloneable을 확장하지 않고, clone 대신 복사 생성자를 통해 인스턴스를 만든다.

❓그렇다면 굳이 clone 메서드롤 protected로 선언해놓은 이유는?

❓복사 생성자를 통해 만드는 방식이 결국엔 생성자를 통해 만드는 방식과 크게 다를 게 없지 않나?
