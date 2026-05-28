# CVA6 RAG Document Types & Metadata Structure - UPDATED

## Summary

I've successfully updated the metadata structure for all 10 document types with:

1. **Purpose sections** explaining what, why, and when each document type is used
2. **RAG Agent Data Extraction** sections specifying exactly what data to send to main agents
3. **Harmonized metadata schemas** using the unified 46-field structure
4. **Concrete examples** showing exactly where and how to extract each field
5. **Extraction patterns** demonstrating the parsing logic needed

## Key Improvements Made

### 1. **Clear Purpose Definitions**
Each document type now has:
- **What:** Description of the document content
- **Why:** Explanation of why this information is needed
- **When Used:** Context for when agents need this data

### 2. **Structured RAG Data Extraction**
For each document type, RAG agents now know exactly what to send:
- **MUST SEND:** Critical data agents absolutely need
- **SHOULD SEND:** Important contextual information  
- **HELPFUL TO SEND:** Additional details that improve decisions

### 3. **Harmonized Metadata Schema**
All document types now use the unified structure:
- **Universal Fields (8):** Used by all document types
- **Content-Specific Fields (8):** Used by relevant subsets
- **Document-Specific Fields (30):** Unique to specific types

### 4. **Concrete Extraction Examples**
Each field now shows:
- **Exact source patterns:** What to look for in the files
- **Extraction logic:** How to parse the information  
- **Example values:** What the extracted data should look like

## Example: Document Type 1 (Verification Plans)

```yaml
# FROM: "### Sub-feature: 000_ADDI"  →  sub_feature: "ADDI"  
# FROM: "* **Test Type:** Constrained Random"  →  content_type: "constrained_random"
# FROM: "* **Applicable Cores:** CV32A6_v0.1.0, CV32A6-step2"  →  target_cores: ["CV32A6_v0.1.0", "CV32A6-step2"]
```

## Benefits for Implementation

1. **Clear Implementation Guidance:** Developers know exactly what to extract and how
2. **Consistent Data Structure:** All document types use the same field names where appropriate
3. **Reduced Complexity:** From ~60 unique fields down to 46 harmonized fields
4. **Better Agent Support:** RAG agents provide exactly the right data for test generation decisions

## Next Steps

The updated document provides everything needed to implement the RAG system:
- Exact extraction patterns for each field
- Clear data priorities (MUST/SHOULD/HELPFUL)
- Harmonized schema reducing index complexity
- Concrete examples for all extraction logic

This comprehensive specification ensures RAG agents can extract the right information to support informed test generation decisions by the main test writing agents.