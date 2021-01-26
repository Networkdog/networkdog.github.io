---
title: "클라우드: App Service에서 더 큰 용량의 디스크가 필요하다면?"
date: 2021-01-23 00:00:00
categories: techm cloud
---

Azure의 대표적인 PaaS 앱 호스팅 서비스는 App Service입니다. 


더 큰 용량의 디스크 공간을 제공하는 인스턴스로 Scale up 합니다.
Storage Account의 file share나 blob container를 마운트하여 사용합니다.
Storage Account의 file share를 home 디렉터리로 사용할 수 있습니다.
같은 리소스 그룹에 같은 지역의 App Service Plan이 있다면 디스크 가용량이 공유됩니다.




하나의 리소스 그룹에 위치하면서 같은 지역에 생성된 App Service Plan은 사용 가능한 디스크 용량이 서로 공유됩니다. 예를 들어, 아래와 같은 리소스를 보유한다면,

    rg-contoso (Resource Group)
        asp-koreacentral-1 (App Service Plan, Korea Central, B1, 10GB)
        asp-koreacentral-2 (App Service Plan, Korea Central, P1V2, 250GB)
        wa-webservice-1 (App Service, asp-koreacentral-1)
        wa-webservice-2 (App Service, asp-koreacentral-2)

B1을 사용하는 App Service Plan에서 호스팅되고 있는 wa-webservice-1은 10GB의 디스크 공간이 주어지지만 B1을 사용하는 App Service Plan에서 호스팅되고 있지만 B1의 한계인 10GB이 가용 디스크 크기로 

이 사용할 수 있는 가용 디스크 공간은 260GB가 됩니다. 