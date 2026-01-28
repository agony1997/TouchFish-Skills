---
name: technical-presentation
description: "Comprehensive technical presentation (PPTX) generation toolkit using python-pptx. When Claude needs to: (1) Create PowerPoint presentations, (2) Generate technical slides, (3) Create product presentations, (4) Build pitch decks, (5) Generate training slides, or (6) 技術簡報、PowerPoint生成、產品簡報、投影片製作、演示文稿"
---

# Technical Presentation Generator

## Overview

This skill provides comprehensive guidance for creating professional PowerPoint presentations using python-pptx library. It covers technical presentations, product demos, pitch decks, training materials, and automated slide generation from data.

## When to use this skill

**ALWAYS use this skill when the user mentions:**
- Creating PowerPoint presentations
- Generating technical slides
- Building product presentations
- Creating pitch decks
- Generating training materials
- Creating business presentations
- Automating slide creation

**Trigger phrases include:**
- "Create PowerPoint" / "建立PowerPoint"
- "Generate slides" / "生成投影片"
- "Technical presentation" / "技術簡報"
- "Product demo slides" / "產品演示簡報"
- "Pitch deck" / "投資簡報"
- "Training slides" / "培訓投影片"
- "PPTX generation" / "PPTX生成"

## How to use this skill

### Workflow Overview

This skill follows a systematic 4-step workflow:

1. **Content Planning** - Define presentation structure and key messages
2. **Slide Design** - Create slides with proper layout and formatting
3. **Content Population** - Add text, images, charts, and data
4. **Finalization** - Review, format, and export presentation

## Python Library: python-pptx

### Installation

```bash
pip install python-pptx
```

### Basic Usage

```python
from pptx import Presentation
from pptx.util import Inches, Pt

# Create new presentation
prs = Presentation()

# Add slide
slide = prs.slides.add_slide(prs.slide_layouts[0])  # Title slide

# Add title
title = slide.shapes.title
title.text = "Technical Presentation"

# Save presentation
prs.save('presentation.pptx')
```

## Creating Technical Presentations

### Title Slide

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN

# Create presentation
prs = Presentation()

# Add title slide (layout 0)
title_slide = prs.slides.add_slide(prs.slide_layouts[0])

# Set title
title = title_slide.shapes.title
title.text = "System Architecture Overview"

# Set subtitle
subtitle = title_slide.placeholders[1]
subtitle.text = "Technical Deep Dive\nQ1 2024"

# Save
prs.save('tech_presentation.pptx')
```

### Content Slide with Bullet Points

```python
from pptx import Presentation
from pptx.util import Pt

prs = Presentation()

# Add bullet slide (layout 1)
slide = prs.slides.add_slide(prs.slide_layouts[1])

# Set title
title = slide.shapes.title
title.text = "Key Features"

# Add bullet points
body_shape = slide.placeholders[1]
tf = body_shape.text_frame
tf.text = "Scalable Architecture"

# Add more bullet points
for point in ["High Availability", "Security First", "Cloud Native"]:
    p = tf.add_paragraph()
    p.text = point
    p.level = 0

prs.save('tech_presentation.pptx')
```

### Slide with Image

```python
from pptx import Presentation
from pptx.util import Inches

prs = Presentation()

# Add blank slide (layout 6)
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add title textbox
left = Inches(0.5)
top = Inches(0.5)
width = Inches(9)
height = Inches(0.5)
txBox = slide.shapes.add_textbox(left, top, width, height)
tf = txBox.text_frame
tf.text = "System Architecture Diagram"

# Add image
left = Inches(1)
top = Inches(1.5)
pic = slide.shapes.add_picture('architecture.png', left, top, width=Inches(8))

prs.save('tech_presentation.pptx')
```

### Slide with Table

```python
from pptx import Presentation
from pptx.util import Inches

prs = Presentation()

# Add blank slide
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add title
left = Inches(0.5)
top = Inches(0.5)
width = Inches(9)
height = Inches(0.5)
txBox = slide.shapes.add_textbox(left, top, width, height)
tf = txBox.text_frame
tf.text = "Performance Metrics"

# Add table
rows = 5
cols = 4
left = Inches(1)
top = Inches(1.5)
width = Inches(8)
height = Inches(3)

table = slide.shapes.add_table(rows, cols, left, top, width, height).table

# Set column headers
headers = ['Metric', 'Current', 'Target', 'Status']
for i, header in enumerate(headers):
    cell = table.cell(0, i)
    cell.text = header
    cell.text_frame.paragraphs[0].font.bold = True

# Add data
data = [
    ['Response Time', '200ms', '150ms', 'On Track'],
    ['Throughput', '1000 req/s', '1200 req/s', 'At Risk'],
    ['Error Rate', '0.5%', '0.3%', 'Behind'],
    ['Uptime', '99.9%', '99.95%', 'Ahead']
]

for i, row in enumerate(data, start=1):
    for j, value in enumerate(row):
        table.cell(i, j).text = value

prs.save('tech_presentation.pptx')
```

### Slide with Chart

```python
from pptx import Presentation
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE
from pptx.util import Inches

prs = Presentation()

# Add blank slide
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add title
left = Inches(0.5)
top = Inches(0.5)
width = Inches(9)
height = Inches(0.5)
txBox = slide.shapes.add_textbox(left, top, width, height)
tf = txBox.text_frame
tf.text = "Quarterly Revenue Growth"

# Create chart data
chart_data = CategoryChartData()
chart_data.categories = ['Q1', 'Q2', 'Q3', 'Q4']
chart_data.add_series('Revenue', (520, 580, 640, 720))

# Add chart
left = Inches(1)
top = Inches(1.5)
width = Inches(8)
height = Inches(4)

chart = slide.shapes.add_chart(
    XL_CHART_TYPE.COLUMN_CLUSTERED,
    left, top, width, height,
    chart_data
).chart

chart.has_legend = True
chart.has_title = False

prs.save('tech_presentation.pptx')
```

## Complete Technical Presentation Example

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.enum.text import PP_ALIGN
from pptx.dml.color import RGBColor
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

def create_technical_presentation():
    """Create a complete technical presentation"""
    prs = Presentation()

    # Slide 1: Title Slide
    slide = prs.slides.add_slide(prs.slide_layouts[0])
    title = slide.shapes.title
    title.text = "E-Commerce Platform"
    subtitle = slide.placeholders[1]
    subtitle.text = "System Architecture & Technical Overview\nJanuary 2024"

    # Slide 2: Agenda
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Agenda"
    body = slide.placeholders[1].text_frame
    agenda_items = [
        "System Overview",
        "Architecture Design",
        "Technology Stack",
        "Performance Metrics",
        "Security Measures",
        "Scalability Strategy",
        "Roadmap"
    ]
    for i, item in enumerate(agenda_items):
        if i == 0:
            body.text = item
        else:
            p = body.add_paragraph()
            p.text = item
            p.level = 0

    # Slide 3: System Overview
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "System Overview"
    body = slide.placeholders[1].text_frame
    body.text = "Cloud-native microservices architecture"

    details = [
        "Built on AWS infrastructure",
        "Node.js + React technology stack",
        "PostgreSQL + Redis for data layer",
        "RESTful API design",
        "Event-driven architecture"
    ]
    for detail in details:
        p = body.add_paragraph()
        p.text = detail
        p.level = 1

    # Slide 4: Architecture Diagram
    slide = prs.slides.add_slide(prs.slide_layouts[6])
    left = Inches(0.5)
    top = Inches(0.5)
    width = Inches(9)
    height = Inches(0.5)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "System Architecture"

    # Add architecture diagram (if image exists)
    try:
        left = Inches(1)
        top = Inches(1.5)
        pic = slide.shapes.add_picture('architecture.png', left, top, width=Inches(8))
    except:
        # Add text placeholder if image doesn't exist
        txBox = slide.shapes.add_textbox(Inches(2), Inches(3), Inches(6), Inches(2))
        tf = txBox.text_frame
        tf.text = "[Architecture Diagram]"
        tf.paragraphs[0].alignment = PP_ALIGN.CENTER

    # Slide 5: Technology Stack
    slide = prs.slides.add_slide(prs.slide_layouts[6])

    # Title
    left = Inches(0.5)
    top = Inches(0.5)
    width = Inches(9)
    height = Inches(0.5)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "Technology Stack"

    # Table for tech stack
    rows = 6
    cols = 2
    left = Inches(2)
    top = Inches(1.5)
    width = Inches(6)
    height = Inches(3.5)

    table = slide.shapes.add_table(rows, cols, left, top, width, height).table

    # Set column headers
    table.cell(0, 0).text = "Layer"
    table.cell(0, 1).text = "Technology"

    # Add data
    tech_stack = [
        ("Frontend", "React 18, Material-UI"),
        ("Backend", "Node.js 18, Express"),
        ("Database", "PostgreSQL 14"),
        ("Cache", "Redis 7"),
        ("Infrastructure", "AWS (EC2, RDS, S3)")
    ]

    for i, (layer, tech) in enumerate(tech_stack, start=1):
        table.cell(i, 0).text = layer
        table.cell(i, 1).text = tech

    # Slide 6: Performance Metrics
    slide = prs.slides.add_slide(prs.slide_layouts[6])

    # Title
    left = Inches(0.5)
    top = Inches(0.5)
    width = Inches(9)
    height = Inches(0.5)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "Performance Metrics"

    # Create chart
    chart_data = CategoryChartData()
    chart_data.categories = ['Response Time', 'Throughput', 'Uptime']
    chart_data.add_series('Current', (200, 1000, 99.9))
    chart_data.add_series('Target', (150, 1200, 99.95))

    left = Inches(1.5)
    top = Inches(1.5)
    width = Inches(7)
    height = Inches(4)

    chart = slide.shapes.add_chart(
        XL_CHART_TYPE.COLUMN_CLUSTERED,
        left, top, width, height,
        chart_data
    ).chart

    chart.has_legend = True

    # Slide 7: Security Measures
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Security Measures"
    body = slide.placeholders[1].text_frame

    security_measures = [
        ("Authentication", ["JWT tokens", "OAuth 2.0 support", "Multi-factor authentication"]),
        ("Data Security", ["AES-256 encryption at rest", "TLS 1.3 in transit", "PCI DSS compliant"]),
        ("Infrastructure", ["WAF protection", "DDoS mitigation", "Regular security audits"])
    ]

    for category, items in security_measures:
        if category == security_measures[0][0]:
            body.text = category
        else:
            p = body.add_paragraph()
            p.text = category
            p.level = 0

        for item in items:
            p = body.add_paragraph()
            p.text = item
            p.level = 1

    # Slide 8: Scalability Strategy
    slide = prs.slides.add_slide(prs.slide_layouts[1])
    title = slide.shapes.title
    title.text = "Scalability Strategy"
    body = slide.placeholders[1].text_frame
    body.text = "Horizontal Scaling"

    strategies = [
        "Auto-scaling groups for services",
        "Database read replicas",
        "CDN for static assets",
        "Caching layer (Redis)",
        "Load balancing",
        "Microservices architecture"
    ]

    for strategy in strategies:
        p = body.add_paragraph()
        p.text = strategy
        p.level = 1

    # Slide 9: Roadmap
    slide = prs.slides.add_slide(prs.slide_layouts[6])

    # Title
    left = Inches(0.5)
    top = Inches(0.5)
    width = Inches(9)
    height = Inches(0.5)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "2024 Technical Roadmap"

    # Table for roadmap
    rows = 5
    cols = 3
    left = Inches(1)
    top = Inches(1.5)
    width = Inches(8)
    height = Inches(4)

    table = slide.shapes.add_table(rows, cols, left, top, width, height).table

    # Headers
    table.cell(0, 0).text = "Quarter"
    table.cell(0, 1).text = "Initiative"
    table.cell(0, 2).text = "Status"

    # Data
    roadmap = [
        ("Q1", "Microservices migration", "In Progress"),
        ("Q2", "Mobile app development", "Planned"),
        ("Q3", "ML recommendation engine", "Planned"),
        ("Q4", "International expansion", "Planned")
    ]

    for i, (quarter, initiative, status) in enumerate(roadmap, start=1):
        table.cell(i, 0).text = quarter
        table.cell(i, 1).text = initiative
        table.cell(i, 2).text = status

    # Slide 10: Thank You
    slide = prs.slides.add_slide(prs.slide_layouts[6])

    # Thank you message
    left = Inches(2)
    top = Inches(2.5)
    width = Inches(6)
    height = Inches(1)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "Thank You"
    tf.paragraphs[0].font.size = Pt(48)
    tf.paragraphs[0].font.bold = True
    tf.paragraphs[0].alignment = PP_ALIGN.CENTER

    # Contact info
    left = Inches(2)
    top = Inches(4)
    width = Inches(6)
    height = Inches(1)
    txBox = slide.shapes.add_textbox(left, top, width, height)
    tf = txBox.text_frame
    tf.text = "Questions?\ntech-team@company.com"
    tf.paragraphs[0].alignment = PP_ALIGN.CENTER

    # Save presentation
    prs.save('technical_presentation.pptx')
    print("Technical presentation created successfully!")

# Generate presentation
create_technical_presentation()
```

## Advanced Formatting

### Custom Fonts and Colors

```python
from pptx import Presentation
from pptx.util import Pt
from pptx.dml.color import RGBColor

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add textbox with custom formatting
left = Inches(2)
top = Inches(2)
width = Inches(6)
height = Inches(1)
txBox = slide.shapes.add_textbox(left, top, width, height)
tf = txBox.text_frame
tf.text = "Custom Formatted Text"

# Apply formatting
p = tf.paragraphs[0]
p.font.name = 'Arial'
p.font.size = Pt(32)
p.font.bold = True
p.font.color.rgb = RGBColor(0, 112, 192)  # Blue
p.alignment = PP_ALIGN.CENTER

prs.save('formatted_presentation.pptx')
```

### Master Slides and Templates

```python
from pptx import Presentation

# Load existing template
prs = Presentation('company_template.pptx')

# Add slides using template layouts
title_slide = prs.slides.add_slide(prs.slide_layouts[0])
content_slide = prs.slides.add_slide(prs.slide_layouts[1])

# Template formatting is automatically applied
prs.save('presentation_from_template.pptx')
```

### Adding Shapes

```python
from pptx import Presentation
from pptx.util import Inches
from pptx.enum.shapes import MSO_SHAPE
from pptx.dml.color import RGBColor

prs = Presentation()
slide = prs.slides.add_slide(prs.slide_layouts[6])

# Add rectangle
left = Inches(1)
top = Inches(2)
width = Inches(3)
height = Inches(1)
shape = slide.shapes.add_shape(
    MSO_SHAPE.RECTANGLE,
    left, top, width, height
)

# Format shape
shape.fill.solid()
shape.fill.fore_color.rgb = RGBColor(0, 112, 192)
shape.line.color.rgb = RGBColor(0, 0, 0)

# Add text to shape
text_frame = shape.text_frame
text_frame.text = "Process Step"
text_frame.paragraphs[0].font.color.rgb = RGBColor(255, 255, 255)

prs.save('shapes_presentation.pptx')
```

## Automated Slide Generation

### Generate Slides from Data

```python
import pandas as pd
from pptx import Presentation
from pptx.util import Inches
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

def generate_slides_from_data(csv_file):
    """Generate presentation slides from CSV data"""
    # Load data
    df = pd.read_csv(csv_file)

    # Create presentation
    prs = Presentation()

    # Title slide
    slide = prs.slides.add_slide(prs.slide_layouts[0])
    title = slide.shapes.title
    title.text = "Data Analysis Report"
    subtitle = slide.placeholders[1]
    subtitle.text = f"Analysis of {len(df)} records"

    # Summary slide with table
    slide = prs.slides.add_slide(prs.slide_layouts[6])

    # Add title
    txBox = slide.shapes.add_textbox(Inches(0.5), Inches(0.5), Inches(9), Inches(0.5))
    txBox.text_frame.text = "Data Summary"

    # Create table from DataFrame
    rows = min(len(df), 10) + 1  # Limit to 10 rows
    cols = len(df.columns)

    table = slide.shapes.add_table(
        rows, cols,
        Inches(0.5), Inches(1.5),
        Inches(9), Inches(4)
    ).table

    # Add headers
    for i, col in enumerate(df.columns):
        table.cell(0, i).text = col

    # Add data
    for i in range(min(len(df), 10)):
        for j, col in enumerate(df.columns):
            table.cell(i + 1, j).text = str(df.iloc[i][col])

    # Chart slide
    if 'value' in df.columns and 'category' in df.columns:
        slide = prs.slides.add_slide(prs.slide_layouts[6])

        # Title
        txBox = slide.shapes.add_textbox(Inches(0.5), Inches(0.5), Inches(9), Inches(0.5))
        txBox.text_frame.text = "Data Visualization"

        # Create chart
        chart_data = CategoryChartData()
        chart_data.categories = df['category'].tolist()[:10]
        chart_data.add_series('Values', df['value'].tolist()[:10])

        chart = slide.shapes.add_chart(
            XL_CHART_TYPE.BAR_CLUSTERED,
            Inches(1), Inches(1.5),
            Inches(8), Inches(4),
            chart_data
        ).chart

    prs.save('data_presentation.pptx')
    print("Data presentation created successfully!")

# Usage
generate_slides_from_data('business_data.csv')
```

## Best Practices

### Presentation Design
- Use consistent layouts throughout
- Limit text per slide (6x6 rule: max 6 bullets, 6 words each)
- Use high-quality images
- Maintain consistent color scheme
- Include slide numbers

### Content Structure
- Start with agenda/overview
- One main idea per slide
- Use visuals to support text
- Include clear section dividers
- End with summary/call-to-action

### Technical Presentations
- Include architecture diagrams
- Show code snippets sparingly
- Use charts for metrics
- Include implementation timelines
- Provide technical appendix

### Automation
- Use templates for consistency
- Generate slides from data sources
- Automate chart creation
- Version control presentations
- Script repetitive tasks

## Quick Reference

### Common Slide Layouts

| Layout Index | Description |
|--------------|-------------|
| 0 | Title Slide |
| 1 | Title and Content |
| 2 | Section Header |
| 3 | Two Content |
| 4 | Comparison |
| 5 | Title Only |
| 6 | Blank |
| 7 | Content with Caption |
| 8 | Picture with Caption |

### Chart Types

| Chart Type | Use Case |
|------------|----------|
| COLUMN_CLUSTERED | Compare values across categories |
| BAR_CLUSTERED | Compare values (horizontal) |
| LINE | Show trends over time |
| PIE | Show proportions |
| AREA | Show cumulative trends |
| XY_SCATTER | Show correlations |

### Color Palette (RGB)

| Color | RGB | Usage |
|-------|-----|-------|
| Blue | (0, 112, 192) | Primary |
| Green | (0, 176, 80) | Success/Positive |
| Red | (255, 0, 0) | Alert/Negative |
| Orange | (255, 192, 0) | Warning |
| Gray | (127, 127, 127) | Neutral |

## Keywords

**English keywords:**
technical presentation, powerpoint generation, pptx creation, slides generation, pitch deck, product demo, training slides, business presentation, python-pptx, automated slides

**Chinese keywords (中文關鍵詞):**
技術簡報, PowerPoint生成, PPTX建立, 投影片生成, 投資簡報, 產品演示, 培訓投影片, 商業簡報, 自動化投影片
