# Telegram.Bot.Extensions.Polling [![NuGet](https://img.shields.io/nuget/v/Telegram.Bot.Extensions.Polling.svg)](https://www.nuget.org/packages/Telegram.Bot.Extensions.Polling/) [![Build Status](https://dev.azure.com/tgbots/Telegram.Bot.Extensions.Polling/_apis/build/status/TelegramBots.Telegram.Bot.Extensions.Polling?branchName=master)](https://dev.azure.com/tgbots/Telegram.Bot.Extensions.Polling/_build/latest?definitionId=3&branchName=master)

Provides `ITelegramBotClient` extensions for polling updates.

## Usage

```csharp
using System;
using System.Threading;
using Telegram.Bot;
using Telegram.Bot.Exceptions;
using Telegram.Bot.Types;

async Task HandleUpdateAsync(ITelegramBotClient botClient, Update update, CancellationToken cancellationToken)
{
    if (update.Message is Message message)
    {
        await botClient.SendTextMessageAsync(message.Chat, "Hello");
    }
}

async Task HandleErrorAsync(ITelegramBotClient botClient, Exception exception, CancellationToken cancellationToken)
{
    if (exception is ApiRequestException apiRequestException)
    {
        await botClient.SendTextMessageAsync(123, apiRequestException.ToString());
    }
}

ITelegramBotClient bot = new TelegramBotClient("<token>");
```

You have two ways of starting to receive updates
1. `StartReceiving` does not block the caller thread. Receiving is done on the ThreadPool.
```c#
using System.Threading;
using Telegram.Bot.Extensions.Polling;

var cts = new CancellationTokenSource();
var cancellationToken = cts.Token;

bot.StartReceiving(new DefaultUpdateHandler(HandleUpdateAsync, HandleErrorAsync), cancellationToken);
```

2. Awaiting `ReceiveAsync` will block until cancellation in triggered (both methods accept a CancellationToken)
```c#
using System.Threading;
using Telegram.Bot.Extensions.Polling;

var cts = new CancellationTokenSource();
var cancellationToken = cts.Token;

await bot.ReceiveAsync(new DefaultUpdateHandler(HandleUpdateAsync, HandleErrorAsync), cancellationToken);
```

Trigger cancellation by calling `cts.Cancel()` somewhere to stop receiving update in both methods.

## Update streams

With .Net Core 3.0+ comes support for an `IAsyncEnumerable<Update>` to stream Updates as they are received.

The package also exposes a more advanced `QueuedUpdateReceiver`, that enqueues Updates.

```csharp
using Telegram.Bot;
using Telegram.Bot.Types;
using Telegram.Bot.Extensions.Polling;

ITelegramBotClient bot = new TelegramBotClient("<token>");
QueuedUpdateReceiver updateReceiver = new QueuedUpdateReceiver(bot);

updateReceiver.StartReceiving();

await foreach (Update update in updateReceiver.YieldUpdatesAsync())
{
    if (update.Message is Message message)
    {
        await Bot.SendTextMessageAsync(
            message.Chat,
            $"Still have to process {updateReceiver.PendingUpdates} updates"
        );
    }
}
```