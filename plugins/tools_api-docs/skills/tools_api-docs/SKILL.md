---
name: tools_api-docs
description: "Comprehensive API documentation generation toolkit using OpenAPI 3.0/Swagger specifications. When Claude needs to: (1) Generate API documentation from code, (2) Create OpenAPI/Swagger specifications, (3) Document REST API endpoints, (4) Generate API reference documentation, (5) Create interactive API docs, or (6) API文檔生成、OpenAPI規格、Swagger文檔、接口文檔、REST API文檔"
---

# API Documentation Generator

## Overview

This skill provides comprehensive guidance for creating professional API documentation using OpenAPI 3.0 (Swagger) specifications. It covers REST API documentation, interactive API explorers, code generation from specs, and automated documentation generation from source code.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Creating API documentation
- Generating OpenAPI/Swagger specifications
- Documenting REST API endpoints
- Creating API reference documentation
- Building interactive API documentation
- Auto-generating API docs from code
- API versioning documentation

**Trigger phrases include:**
- "Create API documentation" / "建立API文檔"
- "Generate OpenAPI spec" / "生成OpenAPI規格"
- "Document REST API" / "記錄REST API"
- "Swagger documentation" / "Swagger文檔"
- "API reference" / "API參考文檔"
- "Interactive API docs" / "互動式API文檔"
- "API specification" / "API規格"

## How to use this skill

### Workflow Overview

This skill follows a systematic 4-step workflow:

1. **Analyze API** - Review API endpoints, parameters, and responses
2. **Create Specification** - Write OpenAPI 3.0 specification
3. **Generate Documentation** - Use tools to create interactive docs
4. **Maintain** - Keep documentation in sync with code

## OpenAPI 3.0 Specification Structure

### Basic OpenAPI Document

```yaml
openapi: 3.0.3
info:
  title: E-Commerce API
  description: REST API for e-commerce platform
  version: 1.0.0
  contact:
    name: API Support
    email: api@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server
  - url: http://localhost:3000/v1
    description: Development server

tags:
  - name: Users
    description: User management operations
  - name: Products
    description: Product catalog operations
  - name: Orders
    description: Order management operations

paths:
  /users:
    # API endpoints
  /products:
    # API endpoints
  /orders:
    # API endpoints

components:
  schemas:
    # Data models
  securitySchemes:
    # Authentication schemes
  responses:
    # Reusable responses
  parameters:
    # Reusable parameters
```

## Complete OpenAPI Example

```yaml
openapi: 3.0.3

info:
  title: E-Commerce API
  description: |
    # E-Commerce Platform API

    This API provides access to the e-commerce platform functionality including:
    - User management and authentication
    - Product catalog browsing and search
    - Shopping cart operations
    - Order placement and tracking
    - Payment processing

    ## Authentication

    All API requests require authentication using Bearer tokens.
    Obtain a token by calling the `/auth/login` endpoint.

    ## Rate Limiting

    API requests are limited to 100 requests per minute per user.

    ## Errors

    The API uses standard HTTP status codes and returns error details in JSON format.
  version: 1.0.0
  termsOfService: https://example.com/terms
  contact:
    name: API Support Team
    email: api-support@example.com
    url: https://example.com/support
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

tags:
  - name: Authentication
    description: User authentication operations
  - name: Users
    description: User management
  - name: Products
    description: Product catalog operations
  - name: Cart
    description: Shopping cart operations
  - name: Orders
    description: Order management
  - name: Payments
    description: Payment processing

paths:
  /auth/login:
    post:
      tags:
        - Authentication
      summary: User login
      description: Authenticate user and receive access token
      operationId: loginUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - email
                - password
              properties:
                email:
                  type: string
                  format: email
                  example: user@example.com
                password:
                  type: string
                  format: password
                  minLength: 8
                  example: SecurePass123
      responses:
        '200':
          description: Login successful
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    type: object
                    properties:
                      token:
                        type: string
                        description: JWT access token
                        example: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
                      user:
                        $ref: '#/components/schemas/User'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '422':
          $ref: '#/components/responses/ValidationError'

  /auth/register:
    post:
      tags:
        - Authentication
      summary: User registration
      description: Register a new user account
      operationId: registerUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserRegistration'
      responses:
        '201':
          description: User registered successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    type: object
                    properties:
                      user:
                        $ref: '#/components/schemas/User'
                      message:
                        type: string
                        example: Verification email sent
        '422':
          $ref: '#/components/responses/ValidationError'

  /users/profile:
    get:
      tags:
        - Users
      summary: Get user profile
      description: Retrieve authenticated user's profile information
      operationId: getUserProfile
      security:
        - BearerAuth: []
      responses:
        '200':
          description: Profile retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/UserProfile'
        '401':
          $ref: '#/components/responses/Unauthorized'

    put:
      tags:
        - Users
      summary: Update user profile
      description: Update authenticated user's profile information
      operationId: updateUserProfile
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserProfileUpdate'
      responses:
        '200':
          description: Profile updated successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/UserProfile'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '422':
          $ref: '#/components/responses/ValidationError'

  /products:
    get:
      tags:
        - Products
      summary: List products
      description: |
        Retrieve a paginated list of products with optional filtering and sorting.

        **Filtering:**
        - By category: `?category=electronics`
        - By price range: `?minPrice=100&maxPrice=500`
        - By search term: `?search=laptop`

        **Sorting:**
        - By price: `?sortBy=price&order=asc`
        - By name: `?sortBy=name&order=desc`
        - By rating: `?sortBy=rating&order=desc`
      operationId: listProducts
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
        - name: category
          in: query
          description: Filter by category
          schema:
            type: string
            example: electronics
        - name: minPrice
          in: query
          description: Minimum price
          schema:
            type: number
            format: float
            minimum: 0
            example: 100
        - name: maxPrice
          in: query
          description: Maximum price
          schema:
            type: number
            format: float
            minimum: 0
            example: 1000
        - name: search
          in: query
          description: Search term
          schema:
            type: string
            example: laptop
        - name: sortBy
          in: query
          description: Sort field
          schema:
            type: string
            enum: [name, price, rating, createdAt]
            default: createdAt
        - name: order
          in: query
          description: Sort order
          schema:
            type: string
            enum: [asc, desc]
            default: desc
      responses:
        '200':
          description: Products retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    type: object
                    properties:
                      products:
                        type: array
                        items:
                          $ref: '#/components/schemas/Product'
                      pagination:
                        $ref: '#/components/schemas/Pagination'

  /products/{productId}:
    get:
      tags:
        - Products
      summary: Get product details
      description: Retrieve detailed information about a specific product
      operationId: getProduct
      parameters:
        - $ref: '#/components/parameters/ProductIdParam'
      responses:
        '200':
          description: Product details retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/ProductDetail'
        '404':
          $ref: '#/components/responses/NotFound'

  /cart:
    get:
      tags:
        - Cart
      summary: Get shopping cart
      description: Retrieve the current user's shopping cart
      operationId: getCart
      security:
        - BearerAuth: []
      responses:
        '200':
          description: Cart retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Cart'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /cart/items:
    post:
      tags:
        - Cart
      summary: Add item to cart
      description: Add a product to the shopping cart
      operationId: addCartItem
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - productId
                - quantity
              properties:
                productId:
                  type: string
                  format: uuid
                  example: 123e4567-e89b-12d3-a456-426614174000
                quantity:
                  type: integer
                  minimum: 1
                  example: 2
      responses:
        '201':
          description: Item added to cart
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Cart'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'
        '422':
          $ref: '#/components/responses/ValidationError'

  /cart/items/{itemId}:
    put:
      tags:
        - Cart
      summary: Update cart item
      description: Update the quantity of an item in the cart
      operationId: updateCartItem
      security:
        - BearerAuth: []
      parameters:
        - name: itemId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - quantity
              properties:
                quantity:
                  type: integer
                  minimum: 1
                  example: 3
      responses:
        '200':
          description: Cart item updated
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Cart'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      tags:
        - Cart
      summary: Remove cart item
      description: Remove an item from the shopping cart
      operationId: removeCartItem
      security:
        - BearerAuth: []
      parameters:
        - name: itemId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Item removed from cart
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Cart'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

  /orders:
    post:
      tags:
        - Orders
      summary: Create order
      description: Place a new order with items from the cart
      operationId: createOrder
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderCreate'
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Order'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '422':
          $ref: '#/components/responses/ValidationError'

    get:
      tags:
        - Orders
      summary: List orders
      description: Retrieve a list of the user's orders
      operationId: listOrders
      security:
        - BearerAuth: []
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
        - name: status
          in: query
          description: Filter by order status
          schema:
            type: string
            enum: [pending, confirmed, shipped, delivered, cancelled]
      responses:
        '200':
          description: Orders retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    type: object
                    properties:
                      orders:
                        type: array
                        items:
                          $ref: '#/components/schemas/Order'
                      pagination:
                        $ref: '#/components/schemas/Pagination'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /orders/{orderId}:
    get:
      tags:
        - Orders
      summary: Get order details
      description: Retrieve detailed information about a specific order
      operationId: getOrder
      security:
        - BearerAuth: []
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Order details retrieved successfully
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/OrderDetail'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        JWT token obtained from `/auth/login` endpoint.
        Include the token in the Authorization header: `Bearer <token>`

  parameters:
    PageParam:
      name: page
      in: query
      description: Page number (1-indexed)
      schema:
        type: integer
        minimum: 1
        default: 1
        example: 1

    LimitParam:
      name: limit
      in: query
      description: Number of items per page
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
        example: 20

    ProductIdParam:
      name: productId
      in: path
      required: true
      description: Product UUID
      schema:
        type: string
        format: uuid
        example: 123e4567-e89b-12d3-a456-426614174000

  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: 550e8400-e29b-41d4-a716-446655440000
        email:
          type: string
          format: email
          example: user@example.com
        firstName:
          type: string
          example: John
        lastName:
          type: string
          example: Doe
        createdAt:
          type: string
          format: date-time
          example: 2024-01-15T10:30:00Z

    UserRegistration:
      type: object
      required:
        - email
        - password
        - firstName
        - lastName
      properties:
        email:
          type: string
          format: email
          example: user@example.com
        password:
          type: string
          format: password
          minLength: 8
          example: SecurePass123
        firstName:
          type: string
          minLength: 1
          example: John
        lastName:
          type: string
          minLength: 1
          example: Doe

    UserProfile:
      allOf:
        - $ref: '#/components/schemas/User'
        - type: object
          properties:
            phone:
              type: string
              example: +1234567890
            address:
              $ref: '#/components/schemas/Address'

    UserProfileUpdate:
      type: object
      properties:
        firstName:
          type: string
          example: John
        lastName:
          type: string
          example: Doe
        phone:
          type: string
          example: +1234567890

    Product:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: 123e4567-e89b-12d3-a456-426614174000
        name:
          type: string
          example: Laptop
        description:
          type: string
          example: High-performance laptop
        price:
          type: number
          format: float
          example: 999.99
        category:
          type: string
          example: Electronics
        imageUrl:
          type: string
          format: uri
          example: https://cdn.example.com/laptop.jpg
        rating:
          type: number
          format: float
          minimum: 0
          maximum: 5
          example: 4.5
        stock:
          type: integer
          minimum: 0
          example: 50

    ProductDetail:
      allOf:
        - $ref: '#/components/schemas/Product'
        - type: object
          properties:
            specifications:
              type: object
              additionalProperties:
                type: string
              example:
                processor: Intel Core i7
                ram: 16GB
                storage: 512GB SSD
            reviews:
              type: array
              items:
                $ref: '#/components/schemas/Review'

    Review:
      type: object
      properties:
        id:
          type: string
          format: uuid
        userId:
          type: string
          format: uuid
        userName:
          type: string
          example: John Doe
        rating:
          type: integer
          minimum: 1
          maximum: 5
          example: 5
        comment:
          type: string
          example: Great product!
        createdAt:
          type: string
          format: date-time

    Cart:
      type: object
      properties:
        id:
          type: string
          format: uuid
        userId:
          type: string
          format: uuid
        items:
          type: array
          items:
            $ref: '#/components/schemas/CartItem'
        totalAmount:
          type: number
          format: float
          example: 1999.98
        updatedAt:
          type: string
          format: date-time

    CartItem:
      type: object
      properties:
        id:
          type: string
          format: uuid
        productId:
          type: string
          format: uuid
        product:
          $ref: '#/components/schemas/Product'
        quantity:
          type: integer
          minimum: 1
          example: 2
        subtotal:
          type: number
          format: float
          example: 1999.98

    Order:
      type: object
      properties:
        id:
          type: string
          format: uuid
        userId:
          type: string
          format: uuid
        status:
          type: string
          enum: [pending, confirmed, shipped, delivered, cancelled]
          example: confirmed
        totalAmount:
          type: number
          format: float
          example: 1999.98
        createdAt:
          type: string
          format: date-time

    OrderDetail:
      allOf:
        - $ref: '#/components/schemas/Order'
        - type: object
          properties:
            items:
              type: array
              items:
                $ref: '#/components/schemas/OrderItem'
            shippingAddress:
              $ref: '#/components/schemas/Address'
            billingAddress:
              $ref: '#/components/schemas/Address'
            paymentMethod:
              type: string
              example: credit_card
            trackingNumber:
              type: string
              example: 1Z999AA10123456784

    OrderItem:
      type: object
      properties:
        id:
          type: string
          format: uuid
        productId:
          type: string
          format: uuid
        productName:
          type: string
          example: Laptop
        quantity:
          type: integer
          example: 2
        unitPrice:
          type: number
          format: float
          example: 999.99
        subtotal:
          type: number
          format: float
          example: 1999.98

    OrderCreate:
      type: object
      required:
        - shippingAddress
        - paymentMethod
      properties:
        shippingAddress:
          $ref: '#/components/schemas/Address'
        billingAddress:
          $ref: '#/components/schemas/Address'
        paymentMethod:
          type: string
          enum: [credit_card, paypal, apple_pay]
          example: credit_card

    Address:
      type: object
      required:
        - street
        - city
        - state
        - zipCode
        - country
      properties:
        street:
          type: string
          example: 123 Main St
        city:
          type: string
          example: San Francisco
        state:
          type: string
          example: CA
        zipCode:
          type: string
          example: "94102"
        country:
          type: string
          example: USA

    Pagination:
      type: object
      properties:
        page:
          type: integer
          example: 1
        limit:
          type: integer
          example: 20
        total:
          type: integer
          example: 150
        pages:
          type: integer
          example: 8

    Error:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            code:
              type: string
              example: VALIDATION_ERROR
            message:
              type: string
              example: Validation failed
            details:
              type: array
              items:
                type: object
                properties:
                  field:
                    type: string
                    example: email
                  message:
                    type: string
                    example: Email format is invalid

  responses:
    Unauthorized:
      description: Unauthorized - Invalid or missing authentication token
      content:
        application/json:
          schema:
            type: object
            properties:
              success:
                type: boolean
                example: false
              error:
                type: object
                properties:
                  code:
                    type: string
                    example: UNAUTHORIZED
                  message:
                    type: string
                    example: Authentication required

    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            type: object
            properties:
              success:
                type: boolean
                example: false
              error:
                type: object
                properties:
                  code:
                    type: string
                    example: NOT_FOUND
                  message:
                    type: string
                    example: Resource not found

    ValidationError:
      description: Validation error
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

## Generating Documentation from OpenAPI Spec

### Using Swagger UI

```bash
# Install Swagger UI
npm install swagger-ui-express

# Serve documentation
```

```javascript
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const YAML = require('yamljs');

const app = express();
const swaggerDocument = YAML.load('./openapi.yaml');

app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerDocument));

app.listen(3000, () => {
  console.log('API documentation available at http://localhost:3000/api-docs');
});
```

### Using Redoc

```bash
# Install Redoc
npm install redoc-cli

# Generate HTML documentation
redoc-cli bundle openapi.yaml -o api-docs.html

# Serve documentation
redoc-cli serve openapi.yaml
```

### Using Postman

```bash
# Import OpenAPI spec into Postman
# File → Import → Select openapi.yaml

# Generate collection from OpenAPI spec
# Postman automatically creates requests for all endpoints
```

## Auto-Generating OpenAPI from Code

### Using swagger-jsdoc (Node.js)

```javascript
const swaggerJsdoc = require('swagger-jsdoc');

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'E-Commerce API',
      version: '1.0.0',
    },
  },
  apis: ['./routes/*.js'],
};

const openapiSpecification = swaggerJsdoc(options);
```

**Annotate routes:**

```javascript
/**
 * @openapi
 * /api/v1/products:
 *   get:
 *     tags:
 *       - Products
 *     summary: List products
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           minimum: 1
 *           default: 1
 *     responses:
 *       200:
 *         description: Products retrieved successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/Product'
 */
router.get('/products', productController.list);
```

### Using FastAPI (Python)

FastAPI automatically generates OpenAPI documentation:

```python
from fastapi import FastAPI, Query
from pydantic import BaseModel
from typing import List, Optional

app = FastAPI(
    title="E-Commerce API",
    description="REST API for e-commerce platform",
    version="1.0.0"
)

class Product(BaseModel):
    id: str
    name: str
    price: float
    category: str

@app.get(
    "/products",
    response_model=List[Product],
    tags=["Products"],
    summary="List products",
    description="Retrieve a paginated list of products"
)
async def list_products(
    page: int = Query(1, ge=1, description="Page number"),
    limit: int = Query(20, ge=1, le=100, description="Items per page"),
    category: Optional[str] = Query(None, description="Filter by category")
):
    # Implementation
    pass

# Auto-generated docs available at:
# http://localhost:8000/docs (Swagger UI)
# http://localhost:8000/redoc (ReDoc)
```

## Best Practices

### OpenAPI Specification

**Structure:**
- Use clear and consistent naming
- Group endpoints by tags
- Reuse components (schemas, parameters, responses)
- Include examples for all requests and responses
- Document error responses

**Descriptions:**
- Write clear, concise descriptions
- Include usage examples
- Document authentication requirements
- Explain query parameters and filters
- Note any special behaviors or limitations

**Versioning:**
- Include API version in URL (`/v1/`, `/v2/`)
- Document version changes in info section
- Maintain separate specs for each major version
- Use semantic versioning

### Documentation Quality

**Completeness:**
- Document all endpoints
- Include all possible responses
- Specify all parameters
- Document authentication methods
- Provide examples for complex operations

**Clarity:**
- Use consistent terminology
- Provide clear descriptions
- Include practical examples
- Document edge cases
- Explain error codes

**Maintenance:**
- Keep documentation in sync with code
- Update docs before deploying changes
- Version documentation alongside code
- Review docs regularly

## Quick Reference

### OpenAPI 3.0 Key Sections

```yaml
openapi: 3.0.3              # OpenAPI version
info:                        # API metadata
servers:                     # API servers
tags:                        # Endpoint groupings
paths:                       # API endpoints
components:
  schemas:                   # Data models
  parameters:                # Reusable parameters
  responses:                 # Reusable responses
  securitySchemes:           # Auth methods
```

### Common Data Types

| Type | Format | Example |
|------|--------|---------|
| string | - | "text" |
| string | email | "user@example.com" |
| string | uri | "https://example.com" |
| string | date | "2024-01-15" |
| string | date-time | "2024-01-15T10:30:00Z" |
| string | uuid | "550e8400-e29b..." |
| integer | int32 | 42 |
| number | float | 123.45 |
| boolean | - | true |
| array | - | [1, 2, 3] |
| object | - | {"key": "value"} |

### HTTP Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET, PUT request |
| 201 | Created | Successful POST request |
| 204 | No Content | Successful DELETE request |
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable Entity | Validation error |
| 500 | Internal Server Error | Server error |

## Keywords

**English keywords:**
api documentation, openapi, swagger, rest api documentation, api specification, api reference, interactive api docs, swagger ui, redoc, api docs generation, openapi 3.0

**Chinese keywords (中文關鍵詞):**
API文檔, OpenAPI, Swagger, REST API文檔, API規格, API參考, 互動式API文檔, API文檔生成, OpenAPI規格
