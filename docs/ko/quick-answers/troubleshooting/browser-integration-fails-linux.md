---
layout: default
title: ShotGrid 데스크톱/브라우저 통합을 Linux에서 실행할 수 없습니다
pagename: browser-integration-fails-linux
lang: ko
---

# {% include product %} 데스크톱/브라우저 통합을 Linux에서 실행할 수 없습니다

Linux에서 {% include product %} 데스크톱을 처음 실행하면 다음 오류 메시지 중 하나가 나타날 수 있습니다. 그러면 해당 오류에 대해 아래 단계를 수행하여 오류가 해결되는지 확인해 보십시오.
여전히 해결되지 않으면 [지원 사이트](https://knowledge.autodesk.com/ko/contact-support)에서 도움을 요청하십시오.

### 목차
- [OPENSSL_1.0.1_EC 또는 HTTPSConnection 관련 문제](#openssl_101_ec-or-httpsconnection-related-issues)
- [libffi.so.5 관련 문제](#libffiso5-related-issues)
- [인증서 유효성 확인 실패 관련 문제](#certificate-validation-failed-related-issues)
- [호환되지 않는 Qt 버전](#incompatible-qt-versions)

## OPENSSL_1.0.1_EC 또는 HTTPSConnection 관련 문제

**오류**

```
importing '/opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so':
 /opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so: symbol ECDSA_OpenSSL, version OPENSSL_1.0.1_EC not defined in file libcrypto.so.10 with link time reference
AttributeError: 'module' object has no attribute 'HTTPSConnection'
```

**솔루션**

OpenSSL을 설치해야 합니다. 설치하려면 다음 명령을 관리자 권한으로 실행합니다.

```
$ yum install openssl
```

## libffi.so.5 관련 문제

**오류**

```
Browser Integration failed to start. It will not be available if you continue.
libffi.so.5: cannot open shared object file: No such file or directory
```

**솔루션**

libffi를 설치해야 합니다. 설치하려면 다음 명령을 관리자 권한으로 실행합니다.

```
yum install libffi
```

libffi를 설치했는데도 여전히 작동하지 않으면 다음 symlink를 생성한 후 {% include product %} 데스크톱을 다시 실행해 보십시오.

```
sudo ln -s /usr/lib64/libffi.so.6.0.1 /usr/lib64/libffi.so.5
```

위 방법으로 성공한 사용자들도 있고, 그렇지 못한 사용자들도 있습니다. 최신 버전 {% include product %} 데스크톱에는 웹 소켓 서버에 대한 종속성이 일부 추가되었습니다. 현재 Shotgun 팀에서 해당 내용을 살펴보고 있습니다.

## 인증서 유효성 확인 실패 관련 문제

**가능한 오류**

```
Browser Integration failed to start. It will not be available if you continue.
Error: There was a problem validating if the certificate was installed.
certutil: function failed: SEC_ERROR_BAD_DATABASE: security library: bad database.
```

**솔루션**

컴퓨터에 Google Chrome이 설치되어 있다면 이를 실행한 다음 {% include product %} 데스크톱을 다시 실행합니다. 그래도 문제가 발생하면 [지원 사이트](https://knowledge.autodesk.com/ko/contact-support)에서 도움을 요청하십시오.

Chrome이 없으면 터미널을 열고 다음 명령을 실행합니다.

```
ls -al $HOME/.pki/nssdb
```

해당 폴더가 검색되면 지원 팀에 문의하고, 다음 로그 파일의 내용을 지원 요청 티켓에 첨부해 주십시오.

```
~/.shotgun/logs/tk-desktop.log
```

폴더가 검색되지 않으면 다음을 입력합니다.

```
$ mkdir --parents ~/.pki/nssdb
$ certutil -N -d "sql:$HOME/.pki/nssdb"
```
암호를 입력하지 마십시오.

{% include product %} 데스크톱을 실행하면 이제 올바로 작동할 것입니다.

## 호환되지 않는 Qt 버전

**가능한 오류**

호환되지 않는 Qt 라이브러리(버전 `0x40805`)를 이 라이브러리(버전 `0x40807`)와 혼합할 수 없음

**솔루션**

이 오류는 대개 재정의가 발생하면서 호환되지 않는 Qt 라이브러리를 로딩하게 됨에 따라 발생합니다.
이 오류가 발생하지 않도록 하려면 다음 명령으로 환경을 수정해 보십시오.

```
unset QT_PLUGIN_PATH
```
