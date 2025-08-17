# TextExtraction Package

## Overview

TextExtraction is a powerful, enterprise-grade text extraction library for .NET that intelligently extracts text and metadata from various document formats. Built with performance, security, and scalability in mind, it supports streaming extraction for large files and provides comprehensive content analysis capabilities.

## 🚀 Key Features

### 📄 **Comprehensive Document Support**
- **PDF Documents**: Advanced text extraction with table detection and metadata preservation
- **Microsoft Office**: Word (.docx, .doc), Excel (.xlsx, .xls), PowerPoint (.pptx, .ppt)
- **Web Formats**: HTML, XHTML, XML with structured content parsing
- **Text Formats**: Plain text, CSV, JSON

### ⚡ **High-Performance Processing**
- **True Streaming**: Process large files without loading entire content into memory
- **Batch Processing**: Configurable batch sizes for optimal performance
- **Memory Optimization**: Built-in garbage collection triggers and memory pressure monitoring

### 🔒 **Enterprise Security**
- **Sensitive Data Detection**: Automatic detection and masking of credit cards, SSNs, emails, phone numbers
- **Content Sanitization**: Configurable data cleaning and validation
- **Security Levels**: Low, Medium, High security validation options
- **Document Integrity**: File validation and corruption detection

### 📊 **Advanced Content Analysis**
- **Metadata Extraction**: Author, creation date, modification date, document properties
- **Structured Content**: Tables, images, links, headings extraction
- **Progress Reporting**: Real-time extraction progress with time estimates

### 🎯 **Developer-Friendly Features**
- **Flexible Configuration**: Extensive options for customizing extraction behavior
- **Error Handling**: Graceful error recovery with detailed error messages
- **Logging Integration**: Built-in support for Microsoft.Extensions.Logging
- **Encoding Detection**: Automatic character encoding detection


## 🛠️ Quick Start

### Basic Setup

```csharp
using AIMapper.TextExtraction.Extensions;
using Microsoft.Extensions.DependencyInjection;

// Register services
services.AddTextExtractionServices(builder =>
{
    builder.ConfigureCore(options =>
    {
        options.MaxFileSizeBytes = 100 * 1024 * 1024; // 100MB
        options.EnableSecurityValidation = true;
        options.SecurityLevel = SecurityLevel.Medium;
    });
});
```

### Simple Text Extraction

```csharp
public class DocumentProcessor
{
    private readonly ITextExtractor _textExtractor;
    
    public DocumentProcessor(ITextExtractor textExtractor)
    {
        _textExtractor = textExtractor;
    }
    
    public async Task<string> ExtractTextAsync(string filePath)
    {
        var result = await _textExtractor.ExtractAsync(filePath, options =>
        {
            options.ExtractMetadata = true;
            options.ExtractStructuredContent = true;
            options.PreserveFormatting = false;
        });
        
        if (result.IsSuccess)
        {
            return result.Text;
        }
        
        throw new Exception($"Extraction failed: {result.ErrorMessage}");
    }
}
```

### Stream-Based Extraction

```csharp
public async Task ProcessLargeFileAsync(Stream fileStream, string fileName, string mimeType)
{
    await foreach (var chunk in _textExtractor.ExtractStreamAsync(
        fileStream, fileName, mimeType, options =>
        {
            options.EnableProgressReporting = true;
            options.BatchSizeBytes = 1024 * 1024; // 1MB chunks
            options.MaxMemoryUsageBytes = 50 * 1024 * 1024; // 50MB max memory
            options.ProgressCallback = progress =>
            {
                Console.WriteLine($"Progress: {progress.PercentComplete:F1}% - {progress.Stage}");
            };
        }))
    {
        if (chunk.IsSuccess)
        {
            // Process chunk content
            await ProcessChunkAsync(chunk);
        }
        else
        {
            Console.WriteLine($"Chunk error: {chunk.ErrorMessage}");
        }
    }
}
```

## 🔧 Configuration Options

### Core Options

```csharp
builder.ConfigureCore(options =>
{
    // File processing limits
    options.MaxFileSizeBytes = 100 * 1024 * 1024; // 100MB
    options.Timeout = TimeSpan.FromMinutes(10);
    
    // Content extraction settings
    options.ExtractMetadata = true;
    options.ExtractStructuredContent = true;
    options.PreserveFormatting = false;
    options.DetectLanguage = true;
    options.PreferredLanguages = new List<string> { "Arabic", "Latin" }; // Actual defaults
    
    // Security settings
    options.EnableSecurityValidation = true;
    options.EnableMasking = true;
    options.SecurityLevel = SecurityLevel.High;
    
    // Performance settings
    options.BatchSizeBytes = 1024 * 64; // 64KB (actual default)
    options.ExcelRowBatchSize = 50;
    options.MaxMemoryUsageBytes = 50 * 1024 * 1024; // 50MB
    
    // Progress reporting
    options.EnableProgressReporting = true;
    options.ProgressReportInterval = TimeSpan.FromSeconds(2); // Actual default
    options.ProgressCallback = progress =>
    {
        // Handle progress updates
    };
    
    // Streaming settings
    options.EnableTrueStreaming = true; // Actual default
});
```


## 📊 Supported File Formats

| Format | MIME Type | Extensions | Features |
|--------|-----------|------------|----------|
| **PDF** | `application/pdf` | .pdf | Text, tables, images, metadata, forms |
| **Word** | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | .docx | Text, tables, images, headers/footers |
| **Excel** | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | .xlsx | Worksheets, formulas, metadata |
| **PowerPoint** | `application/vnd.openxmlformats-officedocument.presentationml.presentation` | .pptx | Slides, text, images, metadata |
| **HTML** | `text/html` | .html, .htm | Text, tables, links, images, metadata |
| **XML** | `text/xml`, `application/xml` | .xml | Structured data, namespaces, attributes |
| **Text** | `text/plain` | .txt | Plain text with encoding detection |
| **CSV** | `text/csv`, `application/csv` | .csv | Structured data with header detection |
| **JSON** | `application/json` | .json | Structured JSON data |

## 🎯 Advanced Usage Scenarios

### Excel Data Processing

```csharp
public async Task<List<Dictionary<string, object>>> ProcessExcelFileAsync(Stream excelStream)
{
    var data = new List<Dictionary<string, object>>();
    
    await foreach (var chunk in _textExtractor.ExtractStreamAsync(
        excelStream, "data.xlsx", "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
        options =>
        {
            options.ExtractStructuredContent = true;
            options.ExcelRowBatchSize = 100; // Process 100 rows at a time
        }))
    {
        if (chunk.ChunkType == ChunkType.StructuredData && chunk.IsSuccess)
        {
            var structuredData = chunk.Content.StructuredContent;
            if (structuredData.ContainsKey("data"))
            {
                var rows = (List<Dictionary<string, object>>)structuredData["data"];
                data.AddRange(rows);
            }
        }
    }
    
    return data;
}
```

### PDF Table Extraction

```csharp
public async Task<List<object>> ExtractPdfTablesAsync(string pdfPath)
{
    var result = await _textExtractor.ExtractAsync(pdfPath, options =>
    {
        options.ExtractStructuredContent = true;
        options.PreserveFormatting = true;
    });
    
    if (result.IsSuccess && result.Content.StructuredContent.ContainsKey("tables"))
    {
        return (List<object>)result.Content.StructuredContent["tables"];
    }
    
    return new List<object>();
}
```

### HTML Content Analysis

```csharp
public async Task<ContentAnalysis> AnalyzeWebPageAsync(string htmlContent)
{
    using var stream = new MemoryStream(Encoding.UTF8.GetBytes(htmlContent));
    
    var result = await _textExtractor.ExtractAsync(stream, "webpage.html", 
        "text/html", options =>
        {
            options.ExtractMetadata = true;
            options.ExtractStructuredContent = true;
        });
    
    return new ContentAnalysis
    {
        Title = result.Metadata.GetValueOrDefault("Title")?.ToString(),
        Description = result.Metadata.GetValueOrDefault("Description")?.ToString(),
        LinkCount = (int)(result.Metadata.GetValueOrDefault("LinkCount") ?? 0),
        ImageCount = result.Content.StructuredContent.ContainsKey("images") 
            ? ((List<object>)result.Content.StructuredContent["images"]).Count 
            : 0,
        WordCount = result.Text.Split(' ', StringSplitOptions.RemoveEmptyEntries).Length
    };
}
```


## 🛡️ Security Features

### Data Masking

The library automatically detects and masks sensitive information:

```csharp
// Credit Card: 4532-1234-5678-9012 → 4532-****-****-9012
// SSN: 123-45-6789 → ***-**-6789
// Email: user@domain.com → use***@domain.com
// Phone: (555) 123-4567 → (555) ***-4567
```


## 📈 Performance Characteristics

### Memory Usage
- **Streaming Mode**: ~10-50MB regardless of file size
- **Batch Mode**: Configurable based on batch size settings
- **Auto GC**: Automatic garbage collection when memory threshold exceeded


### Scalability
- **Large Files**: Supports files up to several GB with streaming
- **Memory Efficiency**: Constant memory usage regardless of file size

## 🔍 Error Handling

### Graceful Error Recovery

```csharp
var result = await _textExtractor.ExtractAsync(filePath, options =>
{
    options.ValidateDocumentIntegrity = true;
    options.EnableSecurityValidation = true;
});

if (!result.IsSuccess)
{
    Console.WriteLine($"Extraction failed: {result.ErrorMessage}");
    
    // Check for warnings
    foreach (var warning in result.Warnings)
    {
        Console.WriteLine($"Warning: {warning}");
    }
    
    // Partial success handling
    if (!string.IsNullOrEmpty(result.Text))
    {
        Console.WriteLine("Partial content extracted successfully");
    }
}
```

### Stream Error Handling

```csharp
await foreach (var chunk in _textExtractor.ExtractStreamAsync(stream, fileName, mimeType))
{
    if (!chunk.IsSuccess)
    {
        if (chunk.IsLastChunk)
        {
            // Critical error - stop processing
            throw new Exception($"Critical extraction error: {chunk.ErrorMessage}");
        }
        else
        {
            // Log warning and continue
            _logger.LogWarning("Chunk extraction warning: {Error}", chunk.ErrorMessage);
            continue;
        }
    }
    
    // Process successful chunk
    await ProcessChunkAsync(chunk);
}
```

## 📚 Best Practices

### 1. Memory Management
```csharp
// ✅ Good: Use streaming for large files
await foreach (var chunk in extractor.ExtractStreamAsync(stream, fileName, mimeType))
{
    // Process chunk immediately
    await ProcessChunkAsync(chunk);
    // Don't accumulate chunks in memory
}

// ❌ Bad: Loading entire large file
var result = await extractor.ExtractAsync(largeFilePath); // May cause OutOfMemoryException
```

### 2. Error Handling
```csharp
// ✅ Good: Comprehensive error handling
try
{
    var result = await extractor.ExtractAsync(filePath, options =>
    {
        options.ValidateDocumentIntegrity = true;
    });
    
    if (!result.IsSuccess)
    {
        _logger.LogError("Extraction failed: {Error}", result.ErrorMessage);
        return null;
    }
    
    return result.Text;
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error during extraction");
    throw;
}
```

### 3. Performance Optimization
```csharp
// ✅ Good: Optimized batch sizes
options.BatchSizeBytes = 1024 * 1024; // 1MB for normal files
options.ExcelRowBatchSize = 100; // 100 rows for Excel
options.MaxMemoryUsageBytes = 50 * 1024 * 1024; // 50MB limit
```

### 4. Security Configuration
```csharp
// ✅ Good: Security-first approach
options.EnableSecurityValidation = true;
options.EnableMasking = true;
options.SecurityLevel = SecurityLevel.High;
options.ValidateDocumentIntegrity = true;
```


---

**TextExtraction** - Intelligent document processing for modern applications.
