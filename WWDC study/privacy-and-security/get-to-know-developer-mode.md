# Get to know Developer Mode

---
## Introduction
### What is Developer Mode?
- IOS16, WatchOS9에 개발자 워크플로를 지원하는 새로운 모드
- Developer Mode 기본값은 비활성화, 기기를 명시적으로 등록해야 사용할 수 있음
- 등록하면 재부팅이나 시스템 업데이트를 해도 유지됨
- 필요에따라 개발자 모드 설정을 자동화 할수 있는 도구도 있음

## Body
**Why Developer Mode?**
- powerful developer features가 표적형 공격에 악용되고 있음
- 대부분 사용자는 이 기능을 사용안함 → 기본값이 비활성화
- 이렇게 하면 개발 기능을 유지하고 사용자의 보안을 강화할수 있음

**Developer Mode and distribution**
- 대부분 일반적인 배포에는 개발자 모드가 필요하지 않음
    - TestFlight를 통한 응용프로그램 배포
    - Enterprise(In-House) distribution (기업 내부 배포)
    - App Store를 통한 응용프로그램 배포
- Developer Mode는 개발자인 사용자가 기기에서 응용프로그램을 개발중인 경우에만 필요함

### Using Developer Mode
**When to turn on Developer Mode**
1. development 서명된 응용프로그램을 실행 및 설치할때 (개인 팀 포함)
2. 응용프로그램을 디버그 또는 instrument(계측?)때
3. 기기에서 테스트 자동화 사용할때

**How to turn on Developer Mode**
- Xcode에 기기를 연결해야만 Developer Mode ****매뉴가 나타남
- iOS 16 베타 버전은 당분간 메뉴 항상 표시
- Apple Configurator를 사용할때 처럼 Xcode없이 development 서명된 응용프로그램을 설치해도 메뉴 표시
- 설정 → 개인정보 보호 및 보안에서 Developer Mode 제어 가능
- 자동화를 위해 devmodectl 사용할수있음 → 나중에 더 설명

설정 방법</br>
Mac에 기기를 연결 → 
Xcode에서 개발자 모드가 설정되어있지 않음, 프로그램 실행 불가 에러 뜸 → 
설정 개인정보 보호 및 보안 개발자 모드에서 전환함 → 
전환하려면 기기 재부팅 → 
재부팅하면 한번 더 킬지 확인하는 메시지 뜸 → 
키면 바로 사용 가능 → 
Xcode로 다시 실행하면 프로그램 실행 가능
단일 기기일 경우만 가능, 여러 기기를 하려면 시간이 오래걸릴수 있음 → 자동화 도구 개발

### Automation flows
- 한 가지 제한: 암호가 없는 기기만 자동으로 개발자 모드 활성화 가능
why? iphone 재시작 시 기기와 상호작용 하기 전 기기 잠금부터 해제해야하기 때문
- MacOS에서 devmodectl을 사용하려면
    - 이미 연결된 단일 기기에 개발자 모드를 활성화하는데 사용하거나
    - 스트리밍 모드를 통해 사용자가 연결한 모든 기기에서 개발자 모드 자동으로 활성화 가능

**암호 없는 기기고 수동으로 개발자 모드 설정하고 싶지도 않을때 사용하는 방법** </br>
Mac에 기기 연결 (두 대) 
mac 터미널에 `$devmodectl streaming`
연결된 기기 자동 재부팅, 개발자 모드도 활성화 됨
개발자 모드 활성화되면 기기에 알림 옴 → 이제 사용 가능

## Conclusion
- iOS 16, watchOS 9에서는 응용프로그램 배포 및 디버깅 같은 일반적인 개발 작업을 하려면 개발자 모드 활성화가 필요함
- 자동화하고싶으면 devmodectl을 사용하셈
