# Primary Connect Order Tracking Workflow

```mermaid
flowchart TD
    Start[User Query: Order Status]:::userNode --> Task[Task: Find status of Order]:::taskNode
    
    Task --> Manager[Magentic Manager Agent]:::managerNode
    
    Manager -->|Plan & Route| GmailAgent[Gmail Search Agent]:::agentNode
    Manager -->|Plan & Route| CSVAgent[CSV Order Search Agent]:::agentNode
    
    GmailAgent -->|Uses Tool| GmailTool[gmail_search_tool]:::toolNode
    CSVAgent -->|Uses Tool| CSVTool[csv_order_search_tool]:::toolNode
    
    GmailTool -->|API Call| GmailAPI[Gmail API Service]:::apiNode
    CSVTool -->|Query| CSVFile[mypc_data.csv]:::dataNode
    
    GmailAPI -->|Returns| EmailData[Email Data: comment snippets, sender, dates]:::rawDataNode
    CSVFile -->|Returns| OrderData[Order Data: comment status, delivery, shipping info]:::rawDataNode
    
    EmailData --> EmailParse[Parse to OrderUpdateEmail Pydantic Model]:::parseNode
    OrderData --> OrderParse[Parse to OrderData Pydantic Model]:::parseNode
    
    EmailParse -->|Structured Data| EmailFields[Fields: comment order_number, supplier_name, shipper_name, estimated_delivery_date, current_status, email_summary, email_received_date, email_sender, email_subject, email_body_snippet, full_email_body]:::modelNode
    
    OrderParse -->|Structured Data| OrderFields[Fields: comment order_number, order_date, supplier_code, supplier_name, customer_site, site_name, pickup_date, delivery_window_start, delivery_window_end, transport_mode, load_id, shipment_id, status, status_timestamp, pallets, cartons, weight_kg, volume_m3, temperature_zone, reference_notes]:::modelNode
    
    EmailFields --> Manager
    OrderFields --> Manager
    
    Manager -->|Synthesizes| FinalResult[Final Result: comment Combined order status from both sources]:::resultNode
    
    FinalResult --> UserResponse[User Response: comment Comprehensive order status with email context and shipment details]:::responseNode
    
    subgraph Authentication
        CredFile[client_secret JSON file]:::authNode --> OAuth[OAuth Flow]:::authNode
        OAuth --> Token[token.json]:::authNode
        Token --> GmailAPI
    end
    
    subgraph Azure OpenAI
        AzureClient[AzureOpenAIChatClient]:::azureNode --> GmailAgent
        AzureClient --> CSVAgent
        AzureClient --> Manager
        AzureCred[AzureCliCredential]:::azureNode --> AzureClient
    end
    
    subgraph Workflow Orchestration
        Builder[MagenticBuilder]:::orchestrateNode --> Workflow[Magentic Workflow]:::orchestrateNode
        Workflow --> Manager
        Workflow -->|max_round_count: 20| RoundLimit[Round Limit]:::orchestrateNode
        Workflow -->|max_stall_count: 3| StallLimit[Stall Limit]:::orchestrateNode
    end
    
    classDef userNode fill:#e1f5ff,stroke:#01579b,stroke-width:3px,color:#000
    classDef taskNode fill:#fff9c4,stroke:#f57f17,stroke-width:2px,color:#000
    classDef managerNode fill:#ce93d8,stroke:#6a1b9a,stroke-width:3px,color:#000
    classDef agentNode fill:#90caf9,stroke:#1565c0,stroke-width:2px,color:#000
    classDef toolNode fill:#a5d6a7,stroke:#2e7d32,stroke-width:2px,color:#000
    classDef apiNode fill:#ffcc80,stroke:#e65100,stroke-width:2px,color:#000
    classDef dataNode fill:#ffab91,stroke:#bf360c,stroke-width:2px,color:#000
    classDef rawDataNode fill:#fff59d,stroke:#f9a825,stroke-width:2px,color:#000
    classDef parseNode fill:#80deea,stroke:#00838f,stroke-width:2px,color:#000
    classDef modelNode fill:#b39ddb,stroke:#4527a0,stroke-width:2px,color:#000
    classDef resultNode fill:#81c784,stroke:#388e3c,stroke-width:3px,color:#000
    classDef responseNode fill:#66bb6a,stroke:#1b5e20,stroke-width:3px,color:#000
    classDef authNode fill:#ffccbc,stroke:#d84315,stroke-width:2px,color:#000
    classDef azureNode fill:#9fa8da,stroke:#283593,stroke-width:2px,color:#000
    classDef orchestrateNode fill:#f48fb1,stroke:#880e4f,stroke-width:2px,color:#000
```

## Workflow Description

This diagram illustrates the Primary Connect order tracking system that:

1. **Receives User Query**: User asks about order status by order number
2. **Manager Orchestration**: Magentic Manager coordinates between specialized agents
3. **Parallel Data Sources**:
   - Gmail Agent searches email for order updates and communications
   - CSV Agent queries local order database for shipment details
4. **Data Retrieval**:
   - Gmail API fetches relevant emails with order updates
   - CSV file provides structured order and shipment data
5. **Structured Parsing**: Results are parsed into Pydantic models for validation
6. **Synthesis**: Manager combines both data sources into a comprehensive response
7. **Authentication**: Gmail uses OAuth 2.0 flow with stored credentials
8. **AI Integration**: Azure OpenAI powers all agents using AzureCliCredential
9. **Workflow Management**: MagenticBuilder orchestrates multi-agent collaboration

## Key Components

- **Gmail Search Tool**: Searches emails for order communications
- **CSV Order Search Tool**: Queries mypc_data.csv for shipment data
- **OrderUpdateEmail Model**: Structures email-based order information
- **OrderData Model**: Structures CSV-based shipment information
- **Magentic Workflow**: Coordinates agent collaboration with round and stall limits
```
