---
title: "클라우드: Storage Account에 접근할 때 발생하는 OutOfRangeInput 오류"
date: 2021-02-21 00:00:00
categories: cloud, eventhubs
---

Azure Event Hubs에 이벤트를 전송하는 클라이언트 응용 프로그램을 작성할 때 보통 참고하게 되는 것이 Client SDK에 포함된 샘플 코드이다. 많은 사용자들이 이 샘플 코드에 살을 붙여 Producer 프로그램을 작성하곤 하는데 여기에는 주의할 점이 있습니다. 예를 들어, 아래의 C 샘플 코드는 상수로 정의한 파티션 키를 이용하여 이벤트를 전송하는데, 이렇게 되면 특정 파티션으로만 이벤트를 전송하게 된다는 점입니다.

```cpp
#include "eventhubclient.h"
#include "eventdata.h"
#include "send.h"
#include "azure_macro_utils/macro_utils.h"
#include "azure_c_shared_utility/threadapi.h"
#include "azure_c_shared_utility/crt_abstractions.h"
#include "azure_c_shared_utility/platform.h"
#include "azure_c_shared_utility/xlogging.h"

static const char* connectionString = "Endpoint=sb://[namespace].servicebus.windows.net/;SharedAccessKeyName=[key name];SharedAccessKey=[key value]";
static const char* eventHubPath = "[event hub name]";

static bool g_bSendProperties = false;
static bool g_bSendPartitionKey = false;

static const char PARTITION_KEY_INFO[]  = "PartitionKeyInfo";
static const char TEST_STRING_VALUE_1[] = "Property_String_Value_1";
static const char TEST_STRING_VALUE_2[] = "Property_String_Value_2";

#define SLEEP_TIME		1000
#define BUFFER_SIZE     128
static unsigned int g_id = 1000;

MU_DEFINE_ENUM_STRINGS(EVENTHUBCLIENT_STATE, EVENTHUBCLIENT_STATE_VALUES);
MU_DEFINE_ENUM_STRINGS(EVENTHUBCLIENT_ERROR_RESULT, EVENTHUBCLIENT_ERROR_RESULT_VALUES);

int Send_Sample(void)
{
    // ...
    EVENTHUBCLIENT_HANDLE eventHubClientHandle = EventHubClient_CreateFromConnectionString(connectionString, eventHubPath);
    if (eventHubClientHandle == NULL)
    {
        (void)printf("ERROR: EventHubClient_CreateFromConnectionString returned NULL!\r\n");
        result = 1;
    }
    else
    {
        // ...
        EVENTDATA_HANDLE eventDataHandle = EventData_CreateWithNewMemory((const unsigned char*)msgContent, msgLength);
        if (eventDataHandle == NULL)
        {
            (void)printf("ERROR: eventDataHandle is NULL!\r\n");
            result = 1;
        }
        else
        {
            if (EventData_SetPartitionKey(eventDataHandle, PARTITION_KEY_INFO) != EVENTDATA_OK)
            {
                (void)printf("ERROR: EventData_SetPartitionKey failed!\r\n");
                result = 1;
            }
            else
            {
                // ...
                if (EventHubClient_Send(eventHubClientHandle, eventDataHandle) != EVENTHUBCLIENT_OK)
                {
                    (void)printf("ERROR: EventHubClient_Send failed!\r\n");
                    result = 1;
                }
                else
                {
                    (void)printf("EventHubClient_Send.......Successful\r\n");
                    result = 0;
                }
            }
            EventData_Destroy(eventDataHandle);
        }
        EventHubClient_Destroy(eventHubClientHandle);
    }

    // ...
    return result; 
}
```

EventData_SetPartitionKey()는 파티션 키 문자열을 인자로 받는데, 파티션 키로 해시 값을 생성하고 파티션 개수로 나누어 이벤트를 전송할 파티션을 배정합니다. 만약, 이벤트를 전송할 때마다 동일한 파티션 키를 사용한다면 모든 이벤트는 하나의 파티션을 향하게 될 것입니다. 따라서, 고르게 파티션을 사용하고 싶다면 EventData_SetPartitionKey()를 명시적으로 호출하지 않거나 매번 변하는 값으로 파티션 키를 지정해야 합니다.