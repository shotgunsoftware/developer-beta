---
layout: default
title: Linux에서 ShotGrid 데스크톱의 데스크톱/시작 관리자 아이콘을 어떻게 설정합니까?
pagename: create-shotgun-desktop-shortcut
lang: ko
---

# Linux에서 {% include product %} 데스크톱의 데스크톱/시작 관리자 아이콘을 어떻게 설정합니까?

현재 {% include product %} 데스크톱 설치 관리자는 단축키를 자동으로 생성하지 않고 항목을 자동으로 실행하지도 않기 때문에 설치 후에 수동으로 해주어야 합니다. 아이콘을 추가하는 과정은 간단하지만, 어느 Linux 버전을 사용 중인지에 따라 달라질 수 있습니다.

{% include product %} 데스크톱 설치 관리자를 실행하고 나면 {% include product %} 데스크톱 실행 파일이 `/opt/Shotgun folder`에 생성됩니다. 실행 파일 이름은 {% include product %}입니다.
설치 관리자와 함께 배포되는 아이콘은 없습니다. [{% include product %} 데스크톱 엔진 github 리포지토리](https://github.com/shotgunsoftware/tk-desktop/blob/aac6fe004bd003bf26316b9859bd4ebc42eb82dc/resources/default_systray_icon.png)에서 다운로드하십시오.
아이콘을 다운로드하고, 실행 파일(`/opt/Shotgun/Shotgun`)로 경로를 지정하고 나면 수동으로 필요한 데스크톱 또는 메뉴 시작 관리자를 생성하십시오. 이 프로세스는 Linux 버전에 따라 달라질 수 있지만 보통 데스크톱을 마우스 오른쪽 버튼으로 클릭한 다음 적절한 메뉴 옵션을 찾아 데스크톱 시작 관리자를 생성할 수 있습니다.