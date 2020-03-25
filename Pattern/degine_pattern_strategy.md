# Strategy Pattern

## 정의

> 

## 1차 요구사항.

> 이동, 기본공격을 할 수 있는 게임의 캐릭터를 만들어줘.!

이런 요구사항이 나오면 이동, 기본공격을 갖는 아래와 같은 인터페이스를 만들어서 재활용하는 방식으로 구현을 시작할 것 같다.  
기본적으로 걷는 move는 동일하니 상위 클래스의 메소드를 사용하면되고, 궁수와 마법사의 공격은 활과 마법으로 공격을 할 수 있게 Override하여 구현하면 된다. 전사 캐릭터가 나와도 동일하게 쉽게 추가할 수 있고 이동하는 기능의 중복된 코드를 제거할 수 있다.!

```java
// 슈퍼클래스 
public class Character {
    public void move() {
        System.out.println("걷는다");
    }

    public void attack() {
        System.out.println("주먹을 휘두른다.");
    }
}

// 궁수
public class Archer extends Character {
    @Override
    public void attack() {
        System.out.println("활을 쏜다.");
    }
}

// 마법사
public class Wizard extends Character{
    @Override
    public void attack() {
        System.out.println("지팡이를 휘두른다.");
    }
}
```

## 2차 요구사항

> 서비스하기 임팩트가 부족하니 파이어볼을 쓰게 만들어줘.!

추가 요구사항을 받았다. 이미 슈퍼클래스를 상속받아 서브클래스들이 사용하고 있기에 슈퍼클래스에 파이어볼을 사용하는 기능을 구현하면 모든 캐릭터에서 사용할 수 있으니 Character에 메소드하나 만들어 넣으면 끝이다. 

```java
public class Character {
    public void move() {
        System.out.println("걷는다");
    }

    public void attack() {
        System.out.println("주먹을 휘두른다.");
    }

    public void doMagic() {
        System.out.println("파이어볼!!");
    }
}
```

## 3차 요구사항

> 파이어볼은.. 마법사만 쓸 수 있어요.;; 궁수가 파이어볼을 어떻게 사용하나요. 궁수는 못 쓰게 해줘.!

그렇다. 궁수는 파이어볼을 못쓴다. 이러다 나중에 마법사가 불화살을 쏘는일이 발생할 수도 있는 구조가 되버렸다.. 슈퍼클래스의 메소드를 사용하니 이런 문제가 발생하니 궁수 클래스에서 doMagic() 매소드를 아무동작 안하도록 재구현 하면 될 것 같다.

```java
public class Archer extends Character {
    @Override
    public void attack() {
        System.out.println("활을 쏜다.");
    }

    @Override
    public void doMagic() {
        // nothing..
    }
}
```

## 여기서 doMagic메소드를 Character클래스에서 제외하고 interface를 사용하여 FireballInterface를 만들고 doMagic을 갖도록 만들면 어떨까? 

이렇게 될 경우 지금은 Wizard하나의 클래스에서 사용하지만 예를 들어 여러 마법사의 클래스들에서 FireballInterface를 상속받은 상태라고 한다면 doMagic의 기능 변경이 필요할 경우 수많은 클래스의 로직 수정이 필요하게 된다.

슈퍼클래스에 추가하자니 다른 클래스에서 공통으로 사용할 수 있게되고 인터페이스를 사용하자니 코드를 재사용하지 못하게 되는 그런 상황에 빠지게 된다.

여기서 **디자인 원칙** 하나를 알고 가자.  
애플리케이션에서 달라지는 부분을 찾아 내고, 달라지지 않는 부분으로부터 분리시킨다. 즉, 코드에 새로운 요구사항이 있을 때마다 바뀌는 부분이 있다면, 그 행동을 바뀌지 않는 다른 부분으로부터 골라내서 분리해야 한다는 것을 알 수 있다. 

> 바뀌는 부분을 따로 뽑아서 캡슐화 시킨다.  
> 그렇게 하면 나중에 바뀌지 않는 부분에는 영향을 미치지 않은 채로 그 부분만 고치거나 확장할 수 있다.

문제를 파악하고 설계를 할수 있는 것과 못하는 것으로 분리하여 재정의 해보자.  
궁수는 파이어볼을 못 쓴다. 마법사만 쓸 수 있다. 아래와 같이 집합군을 만들 수 있다. 

1. MagicInterface 인터페이스를 만들고 
2. ArcherMagic 클래스를 만들어 MagicInterface를 구현한다.
3. WizardMagic 클래스를 만들어 MagicInterface를 구현한다.

```java
public class Character {
    public void move() {
        System.out.println("걷는다");
    }

    public void attack() {
        System.out.println("주먹을 휘두른다.");
    }
}

public class Wizard extends Character{

    private MagicInterface magic = new WizardMagic();

    @Override
    public void attack() {
        System.out.println("지팡이를 휘두른다.");
    }

    public void doMagic() {
        magic.doMagic();
    }
}

public class Archer extends Character {

    private MagicInterface magic = new ArcherMagic();

    @Override
    public void attack() {
        System.out.println("활을 쏜다.");
    }

    public void doMagic() {
        magic.doMagic();
    }
}

// 1. 마법 인터페이스
public interface MagicInterface {
    void doMagic();
}

// 2. 마법사 마법 행동
public class WizardMagic implements MagicInterface {
    @Override
    public void doMagic() {
        System.out.println("파이어볼");
    }
}

// 3. 궁수 마법 행동
public class ArcherMagic implements MagicInterface {
    @Override
    public void doMagic() {
        // nothing..
    }
}
```

위 와 같이 마법의 행동을 Character클래스에서 분리시켜 캡슐화하였고, 코드 또한 재사용할 수 있는 구조로 변경 되었다. 추후에 궁수의 마법행동에 불화살쏘기 로 변경이 될 때에도 하나의 코드 수정으로 변경이 가능한 것이다. 또한, 지금은 2개의 직업이지만 궁수와 비슷한 직업 클래스에서도 ArcherMagic으로 행동을 하고 있던 부분을 다른 행동을 하는 클래스로 변경하기도 쉬워 진다. 

이처럼 다른 클래스에게 역할과 책임을 위임하게 되면 객체지향적 프로그래밍이 가능해 진다. 메시지를 통해 객체간의 협력이 되는 것이다.

## 이제 스트래치지 패턴에 대해서 알아보자.

3차 요구사항을 받아들이면서 리펙토링한 결과물에는 유동적이지 못한 부분이 있다. 예를 들어 궁수가 불화살 마법만 있다는 가정이 되버린다. 만약에 얼음화살 마법을 추가로 배웠을때는 Runtime상태에서 바꾸질 못한다. 