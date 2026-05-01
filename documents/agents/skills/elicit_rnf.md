# Skill: elicit_rnf - Non-Functional Requirements Specialist

This skill focuses on eliciting and documenting measurable non-functional requirements (RNFs) based on the ISO 25010 standard, ensuring they are quantifiable and aligned with project constraints.

## When to use
Use this skill when the user requests Non-Functional Requirements (RNFs) and the `specs/00_problem.md` and/or `documents/01_functional_requirements.md` files already exist.

## Required Input
- `specs/00_problem.md` (Mandatory)
- `documents/01_functional_requirements.md` (Optional, but highly recommended for context)

## Steps to Follow

### 1. Analysis phase
- Read stakeholders, critical flows, and failure modes from the problem specification.
- Review existing functional requirements to identify cross-cutting performance, security, or reliability needs.

### 2. ISO 25010 Mapping
Map the critical flows and system needs to the 8 attributes of the ISO 25010 standard:
1. **Functional Suitability**
2. **Performance Efficiency**
3. **Compatibility**
4. **Usability**
5. **Reliability**
6. **Security**
7. **Maintainability**
8. **Portability**

### 3. RNF Generation
For each attribute, propose 1 to 3 measurable RNFs.
- **Strict Rule**: No aspirational requirements (e.g., "The system must be reliable" is forbidden).
- Every RNF must have a measurable **SLI (Service Level Indicator)** with a unit and a time window.
- Assign a **MoSCoW priority** (Must have, Should have, Could have, Won't have).

### 4. Documentation
List assumptions and measurement sources for each requirement.

## Expected Output Structure
The output should be the content for `documents/02_non_functional_requirements.md`:

### 📌 ISO 25010 Attributes Analysis

#### [Attribute Name]
- **RNF-NN**: [Clear description]
- **Priority**: [MoSCoW]
- **Rationale**: [Why this is needed based on problem/stakeholders]

(Repeat for all 8 attributes)

### 📊 Measurability Matrix

| ID | Attribute | SLI (Indicator) | SLO (Objective) | Measurement Source | Priority |
|:---|:---|:---|:---|:---|:---|
| RNF-01 | Performance | Latency per 100k rows | < 30 minutes | CloudWatch Logs | Must |

### 🛠️ Assumptions & Prerequisites
- (List any technical assumptions or requirements for these measurements to be valid)

## Acceptance Criteria
- All 8 ISO 25010 attributes must be covered.
- Every RNF must include a unit and a measurement window.
- Zero "aspirational" language; everything must be testable.
