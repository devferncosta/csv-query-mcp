# CSV Query MCP Server Documentation

## Overview

The CSV Query MCP Server is a custom Model Context Protocol (MCP) server designed to load, parse, and analyze CSV data files. It enables Claude Desktop to work with CSV data from zip files and directories, providing structured data analysis capabilities.

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Configuration](#configuration)
- [Available Tools](#available-tools)
- [Code Structure](#code-structure)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Features

- **Zip File Support**: Automatically extracts CSV files from zip archives
- **Smart CSV Parsing**: Converts CSV data into structured JSON objects
- **Data Type Detection**: Automatically converts numbers, dates, and boolean values
- **Header Normalization**: Cleans and standardizes column headers
- **Memory Caching**: Loads data once and allows multiple queries
- **Error Handling**: Robust error reporting and validation

## Architecture

The MCP server consists of three main components:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Claude        │    │   MCP Server    │    │   File System   │
│   Desktop       │◄──►│   (Node.js)     │◄──►│   CSV/Zip Files │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Data Flow

1. **Load**: User requests to load CSV files from zip or directory
2. **Extract**: Zip files are extracted, CSV files are identified
3. **Parse**: CSV files are parsed into structured JSON objects
4. **Cache**: Parsed data is stored in memory for fast access
5. **Query**: Claude requests data and analyzes it directly

## Installation

### Prerequisites

- Node.js 18+ 
- Claude Desktop
- TypeScript

### Setup Steps

1. **Create Project Directory**
   ```bash
   mkdir csv-query-mcp
   cd csv-query-mcp
   ```

2. **Install Dependencies**
   ```bash
   npm install
   ```

3. **Build Project**
   ```bash
   npm run build
   ```

4. **Configure Claude Desktop**
   
   Edit your Claude Desktop config file:
   - **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
   - **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`

   ```json
   {
     "mcpServers": {
       "csv-query": {
         "command": "node",
         "args": ["/path/to/csv-query-mcp/build/index.js"]
       }
     }
   }
   ```

5. **Restart Claude Desktop**

## Configuration

### package.json

```json
{
  "name": "csv-query-mcp",
  "type": "module",
  "dependencies": {
    "@modelcontextprotocol/sdk": "^0.5.0",
    "papaparse": "^5.4.1",
    "lodash": "^4.17.21",
    "yauzl": "^2.10.0"
  }
}
```

Key configuration points:
- `"type": "module"` enables ES6 module syntax
- MCP SDK handles communication with Claude Desktop
- Papa Parse provides robust CSV parsing
- Yauzl handles zip file extraction

## Available Tools

### 1. `load_csv_files`

**Purpose**: Load CSV files from zip archives or directories

**Parameters**:
- `source` (string, required): Path to zip file or directory containing CSVs

**Example**:
```
Load CSV files from C:/data/sales_data.zip
```

### 2. `get_data` 

**Purpose**: Retrieve parsed CSV data for Claude to analyze

**Parameters**:
- `filename` (string, required): Name of the CSV file to retrieve
- `sample_size` (number, optional): Limit number of rows returned

**Example**:
```
Get data from sales.csv with sample size 100
```

### 3. `list_loaded_data`

**Purpose**: Show information about currently loaded CSV files

**Parameters**: None

**Example**:
```
List loaded data
```

### 4. `preview_data`

**Purpose**: Show a quick preview of CSV file structure

**Parameters**:
- `filename` (string, required): Name of CSV file to preview
- `rows` (number, optional): Number of rows to show (default: 5)

**Example**:
```
Preview data from customers.csv showing 10 rows
```

## Code Structure

### File Organization

```
csv-query-mcp/
├── src/
│   ├── index.ts          # Main MCP server and tool handlers
│   ├── csv-parser.ts     # CSV parsing logic with Papa Parse
│   └── zip-handler.ts    # Zip file extraction utilities
├── build/                # Compiled JavaScript (auto-generated)
├── package.json          # Dependencies and scripts
└── tsconfig.json         # TypeScript configuration
```

### Core Components

#### 1. Main Server (`index.ts`)

```typescript
class CSVQueryMCPServer {
  private server: Server;                    // MCP server instance
  private csvParser: CSVParser;              // CSV parsing component
  private zipHandler: ZipHandler;            // Zip extraction component
  private loadedData: Map<string, any[]>;    // In-memory data cache
}
```

**Key Responsibilities**:
- Initialize MCP server and register tools
- Handle incoming tool requests from Claude Desktop
- Coordinate between CSV parser and zip handler
- Manage in-memory data cache

#### 2. CSV Parser (`csv-parser.ts`)

```typescript
export class CSVParser {
  async parseCSV(filePath: string): Promise<any[]>
}
```

**Key Features**:
- **Header Normalization**: Converts headers to lowercase with underscores
- **Data Type Detection**: Automatically converts strings to numbers, dates, booleans
- **Data Cleaning**: Removes empty strings, handles commas in numbers
- **Error Handling**: Reports parsing errors while continuing processing

**Papa Parse Configuration**:
```typescript
Papa.parse(fileContent, {
  header: true,           // Use first row as object keys
  skipEmptyLines: true,   // Ignore blank rows
  dynamicTyping: true,    // Auto-convert data types
  transformHeader: ...,   // Clean header names
  transform: ...          // Clean data values
});
```

#### 3. Zip Handler (`zip-handler.ts`)

```typescript
export class ZipHandler {
  async extractZip(zipPath: string, extractPath: string): Promise<string[]>
}
```

**Key Features**:
- **Selective Extraction**: Only extracts CSV files from zip archives
- **Directory Creation**: Automatically creates extraction directories
- **File Path Management**: Returns full paths to extracted CSV files
- **Error Handling**: Provides detailed error messages for zip issues

### Data Transformation Pipeline

1. **Raw CSV Input**:
   ```
   City,Receipts,Revenue
   New York,100,5000
   Los Angeles,80,4200
   ```

2. **Papa Parse Processing**:
   - Headers normalized: `city`, `receipts`, `revenue`
   - Numbers converted: `"100"` → `100`
   - Structure created: Array of objects

3. **Final JSON Output**:
   ```json
   [
     {"city": "New York", "receipts": 100, "revenue": 5000},
     {"city": "Los Angeles", "receipts": 80, "revenue": 4200}
   ]
   ```

4. **Claude Analysis**: Receives structured data for direct analysis

## Usage Examples

### Basic Workflow

1. **Load Data**:
   ```
   Load CSV files from C:/data/Q1_sales.zip
   ```
   *Server extracts zip, parses CSVs, caches data*

2. **Explore Structure**:
   ```
   List loaded data
   ```
   *Shows: sales.csv: 1,247 rows, 8 columns [date, customer, product, amount, city...]*

3. **Preview Data**:
   ```
   Preview data from sales.csv
   ```
   *Shows first 5 rows with column structure*

4. **Ask Questions**:
   ```
   What city generated the most revenue?
   ```
   *Claude automatically uses get_data tool and analyzes*

### Advanced Analysis Examples

**Sales Analysis**:
```
- Which month had the highest sales?
- What's the average order value by region?
- Show me the top 10 customers by total purchases
- Are there any seasonal trends in the data?
```

**Data Quality Checks**:
```
- Are there any duplicate customer IDs?
- What percentage of orders have missing data?
- Which products have unusual pricing?
```

**Comparative Analysis**:
```
- How does Q1 compare to Q4 performance?
- Which sales rep has the best conversion rate?
- What's the geographic distribution of our customers?
```

## Troubleshooting

### Common Issues

#### 1. "ReferenceError: require is not defined"

**Cause**: Mixing CommonJS and ES module syntax
**Solution**: Ensure all imports use ES6 syntax:
```typescript
// ✅ Correct
import { createWriteStream } from 'fs';

// ❌ Incorrect  
const fs = require('fs');
```

#### 2. "Server disconnected unexpectedly"

**Cause**: Runtime error in server code
**Solution**: 
1. Test server manually: `node build/index.js`
2. Check console for error details
3. Verify file paths are correct

#### 3. "No CSV files currently loaded"

**Cause**: Load operation failed or path incorrect
**Solution**:
1. Verify file path exists
2. Check file permissions
3. Ensure zip contains CSV files

#### 4. "Failed to parse CSV file"

**Cause**: Malformed CSV or encoding issues
**Solution**:
1. Check CSV file format
2. Verify file encoding (should be UTF-8)
3. Look for special characters or corrupted data

### Debug Steps

1. **Check MCP Server Status**:
   ```bash
   node build/index.js
   # Should show: "CSV Query MCP server running on stdio"
   ```

2. **Verify Configuration**:
   - Check `claude_desktop_config.json` syntax
   - Ensure file paths use forward slashes or escaped backslashes
   - Restart Claude Desktop after config changes

3. **Test with Simple Data**:
   Create a simple test CSV:
   ```csv
   name,age,city
   John,25,NYC
   Jane,30,LA
   ```

4. **Check Claude Desktop Logs**:
   - Help → Developer Tools → Console
   - Look for MCP-related error messages

### Performance Considerations

- **Large Files**: Use `sample_size` parameter for initial exploration
- **Memory Usage**: Server keeps all data in memory - restart if needed
- **File Size Limits**: No hard limits, but very large files may cause slowdowns

## Advanced Features

### Custom Data Transformations

The CSV parser includes several built-in transformations:

- **Date Detection**: Recognizes common date formats (YYYY-MM-DD, MM/DD/YYYY)
- **Number Parsing**: Handles numbers with commas (1,000 → 1000)
- **Boolean Recognition**: Converts "true"/"false" strings to booleans
- **Null Handling**: Empty strings become null values

### Error Recovery

The server includes comprehensive error handling:

- **Partial Load Success**: If some CSVs fail, others still load
- **Graceful Degradation**: Server continues operating after errors
- **Detailed Error Messages**: Clear descriptions of what went wrong

## Future Enhancements

Potential improvements for future versions:

- **Database Integration**: Store parsed data in SQLite for persistence
- **Google Drive Integration**: Direct access to Google Drive files
- **Data Validation**: Schema validation and data quality checks
- **Export Capabilities**: Save analysis results to files
- **Streaming Support**: Handle very large files with streaming

## Contributing

To modify or extend the server:

1. **Development Mode**:
   ```bash
   npm run dev  # Watches for changes and rebuilds
   ```

2. **Adding New Tools**: Extend the `setupToolHandlers()` method
3. **Custom Parsers**: Modify `csv-parser.ts` for specific data formats
4. **Additional File Types**: Extend `zip-handler.ts` for other archives

## License

This MCP server is provided as-is for educational and development purposes.
