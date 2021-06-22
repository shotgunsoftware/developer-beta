---
layout: default
title: 소프트웨어를 실행하기 전에 환경 변수는 어떻게 설정합니까?
pagename: setting-software-environment-variables
lang: ko
---

# 소프트웨어를 실행하기 전에 환경 변수는 어떻게 설정합니까?

{% include product %} 툴킷을 사용하면 실행 프로세스 중에 후크를 사용하여 환경을 구성하고 커스텀 코드를 실행할 수 있습니다.

Nuke 또는 Maya와 같은 소프트웨어를 실행할 때 {% include product %} 데스크톱 또는 브라우저 통합을 통해 `tk-multi-launchapp`가 실행됩니다.
이 앱은 소프트웨어 실행을 담당하며 {% include product %} 통합이 예상대로 시작되게 합니다. 이 프로세스 중에 후크를 통해 표시되는 두 지점을 사용하여 커스텀 코드를 실행할 수 있습니다.

## before_app_launch.py

[`before_app_launch.py`](https://github.com/shotgunsoftware/tk-multi-launchapp/blob/6a884aa144851148e8369e9f35a2471087f98d16/hooks/before_app_launch.py) 후크는 소프트웨어가 실행되기 직전에 호출됩니다.
이를 통해 실행된 소프트웨어에 전달될 모든 커스텀 환경 변수를 완벽하게 설정할 수 있습니다.

예시:

```python
import os
import tank

class BeforeAppLaunch(tank.Hook):

    def execute(self, app_path, app_args, version, engine_name, **kwargs):

        if engine_name == "tk-maya":
            os.environ["MY_CUSTOM_MAYA_ENV_VAR"] = "Some Maya specific setting"
```

{% include warning title="경고" content="ShotGrid에서 설정된 환경 변수를 완전히 다시 정의하지 않도록 주의해야 합니다.
예를 들어 `NUKE_PATH`(Nuke) 또는 `PYTHONPATH`(Maya)에 경로를 추가해야 하는 경우 경로를 변경하는 대신 기존 값에 경로를 추가해야 합니다.
이 경우 다음과 같이 편리한 방법을 사용할 수 있습니다.

```python
tank.util.append_path_to_env_var(\"NUKE_PATH\", \"/my/custom/path\")
```
" %}

## 커스텀 래퍼

일부 스튜디오에서는 환경 변수 설정 및 소프트웨어 실행을 처리하는 커스텀 래퍼를 사용합니다.
이와 같은 커스텀 코드를 사용하여 환경을 설정하려는 경우 `Software` 엔티티의 [경로 필드](https://support.shotgunsoftware.com/hc/ko/articles/115000067493-Integrations-Admin-Guide#Example:%20Add%20your%20own%20Software)를 실행 가능한 래퍼로 지정하면 `tk-multi-launchapp`이 대신 실행합니다.

{% include warning title="경고" content="이 경우 ShotGrid에서 설정한 환경 변수를 유지해야 합니다. 그러지 않으면 통합이 시작되지 않습니다." %}