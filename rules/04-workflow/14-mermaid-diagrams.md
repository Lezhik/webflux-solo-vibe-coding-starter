# 14 – Mermaid Diagrams – Semantic Visual Standards

**File Path:** rules/04-workflow/14-mermaid-diagrams.md  
**Domain:** TODO: <Project Domain, e.g., "E-commerce Order Processing">  
**Last Updated:** 2026-01-23
**Status:** Active  

<!-- FILE_TITLE: Mermaid Diagram Semantic Standards -->  
<!-- FILE_PATH: rules/04-workflow/14-mermaid-diagrams.md -->  

---

## 🎯 Purpose  
Establish standardized Mermaid diagram conventions to:  
1. **Embed semantics** for LLM-parsable workflows, enabling automated analysis and validation.  
2. **Propagate vibe coding intent** through visual patterns for cross-team alignment (e.g., reactive flows, module boundaries).  
3. **Ensure consistency** in feature specs, architectural docs, and data flows, reducing ambiguity in complex systems like order processing or payment streams.  

These diagrams act as "living artifacts" that integrate with modular architecture rules, supporting CI/CD enforcement and semantic discoverability.

## 📌 Scope  
- **Applies to:** Sequence diagrams, flowcharts, state machines, and module maps in:  
  - Feature branches (`features/<name>/diagrams/`)  
  - Architectural documentation (`docs/architecture/`)  
  - API contracts and reactive stream definitions.  
- **Excludes:** Informal sketches or non-versioned visuals (e.g., brainstorming tools like Draw.io).  
- **Dependencies:**  
  - [07-semantic-comments.md](/rules/02-semantics/07-semantic-comments.md) (for annotation alignment).  
  - [04-modular-architecture.md](/rules/01-project/04-modular-architecture.md) (for boundary tagging).  
  - [05-vibe-coding.md](/rules/02-semantics/05-vibe-coding.md) (for intent propagation).  

---

## 🧩 Diagram Structure Template  

### Mandatory Header  
All diagrams must start with this YAML frontmatter for metadata and parsability:  
```mermaid  
---  
title: <DOMAIN-CONTEXT>: <Core-Functionality>  
  # Example: "Order: Reactive Payment Propagation"  
description: >  
  <Brief purpose (1 sentence) + vibe intent (1 sentence)>  
  # Example: "Visualizes event-driven payment flow from service to UI. @Vibe ensures non-blocking updates for real-time UX."  
author: @<github-handle-or-team>  
modified: YYYY-MM-DD @<edit-type>  # e.g., @Manual_edit or @LLM-generated  
status: draft | active | deprecated  
---  
```  
- **Date Convention:** Update `modified` on every change; use `@Manual_edit` for human tweaks.  
- **Status Enforcement:** Diagrams in PRs must be `active` or `draft` to pass CI.

### Core Syntax Rules  
1. **Diagram Types & Direction**  
   | Type              | Mermaid Syntax     | Use Case Example                  | Direction Recommendation |  
   |-------------------|--------------------|-----------------------------------|---------------------------|  
   | **Sequence Flow** | `sequenceDiagram`  | Service interactions, async calls | N/A (vertical by default) |  
   | **Module Map**    | `graph TD` or `graph LR` | Architecture boundaries, pipelines | `TD` (top-down) for hierarchies; `LR` (left-right) for linear flows |  
   | **State Machine** | `stateDiagram-v2`  | Component lifecycles (e.g., order states: pending → confirmed) | N/A |  

2. **Node Identification**  
   - Use `snake_case` IDs for nodes.  
   - Enclose labels in brackets for readability: `A[ComponentName @Semantic(Role)]`.  
   - Tag boundaries with `@Domain(ModuleName)` for modular clarity.  
   - Example:  
     ```mermaid  
     payment_service[PaymentService @Semantic(Gateway) @Domain(Payment)]  
     ```  

3. **Edge & Flow Conventions**  
   - Label edges with domain-specific verbs or types: `-->|action@Vibe(intent)|`.  
   - Mandatory tags for semantics:  
     - `@dataflow(<type>)` for data exchanges (e.g., `@dataflow(async-event)`).  
     - `@Vibe(<non-functional-aspect>)` for intent (e.g., `@Vibe(real-time)` or `@Vibe(eventual-consistency)`).  
   - Example:  
     ```mermaid  
     payment_service -->|processPayment @dataflow(async-event) @Vibe(atomic)| order_processor  
     ```  

### Visual Styling Standards  
Apply consistent class definitions for semantic coloring and patterns:  
```mermaid  
classDef reactive fill:#fffd96,stroke:#aa0000,stroke-width:2px  /* Yellow for async/reactive streams */  
classDef boundary fill:#90ee90,stroke:#333,stroke-width:2px     /* Green for module edges */  
classDef entity fill:#bae1ff,stroke:#003366                     /* Blue for data entities/DTOs */  
classDef error fill:#ffb3ba,stroke:#d60000                      /* Red for failure paths */  
classDef legacy fill:#ffcccb,stroke:#999                        /* Pink for deprecated components */  
```  
- Assign classes post-diagram: `class NodeA,NodeB reactive`.  
- **Anti-Pattern:** Unstyled or ambiguous flows (e.g., `A --> B` without tags).  
- **Best Practice:** Use yellow (`reactive`) for Vue/backend streams to highlight event-driven patterns.

---

## 📚 Examples  

### 1. Reactive Sequence Diagram (Payment Flow)  
```mermaid  
---  
title: Payment: Order Finalization Sequence  
description: >  
  Depicts user-triggered payment processing with event emission to UI.  
  @Vibe promotes loose coupling via non-blocking streams.  
author: @payments-team  
modified: 2024-06-15 @Manual_edit  
status: active  
---  

sequenceDiagram  
    participant User  
    participant PaymentAPI as PaymentAPI @Semantic(Gateway) @Domain(Payment)  
    participant EventBus as EventBus @Semantic(Dataflow)  
    participant UI as UI @Semantic(View)  

    User ->> PaymentAPI: finalizeOrder(orderId)  
    PaymentAPI ->> EventBus: emitPaymentEvent @dataflow(async-event) @Vibe(atomic)  
    EventBus ->> UI: streamStatus @dataflow(reactive-stream) @Vibe(real-time)  
    Note over PaymentAPI,UI: Error handling: Retry on failure @Vibe(resilient)  

    class PaymentAPI,EventBus reactive  
    class UI entity  
```  

### 2. Module Architecture Flowchart  
```mermaid  
---  
title: Architecture: Order-Payment Module Boundaries  
description: >  
  Maps dependencies between domains to enforce modularity.  
  @Vibe prevents tight coupling in e-commerce pipelines.  
author: @architecture-team  
modified: 2024-06-15 @LLM-generated  
status: active  
---  

graph LR  
    subgraph OrderDomain [@Domain(Order)]  
        A[OrderService @Semantic(Orchestrator)]  
        B[OrderRepo @Semantic(Repository)]  
        A --> B  
    end  

    subgraph PaymentDomain [@Domain(Payment)]  
        C[PaymentService @Semantic(Gateway)]  
        D[PaymentGateway @Semantic(External)]  
        C --> D  
    end  

    A -->|integratePayment @dataflow(event) @Vibe(loose-coupling)| C  

    class OrderDomain,PaymentDomain boundary  
    class A,C reactive  
```  

### 3. State Machine Example (Order Lifecycle)  
```mermaid  
---  
title: Order: State Transitions  
description: >  
  Defines order states from creation to fulfillment.  
  @Vibe ensures idempotent transitions in reactive systems.  
author: @order-team  
modified: 2024-06-15 @Manual_edit  
status: active  
---  

stateDiagram-v2  
    [*] --> Pending: createOrder  
    Pending --> Confirmed: processPayment @dataflow(sync) @Vibe(atomic)  
    Confirmed --> Fulfilled: shipOrder @dataflow(async-event) @Vibe(eventual-consistency)  
    Confirmed --> Failed: paymentError @Vibe(resilient)  
    Failed --> [*]  
    Fulfilled --> [*]  

    class Confirmed,Fulfilled entity  
    note right of Failed: Retry logic applied  
```  

---

## ⚙️ Enforcement Mechanics  

### Validation Pipeline  
Integrate checks into build and CI for syntax, semantics, and compliance:  

1. **Syntax Validation (Mermaid CLI):**  
   ```bash  
   # Local or CI command  
   npx @mermaid-js/mermaid-cli -i features/**/*.md -o output/  
   ```  
   - Fails on invalid Mermaid syntax; blocks commits via Git hooks.  

2. **Semantic Audit (Custom Script):**  
   - Scan for required tags (`@Semantic`, `@Vibe`, `@dataflow`, `@Domain`).  
   - Example Gradle task:  
     ```kotlin  
     // build.gradle.kts  
     tasks.register("validateDiagrams") {  
         inputs.files(fileTree("features").include("**/*.md"))  
         doLast {  
             exec {  
                 commandLine("node", "scripts/audit-mermaid.js", "features/")  // Custom JS for tag checks  
             }  
         }  
     }  
     ```  
   - Warns in CI; auto-fixes simple issues (e.g., missing headers) in PRs.  

3. **Styling & Compliance:**  
   - Use JSON Schema for header validation.  
   - **Failure Actions:** Block PRs without tags; auto-format via pre-commit hooks.  

### Repository Structure  
Place diagrams contextually:  
```
features/  
├── order-processing/  
│   └── diagrams/  
│       ├── sequence.payment_flow.md  # Sequence example  
│       └── architecture.modules.md   # Graph example  
└── payment-gateway/  
    └── diagrams/  
        └── state.order_lifecycle.md  # State machine  
docs/architecture/  
└── global_flows.md  # Cross-feature overviews  
```

---

## 📋 Compliance Checklist  
- [ ] Header includes `title`, `description`, `author`, `modified`, and `status`.  
- [ ] Nodes use `snake_case` IDs with `@Semantic` or `@Domain` tags.  
- [ ] Edges labeled with `@dataflow`/`@Vibe` for all interactions.  
- [ ] Appropriate diagram type and direction selected.  
- [ ] Styling applied (e.g., `classDef reactive` for streams).  
- [ ] Passes `./gradlew validateDiagrams` and Mermaid CLI checks.  
- [ ] Placed in `features/<domain>/diagrams/` or equivalent.  
- [ ] Anti-patterns avoided (e.g., no untagged flows).  

---

## 🔗 Interlocks with Other Rules  
- **Semantics:** Aligns with [07-semantic-comments.md](/rules/02-semantics/07-semantic-comments.md) for tag consistency (e.g., `@Vibe` in code comments).  
- **Modularity:** References [04-modular-architecture.md](/rules/01-project/04-modular-architecture.md) and [29-semantic-discoverability.md](/rules/07-modularity/29-semantic-discoverability.md) for domain boundaries.  
- **Lifecycle:** Integrates with [13-feature-lifecycle.md](/rules/04-workflow/13-feature-lifecycle.md) – diagrams required at "Spec Review" stage.  
- **Build:** Enforced in [22-gradle-multimodule.md](/rules/06-build/22-gradle-multimodule.md) pipelines.  
- **Reviews:** Mandatory in [16-review-checklist.md](/rules/04-workflow/16-review-checklist.md).  

---

## 📜 Revision History  
| Version | Date       | Author            | Changes Summary                          |  
|---------|------------|-------------------|------------------------------------------|  
| 1.0     | 2026-01-23 | LLM-Generated     | Initial draft with basic syntax rules.   |  

---

**Implementation Notes:**  
1. **Customization:** Replace all `TODO:` placeholders and example domains (e.g., "Order") with project specifics before merging.  
2. **Tooling Tips:** Draft in Mermaid Live Editor (mermaid.live) for real-time validation; export to MD for repo.  
3. **Advanced Usage:** For complex flows, add `Note` blocks with `// Why:` comments explaining decisions (e.g., async boundaries).  
4. **LLM Parsability:** Strict tagging enables tools like custom Grok analysis for auto-generating code from diagrams.  
5. **Versioning:** Maintain diagram syntax versions to support evolving Mermaid features without breaking parses.  

This standard combines semantic depth from prior drafts, adds practical examples for e-commerce domains, and streamlines enforcement for seamless adoption in Kilo Code workflows.