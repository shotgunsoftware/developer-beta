---
layout: default
title: 어떻게 하면 ShotGrid 이벤트 데몬을 사용하여 다른 Toolkit Core 모듈을 로드할 수 있습니까?
pagename: toolkit-core-event-daemon
lang: ko
---

# 어떻게 하면 shotgunEvents 데몬을 사용하여 다른 Toolkit Core 모듈을 로드할 수 있습니까?

**이 정보를 공유해 준 [Benoit Leveau @ Milk VFX](https://github.com/benoit-leveau)에게 깊이 감사드립니다.**

## 문제

툴킷의 sgtk API는 프로젝트 중심입니다. 즉, 사용할 프로젝트에서 특별히 가져와야 합니다. 이는 단일 Python 세션에서 다중 프로젝트에 대해 sgtk API 작업을 사용하는 경우 Python은 이름이 동일한 모듈을 한 번만 가져올 수 있기 때문에 문제가 발생할 수 있음을 의미합니다.

[{% include product %} 이벤트 데몬](https://github.com/shotgunsoftware/shotgunEvents)을 사용 중인 경우 특정 이벤트에 대해서는 플러그인 내에서 툴킷 액션을 수행하고 싶을 수 있습니다. 하지만 Python은 모듈을 한 번만 가져오기 때문에 이렇게 하는 것은 위험할 수 있습니다. 따라서 프로젝트 A용 Toolkit Core API를 플러그인을 처음 실행할 때 가져오면 그 버전이 데몬의 사용 기간 동안 가져온 채로 유지되는 버전이 됩니다. 즉, 플러그인으로 발송된 다음 이벤트가 프로젝트 B의 이벤트인 경우 프로젝트 A용 Core API를 사용하여 프로젝트 B를 위한 새 툴킷 객체의 인스턴스화를 시도하면 툴킷에서 오류가 발생할 수 있습니다.

**중앙 집중식 구성을 사용하는 경우의 문제 예시:**

- 이벤트 123은 프로젝트 A의 이벤트입니다.
- 프로젝트 A에 대한 Core API는 `/mnt/toolkit/projectA/install/core/python`에 있습니다.
- 이 디렉토리를 `sys.path`에 접두사로 붙입니다.
- `import sgtk`가 이 위치에서 이 API를 가져옵니다.
- 이 Core API로 툴킷 인스턴스를 인스턴스화하고 일부 액션을 수행합니다.
- Core API 디렉토리를 `sys.path`에서 분리합니다.
- 이벤트 234는 프로젝트 B의 이벤트입니다.
- 프로젝트 B에 대한 Core API는 `/mnt/toolkit/projectB/install/core/python`에 있습니다.
- 이 디렉토리를 `sys.path`에 접두사로 붙입니다.
- Python은 이를 이미 가져온 sgtk로 인식하기 때문에 `import sgtk`는 아무 동작도 하지 않습니다.
- 이 Core API로 툴킷 인스턴스를 인스턴스화하고 일부 액션을 수행합니다.
- 툴킷 코어가 작업을 수행하려는 프로젝트 (B)가 아닌 프로젝트 (A)를 위한 것이기 때문에 오류가 발생합니다.

## 솔루션

아래 예는 다른 버전의 모듈을 이미 가져왔을 수도 있는 경우에 스크립트 또는 플러그인에서 올바른 버전의 sgtk 코어를 가져올 수 있는 방법을 보여 줍니다. 원래 가져온 항목은 언로드되어 Python 메모리에서 제거되기 때문에 새로운 모듈 인스턴스를 성공적으로 가져와 사용할 수 있습니다.

```python
"""
Example of how to import the correct sgtk core code in a script where
a different instance of the module may have already been imported. The
original import is unloaded and removed from memory in Python so the new
instance of the module can be imported and used successfully.

Thanks to Benoit Leveau @ Milk VFX for sharing this.
"""

import os
import sys


def import_sgtk(project):
    """
    Import and return the sgtk module related to a Project.
    This will check where the Core API is located on disk (in case it's localized or shared).
    It shouldn't be used to get several instances of the sgtk module at different places.
    This should be seen as a kind of 'reload(sgtk)' command.

    :param project: (str) project name on disk for to import the Toolkit Core API for.
    """
    # where all our pipeline configurations are located
    shotgun_base = os.getenv("SHOTGUN_BASE", "/mnt/sgtk/configs")

    # delete existing core modules in the environment
    for mod in filter(lambda mod: mod.startswith("tank") or mod.startswith("sgtk"), sys.modules):
        sys.modules.pop(mod)
        del mod

    # check which location to use to import the core
    python_subfolder = os.path.join("install", "core", "python")
    is_core_localized = os.path.exists(os.path.join(shotgun_base, project, "install", "core", "_core_upgrader.py"))
    if is_core_localized:
        # the core API is located inside the configuration
        core_python_path = os.path.join(shotgun_base, project, python_subfolder)
    else:
        # the core API can still be localized through the share_core/attach_to_core commands
        # so look in the core_Linux.cfg file which will give us the proper location (modify this
        # to match your primary platform)
        core_cfg = os.path.join(shotgun_base, project, "install", "core", "core_Linux.cfg")
        if os.path.exists(core_cfg):
            core_python_path = os.path.join(open(core_cfg).read(), python_subfolder)
        else:
            # use the studio default one
            # this assumes you have a shared studio core installed.
            # See https://support.shotgunsoftware.com/entries/96141707
            core_python_path = os.path.join(shotgun_base, "studio", python_subfolder)

    # tweak sys.path to add the core API to the beginning so it will be picked up
    if sys.path[0] != "":
        sys.path.pop(0)
    sys.path = [core_python_path] + sys.path

    # Remove the TANK_CURRENT_PC env variable so that it can be populated by the new import
    if "TANK_CURRENT_PC" in os.environ:
        del os.environ["TANK_CURRENT_PC"]

    # now import the sgtk module, it should be found at the 'core_python_path' location above
    import sgtk
    return sgtk
```

## 분산 구성

위 예에서는 [중앙 집중식 구성](https://developer.shotgunsoftware.com/tk-core/initializing.html#centralized-configurations)을 사용한다고 가정하고 있지만 [분산 구성](https://developer.shotgunsoftware.com/tk-core/initializing.html#distributed-configurations)을 사용할 경우에는 상황이 약간 다릅니다. 분산 구성의 경우 sgtk API를 가져오려면 [부트스트랩(Bootstrap) API](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)를 사용해야 합니다. 부트스트랩(Bootstrap) API를 사용하는 경우 일반적으로 비 프로젝트 중심 sgtk API를 가져온 다음 이를 사용하여 지정된 프로젝트에 대한 엔진을 부트스트랩합니다.
이 부트스트랩(Bootstrap) 프로세스는 sgtk 모듈 교환을 처리하여 부트스트랩 프로세스 마지막에 엔진 오브젝트를 사용할 수 있도록 합니다. 부트스트랩(Bootstrap) 후 sgtk를 가져오면 프로젝트에 적합한 관련 sgtk 모듈을 가져옵니다. 위의 예에서 다중 프로젝트에 대해 sgtk를 로드해야 하는 대신 다중 프로젝트에 대해 부트스트랩(Bootstrapping)해야 합니다. 여기 작은 캐시는 한 번에 하나의 엔진만 실행할 수 있으므로 다른 엔진을 로드하기 전에 먼저 삭제해야 합니다.

{% include warning title="경고" content="구성을 부트스트랩(Bootstrapping)하면 프로세스가 구성을 로컬로 캐시하고 모든 종속성을 다운로드해야 하므로 속도가 느려질 수 있습니다. 이벤트 데몬 플러그인에서 부트스트랩(Bootstrapping)하면 성능에 심각한 영향을 미칠 수 있습니다. 한 가지 가능한 접근 방식은 각 프로젝트 부트스트랩(Bootstrap)에 대해 별도의 Python 인스턴스를 생성하여 플러그인에서 명령을 전달하는 것입니다. 이렇게 하면 필요할 때마다 프로젝트를 다시 부트스트랩(Bootstrapping)하지 않아도 됩니다." %}


예는 다음과 같습니다.

```python
# insert the path to the non project centric sgtk API
sys.path.insert(0,"/path/to/non/project/centric/sgtk")
import sgtk

sa = sgtk.authentication.ShotgunAuthenticator()
# Use the authenticator to create a user object.
user = sa.create_script_user(api_script="SCRIPTNAME",
                            api_key="SCRIPTKEY",
                            host="https://SITENAME.shotgunstudio.com")

sgtk.set_authenticated_user(user)

mgr = sgtk.bootstrap.ToolkitManager(sg_user=user)
mgr.plugin_id = "basic."

engine = mgr.bootstrap_engine("tk-shell", entity={"type": "Project", "id": 176})
# import sgtk again for the newly bootstrapped project, (we don't need to handle setting sys paths)
import sgtk
# perform any required operations on Project 176 ...

# Destroy the engine to allow us to bootstrap into another project/engine.
engine.destroy()

# now repeat the process for the next project, although we don't need to do the initial non-project centric sgtk import this time.
# We can reuse the already import sgtk API to bootstrap the next
...
```

{% include info title="참고" content="중앙 집중식 구성도 부트스트랩(Bootstrapping)할 수 있으므로 혼합하여 사용할 경우 다른 방식이 필요하지 않습니다." %}