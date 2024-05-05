---
title: "iOS 웹뷰 `<video />` 이슈 대응"
last_modified_at: 2024-05-04
tags:
  - ios
  - webview
  - video
toc: true
toc_sticky: true
---

## 환경

- iOS 17.0.3

## Issue - 1

iOS 웹뷰에서 기존의 잘 작동하던 비디오 인라인 실행이 전체화면 모드로 실행되는 이슈가 발생했다.

### 원인

먼저 모바일 환경에서 비디오를 인라인으로 재생하기 위한 3가지 속성(`autoplay`, `muted`, `playsinline`)은 모두 적용하고 있었다. 그래서 모바일 사파리 브라우저에서는 정상적으로 인라인 실행되는 걸 확인했는데, 문제는 웹뷰에서 전체화면으로 실행되는 것이었다. 기존에 문제없던 코드이기도 하고 웹에서의 문제가 아니라 앱을 새로 배포하면서 문제가 발생한 것으로 추측했다.

### 해결책

웹뷰에서 `playsinline`을 적용하려면 [아래 설정이 필요하다는 걸 확인](https://developer.apple.com/documentation/webkit/wkwebviewconfiguration/1614793-allowsinlinemediaplayback)했고, 앱 개발자 분에게 확인 요청을 드리니 설정이 빠져서 배포가 되었다고 한다.

```tsx
// WKWebView config
allowsInlineMediaPlayback = true;
```

설정 추가 후 테스트했을 때 인라인으로 잘 실행되는 걸 확인했다.

## Issue - 2

iOS 17.0~17.1에서 발생하는 이슈로 iOS 17.2에서 [정상적으로 수정](https://developer.apple.com/forums/thread/740225)되었습니다.
{: .notice}

`playsinline` 속성이 적용된 비디오 태그 위로 터치 이벤트가 작동하지 않아 화면 전체를 채울 경우 스크롤을 할 수 없는 현상

- [CodePen](https://codepen.io/gem0303/pen/qBgEeaG) 이 링크에서 확인할 수 있다.

### 원인

이 문제도 원래 잘 작동하던 코드였는데 갑자기 이슈가 터졌다. 당시에 아이폰 NameDrop 기능이 출시되어서 팀원 전부 iOS 17로 업그레이드 한 상태였는데 이게 문젠가 싶어서 업데이트 하지 않은 다른 기기를 통해 확인해보니 역시나 맞았다.

### 해결책

[WebKit bugs report](https://bugs.webkit.org/show_bug.cgi?id=261950)에 해당 이슈는 수정되어 병합되었다고 하는데 정확히는 WebKit code base에서 수정이 완료 되었고 iOS에는 17.2 버전에 반영되었다. 이하 버전에 대응하기 위해 비디오에 오버레이를 추가하는 방식으로 수정하였고 정상적으로 스크롤이 되는 걸 확인했다.

- as is
  ```html
  <video loop autoplay muted playsinline>
    <source src="source.mp4" type="video/mp4" />
    Sorry, your browser doesn't support embedded videos.
  </video>
  ```
- to be
  ```html
  <div className="relative">
    <video loop autoplay muted playsinline>
      <source src="source.mp4" type="video/mp4" />
      Sorry, your browser doesn't support embedded videos.
    </video>
    <div className="absolute inset-0" />
  </div>
  ```
  <figcaption>react, tailwind 환경</figcaption>

## 참고

- [https://bugs.webkit.org/show_bug.cgi?id=261563](https://bugs.webkit.org/show_bug.cgi?id=261563)
- [https://bugs.webkit.org/show_bug.cgi?id=261950](https://bugs.webkit.org/show_bug.cgi?id=261950)
- [https://developer.apple.com/forums/thread/740225](https://developer.apple.com/forums/thread/740225)
- [https://developer.apple.com/documentation/webkit/wkwebviewconfiguration/1614793-allowsinlinemediaplayback](https://developer.apple.com/documentation/webkit/wkwebviewconfiguration/1614793-allowsinlinemediaplayback)
