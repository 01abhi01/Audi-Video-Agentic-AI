graph TD
    subgraph "User Space"
        User([User/Client Application])
    end

    subgraph "Master Agent Service (master_agent.py)"
        direction LR
        Master_Endpoint[/"/v1/message:send"/]
        Master_LLM{LLM as Router}
        Service_Discovery

        Master_Endpoint -- "2. Get available tools" --> Service_Discovery
        Service_Discovery -- "3. Fetch Agent Cards (A2A)" --> Worker_Discovery_Endpoint
        Master_Endpoint -- "4. Route Query to LLM" --> Master_LLM
        Master_LLM -- "5. Selects Appropriate Agent" --> Master_Proxy
        Master_Proxy
    end

    subgraph "Worker Agent Service (dbms_a2a_langgraph_agent.py)"
        direction LR
        Worker_Endpoint[/"/v1/message:send"/]
        Worker_Discovery_Endpoint[/"/.well-known/agent.json"/]
        LangGraph_Core((LangGraph Core Engine<br>langgraph_agent.py))

        Worker_Endpoint -- "7. Invoke Workflow" --> LangGraph_Core
    end

    subgraph "LangGraph Workflow (Internal to Worker)"
        direction TB
        LG_Plan{LLM Planner}
        LG_Execute
        LG_Format

        LangGraph_Core --> LG_Plan
        LG_Plan -- "8. Create Task List" --> LG_Execute
        LG_Execute -- "9. Execute Task" --> Tools
        LG_Execute -- "11. Loop or Finish" --> LG_Format
    end

    subgraph "Tool Layer (Dynamically Loaded)"
        Tools
    end

    subgraph "External Systems"
        DB
    end

    %% --- Connections ---
    User -- "1. User Query (HTTP POST)" --> Master_Endpoint
    Master_Proxy -- "6. Forward Request (A2A)" --> Worker_Endpoint
    Tools -- "10. Query DBMS" --> DB
    DB -- "Data" --> Tools
    LG_Format -- "12. Final Formatted Response" --> Worker_Endpoint
    Worker_Endpoint -- "13. Return Result (A2A)" --> Master_Proxy
    Master_Proxy -- "14. Forward Final Response" --> User

    %% --- Styling ---
    classDef agent fill:#cce5ff,stroke:#333,stroke-width:2px;
    classDef workflow fill:#d5e8d4,stroke:#333,stroke-width:2px;
    classDef tools fill:#f8cecc,stroke:#333,stroke-width:2px;
    classDef external fill:#e1d5e7,stroke:#333,stroke-width:2px;

    class Master_Endpoint,Master_LLM,Service_Discovery,Master_Proxy agent
    class Worker_Endpoint,Worker_Discovery_Endpoint,LangGraph_Core agent
    class LG_Plan,LG_Execute,LG_Format workflow
    class Tools tools
    class DB,User external
