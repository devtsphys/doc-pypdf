# pypdf Reference Card & Cheat Sheet

**Version**: pypdf 4.x+  
**Official Docs**: https://pypdf.readthedocs.io/

-----

## Table of Contents

1. [Installation & Import](#installation--import)
1. [Core Classes](#core-classes)
1. [Reading PDFs](#reading-pdfs)
1. [Writing & Modifying PDFs](#writing--modifying-pdfs)
1. [Text Extraction](#text-extraction)
1. [Page Manipulation](#page-manipulation)
1. [Metadata Operations](#metadata-operations)
1. [Merging & Splitting](#merging--splitting)
1. [Forms & Annotations](#forms--annotations)
1. [Images & Resources](#images--resources)
1. [Encryption & Security](#encryption--security)
1. [Complete Methods Reference](#complete-methods-reference)
1. [Common Workflows](#common-workflows)
1. [Advanced Techniques](#advanced-techniques)

-----

## Installation & Import

```bash
pip install pypdf
```

```python
from pypdf import PdfReader, PdfWriter, PdfMerger
from pypdf.generic import DictionaryObject, ArrayObject
```

-----

## Core Classes

### PdfReader

Read and extract content from PDF files.

```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
```

### PdfWriter

Create and modify PDF files.

```python
from pypdf import PdfWriter

writer = PdfWriter()
```

### PdfMerger

Merge multiple PDF files.

```python
from pypdf import PdfMerger

merger = PdfMerger()
```

### PageObject

Represents a single PDF page.

-----

## Reading PDFs

### Basic Reading

```python
from pypdf import PdfReader

# Open PDF
reader = PdfReader("document.pdf")

# Get number of pages
num_pages = len(reader.pages)

# Access specific page (0-indexed)
page = reader.pages[0]

# Get metadata
metadata = reader.metadata

# Check encryption
is_encrypted = reader.is_encrypted
```

### Reading from Bytes/Stream

```python
from io import BytesIO

# From bytes
with open("document.pdf", "rb") as f:
    pdf_bytes = f.read()
    
reader = PdfReader(BytesIO(pdf_bytes))

# From file-like object
with open("document.pdf", "rb") as f:
    reader = PdfReader(f)
```

### Encrypted PDFs

```python
reader = PdfReader("encrypted.pdf")

if reader.is_encrypted:
    # Decrypt with password
    reader.decrypt("password")
    
    # Alternative syntax
    # reader = PdfReader("encrypted.pdf", password="password")
```

-----

## Writing & Modifying PDFs

### Creating a New PDF

```python
from pypdf import PdfWriter

writer = PdfWriter()

# Add pages from another PDF
reader = PdfReader("source.pdf")
for page in reader.pages:
    writer.add_page(page)

# Save to file
with open("output.pdf", "wb") as f:
    writer.write(f)
```

### Modifying Existing PDF

```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

# Copy all pages
for page in reader.pages:
    writer.add_page(page)

# Modify specific pages
page = writer.pages[0]
page.rotate(90)

# Save
with open("modified.pdf", "wb") as f:
    writer.write(f)
```

### Cloning Documents

```python
writer = PdfWriter(clone_from="template.pdf")
# Now writer contains all pages from template.pdf
```

-----

## Text Extraction

### Extract Text from Entire Document

```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")

# Method 1: Extract all text
full_text = ""
for page in reader.pages:
    full_text += page.extract_text()

# Method 2: Extract with page numbers
text_by_page = {}
for i, page in enumerate(reader.pages):
    text_by_page[i] = page.extract_text()
```

### Extract Text with Layout Preservation

```python
# Extract with layout mode
page = reader.pages[0]
text = page.extract_text(extraction_mode="layout")

# Older versions may use:
# text = page.extract_text(layout_mode=True)
```

### Extract Text from Specific Areas

```python
from pypdf import PdfReader

page = reader.pages[0]

# Get page dimensions
width = page.mediabox.width
height = page.mediabox.height

# Extract text from a rectangle (x0, y0, x1, y1)
# Coordinates are from bottom-left corner
text = page.extract_text(
    extraction_mode="layout",
    layout_mode_space_vertically=False
)
```

### Advanced Text Extraction Options

```python
# Control spacing
text = page.extract_text(
    extraction_mode="layout",
    layout_mode_space_vertically=True,
    layout_mode_scale_weight=1.0
)

# Plain text mode (default)
text = page.extract_text(extraction_mode="plain")
```

### Handling Text Encoding Issues

```python
# Extract with custom encoding handling
text = page.extract_text()

# Clean extracted text
import re
text = re.sub(r'\s+', ' ', text)  # Normalize whitespace
text = text.strip()
```

-----

## Page Manipulation

### Rotating Pages

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

page = reader.pages[0]

# Rotate clockwise
page.rotate(90)   # 90 degrees
page.rotate(180)  # 180 degrees
page.rotate(270)  # 270 degrees (or -90)

writer.add_page(page)
```

### Scaling Pages

```python
page = reader.pages[0]

# Scale by factor
page.scale(0.5, 0.5)  # 50% of original size
page.scale(2, 2)      # 200% of original size

# Scale to specific dimensions
page.scale_to(width=612, height=792)  # US Letter size

# Scale by factor
page.scale_by(0.75)  # 75% of original
```

### Cropping Pages

```python
page = reader.pages[0]

# Crop using coordinates (left, bottom, right, top)
page.cropbox.lower_left = (50, 50)
page.cropbox.upper_right = (550, 750)

# Alternative: using mediabox
page.mediabox.lower_left = (0, 0)
page.mediabox.upper_right = (612, 792)
```

### Merging Pages (Overlays)

```python
# Overlay one page on another (watermark)
background = reader.pages[0]
foreground = watermark_reader.pages[0]

background.merge_page(foreground)
writer.add_page(background)
```

-----

## Metadata Operations

### Reading Metadata

```python
reader = PdfReader("document.pdf")

# Get all metadata
metadata = reader.metadata

# Access specific fields
title = metadata.get('/Title', 'No title')
author = metadata.get('/Author', 'Unknown')
subject = metadata.get('/Subject', '')
creator = metadata.get('/Creator', '')
producer = metadata.get('/Producer', '')
creation_date = metadata.get('/CreationDate', '')
mod_date = metadata.get('/ModDate', '')

# Print all metadata
if metadata:
    for key, value in metadata.items():
        print(f"{key}: {value}")
```

### Writing Metadata

```python
from pypdf import PdfWriter

writer = PdfWriter()

# Add pages...
reader = PdfReader("input.pdf")
for page in reader.pages:
    writer.add_page(page)

# Set metadata
writer.add_metadata({
    '/Title': 'My Document',
    '/Author': 'John Doe',
    '/Subject': 'Python PDF Processing',
    '/Creator': 'pypdf',
    '/Producer': 'pypdf',
})

# Save
with open("output.pdf", "wb") as f:
    writer.write(f)
```

### Copying Metadata

```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

# Copy pages
for page in reader.pages:
    writer.add_page(page)

# Copy metadata
if reader.metadata:
    writer.add_metadata(reader.metadata)
```

-----

## Merging & Splitting

### Merging PDFs

```python
from pypdf import PdfMerger

merger = PdfMerger()

# Append entire PDFs
merger.append("file1.pdf")
merger.append("file2.pdf")
merger.append("file3.pdf")

# Merge specific pages
merger.append("file4.pdf", pages=(0, 3))  # Pages 0-2

# Merge with page ranges
merger.append("file5.pdf", pages=(0, 5))  # First 5 pages

# Write merged PDF
merger.write("merged.pdf")
merger.close()

# Or use context manager
with PdfMerger() as merger:
    merger.append("file1.pdf")
    merger.append("file2.pdf")
    merger.write("merged.pdf")
```

### Advanced Merging

```python
merger = PdfMerger()

# Merge at specific position
merger.merge(position=0, fileobj="cover.pdf")  # Insert at beginning
merger.append("content.pdf")
merger.merge(position=1, fileobj="insert.pdf")  # Insert after first doc

# Merge with bookmarks
merger.append("doc.pdf", outline_item="Chapter 1")

merger.write("output.pdf")
merger.close()
```

### Splitting PDFs

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")

# Split into individual pages
for i, page in enumerate(reader.pages):
    writer = PdfWriter()
    writer.add_page(page)
    
    with open(f"page_{i+1}.pdf", "wb") as f:
        writer.write(f)

# Split into chunks
chunk_size = 5
for i in range(0, len(reader.pages), chunk_size):
    writer = PdfWriter()
    
    for page in reader.pages[i:i+chunk_size]:
        writer.add_page(page)
    
    with open(f"chunk_{i//chunk_size + 1}.pdf", "wb") as f:
        writer.write(f)
```

### Extract Specific Pages

```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

# Extract pages 5-10
for page_num in range(4, 10):  # 0-indexed
    writer.add_page(reader.pages[page_num])

# Extract by list
page_numbers = [0, 3, 7, 15]
for page_num in page_numbers:
    writer.add_page(reader.pages[page_num])

with open("extracted.pdf", "wb") as f:
    writer.write(f)
```

-----

## Forms & Annotations

### Reading Form Fields

```python
reader = PdfReader("form.pdf")

# Get all form fields
if reader.get_form_text_fields():
    fields = reader.get_form_text_fields()
    for field_name, field_value in fields.items():
        print(f"{field_name}: {field_value}")

# Get fields with more details
fields = reader.get_fields()
if fields:
    for field_name, field in fields.items():
        print(f"Name: {field_name}")
        print(f"Value: {field.get('/V', 'N/A')}")
        print(f"Type: {field.get('/FT', 'N/A')}")
```

### Filling Form Fields

```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("form.pdf")
writer = PdfWriter()

# Clone pages
for page in reader.pages:
    writer.add_page(page)

# Update form fields
writer.update_page_form_field_values(
    writer.pages[0],
    {
        "name": "John Doe",
        "email": "john@example.com",
        "date": "2024-01-15"
    }
)

with open("filled_form.pdf", "wb") as f:
    writer.write(f)
```

### Flattening Forms

```python
# Flatten to make fields non-editable
writer = PdfWriter()

reader = PdfReader("filled_form.pdf")
for page in reader.pages:
    writer.add_page(page)

# Flatten all form fields
if "/AcroForm" in writer._root_object:
    writer._root_object["/AcroForm"].update({
        "/NeedAppearances": False
    })

with open("flattened.pdf", "wb") as f:
    writer.write(f)
```

### Working with Annotations

```python
page = reader.pages[0]

# Get annotations
if "/Annots" in page:
    annotations = page["/Annots"]
    for annot in annotations:
        annot_obj = annot.get_object()
        print(f"Type: {annot_obj.get('/Subtype')}")
        print(f"Contents: {annot_obj.get('/Contents')}")
```

-----

## Images & Resources

### Extracting Images

```python
from pypdf import PdfReader
import io
from PIL import Image

reader = PdfReader("document.pdf")

for page_num, page in enumerate(reader.pages):
    # Extract images from page
    if "/XObject" in page["/Resources"]:
        xObject = page["/Resources"]["/XObject"].get_object()
        
        for obj_name in xObject:
            obj = xObject[obj_name]
            
            if obj["/Subtype"] == "/Image":
                # Extract image data
                size = (obj["/Width"], obj["/Height"])
                data = obj.get_data()
                
                # Save based on filter type
                if obj["/ColorSpace"] == "/DeviceRGB":
                    img = Image.frombytes("RGB", size, data)
                    img.save(f"page_{page_num}_{obj_name[1:]}.png")
```

### Extracting All Images (Advanced)

```python
def extract_images_from_pdf(pdf_path, output_folder):
    """Extract all images from PDF"""
    import os
    from PIL import Image
    
    reader = PdfReader(pdf_path)
    
    for page_num, page in enumerate(reader.pages):
        try:
            if "/XObject" in page["/Resources"]:
                xObject = page["/Resources"]["/XObject"].get_object()
                
                for obj_name in xObject:
                    obj = xObject[obj_name]
                    
                    if obj["/Subtype"] == "/Image":
                        size = (obj["/Width"], obj["/Height"])
                        data = obj.get_data()
                        
                        mode = "RGB" if obj["/ColorSpace"] == "/DeviceRGB" else "P"
                        
                        img = Image.frombytes(mode, size, data)
                        img.save(
                            os.path.join(
                                output_folder,
                                f"page{page_num}_{obj_name[1:]}.png"
                            )
                        )
        except Exception as e:
            print(f"Error on page {page_num}: {e}")

# Usage
extract_images_from_pdf("document.pdf", "images/")
```

-----

## Encryption & Security

### Checking Encryption

```python
reader = PdfReader("document.pdf")

if reader.is_encrypted:
    print("PDF is encrypted")
    
    # Try to decrypt
    if reader.decrypt("password"):
        print("Successfully decrypted")
    else:
        print("Incorrect password")
```

### Encrypting PDFs

```python
from pypdf import PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

# Copy pages
for page in reader.pages:
    writer.add_page(page)

# Encrypt with user and owner passwords
writer.encrypt(
    user_password="user_pass",
    owner_password="owner_pass",
    algorithm="AES-256"  # Options: RC4-40, RC4-128, AES-128, AES-256
)

with open("encrypted.pdf", "wb") as f:
    writer.write(f)
```

### Setting Permissions

```python
from pypdf import PdfWriter

writer = PdfWriter()

# Add pages...
reader = PdfReader("input.pdf")
for page in reader.pages:
    writer.add_page(page)

# Encrypt with permissions
writer.encrypt(
    user_password="",  # Empty for no password to open
    owner_password="owner_pass",
    permissions_flag=0b0000_0100  # Allow printing only
)

# Common permission flags (combine with |):
# - Printing: 0b0000_0100
# - Modifying: 0b0000_1000
# - Copying: 0b0001_0000
# - Annotations: 0b0010_0000

with open("protected.pdf", "wb") as f:
    writer.write(f)
```

-----

## Complete Methods Reference

### PdfReader Methods

|Method                           |Description                    |Example                        |
|---------------------------------|-------------------------------|-------------------------------|
|`__init__(stream, password=None)`|Initialize reader              |`PdfReader("file.pdf")`        |
|`pages`                          |List of PageObject instances   |`reader.pages[0]`              |
|`metadata`                       |Document metadata dictionary   |`reader.metadata`              |
|`is_encrypted`                   |Check if PDF is encrypted      |`reader.is_encrypted`          |
|`decrypt(password)`              |Decrypt encrypted PDF          |`reader.decrypt("pass")`       |
|`get_form_text_fields()`         |Get form field values          |`reader.get_form_text_fields()`|
|`get_fields()`                   |Get detailed form fields       |`reader.get_fields()`          |
|`get_num_pages()`                |Get page count (deprecated)    |Use `len(reader.pages)`        |
|`get_page(n)`                    |Get page by number (deprecated)|Use `reader.pages[n]`          |
|`outline`                        |Get document outline/bookmarks |`reader.outline`               |
|`page_labels`                    |Get page label information     |`reader.page_labels`           |
|`page_layout`                    |Get page layout mode           |`reader.page_layout`           |
|`page_mode`                      |Get page mode                  |`reader.page_mode`             |
|`xmp_metadata`                   |Get XMP metadata               |`reader.xmp_metadata`          |

### PdfWriter Methods

|Method                                       |Description          |Example                                      |
|---------------------------------------------|---------------------|---------------------------------------------|
|`__init__(clone_from=None)`                  |Initialize writer    |`PdfWriter()`                                |
|`add_page(page)`                             |Add page to document |`writer.add_page(page)`                      |
|`insert_page(page, index)`                   |Insert page at index |`writer.insert_page(page, 0)`                |
|`add_blank_page(width, height)`              |Add blank page       |`writer.add_blank_page(612, 792)`            |
|`remove_page(page_num)`                      |Remove page by index |`writer.remove_page(0)`                      |
|`write(stream)`                              |Write PDF to stream  |`writer.write(f)`                            |
|`add_metadata(metadata)`                     |Add document metadata|`writer.add_metadata({'/Title': 'Doc'})`     |
|`encrypt(user_pwd, owner_pwd)`               |Encrypt PDF          |`writer.encrypt("pass", "admin")`            |
|`update_page_form_field_values(page, fields)`|Update form fields   |`writer.update_page_form_field_values(p, {})`|
|`clone_reader_document_root(reader)`         |Clone from reader    |`writer.clone_reader_document_root(reader)`  |
|`add_outline_item(title, page_num)`          |Add bookmark         |`writer.add_outline_item("Ch1", 0)`          |
|`add_annotation(page_num, annotation)`       |Add annotation       |`writer.add_annotation(0, annot)`            |
|`append_pages_from_reader(reader)`           |Append all pages     |`writer.append_pages_from_reader(reader)`    |
|`add_js(javascript)`                         |Add JavaScript       |`writer.add_js("code")`                      |
|`add_attachment(filename, data)`             |Add file attachment  |`writer.add_attachment("f.txt", b"data")`    |

### PageObject Methods

|Method                      |Description                 |Example                          |
|----------------------------|----------------------------|---------------------------------|
|`extract_text(mode)`        |Extract text from page      |`page.extract_text()`            |
|`rotate(angle)`             |Rotate page                 |`page.rotate(90)`                |
|`scale(sx, sy)`             |Scale page                  |`page.scale(0.5, 0.5)`           |
|`scale_by(factor)`          |Scale by factor             |`page.scale_by(0.75)`            |
|`scale_to(width, height)`   |Scale to dimensions         |`page.scale_to(612, 792)`        |
|`merge_page(page2)`         |Merge another page (overlay)|`page.merge_page(watermark)`     |
|`add_transformation(ctm)`   |Add transformation matrix   |`page.add_transformation(ctm)`   |
|`compress_content_streams()`|Compress page content       |`page.compress_content_streams()`|
|`mediabox`                  |Get/set media box           |`page.mediabox.width`            |
|`cropbox`                   |Get/set crop box            |`page.cropbox`                   |
|`trimbox`                   |Get/set trim box            |`page.trimbox`                   |
|`artbox`                    |Get/set art box             |`page.artbox`                    |
|`bleedbox`                  |Get/set bleed box           |`page.bleedbox`                  |

### PdfMerger Methods

|Method                           |Description      |Example                                 |
|---------------------------------|-----------------|----------------------------------------|
|`__init__()`                     |Initialize merger|`PdfMerger()`                           |
|`append(fileobj, pages=None)`    |Append PDF       |`merger.append("file.pdf")`             |
|`merge(position, fileobj, pages)`|Merge at position|`merger.merge(0, "file.pdf")`           |
|`write(stream)`                  |Write merged PDF |`merger.write("out.pdf")`               |
|`close()`                        |Close merger     |`merger.close()`                        |
|`add_metadata(metadata)`         |Add metadata     |`merger.add_metadata({'/Title': 'Doc'})`|
|`set_page_layout(layout)`        |Set page layout  |`merger.set_page_layout("/SinglePage")` |
|`set_page_mode(mode)`            |Set page mode    |`merger.set_page_mode("/UseNone")`      |

-----

## Common Workflows

### Workflow 1: Extract All Text from PDF

```python
from pypdf import PdfReader

def extract_all_text(pdf_path):
    """Extract all text from a PDF file"""
    reader = PdfReader(pdf_path)
    
    text_content = []
    
    for i, page in enumerate(reader.pages):
        page_text = page.extract_text()
        text_content.append({
            'page_number': i + 1,
            'text': page_text
        })
    
    return text_content

# Usage
result = extract_all_text("document.pdf")

# Save to file
with open("extracted_text.txt", "w", encoding="utf-8") as f:
    for item in result:
        f.write(f"=== Page {item['page_number']} ===\n")
        f.write(item['text'])
        f.write("\n\n")
```

### Workflow 2: Split PDF by Page Range

```python
from pypdf import PdfReader, PdfWriter

def split_pdf(input_pdf, ranges):
    """
    Split PDF into multiple files based on page ranges.
    
    Args:
        input_pdf: Path to input PDF
        ranges: List of tuples (start, end, output_name)
    """
    reader = PdfReader(input_pdf)
    
    for start, end, output_name in ranges:
        writer = PdfWriter()
        
        for page_num in range(start - 1, end):  # Convert to 0-indexed
            if page_num < len(reader.pages):
                writer.add_page(reader.pages[page_num])
        
        with open(output_name, "wb") as f:
            writer.write(f)

# Usage
split_pdf("document.pdf", [
    (1, 10, "section1.pdf"),
    (11, 20, "section2.pdf"),
    (21, 30, "section3.pdf")
])
```

### Workflow 3: Add Watermark to All Pages

```python
from pypdf import PdfReader, PdfWriter

def add_watermark(input_pdf, watermark_pdf, output_pdf):
    """Add watermark to all pages"""
    reader = PdfReader(input_pdf)
    watermark = PdfReader(watermark_pdf)
    writer = PdfWriter()
    
    watermark_page = watermark.pages[0]
    
    for page in reader.pages:
        # Merge watermark onto page
        page.merge_page(watermark_page)
        writer.add_page(page)
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

# Usage
add_watermark("document.pdf", "watermark.pdf", "watermarked.pdf")
```

### Workflow 4: Combine Multiple PDFs with Bookmarks

```python
from pypdf import PdfMerger

def merge_with_bookmarks(files_and_titles, output_pdf):
    """
    Merge PDFs and add bookmarks for each document.
    
    Args:
        files_and_titles: List of (filepath, bookmark_title) tuples
        output_pdf: Output file path
    """
    merger = PdfMerger()
    
    for filepath, title in files_and_titles:
        merger.append(filepath, outline_item=title)
    
    merger.write(output_pdf)
    merger.close()

# Usage
merge_with_bookmarks([
    ("chapter1.pdf", "Chapter 1: Introduction"),
    ("chapter2.pdf", "Chapter 2: Methods"),
    ("chapter3.pdf", "Chapter 3: Results")
], "book.pdf")
```

### Workflow 5: Extract and Clean Text

```python
from pypdf import PdfReader
import re

def extract_clean_text(pdf_path):
    """Extract and clean text from PDF"""
    reader = PdfReader(pdf_path)
    
    cleaned_text = []
    
    for page in reader.pages:
        # Extract text
        text = page.extract_text()
        
        # Clean text
        text = re.sub(r'\s+', ' ', text)  # Normalize whitespace
        text = re.sub(r'\n+', '\n', text)  # Remove extra newlines
        text = text.strip()
        
        # Remove page numbers (simple pattern)
        text = re.sub(r'^\d+\s*$', '', text, flags=re.MULTILINE)
        
        cleaned_text.append(text)
    
    return '\n\n'.join(cleaned_text)

# Usage
clean_text = extract_clean_text("document.pdf")

with open("clean_text.txt", "w", encoding="utf-8") as f:
    f.write(clean_text)
```

### Workflow 6: Batch Process PDFs

```python
from pypdf import PdfReader, PdfWriter
import os

def batch_process_pdfs(input_folder, output_folder, processing_func):
    """
    Batch process all PDFs in a folder.
    
    Args:
        input_folder: Folder containing input PDFs
        output_folder: Folder for output PDFs
        processing_func: Function to process each PDF
    """
    os.makedirs(output_folder, exist_ok=True)
    
    for filename in os.listdir(input_folder):
        if filename.lower().endswith('.pdf'):
            input_path = os.path.join(input_folder, filename)
            output_path = os.path.join(output_folder, filename)
            
            try:
                processing_func(input_path, output_path)
                print(f"Processed: {filename}")
            except Exception as e:
                print(f"Error processing {filename}: {e}")

# Example processing function
def rotate_all_pages(input_pdf, output_pdf):
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    for page in reader.pages:
        page.rotate(90)
        writer.add_page(page)
    
    with open(output_pdf, "wb") as f:
        writer.write(f)

# Usage
batch_process_pdfs("input/", "output/", rotate_all_pages)
```

-----

## Advanced Techniques

### Technique 1: Extract Tables (Manual Parsing)

```python
from pypdf import PdfReader

def extract_table_text(pdf_path, page_num):
    """
    Extract text with layout preservation for table parsing.
    Note: For complex tables, consider using libraries like tabula-py
    """
    reader = PdfReader(pdf_path)
    page = reader.pages[page_num]
    
    # Extract with layout mode
    text = page.extract_text(extraction_mode="layout")
    
    # Split into lines
    lines = text.split('\n')
    
    # Parse lines into table structure (customize as needed)
    table_data = []
    for line in lines:
        if line.strip():
            # Split by multiple spaces (adjust pattern as needed)
            cells = [cell.strip() for cell in line.split('  ') if cell.strip()]
            if cells:
                table_data.append(cells)
    
    return table_data

# Usage
table = extract_table_text("report.pdf", 0)
for row in table:
    print(row)
```

### Technique 2: Search and Highlight Text

```python
from pypdf import PdfReader, PdfWriter
from pypdf.generic import DictionaryObject, ArrayObject, FloatObject

def search_text(pdf_path, search_term):
    """Find all occurrences of text in PDF"""
    reader = PdfReader(pdf_path)
    results = []
    
    for page_num, page in enumerate(reader.pages):
        text = page.extract_text()
        
        if search_term.lower() in text.lower():
            results.append({
                'page': page_num + 1,
                'text_preview': text[:200]
            })
    
    return results

# Usage
matches = search_text("document.pdf", "important")
for match in matches:
    print(f"Found on page {match['page']}")
```

### Technique 3: Extract Metadata + Text

```python
from pypdf import PdfReader
import json

def extract_full_document_info(pdf_path):
    """Extract comprehensive document information"""
    reader = PdfReader(pdf_path)
    
    info = {
        'metadata': {},
        'pages': [],
        'num_pages': len(reader.pages),
        'is_encrypted': reader.is_encrypted
    }
    
    # Extract metadata
    if reader.metadata:
        info['metadata'] = {
            key: str(value) for key, value in reader.metadata.items()
        }
    
    # Extract text from each page
    for i, page in enumerate(reader.pages):
        page_info = {
            'page_number': i + 1,
            'text': page.extract_text(),
            'width': float(page.mediabox.width),
            'height': float(page.mediabox.height)
        }
        info['pages'].append(page_info)
    
    return info

# Usage
doc_info = extract_full_document_info("document.pdf")

# Save as JSON
with open("document_info.json", "w", encoding="utf-8") as f:
    json.dump(doc_info, f, indent=2, ensure_ascii=False)
```

### Technique 4: Compare Two PDFs

```python
from pypdf import PdfReader

def compare_pdfs(pdf1_path, pdf2_path):
    """Compare text content of two PDFs"""
    reader1 = PdfReader(pdf1_path)
    reader2 = PdfReader(pdf2_path)
    
    differences = []
    
    max_pages = max(len(reader1.pages), len(reader2.pages))
    
    for i in range(max_pages):
        text1 = reader1.pages[i].extract_text() if i < len(reader1.pages) else ""
        text2 = reader2.pages[i].extract_text() if i < len(reader2.pages) else ""
        
        if text1 != text2:
            differences.append({
                'page': i + 1,
                'difference': 'Content differs'
            })
    
    return differences

# Usage
diffs = compare_pdfs("version1.pdf", "version2.pdf")
print(f"Found {len(diffs)} pages with differences")
```

### Technique 5: Extract by Pattern

```python
from pypdf import PdfReader
import re

def extract_by_pattern(pdf_path, pattern):
    """Extract text matching a regex pattern"""
    reader = PdfReader(pdf_path)
    matches = []
    
    regex = re.compile(pattern)
    
    for i, page in enumerate(reader.pages):
        text = page.extract_text()
        
        page_matches = regex.findall(text)
        if page_matches:
            matches.append({
                'page': i + 1,
                'matches': page_matches
            })
    
    return matches

# Usage examples
# Extract email addresses
emails = extract_by_pattern("document.pdf", r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b')

# Extract phone numbers
phones = extract_by_pattern("document.pdf", r'\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}')

# Extract dates
dates = extract_by_pattern("document.pdf", r'\d{1,2}[/-]\d{1,2}[/-]\d{2,4}')

for result in emails:
    print(f"Page {result['page']}: {result['matches']}")
```

### Technique 6: Memory-Efficient Large PDF Processing

```python
from pypdf import PdfReader, PdfWriter

def process_large_pdf_efficiently(input_pdf, output_pdf, process_page_func):
    """
    Process large PDFs page by page to minimize memory usage.
    
    Args:
        input_pdf: Input PDF path
        output_pdf: Output PDF path
        process_page_func: Function that takes a page and returns modified page
    """
    reader = PdfReader(input_pdf)
    writer = PdfWriter()
    
    # Process one page at a time
    for i, page in enumerate(reader.pages):
        processed_page = process_page_func(page)
        writer.add_page(processed_page)
        
        # Write in chunks to avoid memory buildup
        if (i + 1) % 100 == 0:
            print(f"Processed {i + 1} pages...")
    
    with open(output_pdf, "wb") as f:
        writer.write(f)
    
    print(f"Complete! Processed {len(reader.pages)} pages.")

# Example usage
def my_page_processor(page):
    page.compress_content_streams()
    return page

process_large_pdf_efficiently("large.pdf", "processed.pdf", my_page_processor)
```

### Technique 7: Create PDF from Scratch

```python
from pypdf import PdfWriter
from pypdf.generic import RectangleObject

def create_simple_pdf(output_path, pages_content):
    """
    Create a simple PDF from scratch.
    Note: pypdf is primarily for manipulation. For creating content,
    consider ReportLab or fpdf2.
    """
    writer = PdfWriter()
    
    for content in pages_content:
        # Add blank page (US Letter size)
        page = writer.add_blank_page(width=612, height=792)
        
        # Note: Adding actual text content requires more complex operations
        # This creates blank pages. Use ReportLab for content creation.
    
    with open(output_path, "wb") as f:
        writer.write(f)

# For creating PDFs with text, use ReportLab:
"""
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

c = canvas.Canvas("output.pdf", pagesize=letter)
c.drawString(100, 750, "Hello World")
c.showPage()
c.save()
"""
```

-----

## Performance Tips

1. **Use Context Managers**: Always use `with` statements to ensure proper resource cleanup
1. **Process in Chunks**: For large PDFs, process pages in batches
1. **Compress Content**: Use `page.compress_content_streams()` to reduce file size
1. **Avoid Repeated Reading**: Read PDF once and reuse the reader object
1. **Memory Management**: For very large PDFs, process page by page rather than loading all at once

-----

## Common Issues & Solutions

### Issue: Text Extraction Returns Gibberish

**Solution**: PDF may have custom encoding or be scanned. Try OCR tools like pytesseract.

### Issue: Encrypted PDF Can’t Be Opened

**Solution**: Use `decrypt()` method with correct password.

### Issue: Merged PDF Is Too Large

**Solution**: Compress content streams or reduce image quality before merging.

### Issue: Rotated Pages Look Wrong

**Solution**: Ensure you’re applying rotation before adding to writer.

### Issue: Form Fields Not Updating

**Solution**: Check field names with `get_fields()` and ensure exact match.

-----

## Additional Resources

- **Official Documentation**: https://pypdf.readthedocs.io/
- **GitHub Repository**: https://github.com/py-pdf/pypdf
- **PyPI Package**: https://pypi.org/project/pypdf/

-----

## Quick Reference Card

```python
# Read PDF
from pypdf import PdfReader
reader = PdfReader("file.pdf")

# Access pages
page = reader.pages[0]
num_pages = len(reader.pages)

# Extract text
text = page.extract_text()

# Write PDF
from pypdf import PdfWriter
writer = PdfWriter()
writer.add_page(page)
with open("out.pdf", "wb") as f:
    writer.write(f)

# Merge PDFs
from pypdf import PdfMerger
merger = PdfMerger()
merger.append("file1.pdf")
merger.append("file2.pdf")
merger.write("merged.pdf")
merger.close()

# Encrypt
writer.encrypt(user_password="pass", owner_password="admin")

# Metadata
metadata = reader.metadata
writer.add_metadata({'/Title': 'Doc'})

# Rotate
page.rotate(90)

# Scale
page.scale(0.5, 0.5)

# Watermark
page.merge_page(watermark_page)
```

-----

*pypdf Reference Card - Complete Guide to PDF Processing in Python*