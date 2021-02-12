---
title: "클라우드: Storage Account의 Firewall에서 Whitelist를 사용하면 Server-to-server 복사 불가"
date: 2021-02-03 00:00:00
categories: tech cloud
---

AzCopy는 Storage Account 사이의 복사 작업에 server-to-server API를 사용합니다. 복사할 대상을 다운로드 한 후에 다시 업로드하여 복사하지 않고 Storage Account가 그 작업을 대신한다는 의미일 것입니다. AzCopy를 수행하는 위치의 네트워크 속도와 대역폭이 좋지 않을수록 이러한 서버간 복사는 빛을 발하게 됩니다. 그러나 장점이 대부분인 이 서버간 복사가 실제로 Storage Account의 Firewall 기능을 사용하는 경우에 무력화되는 사례를 간혹 접하게 됩니다.

```
Microsoft Azure Storage Explorer는 파일 복사 작업에 AzCopy를 사용합니다.
```

당연한 말이겠지만 서버간 복사가 이루어질 때의 클라이언트 IP는 AzCopy를 실행한 머신의 IP가 아닙니다. 따라서, Storage Account의 Firewall 규칙으로 특정 IP 주소들로만 화이트리스트를 구성했다면 AzCopy의 서버간 복사는 동작할 수 없습니다. Firewall로 인해 서버간 복사에 실패한 AzCopy는 수차례 재시도를 반복하다가 다운로드 후 업로드 방식으로 복사를 시작하게 됩니다. 재시도와 연결 실패로 인해 발생하는 지연은 감수할 수 밖에 없습니다. 그리고 AzCopy가 실행되는 위치가 Azure 외부에 있다면 다운로드에 대한 트래픽 비용이 발생하는 것도 피할 수 없습니다.


