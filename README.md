# Primary Connect Order Tracking Demo

A demonstration application that searches Gmail for order update emails and cross-references them with CSV order data, using Azure OpenAI agents to provide comprehensive order status information.

## Overview

This demo showcases a multi-agent AI system that:
- Searches Gmail for order-related emails using the Gmail API
- Queries local CSV data for shipment and order details
- Uses Azure OpenAI's Agent Framework to orchestrate multiple specialized agents
- Structures data using Pydantic models for validation
- Provides synthesized order status combining both email and database information

For a visual representation of the complete workflow, see the [Workflow Diagram](workflow_diagram.md).

## Prerequisites

Before setting up this demo, ensure you have:

1. **Python 3.10 or higher** installed on your system
2. **Azure CLI** installed and configured
3. **Google Cloud Project** with Gmail API enabled
4. **VS Code** with Python and Jupyter extensions (recommended)
5. **Azure OpenAI Service** access with a deployed model

## Setup Instructions

### 1. Clone or Download the Project

```cmd
cd c:\source\primaryconnect
```

### 2. Create a Virtual Environment (Recommended)

```cmd
python -m venv venv
venv\Scripts\activate
```

### 3. Install Required Packages

Install all dependencies from the requirements file:

```cmd
pip install -r requirements.txt
```

Key packages installed:
- `agent-framework` - Microsoft's Agent Framework for building AI agents
- `google-api-python-client` - Gmail API client
- `google-auth-oauthlib` - OAuth authentication for Google APIs
- `azure-identity` - Azure authentication
- `pandas` - Data manipulation for CSV files
- `pydantic` - Data validation and modeling

### 4. Configure Gmail API Access

#### Step 4.1: Create a Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the **Gmail API**:
   - Navigate to **APIs & Services** > **Library**
   - Search for "Gmail API"
   - Click **Enable**

#### Step 4.2: Create OAuth 2.0 Credentials

1. Go to **APIs & Services** > **Credentials**
2. Click **Create Credentials** > **OAuth client ID**
3. Configure the OAuth consent screen if prompted:
   - Choose **External** user type
   - Add your email as a test user
4. Select **Desktop app** as the application type
5. Download the credentials JSON file
6. Rename it to `client_secret_718290307376-1gn4toeqcsi4pstk24n6pedui6odo5cm.apps.googleusercontent.com.json` (or update the filename in the notebook)
7. Place it in the project root directory

**Note**: On first run, a browser window will open asking you to authorize the application. After authorization, a `token.json` file will be created for subsequent runs.

### 5. Configure Azure OpenAI Access

#### Step 5.1: Azure CLI Authentication

Ensure you're logged into Azure CLI:

```cmd
az login
```

Verify your subscription:

```cmd
az account show
```

#### Step 5.2: Azure OpenAI Service

Make sure you have:
- An Azure OpenAI resource created in your Azure subscription
- A deployed model (e.g., GPT-4, GPT-4o, or GPT-3.5-turbo)
- Appropriate role assignments (e.g., **Cognitive Services OpenAI User**)

The demo uses `AzureCliCredential()` which automatically uses your Azure CLI credentials.

### 6. Prepare Sample Data

The demo includes a CSV file `mypc_data.csv` with sample order data. The file should have the following columns:

- Order Number
- Order Date
- Supplier Code
- Supplier Name
- Customer Site
- Site Name
- Pickup Date
- Delivery Window Start
- Delivery Window End
- Transport Mode
- Load ID
- Shipment ID
- Status
- Status Timestamp
- Pallets
- Cartons
- Weight (kg)
- Volume (m3)
- Temperature Zone
- Reference Notes

If you need to create your own sample data, ensure it follows this structure.

## Running the Demo

### Option 1: Run the Jupyter Notebook

1. Open `searchGmail.ipynb` in VS Code or Jupyter
2. Ensure your Python environment is activated
3. Run cells sequentially from top to bottom
4. The notebook demonstrates:
   - Gmail API authentication
   - Searching Gmail for specific orders
   - Creating AI functions (tools) for Gmail and CSV search
   - Building specialized agents
   - Orchestrating agents with Magentic workflow
   - Streaming results from the multi-agent system

### Option 2: Run Individual Components

You can also run parts of the notebook as standalone Python scripts by extracting the relevant cells.

## Project Structure

```
c:\source\primaryconnect\
├── searchGmail.ipynb           # Main Jupyter notebook with demo code
├── requirements.txt            # Python dependencies
├── mypc_data.csv              # Sample order/shipment data
├── client_secret_*.json       # Google OAuth credentials (you provide)
├── token.json                 # Auto-generated after first Gmail auth
├── workflow_diagram.md        # Visual workflow diagram
├── README.md                  # This file
└── email1.md                  # Sample email content (reference)
```

## Key Components Explained

### Pydantic Models

**OrderUpdateEmail**: Structures email-based order information
- Order number, supplier, shipper details
- Delivery dates and status
- Email metadata (sender, subject, body)

**OrderData**: Structures CSV-based shipment information
- Complete order details
- Shipment tracking (load ID, shipment ID)
- Logistics info (pallets, weight, volume, temperature)

### AI Functions (Tools)

**gmail_search_tool**: Searches Gmail using a query string and returns email snippets

**csv_order_search_tool**: Queries the CSV file by order number and returns structured data

### Agents

**gmail_agent**: Specialized agent that uses the Gmail search tool to find order-related emails

**csv_order_agent**: Specialized agent that uses the CSV search tool to find order details

**Magentic Manager**: Orchestrates the workflow between agents, plans execution, and synthesizes final results

### Workflow Orchestration

The `MagenticBuilder` creates a workflow that:
- Manages multiple specialized agents
- Routes queries to appropriate agents
- Handles up to 20 rounds of agent interaction
- Detects stalls (max 3) to prevent infinite loops
- Streams results in real-time
- Synthesizes a comprehensive final answer

## Usage Examples

### Example 1: Basic Order Query

```python
task = "What is the status of order 12345?"
await run_workflow()
```

### Example 2: Detailed Order Investigation

```python
task = "what is the status of order 12345? Check both email updates and the order CSV data."
await run_workflow()
```

### Example 3: Query Specific Fields

The agents will automatically extract:
- Current order status
- Delivery estimates
- Shipment tracking information
- Email communications history
- Supplier and shipper details

## Troubleshooting

### Gmail API Authentication Issues

**Problem**: Browser doesn't open for OAuth flow
- **Solution**: Ensure you're running in an environment that can open a browser. If on a remote server, consider using `flow.run_console()` instead of `flow.run_local_server()`

**Problem**: "Access blocked: This app's request is invalid"
- **Solution**: Verify your OAuth consent screen is properly configured and your email is added as a test user

### Azure OpenAI Issues

**Problem**: Authentication errors with Azure OpenAI
- **Solution**: Run `az login` and verify you have proper role assignments on your Azure OpenAI resource

**Problem**: Model not found
- **Solution**: Ensure you have a model deployed in your Azure OpenAI resource

### CSV Data Issues

**Problem**: Order not found in CSV
- **Solution**: Verify the order number format matches the CSV (e.g., "4500123456" vs "12345")

**Problem**: Column name errors
## Workflow Visualization

See the [Workflow Diagram](workflow_diagram.md) for a detailed, color-coded Mermaid diagram showing:
- Complete data flow between all components
- Multi-agent orchestration with the Magentic Manager
- Gmail API and Azure OpenAI authentication flows
- Pydantic model structures and field definitions
- Parallel processing of email and CSV data sources
- Result synthesis and response generation

The diagram uses color coding to distinguish between:
- User interactions (light blue)
- AI agents and orchestration (purple, blue, pink)
- Data sources and APIs (orange, coral)
- Data processing and parsing (cyan, yellow)
- Structured models and results (lavender, green)
- Authentication flows
- Pydantic model structures

## Advanced Configuration

### Adjusting Workflow Parameters

You can modify the Magentic workflow settings:

```python
workflow = (
    MagenticBuilder()
    .participants(gmail_search=gmmail_agent, csv_search=csv_agent)
    .with_standard_manager(
        chat_client=manager_chat_client,
        max_round_count=20,      # Increase for more complex queries
        max_stall_count=3,       # Adjust stall detection sensitivity
    )
    .with_plan_review(enable=True)  # Enable plan review for debugging
    .build()
)
```

### Adding More Data Sources

To add additional data sources:
1. Create a new AI function with `@ai_function` decorator
2. Create a specialized agent that uses your new tool
3. Add the agent to the `MagenticBuilder.participants()`

### Customizing Response Format

Modify the Pydantic models to include additional fields or change validation rules:

```python
class OrderUpdateEmail(BaseModel):
    # Add your custom fields here
    custom_field: str = Field(..., description="Description here")
```

## Security Considerations

- **Never commit** `client_secret_*.json` or `token.json` to version control
- Add these files to `.gitignore`
- Rotate OAuth credentials regularly
- Use Azure Managed Identity in production instead of CLI credentials
- Implement proper access controls on CSV data files

## Next Steps

- Extend the demo to query additional data sources (databases, APIs)
- Add more specialized agents for different types of queries
- Implement error handling and retry logic
- Build a web interface using FastAPI or Streamlit
- Deploy the agents as a production service

## Resources

- [Microsoft Agent Framework Documentation](https://learn.microsoft.com/en-us/python/api/agent-framework-core/)
- [Gmail API Python Documentation](https://developers.google.com/gmail/api/quickstart/python)
- [Azure OpenAI Documentation](https://learn.microsoft.com/en-us/azure/ai-services/openai/)
- [Pydantic Documentation](https://docs.pydantic.dev/)

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review the Jupyter notebook comments for inline documentation
3. Consult the official documentation links provided

## License

This is a demonstration project. Ensure you comply with:
- Google API Terms of Service
- Azure OpenAI Service Terms
- Any applicable data privacy regulations for order/email data
