# Hackathon 2025 - Technical Feedback Requirements

## Most Valuable Technical Feedback Award

Filing issues in this repo is **required** for award eligibility. We're looking for quality over quantity.

## Timeline

- **Coding Phase:** November 27, 2025 (3:30 AM UTC) → December 20, 2025 (6:29 PM UTC)
- **Issue Filing:** Anytime during the hackathon (don't wait until the last day)
- **Judging:** Week of January 5, 2025
- **Winners Announced:** January 12, 2025

## Requirements for Award Eligibility

✅ **Include your HackerEarth username** - Must be in the issue for attribution  
✅ **Use the issue template** - Fill out all relevant sections completely  
✅ **File as you encounter problems** - Real-time feedback is more valuable than end-of-hackathon dumps  
✅ **Provide reproduction steps** - We need to be able to recreate the issue  
✅ **Include environment details** - Version, deployment type, backing store, OS, framework

## What Makes Feedback Valuable

### High-Value Feedback ✅

- **Detailed reproduction cases** - "Here's exactly how to trigger this issue with these specific steps"
- **Performance data with actual numbers** - "Query took 47ms with 1M vectors on PostgreSQL backing store"
- **Integration-specific pain points** - "LangChain's VectorStore interface expects X but CyborgDB returns Y, breaking Z"
- **Compliance edge cases** - "HIPAA audit logs require X but current implementation only accomodates Y"
- **Edge cases we missed** - "Concurrent upserts on the same collection cause race condition under load"
- **Documentation gaps with examples** - "Error message says 'invalid configuration' but doesn't specify which field or valid values"

### Low-Value Feedback ❌

- "It doesn't work" with no details
- Questions that belong in the support channel (setup help, API usage questions)
- Feature requests without explaining the problem you're trying to solve
- Issues without HackerEarth username (can't attribute for award)
- Duplicate issues (search first!)
- Vague descriptions without reproduction steps

## Judging Criteria

Issues are evaluated on:

1. **Impact/Severity** - Does this block production deployment? Affect performance claims? Create security risks?
2. **Quality of reproduction** - Can we recreate the issue with the steps provided?
3. **Novelty** - Did you find something nobody else discovered?
4. **Clarity and detail** - Is the issue well-documented with logs, error messages, code snippets?
5. **Technical depth** - Does this expose underlying architectural issues or just surface-level bugs?

**One excellent issue beats ten vague ones.** We'd rather see 3-5 deeply investigated problems than 50 "this broke" reports.

## What We Want to Hear About

### Performance Issues
- Latency measurements that don't match our sub-10ms claims
- Throughput bottlenecks at scale
- Memory consumption problems
- Query optimization opportunities

### Integration Problems
- LangChain integration rough edges
- LlamaIndex compatibility issues
- Framework-specific incompatibilities
- API design friction points

### Deployment Challenges
- Kubernetes configuration problems
- Serverless deployment limitations
- Docker image issues
- Scaling challenges

### Industry-Specific Issues
- Healthcare, fintech, or enterprise requirements we didn't account for
- Compliance blockers for production deployment
- Regulatory gaps in specific verticals
- Security requirements unique to your use case

### Documentation Issues
- Confusing error messages
- Missing code examples
- Unclear API documentation
- Setup instructions that don't work

### Bugs & Crashes
- Reproducible crashes
- Data corruption issues
- Race conditions
- Memory leaks

## Tips for Filing Great Issues

**Search first** - Check if someone already reported it. If they did, add your findings to that issue rather than creating a duplicate.

**One issue per problem** - Don't combine multiple unrelated issues. File separately so we can track and fix them independently.

**Show, don't just tell** - Include:
- Error messages (full stack traces)
- Code snippets that reproduce the issue
- Screenshots if relevant
- Performance measurements
- Configuration files

**Test in isolation** - Verify the issue isn't caused by your application code or environment before filing.

**Follow up** - If we ask clarifying questions, respond promptly. Issues that go stale won't be weighted as heavily.

## Example of Excellent Feedback

**Title:** `[FEEDBACK] LangChain integration: metadata filtering breaks with nested JSON fields`

**Category:** Integration (LangChain/LlamaIndex/other frameworks)

**Description:** 
CyborgDB's LangChain VectorStore implementation doesn't support nested JSON metadata filtering, which is a standard feature in LangChain's VectorStore interface. When using nested metadata like `{"patient": {"mrn": "12345", "dob": "1980-01-01"}}`, queries with filters on nested fields return empty results instead of matching documents.

**Expected Behavior:**
LangChain's metadata filter syntax should work with nested fields:
```python
vectorstore.similarity_search(
    query="symptoms",
    filter={"patient.mrn": "12345"}
)
```

**Actual Behavior:**
- Flat metadata filters work correctly: `{"department": "cardiology"}` ✓
- Nested metadata filters return empty results: `{"patient.mrn": "12345"}` ✗
- No error message - silently fails with 0 results
- Flattening metadata manually works but breaks LangChain compatibility

**Reproduction Steps:**
1. Install CyborgDB LangChain integration: `pip install cyborgdb-langchain`
2. Initialize vectorstore with nested metadata:
```python
from cyborgdb_langchain import CyborgDBVectorStore

vectorstore = CyborgDBVectorStore(api_key="...", collection="patients")
vectorstore.add_texts(
    texts=["Patient presents with chest pain"],
    metadatas=[{"patient": {"mrn": "12345", "dob": "1980-01-01"}, "department": "cardiology"}]
)
```
3. Query with nested filter:
```python
results = vectorstore.similarity_search("chest pain", filter={"patient.mrn": "12345"})
print(len(results))  # Returns 0, should return 1
```
4. Query with flat filter works:
```python
results = vectorstore.similarity_search("chest pain", filter={"department": "cardiology"})
print(len(results))  # Returns 1 correctly
```

**Environment:**
- CyborgDB version: 0.14.0
- Deployment: Service (REST API)
- Backing store: PostgreSQL 15.3
- OS: macOS 14.1
- Framework: LangChain 0.1.0
- Python: 3.11.5

**Performance Data:**
Not applicable - functional issue, not performance.

**Impact:** 
This blocks our healthcare RAG implementation because we need to filter patient records by nested identifiers (MRN, facility ID, encounter ID) to maintain HIPAA compliance. Flattening metadata into flat keys like `patient_mrn` breaks compatibility with our existing LangChain codebase that uses nested structures.

**Additional Context:**
- Tested with other LangChain vectorstores (Pinecone, Weaviate) - nested filtering works correctly
- CyborgDB's REST API documentation doesn't mention nested metadata limitations
- Workaround: Manually flatten metadata before insert, but this requires forking LangChain code
- Error logs show no warnings about unsupported filter syntax

**Suggested Fix:**
Either:
1. Support dot notation for nested fields: `{"patient.mrn": "12345"}`
2. Document the limitation clearly and provide migration guide for nested metadata
3. Add validation that throws an error when nested filters are used instead of silently returning empty results

---

**This issue would score high because:**
- Includes HackerEarth username
- Clear integration-specific problem with production impact
- Minimal reproduction code that's easy to run
- Compares behavior to other vectorstores (establishes expected behavior)
- Explains HIPAA compliance blocker (real-world impact)
- Suggests multiple solution approaches
- Notes documentation gap (silent failure)

## Questions?

- **Setup/usage questions:** Use the [support channel](https://discord.gg/jMF7kdNm)
- **Documentation feedback:** File an issue with the `documentation` label
- **Clarifications on this guide:** Comment on existing issues or ask in support

---

**Remember:** Your insights on where CyborgDB breaks help shape what production-ready encrypted vector search needs to handle. Honest technical feedback—even when it's critical—is exactly what we need.
