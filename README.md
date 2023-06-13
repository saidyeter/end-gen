# end-gen
Endpoint Generator

A Restful API builder library idea using C# Source Generators  
Sometimes we create so much boiler-plate code. And C# Source Generators might be a solution for some cases  

## Concept example

The idea is: developer only needs to make an implementation for an existing entitiy, then this lib should create an api controller that includes CRUD operations for the existing entity.

Let's say we have a `book` entity

```csharp
public class Book
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Author { get; set; }
    public string PublishingHouse { get; set; }
}
```

Let's assume this library provides an abstract class like this

```csharp
public abstract class EndpointInstructions<TEntity, TEntityKey>
{
    public abstract string RoutePrefix { get; set; }
    public abstract IQueryable<TEntity> GetMultipleData();
    public abstract TEntity GetSingleData(TEntityKey id);
    public abstract TEntity CreateData(TEntity updatedData);
    public abstract TEntity UpdateData(TEntityKey id,TEntity updatedData);
    public abstract bool RemoveData();
}
```

Developer will only need implement the `EndpointInstructions` class for `book` entity like below

```csharp
public class BookInstructions : EndpointInstructions<Book, int>
{
    private readonly BookDBContext _dbContext;

    public BookInstructions(BookDBContext dbContext)
    {
        _dbContext = dbContext;
    }

    public override string RoutePrefix { get; set; } = "my-book";
    public override IQueryable<Book> GetMultipleData() => _dbContext.Books.AsQueryable();
    public override Book GetSingleData(int id) => _dbContext.Books.FirstOrDefault(b => b.Id == id);
    public override Book CreateData(Book updatedData) => throw new NotImplementedException();
    public override Book UpdateData(int id, Book updatedBook) => throw new NotImplementedException();
    public override bool RemoveData() => throw new NotImplementedException();
}
```

Then this library should create a controller like below

```csharp
[Route("api/my-book")]
[ApiController]
public class GeneratedBookInstructionsController : ControllerBase
{
    private readonly BookInstructions _bookInstructions;

    public GeneratedBookInstructionsController(BookInstructions bookInstructions)
    {
        _bookInstructions = bookInstructions;
    }
    // GET: api/my-book
    [HttpGet]
    public IActionResult Get()
    {
        var dataList = _bookInstructions.GetMultipleData();

        if (!dataList.Any())
        {
            return NotFound();
        }
        return Ok(dataList);
    }

    // GET: api/my-book/5
    [HttpGet("{id}", Name = "Get")]
    public IActionResult Get(int id)
    {
        var singleData = _bookInstructions.GetSingleData(id);

        if (singleData is null)
        {
            return NotFound();
        }
        return Ok(singleData);
    }

    // ... all other endpoints
}
```

## Why
Actually creating controller is not that hard. And there is not that much boiler-plate code, up there.  
But this will allow some opportunitis for:  
- pagination and filtering by query params like `take`, `skip`, `select` and so on.  
- mapping results  
- pre and post process for endpoints  
Developer always will have control on data, and this will make tasks quicker to produce.  
This idea came from OData and GraphQL. This concepts allows us to manipulate response data according to way how you make a request.  