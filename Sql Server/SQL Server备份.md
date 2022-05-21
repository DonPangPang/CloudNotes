# NetCore中将SQLServer数据库备份为Sql脚本

描述:

最近写项目收到了一个需求, 就是将`SQL Server`数据库备份为Sql脚本, 如果是My Sql之类的还好说, 但是在网上搜了一大堆, 全是教你怎么操作`SSMS`的, 就很d疼!

解决方案:

通过各种查找资料, 还有一些老哥的帮助, 找到了解决方案:

通过`Microsoft.SqlServer.Management.Smo`, `Microsoft.SqlServer.Management.Sdk.Sfc`, `Microsoft.SqlServer.Management.Common`来解决, 但是不巧的是, 这个方法可能只适用于`.Net Framework`, 并且微软已经提供一个合集的类库封装为`Microsoft.SqlServer.Scripts`. 但是我是一个`Net5`的项目!

但是最后还是找到了, 微软封装了一个其它包...emm`Microsoft.SqlServer.SqlManagementObjects`, 此类库可以适用于`Net Core`.

- [NetCore中将SQLServer数据库备份为Sql脚本](#netcore中将sqlserver数据库备份为sql脚本)
  - [基本使用](#基本使用)
  - [开箱即用(封装库Powers.DbBackup)](#开箱即用封装库powersdbbackup)
    - [配置DbBackup](#配置dbbackup)
      - [1. In `Startup.cs`(Net5):](#1-in-startupcsnet5)
      - [2. In `Program.cs`(Net6):](#2-in-programcsnet6)
    - [使用方法](#使用方法)

By: 胖纸不争
NetCore🐧群: 743336452

## 基本使用

```C#
Server server = new Server(
    new ServerConnection(
        // 服务器IP
        _dbBackupOptions.ServerInstance,
        // 登录名
        _dbBackupOptions.Username,
        // 密码
        _dbBackupOptions.Password
        )
);
// 获取数据库
Database templateDb = server.Databases[_dbBackupOptions.DatabaseName];
// 脚本导出路径
string sqlFilePath = string.Format("{0}.sql", $"{dbBackupPath}/{name}");
// 自定义规则
var startWith = _dbBackupOptions.FormatTables.Where(x => x.EndsWith("*")).Select(x => x.TrimEnd('*'));
var endWith = _dbBackupOptions.FormatTables.Where(x => x.StartsWith("*")).Select(x => x.TrimStart('*'));

if (_dbBackupOptions.FormatTables is not null && _dbBackupOptions.FormatTables.Any())
{
    foreach (Table tb in templateDb.Tables)
    {
        if (_dbBackupOptions.FormatTables.Contains(tb.Name) ||
            startWith.Where(x => tb.Name.StartsWith(x)).Any() ||
            endWith.Where(x => tb.Name.EndsWith(x)).Any())
        {
            // 按表获取Sql
            IEnumerable<string> sqlStrs = tb.EnumScript(_dbBackupOptions.ScriptingOptions);
            // 将Sql向文件中追加
            using (StreamWriter sw = new StreamWriter(sqlFilePath, true, Encoding.UTF8))
            {
                foreach (var sql in sqlStrs)
                {
                    sw.WriteLine(sql);
                    sw.WriteLine("GO");
                }
            }
        }
    }
}
else
{
    foreach (Table tb in templateDb.Tables)
    {
        IEnumerable<string> sqlStrs = tb.EnumScript(_dbBackupOptions.ScriptingOptions);
        using (StreamWriter sw = new StreamWriter(sqlFilePath, true, Encoding.UTF8))
        {
            foreach (var sql in sqlStrs)
            {
                sw.WriteLine(sql);
                sw.WriteLine("GO");
            }
        }
    }
}
```

## 开箱即用(封装库Powers.DbBackup)

我针对这个封装了一个类库, `Powers.DBackup`方便简单使用.

GitHub地址: [Powers.DbBackup](https://github.com/DonPangPang/Powers.DbBackup)

### 配置DbBackup

#### 1. In `Startup.cs`(Net5):

```csharp
services.AddDbBackup();
```

`appsettings.json`:

```json
"DbBackupOptions": {
    // remote server
    "ServerInstance": "192.168.31.36",
    // database username
    "Username": "sa",
    // password
    "Password": "sa123.",
    // ddatabase name
    "DatabaseName": "PumInfoShop",
    // output options
    "ScriptingOptions": {
      "DriAll": false,
      "ScriptSchema": true,
      "ScriptData": true,
      "ScriptDrops": false
    },
    // match rules
    /**
     * Include 3 rules:
     * 1. Full name: UserTable
     * 2. Start with: Sys*
     * 3. End with: *Table
     */
    "FormatTables": []
  }
```

**OR**

```csharp
services.AddDbBackup(opts =>
{
    opts.ServerInstance = "127.0.0.1";
    opts.Username = "sa";
    opts.Password = "123456";
    opts.DatabaseName = "TestDb";
    opts.ScriptingOptions = new ScriptingOptions
    {
        DriAll = true,
        ScriptSchema = true,
        ScriptData = true,
        ScriptDrops = false
    };
    /**
     * Include 3 rules:
     * 1. Full name: UserTable
     * 2. Start with: Sys*
     * 3. End with: *Table
     */
    opts.FormatTables = new string[] { "Sys*", "Log*", "UserTable", "*Table" };
});
// Or this way
//services.AddDbBackup(opts => new DbBackupOptions
//{
//    ServerInstance = "127.0.0.1",
//    Username = "sa",
//    // .....
//});
```

#### 2. In `Program.cs`(Net6):

```csharp
builder.Services.AddDbBackup();
```

`appsettings.json`:

```json
"DbBackupOptions": {
    "ServerInstance": "192.168.31.36",
    "Username": "sa",
    "Password": "sa123.",
    "DatabaseName": "PumInfoShop",
    "ScriptingOptions": {
      "DriAll": false,
      "ScriptSchema": true,
      "ScriptData": true,
      "ScriptDrops": false
    },
    "FormatTables": []
  }
```

**OR**

```csharp
builder.Services.AddDbBackup(opts =>
{
    opts.ServerInstance = "127.0.0.1";
    opts.Username = "sa";
    opts.Password = "123456";
    opts.DatabaseName = "TestDb";
    opts.ScriptingOptions = new ScriptingOptions
    {
        DriAll = true,
        ScriptSchema = true,
        ScriptData = true,
        ScriptDrops = false
    };
    /**
     * Include 3 rules:
     * 1. Full name: UserTable
     * 2. Start with: Sys*
     * 3. End with: *Table
     */
    opts.FormatTables = new string[] { "Sys*", "Log*", "UserTable", "*Table" };
});

// Or this way
//builder.Services.AddDbBackup(opts => new DbBackupOptions
//{
//    ServerInstance = "127.0.0.1",
//    Username = "sa",
//    // .....
//});
```

### 使用方法

```csharp
[HttpGet]
public async Task<ActionResult> StartDbBackup()
{
    var rootPath = "D:/";
    var fileName = DateTime.Now.ToString("yyyyMMddhhmmss"); // No ".sql" suffix is required.
    var (path, size) = await DbBackupExtensions.StartBackupAsync(rootPath, fileName);// path is full path

    return Ok(new
    {
        Path = path,
        Size = size
    });
}

[HttpGet]
public async Task<ActionResult> DeleteDbBackup(string filePath)
{
    var (res, msg) = await DbBackupExtensions.DeleteBackup(filePath);

    if (res)
    {
        return Ok(msg);
    }
    else
    {
        return NotFound(msg);
    }
}
```
