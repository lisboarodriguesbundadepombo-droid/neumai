# neumai

neumai is a Python framework for building data pipelines that generate vector embeddings and store them for retrieval-augmented generation (RAG). Connect data sources, configure loading and chunking, and send embeddings with metadata to a vector database.

## Features

- Extract content from websites and PostgreSQL through data connectors.
- Load HTML or JSON content and split it into chunks before embedding.
- Select fields for embedding and retain other fields as retrieval metadata.
- Generate embeddings with OpenAI and write vectors to Weaviate.
- Run similarity searches through the pipeline interface.

## Installation

From the project root, create a virtual environment and install the local package:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install .
```

On Windows, activate the environment with `.venv\Scripts\activate` instead.

## Usage

This example loads a web page, chunks its content, generates OpenAI embeddings, and stores them in Weaviate. You need an OpenAI API key and a running Weaviate instance configured for the sink.

Set your credentials and destination class:

```bash
export OPENAI_API_KEY="your-openai-api-key"
export WEAVIATE_URL="your-weaviate-url"
export WEAVIATE_API_KEY="your-weaviate-api-key"
export WEAVIATE_CLASS_NAME="your-class-name"
```

Save the following as `example.py`:

```python
import os

from neumai.DataConnectors.WebsiteConnector import WebsiteConnector
from neumai.Shared.Selector import Selector
from neumai.Loaders.HTMLLoader import HTMLLoader
from neumai.Chunkers.RecursiveChunker import RecursiveChunker
from neumai.Sources.SourceConnector import SourceConnector
from neumai.EmbedConnectors import OpenAIEmbed
from neumai.SinkConnectors import WeaviateSink
from neumai.Pipelines import Pipeline

website_connector = WebsiteConnector(
    url="https://www.neum.ai/post/retrieval-augmented-generation-at-scale",
    selector=Selector(to_metadata=["url"]),
)

source = SourceConnector(
    data_connector=website_connector,
    loader=HTMLLoader(),
    chunker=RecursiveChunker(),
)

pipeline = Pipeline(
    sources=[source],
    embed=OpenAIEmbed(api_key=os.environ["OPENAI_API_KEY"]),
    sink=WeaviateSink(
        url=os.environ["WEAVIATE_URL"],
        api_key=os.environ["WEAVIATE_API_KEY"],
        class_name=os.environ["WEAVIATE_CLASS_NAME"],
    ),
)

pipeline.run()

results = pipeline.search(
    query="What are the challenges with scaling RAG?",
    number_of_results=3,
)

for result in results:
    print(result.metadata)
```

Run the pipeline and search:

```bash
python example.py
```

The script calls external services and may incur embedding or database charges.

## License

Licensed under Apache-2.0. This project is based on [NeumTry/NeumAI](https://github.com/NeumTry/NeumAI), which is also licensed under Apache-2.0. Retain the upstream license, applicable attribution notices, and any NOTICE file when redistributing derived code.
