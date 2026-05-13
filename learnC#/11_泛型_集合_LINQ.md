# 泛型、集合与 LINQ

## C++ 模板与 C# 泛型的差异

| 维度 | C++ 模板 | C# 泛型 |
|---|---|---|
| 实例化 | 编译期模板实例化 | 运行时支持的泛型机制 |
| 约束 | concepts、SFINAE、模板技巧 | `where T : ...` 约束 |
| 元编程能力 | 极强 | 有限，通常配合反射/Source Generator |
| 错误信息 | 复杂模板时可能很长 | 通常更直接 |
| 代码生成 | 可能为每种类型生成代码 | 值类型/引用类型处理由运行时和 JIT 管理 |

C# 泛型更像“类型安全的运行时泛型”，不是 C++ 模板元编程系统。

## 泛型方法

```csharp
static T FirstOrDefault<T>(IEnumerable<T> values, T fallback)
{
    foreach (var value in values)
    {
        return value;
    }

    return fallback;
}
```

## 泛型约束

```csharp
public sealed class Repository<T>
    where T : class
{
}
```

常见约束：

```csharp
where T : class
where T : struct
where T : notnull
where T : new()
where T : BaseType
where T : IComparable<T>
```

多个约束：

```csharp
public static T Max<T>(T a, T b)
    where T : IComparable<T>
{
    return a.CompareTo(b) >= 0 ? a : b;
}
```

## 集合常用类型

| C++ | C# |
|---|---|
| `std::vector<T>` | `List<T>` / `T[]` |
| `std::array<T, N>` | `T[]`，长度运行时确定；或自定义固定结构 |
| `std::deque<T>` | `Queue<T>` / `Deque` 需第三方或自实现 |
| `std::list<T>` | `LinkedList<T>`，实际较少用 |
| `std::map<K,V>` | `SortedDictionary<K,V>` |
| `std::unordered_map<K,V>` | `Dictionary<TKey,TValue>` |
| `std::set<T>` | `SortedSet<T>` |
| `std::unordered_set<T>` | `HashSet<T>` |

## `IEnumerable<T>` 与延迟执行

```csharp
IEnumerable<int> query = numbers
    .Where(x => x > 0)
    .Select(x => x * 2);
```

这段通常不会立即执行。只有枚举时才执行：

```csharp
foreach (var value in query)
{
    Console.WriteLine(value);
}
```

或强制物化：

```csharp
var list = query.ToList();
```

## LINQ

LINQ 是 C# 非常重要的日常工具：

```csharp
var names = users
    .Where(user => user.IsActive)
    .OrderBy(user => user.Name)
    .Select(user => user.Name)
    .ToList();
```

它适合表达集合变换、过滤、投影、分组。

## 查询语法

```csharp
var query =
    from user in users
    where user.IsActive
    orderby user.Name
    select user.Name;
```

多数团队更常用方法链语法。查询语法在 join、group 等场景有时更清晰。

## LINQ 与性能

LINQ 可能带来：

- 委托调用。
- 迭代器对象。
- 闭包分配。
- 多次枚举。
- 延迟执行导致时机不明显。

普通业务代码优先可读性。性能敏感路径要 profile，再决定是否改为手写循环。

## 多次枚举陷阱

```csharp
IEnumerable<User> activeUsers = users.Where(x => x.IsActive);

var count = activeUsers.Count();
var first = activeUsers.FirstOrDefault();
```

如果 `users` 是数据库查询或流式来源，可能触发两次查询或两次遍历。需要时先物化：

```csharp
var activeUsers = users.Where(x => x.IsActive).ToList();
```

## `IReadOnlyList<T>` 与 API 设计

公共 API 不一定要暴露 `List<T>`：

```csharp
public IReadOnlyList<User> Users => _users;
```

参数可以接收更抽象的类型：

```csharp
public void AddUsers(IEnumerable<User> users)
{
}
```

但如果方法需要索引和 Count，`IReadOnlyList<T>` 比 `IEnumerable<T>` 更准确。

## 字典访问

```csharp
if (map.TryGetValue(id, out var user))
{
    Console.WriteLine(user.Name);
}
```

这比先 `ContainsKey` 再索引更高效。

## 迁移建议

- 不要把 C# 泛型当 C++ 模板。C# 泛型不适合复杂编译期元编程。
- `IEnumerable<T>` 代表可枚举，不代表已经在内存中的集合。
- LINQ 很强，但要理解延迟执行。
- 公共 API 尽量暴露意图明确的抽象，例如 `IReadOnlyList<T>`、`IReadOnlyCollection<T>`。

