---
layout: default
title: 앱 부트스트랩 및 실행
pagename: sgtk-developer-bootstrapping
lang: ko
---

# 앱 부트스트랩(Bootstrapping) 및 실행

이 안내서에서는 커스텀 코드를 실행하거나 앱을 시작할 수 있도록 툴킷 엔진을 초기화하는 과정을 안내합니다. 부트스트랩(Bootstrapping)이라고도 하는 과정입니다.

부트스트랩(Bootstrapping)은 툴킷 엔진이 아직 시작되지 않고 API를 사용해야 하는 경우에 유용합니다.
예를 들어 렌더 팜에서 실행되는 처리 스크립트가 있을 수 있고 툴킷 API를 활용하여 경로 및 컨텍스트를 처리해야 할 수 있습니다.
또는 즐겨찾는 IDE에서 툴킷 앱을 실행할 수도 있습니다.

{% include info title="참고" content="[분산 구성](https://developer.shotgunsoftware.com/tk-core/initializing.html#distributed-configurations)을 사용하는 경우 툴킷 엔진은 툴킷 API 방식을 실행하기 전에 초기화되어야 합니다. [중앙 집중식 구성](https://developer.shotgunsoftware.com/tk-core/initializing.html#centralized-configurations)을 사용하는 경우 엔진을 부트스트랩(Bootstrapping)하지 않고 [팩토리 방식](https://developer.shotgunsoftware.com/tk-core/initializing.html#factory-methods)을 사용하여 API를 사용할 수 있지만 `sgtk`를 가져올 때 프로젝트에 맞는 올바른 Core API의 경로를 수동으로 찾아야 합니다." %}


### 요구사항

- Python 프로그래밍 기본 사항에 대한 이해
- 고급 구성을 사용하는 프로젝트. 구성을 설정하지 않은 경우 ["구성 시작하기"](../getting-started/advanced_config.md) 안내서를 참조하십시오.

### 단계

1. [부트스트랩(Bootstrapping)을 위한 툴킷 API 가져오기](#part-1-importing-the-toolkit-api-for-bootstrapping)
2. [로깅](#part-2-logging)
3. [인증](#part-3-authentication)
4. [엔진 부트스트랩(Bootstrapping)](#part-4-bootstrapping-an-engine)
5. [앱 시작](#part-5-launching-an-app)
6. [전체 스크립트](#part-6-the-complete-script)

## 1부: 부트스트랩(Bootstrapping)을 위한 툴킷 API 가져오기

### 어디에서 sgtk를 가져와야 합니까?

["경로 생성 및 게시"](sgtk-developer-generating-path-and-publish.md) 안내서에 `sgtk` 가져오기 단계에 대해 나와 있습니다.
이 안내서에는 작업할 프로젝트 구성에서 `sgtk` 패키지를 가져와야 한다고 설명되어 있습니다.
맞는 설명이기는 하지만 부트스트랩(Bootstrapping)을 사용할 경우에는 툴킷 API가 다른 프로젝트 구성으로 부트스트랩 작업을 수행할 수 있으므로 어떤 초기 `sgtk` 패키지를 가져오는지는 중요하지 않습니다.
부트스트랩(Bootstrap) 프로세스는 현재 가져온 sgtk 패키지를 새 프로젝트 구성의 툴킷 API로 교체합니다.

### 독립 실행형 Toolkit Core API 다운로드

시작하려면 [`tk-core`](https://github.com/shotgunsoftware/tk-core/tree/v0.18.172/python)에 있는 `sgtk` API 패키지를 가져와야 합니다.
기존 프로젝트에서 가져올 수도 있지만 이 방법은 좀더 까다로울 수 있습니다.
권장되는 방법은 순수하게 부트스트랩(Bootstrapping) 용도로 사용되는 [최신 Core API](https://github.com/shotgunsoftware/tk-core/releases)의 독립 실행형 사본을 다운로드하는 것입니다.
이 파일을 편리하게 가져올 수 있는 위치에 저장해야 합니다.
추가하는 경로가 `sgtk` 패키지가 위치하는 `tk-core` 폴더 내의 `python` 폴더를 가리키는지 확인하십시오.

### 코드

```python
# If your sgtk package is not located in a location where Python will automatically look
# then add the path to sys.path.
import sys
sys.path.insert(0, "/path/to/tk-core/python")

import sgtk
```

## 2부: 로깅

IDE 또는 셸을 통해 이 스크립트를 실행 중인 경우 대부분은 로깅을 활성화하여 출력되게 할 수 있습니다.
이렇게 하려면 [`LogManager().initialize_custom_handler()`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.log.LogManager.initialize_custom_handler)를 실행해야 합니다.
이 작업을 위해 커스텀 처리기를 제공할 필요는 없습니다. 커스텀 처리기를 제공하지 않으면 표준 스트림 기반의 로깅 처리기가 설정됩니다.

필요한 경우 [`LogManager().global_debug = True`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.log.LogManager.global_debug)를 설정하여 보다 상세하게 출력을 표시할 수도 있습니다.
이렇게 설정하면 코드에서 `logger.debug()` 호출 시 출력이 수행됩니다.
로깅은 성능에 영향을 줄 수 있으므로 개발 시에만 디버그 로깅을 활성화하고 정상 작동 중에 가시성을 높이는 것이 중요한 영역에 대한 `logger.info()` 방식 호출의 양을 제한해야 합니다.

```python
import sgtk

# Initialize the logger so we get output to our terminal.
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing).
sgtk.LogManager().global_debug = True
```

## 3부: 인증

{% include product %} 툴킷이 이미 시작된 환경 외부에서 툴킷 API를 사용하는 스크립트를 실행하는 경우 항상 인증을 거쳐야 합니다.
따라서 부트스트랩(Bootstrapping)을 수행하려면 먼저 {% include product %} 사이트에서 툴킷 API를 인증받아야 합니다.

사용자 자격 증명 또는 스크립트 자격 증명으로 인증받을 수 있습니다.

- 앱 시작 또는 사용자 입력이 필요한 일부 코드를 실행하는 등 사용자 대상 프로세스를 부트스트랩(Bootstrapping)하는 것이 목적이라면 사용자 인증을 진행하는 것이 가장 좋습니다(모든 통합의 기본적인 작동 방식임).
- 스크립트를 작성하여 자동화하려고 할 때 사용자가 아직 인증되지 않은 경우 스크립트 자격 증명을 사용해야 합니다.

인증은 [`{% include product %}Authenticator`](https://developer.shotgunsoftware.com/tk-core/authentication.html?highlight=shotgunauthenticator#sgtk.authentication.ShotgunAuthenticator) 클래스를 통해 처리됩니다.
다음은 사용자 인증과 스크립트 인증을 모두 보여 주는 예입니다.

### 사용자 인증

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Optionally you can clear any previously cached sessions. This will force you to enter credentials each time.
authenticator.clear_default_user()

# The user will be prompted for their username,
# password, and optional 2-factor authentication code. If a QApplication is
# available, a UI will pop-up. If not, the credentials will be prompted
# on the command line. The user object returned encapsulates the login
# information.
user = authenticator.get_user()

# Tells Toolkit which user to use for connecting to ShotGrid. Note that this should
# always take place before creating an `Sgtk` instance.
sgtk.set_authenticated_user(user)
```

### 스크립트 인증

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)
```

## 4부: 엔진 부트스트랩(Bootstrapping)

이제 세션을 위해 툴킷 API를 인증받았으므로 부트스트랩(Bootstrapping) 프로세스를 시작할 수 있습니다.
[참조 문서](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)에서 부트스트랩(Bootstrap) API에 관한 많은 정보를 확인할 수 있습니다.

부트스트랩(Bootstrapping) 프로세스는 기본적으로 다음과 같은 단계를 수행합니다.

1. 툴킷 구성 폴더를 검색하거나 찾습니다.
2. 앱 및 엔진과 같은 구성 종속 요소가 [번들 캐시](../../../quick-answers/administering/where-is-my-cache.md#bundle-cache)에 있는지 확인합니다.
   구성 종속 요소가 없고 [`app_store`](https://developer.shotgunsoftware.com/tk-core/descriptor.html#the-shotgun-app-store) 또는 [`{% include product %}`](https://developer.shotgunsoftware.com/tk-core/descriptor.html#pointing-at-a-file-attachment-in-shotgun)과 같은 클라우드 기반 디스크립터를 사용하는 경우 번들 캐시에 이러한 디스크립터를 다운로드합니다.
3. 현재 로드된 Sgtk Core를 구성에 적합한 Sgtk Core로 교체합니다.
4. 엔진, 앱 및 프레임워크를 초기화합니다.


{% include info title="참고" content="일반적으로 부트스트랩(Bootstrapping)은 해당 엔진이 성공적으로 실행되기 위해 필요한 모든 사항을 처리합니다.
그러나 엔진이 부트스트랩(Bootstrapping) 프로세스를 벗어나는 설정 요구사항을 가지고 있다면 개별적으로 처리해야 할 수도 있습니다." %}


### 부트스트랩(Bootstrap) 준비
부트스트랩(Bootstrap)하려면 먼저 [`ToolkitManager`](https://developer.shotgunsoftware.com/tk-core/initializing.html#toolkitmanager) 인스턴스를 작성해야 합니다.

```python
mgr = sgtk.bootstrap.ToolkitManager()
```

툴킷을 부트스트랩(Bootstrapping)하려면 최소한 엔티티, 플러그인 ID 및 엔진에 대해 알아야 합니다.
이 안내서에서는 사용 가능한 모든 매개변수와 옵션 중 일부만을 설명합니다. 전체 내용은 [참조 문서](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)에서 참조하십시오.

#### 플러그인 ID

부트스트랩(Bootstrap) 방식을 호출하기 전에 문자열을 `ToolkitManager.plugin_id` 매개변수로 전달하여 플러그인 ID를 정의할 수 있습니다.
이 안내서에서는 참조 문서에 설명된 규칙에 따라 적합한 플러그인 ID 이름을 제공해야 하므로 `tk-shell` 엔진을 부트스트랩(Bootstrapping)합니다.
```python
mgr.plugin_id = "basic.shell"
```

#### 엔진
Maya 또는 Nuke와 같은 소프트웨어 외의 독립 실행형 Python 환경에서 앱을 시작하거나 툴킷 코드를 실행하는 것이 목표인 경우 `tk-shell`이 부트스트랩(Bootstrapping)할 엔진입니다.

지원되는 소프트웨어 내에서 툴킷 앱을 실행하려는 경우 적절한 엔진(예: `tk-maya` 또는 `tk-nuke`)을 선택해야 합니다.
이 매개변수는 [`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 방식에 직접 전달됩니다. 아래 [엔티티 섹션](#entity)의 예를 참조하십시오.

#### 엔티티
[`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 방식 `entity` 매개변수는 시작한 엔진에 대해 [컨텍스트](https://developer.shotgunsoftware.com/tk-core/core.html#context) 및 [환경](https://developer.shotgunsoftware.com/tk-core/core.html?highlight=environment#module-pick_environment)을 설정하는 데 사용됩니다.
이 엔티티는 구성이 작동하도록 설정된 엔티티 유형 중 하나일 수 있습니다.
예를 들어 `Project` 엔티티를 제공하는 경우 엔진은 프로젝트 환경 설정을 사용하여 프로젝트 컨텍스트에서 시작됩니다.
마찬가지로, 태스크가 `Asset`에 링크되어 있는 `Task` 엔티티를 제공할 수 있으며, 이 엔티티는 `asset_step.yml` 환경을 사용하여 시작됩니다.
이는 기본 구성 동작을 기반으로 하며, [선택한 환경](https://developer.shotgunsoftware.com/ko/487a9f2c/#%ED%88%B4%ED%82%B7%EC%9D%B4-%ED%98%84%EC%9E%AC-%ED%99%98%EA%B2%BD%EC%9D%84-%EA%B2%B0%EC%A0%95%ED%95%98%EB%8A%94-%EB%B0%A9%EC%8B%9D)은 코어 후크, [`pick_environment.py`](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.11/core/hooks/pick_environment.py)를 통해 제어되므로 컨텍스트 또는 기타 매개변수를 기반으로 다른 환경을 선택하도록 변경할 수 있습니다.

유형과 ID를 하나 이상 포함해야 하는 {% include product %} 엔티티 사전(dictionary) 형식으로 엔티티를 제공해야 합니다.

```python
task = {"type": "Task", "id": 17264}
engine = mgr.bootstrap_engine("tk-shell", entity=task)
```

`Project` 외의 다른 엔티티 유형으로 부트스트랩(Bootstrapping)하는 경우 [경로 캐시](https://developer.shotgunsoftware.com/ko/cbbf99a4/)가 동기화되어 있는지 확인해야 할 수 있습니다. 그렇지 않으면 템플릿을 해석하려고 시도하는 등의 경우에 환경을 로드할 수 없을 수 있습니다.
부트스트랩하기 전에는 `Sgtk` 인스턴스가 없으므로 부트스트랩(Bootstrapping) 프로세스가 `Sgtk` 인스턴스를 생성한 후, 엔진을 시작하기 전에 동기화하도록 해야 합니다.
이 작업은 [`ToolkitManager.pre_engine_start_callback`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.pre_engine_start_callback) 특성을 커스텀 방식을 가리키도록 설정하여 수행할 수 있습니다.
그런 다음 해당 방식으로 동기화를 실행할 수 있습니다.

```python
def pre_engine_start_callback(ctx):
    '''
    Called before the engine is started.

    :param :class:"~sgtk.Context" ctx: Context into
        which the engine will be launched. This can also be used
        to access the Toolkit instance.
    '''
    ctx.sgtk.synchronize_filesystem_structure()

mgr.pre_engine_start_callback = pre_engine_start_callback
```


#### 구성 선택

부트스트랩할 구성을 명시적으로 정의할 수도 있고 부트스트랩(Bootstrapping) 로직이 [적절한 구성을 자동 감지](https://developer.shotgunsoftware.com/tk-core/initializing.html#managing-distributed-configurations)하도록 할 수도 있습니다.
적절한 구성이 자동으로 감지되지 않는 경우 폴백 구성을 설정할 수도 있습니다.
이 안내서에서는 프로젝트에 이미 구성이 설정되어 있고 자동으로 검색된다고 가정합니다.

### 부트스트랩(Bootstrapping)

모든 [`ToolkitManager`](https://developer.shotgunsoftware.com/tk-core/initializing.html#toolkitmanager) 매개변수가 설정된 경우 [`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 방식을 호출하면 엔진이 시작되고 엔진 인스턴스로 포인터가 반환됩니다.

다음은 지금까지 코드의 요약입니다.

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Initialize the logger so we get output to our terminal.
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing).
sgtk.LogManager().global_debug = True

# Authentication
################

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)

# Bootstrap
###########

# create an instance of the ToolkitManager which we will use to set a bunch of settings before initiating the bootstrap.
mgr = sgtk.bootstrap.ToolkitManager()
mgr.plugin_id = "basic.shell"

project = {"type": "Project", "id": 176}

engine = mgr.bootstrap_engine("tk-shell", entity=project)
```

## 5부: 앱 시작

이제 엔진 인스턴스가 있으므로 툴킷 API를 사용할 준비가 되었습니다.

앱을 시작하는 방법을 설명하기 전에 엔진을 통해 [현재 컨텍스트](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context), [Sgtk 인스턴스](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk) 및 [{% include product %} API 인스턴스](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.shotgun)를 확인할 수 있습니다.

```python
engine.context
engine.sgtk
engine.shotgun
```
이 안내서의 최종 목표는 앱 시작 방법을 보여 주는 것입니다. 이 시점에서 위의 속성을 사용하여 일부 코드 조각을 테스트하거나 툴킷 API를 사용하는 일부 자동화를 실행할 수 있습니다.

### 앱 시작

엔진이 시작되면 환경에 대해 정의된 모든 앱이 초기화됩니다.
앱이 차례로 엔진에 명령을 등록하며, Maya와 같은 소프트웨어에서 실행하는 경우 엔진은 대개 이 명령을 메뉴에 액션으로 표시합니다.

#### 명령 찾기
등록된 명령을 먼저 확인하려면 [`Engine.commands`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.commands) 특성을 출력하면 됩니다.

```python
# use pprint to give us a nicely formatted output.
import pprint
pprint.pprint(engine.commands.keys())

>> ['houdini_fx_17.5.360',
 'nukestudio_11.2v5',
 'nukestudio_11.3v2',
 'after_effects_cc_2019',
 'maya_2019',
 'maya_2018',
 'Jump to Screening Room Web Player',
 'Publish...',
...]
```

출력된 목록에서 등록된 명령 및 실행 가능한 명령을 확인할 수 있습니다.

#### 명령 실행

현재 표준화된 방식이 없으므로 명령을 실행하는 방법은 엔진에 따라 달라집니다.
`tk-shell` 엔진의 경우 편리한 `Engine.execute_command()` 방식을 사용할 수 있습니다.
앞의 목록에 있는 명령 문자열 이름과 앱의 명령을 전달할 매개변수 목록이 필요합니다.

```python
if "Publish..." in engine.commands:
    # Launch the Publish app, and it doesn't require any arguments to run so provide an empty list.
    engine.execute_command("Publish...",[])
```

`tk-shell` 엔진에서 실행하지 않는 경우 등록된 콜백을 직접 호출하도록 폴백할 수 있습니다.

```python
# now find the command we specifically want to execute
app_command = engine.commands.get("Publish...")

if app_command:
    # now run the command, which in this case will launch the Publish app.
    app_command["callback"]()
```

이제 앱이 시작되고 `tk-shell` 엔진을 실행 중인 경우 출력이 터미널/콘솔에 표시됩니다.

## 6부: 전체 스크립트

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Initialize the logger so we get output to our terminal
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing)
sgtk.LogManager().global_debug = True

# Authentication
################

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)

# Bootstrap
###########

# create an instance of the ToolkitManager which we will use to set a bunch of settings before initiating the bootstrap.
mgr = sgtk.bootstrap.ToolkitManager()
mgr.plugin_id = "basic.shell"

project = {"type": "Project", "id": 176}

engine = mgr.bootstrap_engine("tk-shell", entity=project)

# Optionally print out the list of registered commands:
# use pprint to give us a nicely formatted output.
# import pprint
# pprint.pprint(engine.commands.keys())

if "Publish..." in engine.commands:
    # Launch the Publish app, and it doesn't require any arguments to run so provide an empty list.
    engine.execute_command("Publish...",[])
```
