---
layout: default
title: 앱 및 엔진 구성 참조
pagename: toolkit-apps-and-engines-config-ref
lang: ko
---

# 앱 및 엔진 구성 참조

이 문서에서는 {% include product %} Pipeline Toolkit의 앱, 엔진 및 프레임워크에 대한 구성을 만들 때 포함할 수 있는 다양한 모든 옵션을 개략적으로 설명합니다. 이러한 옵션은 앱의 고급 구성을 수행할 때 유용할 수 있으며, 개발 중에 매개변수를 앱 구성 매니페스트에 추가해야 할 때 중요합니다.

_이 문서에서는 툴킷 구성에 대한 제어 권한이 있는 경우에만 사용할 수 있는 기능에 대해 설명합니다. 자세한 내용은 [{% include product %} 통합 관리자 안내서](https://support.shotgunsoftware.com/hc/ko/articles/115000067493)를 참조하십시오._

# 소개

이 문서에서는 Sgtk에서 구성과 설정을 위해 사용하는 다양한 파일 형식에 대한 사양을 설명합니다. 이 문서는 사용 가능한 다양한 옵션과 매개변수에 대해 간략히 설명하는 참조 문서입니다. 구성을 관리하는 방법에 대한 모범 사례는 다음 문서를 참조하십시오.

[구성 관리 모범 사례](https://support.shotgunsoftware.com/hc/ko/articles/219033168)

# {% include product %} Pipeline Toolkit 환경

툴킷에는 세 가지 주요 구성요소가 있습니다.

- _엔진_은 호스트 응용프로그램(예: Maya 또는 Nuke)과 Sgtk 앱 간에 전환 계층 또는 어댑터를 제공합니다. 앱은 일반적으로 Python과 PySide를 사용하고 엔진은 호스트 응용프로그램을 표준화된 방식으로 표시하는 작업을 담당합니다. 예를 들어 호스트 응용프로그램 상위에 PySide가 없으면 추가합니다.
- _앱_은 비즈니스 로직을 제공하며 본질적으로 특정 작업을 수행하는 도구입니다. 앱은 특정 호스트 응용프로그램에서 작업하거나 둘 이상의 호스트 응용프로그램에서 실행되도록 설계할 수 있습니다.
- _프레임워크_는 엔진, 앱 또는 기타 프레임워크에서 사용할 수 있는 라이브러리입니다. 프레임워크를 사용하면 여러 앱 간에 공유되는 코드 또는 동작을 보다 쉽게 관리할 수 있습니다.

_환경 파일_에는 엔진, 앱 및 프레임워크 컬렉션에 대한 구성 설정이 있습니다. 이러한 컬렉션을 환경이라고 합니다. Sgtk는 파일이나 작업자별로 다른 환경을 시작합니다. 예를 들어 샷 프로덕션용 환경과 리깅용 환경을 구성할 수 있습니다. 각 환경은 단일 yaml 파일입니다.

환경 파일은 `/<sgtk_root>/software/shotgun/<project_name>/config/env`에 있습니다.

yaml 파일의 기본 형식은 다음과 같습니다.

```yaml
    engines:
        tk-maya:
            location
            engine settings

            apps:
                tk-maya-publish:
                    location
                    app settings

                tk-maya-revolver:
                    location
                    app settings

        tk-nuke:
            location
            engine settings

            apps:
                tk-nuke-setframerange:
                    location
                    app settings

                tk-nuke-nukepub:
                    location
                    app settings

    frameworks:
        tk-framework-tools:
            location
            framework settings
```

각 앱과 엔진은 설정을 통해 구성할 수 있습니다. 이러한 설정은 앱/엔진이 매니페스트 파일 `info.yml`에 표시하는 설정 목록과 일치합니다. Sgtk Core의 `v0.18.x`에서는 매니페스트 파일에 지정된 기본값과 다를 경우에만 설정을 지정해야 합니다. 매니페스트 파일 외에도 일반적으로 툴킷 App Store 내의 앱/엔진 페이지에서 구성 가능한 설정을 찾을 수 있습니다.

각 앱, 엔진 및 프레임워크는 각 항목에 대해 정의할 수 있는 다양한 설정 외에도 코드가 있는 위치를 정의해야 합니다. 이 작업은 특수 `location` 매개변수를 사용하여 수행됩니다.

## 코드 위치

환경 파일에 정의된 각 앱, 엔진 또는 프레임워크에는 실행할 앱 버전과 다운로드 위치를 정의하는 `location` 매개변수가 있습니다. 대부분의 경우 `tank updates` 및 `tank install` 명령에 의해 자동으로 처리됩니다. 그러나 구성을 직접 편집하는 경우 툴킷을 배포하고 구조화할 수 있도록 하는 다양한 옵션을 사용할 수 있습니다.

툴킷은 현재 다음 위치 _디스크립터_를 사용한 앱 설치 및 관리를 지원합니다.

- **app_store** 디스크립터는 툴킷 App Store의 항목을 나타냅니다.
- **{% include product %}** 디스크립터는 {% include product %}에 저장된 항목을 나타냅니다.
- **git** 디스크립터는 git 리포지토리의 태그를 나타냅니다.
- **git_branch** 디스크립터는 git 분기의 커밋을 나타냅니다.
- **path** 디스크립터는 디스크상의 위치를 나타냅니다.
- **dev** 디스크립터는 개발자 샌드박스를 나타냅니다.
- **manual** 디스크립터는 커스텀 배포 및 롤아웃에 사용됩니다.

다양한 디스크립터 사용 방법에 대한 자세한 정보는 [툴킷 참조 문서](http://developer.shotgunsoftware.com/tk-core/descriptor.html#descriptor-types)를 참조하십시오.

## 앱 및 엔진 비활성화

때로는 앱이나 엔진을 일시적으로 비활성화하는 것이 유용할 수 있습니다. 이 작업을 수행할 때는 앱 또는 엔진을 로드해야 하는 위치를 지정하는 위치 사전에 `disabled: true` 매개변수를 추가하는 것이 좋습니다. 이 구문은 서로 다른 위치 유형 모두에서 지원됩니다. 예를 들어 다음과 같이 표시될 수 있습니다.

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "disabled": true}
```

또는 특정 플랫폼에서만 앱을 실행하려면 특별한 `deny_platforms` 설정을 사용하여 지정할 수 있습니다.

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "deny_platforms": [windows, linux]}
```

_deny_platforms_ 매개변수의 가능한 값은 `windows`, `linux` 및 `mac`입니다.

## 설정 및 매개변수

각 앱, 엔진 또는 프레임워크는 구성 파일에서 재정의할 수 있는 여러 가지 설정을 명시적으로 정의합니다. 이러한 설정은 문자열, 정수, 목록 등에 명확히 입력됩니다. 자세한 내용은 [툴킷 참조 문서](http://developer.shotgunsoftware.com/tk-core/platform.html#configuration-and-info-yml-manifest)를 참조하십시오.
