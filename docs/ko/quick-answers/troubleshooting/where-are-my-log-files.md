---
layout: default
title: 로그 파일은 어디에 있습니까?
pagename: where-are-my-log-files
lang: ko
---

# 로그 파일은 어디에 있습니까?

기본적으로 {% include product %} 데스크톱 및 통합은 로그 파일을 다음 디렉토리에 저장합니다.

**Mac**

`~/Library/Logs/Shotgun/`

**Windows**

`%APPDATA%\Shotgun\logs\`

**Linux**

`~/.shotgun/logs/`

로그 파일 이름의 형식은 `tk-<ENGINE>.log`입니다. 예를 들면 `tk-desktop.log` 또는 `tk-maya.log`입니다.

[`{% include product %}_HOME` 환경 변수](http://developer.shotgunsoftware.com/tk-core/utils.html#localfilestoragemanager)를 설정하여 사용자의 캐시 위치를 재정의한 경우 로그 파일이 `$SHOTGUN_HOME/logs`에 위치하게 됩니다.

{% include info title="참고" content="ShotGrid 데스크톱에서도 이 디렉토리에 액세스할 수 있습니다. 프로젝트를 선택하고, 프로젝트 이름 오른쪽에 있는 아래쪽 화살표 버튼을 클릭하고, **로그 폴더 열기(Open Log Folder)**를 선택하면 됩니다." %}
