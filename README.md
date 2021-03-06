[![Build Status](https://travis-ci.org/dobrou/CsvLINQPadDriver.svg?branch=master)](https://travis-ci.org/dobrou/CsvLINQPadDriver)

CsvLINQPadDriver
==

CsvLINQPadDriver is LINQPad data context dynamic driver for querying CSV files.

You can query data in CSV files with LINQ, just like it would be regular database. No need to write custom data model, mappings etc.

Driver automatically generates new data types for every CSV file with corresponding properties and mappings for all columns.
Based on column and file names, possible relations between CSV tables are detected and generated.

Website
--
https://github.com/dobrou/CsvLINQPadDriver

Download
--
- v1.6
  - **[CsvLINQPadFileOpen.exe](https://goo.gl/mTvTSK)** - It's installer and CSV context loader in one package.
      - LINQPad **v4** only
      - Automatically installs CSV driver into LINQPad. 
      - Loads new CSV context with files from command-line. You can associate this executable to open CSV files, so they will load directly into LINQPad.
  - [CsvLINQPadDriver.lpx](https://goo.gl/k9taJj) - Manual installation.
      - LINQPad **v4/v5** compatible

Example
--
Let's have 2 CSV files:
```
Lakes.csv:
LakeName,ID
lake1,1
etc.

Fishes.csv:
FishName,FishID,LakeID
xyz,1,1
etc.
```

CsvLINQPadDriver will generate data context similar to:
```csharp
public class CsvDataContext {
  public IEnumerable<Lakes> Lakes { get; set; }
  public IEnumerable<Fishes> Fishes { get; set; }
}
public class Lakes {
  public string LakeName { get; set; }
  public string ID { get; set; }
  public IEnumerable<Fishes> Fishes { get; set; } //All Fishes where Lakes.ID == Fishes.LakeID
}
public class Fishes {
  public string FishName { get; set; }
  public string FishID { get; set; }
  public string LakeID { get; set; }
  public IEnumerable<Lakes> Lakes { get; set; } //All Lakes where Lakes.ID == Fishes.LakeID
}
/// and mappings etc.
```

And you can query data with LINQ like:
```csharp
from lake in Lakes
where lake.LakeName.StartsWith("S") && lake.Fishes.Any()
select new { lake, fishes = lake.Fishes }
```

Prerequisites
--
- LINQPad 4 
- .NET Framework 4.0/4.5

Installation
--
- Using CsvLINQPadFileOpen.exe:
  - Driver will be installed automatically on first run.
- Or driver can be installed manually:
  1. Get CsvLINQPadDriver.lpx file
  - Open LINQPad
  - Click `Add connection` main window
  - Click button `View more drivers...`
  - Click button `Browse` and select downloaded .lpx file

Usage
--
- CSV context can be added to LINQPad same way as any other context.
  - Click `Add connection`
  - Select `CSV Context Driver` and click `Next`
  - Enter CSV file names or Drag&Drop files from explorer. 
    Optionally configure other options. 
  - Query your data
- Or run `CsvLINQPadFileOpen.exe` with CSV files as arguments. 
  - `CsvLINQPadFileOpen.exe mydata.csv`
  - New CSV data context will be created and loaded in LINQPad

Configuration Options
--
- **CSV Files** - list of CSV files and directories. Type one file/dir per line or Drag&Drop files from explorer. Supports special wildcards: `*` and `**`. 
  - `c:\x\*.csv` - all files in folder `c:\x`
  - `c:\x\**.csv` - all files in folder `c:\x` and all sub-directories
- CSV Separator - character used to separate columns in files. Can be `,`,`\t`, etc. If empty, separator is auto-detected.
- Detect relations - if checked, driver will try to detect and generate relations between files.
- Hide relations from .Dump() - if checked - LINQPad will not show relations content in .Dump(). This prevents loading too many data.
- Cache CSV data in memory 
  - if checked - parsed rows from file are cached in memory. This cache survives multiple query runs, even when query is changed. Cache is cleared as soon as LINQPad clears Application Domain of query.
  - if unchecked - disable cache. Multiple enumerations of file content results in multiple reads and parsing of file. Can be significantly slower for complex queries. Significantly reduces memory usage. Useful when reading very large files.
- Ignore files with invalid format - files with strange content not similar to CSV format will be ignored.
- Debug info - additional debug information will be available. For example generated Data Context source.
- Remember this connection - if checked, connection info will be saved and available after LINQPad restart.

Relations
--
There is no definion of relations between CSV files, but we can guess some relations from files and columns names.
Relations between `fileName.columnName` are detected in cases similar to following examples:
- `Fishes.LakeID` <-> `Lakes.ID`
- `Fishes.LakesID` <-> `Lakes.ID`
- `Fishes.LakeID` <-> `Lakes.LakeID`
- `Fishes.ID` <-> `Lakes.FishID`

Performance
--
When executing LINQ query on CSV context:
- Only files used in query are loaded from disk.
- As soon as any record from file is accessed, whole file is loaded into memory.
- Relations are lazy evaluated and retreived using cached Lookup tables.

Don't expect performance comparable with SQL server. But for reasonably sized CSV files there should not be any problem. 

Data types
--
Everything is string. Because there is no data type info in CSV files, this is best we can do.
However, driver provides few extension methods providing easy conversion from string to nullable of common types:
`"123".ToInt()` , `"123".ToDouble()`, etc.

Known Issues / TODO
--
- Some strange Unicode chracters in column names may cause errors in generated data context source code.
- Writing changed objects back to CSV is not directly supported, there is no `.SubmitChanges()` . But you can use LINQPad's `Util.WriteCsv`.

Author
--
- Martin Dobroucký (dobrou@gmail.com)

Credits
--
- [LINQPad](http://www.linqpad.net/)
- [CsvHelper](https://github.com/JoshClose/CsvHelper) - CSV files parsing

License
--
[MIT](http://opensource.org/licenses/MIT), see LICENSE file for details.
 
