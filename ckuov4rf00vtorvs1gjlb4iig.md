## Azure Functions: Requeue Items

### Introduction
As an azure functions dev, I am sure you've come across issues where a `QueueTrigger` azure function fails and after several retry attempt (dequeue count) the items are sent into a poison queue, and after resolving the errors, we seek ways of moving all the items on the poison queue back onto the main queue for reprocessing. 

Most times to move the items from the poison queue back to the main queue, I use the Azure Storage Explorer, but it is a manual task, and I needed a way to automate it, and you can't do so with the queue binders (input/output), you will need to write a separate set of code and then automate it with either a `TimeTrigger` or `HttpTrigger`.

### Show Me The Code
#### Requirement
1. Azure Storage connection string (for accessing the queues)
2. Nuget Library: [Azure.Storage.Queues](https://www.nuget.org/packages/Azure.Storage.Queues/) (v12.5.0 or latest) 

```c#
public static async Task RequeueItems(QueueClient sourceQueue, QueueClient destinationQueue)
        {
            int messageCount = 32;

            var sourceQueueProps = await GetQueuePropertiesAsync(sourceQueue);
            int queuelength = sourceQueueProps.ApproximateMessagesCount;

            if (queuelength > 0)
            {
                var iterate = Math.Ceiling((double)queuelength / messageCount);
                do 
                {                    
                    QueueMessage[] _messages = await sourceQueue.ReceiveMessagesAsync(messageCount);
                    if (_messages.Any())
                    {
                        foreach (var item in _messages)
                        {
                            await destinationQueue.SendMessageAsync(item.Body);
                            await sourceQueue.DeleteMessageAsync(item.MessageId, item.PopReceipt);
                        }
                    }
                    iterate--;
                } while (iterate > 0);
            }
        }
```
The `RequeueItems` function accept two `QueueClient` parameters, one is the source queue (where the queue items are coming from) and the other is a destination queue (where the queue items should go), the function job is a simple **dequeue-enqueue-delete** operation that happens in a loop (the length of the source queue).

To automate it, a `TimerTrigger` will run `RequeueItems` function periodically, after getting the source and destination `QueueClient` objects to be used.

```c#
public static class RequeueUsingStatic
    {
        [FunctionName(nameof(RequeueUsingStatic))]
        public static async Task Run([TimerTrigger("0 0 10-20/2 * * Mon-Fri")] TimerInfo myTimer, ILogger log)
        {
            log.LogInformation($"C# Timer trigger function {nameof(RequeueUsingStatic)} executed at: {DateTime.Now:dd-MMM-yyyy}");

            try
            {
                var azure_storage_Uri = Environment.GetEnvironmentVariable("AzureWebJobsStorage");

                QueueClient sourceQueue = await RequeueExtensions.GetQueueClientAsync(azure_storage_Uri, RequeueSampleData.sourceQueueName);
                QueueClient poisonQueue = await RequeueExtensions.GetQueueClientAsync(azure_storage_Uri, RequeueSampleData.poisonQueueName);

                await RequeueExtensions.RequeueItems(poisonQueue, sourceQueue);
            }
            catch (Exception ex)
            {                
                log.LogCritical($"C# Timer trigger function {nameof(RequeueUsingStatic)} failed at: {DateTime.Now:dd-MMM-yyyy}, Error Message: {ex.Message}");
            }
        }
    }
```
So every two hours between 10 am to 8 pm, from Monday through Friday, the `TimeTrigger` would requeue all the failed items (on the poison queue) back to the main queue for reprocessing, hence depending on your scenario or use case, any trigger should work, such as `HttpTrigger`.

### Conclusion
Besides the use case of moving queue items from the poison queue back to the main queue, the `RequeueItems` function, when needed can be used for moving all items between any two queues in bulk.

Here is a link to the complete [code repo](https://github.com/olakusibe/Olakusibe.AzureFunctions.RequeueItems)


CREDIT: Cover Photo by <a href="https://unsplash.com/@deeezyfree?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Peter Olexa</a> on <a href="https://unsplash.com/s/photos/splash?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>