# MSMapper Package & API Documentation

## Overview

MSMapper is an intelligent data mapping library that leverages artificial intelligence to automatically map data between different formats and schemas. It provides seamless integration with multiple AI providers, advanced fuzzy matching, and real-time streaming capabilities for processing large datasets.

## 🚀 Key Features

### 🤖 **AI-Powered Intelligence**
- **Multiple AI Providers**: OpenAI, Groq, Ollama support with extensible provider architecture
- **Fuzzy Matching**: Advanced string similarity algorithms for property mapping
- **Confidence Scoring**: Reliability metrics for mapping decisions

### ⚡ **High-Performance Mapping**
- **Streaming Processing**: Handle large datasets without memory constraints
- **Batch Operations**: Configurable batch sizes for optimal performance
- **Progress Reporting**: Real-time mapping progress with time estimates

### 🎯 **Advanced Mapping Capabilities**
- **Header Mapping**: Intelligent column/property name mapping
- **Smart Mapping**: AI-driven data transformation and validation for low cost with best result




## 🛠️ Package Configuration & Setup

### Basic Registration

```csharp
using AIMapper.Core.Extensions;
using Microsoft.Extensions.DependencyInjection;

// Register AIMapper services
services.AddMsMapper(builder =>
{
    // Configure AI provider
    builder.UseOpenAi(config =>
    {
        config.ApiKey = "your-openai-api-key";
        config.DefaultModel = "gpt-4.1-mini-2025-04-14";
        config.DefaultMaxTokens = 4096;
    });
    
    // Configure core options
    builder.ConfigureCore(options =>
    {
        options.EnableFuzzyMatching = true;
        options.FuzzyMatchThreshold = 80;
        options.SampleSize = 10;
    });
});
```

### Multi-Provider Setup

```csharp
services.AddMsMapper(builder =>
{
    // Primary provider
    builder.UseOpenAi(config =>
    {
        config.ApiKey = "openai-key";
        config.DefaultModel = "gpt-4.1-mini-2025-04-14";
    });
    
    // Alternative provider
    builder.UseGroq(config =>
    {
        config.ApiKey = "groq-key";
        config.DefaultModel = "moonshotai/kimi-k2-instruct";
    });
    
    // Local provider for development
    builder.UseOllama(config =>
    {
        config.BaseUrl = "http://localhost:11434";
        config.DefaultModel = "llama3.1:8b";
    });
});
```

### Advanced Configuration

```csharp
services.AddMsMapper(builder =>
{
    builder.ConfigureCore(options =>
    {
        // AI Configuration
        options.Timeout = TimeSpan.FromMinutes(2);
        
        // Performance Settings
        options.SampleSize = 10; // Analyze 10 rows for pattern recognition
        
        // Fuzzy Matching
        options.EnableFuzzyMatching = true;
        options.FuzzyMatchThreshold = 80; // 80% similarity threshold
        
        // Progress Reporting
        options.EnableProgressReporting = true;
        options.ProgressCallback = progress =>
        {
            Console.WriteLine($"Mapping Progress: {progress.PercentComplete:F1}%");
        };
        
        // Custom Instructions
        options.CustomInstructions = "Prioritize exact matches over fuzzy matches. " +
                                   "Be conservative with type inference for financial data.";
    });
    
    // Text Extraction Configuration
    builder.ConfigureTextExtraction(textBuilder =>
    {
        textBuilder.ConfigureCore(textOptions =>
        {
            textOptions.MaxFileSizeBytes = 100 * 1024 * 1024; // 100MB
            textOptions.EnableSecurityValidation = true;
            textOptions.SecurityLevel = SecurityLevel.High;
        });
    });
});
```

## 🎯 Core Package Usage

### Basic Data Mapping

```csharp
public class DataMappingService
{
    private readonly IMsMapper _mapper;
    
    public DataMappingService(IMsMapper mapper)
    {
        _mapper = mapper;
    }
    
    // Map to strongly-typed object
    public async Task<List<Employee>> MapEmployeeDataAsync(Stream csvStream)
    {
        var result = await _mapper.MapListAsync<Employee>(
            csvStream, 
            "employees.csv", 
            "text/csv"
        );
        
        if (result.IsSuccess && result.MappedObject != null)
        {
            return result.MappedObject;
        }
        
        throw new Exception($"Mapping failed: {result.ErrorMessage}");
    }
    
    // Map to dynamic schema
    public async Task<object> MapToCustomSchemaAsync(Stream dataStream, string jsonSchema)
    {
        var result = await _mapper.MapAsync(
            dataStream,
            "data.xlsx",
            jsonSchema,
            "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
        );
        
        return result.MappedObject;
    }
}
```

### Smart Mapping with AI Intelligence

```csharp
public async Task<List<Product>> SmartMapProductsAsync(Stream excelStream)
{
    // AI analyzes the data structure and creates optimal mappings
    var result = await _mapper.SmartMappingListAsync<Product>(
        excelStream,
        "products.xlsx",
        configure: options =>
        {
            options.CustomInstructions = "Product names should be titleized. " +
                                       "Prices should be parsed as decimal with 2 decimal places.";
        }
    );
    
    if (result.IsSuccess)
    {
        // Review confidence scores
        foreach (var (property, confidence) in result.Confidence.PropertyConfidences)
        {
            if (confidence < 0.8)
            {
                Console.WriteLine($"Low confidence for {property}: {confidence:F2}");
            }
        }
        
        return result.MappedObject ?? new List<Product>();
    }
    
    return new List<Product>();
}
```

### Header Mapping for Data Analysis

```csharp
public async Task<HeaderMappingResult> AnalyzeDataStructureAsync(Stream dataStream)
{
    // First, analyze headers and infer optimal mapping
    var headerResult = await _mapper.MapHeaderAsync<Customer>(
        dataStream,
        "customer-data.csv",
        configure: options =>
        {
            options.EnableFuzzyMatching = true;
            options.FuzzyMatchThreshold = 85;
        }
    );
    
    if (headerResult.IsSuccess)
    {
        // Review suggested mappings
        foreach (var (source, match) in headerResult.PropertyMappings)
        {
            Console.WriteLine($"{source} → {match.TargetProperty} (Score: {match.Score})");
        }
        
        // Use the mapping result for actual data processing
        var dataResult = await _mapper.MapAsync(
            headerResult,
            dataStream,
            "customer-data.csv"
        );
        
        return headerResult;
    }
    
    throw new Exception("Header analysis failed");
}
```

### Streaming for Large Datasets

```csharp
public async Task ProcessLargeDatasetAsync(Stream dataStream, string fileName)
{
    var processedRecords = new List<Customer>();
    
    await foreach (var chunk in _mapper.MapStreamAsync<Customer>(
        dataStream,
        fileName,
        configure: options =>
        {
            options.EnableProgressReporting = true;
            options.ProgressCallback = async progress =>
            {
                // Update UI or log progress
                await UpdateProgressAsync(progress);
            };
        },
        configureExtraction: extractOptions =>
        {
            extractOptions.BatchSizeBytes = 2 * 1024 * 1024; // 2MB chunks
            extractOptions.ExcelRowBatchSize = 200; // 200 rows per batch
        }))
    {
        if (chunk.IsSuccess && chunk.MappedData != null)
        {
            var customers = (List<Customer>)chunk.MappedData;
            processedRecords.AddRange(customers);
            
            // Process batch immediately to avoid memory buildup
            await ProcessCustomerBatchAsync(customers);
        }
        else
        {
            Console.WriteLine($"Chunk mapping failed: {chunk.ErrorMessage}");
        }
        
        // Check if we've reached the last chunk
        if (chunk.IsLastChunk)
        {
            Console.WriteLine($"Processing completed. Total records: {processedRecords.Count}");
            break;
        }
    }
}
```

## 🌐 REST API Endpoints

The following API endpoints are available when using the sample controllers or implementing similar endpoints in your application:

### Base URL Structure
```
https://your-api-domain.com/api/mapper/
```

### 1. Basic Mapping

#### `POST /api/mapper/map`
Maps uploaded file to strongly-typed object using default schema inference.

**Request:**
```http
POST /api/mapper/map
Content-Type: multipart/form-data

file: [binary file data]
```

**Response:**
```json
{
  "isSuccess": true,
  "mappedObject": [
    {
      "id": 1,
      "year": 2024,
      "activitySection": "Development"
    }
  ],
  "confidence": {
    "overallConfidence": 0.95,
    "confidenceLevel": "High",
    "propertyConfidences": {
      "id": 0.98,
      "year": 0.95,
      "activitySection": 0.92
    }
  },
  "processingTime": "00:00:02.1234567",
  "aiProvider": "OpenAI"
}
```

#### `POST /api/mapper/map-schema`
Maps uploaded file using custom JSON schema.

**Request:**
```http
POST /api/mapper/map-schema
Content-Type: multipart/form-data

file: [binary file data]
schema: {
  "type": "object",
  "title": "DataRecord",
  "description": "Sample data record schema",
  "properties": {
    "Id": {
      "type": "integer",
      "description": "Unique identifier"
    },
    "Year": {
      "type": "integer", 
      "description": "Year value"
    },
    "ActivitySection": {
      "type": "string",
      "description": "Activity section description"
    }
  },
  "required": ["Id", "Year", "ActivitySection"]
}
```

### 2. Header Analysis


#### `POST /api/mapper/map-schema-header`
Analyzes file structure using custom schema and returns header mapping recommendations.

**Request:**
```http
POST /api/mapper/map-schema-header
Content-Type: multipart/form-data

file: [binary file data]
schema: {
  "type": "object",
  "title": "DataRecord",
  "description": "Sample data record schema",
  "properties": {
    "Id": {
      "type": "integer",
      "description": "Unique identifier"
    },
    "Year": {
      "type": "integer", 
      "description": "Year value"
    },
    "ActivitySection": {
      "type": "string",
      "description": "Activity section description"
    }
  },
  "required": ["Id", "Year", "ActivitySection"]
}
```

**Response:**
```json
{
  "isSuccess": true,
  "sourceHeaders": ["emp_id", "full_name", "dept", "annual_salary"],
  "propertyMappings": {
    "emp_id": {
      "sourceProperty": "emp_id",
      "targetProperty": "Id",
      "score": 0.92,
      "matchType": "Fuzzy",
      "algorithm": "LevenshteinDistance"
    },
    "full_name": {
      "sourceProperty": "full_name",
      "targetProperty": "FullName",
      "score": 0.95,
      "matchType": "Fuzzy"
    }
  },
  "typeInferences": {
    "emp_id": {
      "inferredType": "System.Int32",
      "confidence": 0.98,
      "sampleValues": [1, 2, 3, 4, 5]
    }
  },
  "warnings": [],
  "processingTime": "00:00:01.5432"
}
```

### 3. Smart Mapping


#### `POST /api/mapper/smart-map-schema`
Uses AI intelligence with custom schema for automatic mapping and transformation.

**Request:**
```http
POST /api/mapper/smart-map-schema
Content-Type: multipart/form-data

file: [binary file data]
schema: {
  "type": "object",
  "title": "DataRecord",
  "description": "Sample data record schema",
  "properties": {
    "Id": {
      "type": "integer",
      "description": "Unique identifier"
    },
    "Year": {
      "type": "integer", 
      "description": "Year value"
    },
    "ActivitySection": {
      "type": "string",
      "description": "Activity section description"
    }
  },
  "required": ["Id", "Year", "ActivitySection"]
}
```

**Response:**
```json
{
  "isSuccess": true,
  "mappedObject": [...],
  "confidence": {
    "overallConfidence": 0.87,
    "confidenceLevel": "High",
    "uncertainProperties": ["department"]
  },
  "warnings": [
    {
      "propertyName": "department",
      "message": "Multiple possible values detected",
      "warningType": "AmbiguousMapping",
      "confidence": 0.65
    }
  ],
  "metadata": {
    "aiProvider": "OpenAI",
    "modelUsed": "gpt-4",
    "usage": {
      "promptTokens": 1250,
      "completionTokens": 890,
      "totalTokens": 2140
    },
    "processingSteps": [
      "Data structure analysis",
      "Type inference",
      "Property mapping",
      "Data transformation",
      "Validation"
    ]
  }
}
```

### 4. Streaming Endpoints

#### `POST /api/mapper/map-stream`
Processes large files using server-sent events (SSE) for real-time progress updates.

**Request:**
```http
POST /api/mapper/map-stream
Content-Type: multipart/form-data

file: [binary file data]
```

**Response (Server-Sent Events):**
```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"type":"progress","stage":"Extracting","percent":15.5,"itemsProcessed":155,"totalItems":1000,"elapsed":"00:00:03","estimatedRemaining":"00:00:17"}

data: {"type":"chunk","index":0,"chunkType":"StructuredData","mappedData":[...],"isSuccess":true,"processingTime":"00:00:00.234"}

data: {"type":"progress","stage":"Mapping","percent":45.2,"itemsProcessed":452,"totalItems":1000}

data: {"type":"chunk","index":1,"mappedData":[...],"isSuccess":true}

data: {"type":"completed"}
```

### 5. Error Handling

All endpoints return standardized error responses:

```json
{
  "isSuccess": false,
  "errorMessage": "Unable to process file: Unsupported format",
  "warnings": [
    "File size exceeds recommended limits",
    "Some data rows were skipped due to format issues"
  ],
  "processingTime": "00:00:01.234",
  "estimatedCost": 0.01
}
```

## 📊 API Usage Examples

### JavaScript/TypeScript Client

```javascript

// Streaming with progress
async function mapFileWithProgress(file, onProgress) {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await fetch('/api/mapper/map-stream', {
        method: 'POST',
        body: formData
    });
    
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    while (true) {
        const { value, done } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');
        
        for (const line of lines) {
            if (line.startsWith('data: ')) {
                const data = JSON.parse(line.slice(6));
                
                if (data.type === 'progress') {
                    onProgress(data);
                } else if (data.type === 'chunk') {
                    console.log('Received chunk:', data.mappedData);
                } else if (data.type === 'completed') {
                    console.log('Mapping completed');
                    return;
                }
            }
        }
    }
}

// Custom schema mapping
async function mapWithSchema(file) {
    const schema = {
        "type": "object",
        "title": "DataRecord",
        "description": "Sample data record schema",
        "properties": {
            "Id": {
                "type": "integer",
                "description": "Unique identifier"
            },
            "Year": {
                "type": "integer", 
                "description": "Year value"
            },
            "ActivitySection": {
                "type": "string",
                "description": "Activity section description"
            }
        },
        "required": ["Id", "Year", "ActivitySection"]
    };
    
    const formData = new FormData();
    formData.append('file', file);
    formData.append('schema', JSON.stringify(schema));
    
    const response = await fetch('/api/mapper/map-schema', {
        method: 'POST',
        body: formData
    });
    
    return await response.json();
}
```

### cURL Commands

```bash
# Custom schema mapping
curl -X POST "https://your-api.com/api/mapper/map-schema" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@data.csv" \
  -F 'schema={
    "type": "object",
    "title": "DataRecord", 
    "description": "Sample data record schema",
    "properties": {
      "Id": {
        "type": "integer",
        "description": "Unique identifier"
      },
      "Year": {
        "type": "integer",
        "description": "Year value"
      },
      "ActivitySection": {
        "type": "string",
        "description": "Activity section description"
      }
    },
    "required": ["Id", "Year", "ActivitySection"]
  }'

# Schema header analysis
curl -X POST "https://your-api.com/api/mapper/map-schema-header" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@data.xlsx" \
  -F 'schema={
    "type": "object",
    "properties": {
      "Id": {"type": "integer"},
      "Name": {"type": "string"}
    }
  }'

# Smart mapping with schema
curl -X POST "https://your-api.com/api/mapper/smart-map-schema" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@data.xlsx" \
  -F 'schema={
    "type": "object",
    "properties": {
      "Id": {"type": "integer"},
      "Name": {"type": "string"}
    }
  }'

# Schema streaming mapping
curl -X POST "https://your-api.com/api/mapper/map-schema-stream" \
  -H "Content-Type: multipart/form-data" \
  -H "Accept: text/event-stream" \
  -F "file=@large_dataset.csv" \
  -F 'schema={
    "type": "object",
    "properties": {
      "Id": {"type": "integer"},
      "Name": {"type": "string"},
      "Value": {"type": "number"}
    }
  }' \
  --no-buffer
```

## 🔧 Advanced Configuration

### Provider-Specific Settings

#### OpenAI Configuration
```csharp
builder.UseOpenAi(config =>
{
    config.ApiKey = "sk-...";
    config.DefaultModel = "gpt-4.1-mini-2025-04-14"; // or other models
    config.DefaultTemperature = 0.1; // Low temperature for consistent results
    config.DefaultMaxTokens = 4096;
    config.Timeout = TimeSpan.FromMinutes(2);
    config.MaxRetries = 3;
    config.OrganizationId = "org-..."; // Optional
    config.BaseUrl = "https://api.openai.com/v1"; // Custom endpoint
    config.EnableLogging = true;
});
```

#### Groq Configuration
```csharp
builder.UseGroq(config =>
{
    config.ApiKey = "gsk_...";
    config.DefaultModel = "moonshotai/kimi-k2-instruct"; // or other models
    config.DefaultTemperature = 0.1;
    config.DefaultMaxTokens = 2000;
    config.Timeout = TimeSpan.FromMinutes(2);
    config.MaxRetries = 3;
    config.BaseUrl = "https://api.groq.com/openai/v1";
    config.EnableLogging = true;
});
```

#### Ollama Configuration
```csharp
builder.UseOllama(config =>
{
    config.BaseUrl = "http://localhost:11434";
    config.DefaultModel = "llama3.1:8b"; // or "codellama", "mistral"
    config.DefaultTemperature = 0.1;
    config.DefaultMaxTokens = 2000;
    config.Timeout = TimeSpan.FromMinutes(5); // Local models can be slower
    config.MaxRetries = 2;
    config.EnableLogging = true;
    config.UseStreaming = false;
});
```

### Custom AI Provider

```csharp
public class CustomAIProvider : BaseAIProvider
{
    public override string ProviderName => "CustomProvider";
    
    public override AIProviderCapabilities Capabilities => new()
    {
        SupportsStreaming = true,
        SupportsEmbeddings = false,
        MaxTokens = 8192,
        SupportedLanguages = new[] { "en", "ar" }
    };

    public override AIProviderLimits Limits => new()
    {
        RequestsPerMinute = 30,
        RequestsPerDay = 14400,
        TokensPerMinute = 30000,
        CostPerInputToken = 0.0001m,
        CostPerOutputToken = 0.0001m,
        RateLimitWindow = TimeSpan.FromMinutes(1)
    };
    
    public override async Task<AIResponse> MapAsync(AIRequest request, CancellationToken cancellationToken)
    {
        // Implement custom AI logic
        var response = await CallCustomAIServiceAsync(request, cancellationToken);
        
        return new AIResponse
        {
            IsSuccess = true,
            MappedData = response.Data,
            Confidence = response.Confidence,
            Usage = response.Usage,
            Cost = CalculateCost(response.Usage)
        };
    }
    
    private async Task<CustomAIResponse> CallCustomAIServiceAsync(AIRequest request, CancellationToken cancellationToken)
    {
        // Your custom AI service implementation
        throw new NotImplementedException();
    }
}

// Register custom provider
services.AddScoped<IAIProvider, CustomAIProvider>();
```

## 📊 Data Models

### Core Models

```csharp
// Mapping result for strongly-typed objects
public class MappingResult<T> : MappingResult
{
    public new T? MappedObject { get; set; }
}

// Base mapping result
public class MappingResult
{
    public bool IsSuccess { get; set; } = true;
    public object? MappedObject { get; set; }
    public Dictionary<string, object> Metadata { get; set; } = new();
    public List<MappingWarning> Warnings { get; set; } = new();
    public string? ErrorMessage { get; set; }
    public decimal EstimatedCost { get; set; }
    public TimeSpan ProcessingTime { get; set; }
    public string? AIProvider { get; set; }
    public MappingConfidence Confidence { get; set; } = new();
}

// Header mapping analysis result
public class HeaderMappingResult
{
    public List<string> SourceHeaders { get; set; } = new();
    public ITargetSchema TargetSchema { get; set; }
    public bool IsSuccess { get; set; } = true;
    public Dictionary<string, PropertyMatch> PropertyMappings { get; set; } = new();
    public Dictionary<string, TypeInferenceInfo> TypeInferences { get; set; } = new();
    public List<MappingWarning> Warnings { get; set; } = new();
    public string? ErrorMessage { get; set; }
    public TimeSpan ProcessingTime { get; set; }
}

// Streaming chunk data
public class MappingChunk
{
    public int Index { get; set; }
    public MappingChunkType ChunkType { get; set; }
    public object? MappedData { get; set; }
    public bool IsSuccess { get; set; } = true;
    public string? ErrorMessage { get; set; }
    public bool IsLastChunk { get; set; }
    public MappingProgress Progress { get; set; } = new();
    public MappingMetadata Metadata { get; set; } = new();
    public List<MappingWarning> Warnings { get; set; } = new();
    public TimeSpan ProcessingTime { get; set; }
}

// Confidence metrics
public class MappingConfidence
{
    public double OverallConfidence { get; set; } = 0.0; // 0-1
    public Dictionary<string, double> PropertyConfidences { get; set; } = new();
    public string ConfidenceLevel { get; set; } = "Unknown"; // Low, Medium, High
    public List<string> UncertainProperties { get; set; } = new();
}
```

## 🎯 Best Practices

### 1. Cost Management
```csharp
// ✅ Good: Set reasonable cost limits
// use smart mapping

// ✅ Good: Use smaller sample sizes for pattern recognition
options.SampleSize = 10; // Analyze 10 rows instead of all data

// ✅ Good: Choose appropriate AI provider for task complexity
builder.UseGroq(config => { /* Fast, cost-effective */ });
builder.UseOpenAi(config => { /* High accuracy, higher cost */ });
```

### 2. Performance Optimization
```csharp
// ✅ Good: Use streaming for large datasets
await foreach (var chunk in mapper.MapStreamAsync<T>(stream, fileName))
{
    await ProcessChunkAsync(chunk.MappedData);
}

// ✅ Good: Configure appropriate batch sizes
configureExtraction: options =>
{
    options.BatchSizeBytes = 2 * 1024 * 1024; // 2MB chunks
    options.ExcelRowBatchSize = 200; // 200 rows per batch
}
```

### 3. Error Handling & Validation
```csharp
// ✅ Good: Always check mapping results
var result = await mapper.MapAsync<Customer>(stream, fileName);

if (!result.IsSuccess)
{
    _logger.LogError("Mapping failed: {Error}", result.ErrorMessage);
    return null;
}

// Check confidence levels
if (result.Confidence.OverallConfidence < 0.8)
{
    _logger.LogWarning("Low confidence mapping: {Confidence}", 
        result.Confidence.OverallConfidence);
    
    // Review uncertain properties
    foreach (var property in result.Confidence.UncertainProperties)
    {
        _logger.LogWarning("Uncertain property: {Property}", property);
    }
}

return result.MappedObject;
```

### 4. Schema Design
```csharp
// ✅ Good: Use data annotations for better AI understanding
public class Employee
{
    [Description("Unique employee identifier")]
    [Required]
    public int Id { get; set; }
    
    [Description("Employee's full legal name")]
    [Required]
    [MaxLength(100)]
    public string FullName { get; set; } = string.Empty;
    
    [Description("Department or division name")]
    public string? Department { get; set; }
    
    [Description("Annual salary in USD")]
    [Range(0, 1000000)]
    public decimal? Salary { get; set; }
    
    [Description("Date when employee started working")]
    public DateTime? StartDate { get; set; }
}
```


---

**MSMapper** - Intelligent data mapping powered by artificial intelligence.
