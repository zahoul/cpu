# CVA6 RAG Document Types & Metadata Structure

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