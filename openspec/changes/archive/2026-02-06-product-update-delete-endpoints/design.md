## Context

The ProductsApi follows a clean architecture pattern with clear separation of concerns:

**Current Architecture Layers:**
- **API Layer** (`ProductsApi.API`): Controllers expose REST endpoints
- **Application Layer** (`ProductsApi.Application`): Services contain business logic, DTOs for data transfer
- **Domain Layer** (`ProductsApi.Domain`): Entities and repository interfaces
- **Infrastructure Layer** (`ProductsApi.Infrastructure`): Repository implementations using EF Core

**Existing Patterns:**
- Controllers inject `IProductService` and return appropriate HTTP status codes
- Services inject `IProductRepository` and handle entity-to-DTO mapping
- Repository methods are async and use EF Core `DbContext`
- DTOs separate read models (`ProductDto`) from create models (`CreateProductDto`)
- Create operations set `CreatedDate` timestamp in the service layer

**Current State:**
- Only GET (all/by-id) and POST operations exist
- No Update or Delete capabilities
- Product entity includes: Id, Name, Description, Price, Stock, CreatedDate

## Goals / Non-Goals

**Goals:**
- Add HTTP PUT endpoint to update existing products
- Add HTTP DELETE endpoint to remove products
- Maintain consistency with existing architectural patterns
- Return proper HTTP status codes (200 OK for update, 204 No Content for delete, 404 for not found)
- Validate product existence before update/delete operations
- Follow existing async/await patterns throughout all layers

**Non-Goals:**
- Soft delete implementation (hard delete is sufficient)
- Audit trail or change history tracking
- Bulk update/delete operations
- Product archive functionality
- Authorization or role-based access control
- Validation of business rules (price limits, stock thresholds, etc.)

## Decisions

### Decision 1: DTO Design for Update

**Choice:** Create separate `UpdateProductDto` (without Id in body)

**Rationale:**
- Follows existing pattern where `CreateProductDto` excludes Id
- Id comes from route parameter (`/api/products/{id}`)
- Prevents mismatch between route Id and body Id
- Allows all product fields to be updatable (Name, Description, Price, Stock)

**Alternatives Considered:**
- Reuse `CreateProductDto` ? Rejected: semantically different operations
- Include Id in DTO ? Rejected: violates RESTful convention where Id is in route

**Implementation:**
```csharp
public class UpdateProductDto
{
    public string Name { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

### Decision 2: Update Timestamp Handling

**Choice:** Do NOT update or track `CreatedDate` on updates

**Rationale:**
- `CreatedDate` should remain immutable (represents initial creation)
- No `UpdatedDate` field exists in current Product entity
- Adding `UpdatedDate` would require migration and is out of scope

**Alternatives Considered:**
- Add `UpdatedDate` field ? Rejected: requires schema change (non-goal)
- Update `CreatedDate` ? Rejected: semantically incorrect

### Decision 3: Repository Method Signatures

**Choice:** Add `UpdateAsync` and `DeleteAsync` methods to repository interface

**Implementation:**
```csharp
// IProductRepository additions
Task<Product?> UpdateAsync(int id, Product product);
Task<bool> DeleteAsync(int id);
```

**Rationale:**
- `UpdateAsync` returns `Product?` to indicate success/failure (null if not found)
- `DeleteAsync` returns `bool` to indicate whether product existed
- Follows async pattern of existing methods
- Repository handles `SaveChangesAsync()` call

**Alternatives Considered:**
- Void return types ? Rejected: caller can't distinguish success/failure
- Throw exceptions on not found ? Rejected: exceptions should not control flow

### Decision 4: Service Layer Logic

**Choice:** Service checks if product exists before update/delete, maps entities to DTOs

**Update Flow:**
1. Service receives `UpdateProductDto` and id
2. Service calls `_repository.UpdateAsync(id, mappedProduct)`
3. Repository returns updated `Product?` or null
4. Service maps to `ProductDto` or returns null to controller

**Delete Flow:**
1. Service receives id
2. Service calls `_repository.DeleteAsync(id)`
3. Repository returns bool indicating success
4. Service returns bool to controller

**Rationale:**
- Maintains existing separation: service handles business logic, repository handles data access
- Consistent with existing `GetProductByIdAsync` pattern (returns null for not found)

### Decision 5: Controller HTTP Response Codes

**Update Endpoint:**
- 200 OK with `ProductDto` body on success
- 404 Not Found if product doesn't exist

**Delete Endpoint:**
- 204 No Content on successful deletion (no response body)
- 404 Not Found if product doesn't exist

**Rationale:**
- Follows RESTful conventions
- Consistent with existing GET endpoint (returns 404 for not found)
- 204 for DELETE is standard practice (no content to return)

### Decision 6: Repository Implementation with EF Core

**UpdateAsync Implementation:**
```csharp
public async Task<Product?> UpdateAsync(int id, Product product)
{
    var existing = await _context.Products.FindAsync(id);
    if (existing == null) return null;
    
    existing.Name = product.Name;
    existing.Description = product.Description;
    existing.Price = product.Price;
    existing.Stock = product.Stock;
    // Keep existing.CreatedDate unchanged
    
    await _context.SaveChangesAsync();
    return existing;
}
```

**DeleteAsync Implementation:**
```csharp
public async Task<bool> DeleteAsync(int id)
{
    var product = await _context.Products.FindAsync(id);
    if (product == null) return false;
    
    _context.Products.Remove(product);
    await _context.SaveChangesAsync();
    return true;
}
```

**Rationale:**
- Uses `FindAsync` for efficient primary key lookup
- Updates only mutable properties
- `SaveChangesAsync` persists changes
- Returns null/false instead of throwing exceptions for not found

## Risks / Trade-offs

### Risk 1: Concurrent Updates
**Risk:** Two requests update same product simultaneously, last write wins  
**Mitigation:** Acceptable for initial implementation. Can add EF Core concurrency tokens later if needed.

### Risk 2: Partial Updates
**Risk:** `UpdateProductDto` requires all fields; no support for PATCH-style partial updates  
**Mitigation:** Acceptable - full update is simpler and covers most use cases. PATCH can be added later if needed.

### Risk 3: Related Data Cleanup
**Risk:** Deleting a product doesn't check for related data (e.g., orders, reviews if they exist)  
**Mitigation:** Acceptable - no related entities exist in current schema. Future work should add cascade rules or soft delete.

### Risk 4: Input Validation
**Risk:** No validation on UpdateProductDto (negative prices, empty names, etc.)  
**Mitigation:** Acceptable for this change. Can add FluentValidation or DataAnnotations in future enhancement.

### Risk 5: EF Core Tracking Issues
**Risk:** If service creates new Product entity for update, EF tracking could cause issues  
**Mitigation:** Repository fetches existing entity from context, updates properties on tracked entity. This ensures proper change tracking.

## Migration Plan

**Deployment Steps:**
1. Add `UpdateProductDto` class to `ProductsApi.Application/DTOs/ProductDto.cs`
2. Update `IProductRepository` interface with new method signatures
3. Implement `UpdateAsync` and `DeleteAsync` in `ProductRepository`
4. Update `IProductService` interface with new method signatures
5. Implement `UpdateProductAsync` and `DeleteProductAsync` in `ProductService`
6. Add PUT and DELETE action methods to `ProductsController`
7. Build and test locally
8. Deploy to staging/production (no database migrations required)

**Rollback Strategy:**
- No database schema changes, so rollback is straightforward
- Simply redeploy previous version if issues arise
- Existing GET and POST endpoints remain unaffected

**Testing Approach:**
- Unit tests for service layer (mock repository)
- Integration tests for controller ? service ? repository flow
- Manual testing with Swagger/Postman:
  - Update existing product (expect 200 OK)
  - Update non-existent product (expect 404)
  - Delete existing product (expect 204)
  - Delete non-existent product (expect 404)

## Open Questions

~~None - design is complete and ready for implementation.~~

**Note:** All architectural decisions align with existing codebase patterns. No external dependencies or configuration changes required.
