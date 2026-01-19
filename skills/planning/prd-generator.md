---
name: prd-generator
description: |
  Generate comprehensive Product Requirements Documents from project descriptions.
  Creates structured PRDs with features, user flows, and acceptance criteria.
---

# PRD Generator Skill

## Purpose
Create comprehensive Product Requirements Documents that clearly define what to build.

## PRD Template

```markdown
# Product Requirements Document
## [Product Name]

**Version:** 1.0
**Author:** Product Manager Agent
**Date:** [Date]
**Status:** Draft | Review | Approved

---

## 1. Executive Summary

### 1.1 Product Overview
[2-3 sentences describing what this product is and who it's for]

### 1.2 Problem Statement
[What problem does this solve? Why does it matter?]

### 1.3 Proposed Solution
[High-level description of the solution]

---

## 2. Goals & Success Metrics

### 2.1 Business Goals
- [ ] Goal 1: [Description] - Metric: [How to measure]
- [ ] Goal 2: [Description] - Metric: [How to measure]

### 2.2 User Goals
- [ ] Goal 1: [What users want to achieve]
- [ ] Goal 2: [What users want to achieve]

### 2.3 Key Performance Indicators (KPIs)
| KPI | Target | Current | Measurement |
|-----|--------|---------|-------------|
| User signups | 1000/month | 0 | Analytics |
| Task completion | 80% | N/A | User tracking |

---

## 3. User Personas

### 3.1 Primary Persona: [Name]
- **Role:** [Job title/role]
- **Goals:** [What they want to achieve]
- **Pain Points:** [Current frustrations]
- **Tech Savviness:** [Low/Medium/High]

### 3.2 Secondary Persona: [Name]
[Similar structure]

---

## 4. Features & Requirements

### 4.1 MVP Features (Must Have)

#### Feature 1: [Name]
- **Description:** [What it does]
- **User Story:** As a [user], I want to [action] so that [benefit]
- **Acceptance Criteria:**
  - [ ] Criterion 1
  - [ ] Criterion 2
- **Priority:** P0
- **Estimated Effort:** [T-shirt size: S/M/L/XL]

#### Feature 2: [Name]
[Same structure]

### 4.2 Post-MVP Features (Nice to Have)

#### Feature 3: [Name]
- **Priority:** P1/P2
[Abbreviated details]

### 4.3 Out of Scope
- [Feature/capability explicitly not included]
- [Feature/capability explicitly not included]

---

## 5. User Flows

### 5.1 Core Flow: [Name]
```
[Step 1] → [Step 2] → [Step 3] → [Success]
              ↓
         [Error handling]
```

**Steps:**
1. User does X
2. System responds with Y
3. User completes Z

### 5.2 Secondary Flow: [Name]
[Similar structure]

---

## 6. Non-Functional Requirements

### 6.1 Performance
- Page load time: < 3 seconds
- API response time: < 500ms
- Support 1000 concurrent users

### 6.2 Security
- User authentication required
- Data encryption at rest and in transit
- OWASP Top 10 compliance

### 6.3 Accessibility
- WCAG 2.1 AA compliance
- Keyboard navigation support
- Screen reader compatible

### 6.4 Compatibility
- Browsers: Chrome, Firefox, Safari, Edge (latest 2 versions)
- Mobile: iOS 14+, Android 10+
- Responsive: 320px to 1920px

---

## 7. Constraints & Assumptions

### 7.1 Constraints
- Budget: [If applicable]
- Timeline: [If applicable]
- Technical: [Existing systems to integrate]

### 7.2 Assumptions
- Users have internet access
- Users have modern browsers
- [Other assumptions]

### 7.3 Dependencies
- [External service or API]
- [Third-party integration]

---

## 8. Open Questions
- [ ] Question 1: [Details]
- [ ] Question 2: [Details]

---

## 9. Appendix

### 9.1 Glossary
| Term | Definition |
|------|------------|
| [Term] | [Definition] |

### 9.2 References
- [Link to design mockups]
- [Link to competitor analysis]
- [Link to user research]
```

## PRD Generation Process

1. **Gather Requirements**: Ask clarifying questions
2. **Identify Users**: Define who will use this
3. **Define MVP**: Focus on essential features
4. **Write User Stories**: Clear, testable stories
5. **Define Success**: Measurable outcomes
6. **Document Constraints**: Known limitations
7. **Review & Iterate**: Get feedback

## Quality Checklist

- [ ] Problem clearly stated
- [ ] Target users defined
- [ ] Features are prioritized
- [ ] Acceptance criteria are testable
- [ ] Success metrics are measurable
- [ ] Constraints are documented
- [ ] Open questions listed
