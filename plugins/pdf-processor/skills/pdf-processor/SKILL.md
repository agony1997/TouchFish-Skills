---
name: pdf-processor
description: "Comprehensive PDF manipulation toolkit for merging, splitting, extracting text/tables, creating PDFs, adding watermarks, and handling forms. When Claude needs to: (1) Merge multiple PDFs, (2) Split PDF into pages, (3) Extract text or tables from PDF, (4) Create new PDFs, (5) Add watermarks or annotations, (6) Rotate or reorder pages, (7) Extract images, (8) Password protect PDFs, or (9) PDF合併、分割、提取、建立、文字提取、表格提取、加密、浮水印"
---

# PDF Processor

## Overview

This skill provides comprehensive PDF processing capabilities including merging, splitting, text/table extraction, PDF creation, watermarking, and form handling. It uses Python libraries (pypdf, pdfplumber, reportlab) and command-line tools (qpdf, pdftotext) for robust PDF manipulation.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Merging or combining multiple PDF files
- Splitting PDF into separate pages or sections
- Extracting text, tables, or images from PDF
- Creating new PDF documents from scratch
- Adding watermarks or annotations to PDFs
- Rotating, reordering, or manipulating PDF pages
- Password protecting or encrypting PDFs
- Extracting metadata from PDFs
- Converting documents to PDF format

**Trigger phrases include:**
- "Merge PDFs" / "合併PDF"
- "Split PDF" / "分割PDF"
- "Extract text from PDF" / "從PDF提取文字"
- "Extract tables from PDF" / "從PDF提取表格"
- "Create PDF" / "建立PDF"
- "Add watermark" / "加浮水印"
- "Rotate PDF pages" / "旋轉PDF頁面"
- "Password protect PDF" / "加密PDF"
- "Combine PDF files" / "組合PDF檔案"
- "PDF to text" / "PDF轉文字"

## How to use this skill

### Workflow Overview

This skill follows a systematic approach based on the operation type:

1. **Identify Operation** - Determine what PDF operation is needed
2. **Choose Tool** - Select appropriate library or command-line tool
3. **Process PDF** - Execute the operation with proper error handling
4. **Validate Output** - Verify the output PDF is correct and accessible

### Python Libraries

#### pypdf - Basic PDF Operations

Use pypdf for merging, splitting, rotating, and basic manipulations:

```python
from pypdf import PdfReader, PdfWriter

# Read PDF
reader = PdfReader("document.pdf")
print(f"Total pages: {len(reader.pages)}")

# Extract text
text = ""
for page in reader.pages:
    text += page.extract_text()
```

#### pdfplumber - Text and Table Extraction

Use pdfplumber for accurate text extraction and table parsing:

```python
import pdfplumber

# Extract text
with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        print(text)

# Extract tables
with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            print(table)
```

#### reportlab - Create PDFs

Use reportlab to create new PDF documents with text, shapes, and images:

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("output.pdf", pagesize=letter)
c.drawString(100, 750, "Hello, PDF!")
c.save()
```

## Common PDF Operations

### Merge Multiple PDFs

```python
from pypdf import PdfWriter, PdfReader

# Merge PDFs
merger = PdfWriter()

files = ["file1.pdf", "file2.pdf", "file3.pdf"]
for pdf_file in files:
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        merger.add_page(page)

# Write merged PDF
with open("merged.pdf", "wb") as output:
    merger.write(output)

print("PDFs merged successfully!")
```

### Split PDF into Pages

```python
from pypdf import PdfReader, PdfWriter

# Split PDF into individual pages
reader = PdfReader("input.pdf")

for i, page in enumerate(reader.pages, start=1):
    writer = PdfWriter()
    writer.add_page(page)

    with open(f"page_{i}.pdf", "wb") as output:
        writer.write(output)

print(f"Split into {len(reader.pages)} pages")
```

### Split PDF by Page Range

```python
from pypdf import PdfReader, PdfWriter

def split_pdf(input_pdf, start_page, end_page, output_pdf):
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for i in range(start_page - 1, end_page):
        writer.add_page(reader.pages[i])

    with open(output_pdf, "wb") as output:
        writer.write(output)

# Extract pages 1-5
split_pdf("input.pdf", 1, 5, "pages_1_5.pdf")

# Extract pages 10-20
split_pdf("input.pdf", 10, 20, "pages_10_20.pdf")
```

### Extract Text from PDF

```python
import pdfplumber

# Extract all text
with pdfplumber.open("document.pdf") as pdf:
    full_text = ""
    for page in pdf.pages:
        full_text += page.extract_text() + "\n\n"

# Save to text file
with open("extracted_text.txt", "w", encoding="utf-8") as f:
    f.write(full_text)

print("Text extracted successfully!")
```

### Extract Text with Layout Preservation

```python
import pdfplumber

# Extract text preserving layout
with pdfplumber.open("document.pdf") as pdf:
    for i, page in enumerate(pdf.pages, start=1):
        text = page.extract_text(layout=True)

        with open(f"page_{i}_text.txt", "w", encoding="utf-8") as f:
            f.write(text)

print("Text extracted with layout preserved")
```

### Extract Tables from PDF

```python
import pdfplumber
import pandas as pd

# Extract all tables
with pdfplumber.open("document.pdf") as pdf:
    all_tables = []

    for i, page in enumerate(pdf.pages, start=1):
        tables = page.extract_tables()

        for j, table in enumerate(tables, start=1):
            if table:
                # Convert to DataFrame
                df = pd.DataFrame(table[1:], columns=table[0])
                all_tables.append(df)

                # Save each table
                df.to_excel(f"page_{i}_table_{j}.xlsx", index=False)

    # Combine all tables
    if all_tables:
        combined = pd.concat(all_tables, ignore_index=True)
        combined.to_excel("all_tables.xlsx", index=False)

print(f"Extracted {len(all_tables)} tables")
```

### Create Simple PDF

```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("simple.pdf", pagesize=letter)
width, height = letter

# Add text
c.drawString(100, height - 100, "Simple PDF Document")
c.drawString(100, height - 120, "Created with reportlab")

# Add line
c.line(100, height - 140, 400, height - 140)

# Add rectangle
c.rect(100, height - 200, 200, 50)

# Save PDF
c.save()
print("PDF created successfully!")
```

### Create Multi-Page PDF with Formatting

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet

# Create document
doc = SimpleDocTemplate("report.pdf", pagesize=letter)
styles = getSampleStyleSheet()
story = []

# Title
title = Paragraph("Annual Report 2024", styles['Title'])
story.append(title)
story.append(Spacer(1, 12))

# Body text
body_text = """
This is the annual report for 2024. The company has achieved significant
milestones this year, including revenue growth and market expansion.
""" * 5

body = Paragraph(body_text, styles['Normal'])
story.append(body)
story.append(PageBreak())

# Second page
story.append(Paragraph("Financial Summary", styles['Heading1']))
story.append(Paragraph("Revenue increased by 25% year-over-year.", styles['Normal']))

# Build PDF
doc.build(story)
print("Multi-page PDF created!")
```

### Add Watermark to PDF

```python
from pypdf import PdfReader, PdfWriter

# Create or load watermark PDF
watermark_reader = PdfReader("watermark.pdf")
watermark_page = watermark_reader.pages[0]

# Apply to all pages
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    page.merge_page(watermark_page)
    writer.add_page(page)

# Save watermarked PDF
with open("watermarked.pdf", "wb") as output:
    writer.write(output)

print("Watermark added successfully!")
```

### Create Text Watermark

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter
from pypdf import PdfReader, PdfWriter
import io

def create_watermark(text):
    packet = io.BytesIO()
    c = canvas.Canvas(packet, pagesize=letter)
    width, height = letter

    # Set watermark style
    c.setFont("Helvetica", 60)
    c.setFillColorRGB(0.5, 0.5, 0.5, alpha=0.3)

    # Rotate and draw text
    c.saveState()
    c.translate(width/2, height/2)
    c.rotate(45)
    c.drawCentredString(0, 0, text)
    c.restoreState()

    c.save()
    packet.seek(0)
    return PdfReader(packet).pages[0]

# Apply watermark
watermark = create_watermark("CONFIDENTIAL")
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    page.merge_page(watermark)
    writer.add_page(page)

with open("watermarked.pdf", "wb") as output:
    writer.write(output)
```

### Rotate PDF Pages

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Rotate specific pages
for i, page in enumerate(reader.pages):
    if i == 0:  # Rotate first page 90 degrees
        page.rotate(90)
    elif i == 1:  # Rotate second page 180 degrees
        page.rotate(180)
    writer.add_page(page)

with open("rotated.pdf", "wb") as output:
    writer.write(output)
```

### Rotate All Pages

```python
from pypdf import PdfReader, PdfWriter

def rotate_pdf(input_pdf, output_pdf, rotation):
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for page in reader.pages:
        page.rotate(rotation)  # 90, 180, 270
        writer.add_page(page)

    with open(output_pdf, "wb") as output:
        writer.write(output)

# Rotate all pages 90 degrees clockwise
rotate_pdf("input.pdf", "rotated.pdf", 90)
```

### Password Protect PDF

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("document.pdf")
writer = PdfWriter()

# Add all pages
for page in reader.pages:
    writer.add_page(page)

# Encrypt with password
writer.encrypt(
    user_password="user123",      # Password to open
    owner_password="owner456",     # Password to modify
    permissions_flag=0b0100        # Allow printing only
)

with open("encrypted.pdf", "wb") as output:
    writer.write(output)

print("PDF password protected!")
```

### Remove Password from PDF

```python
from pypdf import PdfReader, PdfWriter

# Open encrypted PDF with password
reader = PdfReader("encrypted.pdf", password="user123")
writer = PdfWriter()

# Copy all pages (now decrypted)
for page in reader.pages:
    writer.add_page(page)

# Save without encryption
with open("decrypted.pdf", "wb") as output:
    writer.write(output)

print("Password removed!")
```

### Extract Metadata

```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
meta = reader.metadata

print(f"Title: {meta.title}")
print(f"Author: {meta.author}")
print(f"Subject: {meta.subject}")
print(f"Creator: {meta.creator}")
print(f"Producer: {meta.producer}")
print(f"Creation Date: {meta.creation_date}")
```

### Extract Images from PDF

```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")

for page_number, page in enumerate(reader.pages):
    # Extract images from page
    for image_number, image in enumerate(page.images):
        with open(f"page_{page_number+1}_image_{image_number+1}.{image.name.split('.')[-1]}", "wb") as f:
            f.write(image.data)

print("Images extracted!")
```

## Command-Line Tools

### Using qpdf

```bash
# Merge PDFs
qpdf --empty --pages file1.pdf file2.pdf file3.pdf -- merged.pdf

# Split PDF (extract pages 1-5)
qpdf input.pdf --pages . 1-5 -- pages_1_5.pdf

# Rotate pages (rotate page 1 by 90 degrees)
qpdf input.pdf output.pdf --rotate=+90:1

# Remove password
qpdf --password=mypassword --decrypt encrypted.pdf decrypted.pdf

# Compress PDF
qpdf --compress-streams=y input.pdf compressed.pdf
```

### Using pdftotext

```bash
# Extract text
pdftotext document.pdf output.txt

# Extract text preserving layout
pdftotext -layout document.pdf output.txt

# Extract specific pages (pages 1-5)
pdftotext -f 1 -l 5 document.pdf output.txt

# Extract with encoding
pdftotext -enc UTF-8 document.pdf output.txt
```

## Advanced Operations

### OCR for Scanned PDFs

```python
import pytesseract
from pdf2image import convert_from_path

# Convert PDF pages to images
images = convert_from_path('scanned.pdf', dpi=300)

# OCR each page
full_text = ""
for i, image in enumerate(images, start=1):
    text = pytesseract.image_to_string(image, lang='eng')
    full_text += f"--- Page {i} ---\n{text}\n\n"

# Save extracted text
with open("ocr_output.txt", "w", encoding="utf-8") as f:
    f.write(full_text)

print("OCR completed!")
```

### Batch Process PDFs

```python
from pypdf import PdfReader, PdfWriter
import os
from pathlib import Path

def batch_merge_pdfs(input_dir, output_file):
    merger = PdfWriter()

    # Get all PDF files
    pdf_files = sorted([f for f in os.listdir(input_dir) if f.endswith('.pdf')])

    for pdf_file in pdf_files:
        path = os.path.join(input_dir, pdf_file)
        reader = PdfReader(path)
        for page in reader.pages:
            merger.add_page(page)
        print(f"Added: {pdf_file}")

    with open(output_file, "wb") as output:
        merger.write(output)

    print(f"Merged {len(pdf_files)} files into {output_file}")

# Usage
batch_merge_pdfs("pdf_folder", "combined.pdf")
```

### Extract Specific Pages

```python
from pypdf import PdfReader, PdfWriter

def extract_pages(input_pdf, pages_to_extract, output_pdf):
    """
    Extract specific pages from PDF
    pages_to_extract: list of page numbers (1-indexed)
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for page_num in pages_to_extract:
        writer.add_page(reader.pages[page_num - 1])

    with open(output_pdf, "wb") as output:
        writer.write(output)

# Extract pages 1, 3, 5, 7
extract_pages("input.pdf", [1, 3, 5, 7], "extracted.pdf")

# Extract odd pages
odd_pages = list(range(1, len(PdfReader("input.pdf").pages) + 1, 2))
extract_pages("input.pdf", odd_pages, "odd_pages.pdf")
```

### Reorder Pages

```python
from pypdf import PdfReader, PdfWriter

def reorder_pages(input_pdf, page_order, output_pdf):
    """
    Reorder PDF pages
    page_order: list of page numbers in desired order (1-indexed)
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()

    for page_num in page_order:
        writer.add_page(reader.pages[page_num - 1])

    with open(output_pdf, "wb") as output:
        writer.write(output)

# Reorder: pages 3, 1, 2, 4
reorder_pages("input.pdf", [3, 1, 2, 4], "reordered.pdf")
```

### Convert HTML to PDF

```python
from weasyprint import HTML

# Convert HTML file to PDF
HTML('input.html').write_pdf('output.pdf')

# Convert HTML string to PDF
html_string = """
<html>
<head><title>Test PDF</title></head>
<body>
<h1>Hello, PDF!</h1>
<p>This is a test PDF created from HTML.</p>
</body>
</html>
"""
HTML(string=html_string).write_pdf('from_html.pdf')
```

## Best Practices

### Library Selection
- **pypdf**: Use for merging, splitting, rotating, and basic manipulations
- **pdfplumber**: Use for accurate text and table extraction
- **reportlab**: Use for creating new PDFs from scratch
- **qpdf**: Use for command-line operations and compression

### Performance Optimization
- Use `read_only=True` for large PDFs when only reading
- Process pages in batches for memory efficiency
- Use command-line tools for very large files
- Close file handles explicitly when done

### Error Handling
- Always validate PDF files exist before processing
- Wrap operations in try-except blocks
- Check for corrupted or password-protected PDFs
- Validate page numbers are within range

### Output Quality
- Set appropriate DPI for image extraction (300 for print quality)
- Use compression when creating PDFs to reduce file size
- Preserve metadata when possible
- Test output PDFs in different viewers

## Quick Reference

| Task | Tool | Code/Command |
|------|------|--------------|
| Merge PDFs | pypdf | `merger.add_page(page)` |
| Split PDF | pypdf | Create writer per page |
| Extract text | pdfplumber | `page.extract_text()` |
| Extract tables | pdfplumber | `page.extract_tables()` |
| Create PDF | reportlab | `canvas.Canvas()` |
| Add watermark | pypdf | `page.merge_page(watermark)` |
| Rotate pages | pypdf | `page.rotate(90)` |
| Password protect | pypdf | `writer.encrypt()` |
| Command merge | qpdf | `qpdf --empty --pages ...` |
| OCR scanned PDF | pytesseract | Convert to image first |

## Keywords

**English keywords:**
pdf processor, merge pdf, split pdf, extract text, extract tables, create pdf, pdf watermark, rotate pdf, password protect pdf, pdf manipulation, combine pdf, pdf to text, pdf encryption, pdf images, batch pdf processing

**Chinese keywords (中文關鍵詞):**
PDF處理, 合併PDF, 分割PDF, 提取文字, 提取表格, 建立PDF, PDF浮水印, 旋轉PDF, 加密PDF, PDF操作, 組合PDF, PDF轉文字, PDF加密, PDF圖片, 批量處理PDF
