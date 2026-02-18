# LangChain4j Guide Reference

> Activate this reference when the project uses LangChain4j (detected via `dev.langchain4j` or `quarkus-langchain4j` in `pom.xml`/`build.gradle`, or user request).
> Documentation: https://docs.langchain4j.dev/
> Getting Started: https://docs.langchain4j.dev/get-started
> Examples: https://github.com/langchain4j/langchain4j-examples

## What is LangChain4j?

LangChain4j is a Java framework for building LLM-powered applications. It provides:
- Unified API for multiple LLM providers (OpenAI, Anthropic, Ollama, Azure, etc.)
- AI Services (type-safe, declarative AI interactions)
- Function calling / Tools
- RAG (Retrieval-Augmented Generation)
- Chat memory management
- Structured output extraction

Current version: **1.11.0**

## AI Services

AI Services are the recommended way to interact with LLMs. Define an interface, and LangChain4j implements it.

> Guide: https://docs.langchain4j.dev/tutorials/ai-services

### Plain Java (no framework)

```java
interface Assistant {

    @SystemMessage("You are a helpful coding assistant specializing in Java.")
    String chat(String userMessage);
}

// Build it
Assistant assistant = AiServices.builder(Assistant.class)
    .chatLanguageModel(model)
    .chatMemory(MessageWindowChatMemory.withMaxMessages(20))
    .build();

String answer = assistant.chat("How do I use records in Java 25?");
```

### Quarkus Integration

> Guide: https://docs.langchain4j.dev/tutorials/quarkus-integration
> Quarkiverse docs: https://docs.quarkiverse.io/quarkus-langchain4j/dev/index.html

```java
@RegisterAiService
public interface CodingAssistant {

    @SystemMessage("You are a helpful coding assistant specializing in Java.")
    String chat(@UserMessage String userMessage);
}
```

```java
@ApplicationScoped
public class AssistantResource {

    @Inject
    CodingAssistant assistant;

    // Use the assistant via CDI injection
}
```

**Quarkus configuration** (`application.properties`):

```properties
# Model provider (e.g., OpenAI)
quarkus.langchain4j.openai.api-key=${OPENAI_API_KEY}
quarkus.langchain4j.openai.chat-model.model-name=gpt-4o
quarkus.langchain4j.openai.chat-model.temperature=0.7

# Or Ollama (local)
quarkus.langchain4j.ollama.chat-model.model-id=llama3
quarkus.langchain4j.ollama.base-url=http://localhost:11434

# Or Anthropic
quarkus.langchain4j.anthropic.api-key=${ANTHROPIC_API_KEY}
quarkus.langchain4j.anthropic.chat-model.model-name=claude-sonnet-4-20250514
```

> Quarkus model providers: https://docs.quarkiverse.io/quarkus-langchain4j/dev/model-providers.html

## Structured Output

Extract structured data from LLM responses using records:

```java
record SentimentAnalysis(
    String sentiment,      // "positive", "negative", "neutral"
    double confidence,     // 0.0 to 1.0
    String explanation
) {}

interface SentimentAnalyzer {

    @UserMessage("Analyze the sentiment of: {{text}}")
    SentimentAnalysis analyze(@V("text") String text);
}
```

> Guide: https://docs.langchain4j.dev/tutorials/ai-services#structured-outputs

## Tools (Function Calling)

Tools allow the LLM to call Java methods to perform actions or retrieve information.

> Guide: https://docs.langchain4j.dev/tutorials/tools

### Defining Tools

```java
@ApplicationScoped  // In Quarkus; omit for plain Java
public class ProductTools {

    private final ProductRepository repository;

    @Inject
    public ProductTools(ProductRepository repository) {
        this.repository = repository;
    }

    @Tool("Search products by name. Returns a list of matching products.")
    public List<ProductDTO> searchProducts(String query) {
        return repository.findByNameLike(query).stream()
            .map(ProductDTO::from)
            .toList();
    }

    @Tool("Get the current price of a product by its ID.")
    public BigDecimal getProductPrice(Long productId) {
        return repository.findByIdOptional(productId)
            .map(Product::getPrice)
            .orElseThrow(() -> new NotFoundException("Product not found: " + productId));
    }

    @Tool("Place an order for a product.")
    public OrderConfirmation placeOrder(Long productId, int quantity) {
        // order logic
        return new OrderConfirmation(orderId, "confirmed");
    }
}
```

### Connecting Tools to AI Services

**Plain Java:**

```java
Assistant assistant = AiServices.builder(Assistant.class)
    .chatLanguageModel(model)
    .tools(new ProductTools(repository))
    .chatMemory(MessageWindowChatMemory.withMaxMessages(20))
    .build();
```

**Quarkus** (automatic via CDI — tools are discovered by scanning):

```java
@RegisterAiService(tools = ProductTools.class)
public interface ShoppingAssistant {

    @SystemMessage("You are a shopping assistant. Use the available tools to help users.")
    String chat(@UserMessage String userMessage);
}
```

**Rules for Tools:**
- Use clear, descriptive `@Tool` descriptions — the LLM reads them to decide when to call
- Tool methods must have clear parameter names (use `@P("description")` if needed)
- Keep tools focused on a single action
- Handle errors gracefully; return error messages instead of throwing exceptions when possible
- Tools should be idempotent when feasible

## Chat Memory

> Guide: https://docs.langchain4j.dev/tutorials/chat-memory

### In-Memory (default)

```java
// Fixed window — keeps last N messages
ChatMemory memory = MessageWindowChatMemory.withMaxMessages(20);

// Token-based window — keeps messages within token budget
ChatMemory memory = TokenWindowChatMemory.withMaxTokens(4000, tokenizer);
```

### Per-User Memory (Quarkus)

```java
@RegisterAiService
@SessionScoped  // or use a custom memory provider
public interface UserAssistant {
    String chat(@MemoryId String sessionId, @UserMessage String message);
}
```

**Rules:**
- Always configure chat memory for conversational AI services
- Use `@MemoryId` to separate conversations per user/session
- Consider token limits — use `TokenWindowChatMemory` for large conversations
- For production, implement a persistent `ChatMemoryStore` (database-backed)

## RAG (Retrieval-Augmented Generation)

> Guide: https://docs.langchain4j.dev/tutorials/rag

### Basic RAG Pipeline

```java
// 1. Create an embedding model
EmbeddingModel embeddingModel = OpenAiEmbeddingModel.builder()
    .apiKey(apiKey)
    .modelName("text-embedding-3-small")
    .build();

// 2. Create an embedding store (in-memory for dev)
EmbeddingStore<TextSegment> embeddingStore = new InMemoryEmbeddingStore<>();

// 3. Ingest documents
EmbeddingStoreIngestor ingestor = EmbeddingStoreIngestor.builder()
    .documentSplitter(DocumentSplitters.recursive(500, 50))
    .embeddingModel(embeddingModel)
    .embeddingStore(embeddingStore)
    .build();

Document document = FileSystemDocumentLoader.loadDocument(Path.of("docs/manual.pdf"));
ingestor.ingest(document);

// 4. Create a content retriever
ContentRetriever retriever = EmbeddingStoreContentRetriever.builder()
    .embeddingStore(embeddingStore)
    .embeddingModel(embeddingModel)
    .maxResults(5)
    .minScore(0.7)
    .build();

// 5. Wire into AI Service
Assistant assistant = AiServices.builder(Assistant.class)
    .chatLanguageModel(model)
    .contentRetriever(retriever)
    .build();
```

### RAG in Quarkus

```java
@ApplicationScoped
public class DocumentIngestor {

    @Inject
    EmbeddingStore<TextSegment> store;

    @Inject
    EmbeddingModel embeddingModel;

    public void ingest(Path documentPath) {
        var ingestor = EmbeddingStoreIngestor.builder()
            .documentSplitter(DocumentSplitters.recursive(500, 50))
            .embeddingModel(embeddingModel)
            .embeddingStore(store)
            .build();

        var document = FileSystemDocumentLoader.loadDocument(documentPath);
        ingestor.ingest(document);
    }
}
```

**Quarkus embedding store configuration:**

```properties
# Using the pgvector extension
quarkus.langchain4j.pgvector.dimension=1536
quarkus.langchain4j.pgvector.table-name=embeddings
```

> Quarkus RAG guide: https://docs.quarkiverse.io/quarkus-langchain4j/dev/rag.html

## Prompt Engineering

### System Messages

```java
@RegisterAiService
public interface CodeReviewer {

    @SystemMessage("""
        You are an expert Java code reviewer.
        Focus on:
        - Code correctness and potential bugs
        - Performance implications
        - Java 25 best practices
        - Security vulnerabilities
        Respond with structured feedback.
        """)
    CodeReview review(@UserMessage String code);
}
```

### Prompt Templates

```java
interface Translator {

    @UserMessage("Translate the following text from {{source}} to {{target}}: {{text}}")
    String translate(@V("source") String sourceLang,
                     @V("target") String targetLang,
                     @V("text") String text);
}
```

### Few-Shot Examples in System Message

```java
@SystemMessage("""
    You classify support tickets. Examples:
    
    Input: "My password doesn't work"
    Output: {"category": "authentication", "priority": "high"}
    
    Input: "Can you add dark mode?"
    Output: {"category": "feature-request", "priority": "low"}
    """)
TicketClassification classify(@UserMessage String ticketText);
```

## Model Configuration (Plain Java)

```java
// OpenAI
ChatLanguageModel model = OpenAiChatModel.builder()
    .apiKey(System.getenv("OPENAI_API_KEY"))
    .modelName("gpt-4o")
    .temperature(0.7)
    .maxTokens(4096)
    .build();

// Anthropic
ChatLanguageModel model = AnthropicChatModel.builder()
    .apiKey(System.getenv("ANTHROPIC_API_KEY"))
    .modelName("claude-sonnet-4-20250514")
    .maxTokens(4096)
    .build();

// Ollama (local)
ChatLanguageModel model = OllamaChatModel.builder()
    .baseUrl("http://localhost:11434")
    .modelName("llama3")
    .build();
```

## Maven Dependencies

### Plain Java

```xml
<!-- Core -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.11.0</version>
</dependency>

<!-- OpenAI provider -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>1.11.0</version>
</dependency>

<!-- Anthropic provider -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-anthropic</artifactId>
    <version>1.11.0</version>
</dependency>

<!-- Ollama provider -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-ollama</artifactId>
    <version>1.11.0</version>
</dependency>
```

### Quarkus Extensions

```xml
<!-- Quarkus LangChain4j with OpenAI -->
<dependency>
    <groupId>io.quarkiverse.langchain4j</groupId>
    <artifactId>quarkus-langchain4j-openai</artifactId>
</dependency>

<!-- With Ollama -->
<dependency>
    <groupId>io.quarkiverse.langchain4j</groupId>
    <artifactId>quarkus-langchain4j-ollama</artifactId>
</dependency>

<!-- With pgvector for RAG -->
<dependency>
    <groupId>io.quarkiverse.langchain4j</groupId>
    <artifactId>quarkus-langchain4j-pgvector</artifactId>
</dependency>
```

> Quarkus extensions list: https://docs.quarkiverse.io/quarkus-langchain4j/dev/index.html

## Best Practices

- **Use AI Services** over low-level `ChatLanguageModel` directly — they provide type safety, memory, and tools integration
- **Use records** for structured output extraction
- **Keep @Tool descriptions clear** — the LLM uses them to decide when to call tools
- **Configure chat memory appropriately** — too small loses context, too large wastes tokens
- **Use prompt templates** (`@UserMessage` with `{{variables}}`) over string concatenation
- **Handle rate limits and errors** — configure retries and timeouts on the model builder
- **Secure API keys** — use environment variables or Quarkus config, never hardcode
- **Test AI interactions** — use mocked models or local Ollama for deterministic tests
- **Start with a simple AI Service**, then add tools and RAG incrementally
