# 讓 api 接受和回傳 snake case 的 json

使用平台: .Net Core 6.0

---

這樣的 Model
```csharp
public class Person
{
    public string LastName { get; set; }

    public string FirstName { get; set; }

    public int Age { get; set; }

    public double Money { get; set; }
}
```

<br/>預設是使用 camel case
```json
{
  "lastName": "string",
  "firstName": "string",
  "age": 0,
  "money": 0
}
```

<br/>如果我們希望它是 snake case，需要做點加工。
```json
{
  "last_name": "string",
  "first_name": "string",
  "age": 0,
  "money": 0
}
```

<br/>做法可以在每個 property 掛 attribute 指定想要的名字，這很花工夫，可能打錯名字、少掛 attribute。

那可以用以下的方法，做一次工就套用到整個專案

## 讓 api 在 requst body 和 resonse body 都接受 snake case

使用套件 Newtonsoft.Json

<br/>寫擴充方法
```csharp
public static class JsonSerializationExtensions
{
    private static readonly SnakeCaseNamingStrategy _snakeCaseNamingStrategy
        = new SnakeCaseNamingStrategy();

    private static readonly JsonSerializerSettings _snakeCaseSettings = new JsonSerializerSettings
    {
        ContractResolver = new DefaultContractResolver
        {
            NamingStrategy = _snakeCaseNamingStrategy
        }
    };

    public static string ToSnakeCase<T>(this T instance)
    {
        if (instance == null)
        {
            throw new ArgumentNullException(paramName: nameof(instance));
        }

        return JsonConvert.SerializeObject(instance, _snakeCaseSettings);
    }

    public static string ToSnakeCase(this string @string)
    {
        if (@string == null)
        {
            throw new ArgumentNullException(paramName: nameof(@string));
        }

        return Regex.Replace(@string, @"(\w)([A-Z])", "$1_$2").ToLower();
    }
}
```

<br/>繼承 JsonNamePolicy
```csharp
public class SnakeCaseNamingPolicy : JsonNamingPolicy
{
    public override string ConvertName(string name) => name.ToSnakeCase();
}
```

<br/>在 program.cs 加上
```csharp
builder.Services.AddJsonOptions(options => options.JsonSerializerOptions.PropertyNamingPolicy = new SnakeCaseNamingPolicy());
```

<br/>那麼 api 就接受和回傳 snake case 了
```json
{
  "last_name": "string",
  "first_name": "string",
  "age": 0,
  "money": 0
}
```

---

## 讓 api 在 Query parameters 接受 snake case

<br/>SnakeCaseQueryValueProvider
```csharp
public class SnakeCaseQueryValueProvider : QueryStringValueProvider, IValueProvider
{
    public SnakeCaseQueryValueProvider(BindingSource bindingSource, IQueryCollection values, CultureInfo culture) : base(bindingSource, values,
        culture)
    {
    }

    public override bool ContainsPrefix(string prefix)
    {
        return base.ContainsPrefix(prefix.ToSnakeCase());
    }

    public override ValueProviderResult GetValue(string key)
    {
        return base.GetValue(key.ToSnakeCase());
    }
}
```

<br/>SnakeCaseQueryValueProviderFactory
```csharp
public class SnakeCaseQueryValueProviderFactory : IValueProviderFactory
{
    public Task CreateValueProviderAsync(ValueProviderFactoryContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException(nameof(context));
        }

        var valueProvider =
            new SnakeCaseQueryValueProvider(BindingSource.Query, context.ActionContext.HttpContext.Request.Query, CultureInfo.CurrentCulture);

        context.ValueProviders.Add(valueProvider);

        return Task.CompletedTask;
    }
}
```

<br/>在 program.cs 加上，讓 Query parameters 接受 snake case
```csharp
builder.Services.AddControllers(options => options.ValueProviderFactories.Add(new SnakeCaseQueryValueProviderFactory()));
```

### 讓 swagger 在 Query parameters 的名稱也變成 snake case

```csharp
public class SnakeCaseParameterOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        if (operation.Parameters == null) operation.Parameters = new List<OpenApiParameter>();
        else { 
            foreach(var item in operation.Parameters)
            {             
                item.Name = item.Name.ToSnakeCase();
            }              
        }
    }
}
```

<br/>在 program.cs 加上
```csharp
builder.Services.AddSwaggerGen(s => s.OperationFilter<SnakeCaseParameterOperationFilter>());
```

---

## 讓 model 驗證訊息也回傳 snake case

另一個問題是 model 驗證的 property name 仍是大寫, 我們需要再進一步加工
```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "00-7c56593efe9452ecba145474817c0063-b2393d5143958b47-00",
  "errors": {
    "last_name": [
      "The LastName field is required."
    ],
    "first_name": [
      "The FirstName field is required."
    ]
  }
}
```

```csharp
public class ValidationProblemDetails : ProblemDetails
{
    // 400 status code is usually used for input validation errors
    public const int ValidationStatusCode = (int)HttpStatusCode.BadRequest;

    public ValidationProblemDetails(ICollection<ValidationError> validationErrors)
    {
        ValidationErrors = validationErrors;
        Status = ValidationStatusCode;
        Title = "Request Validation Error";
    }

    public ICollection<ValidationError> ValidationErrors { get; }

    public string RequestId => Guid.NewGuid().ToString();
}
```

```csharp
public class ValidationProblemDetailsResult : IActionResult
{
    public async Task ExecuteResultAsync(ActionContext context)
    {
        var modelStateEntries = context.ModelState
            .Where(e => e.Value.Errors.Count > 0)
            .ToArray();

        var errors = new List<ValidationError>();

        if (modelStateEntries.Any())
        {
            foreach (var (key, value) in modelStateEntries)
            {
                errors.AddRange(value.Errors
                    .Select(modelStateError => new ValidationError(
                        name: key.ToSnakeCase(),
                        description: modelStateError.ErrorMessage)));
            }
        }

        await new JsonErrorResponse<ValidationProblemDetails>(
            context: context.HttpContext,
            error: new ValidationProblemDetails(errors),
            statusCode: ValidationProblemDetails.ValidationStatusCode).WriteAsync();
    }
}
```

```csharp
public class ValidationError
{
    public ValidationError(string name, string description)
    {
        Name = name;
        Description = description;
    }

    public string Name { get; }

    public string Description { get; }
}
```

```csharp
public class JsonErrorResponse<T>
{
    private readonly HttpContext _context;
    private readonly T _error;
    private readonly int _statusCode;

    public JsonErrorResponse(HttpContext context, T error, int statusCode)
    {
        _context = context;
        _error = error;
        _statusCode = statusCode;
    }

    public Task WriteAsync()
    {
        _context.Response.ContentType = "application/json";
        _context.Response.StatusCode = _statusCode;

        return _context.Response.WriteAsync(_error.ToSnakeCase());
    }
}
```

<br/>在 program.cs 加上
```csharp
builder.Services.ConfigureApiBehaviorOptions(options => options.InvalidModelStateResponseFactory = ctx => new ValidationProblemDetailsResult());
```

<br/>那麼驗證訊息就變成如下
```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "request_id": "00-7c56593efe9452ecba145474817c0063-b2393d5143958b47-00",
  "validation_errors": [
    {
        "name": "last_name",
        "description": The last_name field is required.
    },
    {
        "name": "first_name",
        "description": The first_name field is required.
    }
  ]
}
```

---

參考自
- [How to accept and return snake case formatted JSON in ASP Web API](https://maximgorbatyuk.github.io/blog/development/2021-02-20-snake-case-and-asp-net-core/)
- [FromQuery parameters from camelCase/PascalCase to snake_case .Net Core](https://stackoverflow.com/questions/69197645/fromquery-parameters-from-camelcase-pascalcase-to-snake-case-net-core)