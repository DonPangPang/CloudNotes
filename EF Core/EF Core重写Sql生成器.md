# [EF Core] 重写SQL生成器

使用`EF Core`时最烦的就是生成的某些 SQL 其实并不是你想要的结果，例如**外键约束**等等。

一个最简单的例子就是，因为`EF Core`会根据导航属性生成外键约束等原因，导致很多开发者抛弃了更易维护的`Code First`模式，而转为`Db First`以获取更自由的数据库结构。

其实我们可以通过重写`EF Core`的`MigrationsSqlGenerator`来解决：

```csharp
public class CustomMigrationsSqlGenerator : MigrationsSqlGenerator
{
    public CustomMigrationsSqlGenerator(MigrationsSqlGeneratorDependencies dependencies, IMigrationsAnnotationProvider migrationsAnnotations) : base(dependencies)
    {
    }

    protected override void Generate(Microsoft.EntityFrameworkCore.Migrations.Operations.CreateTableOperation operation, IModel? model, MigrationCommandListBuilder builder, bool terminate = true)
    {
        operation.ForeignKeys.Clear();
        base.Generate(operation, model, builder, terminate);
    }
}
```

```csharp
public class CustomDbContext : DbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // ...
        optionsBuilder.ReplaceService<IMigrationsSqlGenerator, CustomMigrationsSqlGenerator>();
        // ...
    }
}
```

上述代码便提供了移除`ForeignKeys`即表结构中的外键约束。当然，需要在`DbContext`中重写`OnConfiguring`方法，使用`optionsBuilder.ReplaceService`将默认的SQL生成器，替换为我们自己实现的部分，就可以在`EF Core`生成数据库结构时，直接去除外键约束了。

初次之外，根据`MigrationsSqlGenerator`暴露出来的 API 来看，我们还能做很多事情，例如重写字段的类型或者在实体上做文章，具体需要大家根据场景进行探寻了。
