# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dolphin is a document image parsing system that extracts structured content from documents using a Vision-Language Model (VLM). It uses a two-stage parsing approach:
1. **Stage 1**: Layout analysis with reading order prediction
2. **Stage 2**: Element-wise content parsing (text, tables, formulas, code)

The project is built on Qwen2.5-VL model via HuggingFace Transformers.

## Common Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Download model
huggingface-cli download ByteDance/Dolphin-v2 --local-dir ./hf_model

# Page-level parsing (single file or directory)
python demo_page.py --model_path ./hf_model --input_path ./demo/page_imgs/page_1.png --save_dir ./results
python demo_page.py --model_path ./hf_model --input_path ./demo/page_imgs --save_dir ./results --max_batch_size 8

# Element-level parsing (text, table, formula, or code)
python demo_element.py --model_path ./hf_model --input_path <image_path> --element_type [table|formula|text|code] --save_dir ./results

# Layout-only parsing
python demo_layout.py --model_path ./hf_model --input_path ./demo/page_imgs/page_1.png --save_dir ./results

# Code formatting (pre-commit)
pre-commit run --all-files
```

## Architecture

### Entry Points
- `demo_page.py` - Full document parsing pipeline (layout + element recognition)
- `demo_element.py` - Single element parsing (table, formula, text, code)
- `demo_layout.py` - Layout detection only (bounding boxes + reading order)

### Core Components
- **DOLPHIN class** (in each demo_*.py) - Wraps Qwen2.5-VL model for inference with batch support
- `utils/utils.py` - Image processing, coordinate mapping, layout parsing, visualization
- `utils/markdown_utils.py` - Converts parsed results to Markdown format

### Processing Pipeline (demo_page.py)
1. `process_document()` - Entry point, handles PDF/image input
2. `process_single_image()` - Runs stage 1 (layout) then stage 2 (element parsing)
3. `process_elements()` - Groups elements by type (tab, equ, code, text, fig) for batch processing
4. `process_element_batch()` - Batch inference for elements of same type

### Element Labels
- `para` - Paragraph text
- `tab` - Table
- `equ` - Formula/equation
- `code` - Code block
- `fig` - Figure
- `sec_0` to `sec_5` - Section headings (different levels)
- `list` - List items
- `distorted_page` - Fallback for photographed/distorted documents

### Output Structure
Results are saved to:
- `output_json/` - Structured JSON with elements, bboxes, reading order
- `markdown/` - Converted Markdown files
- `markdown/figures/` - Extracted figure images
- `layout_visualization/` - Annotated images showing detected layout

## Code Style

- Line length: 120 characters (Black + flake8)
- Import sorting: isort with Black profile
- Pre-commit hooks: isort, Black, flake8, trailing whitespace, YAML validation
