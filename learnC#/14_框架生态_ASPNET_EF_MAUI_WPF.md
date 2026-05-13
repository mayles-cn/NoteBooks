# 框架生态：ASP.NET Core、EF Core、MAUI、WPF

## C# 框架生态的特点

C++ 生态更分散：Qt、Boost、Unreal、CMake、vcpkg、Conan、平台 SDK、各种专用库并存。C# / .NET 生态更集中，常见工程通常围绕 .NET SDK、NuGet 和微软维护的基础框架。

## ASP.NET Core

ASP.NET Core 是现代 .NET Web 框架，适合：

- REST API。
- Web 应用。
- 微服务。
- 实时通信 SignalR。
- 后台服务。
- gRPC。

最小 API 示例：

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/hello", () => "Hello");

app.Run();
```

控制器示例：

```csharp
[ApiController]
[Route("api/users")]
public sealed class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public ActionResult<UserDto> Get(string id)
    {
        return new UserDto(id, "Ada");
    }
}
```

## 依赖注入

ASP.NET Core 内置依赖注入：

```csharp
builder.Services.AddScoped<IUserRepository, SqlUserRepository>();
builder.Services.AddSingleton<IClock, SystemClock>();
```

常见生命周期：

- Singleton：应用生命周期内一个实例。
- Scoped：一次请求或作用域内一个实例。
- Transient：每次请求服务都创建新实例。

生命周期错误是 C# Web 工程常见 bug，例如 singleton 持有 scoped 服务。

## EF Core

Entity Framework Core 是 .NET 主流 ORM：

```csharp
public sealed class AppDbContext : DbContext
{
    public DbSet<User> Users => Set<User>();

    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
}
```

查询：

```csharp
var activeUsers = await db.Users
    .Where(user => user.IsActive)
    .OrderBy(user => user.Name)
    .ToListAsync();
```

注意：EF Core 的 LINQ 会翻译成 SQL。不是所有 C# 表达式都能翻译。复杂查询要看生成 SQL。

## Dapper

Dapper 是轻量级数据库访问库，更接近手写 SQL：

```csharp
var users = await connection.QueryAsync<User>(
    "select Id, Name from Users where IsActive = @IsActive",
    new { IsActive = true });
```

如果你喜欢明确控制 SQL，Dapper 可能比 ORM 更符合 C++ 程序员的控制感。

## WPF

WPF 是 Windows 桌面 UI 框架，适合：

- 企业桌面软件。
- 内部工具。
- 数据录入系统。
- 需要强 Windows 集成的客户端。

它常用 XAML：

```xml
<Window x:Class="Demo.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        Title="Demo">
    <Grid>
        <TextBlock Text="Hello" />
    </Grid>
</Window>
```

常见架构模式是 MVVM。

## WinForms

WinForms 更老，但仍用于大量 Windows 内部工具。它简单直接，适合快速做传统桌面表单，不适合追求现代 UI 表现。

## .NET MAUI

.NET MAUI 是跨平台客户端框架，目标是用一套 .NET 代码构建 Android、iOS、macOS、Windows 应用。适合已有 .NET 团队做跨平台业务客户端，但实际项目中要关注平台差异、控件成熟度和部署链路。

## Unity

Unity 使用 C# 作为主要脚本语言。对于 C++ 背景的人，可以把 C# 用于游戏逻辑、编辑器工具、资源管线，而引擎底层仍由 C++ 支撑。

Unity C# 与普通 .NET 后端 C# 有差异：

- 生命周期由 Unity 引擎驱动。
- `MonoBehaviour` 模型特殊。
- GC 分配对帧率敏感。
- 部分 .NET API 可用性受 Unity 版本影响。

## Worker Service

C# 很适合写后台服务：

```csharp
public sealed class Worker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await Task.Delay(TimeSpan.FromSeconds(1), stoppingToken);
        }
    }
}
```

适合消息消费、定时任务、队列处理、数据同步。

## 框架选择建议

| 场景 | 优先考虑 |
|---|---|
| Web API / 后端服务 | ASP.NET Core |
| 数据库 ORM | EF Core |
| 明确 SQL 控制 | Dapper |
| Windows 桌面工具 | WPF 或 WinForms |
| 跨平台移动/桌面客户端 | .NET MAUI，需评估成熟度 |
| 游戏脚本 | Unity C# |
| 后台常驻任务 | Worker Service |

## 迁移建议

- 先学 ASP.NET Core 和基础 .NET CLI，收益最大。
- 数据访问先理解 EF Core 的 change tracking、LINQ 翻译、迁移机制。
- 桌面方向如果在 Windows，WPF 仍然值得学。
- 不要只学语法，至少做一个“API + 数据库 + 测试 + 发布”的完整小项目。

