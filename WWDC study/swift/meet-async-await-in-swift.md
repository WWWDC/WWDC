---
description: async/await 알아보기
---

# Meet async/await in Swift
---
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
 
##### Ex) 아이템 리스트가 있고 각 행에 사진의 썸네일을 보여줌(서버에 저장된)
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

##### 여기 코드에는 문제가 있음
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

##### Async/Await로 해보기
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


--> 더 추가할 예정 
## Conclusion
