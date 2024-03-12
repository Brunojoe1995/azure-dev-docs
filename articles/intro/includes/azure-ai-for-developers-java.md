## Azure AI reference templates

Azure AI reference templates provide you with well-maintained, easy to deploy reference implementations. These ensure a high-quality starting point for your intelligent applications. The end-to-end solutions provide popular, comprehensive reference applications. The building blocks are smaller-scale samples that focus on specific scenarios and tasks.

### End-to-end solutions

|Link|Description|
|---|---|
|[Get started with the Java enterprise chat sample using RAG](../../java/quickstarts/get-started-app-chat-template.md)|An article that walks you through deploying and using the [Enterprise chat app sample for Java](https://github.com/Azure-Samples/azure-search-openai-demo-java). This sample is a complete end-to-end solution demonstrating the [Retrieval-Augmented Generation (RAG) pattern](/azure/search/retrieval-augmented-generation-overview) running in Azure, using Azure AI Search for retrieval and Azure OpenAI large language models to power ChatGPT-style and Q&A experiences.|

* [Demo video](https://aka.ms/azai/java/video)

### Building blocks

|Link|Description|
|---|---|
|[Build a chat app with Azure OpenAI (Python)](https://github.com/Azure-Samples/chatgpt-quickstart/blob/main/README.md)|A simple Python Quart app that streams responses from ChatGPT to an HTML/JS frontend using JSON Lines over a ReadableStream. (The Python code is provided as a reference and could be adapted to Java.)|
|[Build a LangChain with Azure OpenAI (Python)](https://github.com/Azure-Samples/function-python-ai-langchain)|A sample shows how to take a human prompt as HTTP Get or Post input, calculates the completions using chains of human input and templates. This is a starting point that can be used for more sophisticated chains. (The Python code is provided as a reference and could be adapted to Java.)|
|[Build a ChatGPT Plugin with Azure Container Apps (Python)](https://github.com/Azure-Samples/openai-plugin-fastapi/blob/main/README.md)|A sample for creating ChatGPT Plugin using GitHub Codespaces, VS Code, and Azure. The sample includes templates to deploy the plugin to Azure Container Apps using the Azure Developer CLI. (The Python code is provided as a reference and could be adapted to Java.)|
|[Azure AI Java Template Gallery](https://azure.github.io/awesome-azd/?tags=ai&tags=java)|For the full list of Azure AI templates, visit our gallery. All app templates in our gallery can be spun up and deployed using a single command: _azd up_.|
|[Smart load balancing with Azure Container Apps](../../java/quickstarts/get-started-app-chat-scaling-with-azure-container-apps.md)|This [sample solution](https://github.com/Azure-Samples/openai-aca-lb) is built using the high-performance [YARP C# reverse-proxy framework](https://github.com/microsoft/reverse-proxy) from Microsoft. However, you don't need to understand C# to use it, you can just build the provided Docker image. This is an alternative solution to the [API Management OpenAI smart load balancer](https://github.com/Azure-Samples/openai-apim-lb/), with the same logic.|
|[Smart load balancing with Azure API Management](https://github.com/Azure-Samples/openai-apim-lb/)|The enterprise solution shows how to create an Azure API Management Policy to seamlessly expose a single endpoint to your applications while keeping an efficient logic to consume two or more OpenAI or any API backends based on availability and priority.|

## Azure OpenAI

### End-to-end solutions

|Link|Description|
|---|---|
|[Get started with the Java enterprise chat sample using RAG](../../java/quickstarts/get-started-app-chat-template.md)|An article that walks you through deploying and using the [Enterprise chat app sample for Java](https://github.com/Azure-Samples/azure-search-openai-demo-java). This sample is a complete end-to-end solution demonstrating the Retrieval-Augmented Generation (RAG) pattern running in Azure, using Azure AI Search for retrieval and Azure OpenAI large language models to power ChatGPT-style and Q&A experiences.|

### Building blocks

|Link|Description|
|---|---|
|[Vector Similarity Search with Azure Cache for Redis Enterprise (Python)](https://techcommunity.microsoft.com/t5/azure-developer-community-blog/vector-similarity-search-with-azure-cache-for-redis-enterprise/ba-p/3822059)|An article that walks you through using Azure Cache for Redis as a backend vector store for RAG scenarios. (The Python code is provided as a reference and could be adapted to Java.)|
|[OpenAI solutions with your own data using PostgreSQL (Python)](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/unlocking-the-power-of-open-ai-and-pgvector-with-azure/ba-p/3828539)|An article discussing how Azure Database for PostgreSQL Flexible Server and Azure Cosmos DB for PostgreSQL supports the pgvector extension, along with an overview, scenarios, etc. (The Python code is provided as a reference and could be adapted to Java.)|

### SDKs

|Package|Source code|Releases|Maven|
|---|---|---|---|
|**azure-ai-openai**|[Source code](https://aka.ms/oai/java/sdk)|[Releases](https://azure.github.io/azure-sdk/?search=openai)|[Maven package](https://aka.ms/oai/java/maven)|
|**azure-ai-openai-assistants**|[Source code](https://github.com/Azure/azure-sdk-for-java/tree/azure-ai-openai-assistants_1.0.0-beta.1/sdk/openai/azure-ai-openai-assistants/)|[Releases](https://central.sonatype.com/artifact/com.azure/azure-ai-openai-assistants/versions)|[Maven package](https://central.sonatype.com/artifact/com.azure/azure-ai-openai-assistants/)|

### Samples and guidance

|Link|Description|
|---|---|
|[Get started using GPT-35-Turbo and GPT-4](/azure/ai-services/openai/chatgpt-quickstart?pivots=programming-language-java&tabs=command-line)|An article that walks you through creating a chat completion sample.|
|[Completions](https://github.com/Azure/azure-sdk-for-java/blob/azure-ai-openai_1.0.0-beta.1/sdk/openai/azure-ai-openai/src/samples/java/com/azure/ai/openai/ChatbotSample.java)|A simple example demonstrating how to get completions for the provided prompt.|
|[Streaming Chat Completions](https://github.com/Azure/azure-sdk-for-java/blob/azure-ai-openai_1.0.0-beta.1/sdk/openai/azure-ai-openai/src/samples/java/com/azure/ai/openai/StreamingChatSample.java)|A simple example demonstrating how to use  streaming chat completions.|
|[Switch from OpenAI to Azure OpenAI](https://aka.ms/azai/oai-to-aoai)|An article with guidance on the small changes you need to make to your code in order to swap back and forth between OpenAI and the Azure OpenAI Service.|
|[OpenAI with Microsoft Entra ID Role based access control](/azure/ai-services/authentication?tabs=powershell#authenticate-with-azure-active-directory)|An article that looks at authentication using Microsoft Entra ID.|
|[OpenAI with Managed Identities](/azure/ai-services/openai/how-to/managed-identity)|An article detailing more complex security scenarios that require Azure role-based access control (Azure RBAC). This document covers how to authenticate to your OpenAI resource using Microsoft Entra ID.|
|[More Samples](https://aka.ms/oai/java/samples)|The Azure OpenAI service samples are a set of self-contained Java programs that demonstrate interacting with Azure OpenAI service using the client library. Each sample focuses on a specific scenario and can be executed independently.|
|[More guidance](/azure/ai-services/openai/)|The hub page for Azure OpenAI Service documentation.|

## Open Source integration

### SDKs

|Package|Source code|Releases|Maven|
|---|---|---|---|
|**langchain4j-azure-open-ai**|[Source code](https://github.com/langchain4j/langchain4j/tree/main/langchain4j-azure-open-ai)|[Releases](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-azure-open-ai/versions)|[Maven package](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-azure-open-ai)
|**langchain4j-azure-ai-search**|[Source code](https://github.com/langchain4j/langchain4j/tree/main/langchain4j-azure-ai-search)|[Releases](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-azure-ai-search/versions)|[Maven](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-azure-ai-search)|
|**langchain4j-document-loader-azure-storage-blob**|n/a|[Releases](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-document-loader-azure-storage-blob/versions)|[Maven](https://central.sonatype.com/artifact/dev.langchain4j/langchain4j-document-loader-azure-storage-blob/overview)|

## Other Azure AI services

### End-to-end solutions

|Link|Description|
|---|---|
|[Captioning and Call Center Transcription](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/scenarios)|A repo containing samples for captions and transcriptions in a call center scenario.|

### SDKs 

|Link|Description|
|---|---|
|[Azure AI Document Intelligence SDK](/azure/applied-ai-services/form-recognizer/sdk-preview)|Azure AI Document Intelligence (formerly Form Recognizer) is a cloud service that uses machine learning to analyze text and structured data from documents. The Document Intelligence software development kit (SDK) is a set of libraries and tools that enable you to easily integrate Document Intelligence models and capabilities into your applications.|

### Samples and guidance

|Link|Description|
|---|---|
|[Integrate Speech into your apps with Speech SDK Samples](https://github.com/Azure-Samples/cognitive-services-speech-sdk)|A collection of samples for the Azure Cognitive Services Speech SDK. Links to samples for speech recognition, translation, speech synthesis, and more.|
|[Extract structured data from forms, receipts, invoices, and cards using Form Recognizer in Java](https://github.com/Azure/azure-sdk-for-java/blob/main/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md#azure-form-recognizer-client-library-samples-for-java)|A collection of samples for the Azure.AI.FormRecognizer client library.|
|[Extract, classify, and understand text within documents using Text Analytics in Java](/java/api/overview/azure/ai-textanalytics-readme?view=azure-java-stable&preserve-view=true)|The client Library for Text Analytics. This is part of the [Azure AI Language](/azure/ai-services/language-service) service, which provides Natural Language Processing (NLP) features for understanding and analyzing text.|
|[Document Translation in Java](/azure/ai-services/translator/document-translation/quickstarts/document-translation-rest-api?pivots=programming-language-java)|A quickstart article that explains how to use Document Translation to translate a source document into a target language while preserving structure and text formatting.|
|[Analyze images](/azure/ai-services/computer-vision/sdk/overview-sdk)|Sample code and setup documents for the Microsoft Azure AI Vision SDK|
