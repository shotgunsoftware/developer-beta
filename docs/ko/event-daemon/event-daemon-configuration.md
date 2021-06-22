---
layout: default
title: 구성
pagename: event-daemon-configuration
lang: ko
---

# 구성

다음 안내서는 스튜디오에 대한 {% include product %}Events 구성을 지원합니다.

{% include product %}Events에 대한 대부분의 구성은 `shotgunEventDaemon.conf` 파일에 의해 제어됩니다. 이 파일에는 요구사항에 따라 변경할 수 있는 여러 가지 설정이 있습니다. 대부분의 기본값은 대부분의 스튜디오에서 올바르게 작동하지만, 몇몇 설정은 반드시 구성해야 합니다(특히 {% include product %}EventDaemon이 {% include product %} 서버에 연결할 수 있도록 {% include product %} 서버 URL, 스크립트 이름, 응용 프로그램 키 등).

{% include info title="참고" content="**Windows:** Windows 사용자는 Windows에서 해당하는 구성 파일의 모든 경로를 변경해야 합니다. 로깅을 포함한 모든 경로를 단일 위치로 유지해 단순하게 관리하는 것이 좋습니다. 이 문서에서는 Windows 경로를 설명할 때 `C:\shotgun\shotgunEvents`를 주로 사용합니다." %}

<a id="Edit_shotgunEventDaemon_conf"></a>
## Edit shotgunEventDaemon.conf

{% include product %}Events를 설치했다면 다음 단계는 텍스트 편집기에서 `shotgunEventDaemon.conf` 파일을 열고 스튜디오의 요구사항에 맞게 설정을 수정하는 것입니다. 대부분의 스튜디오에는 기본값이 적합하지만, 데몬을 실행하기 위해서 반드시 제공해야 할 기본값이 없는 설정도 일부 있습니다.

*반드시* 제공해야 하는 항목은 다음과 같습니다.

- {% include product %} 서버 URL
- {% include product %}에 연결하기 위한 스크립트 이름 및 응용프로그램 키
- 실행할 {% include product %}EventDaemon 플러그인에 대한 전체 경로

선택적으로, SMTP 서버 및 이메일 관련 설정을 지정하여 오류 이메일 알림을 설정할 수도 있습니다. 이 작업은 선택 사항이지만 이 값을 설정하도록 선택하면 이메일 섹션의 모든 구성 값을 입력해야 합니다.

데몬에서 성능 문제가 발생한 경우 문제 해결에 도움이 될 수 있는 선택적 타이밍 로그에 대한 섹션도 있습니다. 타이밍 로깅을 활성화하면 자체 개별 로그 파일에 타이밍 정보가 입력됩니다.

<a id="Shotgun_Settings"></a>
### {% include product %} 설정

`[{% include product %}]` 섹션 아래에서 기본 토큰을 `server`, `name` 및 `key`에 대한 올바른 값으로 변경합니다. 이러한 값은 {% include product %}에 연결하는 표준 API 스크립트에 제공한 값과 같아야 합니다.

예

```
server: https://awesome.shotgunstudio.com
name: {% include product %}EventDaemon
key: e37d855e4824216573472846e0cb3e49c7f6f7b1
```

<a id="Plugin_Settings"></a>
### 플러그인 설정

실행할 플러그인을 찾을 위치를 {% include product %}EventDaemon에 알려야 합니다. `[plugins]` 섹션 아래에서 기본 토큰을 `paths`에 대한 올바른 값으로 변경합니다.

여러 위치를 지정할 수 있습니다(데몬을 사용하는 부서나 리포지토리가 여러 개인 경우 유용할 수 있음). 이 값은 읽을 수 있는 기존 디렉토리에 대한 전체 경로여야 합니다.

예

```
paths: /usr/local/shotgun/{% include product %}Events/plugins
```

처음 시작할 때 테스트하기에 적절한 플러그인은 `logArgs.py` 디렉토리에 있는 `/usr/local/shotgun/{% include product %}Events/src/examplePlugins` 플러그인입니다. 이를 지정한 플러그인 폴더로 복사하고 이 폴더를 사용하여 테스트를 진행합니다.

<a id="Location_of_shotgunEventDaemon_conf"></a>
### shotgunEventDaemon.conf의 위치

기본적으로 데몬은 {% include product %}EventDaemon.py가 있는 디렉토리와 `/etc` 디렉토리에서 shotgunEventDaemon.conf 파일을 찾습니다. conf 파일을 다른 디렉토리에 저장해야 하는 경우 현재 디렉토리에서 해당 위치에 대한 심볼릭 링크를 생성하는 것이 좋습니다.

{% include info title="참고" content="위 방법이 효과가 없는 경우 config 파일의 검색 경로는 `shotgunEventDaemon.py` 스크립트 하단의 `_getConfigPath()` 함수에 위치합니다." %}

{% include info title="참고" content="**Windows의 경우** Windows에 `/etc`가 존재하지 않으므로 구성 파일은 Python 파일과 동일한 디렉터리에 있어야 합니다." %}

<a id="Testing_the_Daemon"></a>
## 데몬 테스트

데몬은 백그라운드에서 실행되기 때문에 테스트하기가 어려울 수 있습니다. 항상 어떻게 작동하고 있는지 확인할 수 있는 명확한 방법이 없습니다. 다행스럽게도 {% include product %}EventDaemon에는 이를 포그라운드 프로세스로 실행할 수 있는 옵션이 있습니다. 이제 최소 필수 설정을 완료했으므로 데몬을 테스트하고 어떻게 되는지 확인해 보겠습니다.

{% include info title="참고" content="여기에 사용된 기본값에는 루트 액세스 권한이 필요합니다(예: /var/log 디렉터리에 쓰려는 경우). 제공된 사용 예에서는 이를 위해 `sudo`를 사용하여 실행됩니다." %}

```
$ sudo ./{% include product %}EventDaemon.py foreground
INFO:engine:Using {% include product %} version 3.0.8
INFO:engine:Loading plugin at /usr/local/shotgun/{% include product %}Events/src/examplePlugins/logArgs.py
INFO:engine:Last event id (248429) from the {% include product %} database.
```

스크립트를 시작할 때 위의 행이 표시됩니다(일부 세부 사항은 다를 수 있음). 스크립트를 포그라운드에서 실행하기로 선택했기 때문에 오류가 발생하면 스크립트가 종료됩니다. 중단되면 몇몇 일반적인 오류가 아래에 표시됩니다.

`logArgs.py` 플러그인은 {% include product %}에서 발생한 이벤트를 찾아 로거로 전달합니다. 아주 흥미진진하지는 않지만 스크립트가 실행되고 플러그인이 작동하게 하는 간단한 방법입니다. 분주한 스튜디오에 있다면 이미 빠르게 진행되는 메시지의 흐름을 알아차리셨을 것입니다. 그렇지 않은 경우 웹 브라우저에서 {% include product %} 서버에 로그인하여 값을 변경하거나 값을 생성합니다. 터미널 창에 변경 사항으로 생성된 이벤트 유형에 해당하는 로그 구문이 출력됩니다.

{% include info title="참고" content="logArgs.py 파일에는 적절한 값을 입력해야 하는 변수가 있습니다. 로깅이 올바르게 작동하려면 shotgunEventDaemon.conf 파일에 사용된 것과 동일한 값을 포함하도록 '$DEMO_SCRIPT_NAMES$' 및 '$DEMO_API_KEY$'를 편집해야 합니다." %}

로그 파일에 로깅된 내용이 보이지 않는 경우 {% include product %}EventDaemon.conf에서 로그 관련 설정을 확인하여 ``logging`` 값이 로그 INFO 수준 메시지로 설정되어 있고

```
logging: 20
```

logArgs 플러그인도 INFO 수준 메시지를 표시하도록 구성되어 있는지 확인하십시오. registerCallbacks() 메서드의 끝에는 읽어야 하는 행이 있습니다.

```python
reg.logger.setLevel(logging.INFO)
```

모두 올바르게 표시된다고 가정하면, {% include product %}EventDaemon 프로세스를 중지하려는 경우 터미널에 `<ctrl>-c`을 입력하면 스크립트가 종료되는 것을 확인할 수 있습니다.

<a id="Running_the_Daemon"></a>
## 데몬 실행

테스트 시 모든 것이 순조로웠다고 가정하면, 이제 의도대로 백그라운드에서 데몬을 실행할 수 있습니다.

```
$ sudo ./{% include product %}EventDaemon.py start
```

결과가 표시되지 않으며 터미널에서 컨트롤이 반환되어야 합니다. 두 가지 방법으로 제대로 실행되고 있는지 확인할 수 있습니다. 첫 번째는 실행 중인 프로세스를 확인하고 이것이 그중 하나인지 확인하는 것입니다.

```
$ ps -aux | grep shotgunEventDaemon
kp              4029   0.0  0.0  2435492    192 s001  R+    9:37AM   0:00.00 grep shotgunEventDaemon
root            4020   0.0  0.1  2443824   4876   ??  S     9:36AM   0:00.02 /usr/bin/python ./{% include product %}EventDaemon.py start
```

반환된 두 번째 행에서 데몬이 실행 중임을 알 수 있습니다. 첫 번째 행은 방금 실행한 명령과 일치합니다. 따라서 실행되고 있음을 알 수는 있지만 플러그인이 *제대로* 작동하고 있으며 의도된 역할을 수행하고 있는지 확인하기 위해 로그 파일을 확인해 출력된 내용이 있는지 알아볼 수 있습니다.

```
$ sudo tail -f /var/log/shotgunEventDaemon/shotgunEventDaemon
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
```

웹 브라우저로 돌아가서 엔티티를 약간 변경합니다. 그런 다음 터미널로 다시 돌아가 출력된 내용이 있는지 확인합니다. 다음과 같이 표시되어야 합니다.

```
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
2011-09-09 09:45:31,228 - plugin.logArgs.logArgs - INFO - {'attribute_name': 'sg_status_list', 'event_type': 'Shotgun_Shot_Change', 'entity': {'type': 'Shot', 'name': 'bunny_010_0010', 'id': 860}, 'project': {'type': 'Project', 'name': 'Big Buck Bunny', 'id': 65}, 'meta': {'entity_id': 860, 'attribute_name': 'sg_status_list', 'entity_type': 'Shot', 'old_value': 'omt', 'new_value': 'ip', 'type': 'attribute_change'}, 'user': {'type': 'HumanUser', 'name': 'Kevin Porterfield', 'id': 35}, 'session_uuid': '450e4da2-dafa-11e0-9ba7-0023dffffeab', 'type': 'EventLogEntry', 'id': 276560}
```

출력의 정확한 세부 정보는 다를 수 있지만, 플러그인이 의도된 작업을 수행해 이벤트가 로그 파일에 로깅된 것이 확인되어야 합니다. 로그 파일에 로깅된 내역이 보이지 않을 경우, ``logging`` 값이 INFO 수준 메시지를 로깅하도록 설정되어 있고 logArgs 플러그인도 INFO 수준 메시지를 표시하도록 구성되어 있는지 확인하기 위해 {% include product %}EventDaemon.conf에서 로그 관련 설정을 확인하십시오.

<a id="A_Note_About_Logging"></a>
### 로깅에 대한 참고 사항

로그 회전은 {% include product %} 데몬의 기능이라는 점에 유의하십시오. 로그는 매일 밤 자정에 회전되며 플러그인당 10개의 일일 파일이 보관됩니다.

<a id="Common_Errors"></a>
## 일반적인 오류

다음은 직면할 수 있는 몇몇 일반적인 오류와 그 해결 방법에 대한 몇 가지 설명입니다. 문제가 발생하면 [지원 사이트](https://knowledge.autodesk.com/ko/contact-support)에서 도움을 요청하십시오.

### 잘못된 경로: $PLUGIN_PATHS$

shotgunEventDaemon.conf 파일에서 플러그인 경로를 지정해야 합니다.

### 권한이 거부됨: '/var/log/shotgunEventDaemon'

쓰기 작업을 위해 데몬에서 로그 파일을 열 수 없습니다.

`sudo`로 데몬을 실행하거나 ShotgunEventDaemon.conf의 `logPath` 및 `logFile` 설정에서 지정한 로그 파일에 쓸 수 있는 권한이 있는 사용자로 데몬을 실행해야 할 수 있습니다. (기본 위치는 일반적으로 루트 소유의 `/var/log/shotgunEventDaemon`입니다.)

### ImportError: 이름이 shotgun_api3인 모듈 없음

{% include product %} API가 설치되어 있지 않습니다. 현재 디렉토리에 있거나 `PYTHONPATH`의 디렉토리에 있어야 합니다.

sudo로 실행해야 하는데 `PYTHONPATH`가 올바르게 설정되어 있는 경우, sudo가 환경 변수를 재설정한다는 점을 기억하십시오. `PYTHONPATH`를 유지하거나 sudo -e(?)를 실행하도록 sudoers 파일을 편집할 수 있습니다.

<a id="List_of_Configuration_File_Settings"></a>
## 구성 파일 설정 리스트

<a id="Daemon_Settings"></a>
### 데몬 설정

다음은 일반적인 데몬 작동 설정입니다.

**pidFile**

pidFile은 데몬이 실행 중인 동안 해당 프로세스 ID를 저장하는 위치입니다. 데몬이 실행되는 동안 이 파일이 제거되면 다음 이벤트 처리 루프를 거친 후 완전히 종료됩니다.

디렉토리가 이미 있으며 쓰기 가능해야 합니다. 원하는 대로 파일 이름을 지정할 수 있지만 실행 중인 프로세스와 일치하는 기본 이름을 사용하는 것이 좋습니다.

```
pidFile: /var/log/shotgunEventDaemon.pid
```

**eventIdFile**

eventIdFile은 데몬이 마지막으로 처리된 {% include product %} 이벤트의 ID를 저장할 위치를 나타냅니다. 이를 통해 데몬은 마지막 종료 시 중지된 지점으로 복귀할 수 있으므로 이벤트가 누락되지 않습니다. 데몬이 마지막으로 종료된 이후의 이벤트를 무시하려면 데몬을 시작하기 전에 이 파일을 제거해 시작된 후 새로 생성된 이벤트만 데몬이 처리하게 합니다.

이 파일은 *각* 플러그인에 대한 마지막 이벤트 ID를 추적하여 이 정보를 pickle 형식으로 저장합니다.

```
eventIdFile: /var/log/shotgunEventDaemon.id
```

**logMode**

로깅 모드는 다음 두 값 중 하나로 설정할 수 있습니다.

- **0** = 기본 로그 파일에 모든 로그 메시지 기록
- **1** = 엔진에 대해 하나의 기본 파일, 플러그인당 하나의 파일

값 **1**을 사용할 때 엔진에서 생성되는 로그 메시지는 `logFile` 구성 설정에서 지정한 기본 로그 파일에 로깅됩니다. 플러그인이 로깅하는 모든 메시지는 `plugin.<plugin_name>`라는 파일에 저장됩니다.

```
logMode: 1
```

**logPath**

로그 파일을 저장할 경로(기본 엔진 및 플러그인 로그 파일 모두) 기본 로그 파일의 이름은 아래의 ``logFile`` 설정에 의해 제어됩니다.

```
logPath: /var/log/shotgunEventDaemon
```

{% include info title="참고" content="shotgunEventDaemon에 이 디렉터리에 대한 쓰기 권한이 있어야 합니다. 일반적인 설정에서는 시스템이 시작될 때 데몬이 자동으로 실행되도록 설정되고 해당 시점에 루트 권한이 부여됩니다." %}

**logFile**

기본 데몬 로그 파일의 이름입니다. 로깅은 매일 밤 자정에 회전하는 로그 파일을 최대 10개까지 저장하도록 구성되어 있습니다.

```
logFile: shotgunEventDaemon
```

**logging**

로그 파일로 전송된 로그 메시지의 임계값 수준입니다. 이 값은 기본 디스패치 엔진의 기본값이며 플러그인별 기준으로 재정의될 수 있습니다. 이 값은 Python 로깅 모듈로 전달됩니다. 가장 일반적인 값은 다음과 같습니다.
- **10:** 디버그
- **20:** 정보
- **30:** 경고
- **40:** 오류
- **50:** 중요

```
logging: 20
```

**timing_log**

이 값을 `on`로 설정하여 타이밍 로깅을 활성화하면, 데몬의 성능 문제를 더 간단히 해결할 수 있게 해주는 타이밍 정보가 있는 별도의 로그 파일이 생성됩니다.

각 콜백 호출에 대해 제공되는 타이밍 정보는 다음과 같습니다.

- **event_id** 콜백을 트리거한 이벤트의 ID
- **created_at** 이벤트가 {% include product %}에서 생성된 시점의 ISO 형식 타임스탬프
- **callback** `plugin.callback` 형식으로 호출된 콜백의 이름
- **start** 콜백 처리 시작 시점의 ISO 형식 타임스탬프
- **end** 콜백 처리 종료 시점의 ISO 형식 타임스탬프
- **duration** 콜백 처리 시간의 `DD:HH:MM:SS.micro_second` 형식 지속 시간
- **error** 콜백의 실패 여부 값은 `False` 또는 `True`일 수 있습니다.
- **delay** 이벤트 생성과 콜백별 처리 시작 시간 사이의 지연 시간(`DD:HH:MM:SS.micro_second` 형식)

**conn_retry_sleep**

{% include product %}에 대한 연결이 실패할 경우 다시 연결을 시도할 때까지 대기하는 시간(초)입니다. 이를 통해 간헐적인 네트워크 문제, 서버 재시작, 응용프로그램 유지 관리 작업 등이 발생할 수 있습니다.

```
conn_retry_sleep = 60
```

**max_conn_retries**

오류 수준 메시지를 로깅하기 전에 연결을 다시 시도하는 횟수입니다(아래의 이메일 알림이 구성된 경우 이메일을 보낼 수 있음).

```
max_conn_retries = 5
```

**fetch_interval**

각 이벤트 배치가 처리된 후 새 이벤트를 요청할 때까지 대기할 시간(초)입니다. 이 설정은 일반적으로 조정할 필요가 없습니다.

```
fetch_interval = 5
```

<a id="Shotgun_Settings"></a>
### {% include product %} 설정

다음은 {% include product %} 인스턴스와 관련된 설정입니다.

**server**

연결할 {% include product %} 서버의 URL입니다.

```
server: %(SG_ED_SITE_URL)s
```

{% include info title="참고" content="여기에는 기본값이 없습니다. `SG_ED_SITE_URL` 환경 변수를 ShotGrid 서버의 URL(예: https://awesome.shotgunstudio.com)로 설정하십시오." %}

**name**

{% include product %}EventDaemon이 연결해야 하는 {% include product %} 스크립트 이름입니다.

```
name: %(SG_ED_SCRIPT_NAME)s
```

{% include info title="참고" content="여기에는 기본값이 없습니다. `SG_ED_SCRIPT_NAME` 환경 변수를 ShotGrid 서버의 스크립트 이름(예: `shotgunEventDaemon`)" %}

**key**

위에 지정된 스크립트 이름에 대한 {% include product %} 응용프로그램 키입니다.

```
key: %(SG_ED_API_KEY)s
```

{% include info title="참고" content="여기에는 기본값이 없습니다. `SG_ED_API_KEY` 환경 변수를 위의 스크립트 이름에 대한 응용프로그램 키로 설정하십시오(예:`0123456789abcdef0123456789abcdef01234567`)." %}

**use_session_uuid**

{% include product %} 인스턴스의 모든 이벤트에서 session_uuid를 설정해 플러그인이 생성한 모든 이벤트에 전파합니다. 이렇게 하면 {% include product %} UI가 플러그인의 결과로 발생하는 업데이트를 표시할 수 있습니다.

```
use_session_uuid: True
```

- 이 기능을 사용하려면 {% include product %} server v2.3 이상이 필요합니다.
- 이 기능을 사용하려면 {% include product %} API v3.0.5 이상이 필요합니다.

{% include info title="참고" content="ShotGrid UI는 원래 이벤트를 생성한 브라우저 세션에 *대해서만* 업데이트를 실시간으로 표시합니다. 동일한 페이지가 열려 있는 다른 브라우저 창에는 실시간 업데이트가 표시되지 않습니다." %}

<a id="Plugin_Settings_details"></a>
### 플러그인 설정

**paths**

프레임워크가 로드할 플러그인을 찾아야 하는 쉼표로 구분된 전체 경로 목록입니다. 상대 경로를 사용하지 마십시오.

```
paths: /usr/local/shotgun/plugins
```

{% include info title="참고" content="여기에는 기본값이 없습니다. 플러그인 파일(예: Windows의 경우 `/usr/local/shotgun/shotgunEvents/plugins` 또는 `C:\shotgun\shotgunEvents\plugins`)의 위치로 값을 설정해야 합니다." %}

<a id="Email_Settings"></a>
### 이메일 설정

로그를 지속적으로 추적하기보다는 활성 알림 시스템을 가지고 있을 것이라는 점을 알고 있으므로 이 방식이 오류 보고에 사용됩니다.

아래의 모든 설정이 제공되는 경우 레벨 40(ERROR)보다 높은 오류가 이메일을 통해 보고됩니다.

이메일 알림이 전송되려면 이 모든 값을 입력해야 합니다.

**server**

SMTP 연결에 사용해야 하는 서버입니다. SMTP 연결에 대한 자격 증명을 제공하기 위해 사용자 이름 및 암호 값의 주석 처리를 해제할 수 있습니다. 서버에서 인증을 사용하지 않는 경우에는 `username` 및 `password`에 대한 설정에 주석 처리를 해야 합니다.

```
server: smtp.yourdomain.com
```

{% include info title="참고" content="여기에는 기본값이 없습니다. smtp.yourdomain.com 토큰을 SMTP 서버의 주소로 대체해야 합니다(즉, `smtp.mystudio.com`)." %}

**username**

SMTP 서버에 인증이 필요한 경우 이 행의 주석 처리를 제거하고 SMTP 서버에 연결하는 데 필요한 사용자 이름으로 `SG_ED_EMAIL_USERNAME` 환경 변수를 구성했는지 확인하십시오.

```
username: %(SG_ED_EMAIL_USERNAME)s
```

**password**

SMTP 서버에 인증이 필요한 경우 이 행의 주석 처리를 제거하고 SMTP 서버에 연결하는 데 필요한 암호로 `SG_ED_EMAIL_PASSWORD` 환경 변수를 구성했는지 확인하십시오.

```
password: %(SG_ED_EMAIL_PASSWORD)s
```

**from**

이메일에 사용되어야 하는 보낸 사람 주소입니다.

```
from: support@yourdomain.com
```

{% include info title="참고" content="여기에는 기본값이 없습니다. `support@yourdomain.com`을 유효한 값으로 대체해야 합니다(예: `noreply@mystudio.com`)." %}

**to**

알림을 받을 이메일 주소의 쉼표로 구분된 목록입니다.

```
to: you@yourdomain.com
```

{% include info title="참고" content="여기에는 기본값이 없습니다. `you@yourdomain.com`을 유효한 값으로 대체해야 합니다(예: `shotgun_admin@mystudio.com`)." %}

**subject**

{% include product %} 이벤트 프레임워크에서 보낸 알림을 정렬할 수 있도록 메일 클라이언트가 사용할 수 있는 이메일 제목 접두어입니다.

```
subject: [SG]
```
