## ADDED Requirements

### Requirement: Update product via HTTP PUT

The system SHALL provide an HTTP PUT endpoint at `/api/products/{id}` that accepts product data and updates the corresponding product record in the database.

#### Scenario: Successfully update an existing product
- **WHEN** a PUT request is sent to `/api/products/1` with valid JSON body containing Name, Description, Price, and Stock
- **THEN** the system SHALL update the product with Id=1 in the database
- **THEN** the system SHALL return HTTP 200 OK with the updated ProductDto in the response body

#### Scenario: Update product with all fields changed
- **WHEN** a PUT request updates all mutable fields (Name="Updated Name", Description="New Description", Price=99.99, Stock=50)
- **THEN** the system SHALL persist all changes to the database
- **THEN** the system SHALL preserve the original CreatedDate value
- **THEN** the system SHALL return the updated product with all new values

#### Scenario: Attempt to update non-existent product
- **WHEN** a PUT request is sent to `/api/products/999` where product Id=999 does not exist
- **THEN** the system SHALL NOT create a new product
- **THEN** the system SHALL return HTTP 404 Not Found

### Requirement: Accept update data via UpdateProductDto

The system SHALL accept product update data through an UpdateProductDto object containing Name, Description, Price, and Stock fields.

#### Scenario: Valid update DTO is processed
- **WHEN** an UpdateProductDto is provided with Name="Widget", Description="A useful widget", Price=29.99, Stock=100
- **THEN** the system SHALL map all DTO fields to the product entity
- **THEN** the system SHALL validate and persist the changes

#### Scenario: Update DTO does not contain Id field
- **WHEN** an UpdateProductDto is submitted in the request body
- **THEN** the system SHALL obtain the product Id from the route parameter, not from the DTO
- **THEN** the system SHALL prevent Id mismatch issues

### Requirement: Preserve immutable product properties on update

The system SHALL preserve the product's CreatedDate timestamp when updating other fields.

#### Scenario: CreatedDate remains unchanged after update
- **WHEN** a product with CreatedDate="2024-01-15T10:30:00Z" is updated
- **THEN** the system SHALL keep CreatedDate="2024-01-15T10:30:00Z" unchanged
- **THEN** the system SHALL only modify the fields provided in UpdateProductDto

### Requirement: Update product through service layer

The system SHALL route update requests through the ProductService which orchestrates repository operations and DTO mapping.

#### Scenario: Service layer handles update logic
- **WHEN** the controller receives an update request
- **THEN** the controller SHALL call IProductService.UpdateProductAsync(id, updateDto)
- **THEN** the service SHALL map UpdateProductDto to Product entity
- **THEN** the service SHALL call repository.UpdateAsync(id, product)
- **THEN** the service SHALL map the returned Product to ProductDto

#### Scenario: Service returns null for non-existent product
- **WHEN** UpdateProductAsync is called with an Id that does not exist
- **THEN** the service SHALL receive null from the repository
- **THEN** the service SHALL return null to the controller
- **THEN** the controller SHALL return 404 Not Found

### Requirement: Update product via repository layer

The system SHALL implement UpdateAsync method in the ProductRepository that updates a product entity using Entity Framework Core.

#### Scenario: Repository updates tracked entity
- **WHEN** UpdateAsync(id, product) is called
- **THEN** the repository SHALL use FindAsync to retrieve the existing product by Id
- **THEN** the repository SHALL update the tracked entity's properties
- **THEN** the repository SHALL call SaveChangesAsync to persist changes
- **THEN** the repository SHALL return the updated Product entity

#### Scenario: Repository returns null when product not found
- **WHEN** UpdateAsync is called with an Id that does not exist in the database
- **THEN** FindAsync SHALL return null
- **THEN** UpdateAsync SHALL return null without attempting to save
- **THEN** no database changes SHALL occur
