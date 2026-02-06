## ADDED Requirements

### Requirement: Delete product via HTTP DELETE

The system SHALL provide an HTTP DELETE endpoint at `/api/products/{id}` that removes the corresponding product record from the database.

#### Scenario: Successfully delete an existing product
- **WHEN** a DELETE request is sent to `/api/products/1` where product Id=1 exists
- **THEN** the system SHALL remove the product with Id=1 from the database
- **THEN** the system SHALL return HTTP 204 No Content with no response body

#### Scenario: Attempt to delete non-existent product
- **WHEN** a DELETE request is sent to `/api/products/999` where product Id=999 does not exist
- **THEN** the system SHALL NOT perform any database operations
- **THEN** the system SHALL return HTTP 404 Not Found

#### Scenario: Product is permanently removed from database
- **WHEN** a DELETE request successfully removes a product
- **THEN** subsequent GET requests to `/api/products/{id}` for that product SHALL return 404
- **THEN** the product SHALL NOT appear in GET `/api/products` list results
- **THEN** the deletion SHALL be permanent (hard delete, not soft delete)

### Requirement: Delete product through service layer

The system SHALL route delete requests through the ProductService which orchestrates repository operations.

#### Scenario: Service layer handles delete logic
- **WHEN** the controller receives a delete request for Id=5
- **THEN** the controller SHALL call IProductService.DeleteProductAsync(5)
- **THEN** the service SHALL call repository.DeleteAsync(5)
- **THEN** the service SHALL return a boolean indicating success or failure

#### Scenario: Service returns false for non-existent product
- **WHEN** DeleteProductAsync is called with an Id that does not exist
- **THEN** the service SHALL receive false from the repository
- **THEN** the service SHALL return false to the controller
- **THEN** the controller SHALL return 404 Not Found

#### Scenario: Service returns true for successful deletion
- **WHEN** DeleteProductAsync is called with an Id that exists
- **THEN** the repository SHALL successfully remove the product
- **THEN** the service SHALL receive true from the repository
- **THEN** the service SHALL return true to the controller
- **THEN** the controller SHALL return 204 No Content

### Requirement: Delete product via repository layer

The system SHALL implement DeleteAsync method in the ProductRepository that removes a product entity using Entity Framework Core.

#### Scenario: Repository deletes existing product
- **WHEN** DeleteAsync(id) is called with an existing product Id
- **THEN** the repository SHALL use FindAsync to retrieve the product by Id
- **THEN** the repository SHALL call Remove on the DbContext
- **THEN** the repository SHALL call SaveChangesAsync to persist the deletion
- **THEN** the repository SHALL return true

#### Scenario: Repository returns false when product not found
- **WHEN** DeleteAsync is called with an Id that does not exist in the database
- **THEN** FindAsync SHALL return null
- **THEN** DeleteAsync SHALL return false without attempting to save
- **THEN** no database changes SHALL occur

### Requirement: Return appropriate HTTP status codes for delete operations

The system SHALL return RESTful HTTP status codes that accurately reflect the outcome of delete operations.

#### Scenario: Successful deletion returns 204 No Content
- **WHEN** a product is successfully deleted
- **THEN** the controller SHALL return HTTP 204 No Content
- **THEN** the response body SHALL be empty

#### Scenario: Delete of non-existent product returns 404
- **WHEN** attempting to delete a product that does not exist
- **THEN** the controller SHALL return HTTP 404 Not Found
- **THEN** the response MAY include an error message body

### Requirement: Delete operation is idempotent

The system SHALL handle multiple delete requests for the same product Id safely.

#### Scenario: Second delete request returns 404
- **WHEN** a product with Id=10 is deleted successfully
- **THEN** a subsequent DELETE request to `/api/products/10` SHALL return 404 Not Found
- **THEN** no errors SHALL be thrown
- **THEN** the system remains in a consistent state
