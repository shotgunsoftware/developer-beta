---
layout: default
title: 앱 또는 엔진을 어떻게 제거합니까?
pagename: uninstalling-an-app-or-engine
lang: ko
---

# 앱 또는 엔진을 어떻게 제거합니까?

앱 또는 엔진이 더 이상 존재하지 않도록 구성의 환경 YAML 파일을 편집하여 앱 또는 엔진을 제거할 수 있습니다.
환경 파일을 사용하면 앱을 완전히 제거하는 대신 특정 컨텍스트 또는 엔진에서만 사용할 수 있도록 앱을 구성할 수 있습니다.
일반적인 환경 파일 편집에 대한 자세한 내용은 [이 안내서](../../guides/pipeline-integrations/getting-started/editing_app_setting.md)를 참조하십시오.

## 예시

다음은 기본 구성에서 Publish 앱을 완전히 제거하는 방법을 보여 주는 예입니다.
앱은 환경 설정 내부의 엔진에 추가되므로 앱이 추가된 모든 엔진에서 Publish 앱을 제거해야 합니다.

### 엔진에서 앱 제거

각 엔진에는 [`.../env/includes/settings`](https://github.com/shotgunsoftware/tk-config-default2/tree/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings) 내에 자체 YAML 파일이 있습니다. Publish 앱은 모든 엔진에 포함되어 있으므로 각 엔진 YAML 파일을 수정해야 합니다. 예를 들어 Maya 엔진을 사용하는 경우 [tk-maya.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml)을 열고 Publish 앱에 대한 모든 참조를 제거합니다.

우선 includes 섹션에 앱에 대한 참조가 있습니다.<br/>
[`.../env/includes/settings/tk-maya.yml L18`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L18)

에셋 단계 컨텍스트에 있는 경우 Maya 엔진에도 앱이 포함됩니다.<br/>
[`.../env/includes/settings/tk-maya.yml L47`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L47)<br/>메뉴 즐겨찾기에 이를 추가하는 줄도 있습니다.<br/>
[`.../env/includes/settings/tk-maya.yml L56`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L56)


다음으로 샷 단계 설정에 다음 줄을 반복합니다.<br/>
[`.../env/includes/settings/tk-maya.yml L106`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L106)<br/>
[`.../env/includes/settings/tk-maya.yml L115`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L115)

그런 다음 다른 모든 엔진 환경 yml 파일(예: `tk-nuke`, `tk-3dsmaxplus`, `tk-desktop` 등)에서 이 단계를 반복합니다.

{% include info title="중요" content="이제 통합에서 사용자에게 앱이 표시되지 않으므로 이것으로 충분합니다. 그러나 깨끗하게 유지하기 위해 구성에서 앱에 대한 참조를 완전히 제거하려면 나머지 단계를 완료합니다." %}

### 앱 설정 제거

이러한 모든 엔진 YAML 파일은 [`tk-multi-publish2.yml`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-multi-publish2.yml) 설정 파일을 포함하고 있었습니다. 이제 엔진 YAML 파일에서 해당 참조가 제거되어 이 파일을 완전히 제거할 수 있습니다.

{% include warning title="중요" content="`tk-multi-publish2.yml`을 제거해도 이를 가리키는 엔진 파일이 남아 있는 경우 다음과 같은 내용의 오류가 발생합니다.

    Error
    Include resolve error in '/configs/my_project/env/./includes/settings/tk-desktop2.yml': './tk-multi-publish2.yml' resolved to '/configs/my_project/env/./includes/settings/./tk-multi-publish2.yml' which does not exist! " %}

### 앱 위치 제거

기본 구성에서 모든 앱은 [.../env/includes/app_locations.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml) 파일에 해당 위치 디스크립터를 저장합니다. `tk-multi-publish2.yml`은 이를 참조하므로 [디스크립터 줄](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml#L52-L56)을 제거해야 합니다.
