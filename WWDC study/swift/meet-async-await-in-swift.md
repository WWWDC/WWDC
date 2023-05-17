---
description: async/await 알아보기
---

# Meet async/await in Swift

## Introduction
비동기 프로그래밍은 많은 사람들에게 일상적인 활동이다. 
하지만 장황하고, 복잡하고 부정확하게 사용함
async/await는 그걸 도와줌

## Body
SDK에는 awaitable한 메소드가 많이 있음
예를들어, UIImage
```swift
//UIImage
func prparingThumbnail(of size: CGSize) -> UIImage?
func prpareThumbnail(of size CGSize, completionHandler: @escaping (UIImage?) -> Void)
```

동기 - preparingThumbnail 
동기 함수는 함수가 끝나기를 기다리는 동안 thread를 막음 → 즉, 끝날때까지 thread는 아무것도 할수 없음

비동기 - prepareThumbnail
비동기 함수는 함수가 실행하는동안 thread는 자유 → 즉, 실행하는동안 thread는 다른 작업을 할수있음 
작업이 끝나면 completionHandler로 call 함
비동기 작업이 끝나면 알려주는 방법은 completionHandler, delegate call backs가 있음
 
### Ex) 아이템 리스트가 있고 각 행에 사진의 썸네일을 보여줌(서버에 저장된)
String을 UIImage로 변경하는 방법은 일련의 단계를 거침
1.thumbnailURLRequest로 String → URLRequest
2.URLSession’s dataTask로 request → data
3.UIImage로 data → UIImage
4.prepareThumbnail로 원본 이미지를 썸네일로 render
→ 몇 개의 operations은 이전 작업 결과에 의존함, 그래서 순서가 있음

작업들 중 thumbnailURLRequest와 UIImage 값을 빠르게 반환 그래서 동기작업으로 해도됨
하지만 dataTask와 prepareThumbnail 작업은 시간이 많이 소요 
→ 즉, 순서대로 진행해도 시간이 걸리는 작업이 있기 때문에 비동기 함수를 사용해야함

예를들어, completionHandler
```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    //dataTask(with:completion:)
    //UIImage(data:)
    //prepareThumbnail(of:completionHandler:)
}
```
fetchThumbnail함수 아규먼트는 String, completionHandler
completionHandler는 output을 호출자에게 반환

fetchThumbnail함수가 먼저 실행
그 후, thumbnailURLRequest함수 실행 동기함수여서 completionHandler 없음
동기 → 다른 작업 막음

```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
     //UIImage(data:)
    //prepareThumbnail(of:completionHandler:)
    }
   task.resume()
}
```

URLSession’s dataTask 함수는 비동기여서 completionHandler가 있음
비동기 → 실행되는동안 다른 작업을 할 수 있음 
dataTask 작업 시작하려면 resume() 시작

완료되면 completionHandler가 data, response, error를 처리함

1. error처리
```swift
if let error = error {
    completion(nil, error)
} else if (response as? HTTPURLResponse)?.statusCode != 200 {
    completion(nil, FetchError.badID)
} else {
    //UIImage(data:)
    //prepareThumbnail(of:completionHandler:)
}
```
2. image data 처리 → 동기 
```swift
guard let image = UIImage(data: data!) else {
    return
}
//prepareThumbnail(of:completionHandler:)
```
3. 이미지가 생성되면 썸네일 로드 → completion(thumbnail, nil)
성공하면 thumbnail, 실패하면 nil
```swift
image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumnail in
    guard let thumnail = thumnail else {
      return
    }
 completion(thumnail, nil)
}
```
작업이 진행되는동안 thread는 다른 작업 수행 가능

### 여기 코드에는 문제가 있음
guard else return 구문때문에 completion쓰는 걸 잊음 
→ FetchThumbnail의 호출자는 FetchThumbnail이 작업을 완료하면 실패하더라도 알림을 받음
→ 데이터에서 UIImage를 만들거나 Thumbnail 준비하는 데 실패하면 fetchThumbnail의 호출자에게 알림이 표시되지 않고 행이 업데이트 안됨, spinner만 계속 돌아감
```swift
func fetchThumbnail(for id: String, completion: @escaping (UIImage?, Error?) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(nil, error)
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(nil, FetchError.badID)
        } else {
            guard let image = UIImage(data: data!) else {
                completion(nil, FetchError.badImage)
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumnail in
                guard let thumnail = thumnail else {
                    completion(nil, FetchError.badImage)
                    return
                }
                completion(thumnail, nil)
            }
    }
   task.resume()
}
```

누락된 부분 추가해줌
Swift는 함수를 통해 실행이 어떻게 진행되든 값이 반환되지 않으면 오류가 발생
하지만 여기서는 Swift’s usual error handling mechanism 을 사용할 수 없음
→ Swift가 우리의 작업을 확인할 수 없다는 뜻
→ 컴파일때 에러 확인을 못하고, 런타임때 에러를 확인 
fetchThumbnails는 클로져이기 때문에  항상 호출되는지 확인하고 싶지만 Swift에서는 강제로 호출할 수 있는 방법이 없음
그래서 guard else return에서 error 확인을 못함
→ 쨋든 2개 동기, 2개 비동기를 순차적으로 진행하게 만들었으나 올바르게쓰기 어렵고, 의미모호하고, 따라하기 어려움

```swift
func fetchThumbnail(for id: String, completion: @escaping (Result<UIImage?, Error?>) -> Void) {
    let request = thumbnailURLRequest(for: id)
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        if let error = error {
            completion(.failure(error))
        } else if (response as? HTTPURLResponse)?.statusCode != 200 {
            completion(.failure(FetchError.badID))
        } else {
            guard let image = UIImage(data: data!) else {
                completion(.failure(FetchError.badImage))
                return
            }
            image.prepareThumbnail(of: CGSize(width: 40, height: 40)) { thumnail in
                guard let thumnail = thumnail else {
                    completion(.failure(FetchError.badImage))
                    return
                }
                completion(.success(thumnail))
            }
    }
   task.resume()
}
```
→ 안전한 방법인 Result 타입을 넣어서 수정 하지만 코드가 못생기고 길어짐 

### Async/Await로 해보기
```swift
func fetchThumnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)
    // data(for:)
    //UIImage(data:)
    //thumbnail
}
```
- 아규먼트는 String
- async 키워드
- thtrows 에러처리
- return는 UIImage
로 간단! 

thumbnailURLResquest는 동기함수 
```swift
func fetchThumnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode ==200 else { throw FetchError.badID }
    //UIImage(data:)
    //thumbnail
}
```
URLSettion.shard.data는 datatask와 같음 Foundation에서 제공하고 비동기임
다른점은 datatask awaitable함 
throws때문에 try작성, async때문에 await 작성  
error가 발생하면 fetchThumbnail에서 에러를 던짐 

```swift
func fetchThumnail(for id: String) async throws -> UIImage {
    let request = thumbnailURLRequest(for: id)
    let (data, response) = try await URLSession.shared.data(for: request)
    guard (response as? HTTPURLResponse)?.statusCode ==200 else { throw FetchError.badID }
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
}
```
UIImage(data:) 는 동기 
thumbnail 실행 중 error면 throw 

-> 직선코드 
-> 항상 에러처리 가능  → 안전함 

```swift
    let maybeImage = UIImage(data: data)
    guard let thumbnail = await maybeImage?.thumbnail else { throw FetchError.badImage }
    return thumbnail
```
함수가 아니여도 비동기임 → await 키워드 필요 
프로퍼티, 이니셜라이즈도 비동기면 await 작성   

```swift
    extension UIImage {
        var thumbnail: UIImage? {
            get async {
                let size = CGSize(width: 40, height: 40)
                return await self.byPreparingThumbnail(ofSize: size)
            }
        }
    }
```
thumbnail 프로퍼티는 sdk아니라 작성한거
get에 async를 작성
비동기는 get only만 가능 → set 불가능

```swift
    await works in for loops
```

for문에도 await 가능 

await 키워드는 비동기 함수가 일시 중단 될 수 있다는 것을 나타냄 

### 비동기 함수가 일시 중단된다는 것은 무엇을 의미하는지?
fetchThumbnail에서 thumbnailURLRequest 호출
![image](https://github.com/WWWDC/WWDC/assets/67883020/bf1299a1-e272-4d23-b4e1-d5ceb9d9a5ba)
- 함수가 실행 중인 스레드를 해당 함수로 직접 제어
- thumbnailURLRequest와 같은 일반적인 함수(동기함수)는 스레드가 완료될 때까지 작업을 수행할 때 까지 완전히 점유함
- 함수가 끝나면 값을 리턴하거나 에러를 던짐
- 다시 제어 권한을 돌려줌
→ 일반적인 함수가 스레드 제어를 포기하는 유일한 방법

### async function는 스레드 제어를 포기할 수 있음 → 일시중단함
![image](https://github.com/WWWDC/WWDC/assets/67883020/4618aec4-7800-43b3-8635-85f09896aae9)
- 일반함수와 동일하게 함수를 호출하면 스레드 제어 가능
- 실행되면 비동기 기능을 일시 중단될 수 있음 → 스레드 제어 포기
- 함수에 스레드 제어를 주는게 아니라 스레드에 대한 제어권을 시스템에 제공
- 그러면 함수도 일시 중단함
- 일시 중단이란 "당신이 해야 할 일이 많다는 것을 알고 있습니다. 무엇이 가장 중요한지는 당신이 결정합니다.”를 의미
- 기능이 일시 중단되면 시스템은 스레드를 사용하여 다른 작업을 수행할 수 있음
- 어느 시점에서 시스템은 가장 중요한 작업이 이전에 일시 중단되었던 비동기 함수를 계속 실행하는 것이라고 결정함 
→ 이 시점에 시스템은 비동기 함수를 다시 시작함(resume)
- 그럼 비동기 함수는 다시 스레드를 제어하고 작업할 수 있음 또한 원한다면 스스로 중단도 가능, 필요한 만큼 스스로 중단 가능

async 키워드가 있다고(비동기라고) 무조건 일시중단 하는 건 아님
await 라고 함수가 거기서 일시중단 하는 것도 아님

### fetchThumbnail 함수로 일시중단 시 어떤 일이 일어나는지 다시 확인
![image](https://github.com/WWWDC/WWDC/assets/67883020/032e10e2-51a9-4132-aa93-9e88313fa5de)
- fetchThumbnail이 URLsession의 비동기 데이터 메서드를 호출하면 데이터 메서드는 일시 중단을 통해 비동기 함수만 수행할 수 있는 특수한 방법으로 스레드에서 실행을 중지
- 스레드 제어를 시스템에 제공, URLSession data method에 작업을 예약하도록 시스템에 요청
- 그러나 이 시점에서는 시스템이 제어 상태이므로 작업이 즉시 시작되지 않을 수 있음
만약 fetchThumbnail이 호출된 후 사용자가 버튼을 눌러 일부 데이터를 업로드한다고 한다면 (ex 게시물에 대한 반응) 
시스템은 이전에 대기 중인 작업을 하기전에 사용자의 응답을 게시하는 작업을 자유롭게 실행할 수 있음

![image](https://github.com/WWWDC/WWDC/assets/67883020/76e841b2-aa9a-44d5-8aaa-6598973e8fe3)
- 작업이 완료되면 URLSession data method를 다시 시작하거나 시스템이 다른 작업을 대신 실행할 수 있음
- URLSession data method가 끝나면 다시 fetchThumbnail로 돌아감

## Conclusion
### Swift가 비동기 호출을 await 키워드로 표시해야 한다고 주장하는 이유는?

→ 기능이 일시 중단된 상태에서 다른 작업을 수행할 수 있고 즉, 기능이 일시 중단 되면 앱 상태가 크게 변경될 수 있다는 점때문 
completion handlers 마찬가지임

그러나 async/await은 모든 형식과 들여쓰기가 없기 때문에 
await키워드는 코드 블록이 하나의 트랜잭션으로 실행되지 않음을 인식하는 방법

### async/await에 기억해야할 중요한 사항
1. 함수를 async으로 표시하면 일시 중단이 허용
함수가 자체적으로 일시 중단되면 호출자도 일시 중단 → 호출자도 비동기
2. 비동기 함수에서 한 번 또는 여러 번 일시 중단할 수 있는 위치를 가리키기 위해 await 키워드를 사용
3. 비동기 기능이 일시 중단되는 동안 스레드는 차단되지 않음 시스템은 다른 작업을 자유롭게 예약할 수 있고 나중에 시작되는 일도 먼저 실행할 수 있음
→ 즉, 일시 중단되는 동안 앱의 상태가 크게 변경될 수 있음을 의미
4. 비동기 함수가 다시 시작되면 
호출한 비동기 함수에서 반환된 결과가 원래 함수로 되돌아 옴, 실행은 중단되었던 바로 그 자리에서 계속됨
