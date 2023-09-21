---
theme: dracula
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
transition: slide-left
title: FagDag - Mastering EF Core 
mdc: true
---

# FagDag - Mastering EF Core

Learning from common issues when using EF Core

<div class="pt-12">
  Kenneth Fossen
</div>
<v-click>
<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Let's get going <carbon:arrow-right class="inline"/>
  </span>
</div>
</v-click>
<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/spydx/kefo-ef-core" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---
transition: fade-out
layout: center
---

# Todays topics

<v-clicks>

- What is EF Core
- EF Core 5 -> 6 -> 7 -> 8
- Pause
- 7 Deadly Sins of EF Core
- Practice / Challenges
- Sources

</v-clicks>

---
layout: image-right
transition: fade-out
image: ./kbo.jpg
---

# $ whoami

```hs {1|2-5|6-9|10-12|all}
$ whoami
{
     name: Kenneth Fossen,
     dep: Dragefjellet@Bouvet,
     email: kenneth.fossen@bouvet.no,
     edu: [
         Bachelor i Datatryggleik
         Master i Programvare Utvikling
     ],
     work: [
         Helse Vest IKT Drift & Sikkerhet
     ]
     current: [
          project: { 
              name: CommonLibrary@Spine 
          },
          Software Developer,
          Security Champion,
          #rustaceans
      ],
 }
```
---
transition: fade-out
layout: default
---

# Overview

<Toc maxDepth="1"></Toc>


<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->
---
layout: default
---

# What is EF Core

EF Core *can* serve as a object-relational-mapper (O/RM)

<v-clicks>

- Enabling .NET developers to work with a database using .NET Objects
- Eliminates the need for most of the data-access code that typically need to be written.
- It gives you migrations that allow evolving the database as the model changes.

</v-clicks>

---
layout: two-cols-header
---

### C# Code

```ts {all|8-13}
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    protected override void OnConfigure(DbContextOptionsBuilder o) {
      o.UseSqlServer("connectionString");
    }
}
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    public int Rating { get; set; }
}
```

<div v-click="1">
  <Arrow x1="350" y1="300" x2="450" y2="350" />
</div> 
::right::

### Database table

| BlogId | Url | Rating |
| --- | --- | --- |
| 1 | http://blog.kefo.no | 5 |

---
layout: full

---

## LINQ to SQL

```ts {all|5-6,11-12|6,13|7,14|8|all}
// C# Code
using var db = new BloggingContext();

var blogs = db
    .Blogs
    .Where(b => b.Rating > 3)
    .OrderBy(b => b.Url)
    .ToList(); // Query and fetch data into memory

// SQL OUTPUT
SELECT a."BlogId", a."Url", a."Rating" 
FROM "Blogs" as a
WHERE a."Rating" > 3
ORDER BY a."Url"
```
---
layout: two-cols
title: "EF vs EF Core"
hideInToc: true
---
::default::
# EF 

<v-clicks>

- EF6 is a ORM for .NET Framework
- Supports also .NET Core
- No longer developed

</v-clicks>

::right::
# EF Core

<v-clicks>

- EF Core is modern ORM for .NET
- Support for LINQ
- Change Tracking
- Updates
- Schema Migrations

</v-clicks>
<!--
	EF6 is a object-relational mapper desined fro .NET Framework, but with support for .NET Core.
  This is no longer actively developed.

  EF Core is the modern object-database mapper for .NET
  Support for LINQ queries, change tracking, updates, and schema migrations.
-->


---
layout: iframe-right
url: https://dapperlib.github.io/Dapper/
hideInToc: true

---
# Dapper

<v-clicks>

- A simple object mapper for .NET
- Know to be the fastest
- Low memory footprint (compared to EF Core)
- Complete control over your SQL Queries
- Goto solution when performance is critical

</v-clicks>
<!---
	It is a simple object mapper for .NET, know to be the fastest, and has the least memory overhead.

	You can write raw SQL parameterized queries. 
  
  It give good performance and complete control over the queries, and is considered the goto solution when it is performance critical.
-->

---
layout: default
hideInToc: true

---

# Project: Dapper vs EF Core

```ts {all} {maxHeight:'400px'}
var sql = @"
    SELECT l.[Id]
          ,l.[Name]
          ,l.[Description]
          ,l.[IsCaseSensitive]
          ,l.[IsForeignObject]
          ,l.[IsGlobal]
          ,scopeType.[Name] as ScopeType
          ,case when l.[NamesCasing] = {=UpperCase} then 1 else 0 end as AreNamesUpperCase
          ,l.[Alias]
          ,l.[AttachmentKey]
      FROM [dbo].[Library] l
      LEFT JOIN [dbo].[Library] scopeType ON scopeType.Id = l.ScopeTypeId 
    WHERE l.IsValid = 1
    ORDER BY l.[Name]
    SELECT DISTINCT lt.LibraryId, t.Name
      FROM LibraryTag lt
      JOIN Tag t ON t.Id = lt.TagId
      JOIN Library l ON l.Id = lt.LibraryId
    WHERE l.IsValid = 1
    SELECT DISTINCT lg.LibraryId, g.Name
      FROM LibraryAccessGroup lg
      JOIN AccessGroup g ON g.Id = lg.AccessGroupId
      JOIN Library l ON l.Id = lg.LibraryId
    WHERE l.IsValid = 1
";

using (var multi = _connection.QueryMultiple(sql, new { Casing.UpperCase }))
{
    var libraries = multi.Read<LibraryListItem>().ToList();
    var libraryIndex = libraries.ToDictionary(keySelector: x => x.Id);

    foreach (var tag in multi.Read<(int LibraryId, string Name)>())
    {
        libraryIndex[tag.LibraryId].Tags.Add(tag.Name);
    }

    foreach (var accessGroup in multi.Read<(int LibraryId, string Name)>())
    {
        libraryIndex[accessGroup.LibraryId].AccessGroups.Add(accessGroup.Name);
    }

    return Ok(libraries);
}
```
---
layout: default
---

```ts
var libraryListItems = _context.
  Library
  .TagWith("LibraryList LINQ")
  .AsNoTracking()
  .Where(library => library.IsValid)
  .OrderBy(library => library.Name)
  .Select(library => new LibraryListItem
  {
      Id = library.Id,
      Name = library.Name,
      Desc = library.Description,
      Alias = library.Alias,
      IsGlobal = library.IsGlobal,
      ScopeType = library.ScopeType.Name,
      IsForeignObject = library.IsForeignObject,
      AreNamesUpperCase = library.IsCaseSensitive,
      AttachmentKey = library.AttachmentKey,
      Tags = library
          .LibraryTags
          .Select(tag => tag.Tag.Name)
          .ToList(),
      AccessGroups = library
          .LibraryAccessGroups
          .Select(group => group.AccessGroup.Name)
          .ToList()
  });

return Ok(libraryListItems);
```
---
layout: default
---
# EF Core Changes

<div class="gird-flow-row">
  <div class="col-span-2">
    <h2> 5 -> 6 -> 7 -> 8</h2>
  </div>
  <img src="/efcore.png" class="ml-100 mt-10 h-60">
</div>

---
layout: default
---
## EF Core 5

<br>
<v-click>

  <p class="text-3xl"><mdi-alert class="text-yellow"/> Query Strategy Changed <mdi-alert class="text-yellow"/></p>

  - Performance
  - Complex Queries that was fast **may** be slow

</v-click>

<br>

<v-click>

Lets study this code

```ts {all|2|3|all}
var artists = context
    .Artists
    .Include(e => e.Albums)
    .ToList();
```
</v-click>




<!---

	There was a lot that happened in this version, but one of the most important changes that came to be is the change in how queries have been done agains the database.  

	In the previous versions, all requests to the database was sent as a Single Query.

	This means that a very complex query was sent as multiple queries to the database. This can be a good thing, many small queries, but is also opens up for a problem with data consistency in high throughput systems, but it also means many roundtrips to fetch data.

	In EF Core 5, this complex query was now compiled down to one SQL statement and sent to the database. This for data consistency is a good thing, performance, it can be a horrible thing.

-->
---
layout: two-cols
---
::default::
## Split Query

```sql
SELECT a."Id", a."Name"
FROM "Artists" AS a
ORDER BY a."Id"

SELECT a0."Id", a0."ArtistId", a0."Title", a."Id"
FROM "Artists" AS a
INNER JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."
```
::right::

## Combined Query

```sql
SELECT a."Id", a."Name",
       a0."Id", a0."ArtistId", a0."Title"
FROM "Artists" AS a
LEFT JOIN "Album" AS a0 ON a."Id" = a0."ArtistId"
ORDER BY a."Id", a0."Id"

```


---
layout: default
---
## 3 Options

```ts {all|4}
var blogs = context
    .Blogs
    .Include(blog => blog.Posts)
    .AsSplitQuery()
    .ToList();
```
```ts {all|4}
var blogs = context
    .Blogs
    .Include(blog => blog.Posts)
    .AsSingleQuery()
    .ToList();
```
```ts {all|5}
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlServer(
      @"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True",
      o => o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery)
    );
}
```
---
layout: default
hideInToc: true

---
## From my project: EF optimization to handle timeout

Old strategy fetched CableSpecWire for facility ROM in **11 minutes**, **using 90%** of available compute power.

<v-click>

```ts
if (specification is CodeSetWithReferencedCodesSpecification)
{
    query = query.AsSplitQuery(); // Optimization for large sets
}
```

With this small change the same query **takes 6 seconds**.

</v-click>
---
layout: default
hideInToc: true
---

## EF Core 6

<br>

<v-clicks>

- SQL Server temporal tables
- Migration Bundles
- Pre-convention model configuration
- Compiled models
- Performance
	- 70% better query performance than EF Core 5.0
	- 31% afaster executing untracked queries
	- Heap allocations have been reduced by 43%

</v-clicks>


<!--
- Pre-convention model configuration
	- You can configure default matppers for e.g. all string in a model/databasecontext.

Compiled models
 -	For databases with many relations, 100s to 1000s entity types and relationsships.
- 	The startup time until the first operation is done on the DbContext, may be more than expected, and Compiled models may helps in this case.

-->
---
layout: default
hideInToc: true

---

## EF Core 7

- JSON Columns
- ExecuteUpdate and ExecuteDelete (BulkUpdates)
  - Improved commands are sent to the database, can only act on a single table
  - ⚠️ May cause out of sync with the Entity Tracker ⚠️
- Faster `SaveChanges`  
  - 4x times faster than EF Core 6  
  - Has fewer roundtrips to the database when saving
- Stored Procedure Mapping  
  - You can now map in the `modelBuilder` of the `DbContext` for an Entity what stored procedures should be used, and what parameters to submit to them.

---
layout: default
hideInToc: true
---
## EF Core 7 Continue
- Query Enhancement  
  `.GroupBy()` can now be the final operator in a query.  
  `.GroupBy(s ⇒ s.Author)` group on a entity type  
  `.GroupJoin()` can be used as the final operator  
  `Contains()` now accepts `IReadOnlyCollections`
- Model Building Enhancement

	Indices can be ascending or descending `.IsDescending()` 

    ```bash
    modelBuilder
        .Entity<Post>()
        .HasIndex(post => post.Title)
        .IsDescending();
    ```
---
layout: default
hideInToc: true
---

## EF Core 7: Breaking change

<br>

### Database Connection Strings 

`Encrypt` defaults to `true` for SQL Server connections

- Consequences, you need a **valid** certificate on the SQL Server.
- The client must trust this certificate
- 2 ways to mitigate this: 
  - `TrustServerCertificate=True`
  - `Encrypt=False`

---
layout: default
hideInToc: true
---

# EF Core 8

It will be released this coming November together with .NET8.


- Raw SQL queries for unmapped types  
  The Query `.SqlQuery<T>(“SELECT * FROM T”)` is parameteriezed, SQLi safe.
- DateOnly/TimeOnly support for SQL Server  
  You can use DateOnly / TimeOnly types from .NET6. They are now introduced in EF Core 8.  

  💡 There is also a package to use DateOnly / TimeOnly if you need it EF Core 6 / EF Core 7.

  NuGet `ErikEJ.EntityFrameworkCore.SqlServer.DateOnlyTimeOnly`
- Close to Dapper Performance and Memory Consumption

---
layout: default
hideInToc: true
---
### EF Core 8:	Breaking changes

- The scaffolding of a database now translates to DateOnly and TimeOnly for `date` and `time`.

- `Contains` in LINQ queries may stop working on older SQL Server versions.

- Only supports SQL Server 2014 above

<!--

	Previously, when the `Contains` operator was used in LINQ queries with a parameterized value list. EF generated SQL that was inefficient but worked on all SQL Server versions.

	Starting with EF Core 8, EF now generates SQL that is more efficient but is unsupported on SQL Server 2014 and below.
-->

---
layout: fact
---
# PAUSE
---
layout: image-right
image: ./boisy_kion.jpg
hideInToc: true
---
# After pause

# 7 Deadly Sins 
# Practice 
# Challenges

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
layout: fact
---

# 7 Deadly Sins

---
layout: two-cols-header
hideInToc: true
---

# Deadly Sin: 1

::left:: 
Casting `IQueryable` to `IEnumerable`
<br>

<v-clicks>

Examples
- `.ToList()`
- `.AsEnumerable()`

<p class="text-3xl"><mdi-alert class="text-yellow"/> Will fetch all the data! <mdi-alert class="text-yellow"/></p>

</v-clicks>

::right::

<v-click>

## Code Example

```ts {1-3|5-9|15-17|10-13|1,15|all}
public IEnumerable<Sales> GetSales() {
  return SalesDbContex.Sales;
}

public int CountSalesInDb() {
  return GetSales().Count();
  // SELECT * FROM Sales 
  // in memory count the collection
}
public int CountSalesInDb2() {
  return GetSales2().Count();
  // SELECT COUNT(*) FROM Sales;
}

private IQueryable<Sale> GetSales2() {
   return SalesDbContext.Sales;
}
```

</v-click>
---
layout: default
hideInToc: true
---

# Deadly Sin: 2

## Not using `.AsNoTracking()`

If you are only reading from the database, there there is no need to track the entity.

<v-click>

<p>Perfomance <mdi-lightning-bolt class="text-2xl text-yellow" /></p>

- Can use ~4x **less** memory
- Can be ~5x **faster**

<p class="text-2xl"><mdi-alert class="text-yellow" /> Not for SQLite DB Provider <mdi-alert class="text-yellow" /></p>
</v-click>

---
layout: image
image: /asnotracking.png
hideInToc: true
---

---
layout: two-cols-header
hideInToc: true
---

# Deadly Sin: 3 

::left::
## Explicit Joins

`.Include(x => x.Customers)`

1. They are always included
2. We get all their columns
3. Forget to remove them


<div v-click="6">

## Deadly Sin: 4

Getting all the columns

  <div v-click="7">
  ~2.5 faster with `Select()` and implicit joins.
  </div>
</div>

::right::

# Example

```ts {1-5|4|7-18|16-17|4|10-17|all}
var explicitQuery = _dbContext.Sales
    .AsNoTracking()
    .TagWithContext()
    .Include(x => x.SalesPerson)
    .Where(x => x.SalesPersonId == 1);

var implicitQuery = _dbContext.Sales
    .AsNoTracking()
    .Where(x => x.SalesPersonId == 1)
    .Select(x => new SalesWithSalesPerson {
        CustomerId = x.CustomerId,
        SalesId = x.SalesPersonId,
        ProductId = x.ProductId,
        Quantity = x.Quantity
        SalesPersonId = x.SalesPersonId,
        SalesPersonFirstName = x.SalesPerson.FirstName,
        SalesPersonLastName = x.SalesPersons.LastName
    });
```

---
layout: two-cols-header
hideInToc: true
---

# Deadly Sin: 5

## No pagination

- Fetch all data
- Process in memory

::left::

```ts {all|6-11|all}
var query = context
    .Sales
    .AsNotTracking()
    .Where(x => x.SalesPersonId == salesPersonId);

var result = await query.ToListAsync(ct);
int count = result.Count;

result = result
    .Skip(page * pageSize)
    .Take(pageSize);

return (count, result);
```


::right::

```ts {all|1-6|8-12|all}
var query = context
    .Sales
    .AsNotTracking()
    .Where(x => x.SalesPersonId == salesPersonId);

int count = await query.CountAsync(ct);

query = query
    .Skip(page * pageSize)
    .Take(pageSize);

var result = await query.ToListAsync(ct);    
return (count, result);
```

---
layout: default
hideInToc: true
---

# Deadly Sin: 6

## Not using Cancellation Tokens

- Will stop the Query on the SQL Server also

```ts {all|6,12}
var query = context
    .Sales
    .AsNotTracking()
    .Where(x => x.SalesPersonId == salesPersonId);

int count = await query.CountAsync(ct);

query = query
    .Skip(page * pageSize)
    .Take(pageSize);

var result = await query.ToListAsync(ct);    
return (count, result);
```

<!--

	When query is running on SQL Server
	it will run until completion
	Complex queries can stuff up SQL Server
	Cancelation Tokens on EF Core Queries

-->

---
layout: two-cols-header
hideInToc: true
---

# Deadly Sin: 7

## Inefficient Updates and Deletes

`foreach` will generate `n` SQL statements

::left::

# `foreach`

```ts
foreach (var blog in context.Blogs.Where(b => b.Rating < 3))
{
    context.Blogs.Remove(blog);
}

context.SaveChanges();
```

::right::

# `.ExecuteDelete()`
```ts
context.Blogs
  .Where(b => b.Rating < 3)
  .ExecuteDelete();

// SQL
DELETE FROM [b]
FROM [Blogs] AS [b]
WHERE [b].[Rating] < 3
```

---
layout: default
---

# TIPS

- `.TagWith("Fetching all users)`
- `.Select()` does not need .AsNoTracking()
- Use `-Async` version of the operator e.g (`ToListAsync()`)
- `Select` to only fetch the right columns
- An Index will not be used if there are operations on a Column

```ts
options.EnableDetailedErrors()
options.EnableSensetiveDataLoggging()
options.ConfigureWarnings(wa => 
  was.Log( new EventId[] { 
    CoreEventId.FirstWithOutOrderByAndFilterWarning
    CoreEventId.RowLimitingOperationsWithoutOrdferByWarning
  })
)
```

---
layout: default
hideInToc: true
---

# TIPS Cont.

If you use alot of Include or Join, a view might be a good idea.

```ts
modelBuilder.Entity<PostWithPostType>(e ⇒ entity
      .ToView(“PostWithType”));

```

Use Data Annotation to limit e.g. size of your typs
e.g. `string`

```ts

public string Firstname { get; set; } 

// SQL produced
Firstname nvarchar(max) -- (2GB)

[StringLength(64)] // nvarchar(64)
public string Name { get; set; } 

[MinLength(5), MaxLength(12)] 
public string Password { get; set; } 
```

---
layout: default
---
# Practice / Challenges


https://learn.microsoft.com/en-us/ef/core/get-started/overview/first-app?tabs=netcore-cli

https://learn.microsoft.com/en-us/training/modules/persist-data-ef-core/

https://learn.microsoft.com/en-us/training/modules/build-web-api-minimal-database/



https://github.com/dotnet/EntityFramework.Docs/tree/main/samples/core/Querying/RelatedData


# Expert Challenge 
https://github.com/StefanTheCode/OptimizeMePlease

--- 
layout: default
---

# Source

EF Core Doc: https://learn.microsoft.com/en-us/ef/core/

Book: https://www.manning.com/books/entity-framework-core-in-action-second-edition

SSW EF Core: https://ssw.com.au/rules/rules-to-better-entity-framework/

SSW Linq: https://ssw.com.au/rules/rules-to-better-linq/

Single vs Split Queries: https://learn.microsoft.com/en-us/ef/core/querying/single-split-queries

Sealed Classes: https://code-maze.com/improve-performance-sealed-classes-dotnet/

N+1 Issue : https://levelup.gitconnected.com/the-hidden-performance-killer-in-ef-core-understanding-and-avoiding-the-n-1-query-issue-ce105c6a14e9

---
layout: default
---
# Videos
<Youtube id="dDANjr5MCew" />
<Youtube id="ZKVXl2640ps" />
<Youtube id="jSiGyPHqnpY" />
