## Why

The ProductsApi currently supports creating and reading product records, but lacks the ability to modify existing products or remove them from the database. This limits the API's usefulness for real-world scenarios where products need price updates, description corrections, or removal when discontinued. Completing the CRUD operations will make the API fully functional for product management.

## What Changes

- Add **PUT** endpoint (`/api/products/{id}`) to update existing product records
- Add **DELETE** endpoint (`/api/products/{id}`) to remove product records
- Implement `UpdateProductAsync` method in `ProductService` (Application layer)
- Implement `DeleteProductAsync` method in `ProductService` (Application layer)
- Create `UpdateProductDto` for update request validation
- Add update and delete methods to `IProductService` interface
- Add corresponding repository methods in Infrastructure layer if needed

## Capabilities

### New Capabilities
- `product-update`: Enable modification of existing product records via HTTP PUT, including validation, persistence, and appropriate error handling for non-existent products
- `product-delete`: Enable removal of product records via HTTP DELETE, with proper 404 handling for non-existent products and 204 No Content response on success

### Modified Capabilities
<!-- No existing capabilities are being modified - this extends existing CRUD operations -->

## Impact

**Affected Components:**
- `ProductsApi.API/Controllers/ProductsController.cs` - Add Update and Delete action methods
- `ProductsApi.Application/Interfaces/IProductService.cs` - Add method signatures
- `ProductsApi.Application/Services/ProductService.cs` - Implement business logic
- `ProductsApi.Application/DTOs/` - Add `UpdateProductDto` class
- `ProductsApi.Infrastructure/Repositories/` - May need repository extensions depending on current implementation

**API Changes:**
- New endpoints added (non-breaking changes)
- RESTful API surface area expanded to complete CRUD operations

**Dependencies:**
- No new external dependencies required
- Uses existing Entity Framework Core infrastructure
