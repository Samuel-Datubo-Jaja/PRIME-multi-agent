# Database Package for Alex Financial Planner

This package provides the shared database layer for all Alex services, with Aurora Serverless v2 PostgreSQL and Data API.

## Core Components

### Essential Files

**Package Structure:**
- `pyproject.toml` - Package configuration
- `src/` - Main package code
  - `__init__.py` - Package exports
  - `client.py` - Data API client wrapper with automatic type casting
  - `models.py` - Database models
  - `schemas.py` - Pydantic schemas for validation and LLM integration

**Database Management:**
- `migrations/001_schema.sql` - Database schema definition
- `run_migrations.py` - Execute migrations against Aurora
- `seed_data.py` - Load 22 ETF instruments with validated allocations
- `reset_db.py` - Reset database (drop tables, recreate, load seed data)

**Configuration:**
- `.env` file with Aurora connection details from Terraform outputs

### Test & Verification Files

Three focused test scripts:
- `test_data_api.py` - Initial Aurora connection test
- `test_db.py` - Complete test suite (database operations + Pydantic validation)
- `verify_database.py` - Database state verification (tables, counts, allocations)

## Pydantic Schema Structure

All database operations use Pydantic schemas for validation:

### Literal Types (Constrained Values)
- **Regions**: `north_america`, `europe`, `asia`, `latin_america`, etc.
- **Asset Classes**: `equity`, `fixed_income`, `real_estate`, `commodities`, `cash`, `alternatives`
- **Sectors**: `technology`, `healthcare`, `financials`, plus bond-specific like `treasury`, `corporate`

### Key Features
- Automatic validation that allocations sum to 100%
- Natural language Field descriptions for LLM compatibility
- Type casting for JSONB, numeric, date, and UUID fields
- Suitable for OpenAI/Anthropic function calling and structured outputs

## Usage

### Installation in Other Services
```bash
cd backend/[service_name]
uv add --editable ../database
```

### Import in Python
```python
from src.models import Database
from src.schemas import InstrumentCreate, PortfolioAnalysis

# Initialize database
db = Database()

# Create instrument with validation
instrument = InstrumentCreate(
    symbol="VTI",
    name="Vanguard Total Stock Market ETF",
    instrument_type="etf",
    allocation_regions={"north_america": 100},
    allocation_sectors={"diversified": 100},
    allocation_asset_class={"equity": 100}
)
result = db.instruments.create_instrument(instrument)
```

### Database Operations
```bash
# Reset database with test data
uv run reset_db.py --with-test-data

# Load only seed data (22 instruments)
uv run seed_data.py

# Run tests
uv run test_db.py
uv run test_validation.py
```

## Environment Variables

The package reads Aurora configuration from environment variables (.env file):
- `AURORA_CLUSTER_ARN` - Aurora cluster ARN from Terraform output
- `AURORA_SECRET_ARN` - Secrets Manager ARN from Terraform output
- `AURORA_DATABASE` - Database name (default: 'alex')
- `AWS_REGION` - AWS region (default: 'us-east-1')

## Cost Management

Aurora Serverless v2 costs ~$1.44-$2.88/day. Use the Terraform cost management script:
```bash
cd ../../terraform
uv run aurora_cost_management.py
```

Options:
- Pause cluster (reduces cost to ~$0.20/day)
- Resume cluster
- Destroy cluster (eliminate all costs)

## Testing

Three simple test commands:
```bash
# 1. Initial setup (after Terraform deployment)
uv run test_data_api.py        # Test Data API connection

# 2. Run all tests
uv run test_db.py              # Complete test suite

# 3. Verify database state
uv run verify_database.py      # Shows tables, counts, and validations
```