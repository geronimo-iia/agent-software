---
title: "Roadmap Documentation Standards"
summary: "Standards for creating and maintaining project roadmaps with phase-based development."
read_when:
  - Planning project roadmaps
  - Structuring development phases
  - Archiving completed work
status: draft
last_updated: "2026-07-16"
---

# Roadmap Documentation Standards

## Purpose

Roadmaps provide clear development direction through phased implementation plans. They balance current focus with historical context while maintaining actionable guidance.

## Structure

### Current Roadmap (`docs/roadmap.md`)

**Frontmatter**:
```yaml
---
title: "Project Roadmap"
summary: "Current and future implementation phases for [project]."
read_when:
  - Planning [project] development
  - Deciding what to implement next
  - Understanding scope and sequencing
status: draft
last_updated: "YYYY-MM-DD"
---
```

**Sections**:
1. **Current Status** — completed phases with versions
2. **Completed Phases** — links to archived documentation
3. **Active Phase** — current implementation focus
4. **Future Phases** — planned upcoming work
5. **Deferred Phases** — postponed with rationale
6. **Standards** — references to development standards

### Phase Documentation

Each phase should include:

**Goal**: Clear objective and success criteria
**Commands/Features**: What will be delivered
**Implementation**: Technical approach and architecture
**New modules**: Code organization changes
**Steps**: Ordered implementation tasks
**Exit criteria**: Measurable completion requirements

## Archive Structure

### When to Archive

Archive phases when:
- All exit criteria completed
- Version released and stable
- Implementation details finalized
- Focus shifts to next phase

### Archive Location

```
docs/
├── archive/
│   ├── phase-1-[name].md
│   ├── phase-2-[name].md
│   └── phase-N-[name].md
├── roadmap.md
└── [other-docs].md
```

### Archive Content

**Frontmatter**:
```yaml
---
title: "Phase N — [Name]"
summary: "Completed implementation of [feature/capability]."
status: archived
archived_date: "YYYY-MM-DD"
archived_reason: "Phase completed successfully, functionality integrated into [project] [version]"
---
```

**Sections**:
1. **Goal** — original objective
2. **Commands/Features Implemented** — what was delivered
3. **Key Features Delivered** — major capabilities
4. **Architecture** — technical implementation
5. **Release Artifacts** — version, tests, documentation
6. **Impact** — significance and outcomes

## Content Guidelines

### Focus on Current Work

- Roadmap shows current + future phases only
- Completed work moves to archive with full details
- Archive preserves implementation decisions and rationale
- Links connect roadmap to archived phases

### Phase Naming

- Use descriptive names: "Hub Validation", "Skill Management"
- Include version numbers in archive titles
- Maintain consistent numbering (1, 2, 3, 4.5, 5)
- Sub-phases use decimal notation (4.5, 4.6)

### Exit Criteria

- Use checkboxes for trackable progress
- Include measurable outcomes
- Reference specific files/commands
- Link to release processes

### Standards References

Always include references to:
- Project structure standards
- CI/CD standards  
- Commit message standards
- Versioning standards

## Examples

**Good phase structure**:
```markdown
## Phase 3 — Skill Management

**Goal**: Install and manage skills from registered hubs.

### Commands
[specific commands with examples]

### Implementation
[technical approach]

### Exit criteria
- [ ] Specific measurable outcome
- [ ] Another trackable requirement
```

**Good archive structure**:
```markdown
# Phase 3 — Skill Management ✅ COMPLETED

**Goal**: [original objective]

## Commands Implemented
[what was delivered]

## Impact
[significance and outcomes]
```

## Maintenance

### Regular Updates

- Update `last_updated` when making changes
- Move completed phases to archive promptly
- Keep current status section accurate
- Update version numbers and links

### Version Alignment

- Archive phases when versions are released
- Include version numbers in archive titles
- Reference specific releases in impact sections
- Maintain changelog alignment

This standard ensures roadmaps remain focused and actionable while preserving valuable implementation history.