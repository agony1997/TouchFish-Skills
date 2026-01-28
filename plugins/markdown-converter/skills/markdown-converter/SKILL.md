---
name: markdown-converter
description: "Comprehensive Markdown format conversion toolkit for converting between Markdown and other formats including HTML, PDF, Word (DOCX), and plain text. When Claude needs to: (1) Convert Markdown to HTML/PDF/DOCX, (2) Convert HTML/PDF/DOCX to Markdown, (3) Generate styled documents from Markdown, (4) Extract content to Markdown format, (5) Batch convert documents, or (6) Markdown轉換、格式互轉、HTML轉換、PDF生成、Word轉換、文檔轉換"
---

# Markdown Converter

## Overview

This skill provides comprehensive Markdown conversion capabilities for transforming documents between Markdown and various formats including HTML, PDF, Word (DOCX), and plain text. It primarily uses pandoc as the universal document converter, with additional Python libraries for specific conversion needs.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Converting Markdown to other formats (HTML, PDF, DOCX, etc.)
- Converting other formats to Markdown
- Generating styled PDFs from Markdown
- Creating Word documents from Markdown
- Extracting content as Markdown
- Batch converting document formats
- Publishing Markdown documentation

**Trigger phrases include:**
- "Convert Markdown to HTML/PDF/Word" / "轉換Markdown為HTML/PDF/Word"
- "Convert HTML/PDF/Word to Markdown" / "轉換HTML/PDF/Word為Markdown"
- "Generate PDF from Markdown" / "從Markdown生成PDF"
- "Markdown to DOCX" / "Markdown轉DOCX"
- "Extract to Markdown" / "提取為Markdown"
- "Batch convert Markdown" / "批量轉換Markdown"
- "Publish Markdown documentation" / "發布Markdown文檔"
- "Create HTML from Markdown" / "從Markdown建立HTML"

## How to use this skill

### Workflow Overview

This skill follows a systematic approach based on conversion direction:

1. **Identify Formats** - Determine source and target formats
2. **Choose Tool** - Select pandoc or appropriate Python library
3. **Apply Conversion** - Execute with proper options and styling
4. **Validate Output** - Verify formatting and content preservation

### Primary Tool: pandoc

pandoc is a universal document converter that supports numerous formats.

#### Installation

```bash
# Ubuntu/Debian
sudo apt-get install pandoc

# macOS
brew install pandoc

# Windows
choco install pandoc
```

#### Basic Usage

```bash
# Convert Markdown to HTML
pandoc input.md -o output.html

# Convert Markdown to PDF
pandoc input.md -o output.pdf

# Convert Markdown to DOCX
pandoc input.md -o output.docx
```

## Markdown to Other Formats

### Markdown to HTML

```bash
# Basic conversion
pandoc input.md -o output.html

# With custom CSS
pandoc input.md -o output.html --css=style.css

# Standalone HTML with embedded CSS
pandoc input.md -o output.html --standalone --css=style.css

# With table of contents
pandoc input.md -o output.html --toc --toc-depth=3

# With syntax highlighting
pandoc input.md -o output.html --highlight-style=tango

# Complete example with all features
pandoc input.md -o output.html \
    --standalone \
    --toc \
    --toc-depth=3 \
    --css=style.css \
    --highlight-style=github \
    --metadata title="Document Title"
```

### Markdown to PDF

```bash
# Basic conversion (requires LaTeX)
pandoc input.md -o output.pdf

# With custom margins
pandoc input.md -o output.pdf \
    -V geometry:margin=1in

# With custom font
pandoc input.md -o output.pdf \
    -V mainfont="Times New Roman" \
    -V fontsize=12pt

# With table of contents
pandoc input.md -o output.pdf \
    --toc \
    --toc-depth=3

# Complete professional PDF
pandoc input.md -o output.pdf \
    --pdf-engine=xelatex \
    --toc \
    --toc-depth=3 \
    -V geometry:margin=1in \
    -V mainfont="Arial" \
    -V fontsize=11pt \
    -V documentclass=article \
    --highlight-style=tango \
    --metadata title="Professional Report" \
    --metadata author="John Doe" \
    --metadata date="2024-01-15"
```

### Markdown to Word (DOCX)

```bash
# Basic conversion
pandoc input.md -o output.docx

# With reference document for styling
pandoc input.md -o output.docx --reference-doc=template.docx

# With table of contents
pandoc input.md -o output.docx --toc --toc-depth=3

# With metadata
pandoc input.md -o output.docx \
    --metadata title="Document Title" \
    --metadata author="Author Name" \
    --metadata date="2024-01-15"

# Complete example
pandoc input.md -o output.docx \
    --reference-doc=custom_template.docx \
    --toc \
    --toc-depth=3 \
    --highlight-style=tango \
    --metadata title="Professional Document"
```

### Markdown to Plain Text

```bash
# Convert to plain text
pandoc input.md -o output.txt --to=plain

# With line wrapping
pandoc input.md -o output.txt --to=plain --wrap=auto

# Preserve line breaks
pandoc input.md -o output.txt --to=plain --wrap=preserve
```

## Other Formats to Markdown

### HTML to Markdown

```bash
# Basic conversion
pandoc input.html -o output.md

# With specific Markdown flavor
pandoc input.html -o output.md --to=gfm  # GitHub Flavored Markdown

# Preserve structure
pandoc input.html -o output.md \
    --to=markdown_strict \
    --wrap=preserve

# With ATX-style headers
pandoc input.html -o output.md --markdown-headings=atx
```

### Word (DOCX) to Markdown

```bash
# Basic conversion
pandoc input.docx -o output.md

# With GitHub Flavored Markdown
pandoc input.docx -o output.md --to=gfm

# Extract images
pandoc input.docx -o output.md --extract-media=./images

# With tracked changes
pandoc input.docx -o output.md --track-changes=all
```

### PDF to Markdown

```bash
# Extract text and convert to Markdown
pdftotext input.pdf - | pandoc -f plain -o output.md

# Using Python with pdfplumber
```

```python
import pdfplumber

with pdfplumber.open('input.pdf') as pdf:
    markdown_content = ""
    for i, page in enumerate(pdf.pages, start=1):
        markdown_content += f"## Page {i}\n\n"
        markdown_content += page.extract_text() + "\n\n"

with open('output.md', 'w', encoding='utf-8') as f:
    f.write(markdown_content)
```

### LaTeX to Markdown

```bash
# Convert LaTeX to Markdown
pandoc input.tex -o output.md

# With specific Markdown variant
pandoc input.tex -o output.md --to=gfm
```

## Advanced Conversions

### Markdown to Presentation (Reveal.js)

```bash
# Create HTML presentation
pandoc slides.md -o slides.html -t revealjs -s

# With theme
pandoc slides.md -o slides.html -t revealjs -s \
    -V theme=moon

# With incremental bullets
pandoc slides.md -o slides.html -t revealjs -s \
    --incremental
```

### Markdown to EPUB

```bash
# Create EPUB book
pandoc book.md -o book.epub

# With metadata
pandoc book.md -o book.epub \
    --metadata title="Book Title" \
    --metadata author="Author Name" \
    --epub-cover-image=cover.jpg

# With CSS styling
pandoc book.md -o book.epub --css=style.css
```

### Multiple Markdown Files to Single Document

```bash
# Combine multiple Markdown files
pandoc chapter1.md chapter2.md chapter3.md -o book.pdf

# With separators
pandoc *.md -o combined.html --toc

# Maintain file order
pandoc intro.md main.md conclusion.md -o complete.docx
```

## Python-Based Conversions

### Using markdown2

```python
import markdown2

# Convert Markdown to HTML
with open('input.md', 'r', encoding='utf-8') as f:
    md_content = f.read()

html = markdown2.markdown(md_content, extras=['tables', 'fenced-code-blocks'])

with open('output.html', 'w', encoding='utf-8') as f:
    f.write(html)
```

### Using mistune

```python
import mistune

# Convert Markdown to HTML
with open('input.md', 'r', encoding='utf-8') as f:
    md_content = f.read()

html = mistune.html(md_content)

with open('output.html', 'w', encoding='utf-8') as f:
    f.write(html)
```

### Using pypandoc (Python wrapper for pandoc)

```python
import pypandoc

# Convert Markdown to HTML
pypandoc.convert_file('input.md', 'html', outputfile='output.html')

# Convert Markdown to PDF
pypandoc.convert_file('input.md', 'pdf', outputfile='output.pdf',
                      extra_args=['--pdf-engine=xelatex'])

# Convert Markdown to DOCX
pypandoc.convert_file('input.md', 'docx', outputfile='output.docx')

# Convert HTML to Markdown
pypandoc.convert_file('input.html', 'md', outputfile='output.md')

# Convert with options
pypandoc.convert_file('input.md', 'html', outputfile='output.html',
                      extra_args=['--toc', '--css=style.css', '--standalone'])
```

### HTML to Markdown with html2text

```python
import html2text

# Convert HTML to Markdown
with open('input.html', 'r', encoding='utf-8') as f:
    html_content = f.read()

h = html2text.HTML2Text()
h.ignore_links = False
h.ignore_images = False
markdown = h.handle(html_content)

with open('output.md', 'w', encoding='utf-8') as f:
    f.write(markdown)
```

## Batch Conversions

### Batch Convert Markdown to HTML

```bash
# Convert all Markdown files in directory
for file in *.md; do
    pandoc "$file" -o "${file%.md}.html" --standalone --css=style.css
done
```

```python
import os
import subprocess
from pathlib import Path

def batch_md_to_html(input_dir, output_dir):
    """Convert all Markdown files to HTML"""
    Path(output_dir).mkdir(exist_ok=True)

    for filename in os.listdir(input_dir):
        if filename.endswith('.md'):
            input_path = os.path.join(input_dir, filename)
            output_path = os.path.join(output_dir, filename.replace('.md', '.html'))

            subprocess.run([
                'pandoc',
                input_path,
                '-o', output_path,
                '--standalone',
                '--css=style.css'
            ])
            print(f"Converted: {filename}")

batch_md_to_html('markdown_files', 'html_files')
```

### Batch Convert Markdown to PDF

```python
import os
import subprocess
from pathlib import Path

def batch_md_to_pdf(input_dir, output_dir):
    """Convert all Markdown files to PDF"""
    Path(output_dir).mkdir(exist_ok=True)

    for filename in os.listdir(input_dir):
        if filename.endswith('.md'):
            input_path = os.path.join(input_dir, filename)
            output_path = os.path.join(output_dir, filename.replace('.md', '.pdf'))

            subprocess.run([
                'pandoc',
                input_path,
                '-o', output_path,
                '--pdf-engine=xelatex',
                '--toc',
                '-V', 'geometry:margin=1in'
            ])
            print(f"Converted: {filename}")

batch_md_to_pdf('docs', 'pdf_output')
```

### Batch Convert to Multiple Formats

```python
import os
import subprocess

def convert_to_multiple_formats(input_file):
    """Convert single Markdown file to HTML, PDF, and DOCX"""
    base_name = input_file.replace('.md', '')

    # To HTML
    subprocess.run(['pandoc', input_file, '-o', f'{base_name}.html',
                   '--standalone', '--toc'])

    # To PDF
    subprocess.run(['pandoc', input_file, '-o', f'{base_name}.pdf',
                   '--pdf-engine=xelatex', '--toc'])

    # To DOCX
    subprocess.run(['pandoc', input_file, '-o', f'{base_name}.docx',
                   '--toc'])

    print(f"Converted {input_file} to HTML, PDF, and DOCX")

# Convert all Markdown files
for file in os.listdir('.'):
    if file.endswith('.md'):
        convert_to_multiple_formats(file)
```

## Custom Styling and Templates

### Create Custom HTML Template

```html
<!-- template.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>$title$</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }
        h1, h2, h3 { color: #333; }
        code { background: #f4f4f4; padding: 2px 5px; }
        pre { background: #f4f4f4; padding: 10px; overflow-x: auto; }
    </style>
</head>
<body>
$body$
</body>
</html>
```

```bash
# Use custom template
pandoc input.md -o output.html --template=template.html
```

### Create Custom CSS

```css
/* style.css */
body {
    font-family: 'Georgia', serif;
    max-width: 900px;
    margin: 0 auto;
    padding: 40px;
    color: #333;
    line-height: 1.8;
}

h1 {
    color: #2c3e50;
    border-bottom: 2px solid #3498db;
    padding-bottom: 10px;
}

h2 {
    color: #34495e;
    margin-top: 30px;
}

code {
    background-color: #f8f8f8;
    border: 1px solid #ddd;
    padding: 2px 6px;
    border-radius: 3px;
}

pre {
    background-color: #f8f8f8;
    border: 1px solid #ddd;
    padding: 15px;
    overflow-x: auto;
    border-radius: 5px;
}

table {
    border-collapse: collapse;
    width: 100%;
    margin: 20px 0;
}

th, td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
}

th {
    background-color: #3498db;
    color: white;
}
```

```bash
# Use custom CSS
pandoc input.md -o output.html --css=style.css --standalone
```

### Create Word Template

```bash
# Generate reference document
pandoc -o reference.docx --print-default-data-file reference.docx

# Modify reference.docx in Word (change fonts, styles, etc.)

# Use custom template
pandoc input.md -o output.docx --reference-doc=reference.docx
```

## Documentation Publishing Workflows

### Generate Static Documentation Site

```bash
# Convert all docs to HTML with navigation
mkdir -p site
for file in docs/*.md; do
    filename=$(basename "$file" .md)
    pandoc "$file" -o "site/${filename}.html" \
        --standalone \
        --toc \
        --css=../style.css \
        --template=template.html
done
```

### Create PDF Documentation

```bash
# Combine all documentation into single PDF
pandoc docs/*.md -o documentation.pdf \
    --toc \
    --toc-depth=3 \
    --pdf-engine=xelatex \
    -V geometry:margin=1in \
    -V mainfont="Arial" \
    -V fontsize=11pt \
    --highlight-style=tango
```

### Export to Multiple Formats

```python
import subprocess

def export_documentation(source_dir, output_base):
    """Export documentation to HTML, PDF, and DOCX"""

    # HTML version
    subprocess.run([
        'pandoc', f'{source_dir}/*.md',
        '-o', f'{output_base}.html',
        '--standalone', '--toc', '--css=style.css'
    ], shell=True)

    # PDF version
    subprocess.run([
        'pandoc', f'{source_dir}/*.md',
        '-o', f'{output_base}.pdf',
        '--toc', '--pdf-engine=xelatex'
    ], shell=True)

    # DOCX version
    subprocess.run([
        'pandoc', f'{source_dir}/*.md',
        '-o', f'{output_base}.docx',
        '--toc'
    ], shell=True)

export_documentation('docs', 'documentation')
```

## Best Practices

### Conversion Quality
- Use pandoc for best format compatibility
- Specify metadata (title, author, date) for better output
- Use reference documents for consistent styling
- Test output in target format viewers

### Markdown Writing for Conversion
- Use standard Markdown syntax for best compatibility
- Use ATX-style headers (# ## ###) instead of Setext
- Include blank lines between elements
- Use fenced code blocks with language specification

### Styling and Formatting
- Create custom templates for consistent branding
- Use CSS for HTML output styling
- Use reference documents for Word/PDF styling
- Test with different CSS/themes

### Performance
- Use batch scripts for multiple files
- Consider file size for PDF generation
- Compress images before including in Markdown
- Use appropriate PDF engines (xelatex for Unicode)

### Error Handling
- Validate Markdown syntax before conversion
- Check for missing images or references
- Test with sample content first
- Handle special characters properly

## Quick Reference

| Conversion | Command |
|------------|---------|
| MD → HTML | `pandoc input.md -o output.html --standalone` |
| MD → PDF | `pandoc input.md -o output.pdf --pdf-engine=xelatex` |
| MD → DOCX | `pandoc input.md -o output.docx` |
| HTML → MD | `pandoc input.html -o output.md` |
| DOCX → MD | `pandoc input.docx -o output.md` |
| MD → HTML (CSS) | `pandoc input.md -o output.html --css=style.css -s` |
| MD → PDF (TOC) | `pandoc input.md -o output.pdf --toc` |
| Multiple MD → PDF | `pandoc *.md -o output.pdf --toc` |

## Keywords

**English keywords:**
markdown converter, markdown to html, markdown to pdf, markdown to word, html to markdown, pdf to markdown, docx to markdown, document conversion, pandoc, markdown transformation, format conversion, documentation publishing

**Chinese keywords (中文關鍵詞):**
Markdown轉換, Markdown轉HTML, Markdown轉PDF, Markdown轉Word, HTML轉Markdown, PDF轉Markdown, DOCX轉Markdown, 文檔轉換, 格式轉換, 文檔發布
