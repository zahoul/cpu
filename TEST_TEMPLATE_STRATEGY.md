# CVA6 Test Template Definition Strategy

## Who Defines Templates?

### 1. **Initial Template Creation: Verification Engineers**
- **Domain experts** who understand both RISC-V and CVA6 specifics
- **Template architects** who can identify reusable patterns
- **One-time investment** to create comprehensive template library

### 2. **Template Maintenance: Combination**
- **Manual updates**: Verification engineers for new test types
- **Semi-automated**: RAG system identifies new patterns from existing tests
- **Agent-assisted**: Agents propose new templates based on recurring patterns

## Template Identification Process

### Phase 1: Pattern Analysis (Automated)
```yaml
# RAG system analyzes existing tests to find patterns
pattern_analysis:
  input: All existing test files (Document Types 4, 8)
  process:
    - Extract common code structures
    - Identify repeated instruction sequences
    - Group tests by functionality (CSR, ALU, Memory, etc.)
    - Find parameterizable sections
  output: Pattern candidates for template creation
```

### Phase 2: Template Design (Human + AI)
```yaml
# Verification engineers + AI assistants design templates
template_design:
  input: Pattern candidates from Phase 1
  process:
    - Identify parameterizable elements
    - Design template interface (parameters)
    - Create reusable macro structures
    - Define template metadata
  output: Template specifications
```

## Template Categories to Create

### 1. **CSR Access Templates**
Based on the complex test you showed:

```assembly
# Template: csr_read_only_test.S
# Parameters: csr_name, csr_addr, expected_value, test_patterns[]

.macro csr_read_only_test csr_name, csr_addr, expected_value
    # Setup exception handling
    la x6, exception_handler
    csrw mtvec, x6
    csrw mepc, x0
    csrw mcause, x0
    
    # Test read operation
    csrr x31, \csr_addr
    li x6, \expected_value
    bne x31, x6, csr_fail
    
    # Test write operations (should not change read-only CSR)
    .irp pattern, 0xa5a5a5a5, 0x5a5a5a5a, 0x82cebeaf
        # CSRRW test
        li x4, \pattern
        csrrw x14, \csr_addr, x4
        li x4, \expected_value
        bne x4, x14, csr_fail
        
        # CSRRS test  
        li x4, \pattern
        csrrs x14, \csr_addr, x4
        li x4, \expected_value
        bne x4, x14, csr_fail
        
        # CSRRC test
        li x4, \pattern
        csrrc x14, \csr_addr, x4
        li x4, \expected_value
        bne x4, x14, csr_fail
    .endr
    
    j csr_pass
.endm
```

### 2. **Instruction Testing Templates**
```assembly
# Template: instruction_basic_test.S
# Parameters: instruction, operand_patterns[]

.macro test_instruction inst, rs1_val, rs2_val, expected_result
    li x1, \rs1_val
    li x2, \rs2_val
    \inst x3, x1, x2
    li x4, \expected_result
    bne x3, x4, test_fail
.endm
```

### 3. **Memory Access Templates**
```assembly
# Template: memory_access_test.S
# Parameters: access_type, address_patterns[], data_patterns[]

.macro memory_test_pattern access_type, base_addr, test_data
    la x1, \base_addr
    li x2, \test_data
    \access_type x2, 0(x1)    # Store or Load
    # Validation logic here
.endm
```

## Template Definition Workflow

### Step 1: Existing Test Analysis
```python
# Automated analysis of existing tests
def analyze_existing_tests():
    test_files = glob("cva6/verif/tests/custom/**/*.S")
    
    patterns = {
        'csr_access': extract_csr_patterns(test_files),
        'instruction_test': extract_instruction_patterns(test_files), 
        'memory_test': extract_memory_patterns(test_files),
        'exception_handling': extract_exception_patterns(test_files)
    }
    
    return identify_template_candidates(patterns)
```

### Step 2: Template Specification
```yaml
# Template specification format
template_name: "csr_read_only_test"
category: "csr_access"
description: "Tests read-only CSR behavior with various write attempts"

parameters:
  - name: "csr_name"
    type: "string"
    description: "Human-readable CSR name"
    example: "MARCHID"
    
  - name: "csr_addr" 
    type: "integer"
    description: "CSR address (12-bit)"
    example: 3858
    constraints: [0, 4095]
    
  - name: "expected_value"
    type: "hex_integer"
    description: "Expected read value for this CSR"
    example: "0x00000003"

generated_sections:
  - exception_handler_setup
  - initial_value_read_test
  - write_attempt_tests_with_patterns
  - verification_after_writes
  - pass_fail_logic

dependencies:
  headers: ["model_test.h"]
  macros: ["RVMODEL_DATA_BEGIN", "RVMODEL_DATA_END"]
  
test_framework_integration:
  result_communication: "tohost"
  pass_indicator: "li x1, 0; slli x1, x1, 1; addi x1, x1, 1"
  fail_indicator: "li x1, 1; slli x1, x1, 1; addi x1, x1, 1"
```

### Step 3: Template Implementation
```assembly
# File: templates/csr_read_only_test.S.template
# This is the actual template file with parameter substitution

#include "model_test.h"
.macro init
.endm
.section .text.init
.globl _start
.option norvc
.org 0x00

_start:
    # Exception handler setup
    la x6, exception_handler
    csrw mtvec, x6
    csrw mepc, x0
    csrw mcause, x0

    # Test ${csr_name} CSR (address ${csr_addr})
    csrr x31, ${csr_addr}
    li x6, ${expected_value}
    bne x31, x6, csr_fail
    
    # Write attempt tests with various patterns
% for pattern in test_patterns:
    # Test pattern ${pattern}
    li x4, ${pattern}
    csrrw x14, ${csr_addr}, x4
    li x4, ${expected_value}
    bne x4, x14, csr_fail
% endfor

    j csr_pass

# Standard exception handler (reusable across templates)
exception_handler:
    csrr x30, mepc
    csrr x31, mcause
    addi x2, x2, 2
    beq x31, x2, next
    j csr_fail

next:
    addi x1, x1, 0
    bne x30, x1, next_iter
    j csr_fail

next_iter:
    li x2, 0
    addi x7, x30, 4
    jr x7

# Standard pass/fail logic (reusable)
csr_pass:
    li x1, 0
    slli x1, x1, 1
    addi x1, x1, 1
    sw x1, tohost, t5
    self_loop: j self_loop

csr_fail:
    li x1, 1
    slli x1, x1, 1
    addi x1, x1, 1
    sw x1, tohost, t5
    self_loop_2: j self_loop_2

RVMODEL_DATA_BEGIN
RVMODEL_DATA_END
```

## Template Library Organization

### Directory Structure
```
cva6/verif/templates/
├── csr/
│   ├── csr_read_only_test.S.template
│   ├── csr_read_write_test.S.template
│   └── csr_exception_test.S.template
├── instruction/
│   ├── alu_basic_test.S.template
│   ├── branch_test.S.template
│   └── load_store_test.S.template
├── system/
│   ├── interrupt_test.S.template
│   ├── exception_test.S.template
│   └── privilege_test.S.template
└── common/
    ├── exception_handlers.S
    ├── test_macros.S
    └── framework_integration.S
```

### Template Metadata Database
```yaml
# templates/metadata.yaml
templates:
  - name: "csr_read_only_test"
    file: "csr/csr_read_only_test.S.template"
    category: "csr_access"
    parameters: [...]
    supports_verification_tags: ["VP_CSR_*_READ_ONLY"]
    
  - name: "alu_instruction_test"
    file: "instruction/alu_basic_test.S.template"  
    category: "instruction_test"
    parameters: [...]
    supports_verification_tags: ["VP_ISA_RV32_F000*"]
```

## Integration with RAG Agents

### Agent Query Enhancement
```yaml
# RAG query includes template matching
rag_query:
  query_text: "Test MARCHID CSR read-only behavior"
  metadata_filters:
    document_type: ["verification_plan", "test_template"]
    feature: "CSR_access"
    
# RAG response includes both requirements AND templates
rag_response:
  verification_requirements: {...}
  matching_templates:
    - template_name: "csr_read_only_test"
      confidence: 0.95
      required_parameters:
        csr_name: "MARCHID"
        csr_addr: 3858  # From SystemVerilog packages
        expected_value: "0x00000003"  # From CVA6 config
```

### Template Generation Workflow
```python
def generate_test_from_template(agent_decision, template_spec):
    """
    Agent provides: test intent + parameters
    Template engine provides: low-level implementation
    """
    template = load_template(template_spec.template_name)
    parameters = resolve_parameters(agent_decision, template_spec)
    
    # Parameter resolution uses RAG system
    parameters['csr_addr'] = rag_query_csr_address(parameters['csr_name'])
    parameters['expected_value'] = rag_query_csr_default_value(parameters['csr_name'])
    
    return render_template(template, parameters)
```

## Implementation Timeline

### Phase 1: Template Library Bootstrap (4-6 weeks)
1. **Week 1-2**: Analyze existing tests, identify top 10 patterns
2. **Week 3-4**: Create initial template specifications  
3. **Week 5-6**: Implement and test template engine

### Phase 2: Agent Integration (2-3 weeks)
1. **Week 7-8**: Integrate templates with RAG responses
2. **Week 9**: Test agent → template → generated test workflow

### Phase 3: Template Expansion (Ongoing)
1. Add new templates based on verification plan coverage
2. Refine existing templates based on generated test quality
3. Automated template suggestion based on test patterns

## Benefits of This Approach

1. **Scalability**: Templates handle complexity, agents handle logic
2. **Maintainability**: Update template once, affects all generated tests  
3. **Quality**: Templates written by experts, ensuring correctness
4. **Flexibility**: New templates can be added without agent retraining
5. **Verification Coverage**: Templates ensure comprehensive test patterns