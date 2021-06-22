---
layout: default
title: 경로 생성 및 게시
pagename: sgtk-developer-generating-path-and-publish
lang: ko
---

# 경로 생성 및 게시

이 안내서에서는 파이프라인 통합을 빌드하는 {% include product %} 툴킷 Python API를 시작하는 방법에 대해 설명합니다.

이 안내서를 통해 API를 사용하는 방법에 대한 기본적인 예제를 살펴보십시오. 학습을 모두 마치면 툴킷 API를 가져와 경로를 생성하고 게시할 수 있게 될 것입니다.

### 요구사항

- Python 프로그래밍 기본 사항에 대한 이해
- 고급 구성을 사용하는 프로젝트. 구성을 설정하지 않은 경우 ["구성 시작하기"](링크 필요) 안내서를 참조하십시오.

### 단계

1. [Sgtk 가져오기](#part-1-importing-sgtk)
2. [Sgtk 인스턴스 가져오기](#part-2-getting-an-sgtk-instance)
3. [컨텍스트 가져오기](#part-3-getting-context)
4. [폴더 생성](#part-4-creating-folders)
5. [템플릿을 사용하여 경로 빌드](#part-5-using-a-template-to-build-a-path)
6. [기존 파일 찾기 및 최신 버전 번호 가져오기](#part-6-finding-existing-files-and-getting-the-latest-version-number)
7. [게시된 파일 등록](#part-7-registering-a-published-file)
8. [전체 스크립트로 모두 가져오기](#part-8-the-complete-script)

## 1부: Sgtk 가져오기

툴킷 API는 `sgtk`라는 Python 패키지에 포함되어 있습니다.
각 툴킷 구성에는 [`tk-core`](https://developer.shotgunsoftware.com/tk-core/overview.html)의 일부로 제공되는 자체 API 사본이 있습니다.
프로젝트 구성에서 API를 사용하려면 작업할 구성에서 `sgtk` 패키지를 가져와야 합니다. 이 패키지를 다른 구성에서 가져오면 오류가 발생합니다.

{% include info title="참고" content="경우에 따라 `tank` 패키지에 대한 참조가 있을 수 있습니다. 이는 동일한 작업에 대한 이전 이름입니다. 앞으로 계속 사용할 수 있는 올바른 이름은 `sgtk`입니다." %}

API를 가져오려면 [코어의 python 폴더](https://github.com/shotgunsoftware/tk-core/tree/v0.18.167/python)에 대한 경로가 [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path)에 있는지 확인해야 합니다.
그러나 이 예의 경우 {% include product %} 데스크톱의 Python 콘솔에서 이 코드를 실행하는 것이 좋습니다.
이 경우 올바른 `sgtk` 패키지 경로가 `sys.path`에 이미 추가되어 있습니다.
마찬가지로, {% include product %} 통합이 이미 실행 중인 소프트웨어 내에서 이 코드를 실행하는 경우에는 경로를 추가할 필요가 없습니다.

{% include product %}가 이미 시작된 환경에서 코드를 실행하는 경우 다음과 같이 간단한 방식으로 API를 가져올 수 있습니다.

```python
import sgtk
```
{% include product %} 통합 외부에서 API를 사용하려면(예: 즐겨 사용하는 IDE에서 테스트하는 경우) 먼저 API에 대한 경로를 설정해야 합니다.

```python
import sys
sys.path.append("/shotgun/configs/my_project_config/install/core/python")

import sgtk
```

{% include info title="참고" content="분산 구성을 사용 중이고 툴킷이 이미 부트스트랩(Bootstrapping)되지 않은 환경에서 `sgtk`를 가져오려는 경우에는 다른 방법을 사용해야 합니다. 자세한 내용은 [부트스트랩 안내서](sgtk-developer-bootstrapping.md)를 참조하십시오." %}

## 2부: Sgtk 인스턴스 가져오기

툴킷 API를 사용하려면 [`Sgtk`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk) 클래스의 인스턴스를 만들어야 합니다.

[`Sgtk`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk)는 `sgtk` 패키지의 클래스로, API에 대한 메인 인터페이스 역할을 합니다.
`Sgtk` 인스턴스를 만들면 컨텍스트 가져오기, 폴더 만들기 또는 템플릿 액세스 같은 작업을 수행할 수 있습니다.

API 설명서에 언급된 것처럼 `Sgtk` 인스턴스를 직접 만들지는 않습니다. 다음은 `Sgtk` 인스턴스를 가져오기 위한 몇 가지 옵션입니다.

1. {% include product %} 통합이 이미 실행 중인 환경(예: Maya가 {% include product %}에서 시작된 경우 Maya Python 콘솔)에서 Python 코드를 실행하는 경우 현재 엔진에서 `Sgtk` 인스턴스를 가져올 수 있습니다.
   `Engine.sgtk` 특성은 엔진의 `Sgtk` 인스턴스를 유지합니다.
   따라서 Maya와 같은 응용프로그램에서 다음을 실행할 수 있습니다.

   ```python
   # Get the engine that is currently running.
   current_engine = sgtk.platform.current_engine()

   # Grab the already created Sgtk instance from the current engine.
   tk = current_engine.sgtk
   ```

   [`Engine.sgtk`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk) 특성을 통해 `Sgtk` 인스턴스에 액세스할 수 있습니다.

   *참고: `Engine.sgtk` 특성을 1부에서 가져온 `sgtk` 패키지와 혼동하거나 동일하게 간주해서는 안 됩니다.*

2. [`sgtk.sgtk_from_entity()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_entity) - 엔진이 아직 시작되지 않은 환경에서 실행 중인 경우 이 방식을 사용하여 엔티티 ID에 맞게 `Sgtk` 인스턴스를 가져올 수 있습니다.
   ID를 제공하는 엔티티는 `sgtk` API를 가져온 프로젝트에 속해야 합니다.
   *이 방식은 분산 구성에서 작동하지 않습니다. 자세한 내용은 [부트스트랩 안내서](sgtk-developer-bootstrapping.md)를 참조하십시오.*

3. [`sgtk.sgtk_from_path()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_path) - 구성 경로나 프로젝트 루트 폴더 내부 또는 이에 대한 경로(예: 작업 파일 또는 샷 폴더)를 허용하는 경우를 제외하고 `sgtk_from_entity()`와 비슷합니다.
   *이 방식은 분산 구성에서 작동하지 않습니다. 자세한 내용은 [부트스트랩 안내서](sgtk-developer-bootstrapping.md)를 참조하십시오.*

이 안내서에서는 엔진이 이미 시작된 환경에서 이 코드를 실행한다고 가정하므로 옵션 1을 사용합니다.
또한 `Sgtk` 클래스 인스턴스를 `tk`라는 변수에 저장합니다.
{% include product %} Python 콘솔을 사용 중인 경우 `tk` 변수가 이미 전역 변수로 미리 정의되어 있습니다.

이제 `Sgtk` 인스턴스가 있으므로 API를 사용할 준비가 되었습니다.
이제 게시 스크립트는 다음과 같습니다.

```python
import sgtk

# Get the engine that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the already created Sgtk instance from the current engine.
tk = current_engine.sgtk
```

## 3부: 컨텍스트 가져오기

### 컨텍스트는 무엇이며 필요한 이유는 무엇입니까?

툴킷에서 발생하는 많은 작업은 컨텍스트를 중심으로 이루어집니다. 즉, 사용자가 무슨 작업 중인지 인지하고 이에 맞게 진행한다는 의미입니다.
툴킷 API를 사용하여 작업 중인 엔티티에 대한 중요한 상세 정보를 저장하고 앱 또는 다른 프로세스와 공유할 수 있어야, 이렇게 컨텍스트를 인식한 작동이 가능합니다.
예를 들어 툴킷에서 작업 중인 태스크를 인식할 경우 게시된 파일을 ShotGrid에서 해당하는 태스크에 자동으로 링크할 수 있습니다.

[`Context` 클래스](https://developer.shotgunsoftware.com/tk-core/core.html#context)는 이러한 정보의 컨테이너로 사용됩니다.
클래스 인스턴스 내에 몇 가지 항목 중 `Task`, `Step`, `entity`(예: `Shot` 또는 `Asset`), `Project` 및 현재 `HumanUser`를 저장할 수 있습니다.

지정된 세션에서 원하는 만큼의 컨텍스트 객체 유형을 생성할 수 있습니다. 그러나 엔진이 있는 경우 엔진이 트래킹할 수 있는 단일 컨텍스트의 개념이 적용됩니다.
이 개념이 사용자가 현재 작업 중이고 앱이 작동해야 하는 컨텍스트입니다.

이후 단계에서는 이 컨텍스트를 사용하여 파일을 저장하거나 복사하는 데 사용할 수 있는 경로를 해석합니다.

### 컨텍스트 획득

컨텍스트를 생성하려면 생성자 방식 `Sgtk.context_from_entity()`, `Sgtk.context_from_entity_dictionary()` 또는 `Sgtk.context_from_path()` 중 하나를 사용해야 합니다.
이전 단계에서 만든, `tk` 변수에 저장한 `Sgtk` 인스턴스를 통해 이러한 방식에 액세스합니다.

{% include info title="참고" content="경로에서 컨텍스트를 가져오려면 이미 생성된 폴더가 있어야 합니다. 이 부분은 안내서의 다음 단계에서 설명합니다." %}

그러나 새 컨텍스트를 생성하는 대신 다음과 같이 [2부](#part-2-getting-an-sgtk-instance)에서 수집한 [현재 컨텍스트를 엔진에서 가져올](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context) 수 있습니다.

```python
context = current_engine.context
```
이후 단계에서 샷의 태스크 파일 경로를 해석하는 데 컨텍스트를 사용하므로 관련 정보가 컨텍스트에 포함되어 있어야 합니다.

코드가 툴킷 앱의 일부로 실행되고 앱이 shot_step 환경에서만 실행되도록 구성된 경우에는 적합한 현재 컨텍스트를 가져온다고 충분히 가정할 수 있습니다.
그러나 이 안내서에서는 보다 확실하게 하기 위해 `Sgtk.context_from_entity()`를 사용하여 `Task`(`Shot`에 속해야 함)에서 명시적으로 컨텍스트를 생성합니다.

컨텍스트를 생성할 때는 작업에 필요한 가장 깊은 수준을 제공합니다.
예를 들어 태스크에서 컨텍스트를 만들 수 있으며 툴킷이 나머지 컨텍스트 매개변수를 처리합니다.

```python
context = tk.context_from_entity("Task", 13155)
```

컨텍스트 인스턴스 표현을 출력할 경우 다음과 같이 나타납니다.

```python
print(repr(context))

>> <Sgtk Context:   Project: {'type': 'Project', 'name': 'My Project', 'id': 176}
  Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}
  Step: {'type': 'Step', 'name': 'Comp', 'id': 8}
  Task: {'type': 'Task', 'name': 'Comp', 'id': 13155}
  User: None
  Shotgun URL: https://mysite.shotgunstudio.com/detail/Task/13155
  Additional Entities: []
  Source Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}>

```

태스크만 제공했는데 다른 관련 세부 사항도 채워져 있습니다.

이제 게시 스크립트는 다음과 같이 됩니다.

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)
```

## 4부: 폴더 생성

툴킷은 프로젝트 엔티티를 기반으로 디스크에 폴더 구조를 동적으로 생성할 수 있습니다.

이 단계는 두 가지 목적을 충족합니다.

1. 파일을 배치할 수 있는 디스크에 조직화된 구조를 생성합니다.
2. 이렇게 하면 툴킷이 프로그래밍 방식으로 구조를 이해하고 해당 구조에서 컨텍스트를 파생하고 파일을 배치할 위치를 알 수 있습니다.

나중에 경로를 해석할 수 있도록 디스크에 폴더가 있는지 확인해야 합니다.
이를 위해서는 [Sgtk.create_filesystem_structure()](https://developer.shotgunsoftware.com/tk-core/core.html?#sgtk.Sgtk.create_filesystem_structure) 방식을 사용합니다.

```python
tk.create_filesystem_structure("Task", context.task["id"])
```
컨텍스트 객체를 사용하여 태스크 ID를 가져와 폴더를 생성할 수 있습니다.

이제 코드는 다음과 같아야 합니다.

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task, this Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])
```

이제 모든 준비 단계를 완료했으며 템플릿을 사용한 경로 생성 단계를 진행할 준비가 되었습니다.

## 5부: 템플릿을 사용하여 경로 빌드

### 경로 생성

파일을 배치하거나 툴킷에서 찾을 위치를 확인해야 할 때 템플릿을 사용하여 디스크 내에서의 절대 경로를 해석할 수 있습니다.

[템플릿](https://developer.shotgunsoftware.com/tk-core/core.html#templates)은 기본적으로 컨텍스트 및 기타 데이터를 적용할 때 파일 시스템 경로로 해석할 수 있는 토큰화된 문자열입니다.
템플릿은 [프로젝트의 파이프라인 구성](https://support.shotgunsoftware.com/hc/ko/articles/219039868-Integrations-File-System-Reference#Part%202%20-%20Configuring%20File%20System%20Templates)을 통해 커스터마이즈할 수 있으며 템플릿의 목적은 파일이 저장된 위치에서 작업하기 위한 표준화된 방법을 제공하는 것입니다.

가장 먼저 수행할 작업은 생성하려는 경로에 대한 템플릿 인스턴스를 가져오는 것입니다.
생성한 `Sgtk` 인스턴스를 사용하여 `Sgtk.templates` 속성을 통해 원하는 `Template` 인스턴스에 액세스할 수 있습니다. 이 속성은 키가 템플릿 이름이고 값이 [`Template`](https://developer.shotgunsoftware.com/tk-core/core.html#template) 인스턴스인 사전(dictionary)입니다.

```python
template = tk.templates["maya_shot_publish"]
```

이 예에서는 `maya_shot_publish` 템플릿을 사용합니다.
[기본 구성](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.12/core/templates.yml#L305-L306)에서 해석되지 않은 템플릿 경로는 다음과 같습니다.

```yaml
'sequences/{Sequence}/{Shot}/{Step}/work/maya/{name}.v{version}.{maya_extension}'
```

템플릿은 실제 값으로 해석되어야 하는 키로 구성됩니다.
대부분의 키에 대한 정보가 컨텍스트에 충분히 포함되어 있으므로 이를 사용하여 값을 추출할 수 있습니다.

```python
fields = context.as_template_fields(template)

>> {'Sequence': 'seq01_chase', 'Shot': 'shot01_running_away', 'Step': 'comp'}
```
[`Context.as_template_fields()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Context.as_template_fields) 방식을 사용하면 템플릿 키를 올바르게 해석하기 위한 값을 포함하는 사전(dictionary)이 제공됩니다.
그러나 모든 키에 대해 값이 제공되지는 않습니다. `name`, `version` 및 `maya_extension`은 여전히 누락된 상태입니다.

`maya_extension` 키는 템플릿 키 섹션에서 [기본값을 정의](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.8/core/templates.yml#L139)하므로 기본값을 제외한 다른 값을 원하더라도 해당 값을 제공할 필요가 없습니다.

그러면 `name`과 `version`이 남습니다. 이름은 선택의 문제이므로 기본값을 하드 코딩하거나 인터페이스를 팝업하는 등의 방식으로 사용자가 값을 입력할 수 있는 기회를 제공할 수 있습니다.
지금은 둘 다 모두 하드 코딩하지만, 다음 단계에서는 사용 가능한 다음 버전 번호를 찾는 방법을 알아보겠습니다.

```python
fields["name"] = "myscene"
fields["version"] = 1
```

이제 모든 필드가 준비되었습니다. 다음과 같이 [`Template.apply_fields()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Template.apply_fields)를 사용하여 템플릿을 절대 경로로 해석할 준비가 되었습니다.

```python
publish_path = template.apply_fields(fields)

>> /sg_toolkit/mysite.shotgunstudio.com/my_project/sequences/seq01_chase/shot01_running_away/comp/publish/maya/myscene.v001.ma
```

### 폴더 존재 여부 확인

앞에서 폴더 생성 방식을 실행했지만 모든 폴더가 있는지 확인하기 위해 추가 단계를 수행해야 할 수 있습니다.
이 단계는 예를 들어 템플릿이 스키마에 없는 폴더를 정의하여 이에 따라 원래 `create_filesystem_structure()` 호출에서 생성되지 않은 폴더를 정의하는 경우에 필요할 수 있습니다.

이 작업을 수행할 때 편리하게 사용할 수 있는 몇 가지 방법이 있습니다.
코드가 툴킷 앱 또는 후크에서 실행 중인 경우 [`Application.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Application.ensure_folder_exists) 방식을 사용할 수 있습니다.
엔진이 있는 경우 [`Engine.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.ensure_folder_exists) 방식을 사용할 수 있습니다.
또는 엔진 외부에서 코드를 실행하는 경우 [`sgtk.util.filesystem.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.util.filesystem.ensure_folder_exists) 방식을 사용할 수 있습니다.
전체 파일 경로가 아닌 해당 디렉토리의 폴더만 생성해야 합니다.
[`os`](https://docs.python.org/3/library/os.html) 모듈을 가져오고 [`os.path.dirname(publish_path)`](https://docs.python.org/3/library/os.path.html#os.path.dirname)를 실행하여 폴더의 전체 파일 경로를 추출할 수 있습니다.

### 경로를 사용하여 파일 생성 또는 복사
이 시점에는 경로가 있으며, 이 경로를 사용하여 Maya에 파일을 저장하거나 다른 위치에서 파일을 복사하도록 지시할 수 있습니다.
이 안내서에서는 해당 위치에서 디스크에 파일을 실제로 생성하는 동작을 구현하는 것은 중요하게 다루지 않습니다.
해당 위치에 파일이 없더라도 경로를 게시할 수 있습니다.
그러나 [`sgtk.util.filesystem.touch_file()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.filesystem.touch_file)을 사용하여 툴킷이 디스크에 빈 파일을 생성하도록 할 수 있습니다.


### 최종 결과

```python
import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])

# Get a template instance by providing a name of a valid template in your config's templates.yml.
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"
fields["version"] = 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders.
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Create an empty file on disk. (optional - should be replaced by actual file save or copy logic)
sgtk.util.filesystem.touch_file(publish_path)
```

다음 단계는 하드 코딩하지 않고 다음 버전 번호를 동적으로 처리하는 것입니다.

## 6부: 기존 파일 찾기 및 최신 버전 번호 가져오기

여기에서 사용할 수 있는 두 가지 방법이 있습니다.

1. 이 특별한 예제에서는 게시 파일을 해석하므로 [{% include product %} API](https://developer.shotgunsoftware.com/python-api/)를 사용하여 `PublishedFile` 엔티티에 대해 다음으로 사용 가능한 버전 번호를 쿼리할 수 있습니다.
2. 디스크의 파일을 스캔하고 이미 있는 버전을 확인한 후 다음 버전 번호를 추출할 수도 있습니다.
   이 옵션은 작업 중인 파일이 {% include product %}에서 트래킹되지 않는 경우(예: 작업 파일)에 유용합니다.

첫 번째 옵션이 이 안내서의 예에 가장 적합하지만 두 방법 모두 각자 용도가 있으므로 사용되므로 둘 다 설명하겠습니다.

### {% include product %}에 다음 버전 번호 쿼리

{% include product %} API와 [`summarize()` 방식](https://developer.shotgunsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.summarize)을 사용하여 동일한 이름과 태스크를 공유하는 `PublishedFile` 엔티티 중에서 가장 높은 버전 번호를 가져온 다음 1을 추가하면 됩니다.

```python
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1
```

### 파일 시스템에서 다음 버전 번호 검색

툴킷 API를 사용하여 기존 파일 목록을 수집하고 기존 파일에서 템플릿 필드 값을 추출한 후 다음 버전을 계산할 수 있습니다.

아래 예에서는 작업 파일 템플릿에서 최신 버전을 수집합니다.
작업 파일 템플릿과 게시 파일 템플릿에 동일한 필드가 있다고 가정하면 아래의 방식을 동일한 필드로 두 번 호출하여 가장 높은 게시 및 작업 파일 버전을 확인하고 두 버전의 조합을 사용할지 결정할 수 있습니다.

```python
def get_next_version_number(tk, template_name, fields):
    template = tk.templates[template_name]

    # Get a list of existing file paths on disk that match the template and provided fields
    # Skip the version field as we want to find all versions, not a specific version.
    skip_fields = ["version"]
    file_paths = tk.paths_from_template(
                 template,
                 fields,
                 skip_fields,
                 skip_missing_optional_keys=True
             )

    versions = []
    for a_file in file_paths:
        # extract the values from the path so we can read the version.
        path_fields = template.get_fields(a_file)
        versions.append(path_fields["version"])

    # find the highest version in the list and add one.
    return max(versions) + 1

# Set the version number in the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = get_next_version_number(tk, "maya_shot_work", fields)
```

[`sgtk.paths_from_template()`](https://developer.shotgunsoftware.com/tk-core/core.html?highlight=paths_from_template#sgtk.Sgtk.paths_from_template) 방식은 제공된 템플릿 및 필드와 일치하는 디스크의 파일을 모두 수집합니다.
이 방식은 파일 목록을 찾아서 사용자에게 표시하려는 시나리오에도 유용합니다.

두 옵션 중 하나를 사용하도록 선택할 수 있으며, 이 안내서에서는 코드를 간단하게 유지할 수 있도록 옵션 1의 코드를 사용합니다.

## 7부: 게시된 파일 등록

이제 경로가 생성되고 게시할 준비가 되었습니다. 유틸리티 방식 [`sgtk.util.register_publish()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.register_publish)를 사용하여 이 작업을 수행할 수 있습니다.

{% include product %} API의 [`{% include product %}.create()`](https://developer.shotgunsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.create) 방식을 사용하여 `PublishedFile` 엔티티를 만들 수도 있지만 필요한 모든 필드가 제공되고 올바르게 입력되기 때문에 툴킷 API를 사용하는 것이 가장 좋습니다.

```python
# So as to match the Publish app's default behavior, we are adding the extension to the end of the publish name.
# This is optional, however.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

또한 [Publish 앱](https://support.shotgunsoftware.com/hc/ko/articles/115000097513-Publishing-your-work)이 [자체 API](https://developer.shotgunsoftware.com/tk-multi-publish2/)와 함께 제공된다는 점도 주목할 만합니다.
이 경우에도 기본적으로 동일한 [`sgtk.util.register_publish()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.register_publish) 방식을 사용하지만 컬렉션, 유효성 확인 및 게시를 처리하는 프레임워크를 제공하여 게시 프로세스를 기반으로 빌드합니다.

## 8부: 전체 스크립트

```python
# Initialization
# ==============

import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task
tk.create_filesystem_structure("Task", context.task["id"])

# Generating a Path
# =================

# Get a template instance by providing a name of a valid template in your config's templates.yml
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"

# Get an authenticated Shotgun API instance from the engine
sg = current_engine.shotgun

# Run a Shotgun API query to summarize the maximum version number on PublishedFiles that
# are linked to the task and match the provided name.
# Since PublishedFiles generated by the Publish app have the extension on the end of the name we need to add the
# extension in our filter.
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Creating a file
# ===============

# This is the bit where you would add your own logic to copy or save a file using the path.
# In the absence of any file saving in the example, we'll use the following to create an empty file on disk.
sgtk.util.filesystem.touch_file(publish_path)

# Publishing
# ==========

# So as to match publishes created by the Publish app's, we are adding the extension to the end of the publish name.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

{% include info title="팁" content="지금까지 코드가 약간 길었으므로 다음 단계에서는 여러 방식으로 나누는 방법을 알아보겠습니다." %}

### 최종 의견

이 안내서를 통해 툴킷 API를 시작하는 방법에 대한 기본적인 이해를 갖추게 되었기를 바랍니다.
물론 API를 사용하는 다른 방법도 많이 있으므로 자세한 내용은 [tk-core API](https://developer.shotgunsoftware.com/tk-core/index.html)를 참조해 주십시오.

또한 API에 대한 질문에 답을 구하고 이 안내서에 대해 피드백을 남길 수 있는 [포럼](https://community.shotgunsoftware.com/c/pipeline/6)도 방문해 주십시오.