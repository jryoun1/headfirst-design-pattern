# Observer Pattern

## 기상 모니터링 애플리케이션 개요

![image](https://user-images.githubusercontent.com/45090197/222416962-b74bad80-0770-4960-9e52-714a4586c54a.png)

- 기상 스테이션: 실제 기상 정보를 수집하는 장비
- WeatherData 객체: 기상스테이션으로부터 받은 데이터를 추적하는 객체
- 디스플레이: 기상 조건을 사용자에게 보여주는 기기

이때 WeatherData 객체를 사용해 현재 조건, 기상 통계, 기상 예측 3가지 항목을 디스플레이 장비에서 갱신해서 보여줄 수 있는 장비를 생성해야함.

```Swift
class WeatherData {
    func getTemperature()
    func getHumidity()
    func getPressure()
    func measurementsChanged() // 기상 관측값이 갱신될 때마다 알려주기 위한 메소드
}
```
이때 WeatherData의 소스파일은 위와 같이 생겼다고 하자.

<br>

## 해야할 일 목록

현재 알고 있는 정보 <br>
1. WeatherData 클래스에는 3가지 측정값을 알아낼 수 있는 getter 존재
2. 새로운 기상 측정 데이터가 나올 때마다 `measurementsChanged()` 가 호출됨 

<br> 

해야하는 일 <br>
1. 기상 데이터를 사용하는 세 개의 디스플레이 항목을 구현해야함
- 현재 조건 표시 / 기상 통계 표시 / 기상 예보 표시 
- 이들 디스플레이는 값이 변경될 때마다 업데이트 되어야함

<br>

2. 시스템이 확장가능해야함
- 다른 회사에서도 별도의 디스플레이 항목을 생성할 수 있어야하고, 이러한 디스플레이 항목의 추가/제거가 가능해야함

<br>

## 1차적인 구현 방법

```Swift
class WeatherData {
    // ...
    func measurementsChanged() {
        var temp = getTemperature()
        var humidity = getHumidity()
        var pressure = getPressure()

        currentConditionsDisplay.update(temp, humidity, pressure)
        statisticsDisplay.update(temp, humidity, pressure)
        forecastDisplay.update(temp, humidity, pressure)
    }
}
```
그러나 위와 같이 구현을 하게 되면 아래와 같은 문제점들이 발생하게 된다. <br>

1. 인터페이스가 아닌 구체적인 구현을 바탕으로 코딩되어있음
2. 새로운 디스플레이가 생성될 때마다 WeatherData의 measurementsChanged()가 수정되어야함
3. runtime에 디스플레이 항목이 추가/제거 될 수 있음
4. 바뀌는 부분 (temp, humidity, pressure)이 캡슐화 되어있지 않음

<br>

## 옵져버 패턴

신문, 잡지는 어떻게 구독하고 있을까? <br>

1. 신문사가 신문을 찍어냄
2. 독자가 신문사에 구독 신청을 하면, 신문사는 새로운 신문이 나올 때마다 이를 배달해주고 독자는 이를 받을 수 있음
3. 신문이 더 이상 보기 싫으면 구독을 끊으면 됨
4. 신문사가 영업을 하는 동안 여러 개인 독자, 회사에서 꾸준히 구독 신청 및 해지가 발생하게 됨

따라서 `출판사 + 구독자 = 옵져버 패턴` 이라고 할 수 있다. <br>
이때 출판사를 Subject(주제), 구독자를 Observer(옵저버) 라고 부른다는 것만 알면 된다.  

<br>

### 정의 

> 옵저버 패턴에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다(one - to - many) 의존성을 정의한다.

<br>

클래스다이어그램은 아래와 같다. 

 ![IMG_F00F9AD05B57-1](https://user-images.githubusercontent.com/45090197/222424078-e28fef47-9d24-4972-b6f9-d09513321f23.jpeg)

<br>

Subject: 주제를 나타내는 인터페이스
- 객체에서 옵저버를 등록/탈퇴할 때는 인터페이스 내의 메소드를 사용하면 된다.

ConcreteSubject: 주제 역할을 하는 구상 클래스에서는 Subject 인터페이스를 구현함
- 옵저버의 등록/탈퇴를 하는 메서드 구현
- notifyObservers() 라는 주제 상태 변경될 때 모든 옵져버에게 알리는 메서드 구현필요

Observer: 옵저버가 될 가능성이 있는 객체는 반드시 해당 인터페이스를 구현해야함
- 주제의 상태가 바뀌었을 때 호출되는 update() 메소드를 가지고 있음

ConcreteObserver: Observer인터페이스만 구현했다면 모두 옵저버 클래스가 될 수 있음

> 느슨한 결합(Loose Coupling) <br>
두 객체가 느슨하게 결합되었다 == 그 둘이 상호작용을 하지만 서로를 잘 모른다는 것을 의미 <br>
옵저버 패턴에서 주제와 옵저버는 느슨하게 결합되어있는 객체 디자인을 제공한다.

<br>

- 주제가 옵저버에 대해 아는 것은 옵저버가 Observer 인터페이스를 구현했다는 사실뿐
- 옵저버는 언제든지 새로 추가가 가능하다 
- 새로운 형식의 옵저버가 추가되어도 주제를 변경할 필요가 없다
- 주제와 옵저버는 독립적으로 재사용할 수 있다
- 주제나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않음

> 디자인 원칙 <br>
> 서로 상호작용을 하는 객체 사이에서는 가능하면 느슨하게 결합하는 디자인을 사용해야한다.

<br>

## 기상 스테이션 설계 및 구현

![IMG_A7064E8247C7-1](https://user-images.githubusercontent.com/45090197/222425789-05ad3158-1377-4d63-8b77-03b7425a84df.jpeg)

Subject 인터페이스와 이를 준수하는 WeatherData 클래스 생성하는 코드는 아래와 같다. 

```Swift
protocol Subject {
    func register(observer Observer)
    func remove(observer Observer)
    func notifyObservers()
}

class WeatherData: Subject {
    private var observers = [Observers]()
    private var temp: Float
    private var humidity: Float
    private var pressure: Float

    init() {
        observers = []
    }

    func register(observer: Observer) {
        observers.append(observer)
    }

    func remove(observer: Observer) {
        if let index = observers.firstIndex(of: observer) {
            observers.remove(at: index)
        }
    }

    func notifyObservers() {
        observers.forEach { $0.update(temp, humidity, pressure) }
    }

    func measurementsChanged() {
        notifyObservers()
    }
    func getTemperature() { ... }
    func getHumidity() { ... }
    func getPressure() { ... }
}
```
<br>

디스플레이 항목들을 생성하는 코드
```Swift
protocol Observer {
    func update(temp: Float, humidity: Float, pressure: Float) 
}

protocol Display {
    func display()
}

class CurrentConditionsDisplay: Observer, Display {
    private var weatherData: WeatherData
    private var temp: Float
    private var humidity: Float

    init(weatherData: WeatherData) {
        self.weatherData = weatherData
        self.weatherData.register(Self)
    }

    func update(temp: Float, humidity: Float, pressure: Float) {
        self.temp = temp
        self.humidity = humidity
        display()
    }

    func display() { }
}
```
이때 subject에 대한 레퍼런스인 weatherData는 추후에 옵저버 목록에서 탈퇴할 때 주제 객체에 대한 레퍼런스를 저장해 탈퇴를 편리하게 하기 위해서 저장하게 된다. 

<br>

## Java 내장 옵저버 패턴


이렇게 옵저버 패턴을 구현해봤는데, 실제로 Java에는 이미 내부에서 옵저버 패턴을 지원해주고 있다고 한다. <br>
직접 구현한 부분과 다른점은 기존에 인터페이스였던 Subject가 Observable 클래스로 구현이 되어있다. 

![IMG_F4399E5214E1-1](https://user-images.githubusercontent.com/45090197/222428886-41407a2a-7a9e-4410-ae7d-01389688a267.jpeg)

따라서 기존의 register, remove, notify와 같은 함수들을 구체 주제 클래스에서 매번 구현할 필요없이 상속받는 메서드를 그대로 사용하면 된다. 

### Push, Pull 방식

객체가 옵처버가 되는/탈퇴하는 방법
- Observer 인터페이스를 구현하고 Observable 객체의 addObserver()/deleteObserver()를 호출하면 된다

Observable에서 연락을 돌리는 방법
- Observable super을 확장하는 클래스를 구현한 뒤
- setChanged()를 호출해 객체의 상태가 바뀜을 알림
- notifyObservers() / notifyObservers(Object arg) 둘 중에 하나를 호출 

옵저버가 연락을 받는 방법
- update(Observable o, Object arg)를 구현함 
- o는 연락을 보내는 주제 객체 / arg는 인자로 전달된 데이터 객체이며, 따로 지정하지 않으면 null
- 데이터를 포함해서 보낼 때는 push 방식 
- 전달받은 Observable 객체로부터 원하는 데이터를 가져가는 pull 방식


### 자바 내장 옵저버 패턴의 단점

Observable이 인터페이스가 아닌 클래스이다.
- 서브 클래스를 만들어야하고, 그렇기에 Observable의 기능을 추가할 수 없다 -> 재사용성의 문제
- Observable 인터페이스가 없기에 자바에 내장된 Observer API와 잘 맞는 클래스를 직접 구현하는 것이 불가능하고, java.util의 구현을 다른 구현으로 변경하는 것도 불가능
- 핵심 메소드인 setChanged()는 protected로 선언되어있어 외부에서 호출할 수 없다

<br>

따라서 필요하다면 직접 구현을, 그렇지 않고 상속을 통해서 해결할 수 있다면 기본 제공 옵저버 패턴을 사용하면 된다. 

