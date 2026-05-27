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

## Document Type 3: ISA Specifications (RISC-V Official Manual)

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
document_type: "isa_specification"
extraction_metadata:
  file_path: "riscv-isa-manual/src/unpriv/rv32.adoc"
  file_size_kb: 15
  last_modified: "2024-05-26"
  chapter: "RV32I Base Integer Instruction Set"
  
architecture_metadata:
  target_architecture: "RV32"                    # Extract from RV32I context
  word_size: 32                                  # Extract from "32 bits wide, XLEN=32"
  extension_category: "base_integer"             # Extract from chapter title
  instruction_count: 40                          # Extract from "contains 40 unique instructions"
  
specification_metadata:
  specification_type: "official_risc_v_manual"   # This is the authoritative source
  register_model: "32_general_purpose_registers" # Extract from programmers model
  register_width: 32                             # Extract from XLEN=32
  special_registers: ["x0_hardwired_zero", "pc"] # Extract from register descriptions
  
instruction_categories:
  - category: "integer_register_immediate"       # Extract from section structure
    instructions: ["ADDI", "SLTI", "SLTIU", "XORI", "ORI", "ANDI"]
  - category: "integer_register_register"
    instructions: ["ADD", "SUB", "SLL", "SLT", "SLTU", "XOR", "SRL", "SRA", "OR", "AND"]
  - category: "load_store"
    instructions: ["LB", "LH", "LW", "LBU", "LHU", "SB", "SH", "SW"]
  - category: "control_flow"  
    instructions: ["BEQ", "BNE", "BLT", "BGE", "BLTU", "BGEU", "JAL", "JALR"]
    
normative_requirements:
  - requirement_id: "norm:rv32i_xreg_sz"
    text: "For RV32I, the 32 x registers are each 32 bits wide"
  - requirement_id: "norm:x0eq0"  
    text: "Register x0 is hardwired with all bits equal to 0"
  - requirement_id: "norm:pcreg_op"
    text: "Program counter pc holds the address of the current instruction"
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

## Document Type 9: CVA6 Instruction Specifications

### Source Locations
```
Primary: cva6/verif/docs/VerifPlans/ISA_RV32/RISCV_Instructions.rst
├── cva6/docs/01_cva6_user/RISCV_Instructions_RV32I.rst
├── cva6/config/gen_from_riscv_config/templates/isa_template.yaml
└── cva6/config/gen_from_riscv_config/README.md (instruction details)
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
document_type: "cva6_instruction_specification"
extraction_metadata:
  file_path: "cva6/verif/docs/VerifPlans/ISA_RV32/RISCV_Instructions.rst"
  file_size_kb: 25
  last_modified: "2024-05-25"
  
architecture_metadata:
  target_architecture: "RV32"                   # Extract from RV32I context
  word_size: 32                                 # Extract from file context
  specification_source: "CVA6_project"         # This is CVA6's instruction specification
  
instruction_metadata:
  instruction_name: "ADDI"                      # Extract from "**ADDI**: Add Immediate"
  instruction_category: "Integer Register-Immediate" # Extract from section header
  format: "addi rd, rs1, imm[11:0]"           # Extract from "**Format**:"
  description: "add sign-extended 12-bit immediate to register rs1" # Extract from "**Description**:"
  pseudocode: "x[rd] = x[rs1] + sext(imm[11:0])" # Extract from "**Pseudocode**:"
  invalid_values: "NONE"                       # Extract from "**Invalid values**:"
  exceptions: "NONE"                           # Extract from "**Exception raised**:"
  
operand_metadata:
  destination_register: "rd"                   # Extract from format
  source_register: "rs1"                       # Extract from format
  immediate_field: "imm[11:0]"                 # Extract from format
  immediate_range: "[-2048, 2047]"            # Infer from 12-bit signed
  immediate_encoding: "sign_extended"          # Extract from description
  
semantic_metadata:
  operation_type: "arithmetic"                 # Infer from "Add"
  side_effects: "none"                        # Infer from exceptions
  register_dependencies: ["rs1", "rd"]        # Extract from pseudocode
  supports_agents: ["ISA_Test_Writer"]        # Map to relevant agents
```

## Document Type 10: SystemVerilog Package Definitions

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