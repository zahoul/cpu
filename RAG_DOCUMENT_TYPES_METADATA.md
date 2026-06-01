# CVA6 RAG Document Types & Metadata Structure
## EXTRACTION ROBUSTNESS UPDATE

**IMPORTANT**: All extraction patterns have been validated against multiple real documents per type and include:
- **MULTIPLE PATTERNS**: Primary + fallback extraction methods for each field
- **COMPREHENSIVE MAPPINGS**: All filename variations found in actual repositories  
- **ROBUST DEFAULTS**: Sensible fallbacks when extraction fails
- **REAL DOCUMENT VALIDATION**: Tested against actual file variations, not just examples

## Document Type 1: Verification Plans

### Purpose
**What:** Structured verification requirements that define what needs to be tested and how
**Why:** Provides the authoritative specification of verification goals that test writing agents must fulfill
**When Used:** Primary input for all test generation - defines the "what to test" requirements

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Core Requirements (MUST SEND)
verification_goals: ["All possible rs1 registers are used", "Overflow behavior tested"]
test_approach: "constrained_random"  # or "directed", "assertion_based"
pass_criteria: "Check RM"
coverage_requirements: ["isacov.rv32i_addi_cg.cp_rs1", "isacov.rv32i_addi_cg.cp_rd"]

# Context Information (SHOULD SEND)  
feature_description: "addi rd, rs1, imm[11:0] - Add immediate instruction"
verification_tag: "VP_ISA_RV32_F000_S000_I000"  # For traceability
applicable_cores: ["CV32A6_v0.1.0", "CV32A6-step2"]
requirement_references: ["./RISCV_Instructions.rst"]

# Scope Boundaries (HELPFUL TO SEND)
architecture_scope: "RV32"
feature_category: "Register-Immediate Instructions" 
item_priority: "mandatory"  # infer from plan structure
```

**Agent Decision Support:** This data enables agents to understand WHAT to test, HOW to approach testing, and WHEN the test passes/fails.

### Source Locations
```
Primary verification plans (Markdown):
cva6/verif/docs/VerifPlans/source/*.md
├── dvplan_ISA_RV32.md (296KB) - RV32 instruction verification
├── dvplan_AXI.md (18KB) - AXI interface verification  
├── dvplan_MMU_SV32.md (28KB) - MMU verification
├── dvplan_PMP.md (771KB) - PMP verification
├── dvplan_CVXIF.md (34KB) - CVXIF interface verification
├── dvplan_FENCEI.md (20KB) - FENCE.I instruction verification
├── dvplan_FRONTEND.md (11KB) - Frontend verification
└── dvplan_traps.md (23KB) - Exception/trap verification

```

### Document Structure Example
```markdown
## Feature: RV32I Register-Immediate Instructions
### Sub-feature: 000_ADDI
#### Item: 000
* **Requirement location:** ./RISCV_Instructions.rst
* **Feature Description:** addi rd, rs1, imm[11:0]
* **Verification Goals:** All possible rs1 registers are used
* **Pass/Fail Criteria:** Check RM
* **Test Type:** Constrained Random
* **Coverage Method:** Functional Coverage
* **Applicable Cores:** CV32A6_v0.1.0, CV32A6-step2
* **Unique verification tag:** VP_ISA_RV32_F000_S000_I000
* **Link to Coverage:** isacov.rv32i_addi_cg.cp_rs1
```


### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "verification_plan"                # Fixed value for this document type
file_path: "cva6/verif/docs/VerifPlans/source/dvplan_ISA_RV32.md"  # Full repository path
name: "ISA_RV32"                                  # ROBUST EXTRACTION: Remove "dvplan_" prefix: "dvplan_ISA_RV32.md"→"ISA_RV32", "dvplan_csr-access.md"→"csr-access", OR from heading: "# Module: AXI"→"AXI"
name_type: "module"                               # Fixed: "module" for verification plans
target_architecture: "RV32"                      # MULTIPLE PATTERNS: From filename: "RV32"→"RV32", "RV64"→"RV64", OR from applicable cores: "CV32A6*"→"RV32", "CV64A6*"→"RV64", OR default "RV32/RV64" 
word_size: 32                                     # Derive from target_architecture: RV32→32, RV64→64, mixed→32
category: "verification"                          # Fixed: "verification" for all verification plans
subcategory: "isa"                                # COMPREHENSIVE MAPPING: "ISA_RV32"→"isa", "AXI"→"interface", "CVXIF"→"interface", "MMU_SV32"→"system", "PMP"→"system", "csr-access"→"system", "csr-embedded-access"→"system", "traps"→"system", "FENCEI"→"isa", "FRONTEND"→"system"

# Content-Specific Fields
feature: "Register-Immediate Instructions"         # Extract from "## Feature:" line content
sub_feature: "ADDI"                               # Extract from "### Sub-feature:" after underscore: "000_ADDI" → "ADDI"
content_type: "constrained_random"                # Map from "Test Type": "Constrained Random"→"constrained_random", "Directed"→"directed"
source_type: "project_internal"                   # Fixed: "project_internal" for CVA6 verification plans
target_cores: ["CV32A6_v0.1.0", "CV32A6-step2"]  # Extract from "* **Applicable Cores:**" line, split by comma

# Document-Specific Fields
verification_tag: "VP_ISA_RV32_F000_S000_I000"   # Extract from "* **Unique verification tag:**" line
verification_goals: ["All possible rs1 registers are used", "All possible rd registers are used"]  # Extract from "* **Verification Goals**" section, one per bullet
pass_fail_criteria: "Check RM"                   # Extract from "* **Pass/Fail Criteria:**" line content
coverage_links: ["isacov.rv32i_addi_cg.cp_rs1", "isacov.rv32i_addi_cg.cp_rd"]  # Extract from "* **Link to Coverage:**" line, split by whitespace
requirement_location: "./RISCV_Instructions.rst" # Extract from "* **Requirement location:**" line content

# Examples of extraction patterns:
# MARKDOWN FILES:
# FROM: "### Sub-feature: 000_ADDI"  →  sub_feature: "ADDI"  
# FROM: "* **Test Type:** Constrained Random"  →  content_type: "constrained_random"
# FROM: "* **Applicable Cores:** CV32A6_v0.1.0, CV32A6-step2"  →  target_cores: ["CV32A6_v0.1.0", "CV32A6-step2"]

```

## Document Type 2: Design Implementation

### Purpose
**What:** Actual CVA6 SystemVerilog implementation showing how features are built in hardware
**Why:** Provides implementation-specific details that test agents need to create realistic, hardware-accurate tests
**When Used:** When agents need to understand HOW CVA6 actually implements a feature (interfaces, timing, constraints)

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Hardware Interface Details (MUST SEND)
module_interfaces:
  inputs: [{"name": "operand_a_i", "width": 32, "type": "logic"}]
  outputs: [{"name": "result_o", "width": 32, "type": "logic"}]  
  parameters: [{"name": "WIDTH", "default": 32, "configurable": true}]

# Implementation Constraints (MUST SEND)
hardware_limitations:
  data_width: 32  # or parameterizable
  supported_operations: ["ADD", "SUB", "AND", "OR", "XOR"]  # extract from case statements
  timing_characteristics: "combinatorial"  # or "pipelined", "multi-cycle"

# Design Context (SHOULD SEND)
module_hierarchy: "core.execution_unit.alu"
dependencies: ["ariane_pkg", "riscv_pkg"]  # import statements
design_category: "execution_unit"  # vs memory_unit, control_unit

# Integration Points (HELPFUL TO SEND)
connected_modules: ["issue_stage", "writeback_stage"]  # infer from port connections
signal_protocols: "ready/valid handshaking"  # detect from signal patterns
error_handling: ["overflow_flag", "invalid_op_exception"]  # extract from implementation
```

**Agent Decision Support:** This data helps agents understand implementation constraints, interface requirements, and realistic test scenarios based on actual hardware.

### Source Locations
```
Primary: cva6/core/*.sv, cva6/corev_apu/*.sv
Core modules:
├── cva6.sv (75KB) - Top-level CVA6 module
├── alu.sv (16KB) - Arithmetic Logic Unit  
├── decoder.sv (95KB) - Instruction decoder
├── frontend/frontend.sv (24KB) - Instruction fetch
├── cache_subsystem/*.sv - Cache implementation
├── cva6_mmu/*.sv - Memory Management Unit
├── load_store_unit.sv (33KB) - LSU implementation
├── issue_stage.sv (13KB) - Issue stage
├── commit_stage.sv (17KB) - Commit stage
└── ex_stage.sv (27KB) - Execute stage
```

### Document Structure Example
```systemverilog
module alu
  import ariane_pkg::*;
#(
  parameter int unsigned WIDTH = 32
)(
  input  logic                 clk_i,
  input  logic                 rst_ni,
  input  logic [WIDTH-1:0]     operand_a_i,
  input  logic [WIDTH-1:0]     operand_b_i,
  input  fu_op                 operator_i,
  output logic [WIDTH-1:0]     result_o,
  output logic                 comparison_result_o
);
```

### Metadata Structure  
```yaml
# Universal Fields (harmonized schema)
document_type: "design_implementation"            # Fixed value for this document type
file_path: "cva6/core/alu.sv"                    # Full repository path
name: "alu"                                       # Extract from module declaration: "module alu" → "alu"
name_type: "module"                               # Fixed: "module" for SystemVerilog modules
target_architecture: "RV32/RV64"                 # Detect from parameterization: WIDTH parameter → "RV32/RV64", fixed 32 → "RV32"
word_size: 32                                     # Extract from WIDTH parameter default: "parameter int unsigned WIDTH = 32" → 32
category: "hardware"                              # Fixed: "hardware" for all design implementation files
subcategory: "execution_unit"                    # Map from path: "core/" → "execution_unit", "cache_subsystem/" → "memory", "frontend/" → "fetch"

# Content-Specific Fields  
dependencies: ["ariane_pkg", "riscv_pkg"]        # Extract from import statements: "import ariane_pkg::*;" → ["ariane_pkg"]

# Document-Specific Fields
input_ports: [                                   # Parse module port list for input declarations
  {"name": "operand_a_i", "width": 32, "type": "logic"},  # FROM: "input logic [WIDTH-1:0] operand_a_i"
  {"name": "operand_b_i", "width": 32, "type": "logic"},  # FROM: "input logic [WIDTH-1:0] operand_b_i"  
  {"name": "operator_i", "width": 6, "type": "fu_op"}     # FROM: "input fu_op operator_i"
]
output_ports: [                                  # Parse module port list for output declarations
  {"name": "result_o", "width": 32, "type": "logic"},           # FROM: "output logic [WIDTH-1:0] result_o"
  {"name": "comparison_result_o", "width": 1, "type": "logic"}  # FROM: "output logic comparison_result_o"
]
parameters: [                                    # Extract parameter declarations
  {"name": "WIDTH", "default_value": 32, "type": "int unsigned"}  # FROM: "parameter int unsigned WIDTH = 32"
]

# Examples of extraction patterns:
# FROM: "module alu import ariane_pkg::*;"  →  name: "alu", dependencies: ["ariane_pkg"]
# FROM: "input logic [WIDTH-1:0] operand_a_i,"  →  {"name": "operand_a_i", "width": "WIDTH", "type": "logic"}
# FROM: "cva6/core/alu.sv"  →  subcategory: "execution_unit" (core/ maps to execution_unit)
# FROM: "cva6/cache_subsystem/wt_cache.sv"  →  subcategory: "memory" (cache_subsystem/ maps to memory)
```

## Document Type 3: ISA Specifications (RISC-V Official Manual)

### Purpose
**What:** Official RISC-V architecture specification defining authoritative instruction behavior
**Why:** Provides the "golden reference" for correct instruction semantics that test agents use for expected behavior
**When Used:** When agents need authoritative instruction definitions, architectural rules, and normative requirements

### RAG Agent Data Extraction  
**What RAG agents should send to main test writing agents:**

```yaml
# Normative Behavior (MUST SEND)
instruction_semantics:
  operation: "ADDI adds sign-extended 12-bit immediate to register rs1"
  operand_constraints: {"rs1": "any_register", "rd": "any_register", "imm": "12_bit_signed"}
  architectural_effects: "updates_rd_register_only"
  side_effects: "none"

# Register Model Rules (MUST SEND)
register_behavior:
  x0_constraint: "hardwired_to_zero"  # x0 always reads 0, writes ignored
  register_width: 32  # XLEN for this architecture
  register_count: 32
  special_registers: ["x0_zero", "pc_program_counter"]

# Instruction Encoding (SHOULD SEND)
encoding_format: "I-type"
bit_layout: {"imm": "[31:20]", "rs1": "[19:15]", "funct3": "[14:12]", "rd": "[11:7]", "opcode": "[6:0]"}
opcode_value: "0010011"

# Architectural Context (HELPFUL TO SEND)
extension_category: "base_integer"
privilege_requirements: "any_level"  # user, supervisor, machine
exception_conditions: "none"  # or specific conditions that trigger exceptions
```

**Agent Decision Support:** This data provides the authoritative "correct behavior" that generated tests must verify against.

### Source Locations
```
Primary: riscv-isa-manual/src/**/*.adoc
Main specification:
├── riscv-spec.adoc - Main specification document
├── unpriv/base.adoc - Base instruction sets chapter
├── unpriv/rv32.adoc - RV32I base integer instruction set
├── unpriv/rv64.adoc - RV64I base integer instruction set
├── unpriv/a-st-ext.adoc - Atomic instruction extension
├── unpriv/m-st-ext.adoc - Multiply/divide extension  
├── unpriv/c-st-ext.adoc - Compressed instruction extension
├── unpriv/zicsr.adoc - CSR instruction extension
└── unpriv/code-examples.adoc - Code examples

Privileged specification:
├── priv/priv.adoc - Main privileged architecture
├── priv/machine.adoc - Machine-level architecture
├── priv/supervisor.adoc - Supervisor-level architecture
├── priv/csrs.adoc - Control and status registers
└── priv/hypervisor.adoc - Hypervisor extension
```

### Document Structure Example
```asciidoc
=== RV32I Base Integer Instruction Set

This section describes the RV32I base integer instruction set.

==== Programmers' Model for Base Integer ISA

For RV32I, the 32 `x` registers are each 32 bits wide, i.e., XLEN=32.
Register `x0` is hardwired with all bits equal to 0.
General purpose registers `x1`–`x31` hold values that various
instructions interpret as a collection of Boolean values, or as two's
complement signed binary integers or unsigned binary integers.

==== Integer Computational Instructions

===== Integer Register-Immediate Instructions

ADDI adds the sign-extended 12-bit immediate to register rs1.
Arithmetic overflow is ignored and the result is simply the low XLEN bits of the result.
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "isa_specification"               # Fixed value for this document type
file_path: "riscv-isa-manual/src/unpriv/rv32.adoc"  # Full repository path
name: "RV32I Base Integer Instruction Set"      # Extract from main chapter heading: "=== RV32I Base Integer Instruction Set"
name_type: "specification"                      # Fixed: "specification" for ISA manual chapters
target_architecture: "RV32"                     # Extract from filename: "rv32.adoc" → "RV32", or chapter title "RV32I" → "RV32"
word_size: 32                                   # Extract from text: "32 bits wide" or "XLEN=32" → 32
category: "specification"                       # Fixed: "specification" for all ISA specification documents
subcategory: "base_integer"                     # Extract from document content: main headings, intro text, or instruction mnemonics (see content-based extraction patterns below)

# Content-Specific Fields
content_type: "specification"                   # Fixed: "specification" for authoritative ISA documents
source_type: "official"                         # Fixed: "official" for RISC-V ISA manual

# Document-Specific Fields  
chapter: "RV32I Base Integer Instruction Set"   # Extract from main section heading
register_model: "32_general_purpose_registers"  # Extract from text: "the 32 `x` registers" → "32_general_purpose_registers"
special_registers: ["x0_hardwired_zero", "pc"]  # Extract from text: "Register `x0` is hardwired" → ["x0_hardwired_zero"]
instruction_categories: {                       # Extract from subsection headings
  "integer_register_immediate": ["ADDI", "SLTI", "SLTIU", "XORI", "ORI", "ANDI"],  # FROM: "===== Integer Register-Immediate Instructions"
  "integer_register_register": ["ADD", "SUB", "SLL", "SLT", "SLTU", "XOR", "SRL", "SRA"],  # FROM: "===== Integer Register-Register Operations"  
  "load_store": ["LB", "LH", "LW", "LBU", "LHU", "SB", "SH", "SW"],  # FROM: "===== Load and Store Instructions"
  "control_flow": ["BEQ", "BNE", "BLT", "BGE", "BLTU", "BGEU", "JAL", "JALR"]  # FROM: "===== Control Transfer Instructions"
}

# Examples of extraction patterns:
# FROM: "=== RV32I Base Integer Instruction Set"  →  name: "RV32I Base Integer Instruction Set"
# FROM: "For RV32I, the 32 `x` registers are each 32 bits wide"  →  register_model: "32_general_purpose_registers", word_size: 32
# FROM: "Register `x0` is hardwired with all bits equal to 0"  →  special_registers: ["x0_hardwired_zero"]  
# FROM: "===== Integer Register-Immediate Instructions"  →  instruction_categories: {"integer_register_immediate": [...]}
# FROM: "riscv-isa-manual/src/unpriv/rv32.adoc"  →  target_architecture: "RV32"

# Subcategory extraction from document content:
# Extract from chapter headings, section titles, and document context:

# Content-based patterns → subcategory mappings:
# Look for main heading patterns:
# "=== RV32I Base Integer Instruction Set" → "base_integer"  
# "=== RV64I Base Integer Instruction Set" → "base_integer"
# "=== Integer Multiplication and Division Extension" → "multiply_divide"
# "=== Atomic Instructions" → "atomic" 
# "=== Single-Precision Floating-Point" → "float_single"
# "=== Double-Precision Floating-Point" → "float_double"
# "=== Compressed Instructions" → "compressed"
# "=== Control and Status Register Instructions" → "csr"
# "=== Machine-Level ISA" → "machine_level"
# "=== Supervisor-Level ISA" → "supervisor_level" 
# "=== Hypervisor Extension" → "hypervisor"

# Alternative content patterns if main heading not found:
# Look for chapter intro text:
# "This chapter describes the base integer instruction set" → "base_integer"
# "multiplication and division operations" → "multiply_divide"
# "atomic memory operations" → "atomic"
# "single-precision floating-point" → "float_single" 
# "double-precision floating-point" → "float_double"
# "16-bit instruction encodings" → "compressed"
# "control and status register" → "csr"
# "machine mode" → "machine_level"
# "supervisor mode" → "supervisor_level"
# "hypervisor extension" → "hypervisor"
# "application profile" → "application_profile"
# "infrastructure profile" → "infrastructure_profile"
# "debug module" → "debug_specification"

# If no content patterns match, extract from document context:
# Look for instruction mnemonics in document to infer category:
# Contains "ADDI", "ADD", "SUB" → "base_integer"
# Contains "MUL", "DIV", "REM" → "multiply_divide"
# Contains "LR.W", "SC.W", "AMO" → "atomic"
# Contains "FLW", "FSW", "FADD.S" → "float_single"
# Contains "FLD", "FSD", "FADD.D" → "float_double" 
# Contains "C.ADDI", "C.LW" → "compressed"
# Contains "CSRRW", "CSRRS" → "csr"
```

## Document Type 4: Test Examples

### Purpose  
**What:** Existing test implementations showing proven patterns and approaches
**Why:** Provides concrete examples of test structure, constraints, and implementation techniques for agent reference
**When Used:** When agents need to learn from existing test patterns or avoid duplicating existing coverage

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Test Pattern Examples (MUST SEND)
test_structure:
  test_approach: "directed"  # vs constrained_random, assertion_based
  instruction_sequences: ["li x1, 0x12345678", "csrrw x2, mstatus, x1", "csrrs x3, mie, x1"]
  validation_method: "compare_against_expected"
  result_checking: "branch_on_mismatch"

# Constraint Patterns (MUST SEND)  
constraint_examples:
  register_usage: {"avoid": ["x0"], "preferred": ["x1", "x2", "x3"], "temporary": ["x30", "x31"]}
  value_patterns: ["0xa5a5a5a5", "0x5a5a5a5a", "0x00000000", "0xffffffff"]
  edge_cases: ["boundary_values", "overflow_conditions", "zero_operands"]

# Test Infrastructure (SHOULD SEND)
framework_usage:
  includes: ["model_test.h", "riscv_test.h"]
  macros_used: ["TEST_IMM_OP", "RVTEST_CODE_BEGIN", "RVTEST_CODE_END"]
  result_communication: "tohost_protocol"
  exception_handling: "mtvec_setup_pattern"

# Coverage Insights (HELPFUL TO SEND)
coverage_achieved: ["rs1_register_coverage", "immediate_value_coverage"]
test_limitations: ["only_positive_values", "single_instruction_focus"]
related_tests: ["similar_csr_tests", "related_instruction_tests"]
```

**Agent Decision Support:** This data helps agents learn proven test patterns and avoid reinventing test infrastructure.

### Source Locations
```
Assembly Tests: cva6/verif/tests/custom/**/*.S
SystemVerilog Tests: cva6/verif/env/uvme_cva6/**/*.sv  
Referenced from Testlists: Tests specified in testlist_*.yaml files
```

### Document Structure Example
```assembly
# CSR Access Test
.section .text.init
.globl _start
_start:
    li x1, 0x12345678          # Load immediate into x1
    csrrw x2, mstatus, x1      # Read-write mstatus CSR
    csrrs x3, mie, x1          # Read-set mie CSR
    nop
    j _end
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "test_example"                    # Fixed value for this document type
file_path: "cva6/verif/tests/custom/CSR/csr_access_failing_tests/riscv_marchid_csr_test_0.S"  # Full repository path
name: "riscv_marchid_csr_test_0"                # Extract from filename without extension: "riscv_marchid_csr_test_0.S" → "riscv_marchid_csr_test_0"
name_type: "test"                               # Fixed: "test" for test example files
target_architecture: "RV32"                     # Extract from filename suffix: "_32" → "RV32", or detect from code: "li x1, 0x12345678" (32-bit) → "RV32"
word_size: 32                                   # Derive from architecture: RV32 → 32, RV64 → 64
category: "test"                                # Fixed: "test" for all test example files
subcategory: "csr_access"                       # Extract from path: "CSR/" → "csr_access", "ALU/" → "alu", "Memory/" → "memory"

# Content-Specific Fields
feature: "MARCHID CSR"                          # Extract from filename: "marchid" → "MARCHID CSR", or from comments
content_type: "directed"                        # Infer from structure: specific sequence → "directed", randomized → "constrained_random"
test_framework: "model_test"                    # Extract from #include: "#include \"model_test.h\"" → "model_test"

# Document-Specific Fields
language: "assembly"                            # Detect from file extension: ".S" → "assembly", ".sv" → "systemverilog"
instructions_used: ["li", "csrr", "csrrw", "csrrs", "csrrc", "bne", "j"]  # Parse assembly to extract instruction mnemonics
test_pattern: "csr_read_write_validation"       # Infer from instruction patterns: CSR ops + validation → "csr_read_write_validation"
validation_method: "expected_value_comparison"  # Extract from code pattern: "bne x31, x6, csr_fail" → "expected_value_comparison"

# Examples of extraction patterns:
# FROM: "cva6/verif/tests/custom/CSR/csr_access_failing_tests/riscv_marchid_csr_test_0.S"  →  subcategory: "csr_access" (CSR/ path)
# FROM: "#include \"model_test.h\""  →  test_framework: "model_test"
# FROM: "csrr x31, 3858" + "li x6, 0x00000003" + "bne x31, x6, csr_fail"  →  validation_method: "expected_value_comparison"
# FROM: "li x1, 0x12345678" (32-bit immediate)  →  target_architecture: "RV32"
# FROM: Multiple CSR instructions (csrr, csrrw, csrrs, csrrc)  →  test_pattern: "csr_read_write_validation"
```

## Document Type 5: Test Configurations (Testlists)

### Purpose
**What:** Test execution configuration defining how tests are compiled, run, and integrated into test suites  
**Why:** Provides the infrastructure knowledge agents need to generate properly configured, executable tests
**When Used:** When agents need to understand test compilation, execution environment, and integration requirements

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Compilation Configuration (MUST SEND)
build_settings:
  gcc_options: "-static -mcmodel=medany -fvisibility=hidden -nostdlib -nostartfiles"
  include_paths: ["/riscv-tests/isa/macros/scalar/", "/cva6/verif/tests/custom/common/"]
  target_architecture: "rv32"  # affects compiler flags
  privilege_mode: "machine"  # affects memory layout

# Test Organization (MUST SEND)
test_suite_structure:
  test_category: "isa_tests"  # vs custom_tests, compliance_tests
  execution_count: 1  # iterations per test
  test_naming_convention: "rv32ui-p-addi"  # pattern for generated tests
  file_organization: "separate_file_per_instruction"

# Infrastructure Requirements (SHOULD SEND)
execution_environment:
  simulator_options: "+signature=signature.log +timeout=10000"
  memory_model: "physical_addressing"  # vs virtual_addressing
  core_configuration: "cv32a60x"
  required_extensions: ["rv32i", "rv32m", "rv32c"]

# Integration Points (HELPFUL TO SEND)
test_framework_integration:
  result_collection: "signature_comparison"
  timeout_handling: "10000_cycles"
  parallel_execution: "supported"
  regression_integration: "nightly_suite"
```

**Agent Decision Support:** This data ensures generated tests are properly configured for the target execution environment and test infrastructure.

### Source Locations
```
Primary: cva6/verif/tests/testlist_*.yaml (30+ files)
ISA Tests:
├── testlist_riscv-tests-cv32a60x-p.yaml (14.3KB) - User-mode ISA tests
├── testlist_riscv-arch-test-cv64a6_imafdc_sv39.yaml (38KB) - Architecture tests
├── testlist_riscv-compliance-cv32a60x.yaml (30KB) - Compliance tests
└── testlist_isacov.yaml (3KB) - ISA coverage tests
```

### Document Structure Example
```yaml
common_test_config: &common_test_config
  path_var: TESTS_PATH
  gcc_opts: "-static -mcmodel=medany -fvisibility=hidden"

testlist:
  - test: rv32ui-p-addi
    iterations: 1
    <<: *common_test_config
    asm_tests: <path_var>/riscv-tests/isa/rv32ui/addi.S
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "test_configuration"             # Fixed value for this document type  
file_path: "cva6/verif/tests/testlist_riscv-tests-cv32a60x-p.yaml"  # Full repository path
name: "riscv-tests-cv32a60x-p"                  # ROBUST EXTRACTION: Remove "testlist_" prefix: "testlist_riscv-tests-cv32a60x-p.yaml"→"riscv-tests-cv32a60x-p", "testlist_custom.yaml"→"custom", "testlist_isacov.yaml"→"isacov"
name_type: "testlist"                           # Fixed: "testlist" for test configuration files
target_architecture: "RV32"                     # MULTIPLE PATTERNS: Extract from core name: "cv32a60x"→"RV32", "cv64a6"→"RV64", OR from filename: "rv32"→"RV32", "rv64"→"RV64", OR default "RV32/RV64" if unclear
word_size: 32                                   # Derive from target_architecture: RV32→32, RV64→64, mixed→32
category: "configuration"                       # Fixed: "configuration" for all test configuration files
subcategory: "isa_tests"                        # COMPREHENSIVE MAPPING: "riscv-tests"→"isa_tests", "custom"→"directed_tests", "isacov"→"coverage_tests", "compliance"→"compliance_tests", "cvxif"→"interface_tests", "csr_embedded"→"system_tests", "interrupt"→"system_tests", "arch-test"→"architecture_tests"

# Content-Specific Fields
privilege_level: "machine"                      # Extract from suffix: "-p" → "machine", "-s" → "supervisor", "-u" → "user"
target_cores: ["cv32a60x"]                      # Extract from filename: "cv32a60x" → ["cv32a60x"], "cv64a6" → ["cv64a6"]
test_framework: "riscv_tests"                   # Extract from filename: "riscv-tests" → "riscv_tests", "custom" → "custom_framework"

# Document-Specific Fields
gcc_opts: "-static -mcmodel=medany -fvisibility=hidden -nostdlib -nostartfiles"  # Extract from YAML "gcc_opts:" field
include_paths: ["/riscv-tests/isa/macros/scalar/"]  # Extract from YAML include path specifications or path_var usage
test_references: [                              # Parse YAML testlist entries
  {"test": "rv32ui-p-addi", "iterations": 1, "asm_tests": "<path_var>/riscv-tests/isa/rv32ui/addi.S"},  # FROM: "- test: rv32ui-p-addi"
  {"test": "rv32ui-p-add", "iterations": 1, "asm_tests": "<path_var>/riscv-tests/isa/rv32ui/add.S"}    # FROM: "- test: rv32ui-p-add"  
]

# Examples of extraction patterns:
# FROM: "testlist_riscv-tests-cv32a60x-p.yaml"  →  name: "riscv-tests-cv32a60x-p", subcategory: "isa_tests", target_cores: ["cv32a60x"], privilege_level: "machine"
# FROM: "gcc_opts: \"-static -mcmodel=medany\""  →  gcc_opts: "-static -mcmodel=medany"
# FROM: "- test: rv32ui-p-addi\n  iterations: 1\n  asm_tests: <path_var>/riscv-tests/isa/rv32ui/addi.S"  →  test_references entry
# FROM: "cv32a60x" in filename  →  target_architecture: "RV32", word_size: 32
```

## Document Type 6: Configuration Data

### Purpose
**What:** CVA6 core configuration parameters defining enabled features, extensions, and system parameters
**Why:** Provides agents with knowledge of what features are actually enabled in the target core configuration  
**When Used:** When agents need to generate tests that match the specific CVA6 configuration being verified

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Core Configuration (MUST SEND)
enabled_features:
  base_isa: "rv32i"
  extensions: ["M", "A", "C", "Zicsr", "Zifencei"]  # only test enabled extensions
  custom_extensions: ["Xcorev"]  # CVA6-specific extensions
  privilege_levels: ["M", "U"]  # no supervisor mode in this config

# System Parameters (MUST SEND)
system_configuration:
  xlen: 32  # register width affects test data sizes
  physical_addr_size: 34  # memory addressing limits
  virtual_memory: false  # affects memory test patterns
  cache_configuration: {"icache": "16KB", "dcache": "16KB"}

# Hardware Constraints (SHOULD SEND)
implementation_limits:
  register_file_size: 32
  csr_implemented: ["mstatus", "mie", "mtvec", "mepc", "mcause"]  # only test implemented CSRs
  exception_support: ["illegal_instruction", "ecall", "ebreak"]
  interrupt_support: ["machine_timer", "machine_external"]

# Configuration Context (HELPFUL TO SEND)
core_variant: "cv32a60x"  # affects expected behavior
configuration_purpose: "embedded_application"  # vs high_performance, area_optimized
compliance_target: "riscv_2019_12_31"  # specification version
verification_scope: "instruction_accurate"  # vs cycle_accurate
```

**Agent Decision Support:** This data ensures agents only generate tests for actually implemented features and use correct system parameters.

### Source Locations
```
RISC-V Config: cva6/config/riscv-config/**/*.yaml
├── cv32a60x/spec/isa_spec.yaml - ISA configuration for CV32A60X
├── cv32a65x/spec/isa_spec.yaml - ISA configuration for CV32A65X  
└── cv64a6_imafdc_sv39/spec/isa_spec.yaml - CV64A6 ISA config
```

### Document Structure Example
```yaml
hart_id: 0
xlen: 32
physical_addr_sz: 34
supported_isa: ["rv32i", "rv32m", "rv32a", "rv32c", "rv32zicsr"]
privilege_modes: ["M", "U"]
custom_extensions: ["Xcorev"]
mtvec:
  rst_val: 0x00000000
  mode: ["vectored", "direct"]
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "configuration_data"             # Fixed value for this document type
file_path: "cva6/config/riscv-config/cv32a60x/spec/isa_spec.yaml"  # Full repository path  
name: "cv32a60x"                                # Extract from path directory: ".../cv32a60x/spec/..." → "cv32a60x"
name_type: "core_config"                        # Fixed: "core_config" for configuration data files
target_architecture: "RV32"                     # Extract from core name: "cv32a60x" → "RV32", "cv64a6" → "RV64"
word_size: 32                                   # Extract from YAML "xlen: 32" field
category: "configuration"                       # Fixed: "configuration" for all configuration data files  
subcategory: "isa_specification"                # Map from filename: "isa_spec.yaml" → "isa_specification", "debug_spec.yaml" → "debug_specification"

# Content-Specific Fields
target_cores: ["cv32a60x"]                      # Extract core name from path: ".../cv32a60x/..." → ["cv32a60x"]

# Document-Specific Fields  
physical_addr_sz: 34                            # Extract from YAML "physical_addr_sz: 34" field
supported_isa: ["rv32i", "rv32m", "rv32a", "rv32c", "rv32zicsr"]  # Extract from YAML "supported_isa:" array
custom_extensions: ["Xcorev"]                   # Extract from YAML "custom_extensions:" array  
privilege_levels: ["M", "U"]                    # Extract from YAML "privilege_modes: [\"M\", \"U\"]" field
mtvec_mode: ["vectored", "direct"]              # Extract from YAML "mtvec: mode: [\"vectored\", \"direct\"]"

# Examples of extraction patterns:
# FROM: "cva6/config/riscv-config/cv32a60x/spec/isa_spec.yaml"  →  name: "cv32a60x", target_architecture: "RV32"
# FROM: "xlen: 32"  →  word_size: 32
# FROM: "supported_isa: [\"rv32i\", \"rv32m\", \"rv32a\"]"  →  supported_isa: ["rv32i", "rv32m", "rv32a"]
# FROM: "privilege_modes: [\"M\", \"U\"]"  →  privilege_levels: ["M", "U"]
```

## Document Type 7: Interface Specifications

### Purpose  
**What:** Interface protocol specifications defining signal behavior, timing, and compliance requirements
**Why:** Provides agents with knowledge of interface constraints and protocol rules for generating interface tests
**When Used:** When agents need to generate interface compliance tests, protocol violation tests, or integration tests

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Protocol Requirements (MUST SEND)
interface_constraints:
  protocol_name: "AXI4"
  signal_behaviors: {"AWBURST": "always_INCR", "AWLEN": "0_or_1_only"}
  timing_requirements: {"setup_time": "1ns", "hold_time": "0.5ns"}
  mandatory_sequences: ["AWVALID_before_AWREADY", "WLAST_with_final_data"]

# Signal Definitions (MUST SEND)
signal_specifications:
  control_signals: ["AWVALID", "AWREADY", "WVALID", "WREADY", "BVALID", "BREADY"]
  data_signals: ["AWADDR[63:0]", "WDATA[63:0]", "BRESP[1:0]"]
  width_constraints: {"address": 64, "data": 64, "id": 4}
  encoding_rules: {"BRESP": {"OKAY": "2'b00", "EXOKAY": "2'b01"}}

# Compliance Rules (SHOULD SEND)
protocol_compliance:
  cvx6_specific_constraints: ["burst_type_always_INCR", "no_wrap_bursts"]
  violation_conditions: ["AWVALID_low_with_AWREADY_high", "missing_WLAST"]
  error_responses: {"decode_error": "SLVERR", "timeout": "SLVERR"}  
  performance_requirements: {"max_latency": "100_cycles", "min_throughput": "1GB_s"}

# Integration Context (HELPFUL TO SEND)
system_integration:
  connected_components: ["cache_controller", "memory_subsystem"]
  interface_direction: "master"  # CVA6 is AXI master
  multiplexing: "single_outstanding_transaction"
  error_handling_strategy: "exception_on_error_response"
```

**Agent Decision Support:** This data enables agents to generate tests that verify protocol compliance and detect interface violations.

### Source Locations  
```
CVA6 Interfaces: cva6/docs/**/*.rst, cva6/**/*.md
├── docs/01_cva6_user/AXI_Interface.rst - AXI interface documentation
├── docs/01_cva6_user/Core_Integration.rst - Integration interfaces  
└── corev_apu/instr_tracing/README.md - Instruction tracing interface
```

### Document Structure Example
```rst
AXI Interface
=============

The CVA6 core uses AXI4 interface for memory transactions.

Signal Descriptions
------------------

* AWBURST[1:0]: Write burst type (always INCR = 2'b01)
* ARBURST[1:0]: Read burst type (always INCR = 2'b01)
* AWLEN[7:0]: Write burst length (0 or 1 for CVA6)
* ARLEN[7:0]: Read burst length (0 or 1 for CVA6)
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema) 
document_type: "interface_specification"        # Fixed value for this document type
file_path: "cva6/docs/01_cva6_user/AXI_Interface.rst"  # Full repository path
name: "AXI4"                                    # Extract from document title: "AXI Interface" → "AXI4", or detect from text "AXI4 interface"
name_type: "interface"                          # Fixed: "interface" for interface specification files
target_architecture: "RV32/RV64"                # Extract from document scope: usually "RV32/RV64" for system interfaces  
word_size: 64                                   # Extract from interface width: "64-bit AXI interface" → 64
category: "specification"                       # Fixed: "specification" for all interface specification files
subcategory: "memory_bus"                       # Map from interface type: AXI → "memory_bus", RVFI → "trace", CVXIF → "coprocessor"

# Content-Specific Fields
source_type: "project_internal"                 # Fixed: "project_internal" for CVA6 interface specifications

# Document-Specific Fields
protocol_version: "4.0"                         # Extract from text: "AXI4 interface" → "4.0"
interface_type: "memory_bus"                    # Infer from context: AXI → "memory_bus", debug → "debug_interface"  
direction: "master"                              # Extract from text: "CVA6 core uses AXI4 interface" → "master" (CVA6 is master)
key_signals: ["AWVALID", "AWREADY", "AWBURST", "ARBURST", "AWLEN", "ARLEN"]  # Extract from "Signal Descriptions" sections
signal_constraints: {                           # Extract from signal description text
  "AWBURST": "always INCR = 2'b01",            # FROM: "AWBURST[1:0]: Write burst type (always INCR = 2'b01)"
  "ARBURST": "always INCR = 2'b01",            # FROM: "ARBURST[1:0]: Read burst type (always INCR = 2'b01)"
  "AWLEN": "0 or 1 for CVA6",                  # FROM: "AWLEN[7:0]: Write burst length (0 or 1 for CVA6)"
  "ARLEN": "0 or 1 for CVA6"                   # FROM: "ARLEN[7:0]: Read burst length (0 or 1 for CVA6)"
}
timing_constraints: ["setup", "hold"]           # Extract from timing requirement sections

# Examples of extraction patterns:
# FROM: "AXI Interface" title + "AXI4" in text  →  name: "AXI4"  
# FROM: "AWBURST[1:0]: Write burst type (always INCR = 2'b01)"  →  signal_constraints: {"AWBURST": "always INCR = 2'b01"}
# FROM: "The CVA6 core uses AXI4 interface"  →  direction: "master"
# FROM: "cva6/docs/01_cva6_user/AXI_Interface.rst"  →  subcategory: "memory_bus" (AXI maps to memory_bus)
```

## Document Type 8: RISC-V Test Suite (riscv-tests)

### Purpose
**What:** Reference implementation of RISC-V instruction tests providing proven test patterns and comprehensive coverage
**Why:** Provides agents with battle-tested test approaches, edge cases, and comprehensive instruction coverage patterns
**When Used:** When agents need proven test patterns, comprehensive edge case coverage, or reference implementations

### RAG Agent Data Extraction  
**What RAG agents should send to main test writing agents:**

```yaml
# Proven Test Patterns (MUST SEND)
test_methodology:
  instruction_coverage: ["basic_operation", "edge_cases", "register_combinations"]
  test_case_patterns: ["TEST_IMM_OP(2, addi, 0x00000000, 0x00000000, 0x000)"]
  validation_approach: "golden_result_comparison"
  edge_case_strategy: "boundary_values_plus_random"

# Comprehensive Test Cases (MUST SEND)  
coverage_examples:
  test_cases_count: 16  # for ADDI instruction
  edge_cases_covered: ["min_immediate", "max_immediate", "zero_operands", "register_hazards"]
  register_combinations: ["rs1_x0_special_case", "rd_equals_rs1_hazard", "all_register_coverage"]
  data_patterns: ["0x00000000", "0x7fffffff", "0x80000000", "0xffffffff"]

# Test Infrastructure Patterns (SHOULD SEND)
framework_integration:
  macro_library: ["TEST_IMM_OP", "TEST_RR_OP", "TEST_PASSFAIL", "RVTEST_CODE_BEGIN"]
  exception_handling: "standard_trap_handler"
  result_validation: "signature_based_checking"  
  test_environment: "baremetal_rv32ui"

# Quality Assurance (HELPFUL TO SEND) 
test_quality_metrics:
  coverage_completeness: "architectural_compliance_verified"
  cross_platform_validation: "multiple_implementation_tested"
  regression_stability: "stable_across_risc_v_versions"
  community_validation: "open_source_peer_reviewed"
```

**Agent Decision Support:** This data provides agents with proven, comprehensive test patterns and quality benchmarks for instruction testing.

### Source Locations
```
Primary: riscv-tests/isa/**/*.S, riscv-tests/benchmarks/**/*.c
ISA Tests:
├── isa/rv32ui/*.S - RV32 user-level integer tests (referenced by CVA6 testlists)
├── isa/rv64ui/*.S - RV64 user-level integer tests  
├── isa/rv32mi/*.S - RV32 machine-level integer tests
├── isa/rv64mi/*.S - RV64 machine-level integer tests
├── isa/rv32si/*.S - RV32 supervisor-level integer tests
├── isa/macros/ - Test macro definitions
└── env/ - Test environment headers

Benchmarks:
├── benchmarks/dhrystone/ - Dhrystone benchmark
├── benchmarks/median/ - Median filter benchmark
└── benchmarks/multiply/ - Multiplication benchmark
```

### Document Structure Example
```assembly
# See LICENSE for license details.

#*****************************************************************************
# addi.S
#-----------------------------------------------------------------------------
# Test addi instruction.

#include "riscv_test.h"
#include "test_macros.h"

RVTEST_RV64U
RVTEST_CODE_BEGIN

  #-------------------------------------------------------------
  # Arithmetic tests
  #-------------------------------------------------------------

  TEST_IMM_OP( 2,  addi, 0x00000000, 0x00000000, 0x000 );
  TEST_IMM_OP( 3,  addi, 0x00000002, 0x00000001, 0x001 );
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "riscv_test_suite"               # Fixed value for this document type
file_path: "riscv-tests/isa/rv32ui/addi.S"     # Full repository path
name: "addi"                                    # Extract from filename without extension: "addi.S" → "addi"
name_type: "instruction_test"                  # Fixed: "instruction_test" for riscv-tests files
target_architecture: "RV32"                    # Extract from path: "rv32ui/" → "RV32", "rv64ui/" → "RV64"
word_size: 32                                  # Derive from architecture: RV32 → 32, RV64 → 64
category: "reference_test"                     # Fixed: "reference_test" for all riscv-tests files
subcategory: "instruction_unit"                # Map from path: "ui/" → "instruction_unit", "mi/" → "machine_level", "si/" → "supervisor_level"

# Content-Specific Fields
privilege_level: "user"                        # Extract from path: "ui" → "user", "mi" → "machine", "si" → "supervisor"
content_type: "reference"                      # Fixed: "reference" for official riscv-tests
source_type: "official"                        # Fixed: "official" for riscv-tests suite
test_framework: "riscv_test_macros"            # Extract from includes: "#include \"riscv_test.h\"" → "riscv_test_macros"

# Document-Specific Fields
test_vm: "rv32ui"                              # Extract from directory path: "rv32ui/" → "rv32ui"
instruction_under_test: "ADDI"                 # Extract from filename: "addi" → "ADDI", or from comments "# Test addi instruction"
test_cases_count: 16                           # Count macro invocations: count "TEST_IMM_OP(" occurrences
macro_types_used: ["TEST_IMM_OP", "TEST_IMM_SRC1_EQ_DEST", "TEST_IMM_DEST_BYPASS"]  # Extract unique macro names from code
test_pattern_categories: ["arithmetic_tests", "bypassing_tests", "source_destination_tests"]  # Extract from comment sections

# Examples of extraction patterns:
# FROM: "riscv-tests/isa/rv32ui/addi.S"  →  target_architecture: "RV32", privilege_level: "user", name: "addi"
# FROM: "#include \"riscv_test.h\""  →  test_framework: "riscv_test_macros"
# FROM: "TEST_IMM_OP( 2, addi, 0x00000000, 0x00000000, 0x000 );"  →  macro_types_used includes "TEST_IMM_OP"
# FROM: "# Test addi instruction."  →  instruction_under_test: "ADDI"
```

## Document Type 9: CVA6 Instruction Specifications

### Purpose
**What:** CVA6-specific instruction definitions with implementation details and exceptions
**Why:** Provides agents with CVA6-specific instruction behavior that may differ from generic RISC-V specification  
**When Used:** When agents need CVA6-specific instruction behavior, implementation constraints, or exception handling

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# CVA6-Specific Behavior (MUST SEND)
implementation_specifics:
  instruction_format: "addi rd, rs1, imm[11:0]"
  operation_description: "add sign-extended 12-bit immediate to register rs1"  
  pseudocode: "x[rd] = x[rs1] + sext(imm[11:0])"
  cva6_specific_constraints: "no_overflow_exception"  # vs other implementations

# Exception Behavior (MUST SEND)
exception_handling:
  normal_exceptions: "NONE"  # or specific exceptions this instruction can raise
  invalid_conditions: "NONE"  # or conditions that make instruction invalid
  privilege_requirements: "any_level"  # or specific privilege level needed
  side_effects: "updates_rd_only"  # what else the instruction affects

# Implementation Details (SHOULD SEND)  
cva6_implementation_notes:
  execution_timing: "single_cycle"  # vs multi_cycle, pipelined
  resource_usage: ["alu_port", "register_file_write"]
  hazard_considerations: ["rd_rs1_bypass_required"]
  performance_characteristics: "combinatorial_alu_operation"

# Cross-Reference Context (HELPFUL TO SEND)
specification_context:
  differs_from_risc_v_spec: false  # or list differences
  related_instructions: ["ADD", "SUB", "other_immediate_ops"]
  verification_priority: "high"  # based on usage frequency
  test_complexity: "low"  # implementation complexity affects test strategy
```

**Agent Decision Support:** This data ensures agents generate tests that match CVA6's specific implementation rather than generic RISC-V behavior.

### Source Locations
```
Primary instruction specifications:
├── cva6/verif/docs/VerifPlans/ISA_RV32/RISCV_Instructions.rst - Verification-focused instruction definitions
├── cva6/docs/01_cva6_user/RISCV_Instructions*.rst - User manual instruction documentation
│   ├── RISCV_Instructions_RV32I.rst - Base integer instructions  
│   ├── RISCV_Instructions_RV32A.rst - Atomic instructions
│   ├── RISCV_Instructions_RV32M.rst - Multiply/divide instructions
│   ├── RISCV_Instructions_RV32C.rst - Compressed instructions
│   ├── RISCV_Instructions_RVZicsr.rst - CSR instructions
│   ├── RISCV_Instructions_RVZifencei.rst - Instruction fence
│   └── RISCV_Instructions_RVZ*.rst - Various Z extensions (Zba, Zbb, Zbc, Zbs, etc.)

Architecture documentation:
├── cva6/docs/design/design-manual/source/architecture.adoc - Overall CVA6 architecture
├── cva6/docs/design/design-manual/source/instructions.adoc - Instruction implementation overview
└── cva6/config/gen_from_riscv_config/cv32a65x/isa/isa.adoc - ISA configuration specifics

Configuration templates:
├── cva6/config/gen_from_riscv_config/templates/isa_template.yaml - Instruction template definitions
└── cva6/config/gen_from_riscv_config/README.md - Configuration and instruction generation details
```

### Document Structure Example
```rst
- **ADDI**: Add Immediate

    **Format**: addi rd, rs1, imm[11:0]

    **Description**: add sign-extended 12-bit immediate to register rs1, and store the result in register rd.

    **Pseudocode**: x[rd] = x[rs1] + sext(imm[11:0])

    **Invalid values**: NONE

    **Exception raised**: NONE
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "cva6_instruction_specification" # Fixed value for this document type
file_path: "cva6/verif/docs/VerifPlans/ISA_RV32/RISCV_Instructions.rst"  # Full repository path
name: "ADDI"                                    # ROBUST EXTRACTION: From instruction definition: "- **ADDI**: Add Immediate"→"ADDI", OR from file name: "RISCV_Instructions_RV32I.rst"→"RV32I", "RISCV_Instructions_RVZicsr.rst"→"RVZicsr"
name_type: "instruction"                       # Fixed: "instruction" for instruction specification files  
target_architecture: "RV32"                    # MULTIPLE PATTERNS: From file path: "ISA_RV32/"→"RV32", OR from filename: "RV32I"→"RV32", "RV64I"→"RV64", OR from applicability table: "CV32A6*"→"RV32", "CV64A6*"→"RV64"
word_size: 32                                  # Derive from target_architecture: RV32→32, RV64→64  
category: "specification"                      # Fixed: "specification" for all instruction specification files
subcategory: "cva6_internal"                   # Fixed: "cva6_internal" for CVA6 project specifications

# Content-Specific Fields  
feature: "Integer Register-Immediate"           # ROBUST EXTRACTION: From section headers: "Integer Register-Immediate Instructions"→"Integer Register-Immediate", OR infer from filename: "RV32I"→"Base Integer Instructions", "RV32M"→"Multiplication and Division", "RVZicsr"→"Control and Status Register Instructions"
content_type: "specification"                  # Fixed: "specification" for instruction definitions
source_type: "project_internal"                # Fixed: "project_internal" for CVA6 specifications

# Document-Specific Fields
instruction_category: "Integer Register-Immediate"  # Extract from section header: "Integer Register-Immediate Instructions"
format: "addi rd, rs1, imm[11:0]"              # Extract from "**Format**: addi rd, rs1, imm[11:0]" line
description: "add sign-extended 12-bit immediate to register rs1, and store the result in register rd"  # Extract from "**Description**:" line  
pseudocode: "x[rd] = x[rs1] + sext(imm[11:0])" # Extract from "**Pseudocode**: x[rd] = x[rs1] + sext(imm[11:0])" line
invalid_values: "NONE"                          # Extract from "**Invalid values**: NONE" line
exceptions: "NONE"                              # Extract from "**Exception raised**: NONE" line
operands: {                                     # Parse format string to identify operand types
  "rd": {"type": "destination_register", "description": "destination register"},         # FROM: format "addi rd, ..."
  "rs1": {"type": "source_register", "description": "source register"},                  # FROM: format "addi rd, rs1, ..."
  "imm": {"type": "immediate", "width": 12, "encoding": "sign_extended"}                 # FROM: format "..., imm[11:0]"
}

# Examples of extraction patterns:
# FROM: "- **ADDI**: Add Immediate"  →  name: "ADDI"
# FROM: "**Format**: addi rd, rs1, imm[11:0]"  →  format: "addi rd, rs1, imm[11:0]"
# FROM: "**Pseudocode**: x[rd] = x[rs1] + sext(imm[11:0])"  →  pseudocode: "x[rd] = x[rs1] + sext(imm[11:0])"
# FROM: "cva6/verif/docs/VerifPlans/ISA_RV32/RISCV_Instructions.rst"  →  target_architecture: "RV32"
```

## Document Type 10: SystemVerilog Package Definitions

### Purpose
**What:** SystemVerilog package definitions containing instruction encodings, opcodes, and hardware constants  
**Why:** Provides agents with exact bit-level encodings and hardware constants needed for accurate test generation
**When Used:** When agents need precise instruction encodings, CSR addresses, exception codes, or hardware constants

### RAG Agent Data Extraction  
**What RAG agents should send to main test writing agents:**

```yaml
# Instruction Encoding Constants (MUST SEND)
encoding_definitions:
  opcode_values: {"OpcodeOpImm": "7'b0010011", "OpcodeOp": "7'b0110011"}  
  instruction_formats: {"itype_t": {"imm": "[31:20]", "rs1": "[19:15]", "rd": "[11:7]"}}
  funct_codes: {"ADDI": {"funct3": "3'b000", "opcode": "OpcodeOpImm"}}
  bit_patterns: "32'b_iiiiiiiiiiii_sssss_000_ddddd_0010011"  # for ADDI

# Hardware Constants (MUST SEND)  
system_constants:
  csr_addresses: {"CSR_MSTATUS": "12'h300", "CSR_MARCHID": "12'hF12"}
  exception_codes: {"ILLEGAL_INSTR": "2", "ECALL_M_MODE": "11"}
  register_definitions: {"XLEN": 32, "REG_ADDR_SIZE": 5}
  privilege_encodings: {"PRIV_LVL_M": "2'b11", "PRIV_LVL_U": "2'b00"}

# Type Definitions (SHOULD SEND)
data_structures:
  instruction_types: ["rtype_t", "itype_t", "stype_t", "utype_t"]
  csr_structure_definitions: "mstatus_rv_t with specific bit fields"
  exception_structure: "cause codes and priority encoding"
  interface_types: "axi request/response structures"  

# Implementation Constants (HELPFUL TO SEND)
configuration_parameters:
  default_values: {"WIDTH": 32, "NR_COMMIT_PORTS": 1}
  compile_time_options: "parameterizable vs fixed configurations"
  package_dependencies: ["cva6_config_pkg", "ariane_pkg"]  
  version_specific_constants: "implementation version differences"
```

**Agent Decision Support:** This data provides agents with exact bit-level specifications needed for generating hardware-accurate tests and proper instruction encoding.

### Source Locations
```
Primary: cva6/core/include/*.sv
├── riscv_pkg.sv - RISC-V instruction opcodes and encodings
├── ariane_pkg.sv - CVA6-specific types, constants, and parameters
├── instr_tracer_pkg.sv - Instruction tracing definitions
├── config_pkg.sv - Configuration package definitions
├── cv32a60x_config_pkg.sv - CV32A60X specific configuration
├── cv32a65x_config_pkg.sv - CV32A65X specific configuration  
├── cv64a60ax_config_pkg.sv - CV64A60AX specific configuration
├── build_config_pkg.sv - Build configuration definitions
└── aes_pkg.sv - AES extension definitions
```

### Document Structure Example
```systemverilog
package riscv_pkg;
  // RISC-V Opcodes
  typedef enum logic [6:0] {
    OpcodeOpImm     = 7'b0010011,  // Register-Immediate
    OpcodeOp        = 7'b0110011,  // Register-Register  
    OpcodeLoad      = 7'b0000011,  // Load instructions
    OpcodeStore     = 7'b0100011,  // Store instructions
    OpcodeBranch    = 7'b1100011,  // Branch instructions
    OpcodeJal       = 7'b1101111,  // JAL instruction
    OpcodeJalr      = 7'b1100111   // JALR instruction
  } opcode_e;
  
  // Instruction formats
  typedef struct packed {
    logic [11:0] imm;
    logic [4:0]  rs1; 
    logic [2:0]  funct3;
    logic [4:0]  rd;
    logic [6:0]  opcode;
  } itype_t;
endpackage
```

### Metadata Structure
```yaml
document_type: "systemverilog_package"
extraction_metadata:
  file_path: "cva6/core/include/riscv_pkg.sv"
  file_size_kb: 8
  last_modified: "2024-05-25"
  
architecture_metadata:
  target_architecture: "RV32/RV64"               # Package supports both
  package_scope: "risc_v_definitions"           # Extract from package name
  design_level: "implementation"                # This is implementation-specific
  
package_metadata:
  package_name: "riscv_pkg"                     # Extract from "package riscv_pkg"
  package_category: "instruction_definitions"   # Infer from content
  import_dependencies: []                       # Extract from import statements
  
instruction_encoding_metadata:
  opcode_definitions:
    - name: "OpcodeOpImm"                       # Extract from typedef enum
      value: "7'b0010011"                       # Extract binary value
      decimal_value: 19                         # Convert for reference
      description: "Register-Immediate instructions" # Extract from comments
      instructions: ["ADDI", "SLTI", "SLTIU", "XORI", "ORI", "ANDI"]
    - name: "OpcodeOp"
      value: "7'b0110011"
      decimal_value: 51
      description: "Register-Register instructions"
      instructions: ["ADD", "SUB", "SLL", "SLT", "SLTU", "XOR", "SRL", "SRA", "OR", "AND"]
      
  instruction_formats:
    - format_name: "itype_t"                   # Extract from typedef struct
      fields: 
        - name: "imm"                          # Extract field definitions
          width: 12
          position: "[31:20]"
        - name: "rs1"
          width: 5 
          position: "[19:15]"
        - name: "funct3"
          width: 3
          position: "[14:12]"
        - name: "rd" 
          width: 5
          position: "[11:7]"
        - name: "opcode"
          width: 7
          position: "[6:0]"
          
  constant_definitions:
    - name: "XLEN"                             # Extract parameter definitions
      value: 32
      description: "Register width"
    - name: "NR_COMMIT_PORTS"
      value: 2
      description: "Number of commit ports"
      
semantic_metadata:
  provides_instruction_encoding: true          # Critical for test generation
  provides_opcode_values: true                # Enables bit-accurate tests
  cva6_specific_constants: true               # Implementation-specific values
  supports_agents: ["ISA_Test_Writer", "Interface_Test_Writer"] # Both need encodings
  
cross_reference_metadata:
  links_to_instruction_specs: "Document Type 9" # Cross-ref to instruction behavior
  used_by_design_modules: ["decoder.sv", "compressed_decoder.sv"] # Design usage
  referenced_by_verification: "Verification plans use these opcodes"
```
## Document Type 11: CSR Specifications (YAML)

### Purpose
**What:** Detailed Control and Status Register (CSR) field definitions with exact bit layouts, access permissions, and reset values  
**Why:** Provides agents with precise hardware implementation details needed for generating accurate CSR access tests
**When Used:** When agents need exact CSR field specifications, bit positions, access types, and reset behavior for test generation

### RAG Agent Data Extraction
**What RAG agents should send to main test writing agents:**

```yaml
# Precise CSR Definition (MUST SEND)
csr_specification:
  csr_name: "CSR_MSTATUS"
  csr_address: 0x300
  privilege_mode: "M"  # Machine, Supervisor, User
  description: "Machine Status Register"

# Field-Level Details (MUST SEND)
field_definitions:
  - field_name: "SD"
    bit_range: [31, 31]  # msb, lsb
    access_type: "R"     # R, RW, RO, WO
    reset_value: 0
    description: "SD bit indicates floating-point state"
  - field_name: "TSR"  
    bit_range: [22, 22]
    access_type: "RW"
    reset_value: 0
    description: "Trap SRET: supports intercepting supervisor exception return"

# Access Patterns (SHOULD SEND)
csr_behavior:
  read_side_effects: "none"  # or specific side effects
  write_constraints: ["tsr_requires_m_mode", "some_fields_readonly"]
  reset_behavior: "all_fields_to_specified_values"
  exception_conditions: ["illegal_access_from_u_mode"]

# Integration Context (HELPFUL TO SEND)
usage_context:
  related_csrs: ["MTVEC", "MCAUSE", "MEPC"]  # CSRs used together
  verification_priority: "high"  # based on complexity
  test_complexity: "medium"      # field interdependencies
```

**Agent Decision Support:** This data enables agents to generate CSR tests with exact bit manipulations, correct access patterns, and proper exception handling.

### Source Locations
```
Primary CSR specifications:
cva6/verif/tests/custom/CSR/csr_access_yaml/*.yaml
├── cv32a6_m_ro_csr_test.yaml - Machine-mode read-only CSR definitions
├── cv32a6_m_rw_csr_test.yaml - Machine-mode read-write CSR definitions  
├── cv32a6_s_rw_csr_test.yaml - Supervisor-mode read-write CSR definitions
└── cva6_mscratch_csr_access.yaml - Specific CSR (MSCRATCH) access patterns

Secondary verification configurations:
├── cva6/verif/sim/cva6_base_testlist.yaml - Base test configurations
├── cva6/verif/sim/cva6.yaml - Simulator tool configurations  
└── cva6/verif/env/corev-dv/simulator.yaml - Environment setup
```

### Document Structure Example
```yaml
- csr: CSR_MSTATUS
  description: >
    Machine Status Register
  address: 0x300
  privilege_mode: M
  rv32:
    - field_name: SD
      description: >
       SD bit indicates whether floating-point state is dirty
      type: R
      reset_val: 0
      msb: 31
      lsb: 31
    - field_name: TSR
      description: >
       Trap SRET: supports intercepting supervisor exception return instruction
      type: RW
      reset_val: 0
      msb: 22
      lsb: 22
```

### Metadata Structure
```yaml
# Universal Fields (harmonized schema)
document_type: "csr_specification"               # Fixed value for this document type
file_path: "cva6/verif/tests/custom/CSR/csr_access_yaml/cv32a6_m_rw_csr_test.yaml"  # Full repository path
name: "CSR_MSTATUS"                              # Extract from CSR entry: "- csr: CSR_MSTATUS" → "CSR_MSTATUS"
name_type: "csr"                                 # Fixed: "csr" for CSR specification files
target_architecture: "RV32"                     # Extract from section: "rv32:" → "RV32", "rv64:" → "RV64"  
word_size: 32                                    # Derive from target_architecture: RV32 → 32, RV64 → 64
category: "specification"                        # Fixed: "specification" for CSR definition files
subcategory: "csr_definition"                    # Fixed: "csr_definition" for CSR field specifications

# Content-Specific Fields
privilege_level: "machine"                       # Extract from "privilege_mode: M" → "machine", "S" → "supervisor", "U" → "user"
content_type: "specification"                    # Fixed: "specification" for authoritative CSR definitions
source_type: "project_internal"                  # Fixed: "project_internal" for CVA6 CSR specifications

# Document-Specific Fields
csr_address: 0x300                               # Extract from "address: 0x300" → 0x300
field_definitions: [                             # Extract from rv32/rv64 section field definitions  
  {"name": "SD", "bits": [31,31], "type": "R", "reset": 0},     # FROM: field_name: SD, msb: 31, lsb: 31, type: R, reset_val: 0
  {"name": "TSR", "bits": [22,22], "type": "RW", "reset": 0}    # FROM: field_name: TSR, msb: 22, lsb: 22, type: RW, reset_val: 0
]
access_constraints: ["machine_mode_only"]        # Infer from privilege_mode and field types
reset_behavior: "defined_values"                 # Extract from presence of reset_val fields

# Examples of extraction patterns:
# FROM: "- csr: CSR_MSTATUS"  →  name: "CSR_MSTATUS"
# FROM: "privilege_mode: M"  →  privilege_level: "machine"  
# FROM: "address: 0x300"  →  csr_address: 0x300
# FROM: "field_name: TSR\n  type: RW\n  reset_val: 0\n  msb: 22\n  lsb: 22"  →  field_definitions entry
# FROM: "cv32a6_m_rw_csr_test.yaml"  →  target_architecture: "RV32" (cv32 prefix)
```
