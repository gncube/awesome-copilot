# PlantUML Architect - Diagram Validation Checklist

A comprehensive checklist for teams validating PlantUML diagrams generated with the **PlantUML Architect** agent, ensuring architectural substance, boundary integrity, and production readiness.

---

## Pre-Generation Checklist

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

| Issue | Cause | Fix |
|-------|-------|-----|
| Empty boundaries | Only Rel() calls, no definitions | Add Container() or Component() elements |
| Orphaned tags | Tag used but not defined | Run `AddElementTag()` before using |
| Generic relationships | "uses", "talks to" labels | Replace with verbs: "queries", "publishes" |
| Cluttered legend | Too many custom tags | Consolidate related tags; remove non-essential |
| Poor contrast | Light text on light background | Use `$fontColor="#000000"` for contrast |
| Sprite not rendering | Sprite path wrong or library missing | Verify library include; test sprite name |
| Circular dependencies | Architectural loop without mediation | Add queue/event bus intermediary; explain in docs |
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
