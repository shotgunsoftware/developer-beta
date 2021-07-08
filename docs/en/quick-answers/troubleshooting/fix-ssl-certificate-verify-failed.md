---
layout: default
title: "Fixing the SSL: CERTIFICATE_VERIFY_FAILED issues with the Python API"
pagename: fix-ssl-certificate-verify-failed
lang: en
---

# Fixing the SSL: CERTIFICATE_VERIFY_FAILED issues with the Python API on Windows

The Python API relies on a list of certificates that is bundled with the API and on your machine in order to connect to the various webservices {% include product %} uses. Unfortunately, new certificate authorities can be released and those might not be bundled with the Python API or OS.

While our Python API comes with a very recent copy of the certificates, as of February 21st 2019, there’s a bug that prevents the API from using those certificates for Amazon S3 uploads, even if you are using the latest version of the API. For background please see [this AWS blog post](https://aws.amazon.com/blogs/security/how-to-prepare-for-aws-move-to-its-own-certificate-authority/). To remediate the situation temporarily, you can try the following solutions. 

{% include info title="Note" content="These are temporary workarounds and we're looking into a long-term solution." %}

## Preferred Solution

Add the required CA certificate to the Windows Certificate Store. Windows 7 users may have to first [upgrade to PowerShell 3.0](https://docs.microsoft.com/en-us/office365/enterprise/powershell/manage-office-365-with-office-365-powershell) in order to use this solution, or alternatively use [certutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/certutil) to add the [required certificate](https://www.amazontrust.com/repository/SFSRootCAG2.cer).

1. Start an elevated PowerShell by right-clicking **Start** and then left-clicking **Windows PowerShell (Admin)**

2. Paste the following commands into the PowerShell window and then press Return to execute:

        $cert_url = "https://www.amazontrust.com/repository/SFSRootCAG2.cer"
        $cert_file = New-TemporaryFile
        Invoke-WebRequest -Uri $cert_url -UseBasicParsing -OutFile $cert_file.FullName
        Import-Certificate -FilePath $cert_file.FullName -CertStoreLocation Cert:\LocalMachine\Root

3. If details of the added certificate bearing thumbprint `925A8F8D2C6D04E0665F596AFF22D863E8256F3F` are displayed then the operation is complete and PowerShell can be closed.

## Alternative Solutions

### If you are using the Python API only

1. Upgrade to the Python API **v3.0.39**

2. a. Set `{% include product %}_API_CACERTS` to `/path/to/shotgun_api3/lib/httplib2/cacerts.txt`

   or
   
   b. Update your scripts and set the `ca_certs=/path/to/shotgun_api3/lib/httplib2/cacerts.txt` when instantiating the `Shotgun` object.

### If you are using Toolkit

1. Upgrade to the latest version of the Toolkit API via the `tank core` command or by updating
the `core/core_api.yml` file of your pipeline configuration, depending on how you deploy Toolkit.

2. Download an up-to-date list of certificates at [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem).

3. Set `{% include product %}_API_CACERTS` to the location where you saved this file. Toolkit doesn’t allow you to specify the `ca_certs` parameter when creating connections the way the Python API does.

### If you can’t update the Python API or Toolkit

1. Download an up-to-date list of certificates at [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem). 

2. Set the `SSL_CERT_FILE` environment variable to the location where you saved this file.
