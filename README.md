In ASP.NET Core Web API, loading related data in an efficient way is crucial for performance. Eager loading and lazy loading are two strategies used to retrieve related data when working with Entity Framework Core.

### Eager Loading:

Eager loading retrieves all the related data in a single query, reducing the number of database trips. It can be useful when you know upfront that you will need the related data.

Let's consider an example with two entities: `Author` and `Book`. Each author can have multiple books.

```csharp
public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }

    public List<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public int AuthorId { get; set; }

    public Author Author { get; set; }
}
```

Now, let's use eager loading in a controller action:

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthorController : ControllerBase
{
    private readonly ApplicationDbContext _context;

    public AuthorController(ApplicationDbContext context)
    {
        _context = context;
    }

    [HttpGet("{id}")]
    public IActionResult GetAuthorWithBooks(int id)
    {
        var author = _context.Authors
            .Include(a => a.Books) // Eager loading here
            .FirstOrDefault(a => a.AuthorId == id);

        if (author == null)
        {
            return NotFound();
        }

        return Ok(author);
    }
}
```

In this example, the `Include` method is used to eagerly load the related `Books` when retrieving an `Author`.

### Lazy Loading:

Lazy loading defers the loading of related data until it's explicitly accessed. This can be useful when you want to minimize the amount of data initially loaded.

For lazy loading to work, you need to make properties virtual:

```csharp
public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }

    public virtual List<Book> Books { get; set; }
}

public class Book
{
    public int BookId { get; set; }
    public string Title { get; set; }
    public int AuthorId { get; set; }

    public virtual Author Author { get; set; }
}
```

Then, ensure that lazy loading is enabled in your DbContext:

```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseLazyLoadingProxies(); // Enable lazy loading
    }

    public DbSet<Author> Authors { get; set; }
    public DbSet<Book> Books { get; set; }
}
```

Now, when you retrieve an `Author`, related `Books` will be loaded when accessed:

```csharp
[HttpGet("{id}")]
public IActionResult GetAuthor(int id)
{
    var author = _context.Authors.Find(id);

    if (author == null)
    {
        return NotFound();
    }

    // Books are loaded when accessed due to lazy loading
    var books = author.Books;

    return Ok(author);
}
```

Make sure to configure your project to support lazy loading by installing the appropriate NuGet packages and enabling the necessary configuration.