# RAG Knowledge Base Folder Structure

## Overview
This document defines the folder structure for organizing indexed documents in the CVA6 RAG knowledge base. The structure is designed to support efficient querying by document type, architecture variant, and cross-repository relationships.

## Root Structure

```
knowledge_base/
├── verification_plans/           # Document Type 1
├── testlists/                   # Document Type 2
├── reference_tests/             # Document Type 3
├── instruction_specs/           # Document Type 4
├── system_descriptions/         # Document Type 5
├── uvm_components/              # Document Type 6
├── interface_specs/             # Document Type 7
├── isa_specifications/          # Document Type 8
├── test_infrastructure/         # Document Type 9
└── systemverilog_packages/      # Document Type 10
```

## Detailed Folder Structure

### 1. verification_plans/
```
verification_plans/
├── rv32/
│   ├── isa/                     # ISA verification plans
│   │   ├── dvplan_ISA_RV32.md
│   │   ├── dvplan_ZICSR.md
│   │   └── dvplan_exceptions.md
│   ├── system/                  # System-level plans
│   │   ├── dvplan_MMU.md
│   │   └── dvplan_interrupts.md
│   └── interface/               # Interface plans
│       ├── dvplan_AXI.md
│       └── dvplan_RVFI.md
└── rv64/
    ├── isa/
    ├── system/
    └── interface/
```

### 2. testlists/
```
testlists/
├── rv32/
│   ├── isa/
│   │   ├── RV32I_testlist.yaml
│   │   ├── RV32M_testlist.yaml
│   │   └── ZICSR_testlist.yaml
│   ├── system/
│   │   ├── MMU_testlist.yaml
│   │   └── interrupt_testlist.yaml
│   └── interface/
│       ├── AXI_testlist.yaml
│       └── RVFI_testlist.yaml
└── rv64/
    ├── isa/
    ├── system/
    └── interface/
```

### 3. reference_tests/
```
reference_tests/
├── rv32/
│   ├── ui/                      # User-level integer tests
│   │   ├── add.S
│   │   ├── addi.S
│   │   └── ...
│   ├── uf/                      # User-level floating-point tests
│   ├── si/                      # Supervisor-level integer tests
│   └── mi/                      # Machine-level integer tests
└── rv64/
    ├── ui/
    ├── uf/
    ├── si/
    └── mi/
```

### 4. instruction_specs/
```
instruction_specs/
├── rv32/
│   ├── base/
│   │   ├── arithmetic/
│   │   │   ├── ADD.rst
│   │   │   ├── ADDI.rst
│   │   │   └── ...
│   │   ├── logical/
│   │   │   ├── AND.rst
│   │   │   ├── ANDI.rst
│   │   │   └── ...
│   │   ├── memory/
│   │   │   ├── LW.rst
│   │   │   ├── SW.rst
│   │   │   └── ...
│   │   └── control/
│   │       ├── BEQ.rst
│   │       ├── JAL.rst
│   │       └── ...
│   ├── extensions/
│   │   ├── m_extension/         # Multiply/Divide
│   │   ├── f_extension/         # Single-precision float
│   │   ├── d_extension/         # Double-precision float
│   │   └── c_extension/         # Compressed instructions
│   └── privileged/
│       ├── csr_instructions/
│       ├── trap_instructions/
│       └── virtual_memory/
└── rv64/
    ├── base/
    ├── extensions/
    └── privileged/
```

### 5. system_descriptions/
```
system_descriptions/
├── architecture/
│   ├── pipeline_overview.md
│   ├── execution_units.md
│   └── memory_subsystem.md
├── interfaces/
│   ├── axi_interface.md
│   ├── rvfi_interface.md
│   └── debug_interface.md
├── configurations/
│   ├── cv32a60x_config.md
│   ├── cv64a6_config.md
│   └── fpga_configs.md
└── features/
    ├── branch_prediction.md
    ├── cache_hierarchy.md
    └── exception_handling.md
```

### 6. uvm_components/
```
uvm_components/
├── agents/
│   ├── rvfi_agent/
│   │   ├── uvma_rvfi_agent.sv
│   │   ├── uvma_rvfi_driver.sv
│   │   └── uvma_rvfi_monitor.sv
│   ├── axi_agent/
│   └── interrupt_agent/
├── environments/
│   ├── core_env/
│   └── system_env/
├── sequences/
│   ├── instruction_sequences/
│   ├── interrupt_sequences/
│   └── memory_sequences/
└── scoreboards/
    ├── rvfi_scoreboard/
    └── memory_scoreboard/
```

### 7. interface_specs/
```
interface_specs/
├── rvfi/
│   ├── rvfi_specification.md
│   ├── rvfi_signals.md
│   └── rvfi_protocol.md
├── axi/
│   ├── axi4_specification.md
│   ├── axi_transactions.md
│   └── axi_memory_map.md
├── debug/
│   ├── debug_module_spec.md
│   └── debug_transport_spec.md
└── cvxif/
    ├── cvxif_protocol.md
    └── custom_extensions.md
```

### 8. isa_specifications/
```
isa_specifications/
├── volumes/
│   ├── volume_1_unprivileged.md  # Base ISA, extensions
│   ├── volume_2_privileged.md    # Privileged architecture
│   └── volume_3_debug.md         # Debug specification
├── chapters/
│   ├── rv32i_base.md
│   ├── rv64i_base.md
│   ├── extensions/
│   │   ├── m_extension.md
│   │   ├── f_extension.md
│   │   └── ...
│   └── privileged/
│       ├── machine_mode.md
│       ├── supervisor_mode.md
│       └── virtual_memory.md
└── appendices/
    ├── instruction_listings.md
    └── csr_listings.md
```

### 9. test_infrastructure/
```
test_infrastructure/
├── macros/
│   ├── riscv_test.h
│   ├── test_macros.h
│   └── encoding.h
├── environments/
│   ├── p/                       # Physical memory, single core
│   ├── v/                       # Virtual memory
│   ├── pm/                      # Physical memory, multi-core
│   └── pt/                      # Physical memory, timer interrupts
├── common/
│   ├── link_scripts/
│   ├── startup_code/
│   └── utility_functions/
└── generators/
    ├── test_generators.md
    ├── random_generators.md
    └── constraint_solvers.md
```

### 10. systemverilog_packages/
```
systemverilog_packages/
├── core_packages/
│   ├── riscv_pkg.sv
│   ├── ariane_pkg.sv
│   ├── config_pkg.sv
│   └── build_config_pkg.sv
├── config_packages/
│   ├── rv32_configs/
│   │   ├── cv32a60x_config_pkg.sv
│   │   ├── cv32a65x_config_pkg.sv
│   │   └── ...
│   └── rv64_configs/
│       ├── cv64a6_imafdc_sv39_config_pkg.sv
│       ├── cv64a6_imafdch_sv39_config_pkg.sv
│       └── ...
├── interface_packages/
│   ├── axi_pkg.sv
│   ├── rvfi_pkg.sv
│   └── cvxif_pkg.sv
├── verification_packages/
│   ├── uvma_core_cntrl_pkg.sv
│   ├── uvme_cva6_pkg.sv
│   └── cva6_instr_test_pkg.sv
└── utility_packages/
    ├── math_pkg.sv
    ├── ecc_pkg.sv
    └── cb_filter_pkg.sv
```

## Indexing Strategy

### File Naming Convention
- Use original filenames from repositories
- Add metadata suffixes for duplicates: `filename_repo.ext`
- Maintain source repository path information in metadata

### Metadata Storage
Each document includes:
```yaml
metadata:
  document_type: "verification_plans"
  architecture: "rv32" | "rv64" | "both"
  category: "isa" | "system" | "interface"
  source_repo: "cva6" | "riscv-tests" | "riscv-isa-manual"
  source_path: "original/path/in/repo"
  instruction_class: "arithmetic" | "logical" | "memory" | "control" | "privileged"
  test_level: "unit" | "integration" | "system"
  dependencies: ["list", "of", "related", "documents"]
```

### Cross-Reference Links
- Maintain bidirectional links between related documents
- Index by instruction name, test name, feature name
- Support queries by multiple metadata dimensions

## Query Examples

```python
# Query by document type and architecture
query("document_type:verification_plans AND architecture:rv32")

# Query by instruction class across document types
query("instruction_class:arithmetic AND (document_type:instruction_specs OR document_type:reference_tests)")

# Query for UVM components related to RVFI
query("document_type:uvm_components AND interface:rvfi")

# Cross-repository query for ADDI instruction
query("instruction_name:ADDI")
```

## Storage Considerations
- Total estimated size: ~500MB indexed content
- Support for incremental updates when repositories change
- Efficient search indices on metadata fields
- Version control integration for document updates