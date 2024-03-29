# [C#]測試非同步方法是否有丟出例外

筆記起來，我老是忘記...

<br/>使用套件: 
- NUnit
- FluentAssertions

<br/>Production Code:
假設寫個非同步函式, 要測試它是否如預期非丟出例外
```csharp
public class Target
{
    public async Task Method()
    {
        await Task.Run(() => throw new Exception("Hello"));
    }
}
```

<br/>Test Code:
```csharp
[TestFixture]
public class Test
{
    [Test]
    public void MethodTest()
    {
        var target = new Target();
        
        Func<Task> func = async () => { await target.Method(); };
        
        func.Should().ThrowAsync<Exception>().Where(e => e.Message == "Hello")
            .ConfigureAwait(false).GetAwaiter().GetResult();
    }
}
```
或是這樣寫
```csharp
[TestFixture]
public class Test
{
    [Test]
    public async Task MethodTest()
    {
        var target = new Target();
        
        Func<Task> func = async () => { await target.Method(); };
        
        await func.Should().ThrowAsync<Exception>().Where(e => e.Message == "Hello");
    }
}
```