---
layout: default
title: "Python API에서 SSL: CERTIFICATE_VERIFY_FAILED 문제 수정"
pagename: fix-ssl-certificate-verify-failed
lang: ko
---

# Python API에서 SSL: CERTIFICATE_VERIFY_FAILED 문제 수정

Python API는 {% include product %}에서 사용하는 다양한 웹 서비스에 연결하기 위해 사용자 시스템에 있고 API와 함께 번들로 제공되는 인증서 목록을 사용합니다. 그러나 새 인증서 인증 기관이 릴리즈되어 Python API 또는 OS와 함께 번들로 제공되지 않을 수 있습니다.

Python API는 최신 인증서 사본을 제공하지만 2019년 2월 21일부터 최신 API 버전을 사용하는 경우에도 API에서 Amazon S3 업로드에 대해 해당 인증서를 사용하지 못하도록 하는 버그가 있습니다. 자세한 내용은 [이 AWS 블로그 게시물](https://aws.amazon.com/blogs/security/how-to-prepare-for-aws-move-to-its-own-certificate-authority/)을 참조하십시오. 다음 솔루션을 통해 상황을 일시적으로 해결할 수 있습니다.

{% include info title="참고" content="이 방법은 임시적인 해결 방법이며 장기적 솔루션을 찾고 있는 중입니다." %}

## 기본 솔루션

필요한 CA 인증서를 Windows 인증서 저장소에 추가합니다. Windows 7 사용자가 이 솔루션을 사용하려면 먼저 [PowerShell 3.0으로 업그레이드](https://docs.microsoft.com/ko/office365/enterprise/powershell/manage-office-365-with-office-365-powershell)하거나 [certutil](https://docs.microsoft.com/ko/windows-server/administration/windows-commands/certutil)을 사용하여 [필요한 인증서](https://www.amazontrust.com/repository/SFSRootCAG2.cer)를 추가해야 할 수 있습니다.

1. **시작**을 마우스 오른쪽 버튼으로 클릭한 다음 **Windows PowerShell(관리자)** 항목을 마우스 왼쪽 버튼으로 클릭하여 승격된 PowerShell을 시작합니다.

2. PowerShell 창에 다음 명령을 붙여넣은 후 Return 키를 눌러 실행합니다.

        $cert_url = "https://www.amazontrust.com/repository/SFSRootCAG2.cer"
        $cert_file = New-TemporaryFile
        Invoke-WebRequest -Uri $cert_url -UseBasicParsing -OutFile $cert_file.FullName
        Import-Certificate -FilePath $cert_file.FullName -CertStoreLocation Cert:\LocalMachine\Root

3. 추가된 인증서에 포함된 자세한 지문 `925A8F8D2C6D04E0665F596AFF22D863E8256F3F`가 표시되면 작업이 완료된 것이며 PowerShell을 닫을 수 있습니다.

## 대체 솔루션

### Python API만 사용 중인 경우

1. Python API **v3.0.39**로 업그레이드합니다.

2. a. `{% include product %}_API_CACERTS`를 `/path/to/shotgun_api3/lib/httplib2/cacerts.txt`로 설정합니다.

   또는

   b. 스크립트를 업데이트하고 `Shotgun` 개체를 인스턴스화할 때 `ca_certs=/path/to/shotgun_api3/lib/httplib2/cacerts.txt`를 설정합니다.

### 툴킷을 사용 중인 경우

1. 툴킷 배포 방법에 따라 `tank core` 명령을 통해 최신 버전의 툴킷 API로 업그레이드하거나 파이프라인 구성의 `core/core_api.yml` 파일을 업데이트하는 방법으로 업그레이드합니다.

2. [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem)에서 최신 인증서 목록을 다운로드합니다.

3. `{% include product %}_API_CACERTS`를 이 파일을 저장한 위치로 설정합니다. 하지만 툴킷은 Python API에서처럼 연결을 만들 때 `ca_certs` 매개변수 지정을 허용하지 않습니다.

### Python API 또는 툴킷을 업데이트할 수 없는 경우

1. [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem)에서 최신 인증서 목록을 다운로드합니다.

2. `SSL_CERT_FILE` 환경 변수를 이 파일을 저장한 위치로 설정합니다.
