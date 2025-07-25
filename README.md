# Agentic RAG Using DeepSeek

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python: 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![LangChain: Latest](https://img.shields.io/badge/LangChain-Latest-green.svg)](https://python.langchain.com/)

## Overview

This repository implements an Agentic Retrieval-Augmented Generation (RAG) system that intelligently combines document retrieval from a vector database with real-time web search capabilities. Unlike traditional RAG systems that rely solely on pre-indexed documents, this implementation can dynamically choose the appropriate information source based on the query, ensuring both accuracy and up-to-date responses.

The system first attempts to answer queries using locally stored documents (in this case, a financial report), falling back to web search only when necessary. This approach optimizes for both efficiency and information quality.

## Features

- **Dual Information Sources**: Combines vector database retrieval with web search
- **Intelligent Tool Selection**: Prioritizes local documents before using web search
- **PDF Processing**: Converts PDF documents into searchable vector embeddings
- **Semantic Search**: Uses modern embedding models for accurate retrieval
- **Transparent Reasoning**: Provides visibility into the agent's decision-making process
- **Cloud-ready**: Utilizes Qdrant cloud for vector storage

## Architecture

The system follows this high-level architecture:

1. **Document Processing**: PDF documents are loaded, split into chunks, and converted to vector embeddings
2. **Vector Storage**: Document vectors are stored in a Qdrant database for efficient similarity search
3. **Tool Definition**: Two search tools are created - vector store search and web search
4. **Agent Framework**: An agent is constructed to decide which tool to use based on the query
5. **Response Generation**: The final answer is generated by combining retrieved information with the language model's capabilities

## Prerequisites

- Python 3.12+
- API keys for:
  - Qdrant (vector database)
  - Tavily (web search)
  - DeepSeek API (language model)

## Installation

```bash
# Clone the repository
git clone https://github.com/carloderossi/agentic-rag.git
cd agentic-rag

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows, use: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
export QDRANT_API_KEY="your_qdrant_api_key"
export TAVILY_API_KEY="your_tavily_api_key"
export DEEPSEEK_API_KEY="your_deepseek_api_key"
```

## Configuration

Before running the system, you'll need to:

1. Create a Qdrant collection (or use an existing one)
2. Update the Qdrant URL in the code to point to your collection
3. Ensure your API keys are properly set

## Usage

### Basic Example

```python
from agentic_rag import AgenticRAG

# Initialize the system
rag_system = AgenticRAG(
    documents_path="path/to/your/documents",
    qdrant_url="your_qdrant_url",
    collection_name="your_collection_name"
)

# Ask a question
response = rag_system.query("What were Alphabet's total revenues in Q3 2024?")
print(response)
```

### Advanced Usage

For more control over the system, you can customize various components:

```python
from agentic_rag import AgenticRAG
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_deepseek import ChatDeepSeek

# Custom embedding model
custom_embeddings = HuggingFaceEmbeddings(
    model_name="BAAI/bge-large-en-v1.5", 
    encode_kwargs={"normalize_embeddings": True}
)

# Custom language model
custom_llm = ChatDeepSeek(model="deepseek-reasoner-plus")

# Initialize system with custom components
rag_system = AgenticRAG(
    documents_path="path/to/your/documents",
    qdrant_url="your_qdrant_url",
    collection_name="your_collection_name",
    embeddings=custom_embeddings,
    llm=custom_llm,
    chunk_size=1000,
    chunk_overlap=100,
    verbose=True
)

# Ask a complex question
response = rag_system.query("Compare Alphabet's revenue growth in 2024 to previous years and explain key factors.")
print(response)
```

## Example Output

When querying about information contained in the loaded documents:

```
> Query: Total Google (Alphabet) revenues Q3-2024

Thought: I need to search for information about Google/Alphabet's revenues for Q3 2024.
Action: VectorStoreSearch
Action Input: "Alphabet revenue Q3 2024"

Observation: According to the Q3 2024 financial report, Alphabet reported total revenues of $86.31 billion for the third quarter of 2024, representing a 15% increase year-over-year.

Final Answer: Based on Alphabet's Q3 2024 financial report, the company reported total revenues of $86.31 billion, which represents a 15% increase compared to the same quarter in the previous year.
```

When querying about information that requires web search:

```
> Query: Alphabet total revenue for 2024 and beginning 2025?

Thought: I should first check if this information is in our vector store.
Action: VectorStoreSearch
Action Input: "Alphabet total revenue 2024 and 2025"

Observation: I don't have complete information for full-year 2024 or 2025 revenue in the vector store. The document only contains data up to Q3 2024.

Thought: Since I don't have the complete information in the vector store, I should search the web.
Action: WebSearch
Action Input: "Alphabet total annual revenue 2024 full year and Q1 2025"

Observation: [Detailed web search results]

Final Answer: According to the most recent financial reports and market data, Alphabet reported total revenue of approximately $348.3 billion for the full fiscal year 2024, representing an 11.8% increase from 2023. For Q1 2025, Alphabet reported revenue of $92.4 billion, showing continued growth of 13.6% year-over-year. These results were driven primarily by strong performance in Google Search and YouTube advertising.
```



## Performance Considerations

- **Document Size**: Large documents should be split into appropriate chunks for optimal retrieval
- **Embedding Model Selection**: Larger embedding models provide better semantic understanding but require more resources
- **Query Complexity**: Complex queries may require multiple retrieval steps for complete answers
- **API Costs**: Web search and LLM APIs have associated costs; use the vector store when possible for efficiency

## Extending the System

You can extend this system in several ways:

1. **Support for Additional Document Types**: Add loaders for Word, Excel, HTML, etc.
2. **Multiple Vector Stores**: Implement fallback or specialized vector stores
3. **Result Reranking**: Add a reranking step to improve retrieval precision
4. **Conversation Memory**: Implement persistent memory for multi-turn conversations
5. **Custom Agents**: Create specialized agents for different domains or tasks

## Troubleshooting

### Common Issues

- **Memory Errors**: If you encounter memory issues, try reducing the chunk size or processing documents in batches
- **Rate Limiting**: APIs may have rate limits; implement exponential backoff for retries
- **Vector Store Connectivity**: Ensure proper authentication and network connectivity to Qdrant

### Debugging

Set the `verbose` parameter to `True` when initializing the system to see detailed logs of the agent's reasoning process and API calls.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- [LangChain](https://python.langchain.com/) for the agent framework
- [Qdrant](https://qdrant.tech/) for vector storage
- [Tavily](https://tavily.com/) for web search capabilities
- [DeepSeek](https://deepseek.ai/) for the language model
- [BAAI](https://www.baai.ac.cn/english.html) for the BGE embedding models

## Citation

If you use this code in your research or project, please cite:

```bibtex
@software{agentic_rag_system,
  author = {Carlo De Rossi},
  title = {Agentic RAG System},
  year = {2025},
  url = {https://github.com/carloderossi/agentic-rag}
}
```
