---
description: >-
  현재 문서는 iOS15 기준 WWDC 영상을 보고 정리한 글이며, iOS16 에서 업데이트 된 내용은 What's new in Screen
  Time API 문서로 정리하겠습니다. Tublock 프로젝트를 진행하면서 ScreenTimeAPI를 사용한 App Blocking 기능을
  구현하기 위해 공부해 보았습니다. 오역 및 오타 피드백 환영합니다. 😁
---

# Meet the Screen Time API

## Meet the Screen Time API

## Screen Time API

> 참고 자료 : [원본 영상](https://developer.apple.com/videos/play/wwdc2021/10123/) / [공식 문서](https://developer.apple.com/documentation/screentimeapidocumentation/)

스크린 타임은 사용자와 가족이 앱과 웹사이트를 **얼마나 자주 사용하는지 추적**하고, **제한을 설정**하여 시간을 관리하고, 가족 구성원과 **사용량을 공유**하여 기기 사용 현황을 파악할 수 있으며, 마지막으로 **자녀가 누구와 소통하는지 등을 관리**할 수 있습니다.

### 📱 사용 환경

* `iOS 15`
* `iPadOS 15`

### ✅ 3가지 기본 원칙

1. 기존 제한에 대한 직접 API 액세스를 위한 최신 on Device 프레임워크 제공
2. 사용자 개인 정보 보호
3. 개발자가 새롭고 환상적인 동적 자녀 보호 환경을 만들 수 있도록 보장하는 것

### 🎞️ 3가지 프레임워크

![](https://velog.velcdn.com/images/debby\_/post/60b016a6-b4be-40a0-9049-4c0b5cabf99a/image.png)

#### ✨ 1) Managed Setting Framework(관리 설정)

![](https://velog.velcdn.com/images/debby\_/post/04a8b335-f680-439c-8550-618114f13f4e/image.png)

앱에서 스크린 타임에서 사용할 수 있는 제한 사항에 직접 액세스할 수 있다.

* 제한 설정할 수 있는 것
  * 계정 잠금
  * 비밀번호 변경 방지
  * 휍 트래픽 필터링
  * 애플리케이션 차단

#### ✨ 2) Family Controls Framework(가족 제어)

![](https://velog.velcdn.com/images/debby\_/post/aad14550-3f86-475f-bedc-659b620c45a3/image.png)

가족 제어는 **트위터의 개인정보 보호정책을 주도**한다. 가족 공유를 활용하여 보호자 승인 없이 스크린 타임 API에 액세스 할 수 없도록 한다. 보호자의 승인을 받은 앱은 보호자 승인 없이 기기에서 제거할 수 없다. 또한 앱과 웹 사이트를 나타내는 _**토큰**_을 제공하여 _**스크린 타임 API 전체에서 사용량을 모니터링하거나 제한**_하는 데 사용된다. (가족 그룹 외부에서는 볼 수 없도록 정보가 모두 보호된다.)

#### ✨ 3) Device Activity(디바이스 활동)

![](https://velog.velcdn.com/images/debby\_/post/4a3d7b2c-98bf-4c9b-a3ea-a193c7d62cdc/image.png)

디바이스 활동은 _**앱을 실행하지 않고도 코드를 실행할 수 있는 기능**_을 제공한다. 앱에 웹 & 앱 사용을 모니터링하고 필요할 때 코드를 실행할 수 있는 새로운 방법을 제공한다. 디바이스 활동 일정과 이벤트

* `기기 활동 일정` : 시작될 때와 끝날 때 애플리케이션의 확장 프로그램을 호출하는 시간 창
* `이벤트` : 유저가 기기 활동 일정의 사용 임계값에 도달할 때 내선 번호를 호출하는 사용량 모니터.

앱은 어떤 유형의 사용량을 언제 모니터링할지 선언하기만 하면 된다.

### 🙄 세 가지 프레임워크를 결합하면?

1. 보호자와 자녀의 기기 모두 앱을 설치
2. 보호자가 자녀 기기에서 앱을 열기
3. 앱은 가족 제어를 통해 승인됨
4. 추후 보호자 기기의 앱에서 설정, 제한 및 규칙을 선택
5. 앱이 해당 정보를 자녀 기기로 보냄
6. 자녀 기기에서 앱이 장치 활동으로 일정 및 이벤트를 생성
7. 일정이 발생하거나 이벤트가 발생하면 앱의 장치 활동 확장이 호출됨
8. 확장 프로그램에서 관리 설정으로 제한 설정

### 📲 Demo App : Homework

![](https://velog.velcdn.com/images/debby\_/post/ab0e8b4f-c891-4584-ae71-cb3b17043e61/image.png)

보호자가 원하는 다른 **`앱의 사용량이 누적될 때까지`** 특정 앱에 대한 자녀의 \*\*`액세스를 제한`\*\*하여 좋은 습관을 장려하는 것이 주 기능이다.

#### ✨ 1. Setup and authorizing Family Controls

가족 제어에 대한 승인을 요청하는 방법: **`프로젝트 설정과 패밀리 컨트롤 승인`**

먼저 [Xcode의 해당 Taget에서 Capability를 추가](https://developer.apple.com/documentation/Xcode/adding-capabilities-to-your-app)한다.

```swift
// APP: 권한 요청하기
import FamilyControls

AuthorizationCenter.shared.requestAuthorization { result in
		switch result {
		case .success():
				...
		case .failure(let error):
				...
		}
}
```

내 앱이 이전에 이 iPhone에서 실행된 적이 없기 때문에 요청 승인은 경고와 함께 보호자의 승인을 요청한다. 허용을 탭하면 계속하려면 보호자에게 Apple ID와 비밀번호로 인증하라는 메시지가 표시된다.

이때 보호자가 인증에 성공하면 요청인증을 호출하면 다시 경고 메시지를 표시하지 않고 자동으로 성공 여부를 반환한다.

오용을 방지하기 위해 로그인한 iCloud가 가족 공유를 사용하는 자녀가 아닌 경우 요청 승인은 실패를 반환한다.

앱에서 스크린 타임 API를 사용할 수 있도록 준비하는 것은 이렇게 쉽다.

가족 제어로 인증하면 앱에 다른 권한도 부여된다.

* 기기가 인증되면 사용자는 더 이상 iCloud에서 로그아웃할 수 없음
* 네트워크 확장 프레임워크로 구축된 `on-device web content filters`는 앱에 포함될 수 있으며 자동으로 설치되고 제거할 수 없음
* 이를 통해 앱에서 기기의 웹 트래픽을 필터링할 수 있음

자녀 보호 앱은 자녀가 앱을 실행하지 않을 가능성이 높은데도 자녀의 기기에서 코드가 실행되도록 하는 것이 어려운 부분이다.

스크린 타임 API의 경우, 기기 활동으로 백그라운드 코드 실행을 수행할 수 있는 새로운 방법을 만들었다.

기기 활동 확장은 나머지 스크린 타임 API와 상호 작용하는 주요 방법이 될 것이다.

#### ✨ 2. Shield discouraged apps

반복되는 일정에 따라 보호자가 선택한 사용 금지 앱을 보호

자녀의 기기에서 앱이 실행되지 않고있어도, 보호자가 금지한 앱이 실행되지 않도록 스케줄을 설정하여 반복적으로 작동하게 하는 기능이다.

이 Extension을 구현하려면 `DeviceActivityMonitor`를 기본 클래스로 서브 클래싱 하여 모니터를 만들어주자.

```swift
// EXTENSION: Create a DeviceActivityMonitor

class MyMonitor: DeviceActivityMonitor {
		override func intervalDidStart(for activity: DeviceActivityName) {
				super.intervalDidStart(for: activity)
		}
		override func intervalDidEnd(for activity: DeviceActivityName) {
				super.intervalDidEnd(for: activity)
		}
}
```

이 함수들은 내 일정이 시작되고 종료된 후 기기를 처음 사용할 때 호출된다.

이제 `DeviceActivityName`과 `DeviceActivitySchedule`을 만들어야 한다.

```swift
// APP: Monitor a DeviceActivitySchedule

import DeviceActivity

// 1. 활동을 참조할 수 있는 이름 생성
extension DeviceActivityName {
		static let daily = Self("daily")
}

// 2. 활동을 모니터링할 시간 범위 설정
let schedule = DeviceActivitySchedule(
		intervalStart: DateComponents(hour: 0, minute: 0),
		intervalEnd: DateComponents(hour: 23, minute: 59),
		repeat: true
)

// 3. center를 통해 방금 생성한 이름과 일정으로 `startMonitoring` 호출
let center = DeviceActivityCenter()
try center.startMonitoring(.daily, during: schedule)
```

위 코드를 통해 일정이 시작되고 끝날 때마다 `MyMonitor`가 활동 이름(`DeviceActivityName`)과 함께 호출된다.

```swift
// APP: Choose the Apps to Discourage

import FamilyControls
import SwiftUI

@StateObject var model = MyModel()
@State var isPresented = false

var body: some View {
		Button("Select Apps to Discourage") {
				isPresented = true
		}
		.familyActivityPicker(isPresented: $isPresented, selection: $model.selectionTpDiscourage)
}
```

FamilyControls 프레임워크에는 작업을 위한 SwiftUI 요소인 `familyActivityPicker` (가족 활동 선택기)가 있다.

기본 앱의 UI에서 이 피커를 표시하고 보호자가 앱 or 웹을 카테고리 목록에서 선택한다.

이 피커의 반환값인 불투명 토큰(?)을 사용하여 제한 설정을 할 수 있다.

버튼을 통해 불러온 피커는 반환값을 모델의 속성에 바인딩한다.(`$model.selectionTpDiscourage`)

```swift
// EXTENSION: Shied the Discouraged Apps

import DeviceActivity
import ManagedSettings // 추가

class MyMonitor: DeviceActivityMonitor {
		let store = ManagedSettingsStore() // 추가

		override func intervalDidStart(for activity: DeviceActivityName) {
				super.intervalDidStart(for: activity)

				let model = MyModel() // 모델 인스턴스
				let applicationos = model.selectionToDiscourage.applications // 선택된 앱 List

				store.shield.applications = applications.isEmpty ? nil : applications // store 등록
		}

		override func intervalDidEnd(for activity: DeviceActivityName) {
				super.intervalDidEnd(for: activity)

				store.shield.applications = nil
		}
}
```

```swift
// APP: Adding a DeviceActivityEvent
import DeviceActivity

extension DeviceActivityName {
		static let daily = Self("daily")
}

extension DeviceActivityEvent.Name { // 이벤트 이름 등록
		static let encouraged = Self("encouraged")
}

let schedule = DeviceActivitySchedule(
		intervalStart: DateComponents(hour: 0, minute: 0),
		intervalEnd: DateComponents(hour: 23, minute: 59),
		repeat: true
)

let model = MyModel()
let events = [DeviceActivityEvent.Name: DeviceActivityEvent] = [
		.encouraged: DeviceActivityEvent( // 이벤트 이름에 맞는 이벤트 생성
					applications: model.selectionToEncourage.applicationTokens
					threshold: DataComponents(minute: minutes)
		)
]

let center = DeviceActivityCenter()
try center.startMonitoring(.daily, during: schedule, events: events) // 이벤트 등록
```

#### ✨ 3. Remove shields for meeting goal

권장 앱 사용 시간을 충분히 누적한 다음 차단을 해제하는 방법

```swift
// EXTENSION: Implement theh Threshold Function
import DeviceActivity
import ManagedSettings

class MyMonitor: DeviceActivityMonitor {
		let store = ManagedSettingStore() // store 인스턴스
		...

		// 허용된 앱의 사용량이 누적되었을 때 호출됨
		override func eventDidReachThreshold(_ event: DeviceActivityEvent.Name, activity: DeviceActivityName) {
				super.eventDidReachThreshold(event, activity: activity)

				store.shield.applications = nil // 해당 이벤트가 충족되면 제한 해제
		}
}
```

#### ✨ 4. Customize the shields

앱의 브랜딩과 기능에 맞게 shields를 커스텀 가능

* Material
* Title
* Icon
* Body
* primaryButton → Ask for Access
* secondaryButton → Open App
* Completion Handler

```swift
// EXTENSION: Create a ShieldConfigurationProvider

import ManagedSettings
import ManagedSettingsUI

class MyShieldConfiguration: ShieldConfigurationProvider {
		override func configuration(for application: Application) -> ShieldConfigutaion {
				return ShieldConfiguration(
						backgroundEffect: ...
						backgroundColor: ...
						icon: ...
						title: ShieldConfiguration.Label(
								text: ...
								color: ...
						),
						subtitle: ShieldConfiguration.Label(
								text: ...
								color: ...
						)
				)
		}
}
```

위 함수를 통해 Block 되었을 때의 화면을 커스텀할 수 있다.

```swift
// EXTENSION: Create a ShieldActionHandler

import ManagedSettings

class MyShieldActionExtension: ShieldActionHandler {
		override func handle(action: ShieldAction,
													 for application: Application,
													 completionHandler:
								@escaping (ShieldActionResponse) -> Void {
				switch action {
				case .primaryButtonPressed:
						completionHandler(.defer)
				case .secondaryButtonPressed:
						completionHandler(.close)
				@unknown default:
						fatalError()
				}
		}
}
```

위 함수를 통해 커스텀 핸들러를 지정할 수 있다.
