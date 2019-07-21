# iOS 네트워킹 통신하기

✅ 앞서 다운받은 프로젝트 파일에서 **iOS 폴더**로 이동하면 제가 만들어놓은 iOS 프로젝트가 있습니다.

![ios_path](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/Ios_path.png)

​	  다운받지 않으신 분들은 지금 다운받아주세요!

​	 ‼️ [iOS 프로젝트 다운받기](https://github.com/kyeahen/ExpressionRekognitionMusicService/archive/master.zip)

✅ 시간 관계 상 **네트워킹 위주**로 진행하도록 하겠습니다.

​	  View 부분이 궁금하신 분들은 세미나가 끝난 후, 따로 질문해주시면 감사하겠습니다.

<br/>

---------

<br/>

### ❗️1단계 : 프로젝트 세팅하기

* 터미널 상에서 **iOS > ExpressionRekognitionMusicService**로 이동해주세요.

  이동한 후, 아래 명령어를 실행해주세요.

  라이브러리를 설치하는 과정입니다.

```
$ pod install
```

<br/>

* 파인더로 이동하면 아래와 같이 파일들이 추가되어 있습니다.

  **ExpressionRekognitionMusicService.xcworkspace** 파일을 열어주세요.

![iOS-2](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/iOS-2.png)


<br/>


### ❗️2단계 : 통신 파일 작성하기

* **APIService.swift**
  - API 주소를 편리하게 사용하기 위한 파일입니다.

```swift
/* protocol은 인터페이스입니다. 최소한으로 가져야 할 속성이나 메서드를 정의합니다.
구현은 하지 않습니다. 진짜로 정의만 합니다.*/
protocol APIService {
    
}

/* Extension은 해당 클래스를 확장하는 기능을 갖습니다. 클래스 뿐만이 아니라 다양한 요소들(구조체, 열거형 타입)을 확장시킬 수 있습니다.
 예를 들어, UIButton, UIView와 같은 View 요소들도 확장시킬 수 있고, Int, Double과 같은 타입들도 확장시킬 수 있습니다.
 확장 시킨다는 의미는 해당 요소들에 기능을 추가할 수 있다는 의미입니다.
 여기서는 APIService라는 프로토콜에 기능을 추가하였습니다.
 */

extension APIService {
    
    /* 네트워킹 통신을 할 때, 보통 하나의 서버 URL을 가지고 경로만 바뀌게 됩니다.
    네트워킹 통신 메소드마다 하나하나 주소를 넣는 번거로움을 덜기 위해 또는 주소가 바뀌었을 때, 한번에 변경할 수 있도록
    아래의 메소드를 정의하여 조금더 편리하게 통신할 수 있도록 합니다.
     */
extension APIService {
    static func url(_ path: String) -> String {
        return "http:// EC2퍼블릭 DNS or IPv4 퍼블릭 IP을 넣어주세요 :3000/" + path
    }
}
```

<br/>

* **ImageUploadService.swift** 
  - 사진을 업로드하면 표정을 분석하여 가장 높은 표정 값을 반환받는 네트워크 통신입니다.

```swift
import UIKit
import Alamofire
import SwiftyJSON

//해당 구조체에 프로토콜을 구현하였습니다.
struct ImageUploadService: APIService {
    
    //MARK: 표정 인식 API
    static func postImage(image: UIImage, completion: @escaping (_ result: String) -> Void) {
        
        //앞서 APIService에 정의했던 메소드를 사용하여 경로만 추가합니다.
        let URL = url("api/rekognition")

        let imageData = image.jpegData(compressionQuality: 0.3)
        
        //MultipartFormData 통신
        Alamofire.upload(multipartFormData: { (multipartFormData) in
            
            multipartFormData.append(imageData!, withName: "image", fileName: "photo.jpg", mimeType: "image/jpeg")
            
        }, to: URL, method: .post, headers: nil) { (encodingResult) in
            
            switch encodingResult {
            case .success(request: let upload, streamingFromDisk: _, streamFileURL: _) :
                
                upload.responseData(completionHandler: {(res) in
                    switch res.result {
                        
                    //서버 연결 성공
                    case .success :
                        if let value = res.result.value {
                            
                            //JSON 값 중 emotion에 해당하는 Value 값을 가져오는 것입니다.
                            let emotion = JSON(value)["emotion"].string
                            
                            completion(emotion ?? "")
                        }
                        
                    case .failure(let err):
                        print(err.localizedDescription)
                    }
                })
                
                break
                
            //서버 연결 실패
            case .failure(let err):
                print(err.localizedDescription)
            }
        }
    }
    
    
}



```

  <br/>

* **Music.swift**

  - 서버에서 받는 JSON 값에 맞는 모델을 정의해놓은 파일입니다.

```swift
/* Swift4부터 추가된 프로토콜입니다.
외부표현식 처리를 손쉽게 할 수 있도록 도와줍니다.
저희는 서버에서 JSON 형식으로 반환해주기 때문에 Codable이 JSON 변환 처리를 해주는 것입니다.
서버 측의 Key값과 Type 값을 정확하게 입력해주어야 알맞은 변환이 가능합니다.
 */
struct Music: Codable {
    let id : Int
    let singer : String
    let title : String
    let emotion : String
    let image : String
    let url : String
}

```

 <br/>

* **MusicService.swift** 

  - 표정에 알맞는 음악 추천 리스트를 가져오는 네트워크 통신입니다.

  - 음악 추천 API는 제가 따로 구현해놓았습니다!
  
    제 임의대로 음악 추천을 해놓았으니 양해부탁드립니다☺️

```swift
import Alamofire

struct MusicService {
    
    //MARK: 음악 추천 리스트 API - GET
    static func getMusicRecommandList(emotion: String, completion: @escaping ([Music]) -> Void) {
        
        let URL = "http://ec2-13-125-219-247.ap-northeast-2.compute.amazonaws.com:3000/api/music?emotion=\(emotion)"
        
        Alamofire.request(URL, method: .get, parameters: nil, encoding: JSONEncoding.default, headers: nil).responseData() { res in
            switch res.result {
                
            //서버 연결 성공
            case .success:
                
                if let value = res.result.value {
                    
                    let decoder = JSONDecoder()
                    
                    do {
                        //music에 맞게 디코딩을 합니다.
                        let musicData = try decoder.decode([Music].self, from: value)
                        completion(musicData)
                        
                    } catch {
                        /* catch 문으로 오게되면 제대로 디코딩이 이루어지지 못한 것입니다.
                        보통 codable 실수가 대부분입니다.
                         변수명이 틀렸거나 타입 값이 일치하지 않거나 널처리를 제대로 하지 않은 이유입니다.
                        */
                        print("변수 문제")
                    }
                }
                
                break
             
            //서버 연결 실패
            case .failure(let err):
                
                //에러를 출력해줍니다.
                print(err.localizedDescription)
                break
            }
        }
    }
}

```

  <br/>

### ❗️3단계 : 앱 실행하기

* 자신의 **디바이스**를 연결하거나 **시뮬레이터**를 선택해주세요.

![iOS3](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/iOS3.png)

* **Cmd + R** 또는 아래 사진의 ▶️ 버튼을 누르면 앱이 실행됩니다.

![iOS4](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/iOS4.png)

  <br/>

#### [애뮬레이터로 시연하기]

- 사파리에 들어가서 자신이 원하는 인물 사진을 검색해주세요.

  ![saveimage1](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/saveimage1.png)
  <br/>

- 사진을 길게 눌러서 저장해주세요.

  ![saveimage2](https://github.com/kyeahen/ExpressionRekognitionMusicService/blob/master/Guide/images/saveimage2.png)

  <br/>

### ❗️4단계 :  앱 완성
- 아래 사진을 클릭하시면 시연 영상을 보실 수 있습니다!

  [![Video](http://img.youtube.com/vi/e44lKDPGtyE/0.jpg)](https://youtu.be/e44lKDPGtyE) 


----------

<br/>

## 🚩 다음 목차

- [삭제 가이드]()



