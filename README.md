# TodoAPI - Minimal Web API with .NET 9

This project demonstrates how to create a minimal Web API using ASP.NET Core and an in-memory database for managing to-do items. It supports basic CRUD operations and is based on the official Microsoft tutorial.

## Key Features

- **CRUD Operations**: Supports Create, Read, Update, and Delete for to-do items.
- **In-Memory Database**: Uses `Entity Framework Core` with an in-memory database for temporary data storage.
- **Dependency Injection**: Integrates database context and services into the DI container.
- **Minimal API**: Implements concise and efficient HTTP endpoints.
- **Endpoint Testing**: Demonstrates testing methods with generated requests.

## Requirements

- .NET 9.0 or later
- Visual Studio 2022

## Project Setup

### Create the API Project

1. Open Visual Studio 2022 and select **Create a new project**.
2. Search for the **ASP.NET Core Empty** template, then click **Next**.
3. Name the project `TodoApi` and click **Next**.
4. In the **Additional information** dialog:
   - Select **.NET 9.0**.
   - Uncheck **Do not use top-level statements**.
   - Click **Create**.

### Add NuGet Packages

1. Go to **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**.
2. Select the **Browse** tab and check **Include Prerelease**.
3. Install:
   - `Microsoft.EntityFrameworkCore.InMemory`
   - `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore`

### Implement the API

#### Create the Model (`Todo.cs`)

```csharp
public class Todo
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

#### Define the Database Context (`TodoDb.cs`)

```csharp
using Microsoft.EntityFrameworkCore;

class TodoDb : DbContext
{
    public TodoDb(DbContextOptions<TodoDb> options) : base(options) { }
    public DbSet<Todo> Todos => Set<Todo>();
}
```

#### Add the API Code (`Program.cs`)

```csharp
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<TodoDb>(opt => opt.UseInMemoryDatabase("TodoList"));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();

var app = builder.Build();

app.MapGet("/todoitems", async (TodoDb db) => await db.Todos.ToListAsync());
app.MapGet("/todoitems/complete", async (TodoDb db) => await db.Todos.Where(t => t.IsComplete).ToListAsync());
app.MapGet("/todoitems/{id}", async (int id, TodoDb db) => await db.Todos.FindAsync(id) is Todo todo ? Results.Ok(todo) : Results.NotFound());
app.MapPost("/todoitems", async (Todo todo, TodoDb db) => {
    db.Todos.Add(todo);
    await db.SaveChangesAsync();
    return Results.Created($"/todoitems/{todo.Id}", todo);
});
app.MapPut("/todoitems/{id}", async (int id, Todo inputTodo, TodoDb db) => {
    var todo = await db.Todos.FindAsync(id);
    if (todo is null) return Results.NotFound();
    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;
    await db.SaveChangesAsync();
    return Results.NoContent();
});
app.MapDelete("/todoitems/{id}", async (int id, TodoDb db) => {
    if (await db.Todos.FindAsync(id) is Todo todo)
    {
        db.Todos.Remove(todo);
        await db.SaveChangesAsync();
        return Results.NoContent();
    }
    return Results.NotFound();
});

app.Run();
```

### Run the Application

1. Press `Ctrl+F5` to launch the app.
2. Test endpoints using Swagger or Postman.

## API Endpoints

| HTTP Method | Endpoint             | Description                |
|-------------|----------------------|----------------------------|
| GET         | `/todoitems`         | Retrieve all to-do items   |
| GET         | `/todoitems/complete`| Retrieve completed items   |
| GET         | `/todoitems/{id}`    | Retrieve an item by ID     |
| POST        | `/todoitems`         | Add a new to-do item       |
| PUT         | `/todoitems/{id}`    | Update an existing item    |
| DELETE      | `/todoitems/{id}`    | Delete a to-do item        |

## Example Request

### POST `/todoitems`

**Request Body:**

```json
{
  "name": "Walk the dog",
  "isComplete": true
}
```

**Response:**

```json
{
  "id": 1,
  "name": "Walk the dog",
  "isComplete": true
}
```

## Achievements

- Developed a minimal API with CRUD operations.
- Integrated in-memory database with Entity Framework Core.
- Simplified API development with concise syntax.

## Next Steps

- Add authentication and authorization.
- Transition to a persistent database (e.g., SQL Server).
- Implement unit and integration tests.

## Contributors

- Trung Pham ([GitHub](https://github.com/PhamTrung1204))
