# CE-RISE Integrated Life Cycle Analysis

[![DOI](https://zenodo.org/badge/DOI/TOBEOBTAINED.svg)](https://doi.org/TOBEOBTAINED) [![Schemas](https://img.shields.io/badge/Schema%20Files-LinkML%2C%20JSON%2C%20SHACL%2C%20OWL-32CD32)](https://ce-rise-models.codeberg.page/integrated-lca/)

Repository for the data model used to represent integrated LCA results, including environmental, social, and economic impact indicators, methodological metadata, calculation parameters, characterization choices, and assessment settings. The model captures outcomes of LCA computations, while underlying process and flow data are provided separately by the `product-system` model.


---

## Data Model Structure

The Integrated Life Cycle Analysis data model provides a **flexible framework** for capturing LCA results across environmental, social, and economic dimensions. Rather than prescribing specific indicators, this model allows results to reference external indicator standards and methodologies, enabling compatibility with various LCA methods and evolving standards.

**Core Philosophy**: This model provides a structure for:
- **"What indicators were assessed?"** → References to standard indicators (ISO, ILCD, ReCiPe, etc.)
- **"What are the results?"** → Numeric values with units, uncertainty ranges
- **"Which method was used?"** → Method name, version, characterization factors
- **"How reliable are the results?"** → Data quality, completeness, uncertainty
- **"What standards apply?"** → ISO 14040/14044, PEF, EPD, specific sector standards

### Key Design Principles

1. **Triple Bottom Line Structure**: Environmental, Social, and Economic impacts organized hierarchically
2. **Method Transparency**: Clear documentation of impact assessment methods and characterization models
3. **Uncertainty Quantification**: Confidence intervals, sensitivity analysis results
4. **Comparability**: Standardized units and normalization factors
5. **Interoperability**: Alignment with ISO 14040/14044, GHG Protocol, and social LCA guidelines

### Core Hierarchy

```
IntegratedLCAResults (root)
├── LCAAnalysisInstance (repeatable - multiple analyses possible)
│   ├── 1. LCAStudyMetadata
│   │   ├── StudyIdentifier (REQUIRED - unique UUID)
│   │   ├── StudyName
│   │   ├── StudyDate
│   │   ├── CommissionerInfo
│   │   │   ├── OrganizationName
│   │   │   └── ContactInfo
│   │   ├── PractitionerInfo
│   │   │   ├── OrganizationName
│   │   │   ├── AuthorName
│   │   │   └── ContactInfo
│   │   ├── SoftwareInfo
│   │   │   ├── SoftwareName (SimaPro, openLCA, GaBi, etc.)
│   │   │   ├── SoftwareVersion
│   │   │   └── CalculationTimestamp
│   │   ├── DatabaseInfo
│   │   │   ├── BackgroundDatabase (ecoinvent, GaBi, etc.)
│   │   │   ├── DatabaseVersion
│   │   │   └── DataQualityInfo
│   │   ├── StudyScope
│   │   ├── FunctionalUnit
│   │   └── SystemBoundaries
│   ├── 2. ImpactAssessmentResults
│   ├── ImpactCategory (repeatable)
│   │   ├── CategoryIdentifier (URI/code from standard)
│   │   ├── CategoryName
│   │   ├── ImpactMethod (ReCiPe, ILCD, etc.)
│   │   ├── IndicatorResult
│   │   │   ├── NumericValue
│   │   │   ├── Unit
│   │   │   └── UncertaintyRange
│   │   └── CharacterizationFactors
│   │   └── AggregatedScores
│   │       ├── SingleScore
│   │       ├── NormalizedValues
│   │       └── WeightedResults
│   ├── 3. InventoryResults
│   │   ├── ProductSystemReference (URI/ID to product-system model)
│   │   ├── ProductSystemVersion (version used for calculation)
│   │   ├── CalculationDate (when inventory was accessed)
│   │   └── InventoryChecksum (optional - to detect source changes)
│   ├── 4. InterpretationResults
│   │   ├── DataQualityAssessment
│   │   │   ├── CompletenessCheck
│   │   │   ├── SensitivityAnalysis
│   │   │   └── ConsistencyCheck
│   │   ├── UncertaintyAnalysis
│   │   │   ├── ParameterUncertainty
│   │   │   ├── ModelUncertainty
│   │   │   └── ScenarioUncertainty
│   │   └── Limitations
│   │       ├── DataGaps
│   │       ├── MethodologicalChoices
│   │       └── Assumptions
│   └── 5. StandardCompliance
│       ├── ApplicableStandards (ISO 14040, PEF, etc.)
│       ├── MethodReference (link to method documentation)
│       ├── IndicatorSetReference (ILCD, ReCiPe indicators)
│       └── ValidationStatus
```

### Workflow Sequence

#### **Step 1: LCAStudyMetadata** 
Complete metadata for each LCA analysis instance:
- **StudyIdentifier**: REQUIRED unique UUID for the LCA instance
- **PractitionerInfo**: Author/institution conducting the analysis
- **SoftwareInfo**: LCA software and version used (SimaPro, openLCA, GaBi, Brightway, etc.)
- **DatabaseInfo**: Background database and version (ecoinvent 3.9, GaBi 2023, etc.)
- **FunctionalUnit**: Reference basis for all calculations (e.g., "1 kg of product")
- **SystemBoundaries**: Cradle-to-gate, cradle-to-grave, or gate-to-gate
- **StudyScope**: Goal, intended application, target audience

*Note: Multiple LCAAnalysisInstance objects can exist for the same product system, each with different methods, assumptions, or temporal snapshots.*

#### **Step 2: ImpactAssessmentResults**
Flexible structure for any impact indicators:
- **ImpactCategory**: Repeatable structure that references external indicator definitions
- **CategoryIdentifier**: URI or code pointing to standard indicator (e.g., ILCD:climate-change)
- **IndicatorResult**: Numeric value with unit and uncertainty
- **ImpactMethod**: Which method/version was used (ReCiPe 2016, ILCD 2.0, etc.)

#### **Step 3: InventoryResults**
References to the product system data used (no duplication):
- **ProductSystemReference**: URI or ID pointing to the product-system model
- **ProductSystemVersion**: Which version of the product system was used
- **CalculationDate**: Timestamp when the inventory was accessed for this analysis
- **InventoryChecksum**: Optional checksum to detect if source data has changed

#### **Step 4: InterpretationResults**
Quality and reliability assessment:
- **DataQualityAssessment**: Completeness, sensitivity, consistency checks
- **UncertaintyAnalysis**: Parameter, model, and scenario uncertainties
- **Limitations**: Documented data gaps and assumptions

#### **Step 5: StandardCompliance**
Links to external standards and methods:
- **ApplicableStandards**: Which LCA standards were followed
- **MethodReference**: Link to full method documentation
- **IndicatorSetReference**: Which indicator set was used (with version)
- **ValidationStatus**: Third-party verification status

### How to Represent the Three Components of Integrated LCA

The model supports Environmental, Social, and Economic indicators through a **flexible reference-based system**. Rather than hardcoding categories, the model identifies each dimension through external standard references:

#### **Storing Triple Bottom Line Data**

1. **Environmental Indicators**
   - Use `CategoryIdentifier` with environmental standard URIs (e.g., `ILCD:climate-change`, `ReCiPe:ozone-depletion`)
   - Set `ImpactMethod` to environmental methods (ReCiPe 2016, ILCD 2.0, TRACI, CML-IA)
   - Example: Climate change → CategoryIdentifier: `ILCD:GWP100`, ImpactMethod: `ILCD 2.0`

2. **Social Indicators**
   - Use `CategoryIdentifier` with social standard URIs (e.g., `SHDB:child-labor`, `UNEP-SETAC:fair-wages`)
   - Set `ImpactMethod` to social assessment methods (UNEP/SETAC S-LCA, Social Hotspots Database)
   - Example: Worker safety → CategoryIdentifier: `SHDB:occupational-hazards`, ImpactMethod: `SHDB 2023`

3. **Economic Indicators**
   - Use `CategoryIdentifier` with economic standard URIs (e.g., `ISO15686:life-cycle-cost`, `TCO:maintenance`)
   - Set `ImpactMethod` to economic valuation methods (LCC, TCO, NPV methods)
   - Example: Total cost → CategoryIdentifier: `ISO15686:total-LCC`, ImpactMethod: `ISO 15686-5:2017`

#### **Retrieving by Dimension**

To query results by sustainability dimension:

1. **Filter by Method Pattern**: Environmental methods contain "ReCiPe", "ILCD", "TRACI"; Social contain "SHDB", "UNEP", "SETAC"; Economic contain "LCC", "TCO", "ISO15686"

2. **Filter by URI Namespace**: Environmental URIs typically start with impact method prefixes; Social with social database prefixes; Economic with costing standard prefixes

3. **Use StandardCompliance References**: Check `ApplicableStandards` field for ISO 14040 series (environmental), UNEP/SETAC guidelines (social), or ISO 15686-5 (economic)

#### **Example Multi-Dimensional Impact Structure**
```
ImpactAssessmentResults
├── ImpactCategory[0] → Environmental (CategoryIdentifier: "ILCD:climate-change")
├── ImpactCategory[1] → Environmental (CategoryIdentifier: "ReCiPe:water-depletion")
├── ImpactCategory[2] → Social (CategoryIdentifier: "SHDB:child-labor")
├── ImpactCategory[3] → Social (CategoryIdentifier: "UNEP-SETAC:community-engagement")
├── ImpactCategory[4] → Economic (CategoryIdentifier: "ISO15686:initial-cost")
└── ImpactCategory[5] → Economic (CategoryIdentifier: "TCO:end-of-life-value")
```

This design ensures the model remains flexible while clearly supporting all three sustainability dimensions through internationally recognized standards.

### Data Properties

Each class has a corresponding value property (e.g., `name_value`, `company_id_value`) that holds the actual data. All value properties are string type except where specified otherwise.

#### SQL Identifiers

Every data point in the model includes a `sql_identifier` annotation that serves as a unique, machine-friendly database identifier. These identifiers follow a structured namespace pattern to ensure uniqueness across the entire data model:

**Pattern**: `lca_[category]_[specific_name]`

**Features:**
- **LCA Prefix**: All identifiers start with `lca_` to clearly identify them as belonging to the Integrated LCA data model
- **Hierarchical Namespacing**: Uses category prefixes (`gen_info_`, `mfr_info_`, `imp_info_`, `spec_info_`) to provide context and prevent naming conflicts
- **Database-Friendly**: Uses underscores and avoids special characters for SQL compatibility
- **Unique Across Model**: No duplicate identifiers, even when similar concepts appear in different parts of the hierarchy
- **Reasonable Length**: Abbreviated but meaningful names that balance clarity with practical database usage

**Examples:**
- `lca_study_functional_unit` - Functional unit in study metadata
- `lca_impact_category_id` - Impact category identifier
- `lca_impact_result_value` - Numeric result value
- `lca_inventory_flow_amount` - Elementary flow amount
- `lca_method_reference` - Reference to method documentation

This identifier system enables seamless integration with databases and ensures clear data model composition when combining with other CE-RISE data models.

---

## Development Roadmap

| Step | Component | Criticalities Identified | Solutions Implemented | Status | Missing/TODO |
|------|-----------|-------------------------|----------------------|--------|--------------|
| **1** | **LCAStudyMetadata** | • Need for unique study identification<br>• Variable system boundaries<br>• Different functional units<br>• Commissioner/practitioner tracking | • UUID-based study identifiers<br>• Flexible boundary definitions<br>• Standardized functional unit format<br>• Clear actor identification | **PLANNED** | • Study registry integration<br>• Automated metadata extraction |
| **2** | **ImpactAssessmentResults** | • Multiple indicator standards<br>• Evolving impact methods<br>• Need for flexibility<br>• Method versioning | • Reference-based indicator system<br>• Support for any impact method<br>• Version tracking for methods<br>• URI/UUID for indicators | **PLANNED** | • Indicator registry APIs<br>• Automatic method updates<br>• Cross-method mapping |
| **3** | **InventoryResults** | • Avoiding data duplication<br>• Maintaining traceability<br>• Version control of inputs<br>• Detecting source changes | • Reference-only approach<br>• Link to product-system model<br>• Version tracking<br>• Optional checksums | **PLANNED** | • Automatic reference validation<br>• Version compatibility checks<br>• Change detection alerts |
| **4** | **InterpretationResults** | • Quality assessment standards<br>• Uncertainty quantification<br>• Sensitivity analysis methods<br>• Documentation requirements | • Pedigree matrix implementation<br>• Multiple uncertainty methods<br>• Standardized sensitivity metrics<br>• Structured limitations | **PLANNED** | • Automated quality scoring<br>• Uncertainty propagation tools |
| **5** | **StandardCompliance** | • Multiple LCA standards<br>• Method documentation links<br>• Indicator set versions<br>• Validation tracking | • Standard reference system<br>• External method links<br>• Version control for indicators<br>• Validation status fields | **PLANNED** | • Standard compliance checker<br>• Automated validation |

### Integration Opportunities

- **LCA Software**: Integration with openLCA, SimaPro, GaBi, Brightway
- **Impact Methods**: ReCiPe 2016, IMPACT World+, TRACI, CML-IA
- **Databases**: ecoinvent, GaBi databases, Agri-footprint, Social Hotspots Database
- **Standards**: ISO 14040 series, GHG Protocol, UNEP/SETAC S-LCA guidelines
- **Reporting**: Environmental Product Declarations (EPD), Product Environmental Footprint (PEF)



---

## Publishing

Release artifacts for each version (`schema.json`, `shacl.ttl`, `model.owl`)  
are served directly from this URL:
```
https://ce-rise-models.codeberg.page/<repo-name>/
```


---

## Accessing Previous Releases

If you want to view the files published for version `v1.2.0`, open:

```
https://codeberg.org/CE-RISE-models/<repo-name>/src/tag/pages-v1.2.0/generated/
```

Files available in that directory typically include:

- schema.yaml
- schema.json
- shacl.ttl
- model.ttl
- index.html


---
<a href="https://europa.eu" target="_blank" rel="noopener noreferrer">
  <img src="https://ce-rise.eu/wp-content/uploads/2023/01/EN-Funded-by-the-EU-PANTONE-e1663585234561-1-1.png" alt="EU emblem" width="200"/>
</a>

Funded by the European Union under Grant Agreement No. 101092281 — CE-RISE.  
Views and opinions expressed are those of the author(s) only and do not necessarily reflect those of the European Union or the granting authority (HADEA).  
Neither the European Union nor the granting authority can be held responsible for them.

© 2025 CE-RISE consortium.  
Licensed under [Creative Commons Attribution–NonCommercial 4.0 International (CC BY-NC 4.0)](https://creativecommons.org/licenses/by-nc/4.0/).  
Attribution: CE-RISE project (Grant Agreement No. 101092281) and the individual authors/partners as indicated.

<a href="https://www.nilu.com" target="_blank" rel="noopener noreferrer">
  <img src="https://nilu.no/wp-content/uploads/2023/12/nilu-logo-seagreen-rgb-300px.png" alt="NILU logo" width="40"/>
</a>

Developed by NILU (Riccardo Boero — ribo@nilu.no) within the CE-RISE project.  



