# CVA6 RAG Document Types & Metadata Structure

## Document Type 1: Verification Plans

### Source Locations
```
Primary: cva6/verif/docs/VerifPlans/source/*.md
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
document_type: "verification_plan"
extraction_metadata:
  file_path: "cva6/verif/docs/VerifPlans/source/dvplan_ISA_RV32.md"
  file_size_kb: 296
  last_modified: "2024-05-25"
  line_range: [5, 32]
  
architecture_metadata:
  target_architecture: "RV32"                     # Extract from filename "ISA_RV32"
  word_size: 32                                   # Extract from RV32 vs RV64
  applicable_cores: ["CV32A6_v0.1.0", "CV32A6-step2", "CV64A6-step3"]
  
content_metadata:
  verification_tag: "VP_ISA_RV32_F000_S000_I000"
  module: "ISA_RV32"
  feature: "Register-Immediate Instructions"
  sub_feature: "ADDI"
  item_id: "000"
  
verification_metadata:
  verification_goals: ["All possible rs1 registers are used"]
  pass_fail_criteria: "Check RM"
  test_type: "Constrained Random"
  coverage_method: "Functional Coverage"
  coverage_links: ["isacov.rv32i_addi_cg.cp_rs1"]
  requirement_location: "./RISCV_Instructions.rst"
```

## Document Type 2: Design Implementation

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
document_type: "design_implementation"
extraction_metadata:
  file_path: "cva6/core/alu.sv"
  file_size_kb: 16
  last_modified: "2024-05-25"
  line_range: [18, 150]
  
architecture_metadata:
  target_architecture: "RV32/RV64"               # Extract from WIDTH parameter
  data_width: 32                                 # Extract from WIDTH parameter
  configurable_width: true                      # Detect from parameter usage
  
module_metadata:
  module_name: "alu"
  category: "execution_unit"
  hierarchy_level: "core"
  dependencies: ["ariane_pkg"]
  
interface_metadata:
  input_ports: 
    - name: "operand_a_i"
      width: 32
      type: "logic"
  output_ports:
    - name: "result_o"
      width: 32
      type: "logic"
  parameters:
    - name: "WIDTH"
      default_value: 32
      type: "int unsigned"
```

## Document Type 3: ISA Specifications

### Source Locations
```
Primary: riscv-isa-manual/src/*.adoc
Unprivileged:
├── unpriv/rv32.adoc - RV32 base instruction set
├── unpriv/rv64.adoc - RV64 base instruction set  
├── unpriv/m.adoc - Multiply/divide extension
├── unpriv/a.adoc - Atomic extension
└── unpriv/zicsr.adoc - CSR instructions
```

### Document Structure Example
```asciidoc
==== ADDI

ADDI adds the sign-extended 12-bit immediate to register rs1. 
Arithmetic overflow is ignored and the result is simply the low XLEN bits.

[wavedrom, ,svg]
....
{reg: [
  {bits:  7, name: 'opcode', attr: '0010011'},
  {bits:  5, name: 'rd'},
  {bits:  3, name: 'funct3', attr: '000'},
  {bits:  5, name: 'rs1'},
  {bits: 12, name: 'imm[11:0]'}
]}
....
```

### Metadata Structure
```yaml
document_type: "isa_specification"
extraction_metadata:
  file_path: "riscv-isa-manual/src/unpriv/rv32.adoc"
  file_size_kb: 45
  last_modified: "2024-05-26"
  section_range: ["2.4", "2.4.1"]
  
architecture_metadata:
  target_architecture: "RV32"                    # Extract from filename rv32.adoc
  word_size: 32                                  # Extract from RV32 vs RV64
  extension_category: "base_integer"             # Extract from file context
  
instruction_metadata:
  instruction_name: "ADDI"
  extension: "RV32I"
  instruction_format: "I-type"
  opcode: "0010011"
  funct3: "000"
  
operation_metadata:
  operation_description: "ADDI adds the sign-extended 12-bit immediate to register rs1"
  pseudocode: "x[rd] = x[rs1] + sext(imm)"
  exceptions: []
  constraints: ["Arithmetic overflow is ignored"]
```

## Document Type 4: Test Examples

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
document_type: "test_example"
extraction_metadata:
  file_path: "cva6/verif/tests/custom/common/cva6_csr_access_test_32.S"
  file_size_kb: 3
  last_modified: "2024-05-25"
  language: "assembly"
  
architecture_metadata:
  target_architecture: "RV32"                   # Extract from filename "_32" suffix
  word_size: 32                                 # Extract from filename pattern
  
test_metadata:
  test_name: "cva6_csr_access_test_32"
  test_type: "directed" 
  target_feature: "csr_access"
  referenced_by_testlists: ["testlist_custom.yaml"]   # Cross-reference
```

## Document Type 5: Test Configurations (Testlists)

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
document_type: "test_configuration"
extraction_metadata:
  file_path: "cva6/verif/tests/testlist_riscv-tests-cv32a60x-p.yaml"
  file_size_kb: 14.3
  last_modified: "2024-05-25"
  
architecture_metadata:
  target_architecture: "RV32"                   # Extract from "cv32a60x" in filename
  word_size: 32                                 # Extract from cv32 prefix
  target_core: "cv32a60x"                      # Extract from filename
  privilege_mode: "machine"                    # Extract from "-p" suffix
  
testlist_metadata:
  testlist_name: "riscv-tests-cv32a60x-p"
  test_category: "isa_tests"
  test_count: 45
  
configuration_metadata:
  gcc_opts: "-static -mcmodel=medany"
  include_paths: ["/riscv-tests/isa/macros/scalar/"]
  link_options: ["-nostdlib", "-nostartfiles"]
  
test_references:
  referenced_tests: 
    - test_file: "riscv-tests/isa/rv32ui/addi.S"
      test_name: "rv32ui-p-addi"
      iterations: 1
```

## Document Type 6: Configuration Data

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
document_type: "configuration_data"
extraction_metadata:
  file_path: "cva6/config/riscv-config/cv32a60x/spec/isa_spec.yaml"
  file_size_kb: 5
  last_modified: "2024-05-25"
  
architecture_metadata:
  target_architecture: "RV32"                   # Extract from cv32 prefix
  word_size: 32                                 # Extract from xlen field
  core_name: "cv32a60x"                         # Extract from path
  physical_addr_sz: 34                          # Extract directly
  
isa_metadata:
  base_isa: "rv32i"
  extensions: ["M", "A", "C", "Zicsr"]
  custom_extensions: ["Xcorev"]
  privilege_modes: ["M", "U"]
  
memory_metadata:
  virtual_memory: false
  address_translation: null
```

## Document Type 7: Interface Specifications

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
document_type: "interface_specification"
extraction_metadata:
  file_path: "cva6/docs/01_cva6_user/AXI_Interface.rst"
  file_size_kb: 23
  last_modified: "2024-05-25"
  
architecture_metadata:
  target_architecture: "RV32/RV64"              # Applicable to both
  interface_width: 64                           # Extract from spec
  
interface_metadata:
  interface_name: "AXI4"
  protocol_version: "4.0"
  interface_type: "memory_bus"
  direction: "master"
  
signal_metadata:
  signal_groups: ["address", "data", "control"]
  key_signals: ["AWVALID", "AWREADY", "AWBURST"]
  timing_constraints: ["setup", "hold"]
```

## Document Type 8: RISC-V Test Suite (riscv-tests)

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
document_type: "riscv_test_suite"
extraction_metadata:
  file_path: "riscv-tests/isa/rv32ui/addi.S"
  file_size_kb: 2
  last_modified: "2024-05-27"
  
architecture_metadata:
  target_architecture: "RV32"                   # Extract from rv32ui path
  word_size: 32                                 # Extract from rv32 vs rv64
  privilege_level: "user"                      # Extract from "ui" in rv32ui
  test_vm: "rv32ui"                            # Extract from directory name
  
test_metadata:
  test_name: "addi"                            # Extract from filename
  instruction_under_test: "ADDI"               # Extract from filename/comments
  test_type: "instruction_unit_test"          # Infer from structure
  test_framework: "riscv_test_macros"         # Extract from includes
  
test_content_metadata:
  test_cases_count: 16                         # Count TEST_IMM_OP calls
  test_categories: ["arithmetic", "bypassing", "src_dest"] # Extract from comments
  edge_cases_tested: ["overflow", "underflow", "zero"] # Infer from test values
  macro_types_used: ["TEST_IMM_OP", "TEST_IMM_SRC1_EQ_DEST"] # Extract from code
  
cross_reference_metadata:
  referenced_by_testlists: 
    - "testlist_riscv-tests-cv32a60x-p.yaml"  # Cross-reference with CVA6 testlists
    - "testlist_riscv-tests-cv64a6_imafdc_sv39.yaml"
  cva6_test_name: "rv32ui-p-addi"             # How CVA6 references this test
  shared_macros: ["riscv_test.h", "test_macros.h"] # Shared infrastructure
```