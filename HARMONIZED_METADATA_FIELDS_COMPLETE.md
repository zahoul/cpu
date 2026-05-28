# Complete List of 46 Harmonized Metadata Fields

## Universal Fields (8) - All Document Types

| Field Name | Example | Used In Types |
|------------|---------|---------------|
| `document_type` | "verification_plan" | 1,2,3,4,5,6,7,8,9,10 |
| `file_path` | "cva6/verif/docs/VerifPlans/source/dvplan_ISA_RV32.md" | 1,2,3,4,5,6,7,8,9,10 |
| `name` | "ISA_RV32" | 1,2,3,4,5,6,7,8,9,10 |
| `name_type` | "module" | 1,2,3,4,5,6,7,8,9,10 |
| `target_architecture` | "RV32" | 1,2,3,4,5,6,7,8,9,10 |
| `word_size` | 32 | 1,2,3,4,5,6,7,8,9,10 |
| `category` | "verification" | 1,2,3,4,5,6,7,8,9,10 |
| `subcategory` | "isa" | 1,2,3,4,5,6,7,8,9,10 |

## Content-Specific Fields (8) - Subset of Document Types

| Field Name | Example | Used In Types |
|------------|---------|---------------|
| `feature` | "Register-Immediate Instructions" | 1,4,9 |
| `sub_feature` | "ADDI" | 1,4,9 |
| `content_type` | "constrained_random" | 1,3,4,5,8,9 |
| `source_type` | "official" | 1,3,4,5,8,9 |
| `privilege_level` | "machine" | 5,6,8 |
| `target_cores` | ["CV32A6_v0.1.0", "CV32A6-step2"] | 1,5,6 |
| `dependencies` | ["ariane_pkg"] | 2,7,10 |
| `test_framework` | "riscv_test_macros" | 4,5,8 |

## Document-Specific Fields (30) - Unique to Specific Types

### Type 1: Verification Plans (4 fields)
| Field Name | Example |
|------------|---------|
| `verification_tag` | "VP_ISA_RV32_F000_S000_I000" |
| `verification_goals` | ["All possible rs1 registers are used"] |
| `coverage_links` | ["isacov.rv32i_addi_cg.cp_rs1"] |
| `pass_fail_criteria` | "Check RM" |

### Type 2: Design Implementation (3 fields)
| Field Name | Example |
|------------|---------|
| `input_ports` | [{"name": "operand_a_i", "width": 32, "type": "logic"}] |
| `output_ports` | [{"name": "result_o", "width": 32, "type": "logic"}] |
| `parameters` | [{"name": "WIDTH", "default_value": 32, "type": "int unsigned"}] |

### Type 3: ISA Specifications (4 fields)
| Field Name | Example |
|------------|---------|
| `chapter` | "RV32I Base Integer Instruction Set" |
| `register_model` | "32_general_purpose_registers" |
| `special_registers` | ["x0_hardwired_zero", "pc"] |
| `instruction_categories` | {"integer_register_immediate": ["ADDI", "SLTI"]} |

### Type 4: Test Examples (2 fields)
| Field Name | Example |
|------------|---------|
| `language` | "assembly" |
| `instructions_used` | ["li", "csrrw", "csrrs", "nop"] |

### Type 5: Test Configurations (3 fields)
| Field Name | Example |
|------------|---------|
| `gcc_opts` | "-static -mcmodel=medany" |
| `test_references` | [{"test": "rv32ui-p-addi", "asm_tests": "riscv-tests/isa/rv32ui/addi.S"}] |
| `include_paths` | ["/riscv-tests/isa/macros/scalar/"] |

### Type 6: Configuration Data (4 fields)
| Field Name | Example |
|------------|---------|
| `physical_addr_sz` | 34 |
| `supported_isa` | ["rv32i", "rv32m", "rv32a", "rv32c", "rv32zicsr"] |
| `custom_extensions` | ["Xcorev"] |
| `mtvec_mode` | ["vectored", "direct"] |

### Type 7: Interface Specifications (6 fields)
| Field Name | Example |
|------------|---------|
| `protocol_version` | "4.0" |
| `interface_type` | "memory_bus" |
| `direction` | "master" |
| `key_signals` | ["AWVALID", "AWREADY", "AWBURST"] |
| `signal_constraints` | {"AWBURST": "always INCR = 2'b01"} |
| `timing_constraints` | ["setup", "hold"] |

### Type 8: RISC-V Test Suite (3 fields)
| Field Name | Example |
|------------|---------|
| `test_vm` | "rv32ui" |
| `test_cases_count` | 16 |
| `macro_types_used` | ["TEST_IMM_OP", "TEST_IMM_SRC1_EQ_DEST"] |

### Type 9: CVA6 Instruction Specifications (5 fields)
| Field Name | Example |
|------------|---------|
| `format` | "addi rd, rs1, imm[11:0]" |
| `description` | "add sign-extended 12-bit immediate to register rs1" |
| `pseudocode` | "x[rd] = x[rs1] + sext(imm[11:0])" |
| `invalid_values` | "NONE" |
| `exceptions` | "NONE" |

### Type 10: SystemVerilog Packages (6 fields)
| Field Name | Example |
|------------|---------|
| `opcode_definitions` | {"OpcodeOpImm": "7'b0010011"} |
| `instruction_formats` | {"itype_t": {"imm": 12, "rs1": 5, "opcode": 7}} |
| `constant_definitions` | {"XLEN": 32, "REG_ADDR_SIZE": 5} |
| `type_definitions` | ["rtype_t", "itype_t", "stype_t", "utype_t"] |
| `import_dependencies` | ["cva6_config_pkg"] |
| `csr_definitions` | {"CSR_MSTATUS": "12'h300", "CSR_MIE": "12'h304"} |

## Field Distribution Summary

- **Universal Fields**: 8 fields (used by all 10 document types)
- **Content-Specific Fields**: 8 fields (used by 2-6 document types each)
- **Document-Specific Fields**: 30 fields (unique to 1-2 document types)
- **Total Harmonized Fields**: 46 fields

## Key Consolidations Made

### Before Harmonization (Original Field Count: ~60)
- Name fields: `module`, `module_name`, `test_name`, `testlist_name`, `core_name`, `interface_name`, `instruction_name`, `package_name` (8 fields)
- Size fields: `word_size`, `data_width` (2 fields)
- Category fields: `category`, `extension_category`, `test_category`, `instruction_category`, `package_category`, `interface_type` (6 fields)
- Type fields: `test_type`, `specification_type`, `specification_source` (3 fields)
- Privilege fields: `privilege_mode`, `privilege_level`, `privilege_modes` (3 fields)
- Core fields: `applicable_cores`, `target_core`, `core_name` (3 fields)

### After Harmonization (Consolidated to 16 fields)
- Name fields: `name` + `name_type` (2 fields)
- Size fields: `word_size` (1 field)
- Category fields: `category` + `subcategory` (2 fields)  
- Type fields: `content_type` + `source_type` (2 fields)
- Privilege fields: `privilege_level` (1 field)
- Core fields: `target_cores` (1 field)

**Space Saved**: 25 redundant fields eliminated → 44% reduction in field complexity

## Benefits for RAG Implementation

1. **Index Efficiency**: Single columns for logically equivalent data
2. **Query Consistency**: Same field names across document types
3. **Aggregation Simplified**: Group by `category`/`subcategory` instead of 6 different category fields
4. **Cross-References**: Unified `name` + `name_type` + `target_architecture` for linking
5. **Maintenance**: Single field definition and validation logic