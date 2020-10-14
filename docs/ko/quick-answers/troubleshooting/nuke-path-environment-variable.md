---
layout: default
title: NUKE_PATH 환경 변수를 설정하면 Nuke 통합이 시작되지 않는 이유가 무엇입니까?
pagename: nuke-path-environment-variable
lang: ko
---

# NUKE_PATH 환경 변수를 설정하면 Nuke 통합이 시작되지 않는 이유가 무엇입니까?

통합은 Nuke 시작 프로세스 중에 부트스트랩(Bootstrap) 스크립트가 실행되도록 Nuke 시작 시 `NUKE_PATH` 환경 변수를 설정합니다.
[`before_launch_app.py`](https://github.com/shotgunsoftware/tk-multi-launchapp/blob/6a884aa144851148e8369e9f35a2471087f98d16/hooks/before_app_launch.py) 후크를 실행하기 전에 특별히 `NUKE_PATH`를 정의하는 [`tk-multi-launchapp`](https://support.shotgunsoftware.com/hc/ko/articles/219032968-Application-Launcher#Set%20Environment%20Variables%20and%20Automate%20Behavior%20at%20Launch)입니다.

`os.environ['NUKE_PATH'] = "/my/custom/path"` 등을 사용하여 시작 프로세스 중에 이 환경 변수를 설정하면 환경 변수에서 시작 스크립트 경로를 제거하기 때문에 Shotgun 통합을 시작할 수도 없습니다.

경로를 툴킷 부트스트랩(Bootstrap)으로 유지하는 동안에는 경로를 `NUKE_PATH` 환경 변수에 추가하거나 접두사로 붙이는 다음 함수를 `tank.util`에 사용하십시오.

```python
tank.util.append_path_to_env_var("NUKE_PATH", "/my/custom/path")
```

아니면, `prepend_path_to_env_var()`을 사용하여 경로를 접두사로 붙여도 됩니다.