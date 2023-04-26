# Build global apps: Localization by example

### internatinalization

<aside>
💡 앱을 전 세계의 기기에서 작동시킬 수 있도록 국제화를 해야한다. (internatinalization)
Localization을 제대로 구현하면, 사용자의 언어와 상관 없이 동일한 사용자 경험을 제공해줄 수 있다.
Apple이 제공하는 API를 사용하면, 많은 부분을 국제화하는데에 유용하다.

</aside>

---

### Apple 기본, 날씨앱

- 사용자들이 날씨 앱을 통해 일기예보를 확인한다.
- 모든 사용자들이 동일한 UI를 경험하고 있다.
    
    ![image](https://user-images.githubusercontent.com/68676844/234508545-466022c6-77ad-474b-b521-bf296d85e4e8.png)

    
- 사용자의 설정 값에 따라 상이한 부분들이 존재한다.
    - 현재 날씨 상태에 대한 설명이 현지화(Localized)되어있다.
    - 숫자들이 각 Format(형식)을 가지고 있다.
    - 언어의 방향에 따라 UI의 방향이 조절된다.
    
    ![image](https://user-images.githubusercontent.com/68676844/234508585-af10c934-a367-46fe-8b39-19ff9e4d433e.png)

    

---

### Translation (번역)

1. **날씨 앱**에서 제공하는 Wind에 대한 설명
    - Wind is making it feel cooler (바람이 더 시원하게 느끼게 합니다)
        
        ![image](https://user-images.githubusercontent.com/68676844/234508641-622add5b-49c9-4cf9-957b-22965df68897.png)

        
    - 이것을 사용자가 선택한 언어로 제대로 번역하기 위해서 String(localized)를 사용해서 문자열을 선언해야한다.
        
        ```swift
        let windPerceptionLabelText = String(
        			localized: "Wind is making it feel cooler",
        				comment: "Explains the wind is lowering the apparent temperature") 
        ```
        
2. Mac의 Mail 앱
    - 특정 메일을 Archive 할 수 있고, 사이드 바에 있는 Archive에서 해당 메일을 확인할 수 있다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234508693-19b45238-1763-44f5-9af4-e13a572cf4bf.png)

        
    
    - (문제) 영어와 다르게, 스페인어에서는 폴더 명과, 동작의 이름이 다르다. (Archivado, Archivar)
        
        ![image](https://user-images.githubusercontent.com/68676844/234508734-2acb56ab-c8c5-43a9-8af8-28ded35c1233.png)

        
        → 영어에서는 동일한 단어일지라도, 다른 언어에서는 문맥에 따라 다른 형태로 표현해야하는 경우가 존재한다.
        
    - (해결) Swift 5.7의 새로운 API를 사용해서 이 문제를 해결할 수 있다.
        
        ```swift
        // Swift 5.7, NEW!!
        
        let filter = String(localized: "Archive.label",
        										**defaultValue: "Archive",**
        										comment: "Name of the Archive folder in the sidebar")
        
        let filter = String(localized: "Archive.menuItem",
        										**defaultValue: "Archive",**
        										comment: "Menu item title for moving the email into the Archive folder")
        ```
        
        영어 문자열에서 사용할 수 있는 default value(기본 값)이 사용된다.
        
        이후 현지화된 문자열의 키를 수정해서 번역자가 구별할 수 있도록 하는 것이다.
        
        → 영어로 앱을 실행하는 동안은 동일한 단어가 표시되고, 스페인 번역기는 다른 단어를 제공할 수 있게 되었다. 즉, Archive라는 단어를 두개의 키로 분리해서 (archive.menuitem, archive.label) 관리하여 각 번역기가 다른 형태를 저장할 수 있도록 설정한다.
        
3. 날씨 앱의 제안 기능 (사용자 현재 위치에 따라 날씨를 보여주는 기능)
    - “Show weather in My Location” 또는 “Show weather in Cupertinu” 이런식으로 제안을 할 수 있다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234508833-8c6d289f-89ca-442f-b007-8da8bff3de42.png)

        
    
    - (경고) String Interpolation을 사용해서 구현하는 방법
        
        ```swift
        // WARNING!!
        
        let locationName = "Cupertino" // 도시
        // or
        let locationName = "My Location" // 단순 대명사
        
        let showWeatherUserActivityTitle = String(
        		localized: "Show weather in \(locationName)",
        		comment: "Title for a user activity to show weather at a specific city or \"my location\""
        ```
        
        ```swift
        // 독일어 - 쿠퍼티노 날씨 보여줘
        "Wetter **in** Cupertino anzeigen" // Ok
        
        // 독일어 - 현재 위치의 날씨를 보여줘
        "Wetter in meinum Standort anzeigen" // NO
        "Wetter **an** meinum Standort anzeigen" // OK
        ```
        

- (해결) 두 가지 문자열을 사용해야한다.
    
    ```swift
    String(localized: "Show weather in **\(locationName)**",
    				comment: "Title for a user activity to show weather at a specific city")
    
    String(localized: "Show weather in **My Location**"
    				comment: "Title for a user activity to show weather at the user's current location")
    ```
    
    번역기가 언어에 대해서 적절한 문법을 쓸 수 있도록 분리해줘야한다.
    
    즉, 문장에 변수가 들어가게 된다면 문장 전체에 영향을 주게 된다. (문자열의 결합은 대문자의 설정이나 문법에 영향을 주게 된다)
    
    이 문제를 해결하기 위해서는 반드시 해당 언어를 사용하는 사람들이 테스트를 해주어야한다.
    
    따라서 문법이 다르다면 두 개의 String을 선언해서 문제를 해결하자.
    

- Comments
    - 문자열에 대한 적절한 해설을 넣어 번역가들에게 정보를 제공하는 것이다. (의도를 동일하게 유지하기 위해서) / 번역가는 런타임에 텍스트를 확인하지 못할 수도 있기 때문에 comment에 정보를 담아 앱에서 어떤 역할을 하는지에 대한 이해를 공유해줘야한다.
    - 반드시 포함되어야하는 내용
        
        **What interface element** : 문자열이 표시되는 인터페이스 요소가 무엇인가? “Title”
        
        **What context** : UI 요소 및 화면에 표시되는 위치 (섹션 헤더, context menu, user activity 등) “user Activity”
        
        **What each variable is** : 문자열에 변수가 포함되어있는 경우, 문법을 일치시켜야하기 때문에, 런타임에 해당 값을 설명해주어야한다.
        
    
    ```swift
    String(localized: "Show weather in \(locationName)",
    				comment: "Title for a user activity to show weather at a specific city")ㅉㄷ
    ```
    
1. 날씨 앱의 컨텐츠를 현지화하는 기능 (Localized remote content)
    - 날씨 앱은 날씨를 제어하는 것이 아니라, 서버에서 데이터를 다운로드 받는다.
    - 전 세계의 모든 날씨 정보를 가지고 있고, 해당 컨텐츠를 사용자가 설정한 언어로 표시해줘야한다.
    - 즉, 컨텐츠가 사용자 기기에서 다운로드 되면, 사용자가 설정한 언어로 바꾸어서 보여주어야한다는 의미이다.

- 구현 방법
    - 서버로부터 지원되는 언어의 목록을 요청한다.
        
        언어 ID 배열로 구성되어있으며, iPhone에 저장된 사용자 언어와 비슷한 언어를 추출한다.
        
        ```swift
        // 언어의 ID 배열
        let allServerLanguages = ["bg", "de", "en", "es", "kk", "uk"]
        let language = Bundle.preferredLocalizations(from: allServerLanguages).first
        ```
        
        Apple Framework의 `Bundle.preferredLocalizations` 를 사용하면, 사용자 선호 언어 배열을 받아올 수 있으며, 제일 첫번째 것이 가장 잘 맞는 언어이다.
        
    - 해당 언어를 기반으로 서버에 후속 요청을 수행한다.
        
        사용자가 이해할 수 있는 언어로 응답을 생성하기 위해서 사용하는 것이다.
        
    
    즉, 원격 컨텐츠를 표시할 때는 사용 가능한 언어를 다운로드하고, 사용자의 기본 설정과 일치시킨 후 컨텐츠를 로드하여 해당 결과를 활용해야한다.
    

- 날씨 앱에서의 실제 활용 방법
    
    강수량 확인 컨텐츠에서 6시간 동안 0mm 라고 제공되고 있다. 
    
    6시간이기 때문에 복수형을 사용해야한다.
    
    ![image](https://user-images.githubusercontent.com/68676844/234508934-b2e80d3a-55c4-491f-ae5d-5e5e0913f242.png)

    
    ```swift
    String(localized: "\(amountOfRain) in last \(numberOfHours) hour",
    		comment: "Label showing how much rain has fallen in the last number of hours")
    ```
    
    - 영어와 우크라이나어의 복수형
        
        ![image](https://user-images.githubusercontent.com/68676844/234508997-1b5a606e-564d-4c9c-9d93-0ae69085c43d.png)

        
        이 논리를 코드에서 구현하고 싶지 않기 때문에 Apple의 Framework를 활용한다. (복수 규칙을 인코딩하는 String dictㅍ
        
    - 복수형을 구현하기 위한 방법 (코드에서는 구현하고 싶지 않아)
        - String dicts 사용
        - Automatic Grammar Agreement 사용
            
            ```swift
            String(localized: "\(amoutOfRain) in last **^[\(numberOfHours) hour](inflect: true)**",
            		comment: "Label showing how much rain has fallen in the last number of hours")  
            ```
            
    - (주의) 문장에 숫자가 없다면, 복수 규칙을 사용하지 않는다.
        
        ```swift
        // 여긴 숫자가 없으니까 굳이 복수형을 쓰지 않아도 된다는 것
        
        if selectedCount == 1 {
        	return String(localized: "Remove this city from your favorites")
        } else {
        	return String(localized: "Remove these cities from your favorites")
        }
        ```
        
        ```swift
        // 문장에 숫자가 들어갈 때, 복수에 대한 변형을 갖춘다.
        
        String(localized: "\(amountOfRain) in last ^[\(numberOfHours) hour](inflect: true).",
        		comment: "Label showing how much rain has fallen in the last number of hours")
        ```
        

### Formatter

- 문장에 단위가 있는 경우, Formatter 사용을 고려해야한다.

1. 날씨 앱의 습도를 보여주는 기능
    
    ![image](https://user-images.githubusercontent.com/68676844/234509055-3d983463-b175-4e88-b567-82a6798a9230.png)

    
    - SwiftUI  및 Swift에서는 코드 한 줄로 구현이 가능하다.
        
        ```swift
        let humidity = 54
        
        // SwiftUI View
        Text(humidity, format: .percent)
        
        // Swift
        humidity.formatted(.percent)
        ```
        
        값을 Text로 감싸고, 형식을 지정해준다.
        
        → Formatter가 %의 위치, 공백 문자, 사용자 설정에 따른 숫자 체계를 알아서 지정해준다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509129-4857f2ed-27f3-47c3-b894-fb1701a2991c.png)

        

1. Combine a Formatter with Text
    - 앞으로 24시간 동안 50mm가 예측된다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509183-c34c0692-5ff7-4db0-93e4-b317a5571800.png)

        
    - 영어에서는 굉장히 단순하지만, 스페인어에서는 단수 복수에 따라 다르게 보여주어야 한다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509236-4ac1827b-64ff-459b-ba53-116448d444db.png)

        
    
    - (해결) Formatter와 복수형 규칙을 결합해서 문제를 해결할 수 있다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509273-82ee210f-ab77-48e3-821a-94e8c7a71d65.png)

        
        1단계 : 함수의 선언
        
        ```swift
        func expectedPrecipitationIn24Hours(for valueInMillimeters: Measurement<UnitLength>) -> String {
        	
        	// 시스템에 UnitLength를 요청한다. 
        	// 사용자 설정을 암호화하고, 강우를 보여주는 것을 요청한다. 
        	let preferredUnit = UnitLength(forLocale: .current, usafe: .rainfall)
        
        	// 사용자가 미터법을 사용하도록 설정하지 않았다면, 측정 타입을 선호화는 단위로 변환한다. 
        	let valueInPreferredSystem = valueInMillimeters.converted(to: preferredUnit)
        
        	// API 구성은 우리가 그 값에 대한 형식화된 문자열을 코드 한 줄로 생성할 수 있게 해 준다.
        	let formattedValue = valueInPreferredSystem.formatted(.measurement(width: .narrow, usage: .asProvided))
        
        	// 비가 1mm나 1인치를 초과해서 내린다면, 복수형을 써야한다. 값을 정수로 변환해서 확인할 수 있게 한다. 
        	let intergerValue = Int(valueInPreferredSystem.value.rounded())
        
        	// localized 문자열을 주어진 키로 로딩하고 default value도 설정한다. default value에서 String interpolation을 써서, integerValue와 형식화된 값과, 숫자 24를 포함시킨다. 올바른 숫자 체계가 자동으로 쓰이게 된다. 키는 stringdict에 선언되게 된다.
        	return String(localized: "EXPECTED_RAINFALL"
        								defaultValue: "\(integerValue) \(formatterValue) expected in next \(24)h.",
        								comment: "Label - How much precipitation (2nd formatted value, in mm or Inches) is expected in the next 24hours (3rd, always 24).")
        ```
        
        2단계 : String dict 생성
        
        ![image](https://user-images.githubusercontent.com/68676844/234509372-bdd7b3a2-70e6-473b-85d4-02cdd23050fe.png)

        
        우리가 코드에서 방금 썼던 키로 시작한다. 
        
        ![image](https://user-images.githubusercontent.com/68676844/234509451-39658d3a-6c83-48ab-9b71-85fc6bf93c37.png)

        
        영어에서 문자열을 복수로 바꿀 필요가 없기 때문에 other  범주로 쓴다. 
        
        첫 파라미터는 런타임에 어떤 Category가 선택되는지 정의한다. (우리는 정수값)
        
        둘째, 셋째 파라미터 숫자는 형식화된 문자열에 있다. 런타임에 그 문장이 어떤 모습일지 정의한다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509587-bc2b2f04-8e42-4240-82b5-de39509a3174.png)

        
        스페인어도 구조가 같지만 다른점은 단수와 복수 모두에 대해 번역을 제공하고 있따. 
        
        그래서 결과적으로 영어와 스페인어에서 올바른 문법을 제공할 수 있게  되었다.
        
        ![image](https://user-images.githubusercontent.com/68676844/234509627-b85add09-b108-4376-9641-ad8b359f51c2.png)

        

### Swift Packages

- 문자열들이 의존 상태에 있거나, 앱 모듈 내부에 있다.
    
    또는 Swift Package를 사용해서 다른 개발자에게 배포할 수 있다. (NEW FEATURE!)
    
    1. Swift Package 정의를 위한 구조 선언 및 설정 구축
        
        ```swift
        let package = Package(
        		name: "FoodTruckKit",
        		
        		// 주 언어 설정
        		defaultLocalization: "en",
        
        		products: [
        			.library(
        				name: "FoodTruckKit",
        				targets: ["FoodTruckkit"]),
        		],
        )
        ```
        
    
    1. 문자열 추출
        
        모든 문자열을 추출하고, `xcloc` 파일(번역기로 전송되는 파일)에 들어간다. 
        
        Xcode - Produce - Exprot Localizations - FoodTruckKit Export
        
        ![image](https://user-images.githubusercontent.com/68676844/234509689-6749dc37-a484-4461-8931-fc563c1b64b0.png)

        

1. (현지화된 컨텐츠를 Package로 불러오기)
    
    알아서 올바른 경로에 Localized 된 파일이 삽입 된다. (흐름은 App Localization과 동일함)
    
    Xcode - Product - import Localizations
    
    ![image](https://user-images.githubusercontent.com/68676844/234509733-913cc1e0-e6df-4bf0-ad0b-edc8520149f1.png)

    

1. Swift Package의 문자열 로딩
    
    ```swift
    let title = String(localized: "Wind",
    									**bundle: .module,**
    									comment: "Title for section that shows data about wind")
    ```
    
    Bundle 인자를 명시해야한다.
    

### Layout and SwiftUI

1. 사용자 언어에 따른 UI 변경사항
    
    언어에 따라 상응하는 언어보다 길거나 짧고, 항상 앱의 레이아웃에 영향을 미치게 된다.
    
    ![image](https://user-images.githubusercontent.com/68676844/234509791-a8a44737-c9e4-46ea-93b7-48143019d583.png)

    
- 영어와 힌디어를 보게 되면, Label의 높이에 차이가 존재한다.
    
    UI 요소에 대해서 고정된 높이를 지정하면 안된다. (상황에 따라 텍스트의 높이가 달라질 수 있음을 예측하자)
    

    ![image](https://user-images.githubusercontent.com/68676844/234509875-220ea5a0-2fb8-421b-9739-ba22a3b6c1e7.png)

    

1. 일주일 날씨를 보여주는 기능
    
    제일 긴 글자 (Today)를 기준으로 각 요소들이 수직으로 정렬되고 있다.
    
    영어와 그리스어는 Today가 가장 길지만, 스페인어는 폭이 모두 동일하다. (Label은 항상 유연해야한다.)
    
    ![image](https://user-images.githubusercontent.com/68676844/234509921-bcb3a53c-7270-4e56-a625-27466d042f62.png)

    
    - SwiftUI의 NEW FEATURE `**Grid**`
        
        ![image](https://user-images.githubusercontent.com/68676844/234509966-9b870aea-8718-40fb-8a21-4074f4c232ca.png)

        
        ```swift
        var rows: [Row]
        var body: some View {
        	Grid(alignment: .leading) { // 화면 좌측부터 UI 요소가 시작됨. (우에서 좌로 시작되는 언어에서는 우측부터 시작)
        		ForEach(rows) { row in
        			GridRow { // 각 수평 그룹에 대해 GridRow를 추가한다.
        				// content 선언
        				Text(row.dayOfWeek)
        
        				Image(systemName: row.weathercondition)
        					.symbolRenderingMode(.multicololr)
        
        				// 가장 유연한 요소, 레이아웃 크기가 좁아지면, 캡슐의 크기가 알아서 조절됨
        				Capsule().fill(Color.orange).frame(height: 4)
        				
        				Text(row.maximumTemperature)
        			}
        			.foregroundColor(.white)
        		}
        	}
        }
        
        ```
        
    - 크기 조절, 위치 조절 등을 SwiftUI에서 알아서 자동으로 작업한다.
    
2. 제한된 공간에서 긴 텍스트를 처리하는 방법 (With Apple Watch)
    - 독일어로 번역된 텍스트가 너무 길어서, 한 줄로 표현하기 어려운 상황
        
        아이콘을 제거하는 방법은 권장되지 않고, **텍스트를 여러 줄로 표현**하는 것이 적절
        
        ![image](https://user-images.githubusercontent.com/68676844/234510003-3a351df8-9457-46d9-a739-69f292617e39.png)

        
    
    - Mail 앱에서 또 다른 방법
        
        버튼 제목의 길이가 너무 길어서, 기존 레이아웃을 사용하기 힘든 경우
        
        ![image](https://user-images.githubusercontent.com/68676844/234510053-d8b70020-bc4f-473e-954d-8838e4eff2a4.png)

        
        - (해결) NEW FEATURE `**ViewThatFits**`
            
            공간에 제약이 있어서 뷰를 배치할 수 없을 때, 대안을 제공하는 방법
            
            ```swift
            var body: some View {
            	**ViewThatFits** {
            		horizontalConfiguration()
            		stackedConfiguration()
            	}
            }
            ```
            
            → Localized 뿐만 아니라, 사용자가 텍스트 크기를 조절했을 때, 혹은 다양한 기기에서도 유용하게 작동한다.
            
            ![image](https://user-images.githubusercontent.com/68676844/234510109-7e5c1186-8c04-4cf4-bba1-5b2fa17f953b.png)

            
    
    ### 요약
    
    1. 앱은 영어로만 이루어져 있지 않다.
        
        해외 사용자들, 테스터들의 피드백을 경청하여 반영하자.
        
    2. Formatter를 사용해라.
        
        Swift에서 Formatting하는 방법은 매우 쉽다.
        
        알아서 잘 설정해주니 잘 써라…
        
    3. Localized your Swift Package
        
        Swift Package 제공자로서, 고객들에게 Localized된 경험을 제공해라. (엄청난 영향을 줄 것이다.)
        
    4. Make your layout flexible
        
        레이아웃은 번역된 문자에 대해 유연한 레이아웃을 제공해야한다.
        
        인터페이스 요소를 숨기는 것은 적절하지 않으므로, 이번에 제공된거 잘 사용해서 유연한 레이아웃을 제공해라.
