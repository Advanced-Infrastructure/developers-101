# Automating Postman Collections from Serverless Framework

This guide provides a comprehensive approach to automatically generate rich and detailed Postman collections directly from your Serverless Framework configuration. By using your `serverless.yml` file as the single source of truth for your API structure, you ensure that your documentation remains synchronized with your deployed infrastructure.

## Key Benefits

- **Automated Documentation**: Eliminates manual documentation maintenance
- **Single Source of Truth**: Reduces discrepancies between code and documentation
- **API Gateway Validation**: Enables automatic request validation at the infrastructure level
- **Team Synchronization**: Keeps all team members updated with the latest API changes

## Prerequisites

Before starting, ensure you have:

- Node.js and npm installed
- A Serverless Framework project configured
- Postman account with workspace access
- CI/CD pipeline setup (GitHub Actions, GitLab CI, etc.)

## Step 1: Install and Configure the Plugin

### 1.1 Install the Plugin

Add the `serverless-postman-collection` plugin to your project as a development dependency:

```bash
npm install --save-dev serverless-postman-collection
```

### 1.2 Configure serverless.yml

Update your `serverless.yml` file to include the plugin configuration:

```yaml
# serverless.yml

# Add the plugin to your plugins list
plugins:
  - serverless-python-requirements
  # ... other existing plugins
  - serverless-postman-collection

custom:
  # ... other custom configurations

  # Configure the postman plugin
  postman:
    name: "BE Dataset Service (Auto-Generated)"
    description: "API endpoints for the be-dataset-service, generated from serverless.yml"
    # Output directory for generated files
    filename: "postman/collection.json"
    headers:
      - key: "Content-Type"
        value: "application/json"
      # Use Postman variables for dynamic values
      - key: "Authorization"
        value: "{{authToken}}"
```

## Step 2: Define Request Schemas and Parameters

### 2.1 Create Schema Directory Structure

Organize your JSON schemas in a dedicated directory:

```bash
mkdir schemas
```

### 2.2 Create JSON Schema Files

For each endpoint that requires a request body, create a corresponding JSON schema file. This approach keeps your `serverless.yml` clean and maintainable.

**Example: `schemas/update-dataset-config.json`**

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "DatasetConfigUpdateInput",
  "description": "Payload to update the dataset configuration",
  "type": "object",
  "properties": {
    "displayName": {
      "type": "string",
      "description": "The new display name for the dataset",
      "minLength": 1,
      "maxLength": 255
    },
    "isPublic": {
      "type": "boolean",
      "description": "Set to true to make the dataset public"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string",
        "minLength": 1
      },
      "description": "Array of tags associated with the dataset"
    }
  },
  "required": ["displayName", "isPublic"],
  "additionalProperties": false
}
```

### 2.3 Best Practices for Schema Design

- Use descriptive titles and descriptions
- Define appropriate validation rules (minLength, maxLength, format)
- Set `additionalProperties: false` to prevent unexpected fields
- Include examples where helpful
- Use consistent naming conventions

## Step 3: Configure Function Endpoints

### 3.1 Link Schemas and Parameters in serverless.yml

Reference your external schema files and define query parameters within your function's HTTP event configuration.

```yaml
# serverless.yml

functions:
  be-dataset-service:
    name: be-dataset-service
    handler: handler.handle
    timeout: 30
    memorySize: 512
    events:
      # GET endpoint with query parameters
      - http:
          path: /ds/{dno}/la/{la}/datasets
          method: GET
          cors: true
          request:
            parameters:
              paths:
                dno: true    # Required path parameter
                la: true     # Required path parameter
              querystrings:
                dataset_type: true   # Required query parameter
                status: false        # Optional query parameter
                limit: false         # Optional query parameter
                offset: false        # Optional query parameter

      # PUT endpoint with request body schema
      - http:
          path: /ds/{dno}/la/{la}/datasets/{dataset_id}/config
          method: PUT
          cors: true
          request:
            parameters:
              paths:
                dno: true
                la: true
                dataset_id: true
            schemas:
              application/json:
                schema: ${file(./schemas/update-dataset-config.json)}

      # POST endpoint example
      - http:
          path: /ds/{dno}/la/{la}/datasets
          method: POST
          cors: true
          request:
            parameters:
              paths:
                dno: true
                la: true
            schemas:
              application/json:
                schema: ${file(./schemas/create-dataset.json)}
```

### 3.2 Parameter Configuration Guidelines

- **Required Parameters**: Set to `true`
- **Optional Parameters**: Set to `false`
- **Path Parameters**: Always mark as required (`true`)
- **Schema References**: Use `${file()}` syntax for external schemas

## Step 4: Generate and Test Your Collection

### 4.1 Generate the Collection

Run the Serverless package command to generate your Postman collection:

```bash
serverless package
```

**Note**: You can also use `serverless deploy` which will package and deploy your service.

### 4.2 Verify Generated Files

After running the command, verify that the collection file has been created:

```bash
ls -la postman/
# Should show: collection.json
```

### 4.3 Import into Postman

1. Open Postman application
2. Click the **Import** button in the top navigation
3. Select **File** tab
4. Drag and drop your `postman/collection.json` file
5. Click **Import** to complete the process

### 4.4 Test Your Collection

After importing:

- Verify all endpoints are present
- Check that request bodies have proper schemas
- Confirm query parameters are pre-configured
- Test with sample requests to ensure functionality

## Step 5: Automate Collection Updates (CI/CD Integration)

### 5.1 Obtain Postman API Credentials

#### Get Your Postman API Key

1. Navigate to [Postman API Keys page](https://go.postman.co/settings/me/api-keys)
2. Click **Generate API Key**
3. Provide a descriptive name (e.g., "Serverless CI/CD Integration")
4. Copy and securely store the key

#### Find Your Collection ID

1. Import your collection into Postman (one-time setup)
2. Select the collection in your workspace
3. Click the information icon (ⓘ) in the right sidebar
4. Copy the **Collection ID** from the details panel

### 5.2 Store Secrets in CI/CD Environment

Add the following secrets to your CI/CD platform:

- `POSTMAN_API_KEY`: Your Postman API key
- `POSTMAN_COLLECTION_ID`: Your collection's unique identifier

### 5.3 Create Update Script

Create a deployment script to handle the Postman API interaction:

**File: `scripts/update_postman.sh`**

```bash
#!/bin/bash

# Exit immediately if a command exits with a non-zero status
set -e

# Validate required environment variables
if [ -z "$POSTMAN_API_KEY" ] || [ -z "$POSTMAN_COLLECTION_ID" ]; then
  echo "❌ Error: POSTMAN_API_KEY and POSTMAN_COLLECTION_ID must be set"
  exit 1
fi

COLLECTION_FILE="postman/collection.json"
COLLECTION_ID="$POSTMAN_COLLECTION_ID"
API_KEY="$POSTMAN_API_KEY"

echo "🔄 Updating Postman collection..."

# Validate collection file exists
if [ ! -f "$COLLECTION_FILE" ]; then
  echo "❌ Error: Collection file not found at $COLLECTION_FILE"
  exit 1
fi

# Validate JSON format
if ! jq empty "$COLLECTION_FILE" 2>/dev/null; then
  echo "❌ Error: Invalid JSON in collection file"
  exit 1
fi

# Wrap collection in required format for Postman API
JSON_PAYLOAD=$(jq --argjson collection "$(cat ${COLLECTION_FILE})" '.collection = $collection' <<< '{}')

# Update collection via Postman API
HTTP_STATUS=$(curl -s -o /tmp/postman_response.json -w "%{http_code}" \
  -X PUT "https://api.getpostman.com/collections/${COLLECTION_ID}" \
  --header "Content-Type: application/json" \
  --header "X-Api-Key: ${API_KEY}" \
  --data "${JSON_PAYLOAD}")

# Check response status
if [ "$HTTP_STATUS" -eq 200 ]; then
  echo "✅ Postman collection updated successfully!"
else
  echo "❌ Error: Failed to update collection (HTTP $HTTP_STATUS)"
  cat /tmp/postman_response.json
  exit 1
fi

# Cleanup temporary file
rm -f /tmp/postman_response.json
```

Make the script executable:

```bash
chmod +x scripts/update_postman.sh
```

### 5.4 GitHub Actions Integration

Create or update your GitHub Actions workflow:

**File: `.github/workflows/deploy.yml`**

```yaml
name: Deploy Service and Update Postman Collection

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Deploy to AWS
        run: npx serverless deploy --stage ${{ github.ref == 'refs/heads/main' && 'prod' || 'dev' }}
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Update Postman Collection
        if: github.ref == 'refs/heads/main'
        run: bash ./scripts/update_postman.sh
        env:
          POSTMAN_API_KEY: ${{ secrets.POSTMAN_API_KEY }}
          POSTMAN_COLLECTION_ID: ${{ secrets.POSTMAN_COLLECTION_ID }}

      - name: Notify team
        if: failure()
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: 'Deployment or Postman update failed'
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Troubleshooting

### Common Issues and Solutions

#### Collection Not Generated

**Problem**: No `collection.json` file created after running `serverless package`

**Solutions**:
- Verify plugin is listed in `plugins` section
- Check that `custom.postman` configuration is properly indented
- Ensure output directory exists: `mkdir -p postman`
- Run with verbose logging: `serverless package --verbose`

#### Schema Validation Errors

**Problem**: Invalid JSON schema preventing collection generation

**Solutions**:
- Validate schema files using online JSON schema validators
- Ensure proper JSON syntax (no trailing commas)
- Check file paths in `${file()}` references are correct
- Verify schema follows JSON Schema draft-07 specification

#### API Update Failures

**Problem**: Script fails to update Postman collection

**Solutions**:
- Verify API key has correct permissions
- Confirm collection ID is accurate
- Check network connectivity to Postman API
- Review error messages in script output
- Validate JSON payload format

#### Missing Dependencies

**Problem**: `jq` command not found in CI/CD environment

**Solutions**:
- Install jq in your CI/CD pipeline:
  ```yaml
  - name: Install jq
    run: sudo apt-get update && sudo apt-get install -y jq
  ```
- Use Docker image with jq pre-installed
- Replace jq with Node.js-based JSON processing

## Best Practices and Recommendations

### Schema Management

- **Version Control**: Keep all schema files in version control
- **Naming Convention**: Use descriptive, consistent file names
- **Validation**: Include appropriate validation rules in schemas
- **Documentation**: Add clear descriptions for all properties
- **Reusability**: Create reusable schema components for common structures

### Security Considerations

- **API Keys**: Never commit API keys to version control
- **Environment Isolation**: Use different collections for different environments
- **Access Control**: Limit API key permissions to minimum required scope
- **Rotation**: Regularly rotate API keys and update CI/CD secrets

### Monitoring and Maintenance

- **Collection Validation**: Regularly test generated collections
- **Schema Updates**: Keep schemas synchronized with API changes
- **Team Communication**: Notify team members of significant API changes
- **Backup Strategy**: Maintain backups of critical collections

### Performance Optimization

- **Selective Updates**: Only update collections when schemas change
- **Caching**: Cache generated collections in CI/CD pipelines