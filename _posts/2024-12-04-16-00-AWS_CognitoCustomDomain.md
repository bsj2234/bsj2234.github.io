---
layout: post
title: "AWS Cognito Custom Domain"
date: 2024-12-04
categories: [Network]
tags: [AWS]
media_subpath: /medias/posts
---

# AWS Cognito Custom Domain

[참고자료](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-add-custom-domain.html)

- 이거 참고하면 쉽게 가능함
- 그냥 유저 권한만 추가해주면 시간만 걸리고 어려울건 없는듯
- Cognitto custom domain에있는 Alias를 복사해 Route53에 CNAME에 붙여넣으면 됨(Alias는 옵션키지말고 붙여넣어야함)
- (문서의 권한은 IAM사용자로 만들때에 해당되는듯..)


## 첨언
 - 사실 처음엔 게이트웨이에서 리디렉트해주는걸 원했음
 - 주소가 깔끔해보이고 추가 record가 필요없는 장점이 있을기 때문
 - 근데 뭣도모르고 GatewayAPI에서 HTTP GET으로 endpoint를 cognito url로 설정해버림
 - 당연히 GET으로 html을 직접가져와 뿌려주는것으로 내가 원하던 결과가 나오지 않음
   - 해결방법으로 Lambda나 Mock을 사용해 리디렉트하는 방법이 있다고함
 - 오히려 람다가 필요하고 비용이 증가할 수 있다고 느껴 기본 지원하는 Cognito Custom Domain 설정을 사용함

