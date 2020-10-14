---
layout: default
title: Houdini Shotgun 통합이 시작되지 않는 이유는 무엇일까요?
pagename: houdini-integrations-not-starting
lang: ko
---

# Houdini Shotgun 통합이 시작되지 않는 이유는 무엇일까요?


여기서는 Houdini에서 Shotgun 통합을 시작할 수 없는 가장 일반적인 원인을 살펴봅니다. 이 경우, Houdini는 Shotgun 데스크톱, Shotgun 웹 사이트 또는 tank 명령으로 오류 없이 시작됩니다. 하지만 Houdini가 시작된 후에 Shotgun 메뉴 또는 쉘프가 나타나지 않습니다.

원인은 Shotgun이 재정의된 `HOUDINI_PATH` 환경 변수를 사용하여 시작 스크립트 경로를 전달하기 때문인 경우가 많습니다.

Shotgun에서 Houdini가 시작되면 시작 앱 로직에서는 Shotgun 부트스트랩(Bootstrap) 스크립트 경로를 `HOUDINI_PATH` 환경 변수에 추가합니다. 그러나 Houdini에 [houdini.env 파일](http://www.sidefx.com/docs/houdini/basics/config_env.html#setting-environment-variables)이 있는 경우 문제가 발생할 수 있습니다.
이 파일을 통해 사용자는 Houdini가 로드될 때 제공되는 환경 변수를 설정할 수 있지만 파일에 정의된 모든 값이 현재 세션의 기존 환경 변수를 덮어씁니다.

이 문제를 해결하려면 기존의 `HOUDINI_PATH` 환경 변수를 해당 변수에 대한 새 정의에 포함해야 합니다.

예를 들어 `houdini.env` 파일에 다음과 같은 내용이 이미 있는 경우

    HOUDINI_PATH = /example/of/an/existing/path;&

파일에 정의된 경로 끝에 `$HOUDINI_PATH;`를 추가하고 저장해야 합니다.

    HOUDINI_PATH = /example/of/an/existing/path;$HOUDINI_PATH;&

그러면 Houdini가 시작될 때 Shotgun 설정 값을 유지할 수 있습니다.

{% include warning title="주의" content="Windows에서 `$HOUDINI_PATH`에 문제가 발생하는 경우가 있습니다. 간혹 Shotgun 통합을 여러 번 부트스트랩(Bootstrap)하며 다음과 같은 오류가 발생합니다.

    툴킷 부트스트랩(Bootstrap)에 필요한 변수가 누락되어 있음: TANK_CONTEXT

이 오류가 발생하면 대신 `%HOUDINI_PATH%`를 사용해야 합니다." %}

그래도 문제가 해결되지 않으면 문제를 진단할 수 있도록 당사 [지원 팀](https://support.shotgunsoftware.com/hc/ko/requests/new)에 문의해 주십시오.