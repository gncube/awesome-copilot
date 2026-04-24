# PlantUML Architect - Diagram Validation Checklist

A comprehensive checklist for the **PlantUML Architect** agent's self-healing validation engine, aligned with the 7-test validation suite and 4 TDD self-tests. This guide ensures diagrams are production-ready, architecturally sound, and guaranteed compilable.

**Note**: The agent performs these validations automatically with auto-healing. Use this checklist for manual review or when creating diagrams without the agent.

---

## 🧠 Self-Healing Validation Pipeline

The PlantUML Architect uses a 5-step generation pipeline with automated validation and repair:

```
Step 1: Select Template
         ↓
Step 2: Instantiate Structure (boundaries first, elements inside)
         ↓
Step 3: Apply Tags & Branding
         ↓
Step 4: Add Relationships
         ↓
Step 5: Validate + Auto-Heal (if validation fails: max 2 attempts → fallback to golden template)
```

---

## 🔍 7 PRE-FLIGHT VALIDATION TESTS

### Test 1: Boundary Integrity ✅

**Requirement**: No boundary contains alias-only references

**Check**:

- [ ] Every Container_Boundary, System_Boundary, Component_Boundary contains ≥1 substantive element
- [ ] Substantive elements: Container(), System(), Component(), Node(), Person()
- [ ] Icon macros (FA*, DEV*, etc.) count as substantive
- [ ] NOT just Rel() calls to other aliases

**FAIL**:

```plantuml
Container_Boundary(api) { Rel(user, gateway, "calls") }
```

**PASS**:

```plantuml
Container_Boundary(api) {
    FA_SERVER(gateway, "API Gateway", "node")
    Rel(gateway, db, "queries")
}
```

**Fix Applied**: Fix A – Boundary Repair (adds missing element definitions)

---

### Test 2: Single Declaration Rule ✅

**Requirement**: Each alias is declared exactly once

**Check**:

- [ ] No alias appears in multiple element definitions
- [ ] Each `Container()`, `Component()`, `System()`, etc. has unique alias
- [ ] Duplicates cause FAIL

**FAIL**:

```plantuml
Container(api, "API", "Tech1")
Container(api, "API", "Tech2")  ' Same alias declared twice
```

**PASS**:

```plantuml
Container(api, "API", "Tech1")
Container(db, "Database", "Tech2")
```

**Fix Applied**: Fix C – Duplicate Removal (keeps first valid, removes duplicates)

---

### Test 3: Hierarchy Validity ✅

**Requirement**: Component diagrams MUST use Container_Boundary (no floating components)

**Check**:

- [ ] **Context Level**: Only Person, System, System_Boundary allowed
- [ ] **Container Level**: Only Container, ContainerDb, System_Boundary, Container_Boundary
- [ ] **Component Level**: Only Component, Container_Boundary, Component_Boundary
- [ ] **Deployment Level**: Only Deployment_Node, Node, Node_L, Node_R

**FAIL**:

```plantuml
@startuml
!include C4_Component.puml
Component(auth, "Auth", "Tech", "Desc")  ' Outside any boundary
@enduml
```

**PASS**:

```plantuml
@startuml
!include C4_Component.puml
Container_Boundary(api, "API") {
    Component(auth, "Auth", "Tech", "Desc")
}
@enduml
```

**Fix Applied**: Fix B – Container Correction (wraps components in boundaries)

---

### Test 4: Relationship Integrity ✅

**Requirement**: All Rel() targets exist; no orphan relationships

**Check**:

- [ ] Every relationship source and target are declared aliases
- [ ] No Rel() references undefined aliases
- [ ] All Rel() parameters properly formatted: `Rel(from, to, "label", ?tech)`

**FAIL**:

```plantuml
Rel(api, undefined_db, "queries")  ' undefined_db not declared
```

**PASS**:

```plantuml
Container(api, "API", "Tech", "Desc")
Container(db, "Database", "Tech", "Desc")
Rel(api, db, "queries")  ' Both api and db are declared
```

**Fix Applied**: Fix D/E – Auto-inject tag definitions or remove invalid relationships

---

### Test 5: Legend Integrity ✅

**Requirement**: All $tags used are defined via AddElementTag(); SHOW_LEGEND() or LAYOUT_WITH_LEGEND() present

**Check**:

- [ ] Every `$tags="tag-name"` has matching `AddElementTag("tag-name", ...)`
- [ ] `SHOW_LEGEND()` or `LAYOUT_WITH_LEGEND()` present when tags are used
- [ ] No orphaned tag references

**FAIL**:

```plantuml
Container(api, "API", "Tech", $tags="prod")  ' tag "prod" never defined
```

**PASS**:

```plantuml
AddElementTag("prod", $bgColor="#4F46E5", $fontColor="#FFFFFF", $borderColor="#3730A3")
Container(api, "API", "Tech", $tags="prod")
SHOW_LEGEND()
```

**Fix Applied**: Fix D – Missing Tag Definition (auto-injects AddElementTag)

---

### Test 6: Sprite Safety ✅

**Requirement**: Icon macros and sprites must be resolvable

**Check**:

- [ ] Icon macro prefix valid (FA*, FA5*, FA6*, DEV*, DEV2*, MAT*, GOV*, WE*)
- [ ] Icon macro name exists in tupadr3 v3.0.0 library
- [ ] If using $sprite="...", sprite path valid or inlined
- [ ] All icon includes present: `!include $ICONURL/common.puml` + specific icons

**FAIL**:

```plantuml
DEV_UNDEFINED(db, "Database", "node")  ' UNDEFINED icon doesn't exist
```

**PASS**:

```plantuml
!include $ICONURL/devicons/mysql.puml
DEV_MYSQL(db, "Database", "node")  ' DEV_MYSQL is defined
```

**Fix Applied**: Fix E – Invalid Sprite (removes or replaces with valid icon)

---

### Test 7: Layout Consistency ✅

**Requirement**: LAYOUT_WITH_LEGEND() or SHOW_LEGEND() present when custom tags are used

**Check**:

- [ ] If `AddElementTag()` present → `LAYOUT_WITH_LEGEND()` or `SHOW_LEGEND()` present
- [ ] No conflicting layout options (e.g., LAYOUT_TOP_DOWN + LAYOUT_LEFT_RIGHT)
- [ ] Legend not cluttered (≤15 custom tags recommended)

**FAIL**:

```plantuml
AddElementTag("prod", ...)
Container(api, "API", "Tech", $tags="prod")
' Missing LAYOUT_WITH_LEGEND() or SHOW_LEGEND()
```

**PASS**:

```plantuml
LAYOUT_WITH_LEGEND()
AddElementTag("prod", ...)
Container(api, "API", "Tech", $tags="prod")
SHOW_LEGEND()
```

**Fix Applied**: Automatic insertion of `SHOW_LEGEND()`

---

## 🧪 TDD-STYLE SELF-TESTS (4 INTERNAL VERIFICATIONS)

Before the agent returns a diagram, it asks itself:

### Test Prompt A: "Does every alias appear exactly once?"

```
Search: Scan entire diagram for duplicate alias declarations
Action: Apply Fix C if found
Result: PASS if every alias unique, FAIL if any duplicates
```

### Test Prompt B: "Are any boundaries referencing aliases instead of defining elements?"

```
Search: Scan all boundaries for Rel() without element definitions
Action: Apply Fix A if found
Result: PASS if all boundaries have elements, FAIL if boundary is alias-only
```

### Test Prompt C: "Does this diagram follow correct C4 hierarchy?"

```
Search: Validate element types match diagram level
Context: Person, System, System_Boundary only
Container: Container, ContainerDb, System_Boundary, Container_Boundary only
Component: Component, Container_Boundary, Component_Boundary only
Action: Apply Fix B if found
Result: PASS if hierarchy correct, FAIL if wrong elements at wrong level
```

### Test Prompt D: "Will this compile without syntax errors?"

```
Search: @startuml/@enduml tags, bracket matching, quote matching, macro syntax
Action: Apply Fix D/E if syntax errors found
Result: PASS if valid PlantUML syntax, FAIL on syntax error
```

---

## ✅ PRE-FLIGHT VALIDATION CHECKLIST (15 ITEMS)

Use this before submitting diagrams for architecture review:

### 7 Validation Tests

- [ ] Test 1: Boundary Integrity – No empty boundaries
- [ ] Test 2: Single Declaration – Each alias once
- [ ] Test 3: Hierarchy Validity – Correct C4 level
- [ ] Test 4: Relationship Integrity – All targets exist
- [ ] Test 5: Legend Integrity – All tags defined
- [ ] Test 6: Sprite Safety – All icons resolvable
- [ ] Test 7: Layout Consistency – Legend configured

### 4 TDD Self-Tests

- [ ] TDD A: Every alias appears exactly once
- [ ] TDD B: No boundaries with alias-only references
- [ ] TDD C: C4 hierarchy correct
- [ ] TDD D: Compiles without syntax errors

### Additional Checks

- [ ] **Syntax**: @startuml/@enduml present, brackets matched, quotes closed
- [ ] **Icon Sprites**: All FAx*, DEVx*, MAT\_ macros valid
- [ ] **Colors**: All hex colors valid (#RRGGBB), WCAG 4.5:1 contrast minimum
- [ ] **Completeness**: Every element has label, every relationship has descriptive label

---

## 🔧 AUTO-HEAL STRATEGIES (5 FIXES)

If validation fails, the agent applies these fixes (max 2 attempts):

### Fix A: Boundary Repair

**Problem**: Boundary has alias-only references  
**Action**: Add missing element definitions inside boundary

### Fix B: Container Correction

**Problem**: Component_Boundary without parent container nesting  
**Action**: Restructure into proper Container_Boundary hierarchy

### Fix C: Duplicate Removal

**Problem**: Same alias declared multiple times  
**Action**: Keep first valid instance, remove duplicates

### Fix D: Missing Tag Definition

**Problem**: Tag used but never defined  
**Action**: Auto-inject `AddElementTag()` definitions

### Fix E: Invalid Sprite

**Problem**: Sprite/icon macro not resolvable  
**Action**: Remove $sprite parameter or replace with valid icon macro

---

## 🧱 GOLDEN TEMPLATE FALLBACK

**If after 2 attempts validation still fails:**

1. Agent **FALLS BACK** to golden template (Component/Container/Context)
2. Agent **REBUILDS** diagram using golden template structure
3. Agent **PRESERVES** user's intent (swaps template content, keeps relationships)
4. Agent **MARKS** with comment: `' AUTO-HEALED: 2 attempts, fallback to golden template`

**Result**: Guaranteed compilable, valid C4, production-ready diagram

---

## 📊 Common Validation Issues & Quick Fixes

| Issue                           | Test   | Fix                                       |
| ------------------------------- | ------ | ----------------------------------------- |
| Empty boundary                  | Test 1 | Add `Container()` or `Component()` inside |
| Alias redeclared                | Test 2 | Remove duplicate declarations             |
| Component outside boundary      | Test 3 | Wrap in `Container_Boundary()`            |
| Relationship to undefined alias | Test 4 | Check element declared with same alias    |
| Tag used but undefined          | Test 5 | Add `AddElementTag("tag", ...)`           |
| Icon macro invalid              | Test 6 | Use valid FAx*, DEVx*, or MAT\_ prefix    |
| No legend displayed             | Test 7 | Add `SHOW_LEGEND()`                       |
| Duplicate aliases               | TDD A  | Keep first, remove later declarations     |
| Boundary with only Rel()        | TDD B  | Add substantive element definitions       |
| Wrong element type at level     | TDD C  | Move to correct C4 level or remove        |
| Syntax error in diagram         | TDD D  | Fix bracket/quote/macro syntax            |

---

## 🎯 Validation Workflows

### Workflow 1: Manual Validation (Without Agent)

```
1. Run Test 1-7 checklist
2. Run TDD A-D self-tests
3. Check all 15 pre-flight items
4. Export and verify rendering
5. Submit for review
```

### Workflow 2: Agent-Assisted (With Self-Healing)

```
1. Provide diagram request to @plantuml-architect
2. Agent runs 5-step pipeline with auto-validation
3. If validation fails:
   - Apply Fix A/B/C/D/E (attempt 1)
   - Re-validate
   - If still fails, apply Fix (attempt 2)
   - If still fails, fallback to golden template + mark as AUTO-HEALED
4. Return guaranteed-valid diagram
```

### Workflow 3: Team Review (Post-Generation)

```
1. Run pre-flight checklist (15 items)
2. Verify architectural decisions
3. Validate against organizational standards
4. Cross-reference with existing diagrams
5. Approve for publication
```

---

## 🔗 Related Resources

- **Agent Documentation**: [PlantUML Architect Agent](../agents/plantuml-architect.agent.md)
- **Branding Skill**: [PlantUML C4 Branding](../skills/plantuml-c4-branding/SKILL.md)
- **Icon Sprites**: [tupadr3 plantuml-icon-font-sprites v3.0.0](https://github.com/tupadr3/plantuml-icon-font-sprites)
- **C4 Model**: [c4model.com](https://c4model.com)
- **PlantUML**: [plantuml.com](https://plantuml.com)

Before requesting a diagram from the agent, verify:

- [ ] **Diagram Type Selected**: Context? Container? Component? Dynamic? Deployment? Sequence?
- [ ] **Branding Specified**: NHS? Welsh? GPW? Custom colors provided?
- [ ] **Azure Services Identified**: List all Azure resources (Functions, App Service, SQL, Cosmos, etc.)
- [ ] **Boundary Strategy Defined**: What logical groupings will boundaries represent? (UI Tier, API Tier, Data Tier, etc.)
- [ ] **Stakeholder Audience**: Who will view this diagram? (exec, architects, developers, ops?)
- [ ] **Existing Diagrams Referenced**: Is this part of a multi-level set (Context→Container→Component)?

---

## Post-Generation Validation

### ✅ HARD RULE: Boundary Integrity

**Check Every Boundary:**

```
For each Container_Boundary, System_Boundary, Component_Boundary, or Deployment_Node:

[ ] Boundary contains ≥1 substantive element (not just Rel() calls)
[ ] Element is real: Container(), System(), Component(), Node() – NOT just an alias reference
[ ] Boundary name accurately describes contents
[ ] No orphaned boundaries (every boundary referenced from outside)
[ ] Cross-boundary relationships are explicit and labeled meaningfully
```

**Red Flags - REJECT if Present:**

- ❌ Empty boundary with only `Rel(outer_alias, boundary_item, "label")`
- ❌ Boundary named "Layer" with only data references, no definitions
- ❌ Nested boundaries with no element definitions at parent level

---

### 📋 Syntax Validation Checklist

- [ ] **PlantUML Tags**
  - [ ] `@startuml` and `@enduml` present and matched
  - [ ] Diagram title or identifier present
- [ ] **Library Includes**

  - [ ] `!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_[TYPE].puml` present
  - [ ] Correct C4 library imported (Context, Container, Component, Dynamic, Deployment, Sequence)

- [ ] **Bracket & Parenthesis Matching**

  - [ ] All `{` have matching `}`
  - [ ] All `(` have matching `)`
  - [ ] No trailing commas in parameter lists

- [ ] **Quotes & Escaping**

  - [ ] All string parameters wrapped in quotes: `"Label"`
  - [ ] No unescaped special characters in labels
  - [ ] Hex colors in quotes: `$bgColor="#005EB8"`

- [ ] **Macro Syntax**
  - [ ] All elements follow macro pattern: `Macro(alias, "Label", ?param, ?param)`
  - [ ] Relationship syntax correct: `Rel(from, to, "label", ?tech)`
  - [ ] No malformed `AddElementTag()`, `AddContainerTag()` definitions

---

### 🏗️ Architectural Validation Checklist

- [ ] **Element Definitions**

  - [ ] Every element has a non-empty label (2nd parameter)
  - [ ] Person, System, Container, Component clearly distinguished
  - [ ] Technology/language specified for compute containers (Node.js, Python, Java, C#)
  - [ ] Database type specified (SQL, NoSQL, Cache, Queue)

- [ ] **Relationships**

  - [ ] Every relationship has a descriptive label (NOT generic "uses")
  - [ ] Descriptive verbs used: "queries", "publishes", "authenticates", "transforms", "stores"
  - [ ] Technology/protocol documented where relevant (HTTP, AMQP, SQL/JDBC, gRPC)
  - [ ] Bidirectional relationships (`BiRel()`) used appropriately (not over-used)

- [ ] **Boundary Architecture**

  - [ ] Each boundary represents logical architectural tier/zone
  - [ ] Boundary names convey purpose (API Tier, Data Layer, Security Zone)
  - [ ] Elements within boundary are cohesive (don't mix UI + Database in same boundary)
  - [ ] Cross-boundary relationships document external dependencies

- [ ] **Data Flow Clarity**
  - [ ] Relationships flow top-down or left-right consistently
  - [ ] Synchronous vs. asynchronous patterns clear (direct Rel vs. Queue intermediary)
  - [ ] No circular dependencies without explanation
  - [ ] External integrations clearly marked (`System_Ext`)

---

### 🎨 Branding & Tag Validation Checklist

- [ ] **Branding Tags Applied**

  - [ ] All primary elements tagged with organization brand (nhs, welsh, gpw, custom)
  - [ ] Tag definition present: `AddElementTag("brand", $bgColor="...", ...)`
  - [ ] Consistent branding across all diagram elements
  - [ ] External/3rd-party elements distinguished via `_Ext` variant or different tag

- [ ] **Environment Tags**

  - [ ] Production elements tagged: `$tags="prod"`
  - [ ] Staging/Dev environments tagged: `$tags="staging"`, `$tags="dev"`
  - [ ] Deprecated components tagged: `$tags="deprecated"` with removal note
  - [ ] Environment tags meaningful (not all prod or all dev)

- [ ] **Azure Service Tags**

  - [ ] Azure services tagged with `azure_*` tags (azure-func, azure-sql, azure-apim, etc.)
  - [ ] Sprite icons applied: `$sprite="azurefunctions"`, `$sprite="azuresqldatabase"`
  - [ ] Azure service tags define sprite and tech info

- [ ] **Custom Tags**
  - [ ] All custom tags used in diagram are defined via `AddElementTag()` or `AddContainerTag()`
  - [ ] No orphaned tag references
  - [ ] Tags follow convention (lowercase, hyphen-separated)

---

### 📊 Legend & Visualization Checklist

- [ ] **Legend Generation**

  - [ ] `SHOW_LEGEND()` or `LAYOUT_WITH_LEGEND()` present
  - [ ] Legend displays all custom tags
  - [ ] Legend is readable (≤15 tags recommended)
  - [ ] Legend text is descriptive (e.g., "Production", "Azure Function", not just tag names)

- [ ] **Color & Contrast**

  - [ ] All hex colors valid format: `#RRGGBB`
  - [ ] Text color contrasts with background (4.5:1 WCAG AA minimum)
  - [ ] Branding colors consistent across diagram
  - [ ] External elements visually distinct (different color palette or border)

- [ ] **Sprites & Icons**

  - [ ] Sprites referenced exist in included libraries
  - [ ] Sprite scale reasonable (0.5 – 1.5 typical)
  - [ ] Custom sprites inlined or properly sourced
  - [ ] No overlapping sprites or visual clutter

- [ ] **Layout Clarity**
  - [ ] Layout chosen matches purpose: TOP_DOWN, LEFT_RIGHT, LANDSCAPE
  - [ ] Elements flow logically (no crossing lines where possible)
  - [ ] Boundaries don't overlap unexpectedly
  - [ ] Relationships labeled with sufficient spacing (no label overlap)

---

### 🔗 Multi-Diagram Consistency Checklist

For multi-level diagrams (Context → Container → Component):

- [ ] **Naming Consistency**

  - [ ] Aliases match across diagrams (e.g., `apiSystem` referenced in Context, decomposed in Container)
  - [ ] Element labels consistent (e.g., "User Portal" in Context = "Web Portal" in Container)
  - [ ] Same branding palette applied across all diagrams

- [ ] **Relationship Consistency**

  - [ ] Cross-diagram relationships documented (e.g., Container X inside System Y from Context)
  - [ ] No contradictory relationships between levels
  - [ ] External integrations consistent across all diagrams

- [ ] **Version Traceability**
  - [ ] All diagrams dated or versioned
  - [ ] Architectural decisions tagged/documented (e.g., `$tags="v2.0+approved"`)
  - [ ] Deprecated components clearly marked in all relevant diagrams

---

### 🚀 Production Readiness Checklist

Before publishing/presenting the diagram:

- [ ] **Compilation Verified**

  - [ ] Diagram renders without PlantUML errors
  - [ ] All elements visible (no hidden/clipped content)
  - [ ] SVG or PNG export clean (no artifacts)

- [ ] **Accessibility**

  - [ ] Diagram works in color-blind modes (don't rely on color alone)
  - [ ] Text labels readable at print size (8pt minimum)
  - [ ] High contrast for projected presentations

- [ ] **Documentation**

  - [ ] Diagram title/caption provided
  - [ ] Stakeholder audience identified
  - [ ] Assumptions documented (e.g., "assumes v2.0 architecture")
  - [ ] Related diagrams cross-referenced

- [ ] **Export & Distribution**
  - [ ] SVG exported for scalability (presentations)
  - [ ] PNG exported for documentation (confluence, wiki)
  - [ ] Source .puml file committed to repository
  - [ ] Version control history tracked

---

## Common Issues & Remediation

| Issue                 | Cause                                 | Fix                                                |
| --------------------- | ------------------------------------- | -------------------------------------------------- |
| Empty boundaries      | Only Rel() calls, no definitions      | Add Container() or Component() elements            |
| Orphaned tags         | Tag used but not defined              | Run `AddElementTag()` before using                 |
| Generic relationships | "uses", "talks to" labels             | Replace with verbs: "queries", "publishes"         |
| Cluttered legend      | Too many custom tags                  | Consolidate related tags; remove non-essential     |
| Poor contrast         | Light text on light background        | Use `$fontColor="#000000"` for contrast            |
| Sprite not rendering  | Sprite path wrong or library missing  | Verify library include; test sprite name           |
| Circular dependencies | Architectural loop without mediation  | Add queue/event bus intermediary; explain in docs  |
| Inconsistent branding | Different tags/colors across diagrams | Apply same `AddElementTag()` block to all diagrams |

---

## Team Best Practices

### 1. **Establish Diagram Standards**

- Agree on branding palette (NHS, Welsh, GPW, or custom)
- Define boundary naming convention (e.g., "API Tier", "Data Layer", never just "Layer")
- Document environment tag usage (prod, staging, dev, deprecated)

### 2. **Version Control Diagrams**

```bash
git add diagrams/
git commit -m "Add Container diagram for v2.0 architecture"
```

### 3. **Cross-Reference in Documentation**

Link diagrams in architecture decision records (ADRs):

```markdown
## Architecture Decision Record: Microservices Split

See: `diagrams/container-v2.0.puml` for system architecture
```

### 4. **Review Process**

- Architect reviews for design correctness
- Tech lead reviews for technical accuracy
- Product owner reviews for business logic alignment

### 5. **Update Cadence**

- Review diagrams quarterly
- Update for major architectural changes
- Mark deprecated components with sunset date

---

## Quick Reference: Tag Combinations

### Common Multi-Tag Patterns

```plantuml
' Production critical Azure service
$tags="prod+critical+azure-func+nhs"

' Staging environment, planned deprecation
$tags="staging+deprecated+v1.0"

' Development API service with GPW branding
$tags="dev+gpw+azure-appservice"

' External 3rd-party system
$tags="external+v2.0+api"
```

---

## Validation Automation Script

**Optional: Use this to scan diagrams for common issues:**

```bash
#!/bin/bash
# Scan PlantUML diagram for validation issues

DIAGRAM=$1

# Check for empty boundaries
if grep -q 'Boundary.*{[[:space:]]*}' "$DIAGRAM"; then
  echo "❌ Empty boundary found"
fi

# Check for undefined tags
TAGS=$(grep -o '\$tags="[^"]*"' "$DIAGRAM" | sort -u)
for TAG in $TAGS; do
  if ! grep -q "AddElementTag.*$TAG" "$DIAGRAM"; then
    echo "⚠️  Tag $TAG used but not defined"
  fi
done

# Check for generic relationship labels
if grep -q 'Rel.*"uses"' "$DIAGRAM"; then
  echo "⚠️  Generic relationship label 'uses' found - consider more specific verb"
fi

echo "✓ Scan complete"
```

---

**Use this checklist before submitting diagrams for architecture review. Questions? Refer to [PlantUML C4 Branding Skill](../../skills/plantuml-c4-branding/SKILL.md) or agent documentation.**
