---
layout: default
title: API를 통해 게시물을 작성하려면 어떻게 해야 합니까?
pagename: create-publishes-via-api
lang: ko
---

# API를 통해 게시물을 작성하려면 어떻게 해야 합니까?

sgtk API는 ShotGrid에서 `PublishedFiles` 엔티티를 등록할 수 있는 [편의 방식](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.util.register_publish)를 제공합니다.

또한 [자체 API](https://developer.shotgunsoftware.com/tk-multi-publish2/)와 함께 Publish 앱도 제공합니다.
Publish API는 궁극적으로 Core sgtk API 방식을 사용하여 PublishedFile을 등록하지만 사용자 지정 가능한 컬렉션, 유효성 확인 및 게시 관련 프레임워크도 제공합니다. 이와 관련하여 Publish API 설명서와 함께 [파이프라인 튜토리얼](https://developer.shotgridsoftware.com/cb8926fc/?title=Pipeline+Tutorial)에 자체 게시 플러그인 작성 예제도 제공합니다.

## register_publish() API 방식 사용
로우 {% include product %} API 호출을 사용하여 {% include product %}에서 게시 레코드를 생성하는 것도 가능하지만 툴킷의 편의 방식을 이용하는 것이 훨씬 좋습니다.

게시를 생성하는 모든 툴킷 앱은 [`sgtk.util.register_publish()`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.util.register_publish)라는 API 유틸리티 방식을 사용 중입니다.

기본적으로 이 방식은 {% include product %}에서 새로운 PublishedFile 엔티티를 생성하며, 툴킷 컨셉을 사용하여 생성을 쉽게 만들어줍니다. 여러분의 코드로 무언가를 실행하려면 다음의 행을 이용해야 할 것입니다.

```python
# Get access to the Toolkit API
import sgtk

# this is the file we want to publish.
file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/foreground.v034.nk"

# alternatively, for file sequences, we can just use
# a standard sequence token
# file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/renders/v034/foreground.%04d.exr"

# The name for the publish should be the filename
# without any version number or extension
name = "foreground"

# initialize an API object. If you have used the Toolkit folder creation
# to create the folders where the published file resides, you can use this path
# to construct the API object. Alternatively you can create it from any ShotGrid
# entity using the sgtk_from_entity() method.
tk = sgtk.sgtk_from_path(file_to_publish)

# use the file to extract the context. The context denotes the current work area in Toolkit
# and will control which entity and task the publish will be linked up to. If you have used the Toolkit
# folder creation to create the folders where the published file resides, you can use this path
# to construct the context.
ctx = tk.context_from_path(file_to_publish)

# alternatively, if the file you are trying to publish is not in a location that is
# recognized by toolkit, you could create a context directly from a ShotGrid entity instead:
ctx = tk.context_from_entity("Shot", 123)
ctx = tk.context_from_entity("Task", 123)

# Finally, run the publish command.
# the third parameter (file.nk) is typically the file name, without a version number.
# this makes grouping inside of ShotGrid easy. The last parameter is the version number.
sgtk.util.register_publish(
  tk,
  ctx,
  file_to_publish,
  name,
  published_file_type="Nuke Script",
  version_number=34
)
```

위에 나온 기본 항목 외에도 사용자가 직접 입력할 수 있는 여러 가지 옵션이 있습니다.
전체 매개변수 목록 및 매개변수의 기능에 대한 자세한 내용은 [Core API 설명서](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)를 참조하십시오.

{% include info title="팁" content="코드가 툴킷 앱 내에서 실행 중인 경우 `self.sgtk`를 통해 sgtk 인스턴스를 가져오고 `self.context`를 사용하여 컨텍스트를 가져올 수 있습니다.
앱 내는 아니지만 툴킷 통합이 있는 소프트웨어 내에서 실행될 경우 다음 코드를 사용하여 현재 컨텍스트 및 sgtk 인스턴스에 액세스할 수 있습니다.

```python
import sgtk
currentEngine = sgtk.platform.current_engine()
tk = currentEngine.sgtk
ctx = currentEngine.context
```
" %}