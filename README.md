# EntityFrameworkCore Demo

EntityFrameworkCore Demo in a WebAPI project.

Dependencies: .NET Core 2.2, Microsoft.EntityFrameworkCore v2.2

[Medium Blog Post](https://medium.com/@changhuixu/ientitytypeconfiguration-t-in-entityframework-core-3fe7abc5ee7a)

## Usage

1. Apply EF Migration to your local SQL Server database

   ```powershell
   # if using Package Manger Console in Visual Studio
   Update-Database

   # if using console
   dotnet ef database update
   ```

1. Run Web App

   In `WebBlogs.Web` folder, issue command `dotnet watch run`, then in browser navigate to [http://localhost:5000/swagger/index.html](http://localhost:5000/swagger/index.html). In Swagger page, you can try out APIs.

   Or, in Visual Studio, click run "IIS Express".

## EF Core LINQ Expression

### Normal Query

```csharp
return await _dbContext.Authors
                .Where(x => x.AuthorMembership == AuthorMembership.Gold ||
                            x.AuthorMembership == AuthorMembership.Platinum)
                .Select(x => new AuthorViewModel(x))
                .ToListAsync();
```

```sql
Microsoft.EntityFrameworkCore.Database.Command[20101]
Executed DbCommand (44ms) [Parameters=[], CommandType='Text', ommandTimeout='30']
SELECT [x].[Id], [x].[AuthorMembership], [x].[FirstName], [x].LastName]
FROM [Authors] AS [x]
WHERE ([x].[AuthorMembership] = 2) OR ([x].[AuthorMembership] = 3)
```

### Expression

```csharp
public class Author
{
    ...
    public static Expression<Func<Author, bool>> GoldAndUp = x =>
            x.AuthorMembership == AuthorMembership.Gold ||
            x.AuthorMembership == AuthorMembership.Platinum;
    ...
}
return await _dbContext.Authors
                .Where(Author.GoldAndUp)
                .Select(x => new AuthorViewModel(x))
                .ToListAsync();
```

```sql
Microsoft.EntityFrameworkCore.Database.Command[20101]
Executed DbCommand (4ms) [Parameters=[], CommandType='Text', ommandTimeout='30']
SELECT [x].[Id], [x].[AuthorMembership], [x].[FirstName], [x].LastName]
FROM [Authors] AS [x]
WHERE ([x].[AuthorMembership] = 2) OR ([x].[AuthorMembership] = 3)
```

## Expression and Projection

```csharp
await _dbContext.Authors.Select(x => new AuthorViewModel(x)).ToListAsync();
```

```sql
Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [x].[Id], [x].[AuthorMembership], [x].[FirstName], [x].[LastName]
FROM [Authors] AS [x]
```

```csharp
public static Expression<Func<Author, AuthorViewModel>> Projection1
            => x => new AuthorViewModel
            {
                Id = x.Id,
                FirstName = x.FirstName,
                LastName = x.LastName
            };

await _dbContext.Authors.Select(AuthorViewModel.Projection1).ToListAsync();
```

```sql
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [x].[Id], [x].[FirstName], [x].[LastName]
FROM [Authors] AS [x]
```

```csharp
public static Expression<Func<Author, AuthorViewModel>> Projection2
            => x => new AuthorViewModel(x);

return await _dbContext.Authors.Select(AuthorViewModel.Projection2).ToListAsync();
```

```sql
Executed DbCommand (4ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [x].[Id], [x].[AuthorMembership], [x].[FirstName], [x].[LastName]
FROM [Authors] AS [x]
```

```csharp
var authors = await _dbContext.Authors.ToListAsync();
return authors.Select(AuthorViewModel.Projection1.Compile()).ToList();
```
