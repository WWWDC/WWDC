# What's new in Screen Time API

# 🔵 지난 iOS15 릴리즈 주요 사항

## 🔹 Family Controls

- Screen Time API에 대한 엑세스 권한을 부여하는 역할. 본질적 `게이트웨이`
- 자녀 보호 앱 제거 & 우회 방지
- 가족이 사용하는 앱을 위한 개인 정보 보호 토큰

## 🔹 Managed Setting

- 기기에 지속적인 제한 설정 기능
- 웹 콘텐츠 필터링 제공
- 맞춤형 UI로 앱&웹 보호

## 🔹 Device Activity

- 디바이스 활동 일정의 시작 및 종료 시 코드 실행
- 사용 한계치 도달 시 코드 실행

# 🔵 iOS16 데모 앱 : Worklog

특정 수치들이 충족될 때까지 특정 앱의 사용을 제한하는 기능

## 🔹 `NEW` Family Controls

`iOS15` iCloud 승인을 통해서만 자녀의 기기를 승인할 수 있었음

→ `iOS16` 자신의 기기에서 개별 사용자를 승인할 수 있게 되었음

- 개별 사용자가 스스로 인증
- 여러 개의 개별 인증
- 개별 인증에 대한 앱 제거 또는 iCloud 로그아웃 제한 없음

```swift
import FamilyControls

let center = AuthorizationCenter.shared

Task {
	do {
		try await center.requestAuthorization(for: .individual)
	} catch {
		print(error)
	}
}
```

기존 코드보다 훨씬 간결해졌다.

## 🔹 Managed Settings Store

현재 사용자나 기기에 설정을 적용하는 데이터 저장소

`iOS15` 프로세스당 하나의 ManagedSettingsStore 만 허용됨, 또한 앱과 DeviceActivity의 확장 프로그램은 별도로 ManagedSettingsStore가 필요해서 DeviceActivity에 대한 응답에서 설정을 변경하기 어려웠음

→ `iOS16` 프로세스마다 고유한 이름을 가진 ManagedSettingsStore를 최대 50개 만들 수 있음. 이렇게 명명된 ManagedSettingsStore들(Named stores)은 앱과 모든 앱 확장프로그램에서 자동으로 공유됨. 모든 Named stores에서 한번에 설정 제거도 가능해짐

```swift
import ManagedSettings

extension ManagedSettingsStore.Name {
    static let youTube = Self("YouTube")
}

final class BlockingManager {
    func managedSettingStoreSetup() {
        let youTubeCategory = ActivityCategoryToken() // 이거 안됨
        let youTubeStore = ManagedSettingsStore(named: .youTube)
        youTubeStore.shield.applicationCategories = .specific(except: [youTubeCategory])
        youTubeStore.shield.webDomainCategories = .specific(except: [youTubeCategory])
    }
}
```

영상에선 `ActivityCategoryToken`에 대한 사용 방법을 알려주지 않아서 따로 찾아봤다.

[How to use categories in Managed S… | Apple Developer Forums](https://developer.apple.com/forums/thread/722944)

[ActivityCategoryToken | Apple Developer Documentation](https://developer.apple.com/documentation/managedsettings/activitycategorytoken)

[FamilyActivitySelection | Apple Developer Documentation](https://developer.apple.com/documentation/FamilyControls/FamilyActivitySelection)

[FamilyActivityPicker | Apple Developer Documentation](https://developer.apple.com/documentation/familycontrols/familyactivitypicker)

여기까지 보고 이전에 [작성했던 예제](https://velog.io/@debby_/WWDC-Screen-Time-API#-2-shield-discouraged-apps)를 살펴봤다.

**FamilyActivityPicker**** 에서 유저가 선택한 앱들의 정보를 **FamilyActivitySelection**에 담아 model에 전달한다.

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

[Is it possible to use familly acti… | Apple Developer Forums](https://developer.apple.com/forums/thread/691168)

문제는 SwiftUI 코드라서 `UIHostingController`를 사용해야한다.

[Use SwiftUI in UIKit View Controllers with UIHostingController](https://medium.com/@max.codes/use-swiftui-in-uikit-view-controllers-with-uihostingcontroller-8fe68dfc523b)

[UIHostingController in SwiftUI 2020 (Use in UIViewController) - How To Bridge UIKit with SwiftUI.](https://www.youtube.com/watch?v=z_9EOGDw5uk&t=2s)

아무튼 나머지 코드를 먼저 살펴보자면

```swift
import SwiftUI
import DeviceActivity

extension DeviceActivityName {
		static let activity = Self("activity")
}

let deviceActivityCenter = DeviceActivityCenter()

Button("Allow for Evening") {
                try? deviceActivityCenter.startMonitoring(
                    .activity,
                    during: DeviceActivitySchedule(
                        intervalStart: DateComponents(hour: 18),
                        intervalEnd: DateComponents(hour: 20),
                        repeats: true)
                )
            }
```

```swift
import DeviceActivity
import ManagedSettings

final class BlockingMonitor: DeviceActivityMonitor {
    
    // let database = BarkDataBase() //TODO: Make database for Token of the published category
    override func intervalDidStart(for activity: DeviceActivityName) {
        super.intervalDidStart(for: activity)
        let youTubeStore = ManagedSettingsStore(named: .youTube)
        youTubeStore.clearAllSettings() //youTube 제약이 시작되면 이전의 제약을 모두 clear한다.
    }
    override func intervalDidEnd(for activity: DeviceActivityName) {
        super.intervalDidEnd(for: activity)
        
        let appStore = ManagedSettingsStore(named: .youTube)
//        let appCategory = database.appCategoryToken //TODO: Make database for save&load `youTubeCategoryToken`
        youTubeStore.shield.applicationCategories = .specific([youTubeStore])
        youTubeStore.shield.webDomainCategories = .specific([youTubeStore])
    }
}
```

## 🔹 DeviceActivity

현재 사용자나 기기에 설정을 적용하는 데이터 저장소

`iOS15` 앱과 웹 사이트에 대한 사용량 허용치와 사용 시간대에 반응했음

→ `iOS16` SwiftUI를 사용한 사용자 지정 사용량 보고서를 만들 수 있게 됐음. 사용량 데이터는 새로운 확장 포인트를 통해 사용자에게 표시될 데이터와 화면에 표현될 방식을 사용자 정의 할 수 있음. 이를 `DeviceActivityReport`는 최종 사용자의 개인정보 보호를 보장함

이걸 이용해서 보고서를 개인화하여 뷰에 표시할 수 있고, 원하는 차트타입 뷰를 구현해서 보여주면 된다는데 끝까지 다 보고 적용시켜보려고 SwiftUI 프로젝트 예제도 해봤는데, 코드를 너무 일부만 보여주고 샘플 코드조차 공개를 해놓지 않아서 도대체 어떻게 사용하라는 건지 모르겠다!

나머지는 공식문서를 보고 직접 이해하고 적용해보는 수 밖에 없는 듯 하다.
