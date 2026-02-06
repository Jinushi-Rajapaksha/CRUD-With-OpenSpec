## 1. Create DTOs

- [x] 1.1 Add `UpdateProductDto` class to `ProductsApi.Application/DTOs/ProductDto.cs` with properties: Name, Description, Price, Stock
- [x] 1.2 Verify `UpdateProductDto` does not include Id property (Id comes from route parameter)

## 2. Update Domain Layer Interfaces

- [x] 2.1 Add `Task<Product?> UpdateAsync(int id, Product product)` method signature to `IProductRepository` interface in `ProductsApi.Domain/Interfaces/IProductRepository.cs`
- [x] 2.2 Add `Task<bool> DeleteAsync(int id)` method signature to `IProductRepository` interface

## 3. Implement Repository Methods

- [x] 3.1 Implement `UpdateAsync` method in `ProductRepository` class in `ProductsApi.Infrastructure/Repositories/ProductRepository.cs`
- [x] 3.2 In `UpdateAsync`, use `FindAsync` to retrieve existing product by id
- [x] 3.3 In `UpdateAsync`, return null if product not found
- [x] 3.4 In `UpdateAsync`, update only mutable properties (Name, Description, Price, Stock) keeping CreatedDate unchanged
- [x] 3.5 In `UpdateAsync`, call `SaveChangesAsync` and return the updated product
- [x] 3.6 Implement `DeleteAsync` method in `ProductRepository` class
- [x] 3.7 In `DeleteAsync`, use `FindAsync` to retrieve product by id
- [x] 3.8 In `DeleteAsync`, return false if product not found
- [x] 3.9 In `DeleteAsync`, call `Remove` on DbContext and `SaveChangesAsync`, then return true

## 4. Update Application Layer Interfaces

- [x] 4.1 Add `Task<ProductDto?> UpdateProductAsync(int id, UpdateProductDto productDto)` method signature to `IProductService` interface in `ProductsApi.Application/Interfaces/IProductService.cs`
- [x] 4.2 Add `Task<bool> DeleteProductAsync(int id)` method signature to `IProductService` interface

## 5. Implement Service Methods

- [x] 5.1 Implement `UpdateProductAsync` method in `ProductService` class in `ProductsApi.Application/Services/ProductService.cs`
- [x] 5.2 In `UpdateProductAsync`, map `UpdateProductDto` to `Product` entity (set Name, Description, Price, Stock)
- [x] 5.3 In `UpdateProductAsync`, call `_repository.UpdateAsync(id, product)`
- [x] 5.4 In `UpdateProductAsync`, return null if repository returns null
- [x] 5.5 In `UpdateProductAsync`, map returned Product entity to ProductDto and return it
- [x] 5.6 Implement `DeleteProductAsync` method in `ProductService` class
- [x] 5.7 In `DeleteProductAsync`, call `_repository.DeleteAsync(id)` and return the boolean result

## 6. Add Controller Endpoints

- [x] 6.1 Add `HttpPut` action method named `Update` to `ProductsController` in `ProductsApi.API/Controllers/ProductsController.cs`
- [x] 6.2 Configure PUT endpoint route as `/api/products/{id}` using route parameter
- [x] 6.3 In `Update` method, accept `int id` from route and `UpdateProductDto` from body
- [x] 6.4 In `Update` method, call `_productService.UpdateProductAsync(id, productDto)`
- [x] 6.5 In `Update` method, return `NotFound()` (404) if service returns null
- [x] 6.6 In `Update` method, return `Ok(productDto)` (200) with updated ProductDto if successful
- [x] 6.7 Add `HttpDelete` action method named `Delete` to `ProductsController`
- [x] 6.8 Configure DELETE endpoint route as `/api/products/{id}` using route parameter
- [x] 6.9 In `Delete` method, accept `int id` from route
- [x] 6.10 In `Delete` method, call `_productService.DeleteProductAsync(id)`
- [x] 6.11 In `Delete` method, return `NotFound()` (404) if service returns false
- [x] 6.12 In `Delete` method, return `NoContent()` (204) if service returns true

## 7. Build and Verify

- [x] 7.1 Build the solution and resolve any compilation errors
- [x] 7.2 Verify no existing tests are broken by the changes
- [x] 7.3 Run the application and verify it starts successfully

## 8. Manual Testing

- [x] 8.1 Test PUT `/api/products/{id}` with valid product id and body - expect 200 OK with updated product
- [x] 8.2 Test PUT `/api/products/{id}` with non-existent product id - expect 404 Not Found
- [x] 8.3 Test PUT endpoint updates all mutable fields (Name, Description, Price, Stock)
- [x] 8.4 Test PUT endpoint preserves CreatedDate timestamp
- [x] 8.5 Test DELETE `/api/products/{id}` with existing product id - expect 204 No Content
- [x] 8.6 Test DELETE `/api/products/{id}` with non-existent product id - expect 404 Not Found
- [x] 8.7 Test DELETE permanently removes product (GET after DELETE returns 404)
- [x] 8.8 Test DELETE is idempotent (second delete of same id returns 404, no errors)
- [x] 8.9 Verify existing GET and POST endpoints still work correctly
