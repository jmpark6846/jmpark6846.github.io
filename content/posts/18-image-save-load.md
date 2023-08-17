---
title: "이미지를 파일로 저장/불러오기"
date: 2023-08-16T21:22:47+09:00
draft: false
summary: " "
tags: ["ios", "image"]
---

로컬에서 이미지를 불러와 앱 내 파일로 저장하고 불러와 보자.
이런 작업을 해줄 역할을 ImageManager라는 역할을 만들자. 실제 사용은 전역 객체를 사용하도록 shared를 추가해주자.
```swift
class ImageManager {
    static let shared = ImageManager()
}
```

파일을 저장하기 위해서는 우선 저장 위치를 나타내는 URL을 가지고 있어야 한다. 
파일이름은 "*.jpeg" 형태로 정한다. 파일이름은 중요하지 않아서 아래와 같이 무작위 스트링을 얻어왔다.

```swift
let filename = "\(UUID().uuidString).jpeg"
```

파일이름을 받아 저장할 위치를 URL 타입으로 리턴하는 메소드를 만들었다.

```swift
func url(filename: String) -> URL?{
    guard let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else { return nil }
    return directory.appendingPathComponent(filename)
}
```

iOS앱은 앱 자신의 파일과 폴더에만 접근할 수 있다. 앱 마다 각자의 다큐먼트 폴더를 갖고 있는데 여기에 앱에서 필요한 파일을 쓰거나 불러올 수 있다.  
다큐먼트 주소를 directory에 저장한 다음 파일이름을 붙여서 원하는 주소를 만들었다.


그럼 이제 이미지를 파일로 저장해보자.
```swift
func saveImage(filename: String, image: UIImage, completion: @escaping (Bool) -> Void ) {
    guard let fileURL = ImageManager.shared.url(filename: filename) else { return }
    
    if FileManager.default.fileExists(atPath: fileURL.path) {
        do{
            try FileManager.default.removeItem(at: fileURL)
        }catch {
            print(error)
            return
        }
    }
    
    guard let data = image.jpegData(compressionQuality: 1) else { return }

    do{
        try data.write(to: fileURL)
        completion(true)
    }catch{
        print(error)
        completion(false)
    }
}
```

파일을 저장하기전에 만약 해당 주소에 이미 파일이 있는 경우 파일을 제거한다. 즉 일종의 덮어쓰기인 셈이다.   
메모리에 jpeg 포맷의 데이터로 만든다음 그걸 해당 주소에 쓰는(write) 방법이다. 생각보다 간단하다.


이번엔 이 파일을 `UIImage`로 불러와 보자.

```swift
func loadImage(filename: String) -> UIImage? {
    guard let url = ImageManager.shared.url(filename: filename) else { return nil }
    return UIImage(contentsOfFile: url.path)
}
```

마찬가지로 주소를 얻어오는데, 이 주소를 UIImage에 넘겨주기만 하면 알아서 데이터를 읽어온다.  

위에서 봤던 삭제 기능도 따로 메소드로 만들어주었다.
```swift
func deleteImage(filename: String) {
    guard let url = ImageManager.shared.url(filename: filename) else { return }

    do{
        try FileManager.default.removeItem(at: url)
    }catch {
        print(error)
    }
}
```
아래는 최종 코드이다. 이전 글에서 썸네일을 만드는데 사용했던 이미지 리사이징 메소드도 추가해줬다.



```swift
class ImageManager {
    static let shared = ImageManager()
    static let originalSize = CGSize(width: UIScreen.main.bounds.width * UIScreen.main.scale, height: UIScreen.main.bounds.height * UIScreen.main.scale)
    
    func url(filename: String) -> URL?{
        guard let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first else { return nil }
        
        return directory.appendingPathComponent(filename)
    }
    
    func resizedImage(at url: URL, for size: CGSize) -> UIImage? {
        let options: [CFString: Any] = [
            kCGImageSourceCreateThumbnailFromImageIfAbsent: true,
            kCGImageSourceCreateThumbnailWithTransform: true,
            kCGImageSourceShouldCacheImmediately: true,
            kCGImageSourceThumbnailMaxPixelSize: max(size.width, size.height)
        ]
        
        guard let imageSource = CGImageSourceCreateWithURL(url as NSURL, nil),
              let image = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary)
        else {
            return nil
        }
        
        return UIImage(cgImage: image)
    }
    
    func saveImage(filename: String, image: UIImage, completion: @escaping (Bool) -> Void ) {
        guard let fileURL = ImageManager.shared.url(filename: filename) else { return }
        
        if FileManager.default.fileExists(atPath: fileURL.path) {
            do{
                try FileManager.default.removeItem(at: fileURL)
            }catch {
                print(error)
                return
            }
        }
        
        guard let data = image.jpegData(compressionQuality: 1) else { return }

        do{
            try data.write(to: fileURL)
            completion(true)
        }catch{
            print(error)
            completion(false)
        }
    }
    
    func loadImage(filename: String) -> UIImage? {
        guard let url = ImageManager.shared.url(filename: filename) else { return nil }
        return UIImage(contentsOfFile: url.path)
    }
    
    func loadImage(url: URL) -> UIImage? {
        return UIImage(contentsOfFile: url.path)
    }

    func deleteImage(filename: String) {
        guard let url = ImageManager.shared.url(filename: filename) else { return }

        do{
            try FileManager.default.removeItem(at: url)
        }catch {
            print(error)
        }
    }
}
```