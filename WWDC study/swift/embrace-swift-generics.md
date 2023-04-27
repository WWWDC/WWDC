---
description: Swift에서 Generic과 Protocol을 이용해서 코드를 추상화해보자!
---

# Embrace Swift generics

### 소를 키우는 농장 시스템 구축

<details>

<summary>농장에 필요한 기능</summary>

* 소는 건초(Hay)를 먹는다.
* 건초는 자라면 Alfalfa가 된다.
* Alfalfla를 수확하면 건초가 된다.
* 농장에서는 소에게 먹이를 줘야한다. (알파파 수확 -> 건초 획득 -> 소가 먹는다)

</details>

```swift
import UIKit

// 1) 건초를 먹는 소
struct Cow {
    func eat(_ food: Hay) { }
}

// 2) 자라면 Alfalfa가 되는 건초
struct Hay {
    static func grow() -> Alfalfa { return Alfalfa() }
}

// 3) 수확하면 건초가되는 Alfalfa
struct Alfalfa {
    func harvest() -> Hay { return Hay() }
}

// 4) 농장에서는 소에게 음식을 준다.
struct Farm {
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }
}

```



### 다양한 동물을 키우는 농장 시스템 구축

<details>

<summary>농장에 필요한 기능</summary>

* 각 동물은 먹이를 먹어야하고, 먹이의 종류는 각각 다르다.
* 먹이마다 수확, 재배 방법이 다르다.
* &#x20;농장에서는 동물들에게 먹이를 줘야한다.

</details>

```swift
import Foundation

// MARK: - 소
struct Cow {
    func eat(_ food: Hay) { } // 소는 건초를 먹는다.
}

struct Hay {
    static func grow() -> Alfalfa { return Alfalfa() } // 재배한다.
}

struct Alfalfa {
    func harvest() -> Hay { return Hay() } // 수확하여 건초를 얻는다.
}

// MARK: - 닭
struct Chicken {
    func eat(_ food: Grain) { }
}

struct Grain {
    static func grow() -> Wheat { return Wheat()}
}

struct Wheat {
    func harvest() -> Grain { return Grain() }
}

// MARK: - 말
struct Horse {
    func eat(_ food: Carrot) { }
}

struct Carrot {
    static func grow() -> Root { return Root() }
}

struct Root {
    func harvest() -> Carrot { return Carrot() }
}

// MARK: - 농장
struct Farm {
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }
    
    func feed(_ animal: Chicken) {
        let wheat = Grain.grow()
        let grain = wheat.harvest()
        animal.eat(grain)
    }
    
    func feed(_ animal: Horse) {
        let root = Carrot.grow()
        let carrot = root.harvest()
        animal.eat(carrot)
    }
}
```

* 소, 닭, 말에게 먹이를 주기 위해서 feed 함수를 오버로드했다.
  * 각 오버들이 매우 비슷하게 구성되어있다.
* 동물들이 더 많아지게 되면 어떤 상황이 발생할까?
  * 더 많은 보일러 플레이트가 발생할 것이다.
  * 이처럼 보일러 코드들이 많은 상황에서 일반화를 고려해야한다.

#### 보일러 플레이트(Boilerplate)

최소한의 변경으로 여러곳에서 재사용되며, 반복적으로 비슷한 형태를 띄는 코드

즉, 비슷한 코드를 계속 작성하는 문제가 생긴다



## Identify Common Capabilities (공통 기능 식별하기)

* 아까 보일러 플레이트 문제를 해결하기 위해서 공통 기능을 식별해보자.
  * 동물은 특정 음식을 먹는 공통점이 있다.
  * 하지만, 각 동물마다 먹는 방식이 다르다.
  * 추상화된 코드에서 **`eat()`**을 호출할 수 있어야하고, 각 동물마다 eat이 다르게 동작되어야한다.

### 다형성

* 하나의 코드로 여러 동작을 수행하는 능력을 다형성이라고 한다.
  * 즉, eat이라는 메서드가 상황에 따라 각각 다른 일을 할 수 있는 일이다.
  * 다형성을 통해서 eat이라는 하나의 형태로 다른 일을 시킬 수 있게 된다.

### 다형성의 형태

1. **함수 오버로드 (ad-hoc polymorphism)**
   1. 인수 유형에 따라 다양한 동작을 정의할 수 있다.
   2. 반복적인 코드를 계속 사용하게 된다.(보일러 플레이트)
2. **오버라이딩 (Subtype polymorphism)**
   1. 런타임 시 Type에 따라 런타임에 동작이 결정된다.
3. **Generic (parametric polymorphism)**
   1. 유형 매개변수를 이용한다.
   2. 구체적인 유형 자체가 인자로 사용된다.

### Subtype polymorphism을 이용한 리팩토링

```swift
import Foundation

// 1) 소, 닭, 말을 표현할 수 있는 Animal class의 도입
// 2) 각 동물 struct를 class로 변경한다.
// 3) Animal class를 상속하여, eat 메서드를 재정의한다.

class Animal {
    func eat(_ food: Any) {
        fatalError("Subclass must implement 'eat'")
    }
}


class Cow: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Hay else { fatalError("Cow can't eat \(food)") }
    }
}

struct Hay {
    static func grow() -> Alfalfa { return Alfalfa() } // 재배한다.
}

struct Alfalfa {
    func harvest() -> Hay { return Hay() } // 수확하여 건초를 얻는다.
}

// MARK: - 닭
class Chicken: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Grain else { fatalError("Chicken can't eat \(food)") }
    }
}

struct Grain {
    static func grow() -> Wheat { return Wheat()}
}

struct Wheat {
    func harvest() -> Grain { return Grain() }
}

// MARK: - 말
class Horse: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Carrot else { fatalError("Horse can't eat \(food)") }
    }
}

struct Carrot {
    static func grow() -> Root { return Root() }
}

struct Root {
    func harvest() -> Carrot { return Carrot() }
}

let h = Horse()
h.eat(Carrot())
h.eat(Hay())
```

### 문제점

* 각 동물마다 먹이가 다르기 때문에, eat을 Any로 정의했다(이런 종속성은 클래스로 표현하기 어렵다).
  * 적합하지 않은 먹이를 주는 경우, 런타임에서 오류를 확인할 수 있다.
  * 즉, 적합성 확인이 하위 클래스 구현에 의존되게 되는 문제가 발생했다.
* eat은 재정의하지 않으면 오류를 뱉는다.
  * 필수로 정의되어야함을 의미하지만, 이 역시 런타임에서 오류를 확인할 수 있다.

### Type Parameter로 문제 해결해보기

```swift
import Foundation

class Animal<Food> { // Food: type parameter
    func eat(_ food: Food) { fatalError("Subclass must implement 'eat'") }
}

class Cow: Animal<Hay> {
    override func eat(_ food: Hay) { }
}

struct Hay {
    static func grow() -> Alfalfa { return Alfalfa() } // 재배한다.
}

struct Alfalfa {
    func harvest() -> Hay { return Hay() } // 수확하여 건초를 얻는다.
}

// MARK: - 닭
class Chicken: Animal<Grain> {
    override func eat(_ food: Grain) { }
}

struct Grain {
    static func grow() -> Wheat { return Wheat()}
}

struct Wheat {
    func harvest() -> Grain { return Grain() }
}

// MARK: - 말
class Horse: Animal<Carrot> {
    override func eat(_ food: Carrot) { }
}

struct Carrot {
    static func grow() -> Root { return Root() }
}

struct Root {
    func harvest() -> Carrot { return Carrot() }
}

let c = Cow()
c.eat(Hay())
c.eat(Carrot()) // compile error!!
```

### Type parameter의 문제점

* 동물을 선언할 때마다 먹이의 종류를 명시해줘야한다.
  * 동물의 핵심이 먹이야? -> NO!
  * 핵심 개념도 아닌데, 이렇게 명시적으로 표현하는 것 자체가 어색한 상황
* 더 많은 함수들이 생기면, type parameter 개수 감당 가능?
  * 더 많은 보일러 플레이트가 생길 것이다.
  * 코드가 더 복잡해진다.



## Build an interface

아이디어와 세부 구현을 분리해서 문제를 해결할 수 있어.

protocol을 이용하는거지.

### Protocol

* 추상화 도구로, 유형의 기능을 선언하는 청사진이다.
* 아이디어와 세부 구현을 분리할 수 있다
  * 아이디어: 인터페이스를 통해 표현한다.
* class, struct, enum, actor에서도 활용이 가능하다.

### 정의했던 공통 기능

* 특정 타입의 먹이가 있어야한다. (associatedtype 이용)
* 먹이를 소비하는 작업이 있다.

### 불투명 타입 (Opaque Type)

* 구체적인 타입이 아닌 타입!
  * 파라미터로 받거나, 리턴형에 사용할 수 없다.
  * 프로토콜과 같은 타입이 해당된다.
* 불투명 타입을 파라미터나 리턴형으로 사용하기 위한 방법
  * 값의 범위를 지정해줘야한다.
  * 임의 타입의 이름을 지정해주고, 해당 타입이 특정 프로토콜을 따름을 표현해줄 수 있다.

```swift
func feed<A>(_ animal: A) { } // 정의 불가능
func feed<A: Animal)(_ animal: A) { }
func feed<A>(_ animal: A) where A: Animal { }
func feed(_ animal: some Animal) { }
```



```swift
import Foundation

// 1. 동물의 기능을 인터페이스로 표현한다.
protocol Animal {
    associatedtype Feed: AnimalFeed // 임의의 먹이을 지정한다.
    func eat(_ food: Feed) // 실제 구현은 상세 정보에서 구현한다. -> 어떤 타입이든 파라미터로 받을 수 있다.
}

protocol AnimalFeed {
    associatedtype CropType: Crop where CropType.FeedType == Self
    static func grow() -> CropType
}

protocol Crop {
    associatedtype FeedType: AnimalFeed where FeedType.CropType == Self
    func harvest() -> FeedType
}


// 2. 각 동물이 프로토콜을 따르도록 할 수 있다.
// 프로토콜은 클래스, 구조체, 열거형, actor 등에서도 사용할 수 있다.
struct Cow: Animal {
    func eat(_ food: Hay) { }
}

struct Horse: Animal {
    func eat(_ food: Carrot) { }
}

struct Chicken: Animal {
    func eat(_ food: Grain) { }
}

struct Hay: AnimalFeed {
    static func grow() -> Alfalfa { return Alfalfa() }
}

struct Alfalfa: Crop {
    func harvest() -> Hay { return Hay() }
}


struct Grain: AnimalFeed {
    static func grow() -> Wheat { return Wheat()}
}

struct Wheat: Crop {
    func harvest() -> Grain { return Grain() }
}

struct Carrot: AnimalFeed {
    static func grow() -> Root { return Root() }
}

struct Root: Crop {
    func harvest() -> Carrot { return Carrot() }
}

struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()
        let produce = crop.harvest()
        animal.eat(produce)
    }
}
```

### 여러마리 동물에게 음식을 주는 함수 작성

* 배열 하나에 여러 타입을 담기 위해 사용하는 키워드 Any!
  * generic에서도 마찬가지로 적용할 수 있어.
  * func feedAll(\_ animals: \[Any Animal])

#### Any 키워드

* 타입을 동적으로 저장할 수 있어.
* 그 타입은 런타임에서 결정 돼.
* 그런데, Any 타입의 배열은 정적 타입은 같은데, 동적 타입은 다르겠지?
* 이걸 해결하는 것이 바로! 프로토콜이 되는거야\~!\~!

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Feed.grow()
        let produce = crop.harvest()
        animal.eat(produce)
    }
    
    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            feed(animal)
        }
    }
}
```
