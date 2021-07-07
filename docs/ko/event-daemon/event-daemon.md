---
layout: default
title: 이벤트 구동 트리거 작성
pagename: event-daemon
lang: ko
---


# {% include product %} 이벤트 프레임워크
이 소프트웨어는 [Rodeo Fx](http://rodeofx.com) 및 Oblique의 지원을 받아 [Patrick Boucher](http://www.patrickboucher.com)에서 처음 개발했습니다. 이제 [{% include product %} 소프트웨어](http://www.shotgunsoftware.com) [오픈 소스 이니셔티브](https://github.com/shotgunsoftware)의 일부가 되었습니다.

이 소프트웨어는 라이선스 파일 또는 [오픈 소스 이니셔티브](http://www.opensource.org/licenses/mit-license.php) 웹 사이트에서 찾을 수 있는 MIT 라이선스로 제공됩니다.


## 개요

{% include product %} 이벤트 스트림에 액세스하려면 이벤트 테이블을 모니터링하고 새로운 이벤트를 파악하여 처리하는 작업을 반복하는 것이 좋습니다.

많은 항목이 성공적으로 작동하려면 이 프로세스를 거쳐야 하며, 비즈니스 규칙과 직접적인 관련이 없는 항목은 적용해야 하는지 여부를 결정해야 합니다.

프레임워크의 역할은 비즈니스 로직 구현자를 대신해 따분한 모니터링 작업을 처리해 주는 것입니다.

이 프레임워크는 서버에서 실행되면서 {% include product %} 이벤트 스트림을 모니터링하는 데몬 프로세스입니다. 이벤트가 발견되면 데몬은 이벤트를 일련의 등록된 플러그인으로 전달합니다. 각각의 플러그인은 원하는 대로 이벤트를 처리할 수 있습니다.

데몬은 다음을 처리합니다.

- 지정된 하나 이상의 경로에서 플러그인 등록
- 충돌하는 플러그인을 모두 비활성화
- 플러그인이 디스크에서 변경된 경우 다시 로드
- {% include product %} 이벤트 스트림 모니터링
- 마지막으로 처리된 이벤트 ID와 백로그 기억
- 데몬 시작 시 마지막으로 처리된 이벤트 ID부터 시작
- 연결 오류 확인
- 필요에 따라 stdout, 파일 또는 이메일에 정보 로깅
- 콜백에서 사용되는 {% include product %}에 대한 연결 설정
- 등록된 콜백으로 이벤트 전달

플러그인은 다음을 처리합니다.

- 콜백의 번호를 프레임워크에 등록
- 프레임워크에서 이벤트를 제공하는 경우 단일 이벤트 처리


## 프레임워크의 이점

- 스크립트당이 아니라 모든 스크립트에 대해 하나의 모니터링 메커니즘만 처리합니다.
- 네트워크 및 데이터베이스 로드를 최소화합니다(단일 모니터만으로 여러 이벤트 처리 플러그인에 이벤트 공급).
