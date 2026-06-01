# Metadata Extraction Rule Validation & Fixes

## Issues Found After Cross-Document Analysis

### Document Type 1: Verification Plans
**Problem**: Extraction rules too specific, don't handle all filename patterns
**Files Found**: dvplan_AXI.md, dvplan_CVXIF.md, dvplan_FENCEI.md, dvplan_FRONTEND.md, dvplan_MMU_SV32.md, dvplan_PMP.md, dvplan_csr-access.md, dvplan_csr-embedded-access.md, dvplan_traps.md

**Current Rule Issues**:
- Name extraction: Only handles "dvplan_ISA_RV32" pattern, fails on "dvplan_csr-access" 
- Subcategory extraction: Missing mappings for CVXIF, PMP, csr-embedded-access, etc.
- Target architecture: Assumes always extractable from filename

**Fixed Rules**:
```yaml
name: # MULTIPLE EXTRACTION PATTERNS
  - pattern_1: "filename_after_dvplan"  # "dvplan_ISA_RV32.md" → "ISA_RV32"
  - pattern_2: "first_heading_module"   # "# Module: AXI" → "AXI" 
  - pattern_3: "clean_special_chars"    # "csr-access" → "CSR_ACCESS"

subcategory: # COMPREHENSIVE MAPPING
  - "ISA_RV32" → "isa"
  - "AXI" → "interface" 
  - "CVXIF" → "interface"
  - "MMU_SV32" → "system"
  - "PMP" → "system"
  - "csr-access" → "system"
  - "csr-embedded-access" → "system"
  - "traps" → "system"
  - "FENCEI" → "isa"
  - "FRONTEND" → "system"

target_architecture: # FALLBACK CHAIN
  - pattern_1: extract_from_filename_if_present  # "RV32" from filename
  - pattern_2: extract_from_applicable_cores     # "CV32A6" → "RV32", "CV64A6" → "RV64"
  - pattern_3: default_to_both                   # "RV32/RV64" if unclear
```

### Document Type 2: Testlists  
**Problem**: Testlist naming highly variable
**Files Found**: testlist_custom.yaml, testlist_isacov.yaml, testlist_csr_embedded.yaml, testlist_cvxif.yaml, testlist_riscv-arch-test-cv32a60x.yaml, etc.

**Current Rule Issues**:
- Name extraction assumes simple pattern, fails on compound names
- Target architecture extraction fails on complex filenames
- Content structure varies significantly

**Fixed Rules**:
```yaml
name: # ROBUST EXTRACTION
  - pattern_1: "after_testlist_prefix"     # "testlist_custom.yaml" → "custom"
  - pattern_2: "compound_name_handling"    # "testlist_riscv-arch-test-cv32a60x" → "riscv_arch_test_cv32a60x"
  - fallback: "full_filename_base"         # Use entire basename if patterns fail

target_architecture: # ARCHITECTURE INFERENCE  
  - pattern_1: "cv32_cv64_detection"       # "cv32a60x" → "RV32", "cv64a6" → "RV64"
  - pattern_2: "rv32_rv64_explicit"        # Look for "rv32" or "rv64" in filename  
  - pattern_3: "content_analysis"          # Analyze test content for architecture
  - fallback: "RV32/RV64"                  # Default to both if unclear

content_structure_handling: # VARIABLE FORMATS
  - yaml_with_testlist_key: "standard format"
  - yaml_with_common_config: "config inheritance format"  
  - yaml_minimal: "simple test list only"
```

### Document Type 3: ISA Specifications
**Problem**: Heading formats and file organization varies
**Files Checked**: rv32.adoc, a-st-ext.adoc, various extension files

**Current Rule Issues**: 
- Subcategory extraction assumes specific heading patterns
- Chapter title variations not handled
- Extension naming inconsistent

**Fixed Rules**:
```yaml
name: # HEADING EXTRACTION ROBUSTNESS
  - pattern_1: "=== RV32I Base Integer Instruction Set" → "RV32I Base Integer Instruction Set"
  - pattern_2: "=== ext:a[] Extension for Atomic Instructions" → "ext:a[] Extension for Atomic Instructions" 
  - pattern_3: "=== A Extension for Atomic Instructions" → "A Extension for Atomic Instructions"
  - fallback: "filename_based" # Use filename if heading unclear

subcategory: # CONTENT-BASED ROBUST DETECTION
  primary_patterns: # Look for these heading patterns first
    - "Base Integer" → "base_integer"
    - "Atomic Instructions" | "ext:a[]" → "atomic"  
    - "Multiplication and Division" | "ext:m[]" → "multiply_divide"
    - "Single-Precision" | "ext:f[]" → "float_single"
    - "Double-Precision" | "ext:d[]" → "float_double"
    - "Compressed" | "ext:c[]" → "compressed"
    - "Control and Status Register" | "Zicsr" → "csr"
  
  secondary_patterns: # Content analysis if headings don't match
    - instruction_mnemonics_analysis: 
        - ["ADDI", "ADD", "SUB"] → "base_integer"
        - ["MUL", "DIV", "REM"] → "multiply_divide"  
        - ["LR.W", "SC.W", "AMOADD"] → "atomic"
  
  fallback: "specification_general" # Generic subcategory if patterns fail
```

### Document Type 9: CVA6 Instructions
**Problem**: Multiple file formats and naming conventions  
**Files Found**: RISCV_Instructions_RV32I.rst, RISCV_Instructions_RV32M.rst, RISCV_Instructions_RVZicsr.rst, etc.

**Current Rule Issues**:
- Feature extraction assumes single pattern
- Extension parsing inadequate 
- Architecture detection from varied filenames

**Fixed Rules**:
```yaml
name: # INSTRUCTION SET EXTRACTION  
  - pattern_1: "RISCV_Instructions_RV32I.rst" → "RV32I"
  - pattern_2: "RISCV_Instructions_RV32M.rst" → "RV32M" 
  - pattern_3: "RISCV_Instructions_RVZicsr.rst" → "RVZicsr"
  - pattern_4: "RISCV_Instructions_RVZba.rst" → "RVZba"
  
feature: # FEATURE NAME NORMALIZATION
  - "RV32I" → "Base Integer Instructions"
  - "RV32M" → "Multiplication and Division"  
  - "RV32A" → "Atomic Instructions"
  - "RV32C" → "Compressed Instructions"
  - "RVZicsr" → "Control and Status Register Instructions"
  - "RVZifencei" → "Instruction Fence"
  - pattern_match: "RVZ*" → "{extension_name} Extension"

target_architecture: # ROBUST ARCHITECTURE DETECTION
  - "RV32*" → "RV32"
  - "RV64*" → "RV64" 
  - "RV*" → "RV32/RV64"  # Generic RISC-V
  - applicability_table_analysis: # Parse configuration tables in files
      "CV32A6*" → "RV32"
      "CV64A6*" → "RV64"
```

## Implementation Strategy

1. **Multi-Pattern Extraction**: Each field uses multiple extraction patterns with fallbacks
2. **Content Analysis Backup**: When filename/heading patterns fail, analyze document content
3. **Robust Defaults**: Provide sensible defaults when extraction completely fails
4. **Validation Rules**: Include confidence scoring for extraction quality

## Field Extraction Robustness Levels

- **Level 1 (Robust)**: Universal fields that can be extracted reliably across all documents
- **Level 2 (Moderate)**: Content-specific fields that work for most documents of a type  
- **Level 3 (Fragile)**: Document-specific fields that may fail for some variations

This analysis ensures the extraction rules will work across the full variety of documents found in each document type, not just the cherry-picked examples initially used.