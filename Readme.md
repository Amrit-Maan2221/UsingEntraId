# GO TO https://entra.microsoft.com/
# Have Application Developer Role atleast
# Go to App Registrations
# Register Single Tenant Application ( Intially ) => DemoApi
# Go to Expose an Api and add scope => api.all

To secure your backend API (DemoApi) using Microsoft Entra ID (formerly Azure AD), we need to configure Authentication (verifying who the user is) and Authorization (verifying what they are allowed to do).

Since you have already registered the app and created the api.all scope, here are the next steps to secure the backend:

1. Define App Roles (RBAC)
While Scopes (Delegated Permissions) are great for user-facing apps, App Roles allow you to assign specific permissions to users or other services (Managed Identities).

In the DemoApi registration, go to App roles.

Click Create app role.

Display name: Admin.Access

Allowed member types: Both (Users/Groups + Applications).

Value: API.Admin

Description: Allows full admin access to the API.

Click Apply.

# Create a Demo API

```
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.EntityFrameworkCore;
using Microsoft.Identity.Web;
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddOpenApi();

// Authentication: The AddMicrosoftIdentityWebApi middleware automatically validates the JWT signature, issuer, and expiration.
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));
builder.Services.AddDbContext<ApiDbContext>(opt => opt.UseInMemoryDatabase("DemoInventory"));

builder.Services.AddAuthorization();

var app = builder.Build();
app.MapOpenApi();
app.MapScalarApiReference("/docs");

app.UseHttpsRedirection();

app.UseAuthentication();
app.UseAuthorization();

// Authorization: The .RequireAuthorization() extension ensures no one can hit the /api/products endpoint without a valid token from your specific Entra tenant.
app.MapGet("/api/products", async (ApiDbContext db) => await db.Products.ToListAsync());

app.MapPost("/api/products", async (ApiDbContext db, Product product) =>
{
    db.Products.Add(product);
    await db.SaveChangesAsync();

    return Results.Created($"/api/products/{product.Id}", product);
})
.RequireAuthorization(); // Protected with Entra ID


app.Run();

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}

public class ApiDbContext : DbContext
{
    public ApiDbContext(DbContextOptions<ApiDbContext> options) : base(options) { }
    public DbSet<Product> Products => Set<Product>();
}
```

```
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "D",
    "TenantId": "D8",
    "ClientId": "D",
    "Scopes": "api.all"
  },
  "AllowedHosts": "*"
}
```
