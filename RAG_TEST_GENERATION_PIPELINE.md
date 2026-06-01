# CVA6 RAG Test Generation Pipeline - Complete Flow

## Pipeline Overview

```
User Request → Main Test Agent → RAG System → Document Retrieval → 
Data Extraction → Structured Response → Test Generation → Final Test
```

## Stage 1: User Request & Agent Selection

**Input**: User verification request  
**Process**: Route to appropriate main test writing agent  
**Output**: Assigned agent with clear objective

**Example Requests:**
- "Generate a test for ADDI instruction verification"  
- "Create PMP granularity test for CV32A6"

## Stage 2: Main Test Writing Agent Analysis

**Input**: User request + agent expertise  
**Process**: Agent determines what information is needed  
**Output**: Structured RAG queries

### Agent Types & Query Patterns:
- **ISA Agent**: Needs instruction specs, verification plans, reference tests
- **Interface Agent**: Needs interface specs, verification plans, UVM components  
- **System Agent**: Needs system descriptions, configuration data, verification plans

## Stage 3: RAG System Query Processing (DETAILED)

### 3.1 Query Analysis
**Input**: Natural language or structured query from main agent  
**Process**: Parse query to identify:
- Document types needed
- Architecture scope (RV32/RV64)
- Feature/component focus
- Metadata filters

**Output**: Structured search parameters

### 3.2 Document Retrieval
**Input**: Search parameters  
**Process**: 
1. Search document index by metadata filters
2. Apply architecture/feature constraints
3. Rank documents by relevance
4. Select top relevant documents

**Output**: List of relevant document paths + metadata

### 3.3 Content Extraction  
**Input**: Document list + extraction rules  
**Process**:
1. Load documents from knowledge base
2. Apply document-type-specific extraction patterns
3. Extract structured data per metadata schema
4. Cross-reference using verification_tags

**Output**: Structured data objects per document type

### 3.4 Data Synthesis
**Input**: Extracted data from multiple document types  
**Process**:
1. Combine related information (same verification_tag)
2. Resolve conflicts between sources
3. Structure data per agent requirements
4. Add cross-reference links

**Output**: Comprehensive structured response

## Example 1: ADDI Instruction Test Generation

### Step 1: User Request
```
User: "Generate a test for ADDI instruction verification focusing on edge cases"
→ Routed to: ISA Test Writing Agent
```

### Step 2: Agent Analysis
```
ISA Agent determines needed information:
- ADDI instruction specification (official RISC-V)
- CVA6-specific ADDI implementation details
- Verification requirements for ADDI
- Existing reference tests for patterns
- Test framework requirements
```

### Step 3: RAG Query Formation
```yaml
Query 1: "ADDI instruction specification and verification requirements"
Filters:
  - document_types: ["verification_plan", "isa_specification", "cva6_instruction_specification"]  
  - feature: "ADDI" OR instruction_name: "ADDI"
  - target_architecture: "RV32"
```

### Step 4: Document Retrieval
```yaml
Found Documents:
1. Document Type 1: "dvplan_ISA_RV32.md"
   - verification_tag: "VP_ISA_RV32_F000_S000_I000" 
   - feature: "Register-Immediate Instructions"
   - sub_feature: "ADDI"

2. Document Type 3: "riscv-isa-manual/src/unpriv/rv32.adoc"
   - name: "RV32I Base Integer Instruction Set"
   - subcategory: "base_integer"

3. Document Type 9: "RISCV_Instructions_RV32I.rst"  
   - name: "RV32I"
   - feature: "Base Integer Instructions"
```

### Step 5: Data Extraction
```yaml
From Document Type 1 (Verification Plan):
verification_requirements:
  verification_goals: ["All possible rs1 registers are used", "All possible rd registers are used"]
  test_approach: "constrained_random"
  pass_criteria: "Check RM" 
  verification_tag: "VP_ISA_RV32_F000_S000_I000"
  coverage_requirements: ["isacov.rv32i_addi_cg.cp_rs1", "isacov.rv32i_addi_cg.cp_rd"]

From Document Type 3 (ISA Specification):
instruction_semantics:
  operation: "ADDI adds sign-extended 12-bit immediate to register rs1"
  operand_constraints: {"rs1": "any_register", "rd": "any_register", "imm": "12_bit_signed"}
  side_effects: "none"
  encoding_format: "I-type"
  opcode_value: "0010011"

From Document Type 9 (CVA6 Implementation):
implementation_specifics:
  instruction_format: "addi rd, rs1, imm[11:0]"
  operation_description: "add sign-extended 12-bit immediate to register rs1"
  pseudocode: "x[rd] = x[rs1] + sext(imm[11:0])"
  exceptions: "NONE"
  invalid_values: "NONE"
```

### Step 6: RAG Response Synthesis
```yaml
Comprehensive Response to ISA Agent:
instruction_specification:
  name: "ADDI" 
  format: "addi rd, rs1, imm[11:0]"
  semantics: "x[rd] = x[rs1] + sext(imm[11:0])"
  encoding: {"opcode": "0010011", "funct3": "000", "format": "I-type"}
  
verification_requirements:
  goals: ["test all rs1 registers", "test all rd registers", "test immediate edge cases"]
  approach: "constrained_random"
  success_criteria: "Check RM"
  coverage_points: ["isacov.rv32i_addi_cg.cp_rs1", "isacov.rv32i_addi_cg.cp_rd"]
  
implementation_constraints:
  exceptions: "none"
  special_cases: "none" 
  architectural_effects: "updates_rd_register_only"

cross_references:
  verification_tag: "VP_ISA_RV32_F000_S000_I000"
  related_documents: ["dvplan_ISA_RV32.md", "rv32.adoc", "RISCV_Instructions_RV32I.rst"]
```

### Step 7: Test Generation
```
ISA Agent receives comprehensive data and generates:
1. Test structure based on verification goals
2. Register coverage ensuring all rs1/rd combinations
3. Immediate value edge cases (-2048, 2047, 0, -1)
4. Framework integration with coverage points
5. Pass/fail criteria matching "Check RM" requirement
```

## Example 2: PMP Granularity Test Generation

### Step 1: User Request  
```
User: "Create a test for PMP granularity determination on CV32A6"
→ Routed to: System Test Writing Agent
```

### Step 2: Agent Analysis
```
System Agent determines needed information:
- PMP granularity verification requirements
- PMP implementation details in CVA6
- Memory management configuration
- Test framework for system-level tests
```

### Step 3: RAG Query Formation
```yaml
Query: "PMP granularity verification requirements and implementation"
Filters:
  - document_types: ["verification_plan", "verification_plan_feature", "configuration_data"]
  - feature: "PMP" OR name: "*granularity*"
  - verification_tag: "*PMP*"
  - target_architecture: "RV32"
```

### Step 4: Document Retrieval
```yaml
Found Documents:
1. Document Type 1: "dvplan_PMP.md"
   - subcategory: "system"
   - feature: "TRISTAN Restrictions"

2. Document Type 11: "VP_IP001.yml" 
   - verification_tag: "VP_PMP_F001_S000_I000"
   - name: "PMP granularity"

3. Document Type 6: "cv32a6_imac_sv0_config.yaml"
   - target_cores: ["cv32a6"]
   - supported_isa: ["rv32i", "rv32m", "rv32a", "rv32c"]
```

### Step 5: Data Extraction
```yaml
From Document Type 1 (Verification Plan MD):
verification_context:
  feature_description: "the verif plan is written for 32bits architecture only"
  verification_goals: [general PMP verification goals]
  applicable_cores: ["CV32E40P", "CV32E40S", "CV32E40X", "CV32A6_v0.1.0"]
  
From Document Type 11 (Structured Verification Feature):  
detailed_requirements:
  verification_tag: "VP_PMP_F001_S000_I000"
  feature_name: "PMP granularity"
  verification_goals: "determine the PMP granularity 2^(G+2) bytes by writing zero to pmp(0)cfg, then writing all ones to pmpaddr(0), then reading back pmpaddr(0). G is the index of the least-significant bit set"
  description: "Software may determine the PMP granularity by writing zero to pmp0cfg, then writing all ones to pmpaddr0, then reading back pmpaddr0"
  test_type: 2
  pass_fail_criteria: 11
  reference_document: "RISC-V Privileged Architectures V20211203"
  reference_page: "59"

From Document Type 6 (Configuration Data):
system_constraints:
  target_cores: ["cv32a6"]
  word_size: 32
  privilege_modes: ["M", "U"] 
  physical_addr_sz: 34
```

### Step 6: RAG Response Synthesis
```yaml
Comprehensive Response to System Agent:
test_specification:
  verification_tag: "VP_PMP_F001_S000_I000"
  objective: "Determine PMP granularity through register manipulation"
  
test_procedure:
  algorithm: "Write 0 to pmp0cfg → Write 0xFFFFFFFF to pmpaddr0 → Read pmpaddr0 → Find LSB set position"
  granularity_formula: "2^(G+2) bytes where G = index of least significant bit set"
  
system_requirements:
  privilege_mode: "Machine"  # Required for PMP access
  target_architecture: "RV32"
  applicable_cores: ["CV32A6_v0.1.0"]
  register_access: ["pmp0cfg", "pmpaddr0"]
  
verification_criteria:
  success_condition: "Granularity calculation matches expected value"
  reference_authority: "RISC-V Privileged Spec V20211203 Page 59"
  
implementation_constraints:
  address_space: "34-bit physical addressing"
  supported_modes: ["Machine", "User"]
```

### Step 7: Test Generation
```
System Agent receives comprehensive data and generates:
1. Machine-mode test setup
2. PMP register access sequence (pmp0cfg=0, pmpaddr0=0xFFFFFFFF)
3. Granularity calculation logic
4. Expected result validation for CV32A6
5. Test framework integration with system-level environment
```

## RAG System Benefits Demonstrated

### Information Integration:
- **Multiple Document Types**: Combines verification requirements (Type 1,11) with specifications (Type 3,9) and configuration (Type 6)
- **Cross-References**: Links related information using verification_tag
- **Architecture-Specific**: Filters for CV32A6/RV32 relevant information

### Comprehensive Coverage:
- **What to Test**: From verification plans and structured features  
- **How It Works**: From official specifications and CVA6 implementation
- **System Context**: From configuration data and constraints
- **Test Framework**: From test infrastructure requirements

This pipeline ensures test generation agents receive complete, accurate, and architecturally-appropriate information for creating comprehensive verification tests.