# 데코레이터 패턴

"상속맨, 디자인에 눈을 뜨다" <br>
실행 중에 클래스를 꾸미는, 즉 원래 클래스의 코드는 전혀 바꾸지 않고 객체에 임무를 부여하는 패턴을 알아보자.

<br>

## 스타버즈 커피점

```Swift
class Beverage {
    var description: String
    
    func getDescription() -> String {
        return description
    }
    func cost() -> Double { }
}
```
스타버즈가 처음 사업을 시작할 때, 위와 같은 Beverage 클래스를 두고 여러 종류들에 대해서는 super 클래스를 상속받아서 구현하는 방법으로 구현을 했다고 한다. 

![IMG_9925](https://user-images.githubusercontent.com/45090197/222637236-7f516d88-5694-4ca4-a546-585ac82877e4.jpeg)

그러다보니 메뉴가 많아지니 위와 같이 클래스의 개수가 폭발적으로 늘어나는 문제가 발생한다. <br>

![IMG_9926](https://user-images.githubusercontent.com/45090197/222638008-99e99502-167b-4e14-bed0-5ffa7579e2b5.JPG)

이를 해결하기 위해서 위와 같은 방법을 적용해보자.
인스턴스 변수와 super 클래스의 상속을 써서 추가 사항들을 관리하는 방법이다. 

```Swift
class Beverage {
    var description: String
    var milk: Bool
    var milkCost: Double
    var soy: Bool
    var soyCost: Double
    var mocha: Bool
    var whip: Bool 
    
    func getDescription() -> String {
        return description
    }
    
    func cost() -> Double { 
        var result = 0.0
        if hasMilk() { result += milkCost }
        if hasSoy() { result += soyCost }

        return result
    }

    func hasMilk() -> Bool { self.milk }
    func setMilk() {}
    func hasSoy() -> Bool { self.soy }
    func setSoy() {}
}

class DarkRoast: Beverage {
    init() {
        self.description = "최고의 다크 로스트 커피"
    }

    override func cost() -> Double {
        return 1.99 + super.cost()
    }

    override func setMilk() {
        self.milk = false
    }

    override func setSoy() {
        self.soy = false
    }
}
```

이렇게 하면 기존의 방법과는 달리 클래스의 개수가 폭발적으로 증가하는 문제는 해결되는 것처럼 보이긴한다. <br>
하지만 아직 아래와 같은 문제들이 남아있다.

1. 첨가물의 가격이 바뀔때마다 **기존의 코드를 수정**해야한다
2. 첨가물의 **종류가 추가되면 새로운 메소드가 추가**되어야하고, **super클래스의 cost() 내부도 수정**되어야한다
3. 새로운 음료가 출시될 때, 만약 첨가물이 들어가면 안되는 경우가 있을텐데 이러한 경우 **불필요한 메서드 (아이스티의 경우에는 hasWhip()같은 메서드)를 상속받게 된다**
4. 손님이 **custom한 주문(더블 모카)을 하는 경우에는 대응이 불가능**하다. 

<br>

> 상속 VS 구성과 위임
>
> subclass를 만드는 방식으로 행동을 상속받으면, 해당 행동은 컴파일시에 완전 결정됨 <br>
> 또한 모든 subclass에서 해당 행동을 상속받아야함
> 
> 구성을 통해 객체의 행동을 확장하면, 실행중에 동적으로 행동을 설정할 수 있음 <br>
> 따라서 super클래스의 코드를 건드리지 않고, super에 없는 행동들도 추가할 수 있음 <br>
> 즉, 기존코드를 건드리지 않고 새로운 코드를 만들어서 새로운 기능을 추가 할 수 있음 <br>
> 이는 **기존 코드에 버그가 생기거나 sideEffect가 발생하는 것 자체를 원천봉쇄**할 수 있음 

<br>

> 디자인 원칙 (OCP - open closed principle) <br>
> 클래스는 확장에 대해서 열려 있어야하지만 코드 변경에 대해서는 닫혀 있어야한다.
> 
> 코드에서 확장하는 부분을 선택하는 것은 신중해야하며, 무조건 OCP를 적용하는 것이 좋은 것만은 아니다. 오버엔지니어링일 수 도 있으므로 어디에 적용할지는 신중히 결정하자.

<br>

## 데코레이터 패턴

> 정의 <br>
> 객체에 추가적인 요건을 동적으로 첨가한다. <br>
> 데코레이터는 서브 클래스를 만드는 것을 통해서 기능을 유연하게 확장할 수 있는 방법을 제공한다. 

<br>

 ![IMG_9927](https://user-images.githubusercontent.com/45090197/222641084-35f09941-758f-4759-a817-2a809e55b009.JPG)

1. Decorator의 superClass는 자신이 데코레이트 하고 있는 객체의 super 클래스(Component)와 동일하다
2. 한 객체를 여러 개의 Decorator로 감쌀 수 있다
3. Decorator는 자신이 감싸고 있는 객체와 같은 super 클래스를 가지고 있기 때문에 원래 객체(싸여져 있는 객체)가 들어갈 자리에 Decorator 객체를 집어넣어도 괜찮다
4. **Decorator는 자신이 데코레이트 하고 있는 객체에 어떤 행동을 위임하는 것 외에 원하는 추가적인 작업이 가능하다**
5. 객체를 감싸는 것은 언제든지 가능하기에 runtime에 이를 마음대로 적용할 수 있다


## 데코레이터 in 스타버즈

예를 들어 손님이 모카에 휘핑크림을 추가한 다크로스트 커피를 주문한다고 하자.

1. DarkRoast 객체를 가져온다
2. Mocha 객체(decorator)로 장식
3. Whip 객체(decorator)로 장식
4. cost() 메소드를 호출한다. 이때 첨가물의 가격 계산하는 일은 각각의 객체들에게 위임된다

<br>

![IMG_9928](https://user-images.githubusercontent.com/45090197/222642798-538b733e-4a98-405b-959a-9f64ce01b3de.JPG)

이때 Beverage는 추상클래스로 구현이 되어있는데, 구성요소의 형식만 상속되는 거라면 사실상 클래스가 아닌 인터페이스로 생성해도 될텐데 그 이유가 무엇일까?

실제 데코레이터 패턴에서도 인터페이스, 추상클래스 둘 중에 무엇을 사용해도 상관이 없다. <br>
그러나 **기존 코드를 고치는 것 자체를 피하기 위해서는 추상클래스를 사용하는 것이 좋다.**

<br>

```Swift
protocol Beverage { 
    var description: String { get set }
    func getDescription() -> String 
    func cost() -> Double
}

extension Beverage {
    func getDescription() -> String { description }
}

protocol CondimentDecorator: Beverage { }

class DarkRoast: Beverage {
    init() {
        self.description = "다크로스트 커피"
    }

    func cost() -> Double {
        return 0.99 
    }
}

class HouseBlend: Beverage {
    init() {
        self.description = "하우스 블랜드 커피"
    }

    func cost() -> Double {
        return 0.89 
    }
}

class Mocha: CondimentDecorator {
    let beverage: Beverage 
    init(beverage: Beverage) {
        self.beverage = beverage
    }

    func getDescription() -> String {
        return beverage.getDescription() + ", 모카"
    }
    func cost() -> Double {
        return beverage.cost() + 0.20
    }
}

class Whip: CondimentDecorator {
    let beverage: Beverage 
    init(beverage: Beverage) {
        self.beverage = beverage
    }

    func getDescription() -> String {
        return beverage.getDescription() + ", 휘핑크림"
    }
    func cost() -> Double {
        return beverage.cost() + 0.15
    }
}
```
이제 이를 실제로 사용하는 부분을 보자

```Swift
class StarBuzzCoffee {
    func makeCoffee() {
        let beverage = DarkRoast()
        print("\(beverage.getDescription()) $\(beverage.cost())")

        // 이렇게 매번 생성하는 부분은 추후에 factory pattern을 배우고 개선해보자!!
        var beverage2 = HouseBlend()
        beverage2 = Mocha(beverage2)
        beverage2 = Mocha(beverage2)
        beverage2 = Whip(beverage2)
        print("\(beverage2.getDescription()) $\(beverage2.cost())")
    }
}

//
let starbuzzCoffee = StarBuzzCoffee()
starbuzzCoffee.makeCoffee()
// 다크로스트 커피 $0.99
// 다크로스트 커피, 모카, 모카, 휘핑크림 $1.49
```

데코레이터를 적용했을때 아래와 같은 질문이 생길 수 있다. <br>
Q1. 특정 구상 구성요소인지 확인한 다음 어떤 작업을 처리할 수 있을지? (하우스 블랜드면 이를 할인해주는 코드 etc) <br>
A1. 데코레이터 패턴은 **추상 구성요소 형식을 바탕으로 돌아가는 코드에 대해서만 적용**이 가능하다. 구상 구성요소를 바탕으로 돌아가야하는 코드가 필요하다면 데코레이터 패턴 적용을 다시 검토해보자. <br>

Q2. 데코레이터 패턴을 사용하면 뺴먹을 실수는 없을까 (순서나 두 번 넣어야하는걸 한 번만 넣는다던지) <br>
A2. 데코레이터 패턴을 사용하면 관리하는 객체가 늘어나고, 당연히 실수 가능성이 커진다. 그러나 이는 추후에 팩토리나 빌더 같은 다른 패턴을 써서 사용하면 해결할 수 있다. <br>

## 정리

Decorator 패턴을 사용하면 객체에 추가 요소를 동적으로 더할 수 있다. <br>
상속의 subclass를 만드는 경우에 비해 훨씬 유연하게 기능을 확장할 수 있다. <br>

그러나 특정 구성 요소의 구체적인 형식에 의존하는 경우에는 이 패턴 적용을 고려해 볼 필요가 있으며, 해당 패턴을 사용하면 자잘한 객체들이 매우 많이 추가될 수 있으므로 오버엔지니어링 여부를 잘 판단해보자. 
