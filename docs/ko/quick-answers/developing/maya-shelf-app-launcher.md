---
layout: default
title: Maya에서 툴킷 앱 실행을 위한 쉘프 버튼을 추가하려면 어떻게 해야 합니까?
pagename: maya-shelf-app-launcher
lang: ko
---

# Maya에서 툴킷 앱 실행을 위한 쉘프 버튼을 추가하려면 어떻게 해야 합니까?

Maya에서 툴킷 앱을 실행하기 위한 선반 버튼을 Maya에 추가하는 작업은 정말 간단합니다. 다음은 [Loader 앱](https://support.shotgunsoftware.com/hc/ko/articles/219033078)을 여는 커스텀 쉘프 버튼을 추가하는 방법을 보여 주는 예입니다.

{% include info title="참고" content="이 예는 툴킷이 현재 Maya 세션에서 활성화되어 있다고 가정한 것입니다. 이 예제 코드는 툴킷을 부트스트랩(Bootstrap)하지 않습니다." %}

Maya에서 스크립트 편집기를 열고 다음 Python 코드를 붙여 넣습니다. 

```python
import maya.cmds as cmds

# Define the name of the app command we want to run.
# If your not sure on the actual name you can print the current_engine.commands to get a full list, see below.
tk_app = "Publish..."

try:
    import sgtk

    # get the current engine (e.g. tk-maya)
    current_engine = sgtk.platform.current_engine()
    if not current_engine:
        cmds.error("ShotGrid integration is not available!")

    # find the current instance of the app.
    # You can print current_engine.commands to list all available commands.
    command = current_engine.commands.get(tk_app)
    if not app:
        cmds.error("The Toolkit app '%s' is not available!" % tk_app)

    # now we have the command we need to call the registered callback
    command['callback']()

except Exception, e:
    msg = "Unable to launch Toolkit app '%s': %s" % (tk_app, e)
    cmds.confirmDialog(title="Toolkit Error", icon="critical", message=msg)
    cmds.error(msg)
```

이 코드를 선택하고 커스텀 쉘프로 끌어다 놓습니다. 커스텀 쉘프 버튼으로 작업하는 방법에 대한 자세한 정보는 [Maya 설명서](https://knowledge.autodesk.com/support/maya/learn-explore/caas/CloudHelp/cloudhelp/2016/ENU/Maya/files/GUID-C693E884-F81A-4858-B5D6-3856EB8F394E-htm.html)를 참조하십시오.

이 코드 예를 사용하면 위쪽에 있는 `tk_app` 및 `call_func` 값을 수정하여 Maya에서 활성화되어 있는 툴킷 앱을 실행할 수 있을 것입니다.
