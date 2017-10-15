---
layout: post
title:  "Effective phone number verification"
date:   2017-09-25 0:10:00
description: Effective phone number verification
tag:
toc: true
---

# Effective phone number verification
[원문][source] 
                
전화 번호를 사용하는 앱을 만들려면 사용자가 전화 번호를 소유하고 있는지 확인하는 것이 중요합니다. 
이렇게하는 것은 UX의 관점에서 볼 때 까다롭습니다. 
다른 로케일에서 전화 번호 형식을 이해하는 것 뿐만 아니라 모든 사용자의 SMS를 읽을 수 있는 기능과 같이 방해가 되지 않는 장치 메커니즘을 제공하거나 사용자가 침입하는 장치 사용 권한을 사용하지 않아도 됩니다

Google Play 서비스에는 사용자의 전화 번호를 얻고 기기 권한없이 SMS를 통해 사용자를 확인하는 데 도움이되는 두 가지 새로운 API가 있습니다.                 
1. Phone Selector
2. SMS Retriever

> 참고 : 시작하기 전에 SMS를 수신하고 Google Play 서비스 10.2.x 이상을 실행할 수있는 전화 번호가있는 기기를 만들고 테스트해야합니다.
  
  [source]: https://android-developers.googleblog.com/2017/10/effective-phone-number-verification.html
  
  