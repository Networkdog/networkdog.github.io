---
title: "클라우드: Storage Account에 접근할 때 발생하는 OutOfRangeInput 오류"
date: 2021-01-22 00:00:00
categories: cloud, storage
---

특정 Storage Account의 Blob을 다운로드하는 Powershell 스크립트가 출력하는 문제가 발견된 적이 있었습니다. Get-AzStorageBlob을 호출하여 Blob을 다운로드하는 간단한 코드였는데 말이죠. 

```powershell
$ctx = New-AzStorageContext -StorageAccountName "CONTOSO" -StorageAccountKey $StorageAccountKey
Get-AzStorageBlob -Container "test" -Blob "blob.dat" -Context $ctx
```

"Out of the request inputs is out of range" 오류 메시지가 보였지만, 파라메터의 값이 비어있지도 않았고 범위를 벗어날만한 숫자 파라메터는 없었습니다. 상태 코드가 400 Bad Request인 것을 보니 Get-AzStorageBlob이 보내는 REST API 요청은 Storage Account를 다녀온 것이 분명했습니다. 

```
Get-AzStorageBlob : One of the request inputs is out of range. HTTP Status Code: 400 - HTTP Error Message: One of the request inputs is out of range.
ErrorCode: OutOfRangeInput
ErrorMessage: One of the request inputs is out of range.
At D:\Workspace\outofrange\download_blob.ps1:41 char:17
+ ...     $Blob = Get-AzStorageBlob -Container "test" -Blob "blob.dat" ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Get-AzStorageBlob], StorageException
    + FullyQualifiedErrorId : StorageException,Microsoft.WindowsAzure.Commands.Storage.Blob.Cmdlet.GetAzureStorageBlob     Command     
```

Fiddler를 이용하여 Powershell 프로세스에서 보내는 HTTP 요청을 캡처해 보았습니다. URL에도 문제는 없어 보입니다. 그런데 Host 헤더는 소문자(contoso)인데 API 인증을 위한 Authorization 키에 사용된 이름은 대문자(CONTOSO)인 것이 보이네요. 

```
https://contoso.blob.core.windows.net/test/blob?comp=blocklist&blocklisttype=Committed
...
x-ms-version: 2019-07-07
Host: contoso.blob.core.windows.net

Authorization: SharedKey 
CONTOSO:1jtKy72sATULYIMex53m5UNCWpXavvk17nLm+SihgZk=
```

사실 Azure의 Storage Account는 소문자 이름으로만 생성할 수 있습니다. Storage Account의 API를 호출할 때 Storage Account 파라메터로 대문자가 들어간 이름을 사용하면 인증 헤더에 그 이름이 그대로 들어가서 Preauthenticated 단계에 OutOfRangeInput 오류가 발생하게 되는 것입니다.

![OnlyLowercaseAllowed](/images/2021-01-22-cloud-outofrangeinput-lowercaseonly.png)

그래서 New-AzStorageContext를 호출할 때 Storage Account 이름을 소문자로 변경해서 실행하니 이제는 정상적으로 동작합니다.

```powershell
$ctx = New-AzStorageContext -StorageAccountName "contoso" -StorageAccountKey $StorageAccountKey
Get-AzStorageBlob -Container "test" -Blob "blob.dat" -Context $ctx
```

HTTP 요청 헤더의 Authorization 값에도 그대로 소문자가 들어가니 OutOfRangeInput 오류는 더 이상 발생하지 않습니다.

```
https://contoso.blob.core.windows.net/test/blob?comp=blocklist&blocklisttype=Committed
...
x-ms-version: 2019-07-07
Host: contoso.blob.core.windows.net

Authorization: SharedKey 
contoso:1jtKy72sATULYIMex53m5UNCWpXavvk17nLm+SihgZk=
```