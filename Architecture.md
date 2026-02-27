# AI Testcase Generation Agent

# Technical Architecture Document (V1)

## 1. Document Purpose

This document defines the complete technical architecture for the
AI-powered Testcase Generation Agent designed as a Copilot-style PR
Assistant.

The system is responsible for:

-   

> Generating End-to-End Selenium automation tests (Selenide, Java)

-   
-   

> Automatically generating regression E2E tests for every P1/P2 defect

-   
-   

> Detecting and flagging impacted existing tests (human approval
> required)

-   
-   

> Computing Automation Stability Confidence Score

-   
-   

> Maintaining full bidirectional traceability:

-   

Requirement ↔ Code ↔ Test ↔ Defect ↔ Release

Environment Scope: QA only\
Selector Strategy: XPath\
Architecture Pattern: Hybrid (POM + Utilities)\
LLM Hosting: OpenAI API\
Traceability Engine: Neo4j Graph Database

# 2. Architectural Principles

1.  

> Requirements are the authoritative source of workflow intent.

2.  
3.  

> All generation must be traceable and deterministic.

4.  
5.  

> No hardcoded test data (must use dedicated TDM tool).

6.  
7.  

> XPath-only selector strategy.

8.  
9.  

> Hybrid Page Object Model + Utilities compliance.

10. 
11. 

> Human-in-the-loop for impacted existing tests.

12. 
13. 

> Mandatory regression generation for P1/P2 defects.

14. 
15. 

> Confidence score reflects automation stability, not AI certainty.

16. 
17. 

> QA environment only.

18. 
19. 

> No hallucination --- operate only on supplied context.

20. 

# 3. High-Level Architecture

## 3.1 Logical Architecture Overview

GitHub / Bitbucket PR

│

▼

PR Webhook Listener

│

▼

PR Diff Analyzer

│

├────────► Vector Database (Semantic Retrieval)

│

├────────► Neo4j Graph Database (Traceability)

│

▼

Workflow Orchestration Engine

│

▼

OpenAI Reasoning Layer

│

▼

Selenide Code Generation Engine

│

▼

PR Comment + Patch Suggestion + Impact Report

# 4. Core Components

## 4.1 PR Integration Layer

### Responsibilities

-   

> Listen to PR open/update events

-   
-   

> Extract:

-   -   

> Changed files

-   
-   

> Branch metadata

-   
-   

> Commit history

-   

```{=html}
<!-- -->
```
-   

> Trigger orchestration pipeline

-   
-   

> Post structured PR comments

-   
-   

> Suggest patch files (not auto-merge)

-   

### Constraints

-   

> PR-scoped execution only

-   
-   

> Incremental indexing only

-   
-   

> No full repository reprocessing

-   

## 4.2 Ingestion & Indexing Layer

### 4.2.1 Vector-Embedded Sources (V)

  -----------------------------------------------------------------------
  **Source**                         **Purpose**
  ---------------------------------- ------------------------------------
  Requirements (Primary)             Workflow intent

  Figma UI Specs                     Screen transitions

  Technical Documentation            Negative scenarios

  Defect Database (P1/P2)            Regression triggers

  Existing Test Assets               Overlap detection

  Release Notes                      Regression prioritization
  -----------------------------------------------------------------------

Granularity:

-   

> Requirement: Story + Acceptance Criteria

-   
-   

> UI: Screen + Component level

-   
-   

> Defect: Root cause summary

-   
-   

> Test: Method level

-   

### 4.2.2 Partial Vector (PV) -- Code Repository

Indexed Components:

-   

> UI routes

-   
-   

> Controllers

-   
-   

> Service entry points

-   
-   

> Validation logic

-   
-   

> Feature flags

-   
-   

> XPath locators

-   
-   

> PR branches + main branch + historical commits

-   

Purpose:

-   

> Impact detection

-   
-   

> Locator reuse detection

-   
-   

> DOM volatility analysis

-   
-   

> Stability scoring

-   

## 4.3 Knowledge Graph Layer (Neo4j)

### Node Types

-   

> Requirement

-   
-   

> AcceptanceCriteria

-   
-   

> UIScreen

-   
-   

> CodeComponent

-   
-   

> Commit

-   
-   

> PR

-   
-   

> TestCase

-   
-   

> Defect

-   
-   

> Release

-   

### Relationship Types

-   

> IMPLEMENTS (Code → Requirement)

-   
-   

> VALIDATES (Test → Requirement)

-   
-   

> FIXES (Commit → Defect)

-   
-   

> IMPACTS (PR → CodeComponent)

-   
-   

> REGRESSION_FOR (Test → Defect)

-   
-   

> INTRODUCED_IN (Feature → Release)

-   
-   

> COVERS (Test → Workflow)

-   

### Capabilities

-   

> PR impact traversal

-   
-   

> Requirement-to-code mapping

-   
-   

> Defect-rooted regression detection

-   
-   

> Release-based prioritization

-   
-   

> Traceability matrix generation

-   

# 5. Workflow Orchestration Engine

The orchestration engine controls deterministic reasoning before
invoking the LLM.

## Phase 1 -- PR Impact Analysis

-   

> Analyze changed files

-   
-   

> Identify impacted CodeComponents

-   
-   

> Traverse graph for:

-   -   

> Linked Requirements

-   
-   

> Linked Defects

-   
-   

> Linked Existing Tests

-   
-   

> Linked Releases

-   

## Phase 2 -- Requirement Grounding

-   

> Retrieve authoritative Requirement

-   
-   

> Parse Acceptance Criteria

-   
-   

> Extract workflow steps

-   
-   

> Detect ambiguity

-   

If ambiguity detected:

-   

> Trigger grooming session transcript summarization

-   
-   

> Enrich requirement understanding

-   

## Phase 3 -- UI Flow Reconstruction

Using structured Figma flows:

-   

> Map screen transitions

-   
-   

> Identify UI components

-   
-   

> Extract user actions

-   
-   

> Define expected UI states

-   
-   

> Construct E2E interaction path

-   

## Phase 4 -- Negative Scenario Derivation

Using technical documentation:

-   

> Extract validation rules

-   
-   

> Identify boundary conditions

-   
-   

> Map backend constraints to UI validations

-   
-   

> Generate invalid input scenarios

-   
-   

> Add error message assertions

-   

## Phase 5 -- Defect Regression Generation

For every P1/P2 defect:

1.  

> Extract root cause

2.  
3.  

> Map to affected workflow

4.  
5.  

> Generate dedicated regression E2E test

6.  
7.  

> Strengthen assertions

8.  
9.  

> Tag test appropriately

10. 

Regression generation is mandatory.

## Phase 6 -- Existing Test Impact Detection

If overlap detected:

-   

> Mark test as impacted

-   
-   

> Generate suggested modification diff

-   
-   

> Require human approval

-   
-   

> Do NOT auto-modify

-   

# 6. LLM Reasoning Layer (OpenAI API)

## Role

-   

> Contextual reasoning

-   
-   

> Workflow reconstruction

-   
-   

> Selenide code generation

-   
-   

> Negative scenario creation

-   
-   

> Automation Stability Confidence scoring

-   

## Hard Constraints

-   

> XPath selectors only

-   
-   

> Selenide syntax only

-   
-   

> Hybrid POM compliance

-   
-   

> No hardcoded test data

-   
-   

> QA environment only

-   
-   

> No automatic modification of existing tests

-   
-   

> No hallucination

-   

# 7. Test Code Generation Layer

### Output Structure

/pages/

PageName.java

/tests/

WorkflowE2ETest.java

### Code Requirements

-   

> Use \$x(\"//xpath\")

-   
-   

> Prefer attribute/text-based XPath

-   
-   

> Avoid index-based XPath

-   
-   

> Use shouldBe, shouldHave

-   
-   

> Reuse existing Page Objects

-   
-   

> Reuse existing utilities

-   
-   

> Integrate TDM calls before execution

-   
-   

> No Thread.sleep

-   
-   

> No hardcoded test values

-   

# 8. Automation Stability Confidence Model

## Definition

Measures automation reliability and selector stability.

Not related to AI probability.

## Factors

1.  

> Percentage of locator reuse

2.  
3.  

> Number of newly introduced XPath selectors

4.  
5.  

> Index-based XPath penalty

6.  
7.  

> Page Object reuse ratio

8.  
9.  

> Historical DOM volatility

10. 
11. 

> TDM integration completeness

12. 
13. 

> Acceptance Criteria clarity

14. 

## Output

Range: 0--100%

Example:

-   

> Locator reuse: 92%

-   
-   

> New XPath introduced: 1

-   
-   

> No index-based XPath

-   
-   

> Full TDM integration

-   
-   

> Overall Score: 88%

-   

Confidence score is separate from regression priority.

# 9. Regression Prioritization Model

Driven by:

-   

> Release note impact

-   
-   

> P1/P2 defect linkage

-   
-   

> Graph centrality of changed components

-   
-   

> Core workflow tagging

-   

Priority determines test execution importance, not stability.

# 10. PR Output Structure

PR comment must include:

1.  

> New Tests Generated

2.  
3.  

> Regression Tests (P1/P2)

4.  
5.  

> Impacted Existing Tests (Approval Required)

6.  
7.  

> Automation Stability Confidence Score

8.  
9.  

> Traceability Matrix

10. 

Traceability Table:

\| Requirement \| Code \| Test \| Defect \| Release \|

# 11. Technology Stack

  -----------------------------------------------------------------------
  **Layer**                        **Technology**
  -------------------------------- --------------------------------------
  LLM                              OpenAI API

  Graph DB                         Neo4j

  Vector DB                        Pinecone / Weaviate

  PR Integration                   GitHub / Bitbucket App

  UI Integration                   Figma API

  Defect Integration               Jira API

  Test Data                        Dedicated TDM Tool

  Environment                      QA Only
  -----------------------------------------------------------------------

# 12. Security & Governance

-   

> QA-only data usage

-   
-   

> No production data access

-   
-   

> PR-scoped execution

-   
-   

> Human approval for impacted tests

-   
-   

> No persistent LLM memory

-   
-   

> Token-based API authentication

-   
-   

> Incremental re-indexing

-   

# 13. Non-Goals (V1)

-   

> Multi-environment support

-   
-   

> Self-healing selectors

-   
-   

> Automatic refactoring of existing tests

-   
-   

> Performance testing

-   
-   

> Mobile automation

-   
-   

> Production defect ingestion

-   
-   

> AI-based flaky test healing

-   

# 14. Implementation Readiness Status

✔ Questionnaire completed\
✔ Design decisions finalized\
✔ Traceability model defined\
✔ LLM integration defined\
✔ Governance model defined\
✔ Architecture ready for engineering execution

**Architecture Version:** V1\
**Deployment Model:** Copilot-style PR Assistant\
**Automation Framework:** Selenide (Java)\
**Selector Strategy:** XPath\
**Environment Scope:** QA Only
