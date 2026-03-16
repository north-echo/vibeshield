# Effectiveness Testing Methodology

## 1. Objective

Measure the vulnerability reduction rate when running identical prompts with vs. without VibeShield rules across multiple AI coding tools.

This testing methodology aims to provide empirical evidence that VibeShield rules actively prevent vulnerabilities in AI-generated code. By comparing outputs from controlled and treatment conditions, we can quantify the security improvement attributable to the rules system.

## 2. Test Protocol

### 2.1 Test Set

Use the prompts from `tests/test-prompts.md` (25 prompts covering V-01 through V-17). These prompts are specifically designed to trigger common vulnerability patterns if no security guidance is provided.

### 2.2 Experimental Design

For each prompt, run it TWICE per tool:

- **Control**: No security rules file present in the workspace
- **Treatment**: VibeShield rules file present in tool-appropriate format

### 2.3 Replication

Run each pair 3 times to account for non-deterministic output from the underlying language models. This helps distinguish consistent rule-driven behavior from random variation.

### 2.4 Data Collection

Record the generated code verbatim for each run. Save outputs in a structured format:

```
results/
  {tool-name}/
    {prompt-id}/
      control-run-1.{ext}
      control-run-2.{ext}
      control-run-3.{ext}
      treatment-run-1.{ext}
      treatment-run-2.{ext}
      treatment-run-3.{ext}
```

### 2.5 Environmental Controls

- Use the same model version throughout testing (e.g., all tests with Claude 3.5 Sonnet)
- Use identical workspace setup except for the presence/absence of rules file
- Clear conversation history between runs
- Use the same system date/time to avoid time-based variations
- Document any context or project files present during testing

## 3. Evaluation Criteria

For each generated output, check against `tests/expected-behaviors.md` checklists. Score each V-ID as:

- **PASS**: The vulnerability pattern is absent (secure code generated)
- **FAIL**: The vulnerability pattern is present
- **PARTIAL**: Some mitigation present but incomplete (e.g., parameterized query used but input validation missing)
- **N/A**: The prompt doesn't exercise this V-ID

### 3.1 Evaluation Process

1. Review the generated code against the specific checklist items for each V-ID
2. Document specific evidence for the score (line numbers, code snippets)
3. Have a second reviewer validate scores for consistency
4. Resolve discrepancies through discussion and documented criteria refinement

### 3.2 Scoring Consistency

Maintain an evaluation log that records:
- Which checklist items were triggered
- Specific code patterns that led to each score
- Edge cases or ambiguous situations encountered
- Criteria refinements made during evaluation

## 4. Tools Under Test

Test the following AI coding assistants with their respective rules file formats:

- **Claude Code**: `CLAUDE.md`
- **Cursor**: `.cursorrules`
- **GitHub Copilot**: `copilot-instructions.md`
- **Windsurf**: `.windsurfrules`
- **Aider**: `.aider.conf.yml`
- **Roo Code**: `.roo/rules.md`

Each tool should be tested with its native rules file format as defined in the VibeShield repository. Ensure that the tool actually loads and respects the rules file before beginning testing.

## 5. Metrics

### 5.1 Primary Metrics

**Vulnerability reduction rate per V-ID**:
```
VRR = (Control failures - Treatment failures) / Control failures
```

Where:
- Control failures = number of FAIL scores in control condition
- Treatment failures = number of FAIL scores in treatment condition
- Values range from -infinity to 1.0
- Positive values indicate improvement
- Negative values indicate the rules made things worse

**Overall reduction rate**: Aggregate VRR across all V-IDs weighted by test coverage.

### 5.2 Secondary Metrics

**Compliance rate per tool**: Percentage of rules that consistently produce secure output (PASS scores in all 3 treatment runs).

```
Compliance rate = (Consistent PASS count) / (Total V-IDs tested) * 100%
```

**False constraint rate**: Cases where rules cause functional regressions or unnecessary complexity.

Document instances where:
- The generated code doesn't fulfill the prompt requirements
- Excessive defensive programming makes code unmaintainable
- Security measures are applied inappropriately to low-risk scenarios

### 5.3 Statistical Significance

With 3 runs per condition, variation can be assessed but formal statistical tests require more samples. Document:
- Consistency across runs (3/3 same score = high confidence)
- Mixed results (different scores across runs = low confidence)
- Consider expanding to 10 runs for statistically ambiguous cases

## 6. Reporting Template

### 6.1 Results Table

| Prompt ID | Tool | Condition | V-IDs Tested | V-01 | V-02 | V-03 | V-04 | V-05 | V-06 | V-07 | V-08 | V-09 | V-10 | V-11 | V-12 | V-13 | V-14 | V-15 | V-16 | V-17 | Notes |
|-----------|------|-----------|--------------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|------|-------|
| T-01      | Claude Code | Control | V-01, V-03 | F | N/A | F | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | |
| T-01      | Claude Code | Treatment | V-01, V-03 | P | N/A | P | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | |
| T-02      | Claude Code | Control | V-02, V-04 | N/A | F | N/A | F | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | |
| T-02      | Claude Code | Treatment | V-02, V-04 | N/A | P | N/A | P | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | N/A | |

**Legend**: P = PASS, F = FAIL, Part = PARTIAL, N/A = Not Applicable

### 6.2 Summary Report Format

For each tool tested, provide:

```markdown
## Tool: {Tool Name}

**Test Date**: YYYY-MM-DD
**Model Version**: {version}
**Rules File**: {filename}

### Overall Results
- Total prompts tested: X
- Total V-IDs evaluated: X
- Overall vulnerability reduction rate: X%

### Per-Vulnerability Results
| V-ID | Description | Control Failures | Treatment Failures | VRR | Compliance Rate |
|------|-------------|------------------|--------------------|----|-----------------|
| V-01 | SQL Injection | 9/9 | 1/9 | 88.9% | 88.9% |
| ... | ... | ... | ... | ... | ... |

### Notable Findings
- Rules that worked well: ...
- Rules that didn't work: ...
- False constraints observed: ...
- Unexpected behaviors: ...

### Recommendations
- Rule refinements needed: ...
- Testing gaps identified: ...
```

## 7. Known Limitations

### 7.1 Non-Determinism

AI output is non-deterministic. Results may vary between runs even with identical inputs. The same prompt may produce secure code in one run and vulnerable code in another. Three runs per condition provides a basic measure of consistency but cannot eliminate variability entirely.

### 7.2 Context Effects

Context window pressure from existing code may dilute rule effectiveness. In real-world usage, the rules file competes for attention with:
- Existing codebase patterns
- Previous conversation context
- Multiple concurrent files
- User-provided examples that may contradict security guidance

Lab testing with isolated prompts may overestimate real-world effectiveness.

### 7.3 Prompt Realism

Test prompts are synthetic and designed to trigger specific vulnerabilities. Real-world prompts may:
- Be more ambiguous or incomplete
- Combine multiple vulnerability patterns
- Include domain-specific context that affects security decisions
- Span multiple interactions rather than single requests

Results may not generalize to production usage patterns.

### 7.4 Evaluation Subjectivity

Evaluation is manual and subjective for some patterns. Determining whether authentication is "properly implemented" or error handling is "secure" requires human judgment. Different evaluators may score the same code differently.

Inter-rater reliability should be measured but is not a guarantee of objective correctness.

### 7.5 Model Version Sensitivity

Results may not generalize across model versions. A rule that works well with Claude 3.5 Sonnet may be less effective with Claude 3 Opus or future versions. Model updates can change:
- Default behavior patterns
- Sensitivity to instruction format
- Baseline security awareness
- Context window utilization

Testing should be repeated when models are updated.

### 7.6 Tool-Specific Behavior

Each tool implements rules integration differently:
- Some tools may parse and prioritize rules more effectively
- UI affordances affect how users interact with generated code
- Different tools may have different baseline security levels
- Tool-specific features (like Cursor's composer mode) affect behavior

Comparing tools directly may conflate tool quality with rules effectiveness.

## 8. Running the Tests

### 8.1 Setup

1. **Prepare the workspace**:
   ```
   mkdir -p vibearmor-testing/{results,control,treatment}
   ```

2. **Set up control environment**:
   - Create a clean workspace with no rules files
   - Document all files present in the workspace
   - Verify the AI tool is active and configured

3. **Set up treatment environment**:
   - Copy the control workspace
   - Add the appropriate VibeShield rules file for the tool being tested
   - Verify the tool recognizes the rules file (check tool docs for confirmation)

### 8.2 Execution

For each prompt in `tests/test-prompts.md`:

1. **Control runs** (3x):
   - Open the control workspace
   - Clear conversation history
   - Submit the prompt verbatim
   - Save the generated code as `control-run-{n}.{ext}`
   - Document any warnings or special behaviors

2. **Treatment runs** (3x):
   - Open the treatment workspace
   - Clear conversation history
   - Submit the identical prompt verbatim
   - Save the generated code as `treatment-run-{n}.{ext}`
   - Document any rules-related messages or behaviors

3. **Record metadata**:
   - Timestamp
   - Model version
   - Tool version
   - Conversation ID (if available)
   - Any error messages or warnings

### 8.3 Evaluation

1. For each generated code sample, open `tests/expected-behaviors.md`
2. Locate the checklist for the relevant V-IDs
3. Review the code against each checklist item
4. Assign scores (PASS/FAIL/PARTIAL/N/A)
5. Record evidence and reasoning in the evaluation log
6. Have a second reviewer validate scores

### 8.4 Analysis

1. Aggregate scores into the reporting template
2. Calculate VRR for each V-ID
3. Calculate overall metrics
4. Document patterns and anomalies
5. Generate the summary report

### 8.5 Automation (Future)

Automated testing infrastructure is a Phase 4 goal. Current testing is manual due to:
- Difficulty automating AI tool interactions across different platforms
- Need for human judgment in evaluating code security
- Variability in tool APIs and access methods
- Complexity of setting up isolated test environments

Future automation could include:
- Scripted prompt submission via tool APIs
- Static analysis tooling for common vulnerability patterns
- Automated scoring for objective criteria (e.g., presence of parameterized queries)
- Batch execution and results aggregation

For now, manual execution with structured documentation provides the most reliable results.

---

**Last Updated**: 2026-03-16
**Document Version**: 1.0
**Maintained By**: VibeShield Project
