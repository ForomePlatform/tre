# TRE Architecture with DSL-Driven Provenance - Internal Collaboration Document
## Core Concept
A Trusted Research Environment where every data transformation flows through a Domain-Specific Language (DSL) engine that enforces compliance, captures cell-level provenance, and enables complete reproducibility. The architecture implements defense-in-depth with controlled ingress/egress and medallion-pattern data quality tiers.
## Service Architecture Components
### Management Layer
- **ADMIN**: User management, role assignment, system monitoring, resource allocation
- **RESEARCH WORKBENCH**: Interactive analysis environment (Jupyter, RStudio, statistical tools)
- **DATA ENGINEERING**: Pipeline development, batch processing, workflow design tools
### Application Layer
Provides unified API gateway and service mesh for all underlying data services.
### Data Services Layer
#### Ingress/Egress Controls
- **INGRESS CONTROL**: Validates incoming data against security policies, checks data classification, verifies data use agreements, ensures format compliance
- **EGRESS CONTROL**: Statistical disclosure control (SDC), output review, de-identification verification, approval workflows
#### Core Services
- **DISCOVERY**: Metadata catalog, dataset search, schema registry, data lineage visualization
- **ACCESS CONTROL**: Fine-grained permissions, purpose-based access control (PBAC), consent management, audit logging
- **ORCHESTRATOR**: Workflow scheduling, pipeline coordination, dependency management, resource optimization
- **TRANSFORM**: Execution engine for DSL-defined transformations, supports distributed processing (Spark), handles data quality rules
- **FEDERATION**: Cross-TRE authentication, secure multi-party computation protocols, federated query routing, result aggregation
- **VALIDATION**: Schema validation, business rule checking, predicate evaluation, compliance verification
- **RULES ENGINE**: Compliance-as-code repository, regulatory rule management, dynamic policy evaluation
- **QUALITY CONTROL**: Data quality metrics, anomaly detection, completeness checking, statistical validation
- **PREDICATE LIBRARY**: Reusable validation components (k-anonymity, l-diversity, statistical significance, consent verification)
### Data Layer (Medallion Architecture)
#### Bronze Layer (Raw Zone)
- Immutable landing zone for source data
- Maintains original formats and schemas
- Captures ingestion metadata (source, timestamp, checksums)
- No transformations applied
#### Silver Layer (Trusted Zone)
- Cleaned and standardized data
- Consistent schemas across sources
- Applied pseudonymization and de-identification
- Data quality validated
- Compliance rules enforced
#### Gold Layer (Refined Zone)
- Analytics-ready datasets
- Aggregated for specific research purposes
- Optimized for query performance
- Pre-computed statistical measures
### Infrastructure Foundation
Cloud-native or on-premise deployment with isolated compute, encrypted storage, network segmentation.
## Data Flow Orchestration
### 1. Data Ingress Flow
```
External Source → INGRESS CONTROL (validate) → ORCHESTRATOR (schedule) → Bronze Layer
```
- Ingress Control checks: data use agreement, classification level, format validity
- Orchestrator schedules ingestion based on resource availability
- Bronze preserves raw data with full provenance metadata
### 2. Bronze-to-Silver Transformation
```
Bronze → TRANSFORM (via DSL) → [RULES ENGINE + VALIDATION] → Silver
```
- DSL defines transformation logic
- Rules Engine applies compliance policies
- Validation checks predicates
- Provenance Store captures complete lineage
### 3. Silver-to-Gold Refinement
```
Silver → TRANSFORM (via DSL) → [QUALITY CONTROL] → Gold
```
- Purpose-specific aggregations
- Statistical computations
- Quality metrics validated
- Research-ready outputs generated
### 4. Research Access Flow
```
Researcher → ACCESS CONTROL (authorize) → DISCOVERY (find) → WORKBENCH (analyze) → EGRESS CONTROL (review)
```
- Purpose-based access control
- Metadata-driven discovery
- Controlled analysis environment
- Output review before release
## DSL Engine Architecture
### Meta-Metalanguage Specification
- **Base**: YAML/CWL for workflow definition + Groovy for logic
- **Granularity**: Cell-level transformation tracking
- **Capabilities**:
    - Abstract business rules
    - Regulatory compliance encoding
    - Validation predicates
    - Lineage graph generation
### Provenance Store Structure
- **Transformation Graph**: Directed acyclic graph of all operations
- **Compliance Proofs**: Cryptographic evidence of rule application
- **Validation Results**: Predicate evaluation outcomes
- **Temporal Versioning**: Point-in-time reconstruction capability
## Compliance-as-Code Example (EHDS Context)
```groovy
// European Health Data Space (EHDS) Compliance
transform patient_outcomes {
  from: bronze.clinical_records
  ehds_compliance: {
    // EHDS Article 33: Purpose limitation for secondary use
    purpose_of_use: {
      category: "scientific_research"
      specific_purpose: "cardiovascular_outcomes_study"
      ethics_approval: "ETH-2024-0142"
      lawful_basis: "public_interest_research"
    }
    // EHDS Article 34: Data categories for secondary use
    permitted_categories: [
      "diagnosis_codes",      // ICD-10
      "procedure_codes",      // SNOMED-CT
      "lab_results",         // LOINC codes only
      "medications",         // ATC classification
      "demographics"         // age, gender, region only
    ]
    // EHDS Article 44: Health data access bodies requirements
    access_environment: {
      secure_processing: true
      data_localization: "EU"
      cross_border_transfer: false
    }
    // GDPR Article 89: Safeguards for research
    safeguards: {
      pseudonymization: true
      encryption_at_rest: "AES-256"
      access_logging: "comprehensive"
    }
  }
  pseudonymization_rules: {
    patient_id → hash(salt: project_specific)
    birth_date → age_at_event
    admission_date → relative_timeline(index_date: "diagnosis")
    hospital_id → region_category
    treating_physician → "removed"
  }
  quality_predicates: [
    // Statistical disclosure control
    assert_k_anonymity(k: 5, quasi_identifiers: ["age", "gender", "region"]),
    assert_l_diversity(l: 3, sensitive: "primary_diagnosis"),
    // Data quality
    assert_completeness(critical_fields: ["diagnosis", "outcome"], threshold: 0.95),
    assert_temporal_consistency(date_fields: ["admission", "discharge", "procedure"]),
    // Consent and purpose
    assert_valid_consent(type: "broad_consent_research"),
    assert_purpose_compatibility(declared: "cardiovascular", used_for: "cardiovascular")
  ]
  to: silver.cardiovascular_cohort
  provenance_capture: {
    transformation_graph: true,
    compliance_attestation: generate_signed(),
    validation_report: comprehensive,
    retention_period: "10_years"  // EHDS audit requirement
  }
}
```
## Predicate Library Components
### Privacy Predicates
- `k_anonymity(k, quasi_identifiers)`: Ensure k-indistinguishability
- `l_diversity(l, sensitive_attribute)`: Protect sensitive values
- `t_closeness(t, sensitive_attribute)`: Limit information gain
- `differential_privacy(epsilon, delta)`: Provide formal privacy guarantee
### Compliance Predicates
- `purpose_compatibility(declared, actual)`: Verify use aligns with consent
- `retention_compliance(max_period)`: Check data retention limits
- `cross_border_check(destination)`: Validate international transfers
- `consent_validity(consent_type, purpose)`: Ensure appropriate consent
### Quality Predicates
- `completeness(fields, threshold)`: Verify data completeness
- `consistency(rules)`: Check business rule compliance
- `uniqueness(identifiers)`: Ensure no duplicates
- `temporal_validity(date_fields)`: Validate temporal logic
### Statistical Predicates
- `statistical_power(effect_size, alpha)`: Ensure sufficient sample
- `bias_detection(protected_attributes)`: Check for systematic bias
- `significance(p_value)`: Validate statistical significance
- `sample_representativeness(population)`: Verify generalizability
## Key Architectural Decisions
### Why DSL as Central Orchestrator?
- Single source of truth for all transformations
- Compliance verified at transformation time, not post-hoc
- Complete reproducibility through captured logic
- Enables formal verification of data handling
### Why Medallion Pattern?
- Clear data quality progression
- Separation of concerns (raw preservation vs. analytics)
- Enables parallel research projects on same data
- Facilitates data governance and lifecycle management
### Why Separate Ingress/Egress Control?
- Defense-in-depth security model
- Regulatory requirement for controlled environments
- Enables automated disclosure control
- Provides clear audit boundaries
### Why Predicate Library?
- Reusable compliance components
- Standardization across projects
- Formal verification capabilities
- Enables automated compliance reporting
## Federation Considerations
- Each TRE maintains sovereign provenance store
- DSL definitions can be shared for reproducibility
- Federated queries route through FEDERATION service
- Results aggregation respects local privacy thresholds
- Cross-TRE provenance linked via distributed ledger
## Performance Optimizations
- Cell-level tracking uses probabilistic data structures for scale
- Transformation graph stored as distributed graph database
- Predicates evaluated lazily where possible
- Orchestrator implements intelligent caching and materialization
## Next Implementation Steps
1. Formalize DSL syntax with ANTLR grammar
2. Implement predicate library with formal proofs
3. Design federated provenance protocol
4. Build reference implementation with open-source stack
5. Develop compliance template library for GDPR/EHDS/NHS
---
*This architecture provides comprehensive data governance while maintaining research flexibility. The DSL-centric approach with built-in compliance and complete provenance addresses the core challenge of trust in sensitive data research.*