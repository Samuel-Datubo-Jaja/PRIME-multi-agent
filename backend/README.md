# 🎯 Alex Backend - Agent Orchestration System

This directory contains all the backend Lambda functions for the Alex Financial Planner platform.

## 📁 Project Structure

Each service is a separate uv project with its own Lambda function:

- **`planner/`** - Orchestrator agent that coordinates all other agents
- **`tagger/`** - Classifies financial instruments (ETFs, stocks)
- **`reporter/`** - Generates narrative portfolio analysis
- **`charter/`** - Creates visualization data and charts
- **`retirement/`** - Runs retirement projections and Monte Carlo simulations
- **`database/`** - Shared database models and utilities (used by all services)

## 🚀 Deployment

### Deploy All Lambda Functions

```bash
cd backend

# Deploy all functions (auto-packages missing ones)
uv run deploy_all_lambdas.py

# Force re-package and deploy all functions
uv run deploy_all_lambdas.py --package
```

The deployment script will:
1. Check for existing deployment packages
2. Automatically package any missing functions
3. Upload packages to S3 
4. Update Lambda function code

**Note:** Use `--package` flag when you've made code changes to ensure the latest code is deployed.

### Deploy Individual Function

If you only want to package and deploy a specific function:

```bash
cd backend/planner  # or any other service
uv run package_docker.py  # Creates the deployment package
```

Then use `deploy_all_lambdas.py` which will detect and deploy the new package.

## 🧪 Testing the System

### Quick Test (Recommended)

Run the full orchestration test:

```bash
cd backend/planner
uv run run_full_test.py
```

This will:
1. Create a test job for the test portfolio
2. Submit it to the SQS queue
3. Trigger all 5 Lambda agents
4. Show you the results in real-time

### What You'll See

The test will show:
- **Portfolio being analyzed**: 3 accounts with ETFs like SPY, QQQ, BND
- **Agent coordination**: Planner → Tagger → Reporter/Charter/Retirement
- **S3 Vectors search**: Agents searching the financial knowledge base
- **Final results**: Investment recommendations and retirement projections

### Expected Timeline

- **0-10 seconds**: Job creation and submission
- **10-30 seconds**: Instrument tagging (if needed)
- **30-90 seconds**: Report generation, chart creation, retirement analysis
- **Total time**: ~90-120 seconds

## 📊 Manual Testing

If you want more control over the testing process:

### 1. Verify Test Data Exists
```bash
cd backend/database
uv run verify_database.py
```

### 2. Create and Submit a Job
```bash
cd backend/planner
uv run test_integration.py
```

### 3. Check Job Results
```bash
cd backend/planner
uv run check_jobs.py
```

## 📝 View Live Logs

Watch Lambda execution in real-time:

```bash
# Watch the orchestrator
aws logs tail /aws/lambda/alex-planner --follow --region us-east-1

# Watch all Part 6 lambdas
aws logs tail --follow --region us-east-1 \
  /aws/lambda/alex-planner \
  /aws/lambda/alex-tagger \
  /aws/lambda/alex-reporter \
  /aws/lambda/alex-charter \
  /aws/lambda/alex-retirement
```

## 🔍 How It Works

1. **Planner Lambda** (Orchestrator)
   - Receives job from SQS queue
   - Loads portfolio from Aurora database
   - Checks which instruments need classification
   - Coordinates calls to other agents
   - Compiles final results

2. **Tagger Lambda**
   - Classifies financial instruments
   - Provides asset allocation breakdowns
   - Updates instrument data in database

3. **Reporter Lambda**
   - Generates narrative analysis
   - Searches S3 Vectors for market insights
   - Creates personalized recommendations

4. **Charter Lambda**
   - Creates chart data for visualizations
   - Generates allocation breakdowns
   - Produces performance projections

5. **Retirement Lambda**
   - Runs Monte Carlo simulations
   - Projects retirement readiness
   - Calculates success probabilities

## 🐛 Troubleshooting

### If deployment fails:
- Ensure Docker is running (required for packaging)
- Check AWS credentials: `aws sts get-caller-identity`
- Verify S3 bucket exists: `aws s3 ls | grep alex-lambda-packages`

### If test hangs or fails:
- Check SQS queue for stuck messages:
  ```bash
  aws sqs get-queue-attributes \
    --queue-url https://sqs.us-east-1.amazonaws.com/$(aws sts get-caller-identity --query Account --output text)/alex-analysis-jobs \
    --attribute-names All \
    --region us-east-1
  ```

- Check Lambda errors in CloudWatch:
  ```bash
  aws logs describe-log-groups --query "logGroups[?contains(logGroupName, 'alex')]" --region us-east-1
  ```

- Ensure test data exists:
  ```bash
  cd backend/database
  uv run reset_db.py --with-test-data
  ```

### Common Issues:

1. **"Invalid result type: <class 'str'>"** - Agent missing `output_type` parameter
2. **Rate limits** - Using Claude 3.5 Haiku inference profile helps avoid throttling
3. **Package too large** - Deployment packages must be under 250MB uncompressed

## ✅ Success Indicators

You'll know everything is working when:
- Status changes: `pending` → `running` → `completed`
- Executive summary appears with personalized advice
- 5-7 specific recommendations are generated
- Results reference actual market data from S3 Vectors
- All 5 Lambda functions show successful execution

## 🔧 Development Workflow

1. Make code changes in the appropriate service directory
2. Test locally if possible: `uv run test_local.py`
3. Package and deploy: `uv run deploy_all_lambdas.py --package`
4. Run integration test: `uv run run_full_test.py`
5. Check CloudWatch logs for any errors

## 📚 Additional Resources

- [Part 6 Guide](../guides/6_agents.md) - Detailed agent implementation guide
- [Database Schema](database/src/schemas.py) - Pydantic models and validation
- [Agent Architecture](../guides/agent_architecture.md) - System design documentation

Enjoy watching your AI agents work together! 🚀