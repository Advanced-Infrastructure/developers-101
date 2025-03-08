# Best Practices for OpenAPI Documentation

Writing clear and comprehensive OpenAPI documentation is critical for developer adoption and API longevity. Below are best practices for versioning, model management, and endpoint documentation.

---

## 1. Model Management

Models (schemas) define the structure of request/response data. Consistency here reduces confusion.

### Best Practices:
- **Leverage JSON Schema**: Use OpenAPI's schema syntax to enforce validation.
  
  ```yaml
  components:
    schemas:
      User:
        type: object
        required:
          - id
          - name
        properties:
          id:
            type: integer
            format: int64
          name:
            type: string
            example: "Jane Doe"
  ```
- **Reuse Components**: Define reusable schemas, parameters, and responses under `components` to avoid duplication.
- **Provide Examples**: Include sample data for clarity.
  
  ```yaml
  examples:
    UserExample:
      value:
        id: 1
        name: "Jane Doe"
  ```
- **Document Enums and Constraints**: Specify allowed values, formats, and limits.
  
  ```yaml
  status:
    type: string
    enum: [active, inactive, pending]
    minLength: 4
    maxLength: 8
  ```
- **Version Models**: Introduce new schema versions (e.g., `UserV2`) for breaking changes.

---

## 2. Endpoint Documentation

Clear endpoint docs help developers integrate quickly.

### Best Practices:
- **Summarize and Describe**:
  
  ```yaml
  /users/{id}:
    get:
      summary: Get a user by ID
      description: Retrieves a single user's details, including name and contact info.
  ```
- **Document All Parameters**:
  - Specify path, query, header, and body parameters.
  - Include examples and required flags.
  
  ```yaml
  parameters:
    - in: path
      name: id
      required: true
      schema:
        type: integer
      example: 42
  ```
- **Cover All Responses**:
  - Define HTTP status codes, especially errors (4xx/5xx).
  - Add examples for success and failure cases.
  
  ```yaml
  responses:
    200:
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/User"
    404:
      description: User not found
  ```
- **Use Tags for Grouping**: Organize endpoints by functionality (e.g., `Users`, `Auth`).
  
  ```yaml
  tags:
    - name: Users
      description: Manage user accounts
  ```
- **Show Request/Response Examples**:
  
  ```yaml
  requestBody:
    content:
      application/json:
        schema:
          $ref: "#/components/schemas/User"
        examples:
          UserExample:
            value:
              name: "Jane Doe"
  ```
- **Standardize Error Formats**:
  
  ```yaml
  components:
    schemas:
      Error:
        type: object
        properties:
          code:
            type: integer
          message:
            type: string
  ```

---

## Where do I add documentation?
Good question, cowboy. We use serverless.yml to manage API declarations and deployments to APIGateway. So, it made sense to add a block for endpoint documentation.

All documentation is added in the `docs` folder. The yaml file follows the OpenAPI standards. Read about OpenAPI spec here [openAPI](https://github.com/OAI/OpenAPI-Specification)

Next, edit serverless open `serverless.yml` in `be-api-test` [serverless](https://github.com/Advanced-Infrastructure/be-api-test/blob/dev/serverless.yml)
Under functions, see the documentation block `documentation: ${file(./docs/api/documentation.yml):endpoints.api-endpoint}`

Make sure you declare the documentation block for every endpoint.


---


## Final Tips
- **Validate Your Spec**: Use tools like Swagger Editor or Spectral to lint your OpenAPI file.
- **Automate Documentation**: Generate interactive docs with Swagger UI or Redoc.
- **Keep It Consistent**: Follow the same structure, naming conventions, and style across all endpoints.

Well-documented APIs reduce support overhead and empower developers to build faster. 📘🚀
