# Custom Metadata Support - Investigation

**Date**: 2025-10-22
**Issue**: TODOs at `application/_cli.py:704` and `application/_gui/_page_application_describe.py:702`
**Status**: Investigation / Design Phase

## Current Situation

Both CLI and GUI interfaces for submitting application runs have `custom_metadata=None` hardcoded:

```python
# application/_cli.py:704
run = service.submit_run(
    ...
    custom_metadata=None,  # TODO(Helmut): Add support for custom metadata
    ...
)

# application/_gui/_page_application_describe.py:702
run = service.submit_run(
    ...
    custom_metadata=None,  # TODO(Helmut): Allow user to edit custom metadata
    ...
)
```

## Investigation Questions

### 1. API Support
- ✅ Check: Does the Platform API support custom metadata on run submission?
- Location: Check `platform/resources/runs.py` and codegen models
- Expected: `RunCreationRequest` should have optional `custom_metadata` field

### 2. Data Model
- What is the expected type of custom_metadata?
  - Dict[str, Any]?
  - JSON string?
  - Structured Pydantic model?
- Are there validation requirements?
  - Size limits?
  - Key naming conventions?
  - Value type restrictions?

### 3. Use Cases
- What are the intended use cases for custom metadata?
  - Experiment tracking?
  - User annotations?
  - External system integration IDs?
  - Workflow context?

### 4. User Experience

#### CLI Design Options:
a. **JSON file input** (recommended for complex metadata):
   ```bash
   aignostics application run submit \
     --application-id heta \
     --files "*.svs" \
     --custom-metadata-file metadata.json
   ```

b. **Key-value pairs** (good for simple metadata):
   ```bash
   aignostics application run submit \
     --application-id heta \
     --files "*.svs" \
     --custom-metadata experiment_id=EXP-123 \
     --custom-metadata batch=2025-Q4
   ```

c. **JSON string** (flexible but error-prone):
   ```bash
   aignostics application run submit \
     --application-id heta \
     --files "*.svs" \
     --custom-metadata '{"experiment_id": "EXP-123", "batch": "2025-Q4"}'
   ```

#### GUI Design Options:
a. **JSON editor widget** (NiceGUI JSON editor)
b. **Key-value pair table** (dynamic rows)
c. **Form fields** (if schema is known)

### 5. Validation & Error Handling
- Should custom metadata be validated before submission?
- What happens if API rejects custom metadata?
- Should we provide user feedback on metadata requirements?

## Recommended Implementation Steps

### Phase 1: API Investigation (This Branch)
1. ✅ Create investigation document (this file)
2. Check codegen models for `custom_metadata` field
3. Review Platform API documentation
4. Test API endpoint with custom metadata (if possible)
5. Document findings

### Phase 2: Data Model Design
1. Define Python type for custom_metadata
2. Create Pydantic validation if needed
3. Add size/format validation
4. Update service layer signature

### Phase 3: CLI Implementation
1. Add `--custom-metadata-file` option (JSON file)
2. Add `--custom-metadata` option (key-value pairs, multiple allowed)
3. Merge and validate metadata
4. Update help text and examples

### Phase 4: GUI Implementation
1. Add metadata editor widget to submission page
2. Implement JSON validation in UI
3. Add save/load presets feature
4. Update user guide

### Phase 5: Testing & Documentation
1. Unit tests for metadata validation
2. Integration tests for API calls
3. CLI reference documentation
4. User guide with examples

## Action Items

- [ ] Review `aignx.codegen.models.RunCreationRequest` for custom_metadata field
- [ ] Check Platform API v1.0.0-beta.7 documentation
- [ ] Create example custom_metadata payload
- [ ] Test API with custom_metadata (staging environment)
- [ ] Decide on CLI UX (file vs key-value vs JSON string)
- [ ] Decide on GUI UX (JSON editor vs form)
- [ ] Create implementation plan with effort estimates

## Notes

- This feature has been TODO since initial development
- High user value - enables workflow integration and experiment tracking
- Medium complexity - requires API, CLI, and GUI changes
- Should maintain backward compatibility (metadata is optional)

## Related Files

- `src/aignostics/application/_service.py` - Service layer implementation
- `src/aignostics/application/_cli.py` - CLI interface
- `src/aignostics/application/_gui/_page_application_describe.py` - GUI interface
- `src/aignostics/platform/resources/runs.py` - API client
- `codegen/out/aignx/codegen/models/run_creation_request.py` - API model

## References

- [OE Audit] 2025-10-22
- Platform API v1.0.0-beta.7
