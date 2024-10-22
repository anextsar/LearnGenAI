# LearnGenAI



---

# Azure AI Document Indexing and Query API

This repository contains a Python-based Azure Functions application that handles two core functionalities:
1. **IndexDocuments**: Accepts a document link (PDF or DOCX), extracts text, splits the text into chunks, generates embeddings using Azure OpenAI, and indexes the chunks into Azure Cognitive Search.
2. **QueryKnowledgeBase**: Accepts a search query, performs a vectorized search on indexed document content, and returns relevant information from the documents using Azure OpenAI.

## Features
- **Document Ingestion**: Fetch documents from a URL (supports PDF and DOCX formats).
- **Text Splitting**: Automatically splits long document content into manageable chunks for efficient indexing.
- **Embeddings Generation**: Uses Azure OpenAI to generate text embeddings for each chunk of document content.
- **Azure Cognitive Search Integration**: Stores document embeddings in Azure Cognitive Search for efficient vector search.
- **Natural Language Query**: Uses a combination of Azure OpenAI and Cognitive Search to perform a vectorized search and answer user queries based on the indexed content.
- **Chat Completions**: Generates conversational responses for user queries based on document content.

## Prerequisites
Before running this project, ensure you have the following:
- An **Azure account** with the following services:
  - **Azure OpenAI** with access to the embeddings and GPT models.
  - **Azure Cognitive Search** for storing and querying indexed document embeddings.
  - **Azure Blob Storage** (optional) for document storage.
- **Azure Functions** set up in your Azure portal or using the Azure Functions Core Tools locally.
- **Python 3.8+** installed on your local machine.

## Setup

### 1. Clone the Repository
Clone this repository to your local machine:

```bash
git clone https://github.com/anextsar/azure-ai-index-query.git
cd azure-ai-index-query
```

### 2. Install Dependencies
This project requires several Python libraries. Install them using `pip`:

```bash
pip install -r requirements.txt
```

The key dependencies include:
- `azure-functions`: For creating HTTP triggers in Azure Functions.
- `PyPDF2` and `python-docx`: For handling PDF and DOCX document formats.
- `langchain` and `langchain_openai`: For text splitting and embedding generation.
- `azure-search-documents`: For working with Azure Cognitive Search.
- `requests`: For downloading documents from URLs.

### 3. Set Up Environment Variables
The application uses several environment variables. Add the following environment variables to your Azure Functions environment or a `.env` file for local development:

- `AZURE_OPENAI_API_KEY`: Your Azure OpenAI API key.
- `AZURE_OPENAI_ENDPOINT`: The endpoint for your Azure OpenAI resource.
- `embedding_model`: The embedding model name (e.g., `text-embedding-ada-002`).
- `gpt_model`: The GPT model for generating conversational responses (e.g., `gpt-35-turbo`).
- `Azure_search_endpoint`: Your Azure Cognitive Search endpoint.
- `Azure_search_key`: Your Azure Cognitive Search API key.
- `System_prompt`: A system-level prompt for controlling chat behavior (e.g., prevent small talk).

### 4. Azure OpenAI Configuration
Ensure that your Azure OpenAI service has the correct models deployed:
- Embedding model for document indexing (e.g., `text-embedding-ada-002`).
- GPT model for chat completions (e.g., `gpt-35-turbo`).

### 5. Azure Cognitive Search Setup
Create an index in Azure Cognitive Search using the following fields:
```python
fields = [
    SearchableField(
        name="id",
        type=SearchFieldDataType.String,
        key=True,
        filterable=True
    ),
    SearchableField(
        name="parent_id",
        type=SearchFieldDataType.String,
        searchable=True,
        filterable=True
    ),
    SearchableField(
        name="content",
        type=SearchFieldDataType.String,
        searchable=True,
        analyzer_name="en.microsoft"
    ),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        vector_search_dimensions=1536,
        vector_search_profile_name="myHnswProfile"
    ),
    SearchableField(
        name="metadata",
        type=SearchFieldDataType.String,
        searchable=True
    ),
    SearchableField(
        name="title",
        type=SearchFieldDataType.String,
        searchable=True
    ),
    SimpleField(
        name="source",
        type=SearchFieldDataType.String,
        filterable=True
    )
]
```

Refer to [Azure Cognitive Search documentation](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search) for more details.

## How It Works

### API Endpoints

#### 1. `/http_trigger`
- **Method**: `POST`
- **Description**: This endpoint accepts a document link (`doc_link`) via a POST request. It downloads the document, extracts the text (supports both PDF and DOCX), splits it into chunks, generates embeddings, and indexes the document chunks in Azure Cognitive Search.
  
  **Example Request**:
  ```json
  {
    "doc_link": "https://example.com/document.pdf"
  }
  ```
  
  **Response**:
  - Status: 200 (COMPLETED) if the document is successfully indexed.
  - Status: 400 or 500 if there are errors during processing.

#### 2. `/QueryKnowledgeBase`
- **Method**: `GET`
- **Description**: This endpoint accepts two query parameters: `query` and `index_name`. It performs a vectorized search using the provided query, retrieves the most relevant chunks, and generates a response using Azure OpenAI's GPT model.
  
  **Example Request**:
  ```
  GET /QueryKnowledgeBase?query=What is the document about?&index_name=3log-index
  ```
  
  **Response**:
  - Status: 200 if the response is generated successfully.
  - Status: 500 if any required parameters are missing or there are errors.

### Error Handling
- Proper error handling is implemented for issues like missing or invalid request parameters, failed document downloads, and document parsing errors.
  
### Text Splitting and Embedding
- The `CharacterTextSplitter` from `langchain` is used to split long document content into smaller chunks. This ensures that each chunk is processed efficiently and can be indexed in Azure Cognitive Search.
- Embeddings for each chunk are generated using the Azure OpenAI `embeddings.create` method, and the chunks are then stored in the vector store for later querying.

## Running the Application Locally
1. Install Azure Functions Core Tools if you haven't already:
   ```bash
   npm install -g azure-functions-core-tools@4 --unsafe-perm true
   ```
   
2. Start the Azure Functions application:
   ```bash
   func start
   ```

3. The endpoints will be available at `http://localhost:7071/api/http_trigger` and `http://localhost:7071/api/QueryKnowledgeBase`.

## Deployment to Azure
To deploy this project to Azure:
1. Create an Azure Function App.
2. Set the required environment variables in the Function App settings.
3. Deploy the project using Azure Functions Core Tools:
   ```bash
   func azure functionapp publish <APP_NAME>
   ```

## Contributing
Contributions are welcome! Feel free to open issues or submit pull requests with improvements.

---

