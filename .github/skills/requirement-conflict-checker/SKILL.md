# Skill: Requirement Conflict Checker

## Purpose

Detect and summarize requirement conflicts across specifications in this repository, and within a single spec, providing actionable resolution guidance tied to sources and priorities. Also detect duplicate requirements across specs and within a spec, de-duplicate findings, and produce a separate duplicate-requirements report.

## Scope

- Repository: AI-Talent-Analyst-System-Documentation
- Primary inputs: Feature specs under `001-...` to `009-...`, `Template/spec-template.md`, and `.github/copilot-instructions.md` for predefined decisions.
- Output: A human-readable conflict report with links to source files/lines, categorized by conflict type, severity, and suggested fixes, plus a duplicate-requirements report.

## When to Use

- Before starting a new feature or changing shared entities/flows.
- Prior to release cut when reconciling specs.
- After updates to predefined decisions or cross-cutting features (RBAC, God Mode, multi-tenant, retention).

## Inputs

- Files to scan (default):
  - All `*/spec.md` under numbered feature folders.
  - `Template/spec-template.md` (structure, baseline language cues).
  - `.github/copilot-instructions.md` (authoritative conventions and predefined decisions).
- Optional filters:
  - Feature subset (e.g., `001,004,006`).
  - Conflict types to focus on (e.g., `RBAC`, `DataModel`).

## Outputs

- Structured Markdown conflict report including:
  - Summary (counts by conflict type, severity, files affected)
  - Detailed findings (each conflict with: type, severity, impacted features/entities, source links, rationale, and resolution proposals)
  - Appendix (normalized requirement index)
- Duplicate-requirements report including:
  - Summary (counts by duplicate type, files affected)
  - Detailed duplicates (each duplicate with: scope, canonical requirement, sources, and recommended consolidation)

## Conflict Taxonomy

1. Direct Contradiction
   - Opposing assertions for the same subject ("mandatory" vs "optional"; "must" vs "cannot").
2. Priority Clash
   - Same capability flagged different priorities across specs (e.g., P1 vs P2) creating delivery ambiguity.
3. Data Model Inconsistency
   - Different names/types/constraints for the same entity/field (e.g., `Candidate.status` enums; retention flags; versioning keys).
4. RBAC/Permissions Conflict
   - Permissions that overlap or contradict (e.g., default groups immutable vs group edit stories; enforcement location inconsistencies).
5. Flow/State Conflict
   - Divergent process steps or forbidden transitions vs scenarios (e.g., Feature 003 state transitions vs future enhancements; mandatory resume upload vs bypass cases).
6. Multi-Tenant Boundary Leak
   - Actions crossing company boundaries or God Mode behavior overriding tenant isolation without required audit.
7. Retention/Compliance Mismatch
   - Data retention, deletion rules, audit requirements inconsistent across specs.
8. Versioning Collision
   - Job Description versioning vs other specs assuming single JD; how references resolve over time.
9. Terminology Drift
   - Same concept, different names ("Company Admin" vs "Tenant Admin"), causing implementer ambiguity.
10. Duplicate Requirement

- The same requirement is stated multiple times across specs or within a single spec (including Clarifications/Assumptions vs FRs), causing noise and inflated counts.

## Detection Heuristics

- Linguistic cues:
  - Strong modal verbs: must|shall|required|cannot|forbidden|mandatory|optional|may|should
  - Priority markers: P1|P2|P3
  - RBAC artifacts: permission strings with `resource.action`, "group", "role", "God Mode", "impersonation"
  - Multi-tenant cues: company, tenant, cross-tenant, isolation, scope
  - State cues: status, transition, history, audit
  - Versioning cues: version, immutable, reference, snapshot
- Normalization keys for requirements:
  - FeatureId, Section (Requirements/User Stories/Acceptance), Priority, Subject (Entity/Resource), Action, Constraint, Scope (Tenant/System), Enforcement Point (API/Controller/UI), SourcePath, SourceLine(s), RawText
- Duplicate detection signals:
  - Exact-text matches after normalization (case/whitespace/punctuation)
  - Near-duplicate semantic matches for the same Subject+Action+Constraint
  - Duplicates across sections (Clarifications/Assumptions vs FRs)
- Conflict scoring (severity):
  - High: Direct contradiction on P1 or predefined decision
  - Medium: Flow or RBAC inconsistency impacting enforcement/audit
  - Low: Terminology drift or P2 preference differences

## Workflow

1. Gather Sources
   - Load all default input files unless a filter is provided.
2. Extract Requirements
   - Parse headers and sections: Metadata, User Scenarios, Acceptance Criteria, Requirements & Entities, Success Metrics.
   - Capture all statements with modal verbs, priorities, permission strings, statuses, data fields.
3. Normalize
   - Map each statement to the normalization keys listed above.
   - Derive `Subject` using entity mentions (Candidate, Job, Interview, Group, Permission, Company, Subscription, JD Version).
4. Index Cross-Refs
   - Group by Subject, then by Action/Constraint.
   - Collect all entries that touch predefined decisions from `.github/copilot-instructions.md` (treat as authoritative).
5. Detect Conflicts
   - Direct contradictions: same Subject+Action with opposing Constraint values (mandatory vs optional; allow vs forbid).
   - Priority clashes: same Subject+Action with different Priority across features.
   - Data model: same field with differing type/domain/constraints.
   - RBAC: mismatched permission sets or enforcement points.
   - Flow/state: invalid transitions vs acceptance scenarios; mutually exclusive preconditions.
   - Multi-tenant: operations lacking tenant scope or auditing when impersonating.
   - Retention/compliance: deletion rules vs indefinite retention.
   - Versioning: references assuming single JD vs explicit version snapshots.
6. Detect Duplicates
   - Group requirements by Subject+Action+Constraint and by normalized text.
   - Flag exact duplicates and near-duplicates across different sections or files.
   - If duplicates are found, remove duplicate entries from the conflict report and normalized index (keep a single canonical entry).
7. Rate Severity
   - Apply severity scoring; escalate when predefined decisions are contradicted.
8. Recommend Resolutions
   - Prefer alignment to predefined decisions; otherwise, propose a single source of truth and minimal spec edits.
9. Produce Report
   - Use the Report Template below. Include source links to exact files and lines where possible.
10. Produce Duplicate Report

- Generate REQUIREMENT_DUPLICATES.md listing duplicates removed from the conflict report and where they occur.

## Report Template

- Summary
  - Total conflicts: N
  - By type: {Direct: n1, Priority: n2, DataModel: n3, RBAC: n4, Flow: n5, Tenant: n6, Retention: n7, Versioning: n8, Terminology: n9}
  - Affected features: [001, 002, ...]

- Conflicts
  1. [Type] [Severity]
     - Subject: <Entity/Resource>
     - Description: <Concise contradiction>
     - Impacted: <Features>
     - Sources:
       - <path>(#Lstart-Lend)
       - <path>(#Lstart-Lend)
     - Recommendation: <Actionable, minimal-change proposal>

- Appendix: Normalized Requirements Index
  - <Feature> — <Subject> — <Action> — <Constraint> — <Priority> — <Source link>

## Duplicate Report Template (REQUIREMENT_DUPLICATES.md)

- Summary
  - Total duplicates: N
  - By scope: {WithinSpec: n1, AcrossSpecs: n2}
  - Affected files: [path1, path2, ...]

- Duplicates
  1.  [Scope: WithinSpec|AcrossSpecs]
      - Canonical Requirement: <Normalized requirement>
      - Sources:
        - <path>(#Lstart-Lend)
        - <path>(#Lstart-Lend)
      - Recommendation: <Consolidate into canonical FR or move non-FR restatements to cross-references>

## Usage Instructions (for this assistant)

- Read this SKILL.md; then scan the default inputs unless the user supplies filters.
- Generate the conflict report using the template above with repository-relative links.
- Detect and remove duplicates from the conflict report/index and generate REQUIREMENT_DUPLICATES.md.
- If confidence is low for a suspected conflict, mark it as "Potential" and include rationale.
- Always cross-check against `.github/copilot-instructions.md` and treat its Predefined Design Decisions as authoritative.

## Examples (repository-specific)

- Direct Contradiction (Predefined Decision vs Spec)
  - Decision: Resume upload is mandatory before interview start; failure terminates interview (Feature 001).
  - If any spec states resume upload is optional or skippable → High severity.
- RBAC Conflict
  - System groups (Super Admin, Company Admin) cannot be deleted (Feature 004). Any spec allowing deletion → High severity.
- Tenant Boundary
  - Impersonation (God Mode) must show banner and log all actions (Feature 002). Any spec omitting audit logging → Medium/High severity.
- Versioning
  - JD Versioning (Feature 009): specs assuming mutable JD in historical records without version snapshot link → Medium severity.

## Limitations

- Heuristic and language-based; not an executable validator.
- Requires careful judgment for "should" vs "must" interpretations.
- Line-level linking depends on stable file versions; verify after large edits.
