# AutoGen_Reconstruction
AutoGen Reconstruction from microsoft and use mandarin language.
# 🤖 AutoGen

## 📢 重要通知 (from origin MS README)

- 🎄 (2024/12/19) 哈囉！大部分 AutoGen 團隊成員將在假期期間與親朋好友共度時光，為自己充電！在 12/20-01/06 期間，專案的活動和回應可能會有所延遲。我們期待在新年與大家重新相聚！
- 💬 (2024/12/11) 我們為 AutoGen 社群建立了全新的 Discord 伺服器！歡迎加入：aka.ms/autogen-discord
- ⚠️ (2024/11/14) 為了回應許多關於官方 AutoGen 與其分支專案之間的混淆，我們發布了一份澄清聲明。
- 📦 (2024/10/13) 對 AutoGen 0.2 感興趣的老用戶？請查看活躍維護中的 AutoGen 0.2 分支和 autogen-agentchat~=0.2 PyPi 套件。
- 🚀 (2024/10/02) AutoGen 0.4 是一次從零開始的重寫！想了解更多歷史、目標和未來規劃，請查看我們的部落格文章。在正式發布 0.4 之前，我們很期待與社群合作，收集回饋並改進專案。這是一個重大改變，所以 AutoGen 0.2 仍在 0.2 分支中持續維護和開發。


## 🎯 AutoGen 是什麼？
AutoGen 是一個開源框架，專門打造 AI 代理系統。它簡化了事件驅動、分散式、可擴展且具有彈性的代理應用程式的建立過程。透過 AutoGen，你可以快速建立讓 AI 代理互相協作的系統，讓它們自主執行任務或在人類監督下工作。

## 🌈 主要特色

### 🔄 非同步訊息傳遞
- 代理透過非同步訊息進行溝通
- 支援事件驅動和請求/回應互動模式

### 📝 完整類型支援
- 所有介面都使用類型
- 建置時強制類型檢查，注重品質和一致性

### 🔋 可擴展且分散式
- 設計複雜的分散式代理網路
- 可跨組織界限運作

### 🧩 模組化且可擴展
- 使用可插拔元件自訂系統：
  - 客製代理
  - 工具
  - 記憶體
  - 模型

### 🌐 跨語言支援
- 不同程式語言的代理可互相操作
- 目前支援 Python 和 .NET
- 更多語言即將推出！

### 🔍 可觀察性和除錯
- 內建追蹤和除錯功能
- 支援 OpenTelemetry 工業標準可觀察性

## 🎨 API 分層

### 1️⃣ Core（核心）
- 基礎 API
- 遵循 Actor 模型
- 支援非同步訊息傳遞
- 適合建立可擴展的事件驅動系統

### 2️⃣ AgentChat（代理對話）
- 任務導向的高階 API
- 類似 AutoGen 0.2
- 可定義對話代理
- 組成團隊解決任務

### 3️⃣ Extensions（擴充功能）
- 使用第三方系統實現核心介面
- 包含 OpenAI 模型客戶端
- 支援社群貢獻的擴充功能

## 🎓 新手上路

### 🐍 Python 版本（AgentChat）
```python
First install the packages:

pip install "autogen-agentchat==0.4.0.dev11" "autogen-ext[openai]==0.4.0.dev11"
The following code uses OpenAI's GPT-4o model and you need to provide your API key to run. To use Azure OpenAI models, follow the instruction here.

import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_ext.models.openai import OpenAIChatCompletionClient

# Define a tool
async def get_weather(city: str) -> str:
    return f"The weather in {city} is 73 degrees and Sunny."

async def main() -> None:
    # Define an agent
    weather_agent = AssistantAgent(
        name="weather_agent",
        model_client=OpenAIChatCompletionClient(
            model="gpt-4o-2024-08-06",
            # api_key="YOUR_API_KEY",
        ),
        tools=[get_weather],
    )

    # Define termination condition
    termination = TextMentionTermination("TERMINATE")

    # Define a team
    agent_team = RoundRobinGroupChat([weather_agent], termination_condition=termination)

    # Run the team and stream messages to the console
    stream = agent_team.run_stream(task="What is the weather in New York?")
    await Console(stream)

asyncio.run(main())
```

### 🎯 C# 版本
```csharp
The .NET SDK does not yet support all of the interfaces that the python SDK offers but we are working on bringing them to parity. To use the .NET SDK, you need to add a package reference to the src in your project. We will release nuget packages soon and will update these instructions when that happens.

git clone https://github.com/microsoft/autogen.git
cd autogen
# Switch to the branch that has this code
git switch staging-dev
# Build the project
cd dotnet && dotnet build AutoGen.sln
# In your source code, add AutoGen to your project
dotnet add <your.csproj> reference <path to your checkout of autogen>/dotnet/src/Microsoft.AutoGen/Core/Microsoft.AutoGen.Core.csproj
Then, define and run your first agent:

using Microsoft.AutoGen.Contracts;
using Microsoft.AutoGen.Core;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

// send a message to the agent
var app = await App.PublishMessageAsync("HelloAgents", new NewMessageReceived
{
    Message = "World"
}, local: true);

await App.RuntimeApp!.WaitForShutdownAsync();
await app.WaitForShutdownAsync();

[TopicSubscription("agents")]
public class HelloAgent(
    IAgentContext worker,
    [FromKeyedServices("EventTypes")] EventTypes typeRegistry) : ConsoleAgent(
        worker,
        typeRegistry),
        ISayHello,
        IHandle<NewMessageReceived>,
        IHandle<ConversationClosed>
{
    public async Task Handle(NewMessageReceived item)
    {
        var response = await SayHello(item.Message).ConfigureAwait(false);
        var evt = new Output
        {
            Message = response
        }.ToCloudEvent(this.AgentId.Key);
        await PublishEventAsync(evt).ConfigureAwait(false);
        var goodbye = new ConversationClosed
        {
            UserId = this.AgentId.Key,
            UserMessage = "Goodbye"
        }.ToCloudEvent(this.AgentId.Key);
        await PublishEventAsync(goodbye).ConfigureAwait(false);
    }
    public async Task Handle(ConversationClosed item)
    {
        var goodbye = $"*********************  {item.UserId} said {item.UserMessage}  ************************";
        var evt = new Output
        {
            Message = goodbye
        }.ToCloudEvent(this.AgentId.Key);
        await PublishEventAsync(evt).ConfigureAwait(false);
        await Task.Delay(60000);
        await App.ShutdownAsync();
    }
    public async Task<string> SayHello(string ask)
    {
        var response = $"\n\n\n\n***************Hello {ask}**********************\n\n\n\n";
        return response;
    }
}
public interface ISayHello
{
    public Task<string> SayHello(string ask);
}
dotnet run
```

## 🗺️ 未來規劃

### 🏗️ AutoGen 0.2
- 目前的穩定版本
- 持續接受錯誤修復和小幅改進

### 🚀 AutoGen 0.4
- 全新架構的首次發布（預覽版）
- 專注於：
  - 介面穩定性
  - 文件完整性
  - 教學和範例
  - 內建代理集合
- 未來計畫：
  - 更多程式語言支援
  - 更多內建代理和工作流程
  - 分散式代理部署
  - AutoGen Studio 重新實現
  - 與其他框架整合
  - 進階 RAG 技術

## ❓ 常見問題

