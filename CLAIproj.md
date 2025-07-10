
```mermaid
sequenceDiagram
    actor User
    participant LINE_App as LINE App (CarbonLead)
    participant LINE_API as LINE Messaging API
    box Subcontractor Scope
        participant API_Gateway as Backend API Gateway
        participant Orchestrator as Agent Orchestrator (ADK)
        participant SalesAgent as Sales Agent
        participant SupportAgent as Support Agent
        participant ExpertAgent as Expert Agent
        participant Tools as Agent Tools (DB Access, OCR)
        participant KnowledgeBase as Knowledge Base (Postgres/pgvector)
    end
    User->>LINE_App: Sends message (e.g., "What are your prices?")
    LINE_App->>LINE_API: Forwards message
    LINE_API->>API_Gateway: POST /v1/chat/conversation
    API_Gateway->>Orchestrator: Routes authenticated request
    Orchestrator->>Orchestrator: Analyzes intent ("Sales Query")
    Orchestrator->>SalesAgent: Delegates task: "Answer pricing question"
    SalesAgent->>KnowledgeBase: Performs RAG query for pricing info
    KnowledgeBase-->>SalesAgent: Returns relevant context
    SalesAgent->>SalesAgent: Generates response using LLM
    SalesAgent-->>Orchestrator: Returns generated response
    Orchestrator-->>API_Gateway: Forwards final response
    API_Gateway-->>LINE_API: Sends response
    LINE_API-->>LINE_App: Delivers message
    LINE_App-->>User: Displays response
    User->>LINE_App: "Show my last invoice" (with attachment)
    LINE_App->>LINE_API: Forwards message and attachment
    LINE_API->>API_Gateway: POST /v1/chat/conversation
    API_Gateway->>Orchestrator: Routes request
    Orchestrator->>Orchestrator: Analyzes intent ("Support Query" / "Expert Query")
    alt Support Query
        Orchestrator->>SupportAgent: Delegate: "Fetch last invoice"
        SupportAgent->>Tools: Calls secure DB access tool
        Tools->>KnowledgeBase: SELECT * FROM invoices...
        KnowledgeBase-->>Tools: Returns invoice data
        Tools-->>SupportAgent: Formats data
        SupportAgent->>SupportAgent: Generates response
        SupportAgent-->>Orchestrator: Returns response
    else Expert Query with Document
        Orchestrator->>ExpertAgent: Delegate: "Analyze this invoice for Scope 3"
        ExpertAgent->>Tools: Calls OCR pipeline tool
        Tools->>Tools: Extracts text from attachment
        Tools-->>ExpertAgent: Returns structured text data
        ExpertAgent->>KnowledgeBase: Performs RAG on TGO/ISO standards
        KnowledgeBase-->>ExpertAgent: Returns context on Scope 3
        ExpertAgent->>ExpertAgent: Generates expert advice
        ExpertAgent-->>Orchestrator: Returns response
    end
    Orchestrator-->>API_Gateway: Forwards final response
    API_Gateway-->>LINE_API: Sends response
    LINE_API-->>LINE_App: Delivers message
    LINE_App-->>User: Displays response
