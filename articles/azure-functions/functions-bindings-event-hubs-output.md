---
title: Azure Event Hubs output binding for Azure Functions
description: Learn to write messages to Azure Event Hubs streams using Azure Functions.
ms.assetid: daf81798-7acc-419a-bc32-b5a41c6db56b
ms.topic: reference
ms.custom: ignite-2022
ms.date: 03/04/2022
zone_pivot_groups: programming-languages-set-functions-lang-workers
---

# Azure Event Hubs output binding for Azure Functions

This article explains how to work with [Azure Event Hubs](../event-hubs/event-hubs-about.md) bindings for Azure Functions. Azure Functions supports trigger and output bindings for Event Hubs.

For information on setup and configuration details, see the [overview](functions-bindings-event-hubs.md).

Use the Event Hubs output binding to write events to an event stream. You must have send permission to an event hub to write events to it.

Make sure the required package references are in place before you try to implement an output binding.

## Example

::: zone pivot="programming-language-csharp"

# [In-process](#tab/in-process)

The following example shows a [C# function](functions-dotnet-class-library.md) that writes a message to an event hub, using the method return value as the output:

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

The following example shows how to use the `IAsyncCollector` interface to send a batch of messages. This scenario is common when you are processing messages coming from one Event Hub and sending the result to another Event Hub.

```csharp
[FunctionName("EH2EH")]
public static async Task Run(
    [EventHubTrigger("source", Connection = "EventHubConnectionAppSetting")] EventData[] events,
    [EventHub("dest", Connection = "EventHubConnectionAppSetting")]IAsyncCollector<string> outputEvents,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        // do some processing:
        var myProcessedEvent = DoSomething(eventData);

        // then send the message
        await outputEvents.AddAsync(JsonConvert.SerializeObject(myProcessedEvent));
    }
}
```
# [Isolated process](#tab/isolated-process)

The following example shows a [C# function](dotnet-isolated-process-guide.md) that writes a message string to an event hub, using the method return value as the output:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/EventHubs/EventHubsFunction.cs" range="12-23":::

# [C# Script](#tab/csharp-script)

The following example shows an event hub trigger binding in a *function.json* file and a [C# script function](functions-reference-csharp.md) that uses the binding. The function writes a message to an event hub.

The following examples show Event Hubs binding data in the *function.json* file for Functions runtime version 2.x and later versions. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Here's C# script code that creates one message:

```cs
using System;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out string outputEventHubMessage, ILogger log)
{
    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
    log.LogInformation(msg);   
    outputEventHubMessage = msg;
}
```

Here's C# script code that creates multiple messages:

```cs
public static void Run(TimerInfo myTimer, ICollector<string> outputEventHubMessage, ILogger log)
{
    string message = $"Message created at: {DateTime.Now}";
    log.LogInformation(message);
    outputEventHubMessage.Add("1 " + message);
    outputEventHubMessage.Add("2 " + message);
}
```

---

::: zone-end 
::: zone pivot="programming-language-javascript"  

The following example shows an event hub trigger binding in a *function.json* file and a function that uses the binding. The function writes an output message to an event hub.

The following example shows an Event Hubs binding data in the *function.json* file, which is different for version 1.x of the Functions runtime compared to later versions. 

# [Functions 2.x+](#tab/functionsv2)

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

# [Functions 1.x](#tab/functionsv1)

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

---

Here's JavaScript code that sends a single message:

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    context.log('Message created at: ', timeStamp);   
    context.bindings.outputEventHubMessage = "Message created at: " + timeStamp;
    context.done();
};
```

Here's JavaScript code that sends multiple messages:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();
    var message = 'Message created at: ' + timeStamp;

    context.bindings.outputEventHubMessage = [];

    context.bindings.outputEventHubMessage.push("1 " + message);
    context.bindings.outputEventHubMessage.push("2 " + message);
    context.done();
};
```

::: zone-end  
::: zone pivot="programming-language-powershell" 
 
Complete PowerShell examples are pending.
::: zone-end 
::: zone pivot="programming-language-python"  
The following example shows an event hub trigger binding in a *function.json* file and a [Python function](functions-reference-python.md) that uses the binding. The function writes a message to an event hub.

The following examples show Event Hubs binding data in the *function.json* file.

```json
{
    "type": "eventHub",
    "name": "$return",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Here's Python code that sends a single message:

```python
import datetime
import logging
import azure.functions as func


def main(timer: func.TimerRequest) -> str:
    timestamp = datetime.datetime.utcnow()
    logging.info('Message created at: %s', timestamp)
    return 'Message created at: {}'.format(timestamp)
```

::: zone-end
::: zone pivot="programming-language-java"
The following example shows a Java function that writes a message containing the current time to an Event Hub.

```java
@FunctionName("sendTime")
@EventHubOutput(name = "event", eventHubName = "samples-workitems", connection = "AzureEventHubConnection")
public String sendTime(
   @TimerTrigger(name = "sendTimeTrigger", schedule = "0 */5 * * * *") String timerInfo)  {
     return LocalDateTime.now().toString();
 }
```

In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `@EventHubOutput` annotation on parameters whose value would be published to Event Hub.  The parameter should be of type `OutputBinding<T>` , where `T` is a POJO or any native Java type.

::: zone-end
::: zone pivot="programming-language-csharp"
## Attributes

Both [in-process](functions-dotnet-class-library.md) and [isolated process](dotnet-isolated-process-guide.md) C# libraries use attribute to configure the binding. C# script instead uses a function.json configuration file.

# [In-process](#tab/in-process)

Use the [EventHubAttribute] to define an output binding to an event hub, which supports the following properties.

| Parameters | Description|
|---------|----------------------|
|**EventHubName** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. |
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

# [Isolated process](#tab/isolated-process)

Use the [EventHubOutputAttribute] to define an output binding to an event hub, which supports the following properties.

| Parameters | Description|
|---------|----------------------|
|**EventHubName** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. |
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

# [C# Script](#tab/csharp-script)

The following table explains the binding configuration properties that you set in the *function.json* file.

|function.json property | Description|
|---------|------------------------|
|**type** |  Must be set to `eventHub`. |
|**direction** | Must be set to `out`. This parameter is set automatically when you create the binding in the Azure portal. |
|**name** |  The variable name used in function code that represents the event. |
|**eventHubName** | Functions 2.x and higher. The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. In Functions 1.x, this property is named `path`.|
|**connection**  | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

---

::: zone-end  
::: zone pivot="programming-language-java"  
## Annotations

In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) annotation on parameters whose value would be published to Event Hub. The following settings are supported on the annotation:

+ [name](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.name)
+ [dataType](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.datatype)
+ [eventHubName](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.eventhubname)
+ [connection](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.connection)

::: zone-end 
::: zone pivot="programming-language-javascript,programming-language-python,programming-language-powershell"  

## Configuration

The following table explains the binding configuration properties that you set in the *function.json* file, which differs by runtime version.

# [Functions 2.x+](#tab/functionsv2)

|function.json property | Description|
|---------|------------------------|
|**type** |  Must be set to `eventHub`. |
|**direction** | Must be set to `out`. This parameter is set automatically when you create the binding in the Azure portal. |
|**name** |  The variable name used in function code that represents the event. |
|**eventHubName** | Functions 2.x and higher. The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. |
|**connection**  | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

# [Functions 1.x](#tab/functionsv1)

|function.json property | Description|
|---------|------------------------|
|**type** |  Must be set to `eventHub`. |
|**direction** | Must be set to `out`. This parameter is set automatically when you create the binding in the Azure portal. |
|**name** |  The variable name used in function code that represents the event. |
|**path** | Functions 1.x only. The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. |
|**connection**  | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

---

::: zone-end

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## Usage

::: zone pivot="programming-language-csharp"  
The parameter type supported by the Event Hubs output binding depends on the Functions runtime version, the extension package version, and the C# modality used. 

# [Extension v5.x+](#tab/extensionv5/in-process)

In-process C# class library functions supports the following types:

+ [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

This version of [EventData](/dotnet/api/azure.messaging.eventhubs.eventdata) drops support for the legacy `Body` type in favor of [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody).

Send messages by using a method parameter such as `out string paramName`. To write multiple messages, you can use `ICollector<string>` or `IAsyncCollector<string>` in place of `out string`.

# [Extension v3.x+](#tab/extensionv3/in-process)

In-process C# class library functions supports the following types:

+ [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

Send messages by using a method parameter such as `out string paramName`. To write multiple messages, you can use `ICollector<string>` or `IAsyncCollector<string>` in place of `out string`.

# [Extension v5.x+](#tab/extensionv5/isolated-process)

Requires you to define a custom type, or use a string. 

# [Extension v3.x+](#tab/extensionv3/isolated-process)

Requires you to define a custom type, or use a string.

# [Extension v5.x+](#tab/extensionv5/csharp-script)

C# script functions support the following types:

+ [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

This version of [EventData](/dotnet/api/azure.messaging.eventhubs.eventdata) drops support for the legacy `Body` type in favor of [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody).

Send messages by using a method parameter such as `out string paramName`, where `paramName` is the value specified in the `name` property of *function.json*. To write multiple messages, you can use `ICollector<string>` or `IAsyncCollector<string>` in place of `out string`.

# [Extension v3.x+](#tab/extensionv3/csharp-script)

C# script functions support the following types:

+ [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

Send messages by using a method parameter such as `out string paramName`, where `paramName` is the value specified in the `name` property of *function.json*. To write multiple messages, you can use `ICollector<string>` or
`IAsyncCollector<string>` in place of `out string`.

---

::: zone-end  
::: zone pivot="programming-language-java" 

There are two options for outputting an Event Hub message from a function by using the [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) annotation:

- **Return value**: By applying the annotation to the function itself, the return value of the function is persisted as an Event Hub message.

- **Imperative**: To explicitly set the message value, apply the annotation to a specific parameter of the type [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.OutputBinding), where `T` is a POJO or any native Java type. With this configuration, passing a value to the `setValue` method persists the value as an Event Hub message.
::: zone-end
::: zone pivot="programming-language-powershell" 
 
Complete PowerShell examples are pending.
::: zone-end 
::: zone pivot="programming-language-javascript"

Access the output event by using `context.bindings.<name>` where `<name>` is the value specified in the `name` property of *function.json*.

::: zone-end  
::: zone pivot="programming-language-python"  

There are two options for outputting an Event Hub message from a function:

- **Return value**: Set the `name` property in *function.json* to `$return`. With this configuration, the function's return value is persisted as an Event Hub message.

- **Imperative**: Pass a value to the [set](/python/api/azure-functions/azure.functions.out#set-val--t-----none) method of the parameter declared as an [Out](/python/api/azure-functions/azure.functions.out) type. The value passed to `set` is persisted as an Event Hub message.

::: zone-end

[!INCLUDE [functions-event-hubs-connections](../../includes/functions-event-hubs-connections.md)]

## Exceptions and return codes

| Binding | Reference |
|---|---|
| Event Hub | [Operations Guide](/rest/api/eventhub/publisher-policy-operations) |

## Next steps

- [Respond to events sent to an event hub event stream (Trigger)](./functions-bindings-event-hubs-trigger.md)
 

[EventHubAttribute]: /dotnet/api/microsoft.azure.webjobs.eventhubattribute
