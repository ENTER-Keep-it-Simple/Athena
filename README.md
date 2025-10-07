# Athena 3.0 - An Agent-of-Agents AI Tutor in n8n

Athena is a sophisticated orchestration agent built with n8n that manages a team of specialist AI tutors. When a user asks a question, Athena routes the request to the correct expert—Alkemy for chemistry, Darwin for biology, Newt for physics, or Phi for mathematics—and synthesizes their responses into a single, coherent, and context-aware reply.

This project is designed to run entirely with local models using Ollama, ensuring data privacy and full control over the AI stack.

## Features

-   **Agent Orchestration:** A primary agent (`Athena`) analyzes user queries and dispatches them to the appropriate specialist sub-agent.
-   **Multi-Agent Collaboration:** For interdisciplinary questions (e.g., biochemistry), Athena can invoke multiple agents in parallel and synthesize their outputs.
-   **Retrieval-Augmented Generation (RAG):** Each specialist agent is connected to a dedicated knowledge base in a Qdrant vector store, ensuring answers are grounded in trusted data.
-   **Conversational Memory:** Utilizes Redis to maintain conversation history for each user, allowing for follow-up questions and context-aware interactions.
-   **Local LLM Processing:** Leverages local models via Ollama for both chat logic (`gpt-oss:latest`) and embeddings (`mistral:latest`).
-   **PDF Ingestion:** Users can upload PDF documents along with their questions. The system extracts the text and uses it as context for the agents.
-   **Advanced Input Normalization:** A powerful custom code node cleans and standardizes user input, detects the topic domain, and prepares metadata for the agent.

## Prerequisites

Before you begin, ensure you have the following services running and accessible from your n8n instance:

1.  **n8n:** A self-hosted instance of n8n.
2.  **Ollama:** An instance of [Ollama](https://ollama.com/) with the following models pulled:
    -   `ollama pull gpt-oss:latest`
    -   `ollama pull mistral:latest`
3.  **Redis:** A running [Redis](https://redis.io/) server for conversational memory.
4.  **Qdrant:** A running [Qdrant](https://qdrant.tech/) vector database instance. You must also create the collections used by the agents:
    -   `Chemistry_md_kb_sp`
    -   `Biology_md_kb_sp`
    -   `Physics_md_kb_sp`
    -   `Mathematics_md_kb_sp`

## Setup Instructions

1.  **Clone the Repository:**
    ```bash
    git clone <[repository-url](https://github.com/ENTER-Keep-it-Simple/Athena.git)>
    cd athena-3.0
    ```

2.  **Configure n8n Credentials:**
    Log in to your n8n instance and navigate to the **Credentials** section. Create the following credentials, pointing them to your self-hosted services:
    -   **Ollama API:** For connecting to your Ollama server.
    -   **Redis:** For connecting to your Redis server.
    -   **Qdrant API:** For connecting to your Qdrant database.

3.  **Import the Workflow:**
    -   In your n8n dashboard, click **Import from file**.
    -   Select the `workflows/athena-orchestrator.json` file from this repository.

4.  **Connect Credentials to Nodes:**
    -   Open the imported "Athena 3.0" workflow.
    -   For each of the nodes listed below, select the credential you created in Step 2 from the dropdown menu:
        -   **Ollama Chat Model1:** Connect your Ollama API credential.
        -   **Redis Chat Memory1:** Connect your Redis credential.
        -   **All Qdrant Vector Store Nodes (x4):** Connect your Qdrant API credential.
        -   **Embeddings Ollama3:** Connect your Ollama API credential.

5.  **Activate the Workflow:**
    -   Save the workflow.
    -   Toggle the **Active** switch to ON in the top-right corner.

Your Athena agent is now live and ready to answer questions via the Chat or Webhook trigger.

## How It Works

The workflow operates in a sequential, orchestrated manner:

1.  **Input & Routing:** The `When chat message received` trigger captures user input. An `IF` node checks if a PDF file was included and routes the data accordingly.
2.  **Normalization:** The `Normalize Input` code node cleans the text, extracts metadata (like a session ID and a domain hint), and combines the user's query with any text extracted from a PDF.
3.  **Orchestration:** The main `Athena` agent receives the normalized input. Its system prompt instructs it to act as an orchestrator, calling the specialist agents as tools.
4.  **Tool Execution:** Based on Athena's decision, one or more specialist agents (`Alkemy`, `Darwin`, `Newt`, `Phi`) are invoked. Each specialist agent uses its own prompt, memory, and RAG-tool connected to a specific Qdrant collection to generate an expert response.
5.  **Synthesis & Response:** Athena receives the output from the specialist tool(s) and synthesizes the information into a final, user-facing answer, which is then sent back through the chat interface.
