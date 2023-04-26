# What's new in WKWebView

## 1. Web을 띄울 수 있는 방법

### SFSafariViewController

- 앱에서 브라우저와 똑같은 사용경험을 누리고 싶을 때 사용한다.
- 다만, 깊은 커스텀이 필요하지 않을 때! (단순 브라우저의 출력이 필요한 상황)

### ~~UIWebView~~

- deprecated 될 예정으로, 앞으로는 WKWebView로 사용해야한다.

### WKWebView

- UIWebView보다 빠르고, 반응성이 좋다.
- 웹 컨텐츠와 인터렉션해야하는 앱에서 사용하는 API
    - CSS 기반 UI 작성 가능
    - JavaScript 작성 가능
    - App bound domain을 사용해서 웹 컨텐츠와 상호작용이 가능하다.

## 2. WKWebView의 4가지 변화

### Web content interaction

1. Full screen의 지원
    - 브라우저에서 Java script를 이용하면, 플래시 게임이나 영상을 전체화면으로 볼 수 있다.
    - 이제는 앱으로도 해당 구현이 가능하다.
        
        ```swift
        **webView.configuration.preferences.isElementFullscreenEnabled = true**
        
        webView.loadingHTMLString("""
        <script>
        		button.addEventListener('click', () => {
        			canvas.webkitRequestFullscreen()
        		}, false);
        </script>
        """, baseURL: nil)
        
        let observation = webView.observe(\.fullscreenstate.options:[.new]) {object, change in
        	print("fullscreenState: \(object.fullscreenState)")
        ```
        
2. CSS viewport unit
    - css는 동적 viewport의 크기에 따라 웹 컨텐츠의 레이아웃을 조절해준다.
    - css unit 종류
        - svh, lvh, dvh, svw, lvw, dvw, vi, svi, lvi, dvi, vb, svb, dvb, svmin, lvmin, dvmin, svmax, lvmax, dvmax
        
        웹 개발자들은 해당 unit을 이용해서 최소, 최대 동적 viewport의 크기를 기초로, 레이아웃을 조절할 수 있다. 
        
        - svg, lvh, dvh는 viewport의 다양한 크기를 측정하는 유용한 유닛을 제공한다.
            
           ![image](https://user-images.githubusercontent.com/68676844/234506725-15a90672-6e37-4607-9db6-eb48782fa6f1.png)

        사파리에서 페이지를 열면, 웹 페이지 호스트와 하단 버튼이 보인다.
        화면을 스크롤하면, 버튼이 밑으로 사라지면서 view port의 크기가 커진다.
            
        
    - Swift에서 적용 방법
        
        ```swift
        // CSS viewport unit range inputs
        
        let minimum = UIEdgeInsets(top: 0, left: 0, bottom: 30, right: 0)
        let maximum = UIEdgeInsets(top: 0, left: 0, bottom: 200, right: 0)
        webView.setMinimumViewportInset(minimum, maximumViewportInset: maximum)
        ```
        
3. Find interactions
    - 상호작용 검색기능의 도입
    - WKWebView는 텍스트 출력창이 많기 때문에, 사용자는 주로 특정 텍스트를 검색하는 행위를 수행한다.
        
        (find Interection의 UIFindInterection 객체에 액세스하여 검색 패널을 표시하거나 숨기고 이전 혹은 다음 결과로 이동할 수도 있다. )
        
        ```swift
        // 검색 기능을 사용 여부
        webView.findInteractionEnabled = true
        
        // text 검색 UI 및 Command+F 등의 단축키 사용을 위한 코드
        if let interaction = webView.findInteraction {
        	interaction.presentFindNavigator(showingReplace: false)
        }
        ```
        
        ![image](https://user-images.githubusercontent.com/68676844/234507032-ff9194c0-5e97-4631-af5a-87971e55a307.png)

        

### Content blocking

- WKContentRuleList에 Content blocking이라는 기능이 추가되었다. (Safari 컨텐츠 차단기를 위해 사용하는 API)
- 상황 (ifame에 위키피디아가 포함된 상황) → 위키디피아의 이미지만 block 시킬 수 있음
    
    ![image](https://user-images.githubusercontent.com/68676844/234507073-51db61cb-52c7-4fb1-8ead-f16e6998b0fe.png)
    
    ```swift
    let json = """
    [{
    		"action": {"type": "block"},
    		"trigger": {
    				"resource-type": ["image"],
    				"url-filter": ".*",
    				"if-frame-url": ["https?://([^/]*\\\\.)wikidepia.org/"]
    		}
    }]
    """
    
    // 정규식과 일치하는 프레임의 요청에만 적용된다.
    WKContentRuleListStore.default().compileContentRuleList(forIdentifier: "example_blocker", encodedContentRulList: json) {list, error in
    	guard let list = list else { return }
    	let configuration = WKWebViewConfiguration()
    	configuration.userContentControllerdelegate
    ```
    
    ![image](https://user-images.githubusercontent.com/68676844/234507131-501426b0-4465-46ce-9e1b-dab22b09fb9d.png)

    

### Encrypted media

- iPad OS 16.0에서의 새로운 기능
- 암호화 미디어 확장 프로그램, 미디어 소스 확장 프로그램 APi를 사용하는 컨텐츠가 있을 때 사용할 수 있다.
    - Apple TV+와 같은 프리미엄 컨텐츠도, 기존 MacOS처럼 잘 구동된다.
- 타사 브라우저 WebInspector 활성화 절차도 Safari 활성화 단계와 동일하다.

### Remote WebInspector

- Web inspector에는 웹 컨텐츠 디버깅을 위한 다양한 도구가 존재한다.
    - Dom 확인
    - Java script 실행 디버깅
    - 페이징 로딩 타임라인 확인
- 웹 사이트의 소유자라면, iOS 상 타사 브라우저에서 원격 Web inspector를 실행시켜 직접 감시 및 디버깅도 진행 가능하다.
