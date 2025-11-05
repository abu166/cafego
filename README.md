# Hot Coffee - Coffee Shop Management System

A RESTful API-based coffee shop management system built in Go that handles orders, menu items, and inventory management using a clean, layered architecture.

## Overview

Hot Coffee simulates the backend of a coffee shop's ordering system. The application manages customer orders, product catalogs, and ingredient inventory through a RESTful API interface. When orders are processed, the system automatically validates ingredient availability and updates stock levels accordingly.

## Architecture

The application implements a three-layered architecture pattern that separates concerns into distinct layers:

**Presentation Layer**: Handles HTTP requests and responses, parses input data, validates requests, and formats output. This layer contains handlers for orders, menu items, and inventory operations.

**Business Logic Layer**: Contains core functionality and business rules. Services in this layer process data, enforce business constraints, and coordinate between handlers and repositories. This layer handles operations like order validation, inventory checking, and sales calculations.

**Data Access Layer**: Manages data storage and retrieval using JSON files. Repositories provide interfaces for CRUD operations on orders, menu items, and inventory data while ensuring data consistency.

### Project Structure

```
hot-coffee/
├── cmd/
│   └── main.go
├── internal/
│   ├── handler/
│   │   ├── order_handler.go
│   │   ├── menu_handler.go
│   │   └── inventory_handler.go
│   ├── service/
│   │   ├── order_service.go
│   │   ├── menu_service.go
│   │   └── inventory_service.go
│   └── dal/
│       ├── order_repository.go
│       ├── menu_repository.go
│       └── inventory_repository.go
├── models/
│   ├── order.go
│   ├── menu_item.go
│   └── inventory_item.go
├── data/
│   ├── orders.json
│   ├── menu_items.json
│   └── inventory.json
└── go.mod
```

## Getting Started

### Prerequisites

- Go 1.21 or higher
- No external dependencies required

### Installation

Clone the repository and build the application:

```bash
go build -o hot-coffee .
```

### Running the Application

Start the server with default settings:

```bash
./hot-coffee
```

Start with custom configuration:

```bash
./hot-coffee --port 8080 --dir ./data
```

Display help information:

```bash
./hot-coffee --help
```

**Command Line Options:**

- `--port N`: Specify the port number for the HTTP server
- `--dir S`: Specify the path to the data directory containing JSON files
- `--help`: Display usage information and available options

## API Endpoints

### Order Management

**Create Order**
```
POST /orders
Content-Type: application/json

{
  "customer_name": "John Doe",
  "items": [
    {"product_id": "latte", "quantity": 2},
    {"product_id": "croissant", "quantity": 1}
  ]
}
```

**Get All Orders**
```
GET /orders
```

**Get Order by ID**
```
GET /orders/{id}
```

**Update Order**
```
PUT /orders/{id}
Content-Type: application/json

{
  "customer_name": "Jane Smith",
  "items": [
    {"product_id": "espresso", "quantity": 1}
  ]
}
```

**Delete Order**
```
DELETE /orders/{id}
```

**Close Order**
```
POST /orders/{id}/close
```

### Menu Management

**Add Menu Item**
```
POST /menu
Content-Type: application/json

{
  "product_id": "cappuccino",
  "name": "Cappuccino",
  "description": "Espresso with steamed milk foam",
  "price": 4.00,
  "ingredients": [
    {"ingredient_id": "espresso_shot", "quantity": 2},
    {"ingredient_id": "milk", "quantity": 150}
  ]
}
```

**Get All Menu Items**
```
GET /menu
```

**Get Menu Item by ID**
```
GET /menu/{id}
```

**Update Menu Item**
```
PUT /menu/{id}
```

**Delete Menu Item**
```
DELETE /menu/{id}
```

### Inventory Management

**Add Inventory Item**
```
POST /inventory
Content-Type: application/json

{
  "ingredient_id": "milk",
  "name": "Milk",
  "quantity": 5000,
  "unit": "ml"
}
```

**Get All Inventory Items**
```
GET /inventory
```

**Get Inventory Item by ID**
```
GET /inventory/{id}
```

**Update Inventory Item**
```
PUT /inventory/{id}
```

**Delete Inventory Item**
```
DELETE /inventory/{id}
```

### Reporting

**Get Total Sales**
```
GET /reports/total-sales

Response:
{
  "total_sales": 1247.50
}
```

**Get Popular Items**
```
GET /reports/popular-items

Response:
[
  {
    "product_id": "latte",
    "name": "Caffe Latte",
    "order_count": 45
  },
  {
    "product_id": "espresso",
    "name": "Espresso",
    "order_count": 38
  }
]
```

## Data Models

### Order

```go
type Order struct {
    ID           string      `json:"order_id"`
    CustomerName string      `json:"customer_name"`
    Items        []OrderItem `json:"items"`
    Status       string      `json:"status"`
    CreatedAt    string      `json:"created_at"`
}

type OrderItem struct {
    ProductID string `json:"product_id"`
    Quantity  int    `json:"quantity"`
}
```

### Menu Item

```go
type MenuItem struct {
    ID          string                `json:"product_id"`
    Name        string                `json:"name"`
    Description string                `json:"description"`
    Price       float64               `json:"price"`
    Ingredients []MenuItemIngredient  `json:"ingredients"`
}

type MenuItemIngredient struct {
    IngredientID string  `json:"ingredient_id"`
    Quantity     float64 `json:"quantity"`
}
```

### Inventory Item

```go
type InventoryItem struct {
    IngredientID string  `json:"ingredient_id"`
    Name         string  `json:"name"`
    Quantity     float64 `json:"quantity"`
    Unit         string  `json:"unit"`
}
```

## Data Storage

The application uses JSON files for data persistence. Each entity type is stored in a separate file within the data directory:

**orders.json**: Contains all customer orders with their items, status, and timestamps

**menu_items.json**: Stores product catalog including names, descriptions, prices, and required ingredients for each menu item

**inventory.json**: Tracks ingredient stock levels with quantities and measurement units

Example inventory.json structure:

```json
[
  {
    "ingredient_id": "espresso_shot",
    "name": "Espresso Shot",
    "quantity": 500,
    "unit": "shots"
  },
  {
    "ingredient_id": "milk",
    "name": "Milk",
    "quantity": 5000,
    "unit": "ml"
  },
  {
    "ingredient_id": "flour",
    "name": "Flour",
    "quantity": 10000,
    "unit": "g"
  }
]
```

## Key Features

### Inventory Validation and Management

When an order is created, the system performs the following operations:

1. Validates that all product IDs in the order exist in the menu
2. Calculates total ingredient requirements based on order quantities
3. Checks current inventory levels against requirements
4. Returns an error if any ingredient is insufficient
5. Deducts ingredient quantities from inventory when order is confirmed
6. Updates the inventory.json file with new quantities

If inventory is insufficient, the API returns a 400 Bad Request with details:

```json
{
  "error": "Insufficient inventory for ingredient 'Milk'. Required: 400ml, Available: 150ml."
}
```

### Logging

The application uses Go's log/slog package for structured logging. All significant events are logged with appropriate severity levels and contextual information:

```go
slog.Info("Order created", "orderID", newOrder.ID, "customer", newOrder.CustomerName)
slog.Error("Failed to update inventory", "error", err, "ingredientID", ingredientID)
slog.Warn("Low inventory detected", "ingredientID", item.IngredientID, "quantity", item.Quantity)
```

Logs include timestamps, operation context, entity identifiers, and error details for debugging and monitoring purposes.

### Error Handling

The application implements comprehensive error handling with appropriate HTTP status codes:

- **200 OK**: Successful GET requests
- **201 Created**: Successful resource creation
- **400 Bad Request**: Invalid input or insufficient inventory
- **404 Not Found**: Requested resource does not exist
- **500 Internal Server Error**: Unexpected server errors

All error responses include descriptive messages explaining the issue.

## Development Guidelines

### Code Quality Requirements

- Code must be formatted with gofumpt
- Application must compile without errors
- No panics or unexpected crashes allowed
- Only built-in Go packages permitted

### Implementation Approach

Start by implementing a single entity across all three layers to understand the architecture. For example, implement inventory management first:

1. Create the inventory model in the models package
2. Implement the repository interface and JSON storage logic
3. Create the service layer with business logic
4. Build the HTTP handlers for inventory endpoints
5. Test the complete flow before moving to other entities

### Architectural Principles

Use interfaces to define contracts between layers. This allows for flexibility and testability:

```go
type OrderService interface {
    CreateOrder(order Order) (Order, error)
    GetOrder(id string) (Order, error)
    UpdateOrder(id string, order Order) error
    DeleteOrder(id string) error
}
```

Implement dependency injection to pass repositories to services and services to handlers. This promotes loose coupling and makes unit testing easier.

### Testing Strategy

Before accepting orders, ensure the system has:

- Menu items configured with ingredient requirements
- Sufficient inventory quantities for all ingredients
- Proper validation logic in place

Test edge cases such as:

- Orders with non-existent product IDs
- Orders requiring more ingredients than available
- Concurrent order processing
- Invalid JSON input
- Missing required fields

## Technical Considerations

### Concurrency

The application should handle concurrent HTTP requests safely. Implement proper locking mechanisms when reading and writing JSON files to prevent race conditions and data corruption.

### Data Integrity

Ensure atomic operations when updating inventory. If an order cannot be fulfilled due to insufficient inventory, no inventory changes should be persisted.

### Validation

Implement comprehensive input validation:

- Check for required fields in request bodies
- Validate data types and formats
- Ensure positive quantities and prices
- Verify product and ingredient IDs exist

### Performance

While JSON file storage is suitable for this educational project, consider the following for production systems:

- File locking overhead on concurrent access
- Full file read/write operations for each transaction
- Memory usage when loading large datasets
- Lack of query optimization and indexing

## Learning Outcomes

This project demonstrates practical application of:

- RESTful API design principles and HTTP semantics
- JSON serialization and deserialization in Go
- Layered architecture and separation of concerns
- Interface-based design and dependency injection
- File I/O operations and data persistence
- Structured logging with contextual information
- Comprehensive error handling patterns
- Business logic implementation and validation
- SOLID principles in software design

## Troubleshooting

**Server fails to start:**
- Verify the specified port is not already in use
- Check that the data directory exists and is writable
- Ensure command line arguments are correctly formatted

**Order creation fails:**
- Confirm menu items exist with correct product IDs
- Verify inventory has sufficient quantities
- Check that ingredient IDs in menu items match inventory entries

**Data not persisting:**
- Ensure the application has write permissions to the data directory
- Verify JSON file syntax is valid
- Check logs for file I/O errors

---

This project provides hands-on experience with building production-grade backend systems while maintaining simplicity through file-based storage and standard library usage.