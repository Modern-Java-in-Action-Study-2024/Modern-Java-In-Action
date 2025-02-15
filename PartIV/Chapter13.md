# 디폴트 메서드
## 13.1 변화하는 API

### 13.1.1 API 버전 1
- `Resizable` 인터페이스 초기 버전
```Java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
}
```
- 사용자 구현
```Java
public class Ellipse implements Resizable {
    ...
}
```

### 13.1.2 API 버전 2
- `Resizable` 인터페이스를 개선해달라는 요청을 받은 후 버전
```Java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    void setRelativeSize(int wFactor, int hFactor);  // 추가된 메서드
}
```
> 문제점 발생
> 1. Resizable을 구현하는 모든 클래스는 setRelativeSize 메서드 구현해야 함<br/>
>    그러나 API 초기 버전을 상속받아 직접 구현한 Ellipse 클래스는 setRelativeSize를 구현하지 않음<br/>
>    인터페이스에 새로운 메서드를 추가하면 당장의 바이너리 호환성을 유지될지 몰라도<br/>
>    Ellipse 객체가 인수로 전달되면 Ellipse는 setRelativeSize 메서드를 정의하지 않았으므로 `런타임 에러` 발생
>    
> 2. Ellipse 클래스를 포함한 전체 애플리케이션을 재빌드할 때 `컴파일 에러` 발생

디폴트 메서드가 새롭게 바뀐 인터페이스에서 자동으로 기본 구현을 제공하므로 기존 코드를 고치지 않아도 됨

## 13.2 디폴트 메서드란 무엇인가?
- 인터페이스 내부에서 `default` 키워드를 사용하여 구현할 수 있는 메서드
- 디폴트 메서드를 사용하며 기존 인터페이스를 변경하지 않고도 새로운 기능을 추가할 수 있음
```Java
public interface Resizable extends Drawable {
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    default void setRelativeSize(int wFactor, int hFactor) {
        setAbsoluteSize(getWidth() / wFactor, getHeight() / hFactor);    // 디폴트 메서드
    }
}
```

## 13.3 디폴트 메서드 활용 패턴
### 13.3.1 선택형 메서드 (optional method)
어떤 인터페이스를 구현하는 클래스들이 반드시 필요로 하지는 않지만, 편의상 제공하면 좋은 기능을 구현할 때 사용<br/>
반드시 모든 구현체가 이를 재정의하지 않아도 됨
> List 인터페이스의 sort 메서드, Iterator 인터페이스의 remove 메서드 등

### 13.3.2 동작 다중 상속 (multiple inheritance of behavior)
디폴트 메서드를 통해 다중 상속처럼 동작할 수 있음
```Java
public interface Rotatable {    // 회전
    void setRotationAngle(int angleInDegrees);
    int getRotationAngle();
    default void rotateBy(int angleInDegrees) {
        setRotationAngle((getRotationAngle() + angleInDegrees) % 360);
    }
}

public interface Moveable {    // 움직임
    int getX();
    int getY();
    void setX(int x);
    void setY(int Y);
    default void moveHorizontally(int distance) {
        setX(getX() + distance);
    }
    default void moveVertically(int distance) {
        setY(getY() + distance);
    }
}

public interface Resizable {    // 크기 조절
    int getWidth();
    int getHeight();
    void setWidth(int width);
    void setHeight(int height);
    void setAbsoluteSize(int width, int height);
    default void setRelativeSize(int wFactor, int hFactor) {
        setAbsoluteSize(getWidth() / wFactor, getHeight() + hFactor);
    }
}
```
```Java
public class Monster implements Rotatable, Moveable, Resizable {
    ...
}
...
Monster m = new Monster();
m.rotateBy(180);         // Rotatable 인터페이스의 디폴트 메서드 사용
m.moveVertically(10);    // Moveable 인터페이스의 디폴트 메서드 사용
...
```
> Monster 클래스가 여러 인터페이스를 구현하면서 각 인터페이스의 디폴트 메서드를 조합하여 동작을 확장하고<br/>
> 인터페이스의 기능을 직접 사용할 수 있음
```Java
public class Sun implements Moveable, Rotatable {
    ...
}
```
> Sun 클래스처럼 필요한 기능만 선택적으로 구현 가능

## 13.4 해석 규칙
디폴트 메서드를 사용할 때 충돌을 일으킬 수 있기 때문에 해결 규칙이 필요함

### 13.4.1 알아야 할 세 가지 해결 규칙
> 1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.<br/>
>
> 2. 1번 규칙 이외의 상황에서는 서브인터페이스가 이긴다.<br/>
>    상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다.<br/>
>
> 3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받는 클래스가<br/>
>    명시적으로 디폴트 메서드를 오버라이드하고 호출해야 한다.

### 13.4.2 디폴트 메서드를 제공하는 서브인터페이스가 이긴다.
```Java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A {    // B가 A를 상속 받기 때문에 B가 A를 이김
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    public static void main(String[] args) {
        new C().hello();    // Hello from B 출력
    }
}
```

### 13.4.3 충돌 그리고 명시적인 문제 해결
```Java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}

public class C implements B, A {
    void hello() {
        B.super.hello();    // 명시적으로 인터페이스 B의 메서드를 선택함
    }
}
```

### 13.4.4 다이아몬드 문제
여러 인터페이스에서 동일한 디폴트 메서드를 제공할 경우 발생하는 충돌 문제<br/>
클래스가 어느 디폴트 메서드를 상속해야 하는지 모호해지므로 명시적으로 해결해야 함
```Java
public interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}

public interface B extends A { }

public interface C extends A { }

public class D implements B, C {
    public static void main(String[] args) {
        new D().hello();
    }
}
```
> 컴파일 오류 발생<br/>
> Duplicate default methods named hello with the parameters ()
> and the result type void are inherited from the types B and C<br/>
> D가 B와 C를 동시에 상속하는데, B와 C 모두 A로부터 동일한 디폴트 메서드 hello()를 상속받기 때문

### 다이아몬드 문제 해결방법
1. 명시적으로 재정의하기 (@Override)
```Java
Class D implements B, C {
    @Override
    public void hello() {
        System.out.println("Hello from D");
    }
}
```
2. 특정 인터페이스의 메서드를 선택하여 호출 (super 키워드 사용)
```Java
class D implements B, C {
    @Override
    public void hello() {
        B.super.hello();
    }
}
```

## 13.5 마치며
- 디폴트 메서드의 정의는 default 키워드로 시작하며 일반 클래스 메서드처럼 바디를 가짐
- 디폴트 메서드 덕분에 기존 API를 바꿔도 기존 버전과 호환성을 유지할 수 있음
- 디폴트 메서드를 상속하면서 생기는 충돌 문제를 해결하는 규칙이 있음<br/>
  (클래스&슈퍼클래스 > 서브인터페이스 > 명시적 결정)
