---
name: excel-converter
description: "Comprehensive Excel/CSV/JSON format conversion and data processing toolkit. When Claude needs to convert between Excel (.xlsx, .xlsm), CSV (.csv), JSON (.json), or TSV (.tsv) formats, perform data transformation, merge datasets, or process tabular data. Use for: (1) Converting Excel to CSV/JSON, (2) Converting CSV/JSON to Excel, (3) Merging multiple files, (4) Data cleaning and transformation, (5) Batch format conversion, or (6) Excel/CSV/JSON 格式互轉、資料處理、數據轉換、批量轉換、資料清理"
---

# Excel/CSV/JSON Format Converter

## Overview

This skill provides comprehensive data format conversion capabilities between Excel (.xlsx), CSV (.csv), JSON (.json), and TSV (.tsv) formats. It supports bidirectional conversion, data transformation, merging datasets, and batch processing using pandas and openpyxl libraries.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Converting Excel to CSV, JSON, or other formats
- Converting CSV or JSON to Excel
- Transforming data between different formats
- Merging multiple Excel/CSV files
- Data cleaning and processing operations
- Batch conversion of multiple files
- Format transformation with data manipulation

**Trigger phrases include:**
- "Convert Excel to CSV" / "轉換Excel為CSV"
- "Convert CSV to Excel" / "轉換CSV為Excel"
- "Convert Excel to JSON" / "將Excel轉為JSON"
- "Convert JSON to Excel" / "將JSON轉為Excel"
- "Merge Excel files" / "合併Excel檔案"
- "Batch convert files" / "批量轉換檔案"
- "Transform data format" / "轉換資料格式"
- "Clean Excel data" / "清理Excel資料"
- "Process CSV data" / "處理CSV資料"

## How to use this skill

### Workflow Overview

This skill follows a systematic 4-step workflow:

1. **Identify Source and Target** - Determine input format and desired output format
2. **Load Data** - Read source file(s) using appropriate library
3. **Transform Data** - Apply any data cleaning, filtering, or transformation
4. **Save Output** - Write to target format with proper encoding and formatting

### Python Libraries

#### pandas - Primary Tool for Conversions

pandas is the primary library for format conversions and data manipulation:

```python
import pandas as pd

# Read various formats
df = pd.read_excel('input.xlsx')           # Excel
df = pd.read_csv('input.csv')              # CSV
df = pd.read_json('input.json')            # JSON
df = pd.read_csv('input.tsv', sep='\t')    # TSV

# Write to various formats
df.to_excel('output.xlsx', index=False)    # Excel
df.to_csv('output.csv', index=False)       # CSV
df.to_json('output.json', orient='records') # JSON
df.to_csv('output.tsv', sep='\t', index=False) # TSV
```

#### openpyxl - For Excel-Specific Features

Use openpyxl when you need to preserve formulas, formatting, or multiple sheets:

```python
from openpyxl import load_workbook, Workbook

# Load Excel with formatting
wb = load_workbook('input.xlsx')
sheet = wb.active

# Access data
for row in sheet.iter_rows(values_only=True):
    print(row)

# Create new workbook
wb = Workbook()
ws = wb.active
ws.append(['Header1', 'Header2'])
ws.append([1, 2])
wb.save('output.xlsx')
```

## Common Conversion Tasks

### Excel to CSV

```python
import pandas as pd

# Convert single sheet
df = pd.read_excel('input.xlsx')
df.to_csv('output.csv', index=False, encoding='utf-8')

# Convert specific sheet
df = pd.read_excel('input.xlsx', sheet_name='Sheet2')
df.to_csv('output.csv', index=False)

# Convert all sheets to separate CSV files
excel_file = pd.ExcelFile('input.xlsx')
for sheet_name in excel_file.sheet_names:
    df = pd.read_excel(excel_file, sheet_name=sheet_name)
    df.to_csv(f'{sheet_name}.csv', index=False, encoding='utf-8')
```

### CSV to Excel

```python
import pandas as pd

# Convert single CSV
df = pd.read_csv('input.csv')
df.to_excel('output.xlsx', index=False, sheet_name='Data')

# Merge multiple CSV files into one Excel with multiple sheets
with pd.ExcelWriter('output.xlsx', engine='openpyxl') as writer:
    df1 = pd.read_csv('file1.csv')
    df1.to_excel(writer, sheet_name='Sheet1', index=False)

    df2 = pd.read_csv('file2.csv')
    df2.to_excel(writer, sheet_name='Sheet2', index=False)
```

### Excel to JSON

```python
import pandas as pd

# Convert to JSON array of objects
df = pd.read_excel('input.xlsx')
df.to_json('output.json', orient='records', force_ascii=False, indent=2)

# Convert to JSON with specific structure
data = df.to_dict(orient='records')
import json
with open('output.json', 'w', encoding='utf-8') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# Convert multiple sheets to nested JSON
excel_file = pd.ExcelFile('input.xlsx')
result = {}
for sheet_name in excel_file.sheet_names:
    df = pd.read_excel(excel_file, sheet_name=sheet_name)
    result[sheet_name] = df.to_dict(orient='records')

with open('output.json', 'w', encoding='utf-8') as f:
    json.dump(result, f, ensure_ascii=False, indent=2)
```

### JSON to Excel

```python
import pandas as pd
import json

# Convert JSON array to Excel
with open('input.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

df = pd.DataFrame(data)
df.to_excel('output.xlsx', index=False, sheet_name='Data')

# Convert nested JSON to multiple sheets
with open('input.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

with pd.ExcelWriter('output.xlsx', engine='openpyxl') as writer:
    for key, value in data.items():
        df = pd.DataFrame(value)
        df.to_excel(writer, sheet_name=key, index=False)
```

### CSV to JSON

```python
import pandas as pd

# Convert CSV to JSON
df = pd.read_csv('input.csv')
df.to_json('output.json', orient='records', force_ascii=False, indent=2)

# Convert with custom structure
df = pd.read_csv('input.csv')
records = df.to_dict(orient='records')
import json
with open('output.json', 'w', encoding='utf-8') as f:
    json.dump({'data': records, 'count': len(records)}, f, ensure_ascii=False, indent=2)
```

### JSON to CSV

```python
import pandas as pd
import json

# Convert JSON to CSV
with open('input.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

df = pd.DataFrame(data)
df.to_csv('output.csv', index=False, encoding='utf-8')

# Flatten nested JSON
from pandas import json_normalize
with open('input.json', 'r', encoding='utf-8') as f:
    data = json.load(f)

df = json_normalize(data)
df.to_csv('output.csv', index=False, encoding='utf-8')
```

## Data Transformation Operations

### Data Cleaning

```python
import pandas as pd

df = pd.read_excel('input.xlsx')

# Remove duplicates
df = df.drop_duplicates()

# Remove empty rows
df = df.dropna(how='all')

# Remove empty columns
df = df.dropna(axis=1, how='all')

# Fill missing values
df = df.fillna(0)  # Fill with 0
df = df.fillna(method='ffill')  # Forward fill

# Strip whitespace
df = df.apply(lambda x: x.strip() if isinstance(x, str) else x)

# Remove specific columns
df = df.drop(['Column1', 'Column2'], axis=1)

# Rename columns
df = df.rename(columns={'OldName': 'NewName'})

df.to_excel('cleaned.xlsx', index=False)
```

### Data Filtering

```python
import pandas as pd

df = pd.read_excel('input.xlsx')

# Filter rows by condition
filtered = df[df['Age'] > 18]

# Filter by multiple conditions
filtered = df[(df['Age'] > 18) & (df['Status'] == 'Active')]

# Filter by column values
filtered = df[df['Category'].isin(['A', 'B', 'C'])]

# Filter out specific values
filtered = df[~df['Status'].isin(['Deleted', 'Archived'])]

filtered.to_excel('filtered.xlsx', index=False)
```

### Data Merging

```python
import pandas as pd

# Merge multiple Excel files vertically (append)
files = ['file1.xlsx', 'file2.xlsx', 'file3.xlsx']
dfs = [pd.read_excel(f) for f in files]
merged = pd.concat(dfs, ignore_index=True)
merged.to_excel('merged.xlsx', index=False)

# Merge multiple CSV files
files = ['file1.csv', 'file2.csv', 'file3.csv']
dfs = [pd.read_csv(f) for f in files]
merged = pd.concat(dfs, ignore_index=True)
merged.to_csv('merged.csv', index=False)

# Join two datasets by key
df1 = pd.read_excel('users.xlsx')
df2 = pd.read_excel('orders.xlsx')
merged = pd.merge(df1, df2, on='user_id', how='left')
merged.to_excel('joined.xlsx', index=False)
```

### Data Transformation

```python
import pandas as pd

df = pd.read_excel('input.xlsx')

# Add calculated column
df['Total'] = df['Price'] * df['Quantity']

# Convert data types
df['Date'] = pd.to_datetime(df['Date'])
df['Price'] = df['Price'].astype(float)

# Apply function to column
df['Name'] = df['Name'].str.upper()

# Group and aggregate
summary = df.groupby('Category').agg({
    'Sales': 'sum',
    'Quantity': 'mean'
}).reset_index()

# Pivot table
pivot = df.pivot_table(
    index='Category',
    columns='Month',
    values='Sales',
    aggfunc='sum'
)

df.to_excel('transformed.xlsx', index=False)
```

## Batch Processing

### Batch Convert Excel to CSV

```python
import pandas as pd
import os
from pathlib import Path

input_dir = 'excel_files'
output_dir = 'csv_files'

# Create output directory
Path(output_dir).mkdir(exist_ok=True)

# Convert all Excel files
for filename in os.listdir(input_dir):
    if filename.endswith('.xlsx') or filename.endswith('.xls'):
        input_path = os.path.join(input_dir, filename)
        output_path = os.path.join(output_dir, filename.replace('.xlsx', '.csv').replace('.xls', '.csv'))

        df = pd.read_excel(input_path)
        df.to_csv(output_path, index=False, encoding='utf-8')
        print(f"Converted: {filename}")
```

### Batch Convert CSV to Excel

```python
import pandas as pd
import os
from pathlib import Path

input_dir = 'csv_files'
output_dir = 'excel_files'

Path(output_dir).mkdir(exist_ok=True)

for filename in os.listdir(input_dir):
    if filename.endswith('.csv'):
        input_path = os.path.join(input_dir, filename)
        output_path = os.path.join(output_dir, filename.replace('.csv', '.xlsx'))

        df = pd.read_csv(input_path)
        df.to_excel(output_path, index=False)
        print(f"Converted: {filename}")
```

### Batch Merge and Convert

```python
import pandas as pd
import os

# Find all CSV files in directory
csv_files = [f for f in os.listdir('.') if f.endswith('.csv')]

# Read and merge all CSV files
dfs = []
for file in csv_files:
    df = pd.read_csv(file)
    df['Source'] = file  # Track source file
    dfs.append(df)

merged = pd.concat(dfs, ignore_index=True)

# Save in multiple formats
merged.to_excel('merged_output.xlsx', index=False)
merged.to_csv('merged_output.csv', index=False, encoding='utf-8')
merged.to_json('merged_output.json', orient='records', force_ascii=False, indent=2)

print(f"Merged {len(csv_files)} files into Excel, CSV, and JSON formats")
```

## Advanced Features

### Preserving Excel Formatting

```python
from openpyxl import load_workbook
from openpyxl.styles import Font, PatternFill
import pandas as pd

# Read data with pandas
df = pd.read_excel('input.xlsx')

# Modify data
df['Total'] = df['Price'] * df['Quantity']

# Write back to Excel
df.to_excel('output.xlsx', index=False)

# Apply formatting using openpyxl
wb = load_workbook('output.xlsx')
ws = wb.active

# Format header row
for cell in ws[1]:
    cell.font = Font(bold=True, color='FFFFFF')
    cell.fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')

# Auto-adjust column widths
for column in ws.columns:
    max_length = 0
    column_letter = column[0].column_letter
    for cell in column:
        if cell.value:
            max_length = max(max_length, len(str(cell.value)))
    ws.column_dimensions[column_letter].width = max_length + 2

wb.save('output.xlsx')
```

### Handling Large Files

```python
import pandas as pd

# Read large CSV in chunks
chunk_size = 10000
chunks = []

for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # Process each chunk
    chunk_processed = chunk[chunk['Value'] > 0]
    chunks.append(chunk_processed)

# Combine processed chunks
df = pd.concat(chunks, ignore_index=True)
df.to_excel('processed.xlsx', index=False)

# Alternative: Use iterator for Excel
excel_writer = pd.ExcelWriter('output.xlsx', engine='openpyxl')
for i, chunk in enumerate(pd.read_csv('large_file.csv', chunksize=chunk_size)):
    chunk.to_excel(excel_writer, sheet_name=f'Chunk_{i+1}', index=False)
excel_writer.close()
```

### Encoding Handling

```python
import pandas as pd

# Try different encodings
encodings = ['utf-8', 'utf-8-sig', 'latin1', 'cp1252', 'gbk']

for encoding in encodings:
    try:
        df = pd.read_csv('input.csv', encoding=encoding)
        print(f"Successfully read with {encoding}")
        break
    except UnicodeDecodeError:
        continue

# Save with specific encoding
df.to_csv('output.csv', encoding='utf-8-sig', index=False)  # UTF-8 with BOM
df.to_excel('output.xlsx', index=False)  # Excel handles encoding automatically
```

## 繁體中文處理最佳實踐 (Traditional Chinese Handling)

### ⚠️ Windows 環境中文 Excel 處理的核心原則

在 Windows 環境中處理包含繁體中文的 Excel 檔案時,常遇到以下問題:
1. **編碼問題**: CSV 中文顯示為亂碼
2. **數字格式問題**: 編號如 "1-1-1" 被 Excel 誤認為日期
3. **PowerShell 腳本問題**: 腳本內嵌中文導致執行失敗

**核心解決方案: 資料與腳本分離**

### 方案 A: Python pandas (推薦用於資料轉換)

#### 讀取繁體中文 Excel
```python
import pandas as pd

# 讀取包含中文的 Excel (pandas 自動處理編碼)
df = pd.read_excel('中文資料.xlsx')

# 讀取包含中文的 CSV (必須指定編碼)
df = pd.read_csv('中文資料.csv', encoding='utf-8-sig')  # UTF-8 with BOM
# 或嘗試其他編碼
df = pd.read_csv('中文資料.csv', encoding='big5')      # 台灣繁體中文傳統編碼
```

#### 寫入繁體中文檔案
```python
# 寫入 Excel (自動處理中文,推薦)
df.to_excel('輸出.xlsx', index=False)

# 寫入 CSV (必須指定 UTF-8 with BOM,讓 Excel 正確開啟)
df.to_csv('輸出.csv', index=False, encoding='utf-8-sig')

# 寫入 JSON (保留中文字符)
df.to_json('輸出.json', orient='records', force_ascii=False, indent=2)
```

#### 防止數字被解釋為日期
```python
import pandas as pd

# 讀取時指定欄位為字串型別
df = pd.read_excel('input.xlsx', dtype={'CASE編號': str})

# 或在轉換後強制設為字串
df['CASE編號'] = df['CASE編號'].astype(str)

# 寫入 Excel 時確保格式正確
df.to_excel('output.xlsx', index=False)
```

### 方案 B: PowerShell + Excel COM (用於複雜格式需求)

當需要精確控制 Excel 格式 (欄位寬度、顏色、框線等) 且資料包含繁體中文時,使用此方案。

#### 步驟 1: 建立 UTF-8 資料檔案

建立一個 Tab 分隔的文字檔案 (`testdata.txt`),以 **UTF-8 編碼**儲存:

```
功能名稱	CASE 編號	CASE 主題	系統	測試角色	測試步驟	預期結果
查詢供應商	1-1-1	成功登入系統	WEB	TRAIN_USER	於登入畫面使用帳號登入	進入首頁
	1-1-2	進入主檔頁面	WEB	TRAIN_USER	點選選單進入頁面	顯示頁面內容
新增供應商	2-1-1	進入新增頁面	WEB	TRAIN_USER	點擊「新增」按鈕	進入編輯頁面
```

**注意**:
- 使用 Tab (`\t`) 作為分隔符號
- 檔案必須以 **UTF-8** 編碼儲存 (使用 Notepad++, VS Code 等工具)
- 每行代表一筆資料

#### 步驟 2: 建立 PowerShell 腳本

建立一個**純英文**的 PowerShell 腳本 (`create_excel.ps1`):

```powershell
# create_excel.ps1 (腳本內容不包含中文,避免編碼問題)

# Create Excel COM Object
$excel = New-Object -ComObject Excel.Application
$excel.Visible = $false
$excel.DisplayAlerts = $false

# Create new workbook
$wb = $excel.Workbooks.Add()
$ws = $wb.Sheets.Item(1)
$ws.Name = "TestScript"

# CRITICAL: Set column 2 (CASE編號) as TEXT format to prevent date conversion
$ws.Columns.Item(2).NumberFormat = "@"

# Read UTF-8 data file
$dataPath = "C:\path\to\testdata.txt"
$content = Get-Content -Path $dataPath -Encoding UTF8
$rowNum = 1

# Write data to Excel
foreach ($line in $content) {
    $cols = $line -split "`t"
    $colNum = 1
    foreach ($cell in $cols) {
        $ws.Cells.Item($rowNum, $colNum).Value2 = $cell
        $colNum++
    }
    $rowNum++
}

# Set column widths
$ws.Columns.Item(1).ColumnWidth = 14
$ws.Columns.Item(2).ColumnWidth = 10
$ws.Columns.Item(3).ColumnWidth = 26
$ws.Columns.Item(4).ColumnWidth = 6
$ws.Columns.Item(5).ColumnWidth = 12
$ws.Columns.Item(6).ColumnWidth = 80
$ws.Columns.Item(7).ColumnWidth = 65

# Enable text wrap for long columns
$ws.Columns.Item(6).WrapText = $true
$ws.Columns.Item(7).WrapText = $true

# Format header row (bold + background color)
$headerRange = $ws.Range("A1:G1")
$headerRange.Font.Bold = $true
$headerRange.Interior.Color = 12566463  # Light blue

# Add borders
$usedRange = $ws.UsedRange
$usedRange.Borders.LineStyle = 1

# Freeze first row
$ws.Activate()
$excel.ActiveWindow.SplitRow = 1
$excel.ActiveWindow.FreezePanes = $true

# Save as xlsx (51 = xlOpenXMLWorkbook)
$xlsxPath = "C:\path\to\output.xlsx"
$wb.SaveAs($xlsxPath, 51)

# Close and release COM objects
$wb.Close($false)
$excel.Quit()
[System.Runtime.Interopservices.Marshal]::ReleaseComObject($excel) | Out-Null

Write-Host "Excel file created: $xlsxPath"
```

#### 步驟 3: 執行腳本

```powershell
# 在 PowerShell 中執行
powershell -ExecutionPolicy Bypass -File "C:\path\to\create_excel.ps1"
```

### 關鍵技術要點

#### 1. 防止日期自動轉換 ⭐
```powershell
# 在寫入資料前,先將欄位格式設為文字 (@)
$ws.Columns.Item(2).NumberFormat = "@"
```

這會防止 Excel 將 "1-1-1", "2-3-4" 等編號誤認為日期。

#### 2. 正確讀取 UTF-8 檔案 ⭐
```powershell
# 必須指定 -Encoding UTF8
$content = Get-Content -Path "file.txt" -Encoding UTF8
```

#### 3. Tab 分隔符號處理
```powershell
# PowerShell 中使用反引號 (`) 表示 Tab
$cols = $line -split "`t"
```

#### 4. Excel 檔案格式代碼
```powershell
$wb.SaveAs($path, 51)  # 51 = xlsx (Excel 2007+)
$wb.SaveAs($path, 56)  # 56 = xls (Excel 97-2003)
$wb.SaveAs($path, 6)   # 6 = csv
```

#### 5. 常用顏色代碼
| 代碼 | 顏色 | 用途 |
|------|------|------|
| 12566463 | 淺藍色 | 標題列背景 |
| 65535 | 黃色 | 重點標註 |
| 255 | 紅色 | 錯誤/警告 |
| 5287936 | 綠色 | 成功狀態 |

### 方案比較

| 方案 | 優點 | 缺點 | 適用場景 |
|------|------|------|---------|
| **Python pandas** | 簡單快速、跨平台、適合大量資料 | 無法精確控制格式 | 一般資料轉換、批次處理 |
| **PowerShell + COM** | 完全控制 Excel 格式、欄寬、顏色 | 僅 Windows、需安裝 Excel | 需要特定格式的報表、測試腳本 |

### 常見問題排解

#### Q1: 中文顯示為亂碼
**原因**: 編碼不正確
**解決方案**:
- CSV: 使用 `encoding='utf-8-sig'` (UTF-8 with BOM)
- PowerShell: 確保資料檔案以 UTF-8 儲存,並使用 `-Encoding UTF8` 讀取

#### Q2: CASE 編號 (1-1-1) 變成日期 (Jan-01-01)
**原因**: Excel 自動將連字號格式的數字解釋為日期
**解決方案**:
- pandas: 讀取時指定 `dtype={'CASE編號': str}`
- PowerShell: 寫入前設定欄位格式 `$ws.Columns.Item(2).NumberFormat = "@"`

#### Q3: PowerShell 腳本執行失敗
**原因**: 腳本包含中文字符或編碼問題
**解決方案**:
- 腳本內容只使用英文和符號
- 中文資料放在獨立的 UTF-8 文字檔案中
- 使用 `-ExecutionPolicy Bypass` 執行

#### Q4: pandas 讀取 CSV 時中文亂碼
**解決方案**: 嘗試不同編碼
```python
# 嘗試 UTF-8 with BOM (Excel 匯出的 CSV 通常是這個)
df = pd.read_csv('file.csv', encoding='utf-8-sig')

# 如果是舊版 Excel (台灣),可能是 Big5
df = pd.read_csv('file.csv', encoding='big5')

# 如果是中國大陸,可能是 GBK
df = pd.read_csv('file.csv', encoding='gbk')
```

### 實際範例: 測試腳本轉 Excel

假設你有一個包含測試案例的文字檔 (`testcases.txt`):

```
功能名稱	CASE 編號	測試步驟	預期結果
登入功能	1-1-1	輸入正確帳密	成功登入
	1-1-2	輸入錯誤密碼	顯示錯誤訊息
查詢功能	2-1-1	輸入查詢條件	顯示查詢結果
```

**使用 pandas 轉換 (推薦):**
```python
import pandas as pd

# 讀取 Tab 分隔的文字檔
df = pd.read_csv('testcases.txt', sep='\t', dtype={'CASE 編號': str})

# 寫入 Excel,保留中文
df.to_excel('測試案例.xlsx', index=False)

print("轉換完成!")
```

**使用 PowerShell (需要格式控制時):**
```powershell
# 使用前面的 create_excel.ps1 腳本,修改路徑
$dataPath = "C:\Users\a0304\Desktop\testcases.txt"
$xlsxPath = "C:\Users\a0304\Desktop\測試案例.xlsx"

# 執行腳本...
```

## Best Practices

### Library Selection
- **pandas**: Use for data conversions, cleaning, and transformations (95% of cases)
- **openpyxl**: Use only when you need to preserve Excel formatting or work with formulas

### Data Quality
- Always remove duplicates before converting
- Handle missing values appropriately (fill, drop, or flag)
- Validate data types before conversion
- Check for encoding issues with non-ASCII characters

### Performance Optimization
- Use `chunksize` for large files
- Specify `dtype` when reading to avoid type inference
- Use `usecols` to read only needed columns
- Consider using `pyarrow` engine for faster CSV reading

### File Handling
- Always use `encoding='utf-8'` for CSV files
- Set `index=False` when writing to avoid index column
- Use `force_ascii=False` for JSON with non-ASCII characters
- Create output directories before writing files

### Error Handling
- Wrap file operations in try-except blocks
- Validate file existence before reading
- Check for empty DataFrames before writing
- Log conversion progress for batch operations

## Quick Reference

| Task | Code |
|------|------|
| Excel → CSV | `pd.read_excel('in.xlsx').to_csv('out.csv', index=False)` |
| CSV → Excel | `pd.read_csv('in.csv').to_excel('out.xlsx', index=False)` |
| Excel → JSON | `pd.read_excel('in.xlsx').to_json('out.json', orient='records')` |
| JSON → Excel | `pd.DataFrame(json.load(f)).to_excel('out.xlsx', index=False)` |
| CSV → JSON | `pd.read_csv('in.csv').to_json('out.json', orient='records')` |
| JSON → CSV | `pd.DataFrame(json.load(f)).to_csv('out.csv', index=False)` |
| Merge files | `pd.concat([pd.read_excel(f) for f in files]).to_excel('merged.xlsx')` |
| Remove duplicates | `df.drop_duplicates().to_excel('out.xlsx', index=False)` |

## Keywords

**English keywords:**
excel converter, csv converter, json converter, format conversion, excel to csv, csv to excel, excel to json, json to excel, csv to json, json to csv, data transformation, data conversion, file conversion, batch conversion, merge excel, merge csv, data processing, tabular data, spreadsheet conversion

**Chinese keywords (中文關鍵詞):**
Excel轉換, CSV轉換, JSON轉換, 格式轉換, Excel轉CSV, CSV轉Excel, Excel轉JSON, JSON轉Excel, CSV轉JSON, JSON轉CSV, 資料轉換, 檔案轉換, 批量轉換, 合併Excel, 合併CSV, 資料處理, 表格轉換, 數據轉換
