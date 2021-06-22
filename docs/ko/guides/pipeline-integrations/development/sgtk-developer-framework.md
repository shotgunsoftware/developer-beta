---
layout: default
title: 프레임워크 개발
pagename: sgtk-developer-framework
lang: ko
---

# 자체 프레임워크 개발

## 소개
이 문서에서는 Toolkit 프레임워크 개발에 관한 몇 가지 기술적인 정보를 간략히 소개합니다.

목차:
- [툴킷 프레임워크란?](#what-is-a-toolkit-framework)
- [사전 작성된 {% include product %} 프레임워크](#pre-made-shotgun-frameworks)
- [프레임워크 작성](#creating-a-framework)
- [후크에서 프레임워크 사용](#using-frameworks-from-hooks)

## 툴킷 프레임워크란?

툴킷 [프레임워크](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)는 툴킷 앱과 매우 유사합니다.
주요 차이점은 프레임워크는 자체적으로 실행할 수 없다는 점입니다.
프레임워크는 앱이나 엔진으로 가져오는 방식으로 사용됩니다. 따라서 여러 엔진과 앱에서 사용할 수 있도록 재사용 가능한 로직을 분리할 수 있습니다.
재사용 가능한 UI 구성요소 라이브러리를 프레임워크의 예로 들 수 있습니다. 여기에는 재생 목록 선택기 구성요소가 포함될 수 있습니다.
그런 다음 앱에서 해당 프레임워크를 가져오고 재생 목록 선택기 구성 요소를 메인 앱 UI에 연결할 수 있습니다.

## 사전 작성된 {% include product %} 프레임워크

{% include product %}는 자체 앱을 제작할 때 유용한 몇 가지 사전 작성 [프레임워크](https://support.shotgunsoftware.com/hc/ko/articles/219039798-Integrations-Apps-and-Engines#frameworks)를 제공합니다.
[Qt Widgets](https://developer.shotgunsoftware.com/tk-framework-qtwidgets/) 및 [{% include product %} 유틸리티](https://developer.shotgunsoftware.com/tk-framework-shotgunutils/) 프레임워크는 앱 개발 시 특히 유용합니다.

## 프레임워크 작성

자체 프레임워크를 작성하는 경우 설정은 앱을 작성하는 것과 거의 동일합니다. ["자체 앱 개발"](sgtk-developer-app.md) 안내서에서 자세한 내용을 확인할 수 있습니다.
프레임워크에는 `app.py` 파일 대신 프레임워크 패키지의 루트에 [`Framework`](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#framework) 기본 클래스에서 파생된 클래스가 포함된 `framework.py`가 있습니다. 또한 프레임워크는 엔진에 명령을 등록하지 않습니다.

대신, 프레임워크 인스턴스 자체에 방식을 직접 저장하거나 `python/` 폴더 내에 모듈을 저장할 수 있습니다.
예를 들어 [shotgunutils 프레임워크는 이를 python 폴더에 저장](https://github.com/shotgunsoftware/tk-framework-shotgunutils/tree/v5.6.2/python)합니다.
여기에 액세스하려면 프레임워크를 가져온 다음 [`import_module()` 방식](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Framework.import_module)을 사용하여 하위 모듈에 액세스합니다.

API 문서에는 [프레임워크를 가져오는 방법](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)에 대한 예가 있습니다.

## 후크에서 프레임워크 사용
후크를 통해 몇 가지 공통 로직을 공유하도록 프레임워크를 생성하는 것이 유용할 수 있습니다.
[`Hook.load_framework()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Hook.load_framework) 방식을 이용하면 앱/프레임워크에서 매니페스트 파일에 명시적으로 요구하지 않는 경우에도 앱 또는 기타 프레임워크 후크에서 프레임워크를 사용할 수 있습니다. 코어 후크에서는 이 방식을 사용하더라도 프레임워크를 사용할 수 없습니다.
