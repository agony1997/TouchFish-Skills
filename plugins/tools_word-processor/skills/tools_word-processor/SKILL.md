---
name: tools_word-processor
description: "Comprehensive Word document creation, editing, and conversion toolkit. When Claude needs to: (1) Create new Word documents, (2) Edit existing DOCX files, (3) Add formatting and styles, (4) Insert tables, images, and charts, (5) Convert between formats (DOCX/PDF/HTML), (6) Manage document structure, (7) Add headers/footers, (8) Process templates, or (9) Word文檔建立、編輯、轉換、格式化、表格插入、圖片處理"
---

# Word Processor

## Overview

This skill provides comprehensive Word document processing capabilities including creating new documents, editing existing files, formatting text, inserting tables/images, and converting between formats. It uses python-docx library for document manipulation and pandoc for format conversion.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Creating new Word documents
- Editing existing DOCX files
- Formatting Word documents (fonts, styles, paragraphs)
- Inserting tables, images, or charts into Word
- Converting Word to other formats (PDF, HTML, Markdown)
- Converting other formats to Word
- Managing document headers, footers, and page numbers
- Creating Word templates
- Processing multiple Word documents

**Trigger phrases include:**
- "Create Word document" / "建立Word文檔"
- "Edit DOCX file" / "編輯DOCX檔案"
- "Add table to Word" / "在Word中加入表格"
- "Insert image in Word" / "在Word中插入圖片"
- "Convert Word to PDF" / "將Word轉為PDF"
- "Format Word document" / "格式化Word文檔"
- "Add header/footer" / "加入頁首/頁尾"
- "Word template" / "Word範本"
- "Generate Word report" / "生成Word報告"

## How to use this skill

### Workflow Overview

This skill follows a systematic approach based on the operation type:

1. **Identify Operation** - Determine if creating new, editing existing, or converting
2. **Choose Tool** - Select python-docx for editing or pandoc for conversion
3. **Process Document** - Execute operations with proper formatting
4. **Save Output** - Write document with correct encoding and format

### Python Library: python-docx

#### Installation

```bash
pip install python-docx
```

#### Basic Usage

```python
from docx import Document

# Create new document
doc = Document()

# Add content
doc.add_heading('Document Title', 0)
doc.add_paragraph('This is a paragraph.')

# Save document
doc.save('output.docx')
```

## Creating Word Documents

### Create Simple Document

```python
from docx import Document

# Create new document
doc = Document()

# Add title
doc.add_heading('Project Report', 0)

# Add subtitle
doc.add_heading('Executive Summary', level=1)

# Add paragraphs
doc.add_paragraph('This is the executive summary of the project.')
doc.add_paragraph('The project was completed successfully.')

# Add section
doc.add_heading('Project Details', level=1)
doc.add_paragraph('Detailed information about the project...')

# Save
doc.save('project_report.docx')
print("Document created successfully!")
```

### Add Formatted Text

```python
from docx import Document
from docx.shared import Pt, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH

doc = Document()

# Add heading with formatting
heading = doc.add_heading('Formatted Document', 0)

# Add paragraph with custom formatting
p = doc.add_paragraph()
run = p.add_run('This is bold text. ')
run.bold = True

run = p.add_run('This is italic text. ')
run.italic = True

run = p.add_run('This is underlined text. ')
run.underline = True

# Add colored text
p = doc.add_paragraph()
run = p.add_run('This is red text.')
run.font.color.rgb = RGBColor(255, 0, 0)
run.font.size = Pt(14)

# Add centered paragraph
p = doc.add_paragraph('This paragraph is centered.')
p.alignment = WD_ALIGN_PARAGRAPH.CENTER

doc.save('formatted_document.docx')
```

### Insert Tables

```python
from docx import Document

doc = Document()
doc.add_heading('Sales Report', 0)

# Create table (3 rows, 4 columns)
table = doc.add_table(rows=3, cols=4)
table.style = 'Light Grid Accent 1'

# Add header row
header_cells = table.rows[0].cells
header_cells[0].text = 'Product'
header_cells[1].text = 'Q1'
header_cells[2].text = 'Q2'
header_cells[3].text = 'Total'

# Add data rows
data = [
    ['Product A', '1000', '1200', '2200'],
    ['Product B', '800', '900', '1700']
]

for i, row_data in enumerate(data, start=1):
    cells = table.rows[i].cells
    for j, value in enumerate(row_data):
        cells[j].text = value

doc.save('table_document.docx')
```

### Insert Dynamic Tables from Data

```python
from docx import Document
import pandas as pd

# Read data
df = pd.read_csv('sales_data.csv')

doc = Document()
doc.add_heading('Sales Data Report', 0)

# Create table from DataFrame
table = doc.add_table(rows=len(df) + 1, cols=len(df.columns))
table.style = 'Medium Shading 1 Accent 1'

# Add headers
for i, column in enumerate(df.columns):
    table.rows[0].cells[i].text = column

# Add data
for i, row in df.iterrows():
    for j, value in enumerate(row):
        table.rows[i + 1].cells[j].text = str(value)

doc.save('data_report.docx')
```

### Insert Images

```python
from docx import Document
from docx.shared import Inches

doc = Document()
doc.add_heading('Image Gallery', 0)

# Add image with specific width
doc.add_paragraph('Company Logo:')
doc.add_picture('logo.png', width=Inches(2))

# Add image with caption
doc.add_paragraph('Project Chart:')
doc.add_picture('chart.png', width=Inches(5))
doc.add_paragraph('Figure 1: Sales Chart', style='Caption')

# Add multiple images
doc.add_heading('Image Grid', level=1)
images = ['image1.png', 'image2.png', 'image3.png']
for img in images:
    doc.add_picture(img, width=Inches(2.5))

doc.save('image_document.docx')
```

### Add Headers and Footers

```python
from docx import Document
from docx.shared import Pt

doc = Document()

# Access header
section = doc.sections[0]
header = section.header
header_para = header.paragraphs[0]
header_para.text = "Company Name - Confidential"
header_para.style.font.size = Pt(9)

# Access footer
footer = section.footer
footer_para = footer.paragraphs[0]
footer_para.text = "Page "  # Page number will be added automatically

# Add content
doc.add_heading('Document with Header and Footer', 0)
doc.add_paragraph('This document has custom header and footer.')

doc.save('header_footer_document.docx')
```

### Create Multi-Section Document

```python
from docx import Document
from docx.shared import Inches
from docx.enum.section import WD_SECTION

doc = Document()

# Section 1
doc.add_heading('Section 1: Introduction', 0)
doc.add_paragraph('This is the introduction section.')

# Add page break and new section
doc.add_page_break()

# Section 2
doc.add_heading('Section 2: Main Content', 0)
doc.add_paragraph('This is the main content section.')

# Add section break
doc.add_section(WD_SECTION.NEW_PAGE)

# Section 3
doc.add_heading('Section 3: Conclusion', 0)
doc.add_paragraph('This is the conclusion section.')

doc.save('multi_section_document.docx')
```

## Editing Existing Documents

### Read and Modify Existing Document

```python
from docx import Document

# Open existing document
doc = Document('existing_document.docx')

# Read all paragraphs
for para in doc.paragraphs:
    print(para.text)

# Add new content
doc.add_heading('New Section', level=1)
doc.add_paragraph('This is new content added to the document.')

# Save modified document
doc.save('modified_document.docx')
```

### Find and Replace Text

```python
from docx import Document

def replace_text(doc, old_text, new_text):
    """Replace text in all paragraphs and tables"""
    # Replace in paragraphs
    for para in doc.paragraphs:
        if old_text in para.text:
            for run in para.runs:
                run.text = run.text.replace(old_text, new_text)

    # Replace in tables
    for table in doc.tables:
        for row in table.rows:
            for cell in row.cells:
                if old_text in cell.text:
                    for para in cell.paragraphs:
                        for run in para.runs:
                            run.text = run.text.replace(old_text, new_text)

# Open and modify document
doc = Document('template.docx')
replace_text(doc, '{{NAME}}', 'John Doe')
replace_text(doc, '{{DATE}}', '2024-01-15')
replace_text(doc, '{{AMOUNT}}', '$1,000')
doc.save('filled_template.docx')
```

### Extract Text from Document

```python
from docx import Document

def extract_text(docx_path):
    """Extract all text from Word document"""
    doc = Document(docx_path)

    full_text = []

    # Extract from paragraphs
    for para in doc.paragraphs:
        full_text.append(para.text)

    # Extract from tables
    for table in doc.tables:
        for row in table.rows:
            row_text = [cell.text for cell in row.cells]
            full_text.append('\t'.join(row_text))

    return '\n'.join(full_text)

# Extract and save
text = extract_text('document.docx')
with open('extracted_text.txt', 'w', encoding='utf-8') as f:
    f.write(text)
```

### Modify Table Content

```python
from docx import Document

# Open document
doc = Document('document_with_tables.docx')

# Access first table
table = doc.tables[0]

# Modify specific cell
table.rows[1].cells[2].text = 'Updated Value'

# Add new row
new_row = table.add_row()
new_row.cells[0].text = 'New Item'
new_row.cells[1].text = '100'
new_row.cells[2].text = '200'

# Save modified document
doc.save('modified_table_document.docx')
```

## Document Conversion

### Convert Word to PDF

```bash
# Using LibreOffice command line
soffice --headless --convert-to pdf document.docx

# Using pandoc
pandoc document.docx -o document.pdf
```

### Convert Word to HTML

```bash
# Using pandoc
pandoc document.docx -o document.html

# With custom CSS
pandoc document.docx -o document.html --css=style.css
```

### Convert Word to Markdown

```bash
# Using pandoc
pandoc document.docx -o document.md

# With specific markdown flavor
pandoc document.docx -o document.md --markdown-headings=atx
```

### Convert Markdown to Word

```bash
# Using pandoc
pandoc document.md -o document.docx

# With reference document for styling
pandoc document.md -o document.docx --reference-doc=template.docx
```

### Convert HTML to Word

```bash
# Using pandoc
pandoc document.html -o document.docx

# With metadata
pandoc document.html -o document.docx --metadata title="Document Title"
```

### Batch Convert Documents

```python
import os
import subprocess
from pathlib import Path

def batch_convert_to_pdf(input_dir, output_dir):
    """Convert all DOCX files to PDF"""
    Path(output_dir).mkdir(exist_ok=True)

    for filename in os.listdir(input_dir):
        if filename.endswith('.docx'):
            input_path = os.path.join(input_dir, filename)
            output_path = os.path.join(output_dir, filename.replace('.docx', '.pdf'))

            # Convert using LibreOffice
            subprocess.run([
                'soffice',
                '--headless',
                '--convert-to', 'pdf',
                '--outdir', output_dir,
                input_path
            ])
            print(f"Converted: {filename}")

# Usage
batch_convert_to_pdf('docx_files', 'pdf_files')
```

## Advanced Features

### Create Document from Template

```python
from docx import Document
from datetime import datetime

def fill_template(template_path, output_path, data):
    """Fill Word template with data"""
    doc = Document(template_path)

    # Replace placeholders
    for key, value in data.items():
        placeholder = f'{{{{{key}}}}}'

        # Replace in paragraphs
        for para in doc.paragraphs:
            if placeholder in para.text:
                for run in para.runs:
                    run.text = run.text.replace(placeholder, str(value))

        # Replace in tables
        for table in doc.tables:
            for row in table.rows:
                for cell in row.cells:
                    if placeholder in cell.text:
                        for para in cell.paragraphs:
                            for run in para.runs:
                                run.text = run.text.replace(placeholder, str(value))

    doc.save(output_path)

# Usage
data = {
    'CUSTOMER_NAME': 'John Doe',
    'DATE': datetime.now().strftime('%Y-%m-%d'),
    'AMOUNT': '$1,500',
    'PROJECT': 'Website Development'
}

fill_template('invoice_template.docx', 'invoice_2024.docx', data)
```

### Add List (Bullets and Numbering)

```python
from docx import Document

doc = Document()
doc.add_heading('To-Do List', 0)

# Add bulleted list
doc.add_paragraph('Task 1', style='List Bullet')
doc.add_paragraph('Task 2', style='List Bullet')
doc.add_paragraph('Task 3', style='List Bullet')

# Add numbered list
doc.add_heading('Steps', level=1)
doc.add_paragraph('Step 1: Prepare', style='List Number')
doc.add_paragraph('Step 2: Execute', style='List Number')
doc.add_paragraph('Step 3: Review', style='List Number')

doc.save('list_document.docx')
```

### Add Table of Contents

```python
from docx import Document
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

def add_toc(doc):
    """Add table of contents"""
    paragraph = doc.add_paragraph()
    run = paragraph.add_run()

    # Create TOC field
    fldChar = OxmlElement('w:fldChar')
    fldChar.set(qn('w:fldCharType'), 'begin')

    instrText = OxmlElement('w:instrText')
    instrText.set(qn('xml:space'), 'preserve')
    instrText.text = 'TOC \\o "1-3" \\h \\z \\u'

    fldChar2 = OxmlElement('w:fldChar')
    fldChar2.set(qn('w:fldCharType'), 'separate')

    fldChar3 = OxmlElement('w:t')
    fldChar3.text = "Right-click to update field."

    fldChar4 = OxmlElement('w:fldChar')
    fldChar4.set(qn('w:fldCharType'), 'end')

    r_element = run._r
    r_element.append(fldChar)
    r_element.append(instrText)
    r_element.append(fldChar2)
    r_element.append(fldChar3)
    r_element.append(fldChar4)

# Create document with TOC
doc = Document()
doc.add_heading('Document Title', 0)
add_toc(doc)
doc.add_page_break()

doc.add_heading('Chapter 1', level=1)
doc.add_paragraph('Content for chapter 1...')

doc.add_heading('Chapter 2', level=1)
doc.add_paragraph('Content for chapter 2...')

doc.save('document_with_toc.docx')
```

### Merge Multiple Documents

```python
from docx import Document
from docx.oxml.xmlchemy import OxmlElement

def merge_documents(files, output_path):
    """Merge multiple Word documents"""
    merged_doc = Document(files[0])

    for file in files[1:]:
        sub_doc = Document(file)

        # Add page break
        merged_doc.add_page_break()

        # Copy all elements
        for element in sub_doc.element.body:
            merged_doc.element.body.append(element)

    merged_doc.save(output_path)

# Merge documents
files = ['doc1.docx', 'doc2.docx', 'doc3.docx']
merge_documents(files, 'merged_document.docx')
```

### Apply Custom Styles

```python
from docx import Document
from docx.shared import Pt, RGBColor
from docx.enum.style import WD_STYLE_TYPE

doc = Document()

# Create custom paragraph style
styles = doc.styles
custom_style = styles.add_style('CustomStyle', WD_STYLE_TYPE.PARAGRAPH)

# Set font
font = custom_style.font
font.name = 'Arial'
font.size = Pt(12)
font.color.rgb = RGBColor(0, 0, 139)

# Set paragraph spacing
paragraph_format = custom_style.paragraph_format
paragraph_format.space_before = Pt(12)
paragraph_format.space_after = Pt(12)

# Use custom style
doc.add_heading('Document with Custom Style', 0)
doc.add_paragraph('This paragraph uses the custom style.', style='CustomStyle')

doc.save('custom_style_document.docx')
```

## Best Practices

### Document Creation
- Use consistent heading levels (0 for title, 1-3 for sections)
- Apply built-in styles for better compatibility
- Set proper margins and page sizes
- Use page breaks instead of multiple empty paragraphs

### Text Formatting
- Keep formatting simple and consistent
- Use styles instead of direct formatting
- Prefer built-in paragraph styles (Normal, Heading 1, etc.)
- Set font sizes using Pt() for consistency

### Tables
- Use appropriate table styles
- Set column widths explicitly for large tables
- Merge cells only when necessary
- Keep table headers on each page for long tables

### Images
- Use appropriate image sizes (not too large)
- Set explicit width/height using Inches()
- Compress images before inserting
- Use supported formats (PNG, JPG)

### Conversion
- Use pandoc for format conversions
- Specify reference documents for styling
- Test converted documents in target format
- Handle special characters and encoding properly

### Error Handling
- Validate file paths before processing
- Check document structure before modification
- Handle missing fonts gracefully
- Back up original documents before editing

## 繁體中文處理最佳實踐 (Traditional Chinese Handling)

### python-docx 中文支援

python-docx 對 Unicode (包含繁體中文) 的支援良好,可以直接在程式碼中使用中文:

```python
from docx import Document
from docx.shared import Pt, RGBColor
from docx.oxml.ns import qn

# 建立包含繁體中文的文檔
doc = Document()

# 加入中文標題
doc.add_heading('專案報告', 0)
doc.add_heading('執行摘要', level=1)

# 加入中文段落
doc.add_paragraph('這是一份完整的專案報告。')
doc.add_paragraph('專案已於 2024 年 1 月順利完成。')

# 儲存 (自動處理 Unicode)
doc.save('專案報告.docx')
```

### 設定中文字型

Word 文檔中,中文字體需要特別設定:

```python
from docx import Document
from docx.shared import Pt
from docx.oxml.ns import qn

doc = Document()

# 加入段落
para = doc.add_paragraph('這是繁體中文內容')

# 設定中文字型 (重要!)
for run in para.runs:
    run.font.name = 'Microsoft YaHei'  # 微軟雅黑 (簡體中文)
    # 或
    run.font.name = 'Microsoft JhengHei'  # 微軟正黑體 (繁體中文,推薦)
    # 或
    run.font.name = 'DFKai-SB'  # 標楷體 (台灣常用)

    # 設定東亞字體 (確保中文正確顯示)
    run._element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')

    # 設定字體大小
    run.font.size = Pt(12)

doc.save('中文文檔.docx')
```

### 常用繁體中文字型

| 字型名稱 | 英文名稱 | 適用場景 | Windows 內建 |
|---------|---------|---------|-------------|
| 微軟正黑體 | Microsoft JhengHei | 現代、簡潔 | ✅ |
| 標楷體 | DFKai-SB | 傳統、正式 | ✅ |
| 新細明體 | PMingLiU | 傳統、閱讀 | ✅ |
| 微軟雅黑 | Microsoft YaHei | 簡體為主 | ✅ |
| 思源黑體 | Noto Sans CJK TC | 開源、現代 | ❌ 需安裝 |

### 建立完整的繁體中文文檔範例

```python
from docx import Document
from docx.shared import Pt, Inches, RGBColor
from docx.oxml.ns import qn
from docx.enum.text import WD_PARAGRAPH_ALIGNMENT

def set_chinese_font(run, font_name='Microsoft JhengHei', size=12):
    """設定繁體中文字型"""
    run.font.name = font_name
    run._element.rPr.rFonts.set(qn('w:eastAsia'), font_name)
    run.font.size = Pt(size)

# 建立文檔
doc = Document()

# 設定預設字型 (整份文件)
style = doc.styles['Normal']
font = style.font
font.name = 'Microsoft JhengHei'
style.element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')
font.size = Pt(12)

# 加入標題
title = doc.add_heading('測試案例報告', 0)
title_run = title.runs[0]
set_chinese_font(title_run, 'Microsoft JhengHei', 18)

# 加入子標題
subtitle = doc.add_heading('功能測試結果', level=1)
subtitle_run = subtitle.runs[0]
set_chinese_font(subtitle_run, 'Microsoft JhengHei', 14)

# 加入段落
para = doc.add_paragraph('本次測試涵蓋以下功能模組：')
for run in para.runs:
    set_chinese_font(run)

# 加入項目列表
items = ['登入功能', '查詢功能', '新增功能', '刪除功能']
for item in items:
    para = doc.add_paragraph(item, style='List Bullet')
    for run in para.runs:
        set_chinese_font(run)

# 加入表格
table = doc.add_table(rows=4, cols=4)
table.style = 'Light Grid Accent 1'

# 表格標題
headers = ['功能名稱', 'CASE 編號', '測試步驟', '預期結果']
for i, header in enumerate(headers):
    cell = table.rows[0].cells[i]
    cell.text = header
    for para in cell.paragraphs:
        for run in para.runs:
            set_chinese_font(run)
            run.bold = True

# 表格內容
test_data = [
    ['登入功能', '1-1-1', '輸入正確帳密', '成功登入系統'],
    ['登入功能', '1-1-2', '輸入錯誤密碼', '顯示錯誤訊息'],
    ['查詢功能', '2-1-1', '輸入查詢條件', '顯示查詢結果']
]

for i, row_data in enumerate(test_data, start=1):
    for j, cell_data in enumerate(row_data):
        cell = table.rows[i].cells[j]
        cell.text = cell_data
        for para in cell.paragraphs:
            for run in para.runs:
                set_chinese_font(run)

# 儲存
doc.save('測試案例報告.docx')
print('繁體中文文檔建立完成!')
```

### 從文字檔讀取中文內容

```python
from docx import Document
from docx.oxml.ns import qn

# 讀取 UTF-8 編碼的文字檔
with open('內容.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()

doc = Document()

# 設定預設字型
style = doc.styles['Normal']
font = style.font
font.name = 'Microsoft JhengHei'
style.element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')

# 寫入內容
for line in lines:
    doc.add_paragraph(line.strip())

doc.save('輸出文檔.docx')
```

### 轉換編碼

如果原始檔案編碼不是 UTF-8:

```python
# 嘗試不同編碼
encodings = ['utf-8', 'utf-8-sig', 'big5', 'cp950', 'gbk']

for encoding in encodings:
    try:
        with open('file.txt', 'r', encoding=encoding) as f:
            content = f.read()
        print(f"成功使用 {encoding} 編碼讀取")
        break
    except UnicodeDecodeError:
        continue
```

### Word 轉 PDF (保留中文)

使用 LibreOffice (跨平台,免費):

```python
import subprocess

# Windows
subprocess.run([
    'soffice',
    '--headless',
    '--convert-to', 'pdf',
    '--outdir', 'output',
    'input.docx'
])

# 或使用 python-docx2pdf (僅 Windows)
# pip install docx2pdf
from docx2pdf import convert

convert('中文文檔.docx', '中文文檔.pdf')
```

### 使用範本 (包含中文)

```python
from docx import Document
from docx.oxml.ns import qn

# 開啟範本
doc = Document('範本.docx')

# 替換中文占位符
replacements = {
    '{{客戶名稱}}': '王小明',
    '{{專案名稱}}': '系統開發專案',
    '{{日期}}': '2024/01/27'
}

for para in doc.paragraphs:
    for old, new in replacements.items():
        if old in para.text:
            for run in para.runs:
                run.text = run.text.replace(old, new)

# 確保字型正確
for para in doc.paragraphs:
    for run in para.runs:
        if run.text:
            run.font.name = 'Microsoft JhengHei'
            run._element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')

doc.save('已填寫範本.docx')
```

### 常見問題排解

#### Q1: 中文字顯示為方塊 (□□□)
**原因**: 未設定東亞字體
**解決方案**:
```python
run._element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')
```

#### Q2: 讀取文字檔中文亂碼
**原因**: 編碼不正確
**解決方案**: 使用 `encoding='utf-8'` 或 `encoding='big5'`

#### Q3: 轉 PDF 後中文變亂碼
**原因**: PDF 轉換工具不支援中文字型
**解決方案**: 使用 LibreOffice 或 docx2pdf (Windows)

#### Q4: 字型名稱無效
**原因**: 系統未安裝該字型
**解決方案**:
- 確認字型已安裝: 控制台 → 字型
- 使用系統內建字型 (Microsoft JhengHei, DFKai-SB)

### 推薦字型設定

**正式文件**: 標楷體 (DFKai-SB)
```python
run.font.name = 'DFKai-SB'
run._element.rPr.rFonts.set(qn('w:eastAsia'), 'DFKai-SB')
```

**現代文件**: 微軟正黑體 (Microsoft JhengHei)
```python
run.font.name = 'Microsoft JhengHei'
run._element.rPr.rFonts.set(qn('w:eastAsia'), 'Microsoft JhengHei')
```

**閱讀文件**: 新細明體 (PMingLiU)
```python
run.font.name = 'PMingLiU'
run._element.rPr.rFonts.set(qn('w:eastAsia'), 'PMingLiU')
```

## Quick Reference

| Task | Code |
|------|------|
| Create document | `doc = Document()` |
| Add heading | `doc.add_heading('Title', 0)` |
| Add paragraph | `doc.add_paragraph('Text')` |
| Add table | `doc.add_table(rows=3, cols=4)` |
| Insert image | `doc.add_picture('image.png', width=Inches(2))` |
| Save document | `doc.save('output.docx')` |
| Open existing | `doc = Document('file.docx')` |
| Add page break | `doc.add_page_break()` |
| Bold text | `run.bold = True` |
| Convert to PDF | `soffice --headless --convert-to pdf file.docx` |

## Keywords

**English keywords:**
word processor, create word document, edit docx, word formatting, insert table, insert image, word conversion, docx to pdf, word template, document editing, python-docx, word automation, generate report

**Chinese keywords (中文關鍵詞):**
Word處理, 建立Word文檔, 編輯DOCX, Word格式化, 插入表格, 插入圖片, Word轉換, DOCX轉PDF, Word範本, 文檔編輯, Word自動化, 生成報告
