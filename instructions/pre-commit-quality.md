---
description: Pre-Commit Quality Control Rules
globs:
alwaysApply: true
version: 1.0
encoding: UTF-8
---

# Pre-Commit Quality Control

<ai_meta>
  <rules>Before committing any implementation, run quality control using the best available agents. Validate usage, duplication, redundancy, interface compliance, security and performance. Block merge if criteria are not met.</rules>
  <format>UTF-8, LF, 2-space indent</format>
</ai_meta>

## Overview

This instruction ensures that any code committed to the repository has passed strict quality control.  
The checks are systematic, automated where possible, and must be validated before the commit is allowed.

<process_flow>

<step number="1" name="code_usage">

### Step 1: Code Usage
- Verify that all code introduced is actually referenced and used in the application.
- Detect any dead code (unused functions, unused imports, stale files).
- Flag as `TODO: unused_code` if found.

</step>

<step number="2" name="duplication">

### Step 2: Duplication
- Run duplication detection across the codebase.
- Highlight exact or near-duplicate blocks.
- Enforce single responsibility: no copy/paste implementations.
- Flag as `TODO: duplicate_code` if detected.

</step>

<step number="3" name="redundancy">

### Step 3: Redundancy
- Check for redundant logic that overlaps with existing modules.
- Detect over-engineering or features already provided elsewhere.
- Recommend consolidation where appropriate.

</step>

<step number="4" name="interfaces">

### Step 4: Interface & Contracts
- Ensure all public functions, APIs, and components respect defined interfaces/contracts.
- Verify typings, schemas, and props against DS or API definitions.
- Block commit if contract violations exist.

</step>

<step number="5" name="security">

### Step 5: Security
- Validate input sanitization and safe handling of user data.
- Ensure no secrets, tokens, or sensitive information are committed.
- Run security linters/scanners where applicable.
- Flag common vulnerabilities (SQL injection, XSS, CSRF, etc.).

</step>

<step number="6" name="performance">

### Step 6: Performance
- Check for unnecessary loops, N+1 queries, or inefficient patterns.
- Validate caching, memoization, and resource usage.
- Run benchmarks if critical codepaths are affected.
- Flag regressions compared to baseline metrics.

</step>

<step number="7" name="final_gate">

### Step 7: Final Gate
- All prior steps must pass.
- If any blocking issues remain → STOP, output TODOs, and prevent commit.
- If clean → allow commit.

</step>

</process_flow>

## Execution Summary

<final_checklist>
  <verify>
    - [ ] No unused code
    - [ ] No duplication
    - [ ] No redundant logic
    - [ ] All interfaces respected
    - [ ] No security vulnerabilities
    - [ ] Performance baseline preserved
  </verify>
</final_checklist>

<execution_order>
  1. Verify code usage
  2. Detect duplication
  3. Review redundancy
  4. Check interfaces
  5. Run security audit
  6. Evaluate performance
  7. Final gate decision
</execution_order>