---
layout: default
title: 애니메이션 파이프라인 튜토리얼
pagename: toolkit-pipeline-tutorial
lang: ko
---

# 애니메이션 파이프라인 튜토리얼

이 튜토리얼에서는 애니메이션이나 시각 효과 프로덕션을 위한 단순하면서도 전형적인 파이프라인을 빌드하는 방법을 다룹니다. 이 튜토리얼을 수행하면 에셋을 모델링에서부터 모양 개발, 그리고 프로덕션 씬으로 푸시하는 데 필요한 모든 부분을 제공하는 파이프라인을 빌드하게 됩니다.

이 파이프라인에서 다루는 대부분의 워크플로우는 Shotgun의 기본 제공 통합에서 기본적으로 작동합니다. 이 튜토리얼은 스튜디오에서 커스텀 솔루션을 많이 빌드하는 파이프라인 부분에 대해 툴킷 플랫폼을 사용하여 아티스트 워크플로우를 커스터마이즈하는 프로세스를 안내합니다.

다음은 이 튜토리얼에서 빌드할 파이프라인의 개략적인 뷰입니다.

{% include figure src="./images/tutorial/image_0.png" caption="파이프라인 개요" %}

## 파이프라인 개요

편의상 사용되는 DCC(디지털 컨텐츠 생성) 소프트웨어는 최소로 유지되며 Maya 및 Nuke로 제한됩니다. 마찬가지 이유로 파이프라인 단계 간에 전달되는 데이터는 Maya ascii 파일, Alembic 캐시 및 렌더링된 이미지 시퀀스로 제한됩니다.

{% include info title="참고" content="이 튜토리얼에 설명된 간단한 파이프라인은 프로덕션에서 테스트되지 않았으므로 Shotgun 기반 파이프라인을 빌드하는 방법에 대한 예제로만 사용해야 합니다." %}

## 필수 요건

* **작업 중인 Shotgun 프로젝트** - 이 튜토리얼에서는 프로덕션 데이터 트래킹 및 관리를 위해 Shotgun을 사용하는 환경이라고 가정합니다.

* **Shotgun 통합 이해** - Shotgun은 통합 기능을 통해 수동 구성이 필요 없는 몇 가지 간단한 프로덕션 워크플로우를 제공합니다. 이 튜토리얼에 설명된 수동 구성 및 커스터마이제이션에 대해 자세히 알아보기 전에 이러한 워크플로우의 기능 및 범위를 이해해야 합니다. Shotgun 통합에 대한 자세한 정보는 [여기](https://support.shotgunsoftware.com/hc/ko/articles/115000068574)를 참조하십시오.

* **Maya 및 Nuke 환경** - 이 튜토리얼은 Nuke와 Maya를 사용하여 간단한 파이프라인을 빌드하도록 설계되었습니다. Shotgun에서 제공하는 통합을 커스터마이즈하기 위해서는 이러한 패키지를 기본적으로 이해하고 있어야 합니다.

* **Python 실무 지식** - 이 튜토리얼에서는 Python으로 작성된 "후크"를 통해 Shotgun 통합 기능을 수정해야 합니다.

* **YAML 사용 경험** - 빌드할 파이프라인의 대부분의 구성은 YAML 파일을 수정하여 처리됩니다.

## 추가 리소스

* [Shotgun 지원 사이트](https://support.shotgunsoftware.com/hc/ko)

* [Shotgun 통합](https://www.shotgunsoftware.com/kr/integrations/)

   * [사용자 안내서](https://support.shotgunsoftware.com/hc/ko/articles/115000068574)

   * [관리자 안내서](https://support.shotgunsoftware.com/hc/ko/articles/115000067493)

   * [개발자 안내서](https://support.shotgunsoftware.com/hc/ko/articles/115000067513)

# 프로젝트 작성 및 설정

이 튜토리얼에서는 Shotgun에 새 프로젝트를 만들고 프로덕션을 위해 시작할 준비가 된 것처럼 구성해야 합니다. 여기에는 필요한 모든 Shotgun 엔티티가 제대로 배치되고 링크되어 있는지 확인하는 것도 포함됩니다. 이 튜토리얼에서는 에셋, 시퀀스, 샷 및 태스크 엔티티가 필요하며 새 프로젝트에서 기본값으로 사용할 수 있어야 합니다. 다음을 만듭니다.

* 두 개의 **에셋**:

   * **_주전자_** 캐릭터

   * **_테이블_** 소품

* 하나의 **시퀀스**

* 만든 **시퀀스**에 링크된 하나의 **샷**

* 파이프라인 단계별 **태스크**

다음은 구성된 프로젝트 엔티티가 Shotgun에서 어떻게 표시되는지 보여 주는 일부 스크린샷입니다.

{% include figure src="./images/tutorial/image_1.png" caption="주전자 및 테이블 에셋" %}

{% include figure src="./images/tutorial/image_2.png" caption="시퀀스에 링크된 샷" %}

{% include figure src="./images/tutorial/image_3.png" caption="태스크" width="400px" %}

## 소프트웨어 시작 관리자

다음으로, Shotgun 데스크톱에서 Maya 및 Nuke를 시작할 수 있는지 확인해야 합니다. 이러한 각 패키지는 데스크톱에서 해당 아이콘을 클릭하여 시작할 수 있습니다. 각 패키지의 적절한 버전이 시작되는지 확인합니다.

응용프로그램이 데스크톱에 표시되지 않거나 예상한 버전이 시작되지 않을 경우 소프트웨어 엔티티를 통해 Shotgun에서의 시작을 수동으로 구성해야 할 수 있습니다.

{% include figure src="./images/tutorial/image_4.png" caption="Shotgun에 정의된 기본 소프트웨어 엔티티" %}

소프트웨어 엔티티는 프로덕션에 사용할 DCC 패키지를 구동하는 데 사용됩니다. 기본적으로 통합은 표준 설치 위치에서 이러한 패키지를 검색하고 데스크톱을 통해 시작할 수 있도록 합니다. 둘 이상의 버전을 설치하거나 표준 위치가 아닌 곳에 설치한 경우 아티스트의 시작 환경을 조정하기 위해 Shotgun의 해당 소프트웨어 엔티티 항목을 업데이트해야 할 수도 있습니다.

소프트웨어 엔티티 및 적절한 구성 방법에 대한 자세한 정보는 [통합 관리자 안내서](https://support.shotgunsoftware.com/hc/ko/articles/115000067493-Integrations-Admin-Guide#Configuring%20software%20launches)를 참조하십시오. 예상한 방식으로 DCC가 시작되면 다음 섹션으로 진행할 수 있습니다.

# 구성

구성은 프로젝트에 대한 아티스트 워크플로우를 정의합니다. 여기에는 아티스트가 시작하는 DCC 내에 포함할 Shotgun 통합, 프로젝트의 폴더 구조 정의 방법 및 아티스트가 데이터를 공유할 때 만든 파일 및 폴더에 대한 명명 규칙 지정이 포함됩니다.

기본적으로 모든 새 프로젝트는 다양한 기성 소프트웨어 패키지를 사용하여 아티스트 간에 공유 파일에 대한 기본 워크플로우를 제공하는 기본 [Shotgun 통합](https://support.shotgunsoftware.com/hc/ko/articles/115000068574)을 사용하도록 구성됩니다. 다음 섹션에서는 프로젝트의 파이프라인 구성을 인계받고 스튜디오에 맞게 커스터마이즈하는 방법에 대해 설명합니다.

## 프로젝트 구성 설정하기

Shotgun 데스크톱(데스크톱)을 사용하여 프로젝트의 구성을 인계받습니다. 데스크톱 내에서 마우스 오른쪽 버튼을 클릭하거나 오른쪽 하단의 사용자 아이콘을 클릭하면 팝업 메뉴가 표시됩니다. **고급 프로젝트 설정...(Advanced project setup…)** 옵션을 선택하고 마법사를 수행하여 프로젝트 구성을 로컬로 설치합니다. 아래 이미지는 필요한 단계를 보여 줍니다. 통합 관리자 안내서에 설명된 [파이프라인 구성 가져오기](https://support.shotgunsoftware.com/hc/ko/articles/115000067493-Integrations-Admin-Guide#Taking%20over%20a%20Pipeline%20Configuration) 단계를 수행할 수도 있습니다.

{% include figure src="./images/tutorial/image_5.png" caption="데스크톱 팝업 메뉴에서 **고급 프로젝트 설정...(Advanced project setup…)**을 선택합니다" %}

{% include figure src="./images/tutorial/wizard_01.png" caption="**Shotgun 기본값**(Shotgun Default) 구성 유형을 선택합니다." %}

{% include figure src="./images/tutorial/wizard_02.png" caption="**기본값 구성**(Default configuration)을 선택합니다." %}

이번에 처음으로 Shotgun 프로젝트를 설정하는 경우 프로젝트 데이터의 저장소 위치를 정의하라는 메시지가 표시될 수도 있습니다.  아니면 기존 저장소 위치를 선택할 수 있습니다.

{% include figure src="./images/tutorial/wizard_03.png" caption="새 저장소를 만듭니다." %}

{% include figure src="./images/tutorial/wizard_04.png" caption="새 저장소 이름을 지정합니다.  이 저장소는 사이트 전체 수준의 저장소이며 프로젝트별 저장소가 아닙니다." %}

{% include figure src="./images/tutorial/wizard_05.png" caption="사용할 운영 체제에서 이 저장소에 액세스할 수 있는 경로를 설정합니다." %}

**사이트 기본 설정**(Site Preferences)의 **파일 관리**(File Management) 섹션에서 Shotgun 사이트에 대한 저장소를 보고 편집할 수 있습니다.  이러한 설정에 대한 자세한 내용은 [여기](https://support.shotgunsoftware.com/hc/ko/articles/219030938)에서 확인할 수 있습니다.

저장소 위치를 선택했으므로 이제 해당 위치에서 새 프로젝트에 대한 디렉토리 이름을 선택합니다.

{% include figure src="./images/tutorial/wizard_06.png" caption="프로젝트의 파일을 저장할 폴더 이름을 입력합니다." %}

이 튜토리얼에서는 중앙 집중식 구성을 사용합니다.  **분산 설정**(Distributed Setup) 옵션은 다른 혜택을 제공할 수 있는 대체 옵션을 제공하며 빠른 공유 저장소가 없는 스튜디오의 경우 자주 사용하는 옵션이 될 수 있습니다.  다른 구성 설정의 장단점에 대한 자세한 내용은 [툴킷 관리](https://www.youtube.com/watch?v=7qZfy7KXXX0&list=PLEOzU2tEw33r4yfX7_WD7anyKrsDpQY2d&index=2) 프리젠테이션에서 확인할 수 있습니다.

사이트 전체 수준의 저장소와 달리 구성은 프로젝트별로 다르므로 여기서 선택하는 디렉토리는 구성을 저장하는 데 직접 사용됩니다.

{% include figure src="./images/tutorial/wizard_07.png" caption="현재 운영 체제에 대해 선택하는 구성 경로를 기록합니다." %}

위 화면에서 선택하는 폴더가 구성이 설치될 위치입니다. 이 튜토리얼에서는 이 폴더의 구성 컨텐츠를 살펴보고 수정합니다.

위 화면에서 **설정 실행(Run Setup)**을 클릭하면 데스크톱에서 구성에 필요한 모든 구성 요소를 다운로드하고 설치하기 시작합니다. 설치 프로세스를 완료하는 데 몇 분 정도 걸릴 수 있습니다. 설치가 완료되면 전체 프로젝트 구성의 로컬 사본이 생기고 이를 다음 단계에서 수정합니다.

데스크톱 설치 튜토리얼에서 지정한 구성 위치는 Shotgun에서 해당 프로젝트에 대한 파이프라인 구성(Pipeline Configurations) 페이지에 기록됩니다.

{% include figure src="./images/tutorial/image_10.png" caption="Shotgun에서 파이프라인 구성 엔티티 복제" %}

다음 섹션을 위한 준비로 이 폴더의 컨텐츠를 숙지합니다.

## 구성 편성

간단한 파이프라인 빌드 프로세스를 시작하기 전에 파이프라인 구성 편성 및 작동 방식을 이해해야 합니다. 다음 그래프에서 구성의 주요 구성 요소 및 해당 용도를 중점적으로 설명합니다. 구성 및 관리에 대한 자세한 정보는 [툴킷 관리](https://support.shotgunsoftware.com/hc/ko/articles/219033178-Administering-Toolkit) 문서를 참조하십시오.

{% include figure src="./images/tutorial/image_11.png" %}

### 프로젝트 스키마

이 튜토리얼에서 빌드할 간단한 파이프라인은 기본 구성에서 제공되는 프로젝트 스키마를 사용합니다. **`config/core/schema`** 폴더를 검색하면 툴킷 앱에서 디스크에 파일을 작성할 때 생성되는 구조에 대해 파악할 수 있습니다. 프로젝트 디렉토리 구조 구성에 대한 자세한 정보는 [파일 시스템 구성 참조](https://support.shotgunsoftware.com/hc/ko/articles/219039868) 설명서를 참조하십시오.

### 템플릿

또한 이 튜토리얼에서는 기본 파이프라인 구성에 정의된 템플릿을 사용합니다. **`config/core/templates.yml`** 파일을 열고 디스크상의 경로에 입력 및 출력 파일을 매핑하기 위해 앱에서 가장 많이 사용되는 템플릿을 가져올 수 있습니다. 템플릿 지정 시스템에 대한 자세한 정보는 [파일 시스템 구성 참조](https://support.shotgunsoftware.com/hc/ko/articles/219039868) 설명서를 참조하십시오.

### 후크

이 튜토리얼의 대부분에는 아티스트 워크플로우를 커스터마이즈하기 위한 앱 후크 수정 작업이 포함됩니다. 이 커스터마이제이션에 대해 자세히 알아보기 전에 후크가 무엇인지, 어떻게 작동하는지, 어디에 있는지에 대한 기본적인 이해가 필요합니다. [관리](https://support.shotgunsoftware.com/hc/ko/articles/219033178#Hooks) 및 [구성](https://support.shotgunsoftware.com/hc/ko/articles/219033178#Hooks) 설명서의 후크 섹션을 참조하십시오.

튜토리얼을 진행하면서 툴킷 앱 중 하나에서 정의된 후크를 "인계"받으라는 요청을 받게 됩니다. 앱 후크를 인계받는 프로세스는 간단합니다. 해당 요청을 받을 때마다 다음 단계를 수행하기만 하면 됩니다.

1. 구성의 설치 폴더에서 재지정할 후크가 포함된 **앱을 찾습니다**. 해당 앱의 **`hooks`** 하위 디렉토리를 검색하여 재지정할 후크 파일을 찾습니다.

2. 구성의 최상위 수준 **`hooks`** 디렉토리에 **후크를 복사하고** 필요한 경우 이름을 바꿉니다.

{% include figure src="./images/tutorial/image_12.png" %}

파일이 구성의 **`hooks`** 폴더에 있기만 하면 코드를 변경하고 커스터마이즈할 수 있습니다. 해당 앱을 이 새 위치로 지정하려면 추가 단계가 필요합니다. 이 단계는 튜토리얼의 후반부에 나옵니다.

# 파이프라인 빌드

이제 파이프라인을 빌드할 준비가 되었습니다. Shotgun에서 프로젝트를 설정하고 데스크톱을 통해 Maya & Nuke를 시작할 수 있으며 프로젝트 구성을 제어했습니다. 또한 구성의 기본 구조를 이해하고 아티스트 워크플로우를 구체화할 준비가 되었습니다.

다음 섹션에서는 각 파이프라인 단계를 수행하면서 기본 제공 기능을 중점적으로 설명하고 Shotgun 통합을 커스터마이즈하는 프로세스를 안내합니다. 이 섹션을 마칠 때쯤에는 완벽한 기능을 갖춘 간단한 전체 프로덕션 파이프라인을 갖게 됩니다. 또한 아티스트가 프로덕션에서 작업할 때 수행하는 단계도 파악할 수 있습니다.

{% include info title="참고" content="이 튜토리얼의 모든 코드 및 구성은 [**`tk-config-default2`** 리포지토리](https://github.com/shotgunsoftware/tk-config-default2/tree/pipeline_tutorial/)의 **`pipeline_tutorial`** 분기에서 찾을 수 있습니다. 파일이 있을 위치, 코드를 추가할 위치 등의 힌트가 필요한 경우 이 분기를 활용하십시오." %}

## 모델링 워크플로우

간단한 파이프라인의 첫 번째 단계는 모델링입니다. 이 섹션에서는 프로젝트의 주전자 에셋에 대한 첫 번째 반복을 만듭니다. 이를 프로젝트의 폴더 구조로 디스크에 저장한 다음 게시합니다.

먼저 Shotgun 데스크톱에서 Maya를 시작합니다.

Maya가 완전히 로드되면 파일 열기(File Open) 대화상자가 나타납니다. 이 대화상자를 사용하여 프로젝트 내의 기존 Maya 파일을 찾을 수 있습니다. 또한 Shotgun 통합에서 인식하는 새 파일을 만들 수 있습니다.

에셋(Assets) 탭을 선택하고 주전자의 모델링 태스크로 찾아 들어갑니다. 이 태스크에 대한 아티스트 작업 파일이 아직 없으므로 **+ 새 파일(+ New File)** 버튼을 클릭합니다.

{% include figure src="./images/tutorial/image_13.png" %}

이 버튼을 클릭하여 비어 있는 새 Maya 세션을 만들고 현재 작업 중인 컨텍스트를 주전자 에셋의 모델 태스크로 설정합니다.

{%include info title="참고" content="이 튜토리얼에서 언제든지 Maya 또는 Nuke의 Shotgun 메뉴를 통해 Shotgun 패널을 시작할 수 있습니다. 이 패널에서는 DCC를 종료하지 않고 프로젝트 데이터에 뷰를 제공합니다. 현재 작업 중인 컨텍스트와 해당 컨텍스트 내의 최근 액티비티가 표시됩니다. 또한 패널로 직접 피드백에 대한 노트를 추가할 수 있습니다. 자세한 정보는 [Shotgun 패널 설명서](https://support.shotgunsoftware.com/hc/ko/articles/115000068574-Integrations-user-guide#The%20Shotgun%20Panel)를 참조하십시오." %}

다음으로, 주전자를 모델링하거나 제공된 주전자를 [다운로드](https://raw.githubusercontent.com/shotgunsoftware/tk-config-default2/pipeline_tutorial/resources/teapot.obj)하여 가져옵니다.

{% include figure src="./images/tutorial/image_14.png" %}

주전자 모델이 만족스러우면 **Shotgun > 파일 저장...(File Save…)** 메뉴 액션을 선택합니다. 이 대화상자에 지정된 이름, 버전 및 유형으로 파일을 저장할지 묻는 메시지가 표시됩니다.

{% include figure src="./images/tutorial/image_15.png" %}

대화상자에 전체 저장 경로를 지정하라는 메시지는 표시되지 않는데 이는 앱이 **`maya_asset_work`** 템플릿으로 저장하도록 구성되었기 때문입니다. 이 템플릿은 기본적으로 다음과 같이 정의됩니다.

**`@asset_root/work/maya/{name}.v{version}.{maya_extension}`**

토큰화된 필드, **`{name}`**, **`{version}`** 및 **`{maya_extension}`**은 모두 앱에서 전체 경로를 입력해야 하는 필드입니다. 템플릿의 **`@asset_root`** 부분은 다음으로 정의됩니다.

**`assets/{sg_asset_type}/{Asset}/{Step}`**

여기서 토큰화된 필드는 위에서 새 파일을 만들 때 설정한 현재 작업 중인 컨텍스트에 따라 툴킷 플랫폼에 의해 자동으로 추정될 수 있습니다.

또한 대화상자 하단에서 작성될 파일 이름 및 경로의 미리보기를 확인합니다. 프로젝트 구성을 설정할 때 정의한 기본 저장소 및 프로젝트 폴더가 템플릿 경로의 루트를 구성합니다.

**저장(Save)** 버튼을 클릭하여 주전자 모델을 저장합니다.

이때 주의해야 할 중요한 점은 방금 완료한 단계가 아티스트가 파이프라인 전체에서 작업 파일을 열고 저장할 때 수행할 단계와 동일하다는 점입니다. 파일 열기(File Open) 및 파일 저장(File Save) 대화상자는 Workfiles 앱의 일부입니다. 이 "다중" 앱은 Shotgun 통합에서 지원하는 모든 DCC에서 실행되며 모든 아티스트에 대해 일관된 워크플로우를 제공합니다.

다음 단계는 주전자를 약간 변경하는 것입니다. 뚜껑 지오메트리가 나중에 리깅할 수 있도록 모델의 나머지 부분과 분리되는지 확인합니다.

{% include figure src="./images/tutorial/image_16.png" %}

작업이 만족스러우면 **Shotgun > 파일 저장...(File Save…)** 메뉴 액션을 다시 실행합니다. 이번 대화상자에서는 기본적으로 버전 번호가 2로 설정됩니다. 파일 버전 자동 증분을 사용하면 아티스트가 수행한 전체 작업 내역을 관리할 수 있습니다. 저장(Save) 버튼을 클릭합니다.

{% include figure src="./images/tutorial/image_17.png" %}

주전자 모델을 버전 2로 저장하고 나면 튜토리얼의 이번 섹션에서 마지막 단계를 진행할 준비가 된 것입니다.

이제 주전자 모델이 준비되면 서페이스 처리 및 리깅할 수 있도록 게시해야 합니다. 게시하려면 **Shotgun > 게시...(Publish…)** 메뉴 액션을 클릭합니다. Publish 앱 대화상자가 표시됩니다.

{% include figure src="./images/tutorial/image_18.png" %}

대화상자에 게시될 항목의 트리가 표시됩니다. 트리에서 일부는 게시할 항목을 나타내고 일부는 게시 작업 중 수행할 액션을 나타냅니다.

대화상자의 왼쪽에 현재 Maya 세션을 나타내는 항목이 표시됩니다. 그 아래에 **Shotgun에 게시(Publish to Shotgun)** 하위 액션이 표시됩니다. **모든 세션 지오메트리(All Session Geometry)**를 나타내는 추가 항목이 현재 세션의 하위 항목으로 표시됩니다. 여기에도 **Shotgun에 게시(Publish to Shotgun)** 하위 액션이 있습니다.

{% include info title="참고" content="**모든 세션 지오메트리**(All Session Geometry) 항목이 표시되지 않으면 Maya에서 [Alembic 내보내기 플러그인이 활성화되어 있는지](https://support.shotgunsoftware.com/hc/ko/articles/219039928-Publishing-Alembic-From-Maya#Before%20You%20Begin) 확인합니다." %}

트리 왼쪽의 항목을 클릭하여 Publish 앱을 탐색합니다. 수행할 항목을 선택하면 게시되는 항목에 대한 설명을 입력할 수 있습니다. 오른쪽의 카메라 아이콘을 클릭하여 항목과 관련된 스크린샷을 찍을 수 있습니다.

준비가 되었으면 오른쪽 하단에 있는**게시(Publish)** 버튼을 클릭하여 현재 작업 파일과 주전자 지오메트리를 게시합니다. 완료되면 Shotgun에서 주전자 에셋을 검색하여 게시가 성공적으로 완료되었는지 확인할 수 있습니다.

{% include figure src="./images/tutorial/image_19.png" %}

위 이미지에서 주전자 모델이 포함되어 있는 게시된 Alembic 파일을 볼 수 있습니다. Maya 세션 파일에 대한 게시도 볼 수 있습니다. 이러한 게시는 Publish 앱의 트리 뷰에 있는 항목에 해당됩니다.

파일 저장(File Save) 대화상자 사용 시 만든 작업 파일과 마찬가지로 이러한 두 게시의 출력 경로는 템플릿으로 구동됩니다. 다음과 같이 표시됩니다(나중에 앱에서 이 템플릿이 구성된 위치를 확인할 수 있음).

**Maya 세션 게시:**

**`@asset_root/publish/maya/{name}.v{version}.{maya_extension}`**

이 템플릿은 기본적으로 작업 파일 템플릿과 매우 유사하며 유일한 차이점은 **`publish`** 폴더에 있다는 것입니다.

**에셋 게시:**

**`@asset_root/publish/caches/{name}.v{version}.abc`**

이 템플릿은 Maya 세션 게시 템플릿과 유사하지만 파일이 **`caches`** 폴더에 작성됩니다.

File Save 대화상자와 달리, 게시할 때는 이름, 버전 또는 파일 확장자 값을 제공할 필요가 없습니다. 왜냐하면 기본적으로 게시자가 작업 파일 경로에서 이러한 값을 가져오기 때문입니다. 후드에서 작업 템플릿을 통해 이러한 값을 추출한 다음 게시 템플릿에 적용합니다. 이 개념은 템플릿을 사용하여 한 파이프라인 단계의 출력을 다른 파이프라인 단계의 입력에 연결하는 방법 및 툴킷 플랫폼과 관련하여 중요한 개념입니다. 자세한 정보는 이후 섹션에서 다룹니다.

디스크에서 파일을 찾아 올바른 위치에 만들어졌는지 확인합니다.

축하합니다! 주전자의 첫 번째 게시된 반복을 성공적으로 만들었습니다. 학습한 내용을 활용하여 테이블 소품의 모델링 태스크에서 테이블 모델을 게시할 수 있는지 확인합니다. 결과는 다음과 같아야 합니다.

{% include figure src="./images/tutorial/image_20.png" %}

다음 단계는 서페이스 처리 워크플로우입니다.

## 서페이스 처리 워크플로우

이 섹션은 모델링 섹션에서 배운 내용을 기반으로 합니다. Loader 앱을 사용하여 이전 섹션에서 만든 주전자 모델을 로드하는 방법을 배우게 됩니다. 또한 Publish 앱을 커스터마이즈하여 주전자 셰이더를 게시하는 방법도 배우게 됩니다.

데스크톱에서 Maya를 실행하여 시작합니다. 이전 섹션의 작업 후에 계속 Maya가 열려 있는 경우 Maya를 다시 시작할 필요가 없습니다. Maya가 열려 있으면 **Shotgun > 파일 열기...(File Open…)** 메뉴 항목을 사용하여 Workfiles 앱을 엽니다. 모델링 섹션에서와 마찬가지로 에셋(Assets) 탭을 사용하여 주전자 에셋의 태스크로 찾아 들어갑니다. 이때 서페이스 처리 태스크를 선택하고 **+ 새 파일(+ New File)**을 클릭합니다.

{% include figure src="./images/tutorial/image_21.png" width="450px" %}

이제 주전자의 서페이스 처리 태스크에서 작업하게 됩니다. 올바른 프로덕션 컨텍스트에 있는지 쉽게 확인하는 방법은 Shotgun 메뉴의 첫 번째 항목을 확인하는 것입니다.

{% include figure src="./images/tutorial/image_22.png" %}

다음으로 주전자 모델을 새 서페이스 처리 작업 파일로 로드해야 합니다. 이렇게 하려면 Maya의 **Shotgun > 로드...(Load…)** 메뉴 항목을 통해 Loader 앱을 시작합니다.

{% include figure src="./images/tutorial/image_23.png" %}

Loader 앱의 레이아웃은 Workfiles 앱과 유사하지만 지금은 작업 파일을 여는 대신 게시된 파일을 검색하여 로드합니다.

에셋(Assets) 탭에서 주전자 캐릭터를 찾아 이전 섹션에서 만든 주전자 게시를 표시합니다. Maya 씬 및 Alembic 캐시 게시를 볼 수 있습니다. Alembic 캐시 게시를 선택하면 대화상자 오른쪽에 상세 정보가 표시됩니다. 그런 다음 Alembic 캐시 게시의 액션(Actions) 메뉴에서 **참조 만들기(Create Reference)** 항목을 클릭합니다. 기본적으로 추가 액션을 수행할 수 있게 로더가 계속 열려 있지만 로더를 닫고 계속 진행할 수 있습니다. 모델링 태스크에서 주전자 게시를 가리키는 참조가 생성된 것을 Maya에서 볼 수 있습니다.

{% include figure src="./images/tutorial/image_24.png" %}

다음으로, 주전자에 간단한 절차 셰이더를 추가합니다.

{% include figure src="./images/tutorial/image_25.png" %}

파이프라인을 빌드할 때 셰이더 관리는 시간이 오래 걸리는 복잡한 태스크일 수 있습니다. 이는 특히 스튜디오에만 해당되는 경우가 많습니다. 왜냐하면 제공되는 Maya 통합에서 셰이더 또는 텍스처 관리 기본 기능을 처리하지 않기 때문입니다.

계속하기 전에 **Shotgun > 파일 저장...(File Save…)** 메뉴 액션을 사용하여 현재 세션을 저장합니다.

### 커스텀 셰이더 게시

이 간단한 파이프라인의 목적에 맞게 게시자 앱을 커스터마이즈하여 서페이스 처리 단계에서 추가 게시 항목으로 Maya 셰이더 네트워크를 내보냅니다. 튜토리얼 후반에서 다운스트림 참조 시 셰이더를 Alembic 지오메트리 캐시에 다시 연결할 수 있는 빠르고 간편한 솔루션을 만듭니다.


{% include info title="참고" content="추가할 커스터마이즈는 확실히 매우 간단하고 쉽습니다. 보다 강력한 솔루션을 위해 외부 이미지를 텍스처 맵으로 사용하는 에셋 관리 측면뿐만 아니라 서페이스 처리된 캐릭터의 대체 표현을 고려해야 할 수 있습니다. 이 예제는 실제 솔루션을 빌드하기 위한 시작점만 제공합니다." %}

{% include info title="참고" content="게시자 플러그인을 작성하는 방법에 대한 전체 상세 정보는 [여기](https://developer.shotgunsoftware.com/tk-multi-publish2/)에서 확인할 수 있습니다." %}

#### Maya 컬렉터 재지정

먼저 Publish 앱의 컬렉션 로직을 수정해야 합니다. 게시자는 앱에 게시하고 표시할 "수집" 항목에 대한 로직을 정의하는 컬렉터 후크로 구성됩니다. 프로젝트 구성 내의 다음 파일에서 구성된 앱의 설정을 찾을 수 있습니다.

**`env/includes/settings/tk-multi-publish2.yml`**

이 파일은 모든 아티스트 환경 내에서 Publish 앱이 사용되는 방법을 정의합니다. 파일을 열고 **Maya** 섹션, 특히 **에셋 단계**에 대한 구성을 검색합니다. 다음과 같이 표시됩니다.

{% include figure src="./images/tutorial/image_26.png" %}

컬렉터 설정은 게시자의 컬렉션 로직이 존재하는 후크를 정의합니다. 기본적으로 이 값은 다음과 같습니다.

**`collector: "{self}/collector.py:{engine}/tk-multi-publish2/basic/collector.py"`**

이 정의에는 두 개의 파일이 포함되어 있습니다. 후크 설정에 여러 파일이 나열된 경우 상속을 의미합니다. 첫 번째 파일에는 설치된 Publish 앱의 후크 폴더로 평가할 **`{self}`** 토큰이 포함되어 있습니다. 두 번째 파일에는 현재 엔진(이 경우 설치된 Maya 엔진)의 후크 폴더로 평가할 **`{engine}`** 토큰이 포함되어 있습니다. 요약하면, 이 값은 Maya 고유의 컬렉터에 Publish 앱의 컬렉터가 상속됨을 의미합니다. 앱의 컬렉터 후크는 실행 중인 DCC에 관계없이 유용한 로직을 갖기 때문에 이러한 형태가 게시자 구성의 일반적인 패턴입니다. DCC 고유 로직은 해당 기본 로직으로부터 상속받고 기본 로직을 확장하여 현재 세션에 해당되는 항목을 수집합니다.

{% include info title="참고" content="에셋 단계 환경에 대한 컬렉터 설정만 변경하므로 다른 컨텍스트(예: 샷 단계)에서 작업 중인 아티스트는 변경 사항을 볼 수 없습니다. 아티스트는 제공되는 기본 Maya 컬렉터를 계속 사용합니다." %}

**구성** 섹션에서 후크를 인계받는 방법을 배웠습니다. 구성에 Maya 엔진의 컬렉터 후크를 인계받아 커스터마이제이션 프로세스를 시작합니다.

{% include figure src="./images/tutorial/image_27.png" %}

위의 이미지는 이 작업을 수행하는 방법을 보여 줍니다. 먼저 프로젝트 구성의 **hooks** 폴더에 폴더 구조를 만듭니다. 나중에 다른 DCC에 대해 동일한 후크를 재지정할 수 있으므로 이렇게 하면 컬렉터 플러그인에 약간의 네임스페이스를 제공합니다. 다음으로, install 폴더에서 Maya 엔진의 컬렉터 후크를 새 후크 폴더 구조로 복사합니다. 이제 구성에 다음 경로를 가진 Maya 컬렉터 사본이 생겼습니다.

**`config/hooks/tk-multi-publish2/maya/collector.py`**

다음으로, 새 후크 위치를 가리키도록 publish2 설정 파일을 업데이트합니다. 이제 컬렉터 설정은 다음 값을 갖습니다.

**`collector: "{self}/collector.py:{config}/tk-multi-publish2/maya/collector.py"`**

**`{config}`** 토큰을 확인합니다. 이제 경로가 프로젝트 구성에서 hooks 폴더로 해석됩니다. 새 컬렉터 사본이 앱 자체에서 정의되는 컬렉터에서 상속됩니다.

{% include info title="참고" content="이때 게시했으면 게시 로직은 새 위치에서 간단하게 복사하고 참조한 컬렉터와 정확히 동일합니다." %}

이제 원하는 IDE 또는 텍스트 편집기에서 컬렉터 사본을 연 다음 **`process_current_session`** 방식을 찾습니다. 이 방식은 현재 DCC 세션에서 모든 게시 항목 수집을 담당합니다. 새 게시 유형을 수집하게 되므로 이 방식의 하단으로 이동하여 다음 줄을 추가합니다.

**`self._collect_meshes(item)`**

이 줄은 현재 세션에 있는 모든 메쉬를 수집하기 위해 추가할 새 방식입니다. 이 방식은 나중에 만들 예정인 셰이더 게시 플러그인이 작동할 수 있는 메쉬 항목을 만듭니다. 전달되는 항목은 메쉬 항목의 상위가 될 세션 항목입니다.

{% include info title="참고" content="이 방식은 기존 게시 플러그인을 수정하는 매우 직접적인 접근 방법입니다. 게시자 구조 및 움직이는 모든 부분에 대한 자세한 정보는 [개발자 문서](http://developer.shotgunsoftware.com/tk-multi-publish2/)를 참조하십시오." %}

이제 파일 하단에 아래 새 방식 정의를 추가합니다.

```python
    def _collect_meshes(self, parent_item):
       """
       Collect mesh definitions and create publish items for them.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "mesh.png"
       )

       # iterate over all top-level transforms and create mesh items
       # for any mesh.
       for object in cmds.ls(assemblies=True):

           if not cmds.ls(object, dag=True, type="mesh"):
               # ignore non-meshes
               continue

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.mesh) that will be
           # used by an associated shader publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name (the group name). In the future, other publish
           # plugins might attach to these mesh items to publish other things
           mesh_item = parent_item.create_item(
               "maya.session.mesh",
               "Mesh",
               object
           )

           # set the icon for the item
           mesh_item.set_icon_from_path(icon_path)

           # finally, add information to the mesh item that can be used
           # by the publish plugin to identify and export it properly
           mesh_item.properties["object"] = object
```

코드가 주석 처리되어 있으며 수행할 작업에 대한 정보를 제공합니다. 요점은 이제 현재 세션에서 최상위 수준 메쉬에 대한 메쉬 항목을 수집하는 로직을 추가한 것입니다. 그러나 이때 게시자를 실행하면 항목 트리에서 어떤 메쉬 항목도 볼 수 없습니다. 왜냐하면 작동하도록 정의된 게시 플러그인이 없기 때문입니다. 다음으로, 메쉬 항목을 연결하고 다운스트림을 사용하기 위해 해당 항목 게시를 처리할 새 셰이더 게시 플러그인을 작성합니다.

{% include info title="참고" content="위 코드에는 메쉬 항목에 대한 아이콘을 설정하는 호출이 있습니다. 이 작업을 위해 다음과 같이 지정된 경로로 구성에 아이콘을 추가해야 합니다." %}

**`config/hooks/tk-multi-publish2/icons/mesh.png`**

#### 셰이더 게시 플러그인 만들기

다음 단계에서는 메쉬의 셰이더를 디스크로 내보내 게시할 수 있는 게시 플러그인에 새로 수집한 메쉬 항목을 연결합니다. 이렇게 하려면 새 게시 플러그인을 만들어야 합니다. [이 후크의 소스 코드에 대한 이 링크를 따라 이동하고](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_shader_network.py) 이를 **`hooks/tk-multi-publish2/maya`** 폴더에 저장하고 이름을 **`publish_shader_network.py`**로 지정합니다.

{% include info title="참고" content="이 플러그인은 툴킷 플랫폼 및 게시 코드를 처음 사용하는 경우 사용할 코드 집합입니다. 지금은 걱정할 필요가 없습니다. 이 튜토리얼을 진행하면서 진행되는 상황을 이해하고 게시자 기능을 사용하게 됩니다. 지금은 파일을 생성하고 그 용도가 셰이더 네트워크를 디스크에 기록하는 것이라는 점만 알아 두시면 됩니다." %}

셰이더를 게시하기 전의 마지막 단계는 새 셰이더 게시 플러그인에서 정의한 템플릿 및 구성을 추가하는 것입니다. **`settings`** 속성에서 플러그인에 의해 정의된 설정을 다음과 같이 볼 수 있습니다.

```python
    @property
    def settings(self):
       "”” … "””

       # inherit the settings from the base publish plugin
       plugin_settings = super(MayaShaderPublishPlugin, self).settings or {}

       # settings specific to this class
       shader_publish_settings = {
           "Publish Template": {
               "type": "template",
               "default": None,
               "description": "Template path for published shader networks. "
                              "Should correspond to a template defined in "
                              "templates.yml.",
           }
       }

       # update the base settings
       plugin_settings.update(shader_publish_settings)

       return plugin_settings
```


이 방식은 플러그인에 대한 구성 인터페이스를 정의합니다. **"게시 템플릿(Publish Template)"** 설정에는 디스크에 셰이더 네트워크를 작성하는 플러그인이 필요합니다. 새 게시 플러그인을 게시자 구성에 추가하고 템플릿 설정을 포함합니다. 이는 컬렉터를 인계받기 전에 수정한 구성 블록과 동일합니다. 해당 내용은 다음 파일에 정의되어 있습니다.

**`env/includes/settings/tk-multi-publish2.yml`**

이제 구성은 다음과 같습니다.

{% include figure src="./images/tutorial/image_28.png" %}

마지막으로, 구성에 새 **`maya_shader_network_publish`** 템플릿을 정의해야 합니다. 다음 파일을 편집하여 추가합니다.

**`config/core/templates.yml`**

에셋 관련 Maya 템플릿이 정의된 섹션을 찾아 새 템플릿 정의를 추가합니다. 정의는 다음과 같습니다.

{% include figure src="./images/tutorial/image_29.png" %}

모든 것이 정의되었습니다. 셰이더를 게시할 메쉬를 찾기 위해 Publish 앱의 컬렉터 후크를 재지정했습니다. 수집한 셰이더 항목에 연결할 새 게시 플러그인을 구현하고 디스크에 셰이더 네트워크를 작성할 새 게시 템플릿을 정의하고 구성했습니다.

{% include info title="참고" content="구성을 커스터마이즈하는 동안 Maya를 닫아도 괜찮습니다. 간단히 Maya를 다시 시작하고 파일 열기(File Open) 대화상자를 사용하여 서페이스 처리 작업 파일을 열 수 있습니다. 아래의 다시 로드 단계는 건너뛸 수 있습니다." %}

##### Shotgun 통합 다시 로드

커스터마이제이션을 시도하려면 Maya 세션에서 통합을 다시 로드해야 합니다. 이렇게 하려면 **Shotgun > [태스크 이름(Task Name)] > 작업 영역 정보...(Work Area Info…)** 메뉴 액션을 클릭합니다.

{% include figure src="./images/tutorial/image_30.png" %}

현재 컨텍스트에 대한 정보를 제공하는 Work Area Info 앱이 시작됩니다. 구성을 변경할 때 통합을 다시 로드할 수 있는 유용한 버튼도 있습니다. 버튼을 클릭하여 앱 및 엔진을 다시 로드한 다음 대화상자를 닫습니다.

{% include figure src="./images/tutorial/image_31.png" %}

### 셰이더 네트워크 게시

이제 프로젝트 구성 변경 결과를 보겠습니다. Shotgun 메뉴에서 Publish 앱을 시작합니다. 다음과 같이 **Publish Shaders** 플러그인이 연결된 수집된 주전자 메쉬 항목이 표시됩니다.

{% include figure src="./images/tutorial/image_32.png" %}

작업 설명을 입력하고 게시된 파일과 연결할 서페이스 처리된 주전자의 썸네일을 캡처합니다. 마지막으로, 게시(Publish)를 클릭하여 주전자 셰이더를 디스크로 내보내고 파일을 Shotgun의 게시로 등록합니다. 완료되면 세션 게시 플러그인이 자동으로 작업 파일을 다음 사용 가능한 버전으로 저장했는지 확인합니다. 지금까지가 Shotgun 통합에서 지원되는 모든 DCC 내의 기본 동작입니다.


이제 Shotgun에서 주전자 에셋을 검색하여 모두 예상대로 작동되는지 확인할 수 있습니다.

{% include figure src="./images/tutorial/image_33.png" %}

축하합니다! 성공적으로 파이프라인을 커스터마이즈하고 주전자에 대한 셰이더를 게시했습니다. 학습한 내용을 활용하여 테이블 소품의 서페이스 처리 태스크에서 셰이더를 게시할 수 있는지 확인합니다. 결과는 다음과 같아야 합니다.

{% include figure src="./images/tutorial/image_34.png" %}

다음 단계는 리깅 워크플로우입니다.

## 리깅 워크플로우

이제는 Shotgun에서 제공하는 Workfile 및 Publish 앱을 사용하여 작업 파일을 열거나 만들고 저장하고 게시하는 것이 어렵지 않을 것입니다. 또한 Loader 앱을 사용하여 업스트림에서 게시를 로드하기도 했습니다. 학습한 내용을 활용하여 다음 태스크를 완료합니다.

* Shotgun 데스크톱에서 Maya 시작

* 주전자 에셋의 리깅 단계에서 새 작업 파일 만들기

* 모델링 단계에서 주전자 alembic 캐시 게시 로드(참조)

* 주전자 뚜껑을 리깅하여 열고 닫기(간단하게 유지)

* 주전자 리그 저장 및 게시

Shotgun에 다음과 같이 나타납니다.

{% include figure src="./images/tutorial/image_35.png" %}

다음으로, 아티스트가 해당 워크플로우에서 업스트림 변경을 처리하는 방법을 알아보겠습니다. 모델링 작업 파일을 열고 주전자 모델을 약간 변경합니다. 그런 다음 업데이트된 작업을 게시합니다. 결과는 다음과 같습니다.

{% include figure src="./images/tutorial/image_36.png" %}

주전자의 리깅 단계에서 작업 파일을 다시 엽니다(**Shotgun > 파일 열기...(File Open…)**). 이제 **Shotgun > 씬 분할...(Scene Breakdown…)** 메뉴 액션을 시작합니다. 작업 파일에 참조한 모든 업스트림 게시를 보여 주는 Breakdown 앱이 시작됩니다. 이 경우 업스트림 주전자 모델만 있습니다. 다음과 같이 표시됩니다.

{% include figure src="./images/tutorial/image_37.png" width="400px" %}

앱은 각 참조에 대해 두 개의 표시기 중 하나를 표시합니다. 초록색 체크 표시는 참조된 게시가 최신 버전임을 나타내고 빨간색 "x"는 사용 가능한 최신 게시물이 있음을 나타냅니다. 이 경우에는 사용 가능한 최신 게시물이 있음을 확인할 수 있습니다.

이제 참조된 주전자 Alembic 캐시 항목을 선택한 다음(또는 하단의 **모든 빨간색 항목 선택(Select All Red)** 버튼 클릭) **선택 항목 업데이트(Update Selected)**를 클릭합니다.

앱에서 Maya 참조가 주전자 Alembic 캐시의 최신 반복으로 업데이트됩니다. 이제 파일에 새 모델이 표시됩니다.

{% include figure src="./images/tutorial/image_40.png" width="400px" %}

새 모델에 고려해야 하는 리깅 설정을 조정한 다음 변경 사항을 게시합니다.

다음 섹션에서는 샷 컨텍스트에서 작업합니다. 다음 단계는 샷 레이아웃입니다.

## 레이아웃 워크플로우

이 섹션에서는 프로젝트에 대해 만든 샷에서 작업을 시작합니다. 이전 섹션에서 만든 에셋을 로드하고 샷을 실행해 봅니다. 그런 다음 게시자를 다시 커스터마이즈하고 이번에는 샷 카메라를 게시합니다.

이전 섹션에서 학습한 내용을 활용하여 다음 태스크를 완료합니다.

* Shotgun 데스크톱에서 Maya 시작

* 샷의 레이아웃 단계에서 새 작업 파일 만들기(힌트: Loader에서 샷(Shots) 탭 사용)

* 주전자의 리깅 단계에서 주전자 게시 로드(참조)

* 테이블의 모델 단계에서 테이블 게시 로드(참조)

이제 테이블에 주전자가 있는 간단한 씬을 실행합니다. 씬에 **camMain**이라는 카메라를 추가하고 몇 프레임에 애니메이션을 적용하여 샷의 카메라 이동을 만듭니다.

{% include figure src="./images/tutorial/image_41.gif" %}

샷 레이아웃이 만족스러우면 **Shotgun > 파일 저장...(File Save…)** 메뉴 액션을 통해 파일을 저장합니다. 이때 계속 진행하여 게시하면 게시할 수 있는 항목으로 전체 Maya 세션만 표시됩니다.

파이프라인에 추가하여 많은 유연성을 제공하는 쉬운 커스터마이제이션은 다른 패키지에 쉽게 가져올 수 있는 파일 형식으로 독립 실행형 카메라를 게시하는 기능입니다. 이렇게 하면 일반적으로 레이아웃에 카메라를 한 번 생성한 다음 애니메이션, 조명 및 합성 등의 다른 모든 파이프라인 단계를 수행하고 직접 사용할 수 있습니다.

### 카메라 수집

셰이더 게시에서와 같이 첫 번째 단계는 컬렉터 후크를 커스터마이즈하는 것입니다. 이미 Maya에 대한 컬렉터 후크를 인계받고 에셋 단계에 구성했습니다. 이제 샷 파이프라인 단계에 대한 구성을 업데이트해야 합니다. 이렇게 하려면 게시자의 구성 파일을 수정하고 Maya 샷 단계 컬렉터 설정을 편집합니다.

{% include figure src="./images/tutorial/image_42.png" %}

이제 샷 컨텍스트 내의 태스크에서 작업할 때 커스텀 컬렉터 로직이 실행됩니다. 다음 단계에서는 커스텀 카메라 컬렉션 로직을 추가합니다.

커스텀 컬렉터 후크를 열고 서페이스 처리 섹션에서 메쉬를 수집하는 호출을 추가한 **`process_current_session`** 방식의 하단에 다음 방식 호출을 추가합니다.

**`self._collect_cameras(item)`**

다음으로 파일 하단에 다음과 같이 방식 자체를 추가합니다.

```python
    def _collect_cameras(self, parent_item):
       """
       Creates items for each camera in the session.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "camera.png"
       )

       # iterate over each camera and create an item for it
       for camera_shape in cmds.ls(cameras=True):

           # try to determine the camera display name
           try:
               camera_name = cmds.listRelatives(camera_shape, parent=True)[0]
           except Exception:
               # could not determine the name, just use the shape
               camera_name = camera_shape

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.camera) that will be
           # used by an associated camera publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name. In the future, other publish plugins might attach to
           # these camera items to perform other actions
           cam_item = parent_item.create_item(
               "maya.session.camera",
               "Camera",
               camera_name
           )

           # set the icon for the item
           cam_item.set_icon_from_path(icon_path)

           # store the camera name so that any attached plugin knows which
           # camera this item represents!
           cam_item.properties["camera_name"] = camera_name
           cam_item.properties["camera_shape"] = camera_shape
```

다시 한 번 언급하지만 코드가 주석 처리되어 있으며 수행할 작업에 대한 정보를 제공합니다. 현재 세션의 모든 카메라에 대한 카메라 항목을 수집하는 로직을 추가했습니다. 이전과 마찬가지로, 이때 게시자를 실행하면 항목 트리에서 어떤 카메라 항목도 볼 수 없습니다. 왜냐하면 작동하도록 정의된 게시 플러그인이 없기 때문입니다. 다음으로, 이러한 항목을 연결하고 다운스트림을 사용하기 위해 해당 항목 게시를 처리할 카메라 게시 플러그인을 작성합니다.

{% include info title="참고" content="위 코드에는 카메라 항목에 대한 아이콘을 설정하는 호출이 있습니다. 이 작업을 위해 다음과 같이 지정된 경로로 구성에 아이콘을 추가해야 합니다." %}

**`config/hooks/tk-multi-publish2/icons/camera.png`**

### 커스텀 카메라 게시 플러그인

다음 단계에서는 메쉬의 셰이더를 디스크로 내보내 게시할 수 있는 게시 플러그인에 새로 수집한 메쉬 항목을 연결합니다. 이렇게 하려면 새 게시 플러그인을 만들어야 합니다. [이 후크의 소스 코드에 대한 이 링크를 따라 이동하고](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_camera.py) 이를 **`hooks/tk-multi-publish2/maya`** 폴더에 저장하고 이름을 **`publish_camera.py`**로 지정합니다.

### 카메라 게시 구성

마지막으로, 샷 단계에 대한 Publish 앱의 구성을 업데이트해야 합니다. 설정 파일을 편집하여 새 플러그인을 추가합니다.

**`env/includes/settings/tk-multi-publish2.yml`**

이제 구성은 다음과 같습니다.

{% include figure src="./images/tutorial/image_43.png" %}

새 플러그인의 **`settings`** 방식에서 정의한 대로 파일에 추가된 두 개의 설정을 확인할 수 있습니다. 셰이더 플러그인과 마찬가지로 카메라 파일이 작성될 위치를 정의하는 **게시 템플릿(Publish Template)** 설정이 있습니다. 카메라 설정은 플러그인이 작동해야 할 카메라를 구동하는 카메라 문자열의 목록입니다. 즉, 일부 유형의 카메라 명명 규칙이 있고 이 설정을 사용하면 사용자가 규칙과 일치하지 않는 카메라의 항목을 게시할 수 없습니다. 위의 이미지에서는 **`camMain`** 카메라만 게시할 수 있도록 제공됩니다. 추가한 플러그인의 구현은 **`cam*`** 같이 와일드카드 패턴으로도 작동합니다.

변경 사항을 테스트하기 전 마지막 단계는 새 카메라 게시 템플릿에 대한 정의를 추가하는 것입니다. **`config/core/templates.yml`** 파일을 편집하고 Maya 샷 템플릿 섹션에 템플릿 정의를 추가합니다.

{% include figure src="./images/tutorial/image_44.png" %}

이제 새 플러그인을 사용하여 카메라를 게시할 수 있습니다. **Work Area Info** 앱을 사용하여 통합을 다시 로드한 다음 게시자를 시작합니다.

{% include figure src="./images/tutorial/image_45.png" %}

이미지에서 볼 수 있듯이, 새 카메라 항목이 수집되고 게시 플러그인이 연결됩니다. 계속 진행하여 **게시(Publish)**를 클릭하여 디스크에 카메라를 작성하고 Shotgun에 등록합니다.

{% include info title="참고" content="Alembic 내보내기와 유사하게 카메라 게시 플러그인을 사용하려면 FBX 내보내기 플러그인을 로드해야 합니다.  카메라 게시 플러그인 항목이 표시되지 않으면 FBX 플러그인이 로드되었는지 확인하고 게시자를 다시 시작합니다." %}

Shotgun에 다음과 같이 나타납니다.

{% include figure src="./images/tutorial/image_46.png" %}

이제 완료되었습니다. 다음 단계는 애니메이션입니다.

## 애니메이션 워크플로우

지금까지는 커스텀 파일 유형/컨텐츠를 디스크에 작성하고 다른 파이프라인 단계에서 공유하기 위해 Publish 앱만 커스터마이즈했습니다. 이 섹션에서는 Loader 앱의 구성을 커스터마이즈하여 커스텀 게시를 가져오거나 참조할 수 있도록 하는 작업을 완료합니다.

이전 섹션에서 학습한 내용을 활용하여 다음 태스크를 완료합니다.

* Shotgun 데스크톱에서 Maya 시작

* 샷의 애니메이션 단계에서 새 작업 파일 만들기

* 샷의 레이아웃 단계에서 Maya 세션 게시 로드(참조)

{% include info title="참고" content="레이아웃 세션 게시 파일에 카메라가 포함되었음을 확인할 수 있습니다. 강력한 파이프라인에서 별도의 카메라 게시 파일이 하나의 실제 카메라 정의가 될 수 있도록 세션 게시에서 카메라를 명시적으로 숨기거나 제외할 수 있습니다. 계속 진행하여 참조로 포함된 카메라를 삭제하거나 숨깁니다." %}

### 커스텀 카메라 로더 액션

카메라 게시를 가져오거나 참조하도록 Loader 앱을 커스터마이즈하려면 앱의 설정 파일을 편집해야 합니다. 구성에서 파일 경로는 다음과 같습니다.

**`config/env/includes/settings/tk-multi-loader2.yml`**

Maya에 대해 앱이 구성된 섹션을 찾고 **`action_mappings`** 설정의 액션 목록에 다음 줄을 추가합니다.

**`FBX Camera: [reference, import]`**

커스텀 카메라 게시 플러그인에서, 디스크에 카메라를 작성하는 데는 Maya의 **`FBXExport`** mel 명령이 사용되었으며 Shotgun에 파일을 등록하는 데 사용된 게시 유형은 **`FBX Camera`**입니다. 설정에 추가한 줄은 **`FBX Camera`** 유형의 게시에 대해 **`reference`** 및 **`import`** 액션을 표시하도록 로더에 지시합니다. 이러한 액션은 Loader 앱의 [tk-maya-actions.py](https://github.com/shotgunsoftware/tk-multi-loader2/blob/master/hooks/tk-maya_actions.py) 후크에 정의되어 있습니다. 이러한 액션은 Maya가 참조하거나 가져올 수 있는 모든 파일 유형을 처리하는 방식으로 구현됩니다. 커스텀 플러그인에 의해 생성된 **`.fbx`** 파일은 이 범주에 해당하므로 이 작업이 게시된 카메라를 로드하는 데 필요한 유일한 변경 사항입니다.

이제 앱 설정은 다음과 같습니다.

{% include figure src="./images/tutorial/image_47.png" width="400px" %}

이제 **Work Area Info** 앱을 통해 통합을 다시 로드하고 새 설정을 선택한 다음 레이아웃에서 게시된 카메라를 찾습니다.

{% include figure src="./images/tutorial/image_48.png" %}

새 게시 유형별로 필터링한 다음 카메라에 대한 참조를 만듭니다. 로더를 닫고 새 참조 카메라를 사용하여 이전 섹션에서 만든 카메라 모션을 다시 재생할 수 있습니다.

다음으로, 주전자 모델이 무언가를 수행하는 애니메이션을 적용합니다(간단하게 유지).

{% include figure src="./images/tutorial/image_49.gif" %}

애니메이션이 만족스러우면 이전 섹션에서와 같이 작업 파일을 저장하고 게시합니다.

다음 단계는 조명입니다.

## 조명 워크플로우

이 섹션에서는 이전 섹션에서 게시한 모든 것을 함께 가져와 샷을 렌더링합니다. 이렇게 하려면 주전자 에셋의 서페이스 처리 단계에서 게시한 셰이더를 로드하도록 Loader 앱을 커스터마이즈합니다.

먼저, 이전 섹션에서 학습한 내용을 활용하여 다음 태스크를 완료합니다.

* Shotgun 데스크톱에서 Maya 시작

* 샷의 조명 단계에서 새 작업 파일 만들기

* 샷의 애니메이션 단계에서 Maya 세션 게시 로드(참조)

* 샷의 레이아웃 단계에서 카메라 게시 로드(참조)

### 커스텀 셰이더 로더 액션

서페이스 처리 단계에서 게시한 셰이더를 로드하려면 이전 섹션에서 언급한 **`tk-maya-actions.py`** 후크를 인계받아야 합니다. 해당 후크를 설치 위치에서 구성으로 복사합니다.

{% include figure src="./images/tutorial/image_50.png" %}

이 후크는 지정된 게시에 대해 수행할 수 있는 액션 목록 생성을 담당합니다. Loader 앱은 제공된 통합에서 지원되는 각 DCC에 대해 이 후크와 다른 버전을 정의합니다.

서페이스 처리 워크플로우 섹션에서 게시된 셰이더는 바로 Maya 파일이므로 내보낸 카메라와 같이 기존 로직을 변경하지 않고 로더에서 참조할 수 있습니다. 유일하게 변경해야 할 사항은 셰이더가 파일에 참조된 후 적절한 메쉬에 셰이더를 연결하도록 액션 후크에 새 로직을 추가하는 것입니다.

액션 후크의 마지막에(클래스 외부) 다음 방식을 추가합니다.

```python
    def _hookup_shaders(reference_node):
       """
       Reconnects published shaders to the corresponding mesh.
       :return:
       """

       # find all shader hookup script nodes and extract the mesh object info
       hookup_prefix = "SHADER_HOOKUP_"
       shader_hookups = {}
       for node in cmds.ls(type="script"):
           node_parts = node.split(":")
           node_base = node_parts[-1]
           node_namespace = ":".join(node_parts[:-1])
           if not node_base.startswith(hookup_prefix):
               continue
           obj_pattern = node_base.replace(hookup_prefix, "") + "\d*"
           obj_pattern = "^" + obj_pattern + "$"
           shader = cmds.scriptNode(node, query=True, beforeScript=True)
           shader_hookups[obj_pattern] = node_namespace + ":" + shader

       # if the object name matches an object in the file, connect the shaders
       for node in (cmds.ls(references=True, transforms=True) or []):
           for (obj_pattern, shader) in shader_hookups.iteritems():
               # get rid of namespacing
               node_base = node.split(":")[-1]
               if re.match(obj_pattern, node_base, re.IGNORECASE):
                   # assign the shader to the object
                   cmds.select(node, replace=True)
                   cmds.hyperShade(assign=shader)
```


이제 셰이더 연결 로직을 호출하기 위해 **`_create_reference`** 방식 마지막에 다음 두 줄을 추가합니다.

```python
    reference_node = cmds.referenceQuery(path, referenceNode=True)
    _hookup_shaders(reference_node)</td>
```


코드는 새 참조가 만들어질 때마다 실행되므로 셰이더가 이미 파일에 있는 경우 새 지오메트리 참조 시 셰이더를 할당해야 합니다. 마찬가지로, 셰이더를 참조하고 지오메트리가 이미 존재할 때 작동합니다.

{% include info title="참고" content="이 연결 로직은 매우 억지스럽고 프로덕션에서 바로 사용할 수 있는 파이프라인을 구현할 때 고려해야 할 네임스페이스 및 기타 Maya와 관련된 미묘한 문제를 적절하게 처리하지 못합니다." %}

마지막으로, 다음 파일을 편집하여 샷의 로더 설정을 새 후크로 지정합니다.

**`config/env/includes/settings/tk-multi-loader2.yml`**

또한 Maya 셰이더 네트워크 게시 유형을 참조 액션과 연결합니다. 이제 로더 설정은 다음과 같습니다.

{% include figure src="./images/tutorial/image_51.png" %}

이제 **Work Area Info** 앱을 통해 통합을 다시 로드하여 새 설정을 선택한 다음 서페이스 처리에서 게시된 셰이더를 검색합니다.

주전자 셰이더 네트워크 게시에 대한 참조를 만듭니다.

{% include figure src="./images/tutorial/image_52.png" %}

이제 테이블 셰이더 네트워크를 로드합니다. Maya에서 하드웨어 렌더링을 켜면 셰이더가 애니메이션 단계에서 자동으로 메쉬 참조에 연결해야 합니다.

{% include figure src="./images/tutorial/image_53.png" %}

이제 씬에 약간의 조명을 추가합니다(간단하게 유지).

{% include figure src="./images/tutorial/image_54.png" %}

### Maya 렌더 게시

디스크에 샷을 렌더링합니다.

{% include figure src="./images/tutorial/image_54_5.gif" %}

{% include info title="참고" content="여기에서 볼 수 있듯이 주전자 및 테이블 에셋 둘 다 서페이스 처리에 문제가 있습니다. 이 튜토리얼에서는 의도된 예술적 선택이었다고 가정합니다. 이러한 문제를 해결하려면 언제든지 이러한 에셋의 서페이스 처리 작업 파일을 로드하고 셰이더를 조정하여 다시 게시할 수 있습니다. 이 경우, 조명 작업 파일의 참조를 업데이트하고 다시 렌더링해야 합니다. 단계를 진행하면 참조를 다시 로드한 후 Breakdown 앱이 업데이트된 셰이더를 다시 연결하지 않는다는 사실을 알 수 있습니다. 셰이더 참조를 연결하도록 로더를 수정한 경험을 기반으로 필요한 로직을 추가하도록 Breakdown 앱의 씬 작업 후크를 업데이트할 수 있어야 합니다. 힌트: [이 파일](https://github.com/shotgunsoftware/tk-multi-breakdown/blob/master/hooks/tk-maya_scene_operations.py#L69)의 업데이트 방식을 참조하십시오." %}

제공된 Shotgun 통합은 파일에 정의된 렌더 레이어를 확인하여 이미지 시퀀스를 수집합니다. 렌더가 완료되면 게시자를 시작합니다. 렌더링된 시퀀스가 트리의 항목으로 표시됩니다.

{% include figure src="./images/tutorial/image_55.png" %}

계속 진행하여 세션 및 렌더링된 이미지 파일 시퀀스를 게시합니다. Shotgun에 다음과 같이 나타납니다.

{% include figure src="./images/tutorial/image_56.png" %}

다음 단계는 합성입니다.

## 합성 워크플로우

이 마지막 튜토리얼 섹션에서는 Nuke에서 제공하는 기본 통합의 일부를 소개합니다. 이전 섹션에서 확인한 앱 외에도 Shotgun 인식 쓰기 노드와 리뷰를 위해 다른 작업자에게 빠르게 렌더를 보낼 수 있는 앱에 대해 알아봅니다.

다음 단계를 수행하여 작업 파일을 준비합니다.

* Shotgun 데스크톱에서 Nuke를 시작합니다.

* Maya에서와 마찬가지로, Shotgun > 파일 열기...(File Open…) 메뉴 액션을 사용하여 샷의 합성 단계에서 새 작업 파일을 만듭니다.


Loader 앱을 통해 이전 섹션에서 렌더링 및 게시된 이미지 시퀀스를 로드합니다.

{% include figure src="./images/tutorial/image_57.png" %}

**`Image`** 및 **`Rendered Image`** 게시 유형(유형은 파일 확장자에 따라 다름)에 정의된 액션은 **읽기 노드 만들기(Create Read Node)**입니다. 이 액션을 클릭하여 Nuke 세션에 새 **`Read`** 노드를 만듭니다.

Nuke 프로젝트 설정 출력 형식이 렌더링된 이미지와 일치하는지 확인합니다. 배경으로 사용할 균일 색상을 만들고 읽기 노드와 병합합니다. 합성을 볼 뷰어를 연결합니다.

{% include figure src="./images/tutorial/image_58.png" %}

합성에 만족하면 **Shotgun > 파일 저장...(File Save…)** 메뉴 액션을 사용하여 작업 파일을 저장합니다.

다음으로, Nuke의 왼쪽 메뉴에 있는 Shotgun 로고를 클릭합니다. 해당 메뉴에서 Shotgun 인식 쓰기 노드 중 하나를 클릭합니다.

{% include figure src="./images/tutorial/image_59.png" width="400px" %}

Shotgun Write Node 앱에서는 기본 제공 Nuke Write node의 맨 위에 현재 Shotgun 컨텍스트를 기반으로 출력 경로를 자동으로 평가하는 레이어를 제공합니다.

{% include figure src="./images/tutorial/image_60.png" %}

디스크로 이미지 프레임을 렌더링합니다. 이제 Nuke 세션을 게시하여 렌더링된 이미지와 작업 파일을 연결할 수 있습니다. 기본적으로 게시자는 렌더링된 프레임을 수집하고 플러그인을 연결하여 Shotgun으로 프레임을 등록합니다. 두 번째 플러그인은 백그라운드에서 실행되는 리뷰 제출이라는 통합 방식으로 리뷰할 프레임을 업로드합니다. 이 앱은 Nuke를 사용하여 업로드되어 리뷰에 사용할 수 있는 Quicktime을 생성합니다.

{% include figure src="./images/tutorial/image_61.png" %}

또 다른 유용한 통합은 Quick Review 앱입니다. 이 앱은 Quicktime을 빠르게 생성하고 리뷰를 위해 Shotgun에 업로드하는 출력 노드입니다. 이 앱은 Shotgun Write Node 옆의 왼쪽 메뉴에서 찾을 수 있습니다.

{% include figure src="./images/tutorial/image_62.png" width="400px" %}

빠른 리뷰 노드를 만든 다음 Upload 버튼을 클릭하여 디스크로 입력을 렌더링하고 Quicktime을 생성하고 리뷰를 위해 결과를 Shotgun에 업로드합니다. 프레임을 제출하기 전에 몇 가지 표준 옵션이 제공됩니다.

{% include figure src="./images/tutorial/image_63.png" %}

업로드된 Quicktime을 모두 보려면 Shotgun의 미디어(Media) 탭을 확인합니다.

{% include figure src="./images/tutorial/image_64.png" %}

Shotgun의 미디어 리뷰에 대한 자세한 정보는 [공식 설명서](https://support.shotgunsoftware.com/hc/ko/sections/204245448-Review-and-approval)를 참조하십시오.

# 결론

축하합니다. 모두 완료했습니다. 이 튜토리얼이 Shotgun 통합을 사용하여 고유한 커스텀 파이프라인을 빌드하는 시작점이 되었기를 바랍니다. 기본 통합을 확장하여 스튜디오의 특정 요구 사항을 충족하는 방법을 이해할 수 있어야 합니다.

[shotgun-dev Google 그룹](https://groups.google.com/a/shotgunsoftware.com/forum/?fromgroups&hl=ko#!forum/shotgun-dev)에서 다른 스튜디오에서는 툴킷을 어떻게 사용하는지 질문하고 배울 수 있습니다.  최신 게시물을 보려면 등록하십시오.

기본 통합에서 제공되지 않는 기능이나 워크플로우가 있으면 언제든 자체 앱을 작성할 수 있습니다. 첫 번째 앱 작성을 시작하려면 [이 문서](https://support.shotgunsoftware.com/hc/ko/articles/219033158)를 참조하십시오.

언제나처럼 이 튜토리얼에 대한 추가 질문이나 Shotgun 또는 툴킷 플랫폼에 대한 일반적인 질문이 있으면 언제든지 [티켓을 제출해 주십시오](https://support.shotgunsoftware.com/hc/ko/requests/new).
