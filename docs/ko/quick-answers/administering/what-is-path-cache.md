---
layout: default
title: 경로 캐시가 무엇입니까? 파일 시스템 위치가 무엇입니까?
pagename: what-is-path-cache
lang: ko
---

# 경로 캐시가 무엇입니까? 파일 시스템 위치가 무엇입니까?

경로 캐시는 툴킷이 디스크의 폴더와 {% include product %}의 엔티티 간 연결을 트래킹하는 데 사용합니다. 마스터 캐시는 `FilesystemLocation` 엔티티 유형을 사용하여 {% include product %}에 저장됩니다. 그러면 각 사용자는 [디스크의 툴킷 캐시 디렉토리에 로컬로 저장](./where-is-my-cache.md)되는 각자의 경로 캐시 버전을 갖게 되고, 이 경로 캐시는 응용프로그램이 시작되거나 폴더가 생성될 때마다 백그라운드에서 동기화됩니다.

보통 경로 캐시는 수동으로 수정하지 않는 것이 좋습니다. 저희의 내부 프로세스를 통해 여러분의 로컬 캐시가 {% include product %}의 파일 시스템 위치 엔티티와 동기화될 뿐 아니라, 모든 사용자의 컴퓨터가 {% include product %}와 동기화 상태를 유지할 수 있도록 이벤트 로그 항목도 생성됩니다.

경로 캐시를 수정하는 데 사용할 수 있는 몇 가지 tank 명령이 있습니다.

- `tank unregister_folders` 명령은 경로 캐시 연결을 제거합니다.
- `tank synchronize_folders` 명령은 로컬 경로 캐시와 {% include product %}의 동기화를 실행합니다.

보통은 이 명령을 실행할 필요가 없지만 상황에 따라서는 실행하는 것이 유용할 수 있습니다.
예를 들어, `unregister_folders` 명령은 프로젝트 내 엔티티의 이름을 바꾸거나 엔티티를 다시 생성하기 전에 실행해야 합니다.