# effective-java 3/e


## 객체 생성과 파괴
1/10 (목) - item 1~7

### 1. 생성자 대신 팩터리 메서드를 고려하라

정적 팩토리 메소드를 쓰래. (static factory method)
```
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
이렇게 생겼음

정적 팩토리 메소드가 생성자보다 좋은 장점 5개

1. 이름을 가질 수 있다.
2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
3. 반환타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

그럼에도 안좋은점 2가지

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다.


정적 팩토리 메소드와 public 생성자는 각자의 쓰임새가 있긴하지만 정적팩토리를 사용하는게 유리한경우가 더 많다. 무작정 public 생성자를 제공하지 말자.



### 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

1번의 정적팩토리나 생성자는 선택적 매개변수가 많을 경우 만들기 어려워짐.

보통은 점층적 생성자 패턴을 많이 이용했겠지만 확장의 한계가 있다. 못하는것은 아니나 이건 아니다 싶게 많이 만들어내야 할떄가 있음

이걸 자바빈즈 패턴으로 만들수도 있는데, 이는 인스턴스를 만들기 쉽고 읽기 쉬운 코드로 만들어 준다. 그러나 객체 하나를 만드려면

메소드를 여러개로 호출해야하고 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태가 된다.

빈즈 패턴은 클래스를 불변으로 만들수 없기떄문에 스레드 세이프하려면 추가 작업이 꼭 필요하다.

freezing 을 하는 방법도 있지만 다루기 어렵고 결정적으로 freeze 메소드를 확실히 호출 해줬는지 알 수 없어 런타임 오류에 취약하다

그러니까 다 때려치고, 생성자나 정적 팩토리가 처리해야할 매개변수가 많다면 '빌더' 를 고려하라는 것이다.

빌더 패턴의 대표적인 예는 아래와 같다.

```
public class NutritionFacts {

    private final int 변수1;
    private final int 변수2;
    private final int 변수3;
    private final int 변수4;
    private final int 변수5;
    private final int 변수6;

    public static class Builder {

        // 필수
        private final int 변수1;
        private final int 변수2;

        // 옵션
        private int 변수3 = 0;
        private int 변수4 = 0;
        private int 변수5 = 0;
        private int 변수6 = 0;

        // 필수용은 빌더 생성자로
        public Builder(int 변수1, int 변수2) {
            this.변수1 = 변수1;
            this.변수2 = 변수2;
        }

        // 옵션은 아래 처럼
        public Builder 변수3 (int val) { 변수3=val; }
        public Builder 변수4 (int val) { 변수4=val; }
        public Builder 변수5 (int val) { 변수5=val; }
        public Builder 변수6 (int val) { 변수6=val; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }

    }

    // 이것으로 NutritionFacts 는 불변이 된다.
    private NutritionFacts(Builder builder) {
        변수1 = builder.변수1;
        변수2 = builder.변수2;
        변수3 = builder.변수3;
        변수4 = builder.변수4;
        변수5 = builder.변수5;
        변수6 = builder.변수6;
    }
}


// 그러고 이런식으로 호출가능
NutritionFacts food = new NutritionFacts.Builder(200, 8)
                            .변수1(1)
                            .변수2(2)
                            .변수3(300)
                            .변수4(230)
                            .변수5(123)
                            .변수6(200);

```


### 3. private 생성자나 열거 타입으로 싱글턴임으로 보증하라

싱글턴은 무상태 객체나 설계상 유일해야하는 시스템 컴포넌트에서 사용

생성자를 private 로 놓아 생성자를 감춰버리고 public static 멤버를 남겨두는 방식을 취한다.
```
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuiling() { ... }
}
```
위 방법은 전체 시스템에서 딱 한개 밖에 없는 객체인것을 의미하지만, 리플렉션 기술로 무마될 수 있다.

정적 팩토리 방식의 싱글톤도 있다.
```
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}

```
이것도 위에 있는 예제와 동일한 기능을 하겠지만 싱글턴이 아니게 바꾸기 편하다.


### 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

### 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

### 6. 불필요한 객체 생성을 피하라

### 7. 다 쓴 객체 참조를 해제하라