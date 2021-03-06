---
title: 빌드/테스트는 내가 해줄게. 너는 코딩에 집중해 (by GitHub Pull Request Builder)
date: 2020-09-07 10:09:56
categories: 
  - tech
tags:
  - Github
  - pullRequest
  - Jenkins
  - ci
  - archives-2020
url : /2020/09/07/github-pullrequest-build/
featuredImagePreview: /images/github-pullrequest-build/programmer-github-jenkins.jpg
images :
  - /images/github-pullrequest-build/programmer-github-jenkins.jpg
---

　git 은 분산 버전 관리 시스템 중 가장 잘 알려져 있다고 해도 과언이 아닐 정도로 대부분의 시스템에서 사용되고 있는 것 같다. 이를 웹서비스에서 보다 편하게 사용할 수 있도록 한 시스템이 Github. <!--more -->Github 을 사용하는 이유 중에 가장 큰 이유를 하나만 이야기해보자면 바로 온라인상에서 코드 리뷰를 할 수 있는 pullRequest라는 기능 때문이 아닐까 조심스럽게 생각을 해본다.

　pullRequest는 work branch에서 작업한 내용을 base branch로 merge 전 꼭 코드 리뷰가 아니더라도 작업한 내용에 대해서 다양한 검사를 자동화할 수 있는 강력한 기능들이 많다. 이러한 자동화는 CI(지속적 통합) 관점에서 매우 중요한데 코드에 대해 체크해야 할 부분들(빌드, 테스트, 정적 분석 등)을 "알아서" 해준다면 작업자는 오롯이 비즈니스 로직 개발에 대해서만 신경 쓸 수 있으니 생산성 절약 측면에서 엄청난 효과를 볼 수 있다.

{{< image src="/images/github-pullrequest-build/car.gif" caption="내가 하는일에만 집중할 수 있게! <br> 출처 : https://www.clien.net/service/board/park/10453442" width="50%" >}}

 이번 포스팅에서는 그중에서도 아주 간단한 설정만으로 work branch의 빌드 상태를 검사해 볼 수 있는 Jenkins의 Github Pull Request Builder를 설치 및 활용해 보고자 한다. 
> 사실 최근 팀에서 CI 서버를 이전해야 했었다. 머릿속에서는 어떻게 하면 되겠지 싶었지만 막상 해보려니 Jenkins 버전업도 되었고 뭐부터 해야 할지 허둥대는 필자가 부끄러웠다. 이참에 정리를 해보며 다시 한번 리마인드 하는 시간을 가져보고자 한다. (이래서 기억보다 기록이 중요하다.)

## 준비물
　전체적인 흐름은 아래 그림처럼 흘러가기 때문에 당연히 서버에 Jenkins 가 설치되어 있어야 한다. Jenkins 설치는 필자의 포스팅([Jenkins 설치 치트키](https://taetaetae.github.io/2018/12/02/jenkins-install/))를 참고해 보는 것도 좋을 것 같다.

{{< image src="/images/github-pullrequest-build/programmer-github-jenkins.jpg" caption="전체적인 흐름" width="50%" >}}

　참고로 필자는 GitHub Enterprise 버전에서 사용했는데 일반 Github에서도 동일한 방법으로 사용 가능하다.

### Github과 Jenkins의 연동을 위한 2가지 설정
　Github 과 Jenkins 가 통신이 되도록 설정해 줘야 한다. 그래야 Github의 코드를 받아서 Jenkins 가 빌드를 하고 그 빌드 결과를 다시 Github에 리포트가 가능해지기 때문이다. 먼저 첫 번째로 ssh 설정으로 Github의 코드를 가져오도록 ssh 설정을 해두자. ssh 설정하는 방법은 필자의 포스팅([Github과 Jenkins 연동하기](https://taetaetae.github.io/2018/02/08/github-with-jenkins/))편을 확인해보면 될 것 같다. 

　그다음으로 아래에서 이야기할 `GitHub Pull Request Builder`라는 Jenkins plugin 이 빌드가 끝난 뒤에 결과를 리포팅 해줄 수 있는 인증 토큰을 발급받아두자. Github > Settings > Developer settings > Personal access tokens 화면에서 키를 생성하고 만들어진 키를 저장해 둔다. (이 키는 보안에 유의해야 하고, 화면 경고(?)에서도 볼 수 있듯이 키는 생성 시 한 번밖에 볼 수 없기 때문에 미리 저장해 둬야 한다.)

{{< image src="/images/github-pullrequest-build/github-access-token.jpg" caption="인증토큰을 미리 받아두자." width="50%" >}}

## Jenkins 설정
　Jenkins > 관리 > pluginManager에 들어가 `GitHub Pull Request Builder`를 검색 후 설치해 준다. 그러고 나서 Jenkins > 관리 > 환경설정에 들어가 보면 아래와 같이 `GitHub Pull Request Builder` 항목이 생긴 것을 확인할 수 있고 위에서 설정한 인증토큰을 아래처럼 등록 후 저장을 한다.

{{< image src="/images/github-pullrequest-build/add-github-access-token.jpg" caption="credentials 을 위에서 발급받은 인증토큰으로 등록해준다." width="50%" >}}

　Jenkins job을 하나 만들고 pullRequest 가 발생했을 때 자동으로 실행될 수 있도록 설정을 해준다. 먼저 General 탭에 `Github project`에 Github url 을 적어주고 

{{< image src="/images/github-pullrequest-build/jenkins-general.jpg" caption="" width="50%" >}}

　소스 코드 관리 탭에서 ssh 주소를 적고 위에서 미리 설정한 ssh 키로 credentials 값을 넣어준다. 전에도 이야기했지만 이 부분에서 오류가 발생하면 빨간색 글씨로 오류 내용이 나오고 아래 화면처럼 오류가 없다면 아무것도 안 나온다. Refspec 에 `+refs/pull/*:refs/remotes/origin/pr/*` 라고 적어주고 브랜치 설정은 파라미터로 받아와서 pullRequest를 발생시킨 브랜치를 빌드 할 수 있도록 `${sha1}` 라고 적어주자.

{{< image src="/images/github-pullrequest-build/jenkins-sourceCode.jpg" caption="" width="50%" >}}

　앞서 이야기했듯이 (제목처럼) pullReqeust를 등록하면 자동으로 빌드가 돌아가게 해야 한다. 그러기 위해서는 이 Jenkins job 을 위에서 설치한 `GitHub Pull Request Builder` 플러그인으로 빌드를 유발해야 한다. 아래 화면처럼 `GitHub Pull Request Builder` 항목을 체크하고 Use github hooks for build triggering에 체크를 해줘서 해당 Github Repository에 webHook 이 등록되게 해주자. 또한 Admin list에서 pullRequest 등록 시 허용할 사용자의 id를 적어준다.

{{< image src="/images/github-pullrequest-build/jenkins-buildTrigger.jpg" caption="" width="50%" >}}

　위에서 등록한 설정으로 인해 해당 Github Repository에 webHook이 자동으로 등록된 것을 확인할 수 있다. 

{{< image src="/images/github-pullrequest-build/jenkins-webhook.jpg" caption="" width="50%" >}}


## 테스트
　자, 그럼 이제 설정을 끝냈으니 실제로 pullRequest를 등록하면 어떤 일이 발생하는지 알아보자. 등록을 하자마자 자동으로 아래처럼 빌드가 돌아가는 것을 볼 수 있고

{{< image src="/images/github-pullrequest-build/pullRequest-build-pending.jpg" caption="" width="50%" >}}

　`Details`를 누르면 해당 Jenkins job으로 이동되게 되는데 열심히 빌드가 되는 걸 볼 수 있다. 시간이 지나고 빌드가 완료 된 다음 pullRequest를 보면 빌드가 성공되었다는 표시를 볼 수 있다.

{{< image src="/images/github-pullrequest-build/pullRequest-successful.jpg" caption="" width="50%" >}}

　빌드가 실패하면 merge를 하지 못하게 하는 옵션도 있다. base branch를 `보호`하는 옵션인데, 아래 화면처럼 빌드 가 실패하면 merge 버튼이 활성화되지 못하도록 해두고 

{{< image src="/images/github-pullrequest-build/github-branch-protection.jpg" caption="" width="50%" >}}

　임의로 빌드를 실패하게 하면 아래처럼 merge버튼이 비활성화되는 걸 볼 수 있다.

{{< image src="/images/github-pullrequest-build/github-build-fail.jpg" caption="" width="50%" >}}

## 마치며
　위에서 설명한 기능을 잘 활용하면 정말 무궁무진하게 활용이 가능하다. 필자가 운영하는 컴포넌트는 이런저런 이유로 빌드가 오래 걸리는데 계속 기다리는 시간조차 유의미한 시간에 사용하고 싶어 빌드가 끝나면 메신저로 빌드 결과를 알려주도록 하였다. 또한 이 빌드 트리거를 잘 활용하면 정적 분석을 한다든지 주요 기능 테스트를 해서 보다 코드의 안정성을 올리고 개발 생산성 또한 올릴 수 있는 좋은 방법이라 생각한다.