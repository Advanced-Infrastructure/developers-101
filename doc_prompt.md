# Python Controller Documentation Generator

## Prompt Structure

```
You are a technical documentation specialist. Generate comprehensive documentation for the Python controller code provided below.

**Context**: This is a controller in a Python web service that handles specific business logic and API endpoints.

**Requirements**:
1. Analyze the controller class and its methods
2. Identify the business domain and purpose
3. Document each endpoint/method with proper structure
4. Include error handling, dependencies, and data models
5. Follow the provided documentation template strictly

**Code to Document**:
```python
[INSERT_CONTROLLER_CODE_HERE]
```

**Documentation Template to Follow**:
[INSERT_TEMPLATE_BELOW]

Generate the documentation following the exact template structure. Focus on clarity, completeness, and technical accuracy.
```

## Documentation Template

```markdown
# [Controller Name] Controller

## Overview
**Purpose**: [Brief description of what this controller handles]
**Domain**: [Business domain - e.g., User Management, Orders, Payments]
**Base Path**: [API base path if applicable]

## Dependencies
- **Framework**: [Flask/FastAPI/Django]
- **Database**: [Database type/ORM]
- **External Services**: [List any external APIs/services]
- **Key Libraries**: [Important imports]

## Class Structure

### `[ControllerClassName]`
**Description**: [What this controller class does]

**Attributes**:
- `attribute_name` (type): Description
- `another_attr` (type): Description

## Endpoints/Methods

### `[method_name]` - [HTTP_METHOD] `/path`

**Purpose**: [What this method does]

**Parameters**:
- `param1` (type, required/optional): Description
- `param2` (type, required/optional): Description

**Request Body** (if applicable):
```json
{
  "field1": "string",
  "field2": "number"
}
```

**Response**:
```json
{
  "status": "success",
  "data": {
    "result": "object"
  }
}
```

**Error Responses**:
- `400 Bad Request`: [When this occurs]
- `404 Not Found`: [When this occurs]
- `500 Internal Server Error`: [When this occurs]

**Business Logic**:
1. [Step 1 of what the method does]
2. [Step 2 of the process]
3. [Step 3 and result]

**Dependencies**:
- Database tables: [table1, table2]
- External APIs: [service_name]
- Other controllers: [if any]

---

### `[next_method_name]` - [HTTP_METHOD] `/path`
[Repeat structure for each method]

## Data Models

### Input Models
```python
class [InputModel]:
    field1: str
    field2: int
    field3: Optional[bool]
```

### Output Models
```python
class [OutputModel]:
    id: int
    name: str
    created_at: datetime
```

## Error Handling
- **Validation Errors**: [How handled]
- **Database Errors**: [How handled]
- **External Service Errors**: [How handled]

## Security Considerations
- **Authentication**: [Required/Not required]
- **Authorization**: [Role-based/Permission-based]
- **Input Validation**: [What validations are in place]

## Testing
**Unit Tests**: [Location/Coverage]
**Integration Tests**: [Endpoints covered]

## Configuration
**Environment Variables**:
- `VAR_NAME`: Description and default value
- `ANOTHER_VAR`: Description and default value

## Usage Examples

### Example 1: [Common Use Case]
```bash
curl -X POST http://api.example.com/endpoint \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```

**Response**:
```json
{
  "status": "success",
  "data": {...}
}
```

## Notes
- [Important implementation details]
- [Known limitations]
- [Future improvements]

---
**Last Updated**: [Date]
**Maintainer**: [Team/Person]
**Version**: [Version number]
```

## Usage Script

```python
# generate_docs.py
import os
import openai
from pathlib import Path

def generate_controller_docs(controller_file, output_dir):
    # Read controller code
    with open(controller_file, 'r') as f:
        code = f.read()
    
    # Load prompt template
    prompt = f"""
    You are a technical documentation specialist. Generate comprehensive documentation for the Python controller code provided below.

    **Context**: This is a controller in a Python web service that handles specific business logic and API endpoints.

    **Requirements**:
    1. Analyze the controller class and its methods
    2. Identify the business domain and purpose  
    3. Document each endpoint/method with proper structure
    4. Include error handling, dependencies, and data models
    5. Follow the provided documentation template strictly

    **Code to Document**:
    ```python
    {code}
    ```

    **Documentation Template to Follow**:
    {TEMPLATE_HERE}

    Generate the documentation following the exact template structure. Focus on clarity, completeness, and technical accuracy.
    """
    
    # Call LLM API
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=4000
    )
    
    # Save documentation
    controller_name = Path(controller_file).stem
    output_file = f"{output_dir}/{controller_name}_documentation.md"
    
    with open(output_file, 'w') as f:
        f.write(response.choices[0].message.content)
    
    print(f"Documentation generated: {output_file}")

# Usage
generate_controller_docs("controllers/user_controller.py", "docs/controllers")
```

## Enhanced Script with Error Handling

```python
# enhanced_generate_docs.py
import os
import openai
from pathlib import Path
import json
import logging

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class ControllerDocGenerator:
    def __init__(self, api_key, model="gpt-4"):
        self.client = openai.OpenAI(api_key=api_key)
        self.model = model
        
    def load_template(self, template_file=None):
        """Load documentation template from file or use default"""
        if template_file and Path(template_file).exists():
            with open(template_file, 'r') as f:
                return f.read()
        
        # Default template (insert the template from above here)
        return """[DEFAULT_TEMPLATE_HERE]"""
    
    def generate_docs(self, controller_file, output_dir, template_file=None):
        """Generate documentation for a controller file"""
        try:
            # Validate inputs
            if not Path(controller_file).exists():
                raise FileNotFoundError(f"Controller file not found: {controller_file}")
            
            Path(output_dir).mkdir(parents=True, exist_ok=True)
            
            # Read controller code
            with open(controller_file, 'r', encoding='utf-8') as f:
                code = f.read()
            
            # Load template
            template = self.load_template(template_file)
            
            # Create prompt
            prompt = f"""
You are a technical documentation specialist. Generate comprehensive documentation for the Python controller code provided below.

**Context**: This is a controller in a Python web service that handles specific business logic and API endpoints.

**Requirements**:
1. Analyze the controller class and its methods
2. Identify the business domain and purpose  
3. Document each endpoint/method with proper structure
4. Include error handling, dependencies, and data models
5. Follow the provided documentation template strictly

**Code to Document**:
```python
{code}
```

**Documentation Template to Follow**:
{template}

Generate the documentation following the exact template structure. Focus on clarity, completeness, and technical accuracy.
"""
            
            # Call LLM API
            logger.info(f"Generating documentation for {controller_file}...")
            response = self.client.chat.completions.create(
                model=self.model,
                messages=[{"role": "user", "content": prompt}],
                max_tokens=4000,
                temperature=0.3
            )
            
            # Save documentation
            controller_name = Path(controller_file).stem
            output_file = Path(output_dir) / f"{controller_name}_documentation.md"
            
            with open(output_file, 'w', encoding='utf-8') as f:
                f.write(response.choices[0].message.content)
            
            logger.info(f"Documentation generated: {output_file}")
            return output_file
            
        except Exception as e:
            logger.error(f"Error generating documentation: {str(e)}")
            raise
    
    def generate_batch_docs(self, controllers_dir, output_dir, template_file=None):
        """Generate documentation for all controllers in a directory"""
        controllers_path = Path(controllers_dir)
        
        if not controllers_path.exists():
            raise FileNotFoundError(f"Controllers directory not found: {controllers_dir}")
        
        python_files = list(controllers_path.glob("*.py"))
        
        if not python_files:
            logger.warning(f"No Python files found in {controllers_dir}")
            return []
        
        generated_files = []
        
        for controller_file in python_files:
            try:
                output_file = self.generate_docs(
                    str(controller_file), 
                    output_dir, 
                    template_file
                )
                generated_files.append(output_file)
            except Exception as e:
                logger.error(f"Failed to process {controller_file}: {str(e)}")
                continue
        
        logger.info(f"Generated documentation for {len(generated_files)} controllers")
        return generated_files

# Usage Examples
if __name__ == "__main__":
    # Initialize generator
    generator = ControllerDocGenerator(api_key="your-openai-api-key")
    
    # Generate single controller documentation
    generator.generate_docs(
        controller_file="controllers/user_controller.py",
        output_dir="docs/controllers"
    )
    
    # Generate documentation for all controllers
    generator.generate_batch_docs(
        controllers_dir="controllers/",
        output_dir="docs/controllers",
        template_file="templates/controller_template.md"  # Optional custom template
    )
```

## Configuration File

```yaml
# config.yaml
documentation:
  openai:
    api_key: "${OPENAI_API_KEY}"
    model: "gpt-4"
    max_tokens: 4000
    temperature: 0.3
  
  paths:
    controllers_dir: "controllers/"
    output_dir: "docs/controllers/"
    template_file: "templates/controller_template.md"
  
  settings:
    batch_processing: true
    overwrite_existing: true
    include_tests: true
```

## CLI Integration

```python
# cli.py
import click
import yaml
from pathlib import Path
from enhanced_generate_docs import ControllerDocGenerator

@click.command()
@click.option('--controller', '-c', help='Single controller file to document')
@click.option('--controllers-dir', '-d', help='Directory containing controllers')
@click.option('--output-dir', '-o', default='docs/controllers', help='Output directory')
@click.option('--template', '-t', help='Custom template file')
@click.option('--config', help='Configuration file')
@click.option('--api-key', envvar='OPENAI_API_KEY', help='OpenAI API key')
def generate_docs(controller, controllers_dir, output_dir, template, config, api_key):
    """Generate documentation for Python controllers using LLM"""
    
    # Load configuration if provided
    if config and Path(config).exists():
        with open(config, 'r') as f:
            config_data = yaml.safe_load(f)
        
        # Override with config values
        api_key = api_key or config_data['documentation']['openai']['api_key']
        output_dir = output_dir or config_data['documentation']['paths']['output_dir']
        template = template or config_data['documentation']['paths']['template_file']
    
    if not api_key:
        raise click.ClickException("OpenAI API key is required")
    
    generator = ControllerDocGenerator(api_key=api_key)
    
    try:
        if controller:
            # Generate single controller docs
            generator.generate_docs(controller, output_dir, template)
        elif controllers_dir:
            # Generate batch docs
            generator.generate_batch_docs(controllers_dir, output_dir, template)
        else:
            raise click.ClickException("Either --controller or --controllers-dir must be specified")
            
        click.echo("✅ Documentation generation completed successfully!")
        
    except Exception as e:
        raise click.ClickException(f"❌ Error: {str(e)}")

if __name__ == '__main__':
    generate_docs()
```

## Usage Examples

### Command Line Usage

```bash
# Generate docs for single controller
python cli.py --controller controllers/user_controller.py --output-dir docs/

# Generate docs for all controllers in directory
python cli.py --controllers-dir controllers/ --output-dir docs/controllers/

# Use custom template
python cli.py --controllers-dir controllers/ --template custom_template.md

# Use configuration file
python cli.py --config config.yaml
```

### Integration in CI/CD

```yaml
# .github/workflows/generate-docs.yml
name: Generate Controller Documentation

on:
  push:
    paths:
      - 'controllers/**/*.py'
  workflow_dispatch:

jobs:
  generate-docs:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      
      - name: Install dependencies
        run: |
          pip install openai click pyyaml
      
      - name: Generate documentation
        run: |
          python cli.py --controllers-dir controllers/ --output-dir docs/controllers/
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      
      - name: Commit documentation
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add docs/
          git commit -m "Auto-update controller documentation" || exit 0
          git push
```