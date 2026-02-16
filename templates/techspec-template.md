# Technical Specification

## 1. Executive Summary
[Summarize the implementation approach, major architecture decisions, and why this approach is preferred.]

## 2. Inputs and Preconditions
- PRD Path: `tasks/prd-[FEATURE_SLUG]/prd.md`
- Functional Requirements in Scope: `[RF01, RF02, ...]`
- Assumptions: `[ASSUMPTIONS]`
- Preconditions Verified: `[YES/NO + NOTES]`

## 3. System Architecture

### 3.1 Component Overview
[List every component that is new or modified.]

| Component | Type (`new|modified|existing`) | Responsibility | RF Coverage |
|---|---|---|---|
| `[COMPONENT_NAME]` | `[TYPE]` | `[RESPONSIBILITY]` | `[RF_IDS]` |

### 3.2 Data Flow
[Describe the end-to-end flow, key state transitions, and boundaries between components.]

## 4. Implementation Design

### 4.1 Core Interfaces
[Define key interfaces and contracts. Keep each code example compact.]

```ts
// Example
interface ServiceName {
  method(input: InputType): Promise<OutputType>;
}
```

### 4.2 Data Models
[Define core domain models and request/response shapes.]

### 4.3 API Endpoints
[Include only if applicable.]

| Method | Path | Purpose | Request | Response |
|---|---|---|---|---|
| `[GET|POST|...]` | `[PATH]` | `[PURPOSE]` | `[REQUEST_SCHEMA_OR_REF]` | `[RESPONSE_SCHEMA_OR_REF]` |

## 5. Integration Points
[Include external/internal dependencies and failure handling.]

| Dependency | Purpose | Auth | Failure Modes | Timeout/Retry Strategy |
|---|---|---|---|---|
| `[SERVICE_OR_API]` | `[PURPOSE]` | `[AUTH_METHOD]` | `[FAILURE_MODES]` | `[TIMEOUT_AND_RETRY]` |

## 6. Requirements Traceability (Mandatory)
[Every in-scope `RFxx` must map to design and validation evidence.]

| RF ID | Requirement Summary | Design Element (Component/Interface) | Decision ID | Validation Coverage |
|---|---|---|---|---|
| `RF01` | `[SUMMARY]` | `[DESIGN_ELEMENT]` | `[D01]` | `[TEST_CASE_OR_COMMAND]` |

## 7. Testing Strategy

### 7.1 Unit Tests
- Scope: `[COMPONENTS_TO_TEST]`
- Mocks: `[EXTERNAL_DEPENDENCIES_TO_MOCK]`
- Critical Cases: `[CRITICAL_CASES]`

### 7.2 Integration Tests
- Scope: `[INTEGRATION_BOUNDARIES]`
- Test Data / Setup: `[SETUP]`

### 7.3 End-to-End Tests
- Scope: `[USER_JOURNEYS]`
- Tooling: `[PLAYWRIGHT_OR_OTHER]`

## 8. Development Sequencing

### 8.1 Build Order
1. `[STEP_1]` — `[WHY_FIRST]`
2. `[STEP_2]` — `[DEPENDENCY_OR_RATIONALE]`
3. `[STEP_3]` — `[DEPENDENCY_OR_RATIONALE]`

### 8.2 Technical Dependencies
- `[BLOCKING_DEPENDENCY_OR_INFRA]`

## 9. Observability and Monitoring
- Metrics: `[METRICS_TO_EXPOSE]`
- Logs: `[STRUCTURED_LOG_REQUIREMENTS]`
- Dashboards / Alerts: `[DASHBOARD_OR_ALERTING_EXPECTATIONS]`

## 10. Technical Considerations

### 10.1 Key Decisions
| Decision ID | Decision | Rationale | Trade-off |
|---|---|---|---|
| `D01` | `[DECISION]` | `[RATIONALE]` | `[TRADE_OFF]` |

### 10.2 Known Risks and Mitigations
| Risk | Impact | Mitigation | Owner |
|---|---|---|---|
| `[RISK]` | `[LOW|MEDIUM|HIGH]` | `[MITIGATION]` | `[OWNER]` |

### 10.3 Assumptions and Unknowns
| Item | Type (`assumption|unknown`) | Validation Plan |
|---|---|---|
| `[ITEM]` | `[TYPE]` | `[PLAN]` |

### 10.4 Standards and Rule Compliance
[List applicable project rules/policies and how the implementation complies.]

- `[RULE_OR_STANDARD]`: `[COMPLIANCE_NOTE]`

## 11. Relevant and Dependent Files
- `[FILE_PATH]` — `[WHY_RELEVANT]`
