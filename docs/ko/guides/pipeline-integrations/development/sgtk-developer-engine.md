---
layout: default
title: 엔진 개발
pagename: sgtk-developer-engine
lang: ko
---

# 자체 엔진 개발

## 소개
이 문서에서는 툴킷 엔진 개발에 관한 몇 가지 기술적인 정보를 간략히 소개합니다.

목차:
- [툴킷 엔진이란?](#what-is-a-toolkit-engine)
- [시작하기 전에 알아야 할 사항](#things-to-know-before-you-start)
- [엔진 통합 방식](#approaches-to-engine-integration)
   - [QT, PyQt/PySide 및 Python이 포함되는 호스트 소프트웨어](#host-software-includes-qt-pyqtpyside-and-python)
   - [QT 및 Python은 포함되지만 PySide/PyQt는 포함되지 않는 호스트 소프트웨어](#host-software-includes-qt-and-python-but-not-pysidepyqt)
   - [Python이 포함되는 호스트 소프트웨어](#host-software-includes-python)
   - [Python이 포함되지 않지만 플러그인을 작성할 수 있는 호스트 소프트웨어](#host-software-does-not-contain-python-but-you-can-write-plugins)
   - [스크립트 작성 기능을 전혀 제공하지 않는 호스트 소프트웨어](#host-software-provides-no-scriptability-at-all)
- [Qt 창 부모-자식 관리](#qt-window-parenting)
- [시작 동작](#startup-behavior)
- [호스트 소프트웨어 위시리스트](#host-software-wish-list)

## 툴킷 엔진이란?
엔진 개발 시, 엔진에 로드되는 다양한 툴킷 앱 및 프레임워크와 호스트 소프트웨어를 효과적으로 연결할 수 있습니다.
엔진을 사용하면 소프트웨어 간의 차이점을 추상화할 수 있기 때문에 Python 및 QT를 사용하여 소프트웨어에 구속받지 않는 방식으로 앱을 작성할 수 있습니다.

엔진은 파일 모음으로 [앱과 구조가 비슷합니다](sgtk-developer-app.md#anatomy-of-the-template-starter-app). 엔진은 `engine.py` 파일을 포함하며 이는 코어 [`Engine` 기본 클래스](https://github.com/shotgunsoftware/tk-core/blob/master/python/tank/platform/engine.py)에서 파생되어야 합니다.
그러면 다양한 엔진이 내부 복잡성에 따라 이 기본 클래스의 여러 요소를 다시 구현합니다.
엔진은 일반적으로 다음과 같은 서비스를 처리 또는 제공합니다.

- 메뉴 관리. 엔진 시작 시 앱이 로드되고 나면 엔진은 해당 {% include product %} 메뉴를 생성하고, 여러 앱을 이 메뉴에 추가합니다.
- 일반적으로 로깅 방식은 소프트웨어의 로그/콘솔에 기록하도록 재정의됩니다.
- UI 대화상자 및 창을 표시하는 방식. 엔진이 QT를 처리하는 방식이 기본적인 기본 클래스 동작과 다를 경우 이 방식은 툴킷 앱 및 기본 호스트 소프트웨어 창 관리 설정에서 시작한 창을 원활하게 통합할 수 있도록 재정의됩니다.
- 앱에 의해 등록된 모든 명령 객체를 포함하는 `commands` 사전(dictionary) 제공. 보통 메뉴 항목이 생성될 때 이 사전에 액세스합니다.
- 기본 클래스는 시작 프로세스의 여러 지점에서 실행되는 다양한 init 및 destroy 방식을 제공합니다. 시작 및 종료 실행을 제어하기 위해 이 방식을 재정의할 수 있습니다.
- 자동 소프트웨어 탐색 및 시작 시 `tk-multi-launchapp`에 의해 호출되는 시작 로직.

엔진은 [`sgtk.platform.start_engine()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.start_engine) 또는 [`sgtk.bootstrap.ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 방식을 사용하여 툴킷 플랫폼에 의해 시작됩니다.
이 명령은 구성 파일을 읽고, 엔진을 시작하고, 모든 앱을 로드하는 등의 역할을 수행합니다.
엔진의 목표는 일단 시작된 후에 앱에 일관된 Python/QT 인터페이스를 제공하는 것입니다.
모든 엔진이 같은 기본 클래스를 구현하므로 앱은 UI 생성 등의 작업을 수행하기 위해 엔진에서 이 방식을 호출할 수 있습니다.
호스트 소프트웨어 내에서 원활하게 작동하도록 이 방식을 구현하는 것은 각 엔진의 몫입니다.

## 시작하기 전에 알아야 할 사항

Shotgun 팀에서는 가장 일반적으로 사용되는 컨텐츠 생성 소프트웨어에 대한 [통합](https://support.shotgunsoftware.com/hc/ko/articles/219039798-Integrations-Apps-and-Engines)을 제공합니다.
[툴킷 커뮤니티 멤버가 빌드하고 공유한](https://support.shotgunsoftware.com/hc/ko/articles/219039828-Community-Shared-Integrations) 엔진도 있습니다. 하지만 툴킷 엔진이 없는 소프트웨어에 대한 파이프라인 통합이 필요한 경우도 있습니다.

시간과 리소스가 있다면 누락된 엔진 중 사용하고 싶은 엔진을 직접 작성하여 툴킷 커뮤니티(그리고 여러분 자신)를 지원해 주시기 바랍니다!

코드 작성을 시작하기 전에 [저희에게 알려 주십시오!](https://knowledge.autodesk.com/ko/contact-support) 약속을 드리기는 어렵지만 귀하의 의견을 제안해 주시면 감사하겠습니다.
관심이 있거나 같은 엔진에서 작업한 경험이 있는 다른 사용자들과 연결해 드릴 수 있을 것입니다.
가능한 경우 툴킷을 통합하려는 소프트웨어의 기술 담당자나 개발자와의 커뮤니케이션 채널을 열어 두면
가능성이나 장애물에 대한 정보를 파악할 수 있어 작업을 진행하는 데 도움이 될 것입니다.
커뮤니케이션 채널을 설정하고 생각하고 있는 기본적인 사항에 대한 이야기를 나눈 후에는 저희 팀을 대화에 초대해 함께 미팅을 진행하면서 엔진의 특정 요소에 대한 대화를 나눌 수 있습니다.
[{% include product %} 커뮤니티 포럼](https://community.shotgunsoftware.com/c/pipeline)에서 툴킷 커뮤니티에 참여할 수도 있습니다.

더욱 발전할 새로운 통합을 기대합니다! 툴킷 커뮤니티에 열성적으로 참여해 주시는 여러분의 노력에 늘 감사드립니다.

{% include info title="팁" content="[자체 앱 개발](sgtk-developer-app.md)에는 앱 개발에 대한 단계별 안내가 포함되어 있으며 이 안내서에서 다루지 않는 엔진 개발에 적용되는 원칙도 포함됩니다." %}

## 엔진 통합 방식

호스트 앱의 기능이 무엇인지에 따라 엔진 개발의 복잡성이 결정됩니다.
이 섹션에서는 엔진 개발 중에 지금까지 발견된 여러 복잡성 수준을 간략하게 살펴봅니다.


### Qt , PyQt/PySide 및 Python이 포함되는 호스트 소프트웨어
이 방식이 최선의 툴킷 설정입니다. 엔진을 Qt, Python 및 PySide를 지원하는 호스트 소프트웨어에 구현하는 작업은 매우 직관적입니다.
[Nuke 엔진](https://github.com/shotgunsoftware/tk-nuke) 또는 [Maya 엔진](https://github.com/shotgunsoftware/tk-maya)이 좋은 예입니다. 통합은 단지 일부 로그 파일 관리를 연결하는 것이며, 코드를 작성하여 {% include product %} 메뉴를 설정합니다.


### Qt 및 Python은 포함되지만 PySide/PyQt는 포함되지 않는 호스트 소프트웨어
이 소프트웨어 클래스에는 [Motionbuilder](https://github.com/shotgunsoftware/tk-motionbuilder) 등이 포함되며 비교적 통합이 간단합니다.
호스트 소프트웨어 자체는 Qt로 작성되고, Python 인터프리터를 포함하고 있기 때문에 PySide 또는 PyQt 버전을 컴파일하고, 엔진과 함께 배포할 수 있습니다.
그러면 이 PySide가 Python 환경에 추가되고, Python을 이용한 Qt 객체 액세스가 가능해집니다.
일반적으로 PySide를 컴파일할 때는 샷 응용프로그램을 컴파일할 때 사용한 정확한 컴파일러 설정을 사용해야 올바로 작동합니다.


### Python이 포함되는 호스트 소프트웨어
이 소프트웨어 클래스에는 타사 통합 [Unreal](https://github.com/ue4plugins/tk-unreal) 등이 포함됩니다.
이 호스트 소프트웨어는 비-Qt UI를 갖고 있지만 Python 인터프리터를 포함합니다.
즉, Python 코드를 환경 내에서 실행할 수 있지만 실행 중인 기존 Qt 이벤트 루프는 없습니다.
이 경우에는 Qt 및 PySide가 엔진에 포함되어야 하며, Qt 메시지 펌프(이벤트) 루프가 UI 내 메인 이벤트 루프와 연결되어야 합니다.
간혹 이 작업을 정확히 수행하기 위한 특별한 방식이 호스트 소프트웨어에 포함되기도 합니다.
포함되어 있지 않으면 정렬을 수행하여 Qt 이벤트 루프가 정기적으로(예: on-idle 호출을 통해) 실행되도록 해야 합니다.


### Python이 포함되지 않지만 플러그인을 작성할 수 있는 호스트 소프트웨어
이 클래스에는 [Photoshop](https://github.com/shotgunsoftware/tk-photoshopcc) 및 [After Effects](https://github.com/shotgunsoftware/tk-aftereffects)가 포함됩니다.
Python 스크립팅이 없지만 C++ 플러그인을 생성할 수 있습니다.
이 경우에는 IPC 레이어를 포함하며, 시작 시 별도 프로세스로 Qt 및 Python을 실행하는 플러그인을 생성하는 것이 일반적인 전략입니다.
보조 프로세스가 실행되고 나면 IPC 레이어를 사용하여 명령이 양방향으로 전송됩니다.
이 호스트 소프트웨어 유형의 경우 적절한 엔진 솔루션을 얻기 위해서는 상당한 작업이 필요합니다.

{% include info title="팁" content="Shotgun 팀에서는 Photoshop 및 After Effects 엔진을 사용하여 실제로 [Adobe 플러그인을 처리하는 프레임워크](https://github.com/shotgunsoftware/tk-framework-adobe)를 만들었습니다.
두 엔진 모두 프레임워크를 사용해 호스트 소프트웨어와 커뮤니케이션하고 나머지 Adobe 제품군에 대한 엔진을 더 쉽게 빌드할 수 있습니다." %}


### 스크립트 작성 기능을 전혀 제공하지 않는 호스트 소프트웨어
호스트 소프트웨어를 프로그래밍 방식으로 액세스할 수 없는 경우에는 엔진을 생성할 수 없습니다.


## Qt 창 부모-자식 관리
보통 창 부모-자식 관리에는 특별한 주의를 기울여야 합니다.
일반적으로 PySide 창은 위젯 계층에 친부모가 없으며, 이는 명시적으로 호출해야 합니다.
창 부모-자식 관리는 일관된 환경을 제공하는 데 중요한 요소이며, 이를 구현하지 않으면 툴킷 앱 창이 메인 창 뒤에 표시되어 혼동을 줄 수 있습니다.

## 시작 동작
엔진은 소프트웨어 시작 방법 및 통합의 시작 방법도 담당합니다.
이 로직은 `tk-multi-launchapp`이 엔진을 사용해 소프트웨어를 시작하려고 할 때 호출됩니다.
이 설정 방법에 대한 자세한 내용은 [코어 설명서](https://developer.shotgunsoftware.com/tk-core/initializing.html?highlight=create_engine_launcher#launching-software)에서 확인할 수 있습니다.

## 호스트 소프트웨어 위시리스트
다음 호스트 소프트웨어 특성을 툴킷 엔진이 활용할 수 있습니다.
많은 특성이 지원될수록 엔진 사용 환경이 향상됩니다!

- 기본 제공 Python 인터프리터, Qt 및 PySide!
- 소프트웨어 시작/초기화 시 코드를 실행할 수 있는 능력.
- 두 시점(소프트웨어가 시작 및 실행되고 있을 때와 UI가 완전히 초기화되었을 때)에 코드에 액세스하여 자동 실행할 수 있는 능력.
- 파일 시스템 상호 작용을 래핑하는 API 명령: 열기, 저장, 다른 이름으로 저장, 참조 추가 등.
- UI 요소를 추가하기 위한 API 명령

   - 커스텀 Qt 위젯을 앱에 패널로 추가(이상적으로는 번들 PySide를 통해 추가)
   - 커스텀 메뉴/상황에 맞는 메뉴 항목 추가
   - 노드 기반 패키지의 커스텀 노드(상호 작용을 위한 커스텀 UI 통합 용이)
   - 선택한 항목/노드 등을 가져오기 위한 인트로스펙션
- 유연한 이벤트 시스템
   - "흥미로운" 이벤트가 커스텀 노드 트리거 가능
- 비동기적 UI 실행 지원
   - 예를 들면, 커스텀 메뉴 항목이 트리거되면 나타나지만 인터페이스를 잠그지 않는 팝업 대화상자
   - 커스텀 UI 창의 부모-자식 관계가 올바로 지정되도록 최상위 창에 핸들 제공