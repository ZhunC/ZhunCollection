---
share: "True"
categories: Note
postType: post
title: A Cadence-centered Approach to Iterative Production-level Software Development
tags:
  - -sw_dev
  - -project_management
---
Date: Aug.25 2025

## 0. Prefacing Statement

This document outlines the full cycle of software development, and its iteration starting from when a functionality is requested to when a software is released, based on my personal learnings, experiences, and understanding. It is intended as a manual for personal and group projects alike. 

This development cycle is summarized from my personal experiences in multiple R&D and product-oriented projects. It very much agrees in philosophy with several industrial standard models like Agile or the V-model. At the same time, my own experiences have led me to emphasize certain gating steps — such as formal proof-of-concept prototyping, static analysis, and output quality validation. These steps often emerge in practice with unexpected importance to the project, even when not explicitly outlined in some other frameworks. These represent my focus on bridging R&D exploration with consumer-safe production readiness.

**Regulatory Note**: In regulated industries (e.g., aviation, automotive safety, medical devices), specifications are required very early and must be formally controlled. Such domains mandate traceability from requirements through design to verification and certification. The cycle below is flexible but can be adapted to meet such compliance obligations.

---

## 1. Process Outline and Cadence

### Development Cycle Overview

In the following outline, I attempt to summarize a desired development cycle for production-ready, consumer-safe software. 

1. **Functionality Request**
2. **Proof-of-concept Prototyping**
3. **Technical Specification Definition**
4. **Production SW Implementation**
5. **Static Analysis**
6. **Unit Testing**
7. **Integration Testing**
8. **Peer Code Review**
9. **Production SW Use Case Testing and KPI Testing** 
10. **User Acceptance Testing**
11. **Release Review/Go-No-Go Decision**
12. **Post-Deployment Monitoring**

### Cadence Structure

- **New Feature Planning Cycle**: Steps 1–3 occur as fast as needed and possible  
- **New Feature Integration Request**: Step 3 communicates to step 4 every **3 weeks**  
- **Development & Testing Cycle**: Steps 4–9 occur in **weekly iterations**  
- **Deployment Cycle**: Steps 10–12 occur as needed based on release readiness and overall project timeline  
- **Feedback Loop**: Any issues discovered in steps 5–12 is fed back to step 3 (spec revision for architectural changes) or step 4, depending on if it is a design issue (goes to step 3) or an implementation issue (goes to step 4).  
- **Parallelism Note**: While steps are written sequentially for clarity, in practice the **pre-specification cycle, implementation cycle, and testing cycle often operate in parallel**. Prototypes, partial implementations, and early test stubs are evaluated while specifications are being refined, and test results continuously feed back into design.  
- **Team Ownership**: Team functionality and title varies from place to place, and for generality all teams are categorized into the following four categories with some example title marked out. These four categories will be used to identify whose major responsibility it is in each step:  
  - Stakeholder Owners: For example, Product Development, Market Research, etc.  
  - Architecture Owners: For example, R&D/Systems Engineering, etc.  
  - Implementation Owners: For example, Software/Firmware/Mechanical/Application Engineering, etc.  
  - Quality Owners: For example, Testing, Quality Assurance, etc.  
- **Budgeting Note**: Three financial checkpoints are embedded in this cycle to balance technical iteration with business oversight:  
  1. **Pre-Prototype Budget Estimate & Approval** – assess whether prototyping investment is justified.  
  2. **Pre-Specification Refined Estimate** – update cost projection after feasibility and prototype results.  
  3. **Pre-Release Cost Report** – confirm implementation and testing expenditures before batch release.  
  Unlike full Stage-Gate frameworks, which place budget and market checks at every decision point, this approach provides lighter oversight tailored for engineering teams, while still allowing alignment with Stage-Gate governance if required.

---

## 2. Process Task Definitions and Major Responsibility Teams

### Step 1: Functionality Request 

**Purpose**: Define new features or enhancements  

**Major Responsibility Team:** All Owners  

**Input**:
- Business needs and customer pain points  
- Stakeholder requirements and market analysis  
- R&D algorithmic enhancements and technical innovations  

**Tasks**:
- Gather business, legal, and technical requirements  
- Document R&D enhancement proposals and feasibility  
- Clarify acceptance criteria  
- Prepare preliminary budget estimate and seek approval  

**Output**:
- Business and legal requirements  
- Desired user use cases  
- Approved preliminary budget  

---

### Step 2: Proof-of-concept Prototyping

**Purpose**: Rapidly conduct software prototyping to further understand feasibility and project impact of each functionality request  

**Major Responsibility Team:** Architecture Owners  

**Input**:
- Business and legal requirements  
- Desired user use cases  
- Approved preliminary budget  

**Tasks**:
- Attempt to design and implement core functionalities  
- Assess technical feasibility and difficulty  
- Explore various implementation approaches  
- Estimate resources and timeline  
- Refine budget projection based on feasibility  

**Output**:
- Working prototype demonstrating core functionality  
- Technical feasibility or difficulty assessment  
- Implementation approach recommendations  
- Updated resource and timeline estimates  
- Refined budget estimate  

---

### Step 3: Technical Specification Definition

**Purpose**: Create detailed technical blueprint for production implementation  

**Major Responsibility Team:** Architecture Owners  

**Input**:
- Validated prototype and test results  
- Business requirements and desired user use cases  
- Refined budget estimate  

**Tasks**:
- Design system architecture, including its interface specifications and internal logics  
- Document security and performance requirements  
- Finalize specification and budget for approval  

**Output**:
- System architecture design / Control or UI Flow  
- Security and performance requirements  
- Approved specification and budget  

---

### Step 4: Production SW Implementation

**Purpose**: Build production-ready software according to specifications  

**Major Responsibility Team:** Implementation Owners  

**Input**:
- System architecture design / Control and UI Flow  
- Security and performance requirements  

**Tasks**:
- Implement production code following specifications  
- Create internal documentation and code comments  
- Follow coding standards and best practices  

**Output**:
- Production code implementing specified functionality  
- Internal documentation and code comments  

---

### Step 5: Static Analysis

**Purpose**: Automated code quality and security assessment  

**Major Responsibility Team:** Implementation Owners  

**Input**:
- Production code  

**Tasks**:
- Run automated code quality analysis and resolve reported problems  

**Output**:
- Production code  

---

### Step 6: Unit Testing

**Purpose**: Verify individual modules produce correct mathematical and functional outcomes  

**Major Responsibility Team:** Quality Owners  

**Input**:
- Production code  
- System architecture design / Control and UI Flow  

**Tasks**:
- Write and append comprehensive unit tests with >80% coverage  
- Test edge cases and create performance benchmarks  
- Validate regression scenarios  

**Output**:
- Unit test suite with coverage reports  
- Performance benchmarks  
- Test validation results  

---

### Step 7: Integration Testing

**Purpose**: Verify components work together correctly  

**Major Responsibility Team:** Quality Owners  

**Input**:
- Production code  
- System architecture design / Control and UI Flow  

**Tasks**:
- Test component interactions and data flow  
- Execute integration test scenarios  

**Output**:
- Integration test results  
- Component validation confirmation  

---

### Step 8: Peer Code Review

**Purpose**: Review weekly code commits and validate code quality, logic, and architectural compliance before internally releasing commits  

**Major Responsibility Team:** Architecture Owners, Quality Owners  

**Input**:
- Production code  
- Test results and static analysis reports  
- Control or UI Flow  

**Tasks**:
- Review code logic, quality, and architectural compliance  
- Document findings and approvals  

**Output**:
- Code review and internal release approvals  

---

### Step 9: Production SW Use Case Testing and KPI Testing

**Purpose**: Validate user use cases work as expected, and confirm quality within KPIs  

**Major Responsibility Team:** Quality Owners  

**Input**:
- Internally released code  
- Business requirements, Control and UI Flow  

**Tasks**:
- Execute and verify desired user use cases  
- Verify performance and KPI benchmarks  

**Output**:
- Use case testing results  
- KPI test results  

---

### Step 10: User Acceptance Testing

**Purpose**: Conduct company internal verifications that users are satisfied  

**Major Responsibility Team:** Stakeholder Owners  

**Input**:
- Internally released code  
- Business requirements and desired user use cases  

**Tasks**:
- Collect feedback and verify user use cases  

**Output**:
- Business requirement verification  
- User feedback  

---

### Step 11: Release Review/Go-No-Go Decision

**Purpose**: Final quality gate and deployment readiness assessment  

**Major Responsibility Team:** All Owners (Quality, Implementation, Architecture, Stakeholders)  

**Input**:
- All test and performance results  
- Internally released code  
- Cost and budget reports  

**Tasks**:
- Review quality and assess deployment risks  
- Verify readiness and make go/no-go decision  
- Prepare rollback plans  
- Review cost report before release  

**Output**:
- Go/No-Go decision documentation  
- Rollback plan  
- Final cost report  

---

### Step 12: Post-Deployment Monitoring

**Purpose**: Ensure software performs correctly in production and in market  

**Major Responsibility Team:** Quality Owners  

**Input**:
- Deployed system  
- Performance baselines  

**Tasks**:
- Review, root-cause, and properly address issues reported by customer service or monitoring tools  
- Feed learnings back into requirements and specifications  

**Output**:
- Issue reports  
- Continuous improvement updates  

