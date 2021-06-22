---
layout: default
title: 내 컨텍스트에 존재하는 값이 as_template_fields()에 누락되었습니다.
pagename: as-template-fields-missing-values
lang: ko
---

# 내 컨텍스트에 존재하는 값이 as_template_fields()에 누락되었습니다.

[as_template_fields()](https://developer.shotgunsoftware.com/tk-core/core.html?#sgtk.Context.as_template_fields) 방식은 템플릿의 키에 해당하는 폴더가 아직 생성되지 않은 경우 경로 캐시를 사용하며, 그러면 필드가 반환되지 않습니다. 이 문제는 여러 이유에서 발생할 수 있습니다.

- 템플릿 정의 및 스키마가 동기화 상태여야 합니다. 이 템플릿 정의, 또는 파이프라인 구성의 스키마를 수정했는데 둘 모두 동기화되지 않은 경우 예상한 필드가 반환되지 않습니다.
- 이 특정 컨텍스트에 대해 폴더가 생성되지 않았습니다. 아직 생성되지 않았다면 경로 캐시에 일치되는 레코드가 없게 되고, 따라서 예상한 필드가 반환되지 않습니다.
