# git-learning
learning and experiments on git commands
## Automated Serialization Compatibility Checking - Solution Proposal

### Problem
Serialization field changes (renames, type changes, transient modifiers) break production when deserializing old data from cache/Kafka/databases. Currently caught only in production incidents.

### Solution
Implement automated serialization compatibility checking in Bitbucket Pipelines that runs on every PR.

### How It Works

**1. One-Time Setup:**
- Generate baseline snapshot (JSON file) of all Serializable class fields
- Commit baseline to repository (~5MB for 10M LOC)
- Add `bitbucket-pipelines.yml` configuration

**2. Automatic PR Checks:**
- Bitbucket runs on every PR automatically
- Git-aware: Only checks changed files (not entire codebase)
- Compares changed classes against baseline
- Blocks merge if breaking changes detected

**3. Performance:**
- PR check: 30-60 seconds (incremental)
- Baseline lookup: O(1) via HashMap, not linear search
- Cost: Free (within Bitbucket's 500 min/month tier)

### Why Snapshot is Required

**Comparison needs "before" state:**
```
Baseline (6 months ago): userName field exists
Current PR: fullName field exists
→ Detects rename breaking change ✅
```

**Without snapshot:** No reference point to compare against - can't detect what changed from what.

**Efficiency:** Parse codebase once (baseline generation), reuse forever. Alternative would be parsing git history on every check (90x slower).

### Technical Implementation
- **Tool:** ArchUnit + Custom Git-aware checker
- **Storage:** JSON baseline in `src/test/resources/baseline/`
- **CI/CD:** Bitbucket Pipelines (ready-to-use YAML provided)
- **Maintenance:** Baseline auto-updates when intentional changes approved

### Benefits
✅ Zero production serialization incidents  
✅ Fast feedback (30-60 sec per PR)  
✅ Clear error messages with fix suggestions  
✅ No manual review needed  
✅ No cost (free tier sufficient)

### Deliverables Ready
- Complete implementation code
- Bitbucket Pipelines configuration
- Sample project demonstrating workflow
- Documentation

**Recommendation:** Implement immediately. Low effort, high impact protection.


private String maskSensitiveData(String json) {
    // Mask any account number that follows the pattern PREFIX-NUMBERS
    return json.replaceAll(
        "(\"\\w*AccountNumber\"\\s*:\\s*\"[A-Z]+-?)(\\d+)(\")",
        "$1*****$3"
    );
}
