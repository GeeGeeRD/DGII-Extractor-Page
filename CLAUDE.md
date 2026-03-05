# DGII-Extractor-Page

## What It Does

Client-side web application that extracts data from Dominican Republic tax authority (DGII) PDF forms and exports them to structured Excel files. No server needed — runs entirely in the browser.

## Architecture

- **Single file:** `index.html` (~1600 lines) contains all HTML, CSS, and JavaScript
- **PDF.js 3.11.174** — Extracts text with x/y coordinates from PDF pages
- **XLSX 0.18.5** — Generates Excel (.xlsx) files from extracted data

## Extraction Flow

1. User uploads PDF files (drag-and-drop or file input)
2. `extractFromPDF()` loads each PDF with PDF.js
3. For each page: extracts text items with `{text, x, y}` coordinates
4. `groupIntoRows(items, tolerance=5)` groups items by Y-position into logical rows
5. Form type is auto-detected from text patterns (e.g., `/E/` → Anexo E)
6. Type-specific parser function extracts fields using x-coordinate ranges
7. `exportExcel()` outputs structured data to Excel with proper formatting

## Supported Forms

| Form | Parser Function | Description |
|------|----------------|-------------|
| IR-17 | `parseIR17()` | Retenciones |
| IT-1 | `parseIT1()` | Ventas (ITBIS) |
| Anexo A | `parseAnexoA()` | 607 Compras |
| IR-3 | `parseIR3()` | Salarios |
| IR-2 | `parseIR2()` | Declaración Jurada del ISR Sociedades |
| ACT | `parseACT()` | Activos |
| A-1 | `parseA1()` | Anexo A-1 |
| B1 | `parseB1()` | Anexo B1 |
| D | `parseD()` | Anexo D |
| D1 | `parseD1()` | Anexo D1 |
| D2 | `parseD2()` | Anexo D2 |
| **E** | **`parseE()`** | **Anexo E — Datos Complementarios** |
| G | `parseG()` | Anexo G |
| H1 | `parseH1()` | Anexo H1 — Accionistas |
| H2 | `parseH2()` | Anexo H2 — Beneficiarios |
| J | `parseJ()` | Anexo J |

## Key Code Sections

| Section | Lines (approx) | Description |
|---------|------|-------------|
| CSS Styles | 9-115 | All styling |
| Field definitions | 115-530 | `IR17_FIELDS`, `IT1_FIELDS`, `ANEXOA_FIELDS`, `IR3_FIELDS`, `*_DESC` constants |
| Utility functions | 535-550 | `groupIntoRows()`, `parseFmt()`, `lastNum()`, `isNum()` |
| Form parsers | 550-1076 | Individual parser functions for each form type |
| Form detection | 1078-1093 | `detectIR2Type()` — identifies IR-2 sub-forms |
| Metadata extraction | 1095-1115 | `extractMeta()` — RNC, período, razón social |
| Main extraction | 1127-1168 | `extractFromPDF()` — orchestrates the full flow |
| UI event handlers | 1173-1470 | File handling, preview rendering, tabs |
| Excel export | 1474-1580 | `exportExcel()` — generates .xlsx output |

## Anexo E Structure

The Anexo E form has 4 sections:

### Section A: Pérdidas de Años Anteriores (rows 1-5, row 6 = totals)
12 columns per row (a through l):
- a) Año de la Pérdida
- b) Pérdida al Inicio del Periodo
- c) Índice de Inflación (%)
- d) Cantidad Ajuste Por Inflación
- e) Pérdida Ajustada Por Inflación
- f) Periodos por Compensar
- g) Pérdida a Compensar
- h) Renta Neta Imponible Antes de la Pérdida
- i) % de la Renta Neta Imponible
- j) Límite de Pérdida a Compensar
- k) Pérdida Compensable Periodo
- l) Pérdida Pendiente de Compensar

Data keys: `c{row}_{col}` (e.g., `c1_a`, `c3_g`, `c5_l`)
Total keys: `c6_total_{col}` (e.g., `c6_total_d`, `c6_total_l`)

### Section B: Distribución de los Beneficios (rows 7-10)
### Section C: Saldo de Pérdidas de Capital (rows 11-14)
### Section D: Ingresos Brutos Sujetos al Pago de Anticipos (rows 15-22)

Sections B-D use single-value extraction: `c{row}` (e.g., `c7`, `c15`, `c22`)

## Debugging

The app generates debug logs with x/y coordinates for every extracted text item:
```
Y=129: [x=30]"1"  [x=81]"2,019.00"  [x=137]"4,035,793.75"  [x=219]"3.29" ...
```

To view: process a PDF, then click "Ver texto extraído" in the debug section.

### X-Coordinate Ranges for Anexo E Section A

These ranges map PDF text x-positions to columns (calibrated from DGII PDF output):

| Column | X Range | Header X |
|--------|---------|----------|
| a | 60-120 | 42 |
| b | 120-210 | 104 |
| c | 210-260 | 165 |
| d | 260-325 | 234 |
| e | 325-400 | 297 |
| f | 400-450 | 373 |
| g | 450-545 | 426 |
| h | 545-595 | 491 |
| i | 595-655 | 559 |
| j | 655-730 | 624 |
| k | 730-775 | 686 |
| l | 775+ | 752 |

## Helper Functions

- `isNum(t)` — matches `/^-?[\d,]+\.\d{2}$/` (numbers with 2 decimal places, optional commas)
- `parseFmt(s)` — removes commas and parses to float
- `lastNum(row)` — returns the last numeric value in a row (used for single-value fields)
- `groupIntoRows(items, tol)` — groups text items by Y-coordinate with pixel tolerance
