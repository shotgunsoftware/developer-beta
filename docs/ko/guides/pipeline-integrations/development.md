---
layout: default
title: 개발
pagename: toolkit-development
lang: ko
---

# 개발

## 툴킷은 무엇일까요?

툴킷은 파이프라인 통합의 토대가 되는 플랫폼입니다.
예를 들어 Maya에서 {% include product %} Panel 앱을 사용하거나 {% include product %} Create에서 Publish 앱을 실행하는 경우 툴킷 플랫폼을 기반으로 하는 도구를 사용하게 됩니다.

## 툴킷을 사용하여 개발하려면 어떻게 해야 합니까?

툴킷을 사용하여 개발하는 방법에는 여러 가지가 있습니다.

- 후크라고 하는 커스텀 코드를 작성하여 기존 앱, 엔진 또는 프레임워크 동작을 확장할 수 있습니다.
- 자체 앱, 엔진 또는 프레임워크를 작성할 수 있습니다.
- 또는 API를 사용하는 독립 실행형 스크립트를 작성할 수 있습니다.

이러한 작업을 수행하려면 툴킷 API를 사용하는 방법을 이해하는 것이 중요합니다.

{% include product %}에는 전체적으로 세 가지 주요 API가 있습니다.
- [{% include product %} Python API](https://developer.shotgridsoftware.com/python-api)
- [{% include product %} REST API](https://developer.shotgridsoftware.com/rest-api/)
- [{% include product %} 툴킷 API](https://developer.shotgridsoftware.com/tk-core)

툴킷 API는 {% include product %} Python API 또는 REST API와 함께 사용하도록 설계된 Python API이며 대체용 API가 아닙니다.
툴킷 API에 몇 가지 래퍼 방식이 있지만 일반적으로 {% include product %} 사이트에서 데이터에 액세스해야 할 때는 {% include product %} Python API 또는 REST API를 사용합니다.

대신 툴킷 API는 파일 경로의 통합 및 관리에 중점을 둡니다.
일부 툴킷 앱 및 프레임워크에는 [자체 API](../../reference/pipeline-integrations.md)도 있습니다.

이 문서에서는 툴킷을 사용하여 개발하는 방법에 대해 설명합니다.