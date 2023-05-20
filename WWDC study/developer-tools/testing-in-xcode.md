# Testing in Xcode

## 간단한 개념 설명

테스트는 아주 개발의 중요한 단계이다. 소스 코드에서 버그를 찾을 수도 있고, 앱의 예상 동작과 실제 동작이 일치하는지를 확인한다. 테스트 전략을 설계하는데 Test Pyramid에 대한 개념이 있다. (테스트의 종류와 비중)

### 피라미드 모델 (Test Pyramid)

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

#### 단위 테스트 (Unit Test)

* 가장 하위에 위치하는 테스트로, 단일 소스 코드 단위로 테스트한다. (일반적으로 **함수**나 **메서드**를 포함)
  * 모든 테스트의 기초가 되기 때문에, 모든 기능을 포함해야하므로 많은 양의 테스트를 작성해야한다.
  * 함수에 매개변수를 입력하여, 예상 출력을 반환하는지 확인하며 수행된다.
* 빠르고 간단하며, 기능의 버그를 발견하는데 유용하다.

#### 통합 테스트 (Integration Test)

* 서로 다른 구성요소가 올바르게 상호 작용하는지 테스트한다.
  * 개별 하위 시스템 또는 클래스 클러스터를 대상으로 진행한다.
  * 예를 들어 DB 연결, 네트워크 통신, 외부 서비스와 상호 작용과의 테스트가 포함된다.
* 통합 테스트를 구현하기 위해서는 개별 기능이 올바른지 확인 되어야한다. (Unit test이 먼저 진행되어야함)
* 단위 테스트보다 더 많은 리소스와 시간이 소요되며, 앱의 전반적인 품질을 확인에도 유용하다.
  * 실행 시간에서는 Unit test보다 오 걸리지만, 한번에 더 많은 기능을 테스트하게 된다.

#### 사용자 인터페이스 테스트 (UI Test)

* 앱의 실제 UX를 시뮬레이션하여 동작을 확인하고, 테스트한다.
  * 가장 오래 걸리는 테스트로, 사용자가 실제 앱을 사용하는 것과 유사한 시나리오를 포착할 수 있다.
* 앱의 UI가 자주 변경될 수 있기 때문에, 많은 유지보수가 필요한 작업이다



#### Test Pyramid

테스트 피라미드는 3가지 테스트 유형의 비율을 나타내며, 해당 비율을 유지하여 **효율적인 테스트 커버리지를 제공**하며, **실행 시간과 비용을 줄이기 위해 고안된 개념**이다. 즉, 테스트 제품군이 필요한 범위를 지정할 수 있도록 도움을 주는 개념이다.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 실습

### 앱 만들기

**Travel App** : 사용자의 현재 위치를 가져와서, 특정 도시까지의 거리를 계산하는 어플리케이션. Apple에서 코드를 제공해주지 않아서.. 대충만.. 만들었습니다.&#x20;

프로젝트 생성 시 반드시 <mark style="color:red;">**include Tests로 프레임워크를 추가**</mark>해주시는게 편리합니다.

대충 만들어본 travel app [https://github.com/JIWON1923/XCTest/releases/tag/app-version-1.0.0](https://github.com/JIWON1923/XCTest/releases/tag/app-version-1.0.0)

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>



### DistanceCaculator Unit Test

1.  test 파일 만들기



    <figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
2. city function 확인

city함수는 딕셔너리에서 "New York"이라는 키 값이 있다면, 해당 City 구조체 값을 반환하고, 없다면 nil을 리턴하는 함수이다

<pre class="language-swift"><code class="lang-swift"><strong>  func testCoordinatesOfNewYork() throws {
</strong>        
        // 1) 객체 생성
        let calculator = DistancaCaculator()
        
        // 2) 테스트하고자하는 city 함수 호출해야함.
        // 이슈 1 : city는 Optional을 리턴함. 하지만 여기선 not nil을 기준으로 test 필요. 따라서 XCTUnwrap 이용
        // 이슈 2 : XCTUnwrap함수가 Error를 뱉지만, 그것에 대한 핸들을 따로 해주지 않음 -> 함수에 throws 추가
        let city = try XCTUnwrap(calculator.city(forName: "NewYork"))
        
        XCTAssertEqual(city.coordinates.latitude, 40.7128)
        XCTAssertEqual(city.coordinates.longitude, -74.0060)
<strong>  }
</strong></code></pre>



3. distanceInMiles 테스트

```swift
func testParisToNewYork() throws {
        // 1) test를 실행할 때마다 Caculator 객체가 생성되는 것을 방지하기 위해 setUp에서 객체를 생성한다. (13, 16번째 줄)
        
        // 2) 두 도시 사이 거리를 계산한다.
        let distanceInMiles = try calculator.distanceInMiles(from: "Paris", to: "NewYork")
        
        // 3) Equal은 아주 정확한 값을 비교해서, 원하는 테스트를 만들어내지 못함
        // XCTAssertEqual(distanceInMiles, 3636)
        
        // 4) accuracy를 이용해서 해결 - accuracy는 1만큼 오차를 허용한다는 의미
        XCTAssertEqual(distanceInMiles, 3636, accuracy: 1)
 }
```



4. 정확한 에러를 반환하는지 확인하는 테스트

```swift
func testCupertinoNotRecognized() {
        XCTAssertThrowsError(try calculator.distanceInMiles(from: "Cupertino", to: "NewYork")) { error in
            XCTAssertEqual(error as? DistancaCaculator.Error, .unknownCity("Cupertino"))
        }
}
```

###

### UITest

참고: UI Test를 위해서는 보이스오버가 선행되어야합니다!\\

1. 스와이프가 제대로 수행되는지?

```swift
func testMilesToParis() {
        // 1) TabBar가 있다면 (나는 없음) - 탭바로 이동
        // app.tabBars.buttons["Discover"].tap()
        
        // 2) image 스와이프 하는 방법
        // 빈 곳에 break point를 설정하고, test diamond를 실행한다.
        // po app, ui element에 대한 정보를 얻을 수 있다. (다만 image label 설정 필요)
        // po app.images // images에 대한 정보만 얻을 수 잇다
        
        let newYorkImage = app.images["New York Image"]
        newYorkImage.swipeLeft()
        XCTAssert(app.staticTexts["Paris"].isHittable)
}
```



### Code Coverage

* XCTest의 기능 중 하나로, 소스 코드의 행이 실행된 횟수를 측정하여 시각화된 형태로 보여준다.

![](broken-reference)
