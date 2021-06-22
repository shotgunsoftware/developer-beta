---
layout: default
title: 두 개의 익스텐션이 설치된 경우 Photoshop 통합 문제 해결
pagename: two-photoshop-shotgun-extensions
lang: ko
---

# 두 개의 익스텐션이 설치된 경우 Photoshop 통합 문제 해결

## 어떤 문제가 있습니까?

After Effects 통합 릴리즈에는 {% include product %}와 통합되는 모든 Adobe 앱에서 사용할 수 있는 공통 플러그인이 있습니다. 이를 구현하는 과정에서, 이전 Photoshop 통합과의 호환성을 유지하고 스튜디오에서 확실하게 해당 업데이트로 전환할 수 있도록 하기 위해 익스텐션의 이름을 변경해야 했습니다.

하지만 이로 인해 업그레이드할 때 두 개의 {% include product %} 익스텐션이 설치될 수 있습니다.

![Photoshop 메뉴에 여러 개의 {% include product %} 익스텐션 표시.](/images/photoshop-extension-panel.png)

**{% include product %} Adobe Panel**은 새로운 익스텐션으로, `v1.7.0` 이상의 Photoshop 통합을 사용하는 경우에 사용해야 합니다.

## 문제를 어떻게 해결할 수 있습니까?

이전 익스텐션을 제거하려면 홈 디렉토리의 Adobe 설치 위치에서 제거할 수 있습니다. 해당 폴더는 Photoshop 시작 시 디버그 출력에서 볼 수 있으며 다음과 같습니다.

- OSX: `~/Library/Application Support/Adobe/CEP/extensions/com.sg.basic.ps`
- Windows: `%AppData%\Adobe\CEP\extensions\com.sg.basic.ps`

![Photoshop 메뉴에 여러 개의 {% include product %} 익스텐션 표시.](/images/shotgun-desktop-console-photoshop-extension.png)

Photoshop을 종료하고 해당 디렉토리를 제거하고 난 후 다시 시작하면 하나의 익스텐션만 확인됩니다.

{% include info title="참고" content="여러 환경 또는 여러 구성에 Photoshop 통합이 있고 이전 플러그인과 새 플러그인이 혼합된 경우 사용자가 이전 통합을 사용하여 Photoshop을 시작하면 이전 플러그인이 반환됩니다. 한 번에 정리할 수 있도록 전체적으로 Photoshop을 업데이트하는 것이 좋습니다." %}