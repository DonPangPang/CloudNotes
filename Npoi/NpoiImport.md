# Npoi导入Excel

Npoi导入Excel其实只要读成DataTable就可以随意操作了, 比如转为Entity...

By: 胖纸不争
NetCore🐧群: 743336452

核心代码:

```csharp
public class ExcelImport
{
    public string FilePath { get; set; }
    public string SheetName { get; set; }

    private DataTable _dataTable;

    /// <summary>
    /// 初始化文件路径和Sheet名称
    /// </summary>
    /// <param name="filePath">  </param>
    /// <param name="sheetName"> </param>
    public ExcelImport(string filePath, string sheetName = "Sheet1")
    {
        FilePath = filePath;
        SheetName = sheetName;
    }

    /// <summary>
    /// 转为指定类型
    /// </summary>
    /// <typeparam name="T"> </typeparam>
    /// <returns> </returns>
    public List<T> ToList<T>() where T : IExcelStruct
    {
        if (_dataTable is null) ToDataTable();

        var list = new List<T>();
        var type = typeof(T);
        var properties = type.GetProperties();
        foreach (DataRow row in _dataTable.Rows)
        {
            var t = Activator.CreateInstance<T>();
            foreach (var property in properties)
            {
                var prop_name = property.GetCustomAttribute<ExcelColumnAttribute>().Name;
                var value = row[prop_name];
                if (value != DBNull.Value)
                {
                    var d = Convert.ChangeType(value, property.PropertyType);
                    property.SetValue(t, d);
                }
            }
            list.Add(t);
        }
        return list;
    }

    /// <summary>
    /// Excel转为DataTable
    /// </summary>
    /// <returns> </returns>
    private ExcelImport ToDataTable()
    {
        _dataTable = new DataTable();
        using (var fs = new FileStream(FilePath, FileMode.Open, FileAccess.Read))
        {
            var workbook = WorkbookFactory.Create(fs);
            var sheet = workbook.GetSheet(SheetName);
            if (sheet == null)
            {
                throw new Exception("Excel sheet not found");
            }

            var headerRow = sheet.GetRow(0);
            var headerRowCount = headerRow.LastCellNum;
            for (var i = headerRow.FirstCellNum; i < headerRowCount; i++)
            {
                var column = new DataColumn(headerRow.GetCell(i).StringCellValue);
                _dataTable.Columns.Add(column);
            }

            var rowCount = sheet.LastRowNum;
            for (var i = sheet.FirstRowNum + 1; i <= sheet.LastRowNum; i++)
            {
                var row = sheet.GetRow(i);
                var dataRow = _dataTable.NewRow();
                for (var j = row.FirstCellNum; j < headerRowCount; j++)
                {
                    if (row.GetCell(j) != null)
                    {
                        dataRow[j] = row.GetCell(j).ToString();
                    }
                }

                _dataTable.Rows.Add(dataRow);
            }
        }

        return this;
    }
}
```

封装了一个库, 用来快速导入: [Powers.NpoiExcel](https://github.com/DonPangPang/Powers.NpoiExcel)

## Powers.NpoiExcel

Now Support: Import Excel to Entity

1: Create your entity

```csharp
public class User : IExcelStruct
{
    [ExcelColumn(Name = "姓名")]
    public string Name { get; set; }

    [ExcelColumn(Name = "年龄")]
    public int Age { get; set; }

    [ExcelColumn(Name = "性别")]
    public string Gender { get; set; }

    [ExcelColumn(Name = "生日")]
    public DateTime Born { get; set; }
}
```

2: How to use

```csharp
var path = Environment.CurrentDirectory + "/files/test.xlsx";

var data = new ExcelImport(path).ToList<User>();

_testOutputHelper.WriteLine("姓名\t年龄\t性别\t生日");

foreach (var item in data)
{
    _testOutputHelper.WriteLine($"{item.Name}\t{item.Age}\t{item.Gender}\t{item.Born}");
}
```

3: Result or run xUnit.Test

```terminal
 Powers.NpioExcel.TestProject.ExcelTests.Test1
   持续时间: 244 毫秒

  标准输出: 
    姓名	年龄	性别	生日
    张三	20	男	2022/5/1 0:00:00
    李四	21	女	2022/5/2 0:00:00
    王五	22	男	2022/5/3 0:00:00
    赵六	23	女	2022/5/4 0:00:00
    田七	24	男	2022/5/5 0:00:00
    老八	25	女	2022/5/6 0:00:00
    小汉堡	26	男	2022/5/7 0:00:00
    奥里给	27	女	2022/5/8 0:00:00
    嘿嘿嘿	28	男	2022/5/9 0:00:00
```
