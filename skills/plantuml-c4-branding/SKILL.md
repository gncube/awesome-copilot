---
name: "plantuml-c4-branding"
description: "Reusable PlantUML C4 branding templates, tag libraries, and styling configurations for consistent enterprise diagram generation. Includes NHS, Welsh Government, GPW, and custom brand starter templates."
---

# PlantUML C4 Branding & Tag Libraries

A comprehensive skill providing production-ready branding templates and pre-built tag libraries for consistent C4-PlantUML diagram styling across organizational standards.

## Overview

This skill packages:

- **Pre-configured branding palettes** (NHS, Welsh, GPW, and custom templates)
- **Reusable tag definitions** for common diagram elements
- **Azure service icon mappings** with consistent styling
- **Starter templates** for rapid diagram creation

## Available Branding Palettes

### 1. NHS (National Health Service)

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

' NHS Branding - Healthcare Blue
AddElementTag("nhs", $bgColor="#005EB8", $fontColor="#FFFFFF", $borderColor="#003087")
AddElementTag("nhs-external", $bgColor="#0078D4", $fontColor="#FFFFFF", $borderColor="#005EB8")
AddElementTag("nhs-data", $bgColor="#003087", $fontColor="#FFFFFF", $borderColor="#001a47")

' NHS Relationships
AddRelTag("nhs-secure", $lineColor="#005EB8", $textColor="#005EB8", $lineStyle="BoldLine()")
AddRelTag("nhs-audit", $lineColor="#003087", $textColor="#003087", $lineStyle="DashedLine()")
```

**Usage**: Healthcare systems, patient portals, clinical data platforms

---

### 2. Welsh Government (Llywodraeth Cymru)

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

' Welsh Digital / Public Sector Green
AddElementTag("welsh", $bgColor="#007A33", $fontColor="#FFFFFF", $borderColor="#005A24")
AddElementTag("welsh-external", $bgColor="#009640", $fontColor="#FFFFFF", $borderColor="#007A33")
AddElementTag("welsh-public", $bgColor="#005A24", $fontColor="#FFFFFF", $borderColor="#003d19")

' Welsh Relationships
AddRelTag("welsh-citizen", $lineColor="#007A33", $textColor="#007A33", $lineStyle="SolidLine()")
AddRelTag("welsh-integration", $lineColor="#009640", $textColor="#009640", $lineStyle="DottedLine()")
```

**Usage**: Public sector services, civic platforms, government integrations

---

### 3. GPW (General Purpose Web/Enterprise)

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

' GPW Enterprise Purple/Blue
AddElementTag("gpw", $bgColor="#4F46E5", $fontColor="#FFFFFF", $borderColor="#3730A3")
AddElementTag("gpw-external", $bgColor="#6366F1", $fontColor="#FFFFFF", $borderColor="#4F46E5")
AddElementTag("gpw-system", $bgColor="#3730A3", $fontColor="#FFFFFF", $borderColor="#2d2061")

' GPW Relationships
AddRelTag("gpw-api", $lineColor="#4F46E5", $textColor="#4F46E5", $lineStyle="BoldLine()")
AddRelTag("gpw-legacy", $lineColor="#3730A3", $textColor="#3730A3", $lineStyle="DashedLine()")
```

**Usage**: Enterprise applications, SaaS platforms, corporate systems

---

### 4. Custom Brand Template

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Context.puml

' === CUSTOMIZE THESE COLORS ===
!$PRIMARY_COLOR = "#1a5490"        ' Primary brand color
!$PRIMARY_DARK = "#0d2942"         ' Darker variant
!$PRIMARY_LIGHT = "#4a8fd9"        ' Lighter variant
!$SECONDARY_COLOR = "#f39c12"      ' Accent color
!$NEUTRAL_BG = "#f5f5f5"           ' Neutral background

' Primary brand elements
AddElementTag("custom-primary", $bgColor=$PRIMARY_COLOR, $fontColor="#FFFFFF", $borderColor=$PRIMARY_DARK)
AddElementTag("custom-secondary", $bgColor=$SECONDARY_COLOR, $fontColor="#FFFFFF", $borderColor=$PRIMARY_COLOR)
AddElementTag("custom-neutral", $bgColor=$NEUTRAL_BG, $fontColor=$PRIMARY_COLOR, $borderColor=$PRIMARY_COLOR)

' Custom relationships
AddRelTag("custom-main", $lineColor=$PRIMARY_COLOR, $textColor=$PRIMARY_COLOR)
AddRelTag("custom-alert", $lineColor=$SECONDARY_COLOR, $textColor=$SECONDARY_COLOR, $lineStyle="BoldLine()")
```

**Usage**: Apply to your organization's brand colors and guidelines

---

## Azure Service Tag Library

### Pre-configured Azure Services

```plantuml
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

' === AZURE COMPUTE ===
AddContainerTag("azure-func", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurefunctions", $techn="Azure Function", $legendText="Serverless compute")
AddContainerTag("azure-aks", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurekubernetesservice", $techn="AKS", $legendText="Kubernetes cluster")
AddContainerTag("azure-appservice", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azureappservice", $techn="App Service", $legendText="Web/API hosting")
AddContainerTag("azure-logicapp", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurelogicapps", $techn="Logic App", $legendText="Workflow automation")

' === AZURE DATA ===
AddContainerTag("azure-sql", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azuresqldatabase", $techn="SQL Database", $legendText="Relational database")
AddContainerTag("azure-cosmos", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurecosmosdb", $techn="Cosmos DB", $legendText="NoSQL database")
AddContainerTag("azure-storage", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurestorage", $techn="Storage Account", $legendText="Blob/File storage")

' === AZURE INTEGRATION ===
AddContainerTag("azure-servicebus", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azureservicebus", $techn="Service Bus", $legendText="Message queue")
AddContainerTag("azure-eventhub", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azureeventhubs", $techn="Event Hub", $legendText="Event streaming")
AddContainerTag("azure-apim", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azureapimanagement", $techn="API Management", $legendText="API gateway")

' === AZURE SECURITY ===
AddContainerTag("azure-keyvault", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurekeyvault", $techn="Key Vault", $legendText="Secrets management")
AddContainerTag("azure-managed-id", $bgColor="#FFE6E6", $borderColor="#0078D4", $sprite="azureidentity", $techn="Managed Identity", $legendText="Identity management")

' === AZURE MONITORING ===
AddContainerTag("azure-appinsights", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azureapplicationinsights", $techn="App Insights", $legendText="APM & telemetry")
AddContainerTag("azure-monitor", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azuremonitor", $techn="Azure Monitor", $legendText="Monitoring & alerts")
```

---

## Element Type Tag Templates

### Person Roles

```plantuml
AddPersonTag("admin", $sprite="osa_user_audit", $fontColor="#d73027", $legendText="System Administrator")
AddPersonTag("developer", $sprite="osa_user_green", $fontColor="#1a9850", $legendText="Developer")
AddPersonTag("customer", $sprite="osa_user_blue", $fontColor="#0078D4", $legendText="End User / Customer")
AddPersonTag("support", $sprite="osa_user_orange", $fontColor="#f39c12", $legendText="Support Team")
```

### Container Types

```plantuml
AddContainerTag("web-app", $bgColor="#E3F2FD", $borderColor="#1976D2", $sprite="devicons/angular")
AddContainerTag("api", $bgColor="#F3E5F5", $borderColor="#7B1FA2", $sprite="devicons/java")
AddContainerTag("worker", $bgColor="#E8F5E9", $borderColor="#388E3C", $sprite="devicons/python")
AddContainerTag("database", $bgColor="#FCE4EC", $borderColor="#C2185B")
AddContainerTag("queue", $bgColor="#FFF3E0", $borderColor="#E65100")
```

### Environment Tags

```plantuml
AddElementTag("prod", $bgColor="#FFEBEE", $borderColor="#D32F2F", $legendText="Production")
AddElementTag("staging", $bgColor="#FFF3E0", $borderColor="#F57C00", $legendText="Staging")
AddElementTag("dev", $bgColor="#E8F5E9", $borderColor="#388E3C", $legendText="Development")
AddElementTag("deprecated", $bgColor="#ECEFF1", $borderColor="#78909C", $legendText="Deprecated (remove)")
```

---

## Starter Template: Azure + NHS Branding

```plantuml
@startuml NHS_Azure_System
!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

LAYOUT_WITH_LEGEND()

' NHS + Azure branding
AddElementTag("nhs", $bgColor="#005EB8", $fontColor="#FFFFFF", $borderColor="#003087")
AddContainerTag("azure-func", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azurefunctions")
AddContainerTag("azure-sql", $bgColor="#E6F2FF", $borderColor="#0078D4", $sprite="azuresqldatabase")

Person(patient, "Patient", "NHS Service User", $tags="nhs")

System_Boundary(nhs_sys, "NHS Digital Platform", $tags="nhs") {
    Container(api, "Patient API", "Azure Function", "REST API for patient data", $tags="azure-func")
    Container(portal, "Patient Portal", "React", "Web interface for appointment booking", $tags="web-app+nhs")
    ContainerDb(db, "Patient Records DB", "Azure SQL", "HIPAA-compliant data store", $tags="azure-sql")
}

System_Ext(gp, "GP Practice System", "External GP software")

Rel(patient, portal, "Books appointments via")
Rel(portal, api, "Queries patient data")
Rel(api, db, "Reads/writes HIPAA data")
Rel(api, gp, "Integrates with", "HL7/FHIR")

SHOW_LEGEND()
@enduml
```

---

## Usage Guide

### Importing Branding into Your Diagram

1. **Copy the branding block** (e.g., NHS or GPW) into your PlantUML diagram
2. **Apply tags to elements** using the `$tags` parameter:
   ```plantuml
   Container(myFunc, "My Function", "Azure Function", $tags="azure-func+nhs")
   ```
3. **Combine tags** with `+` to blend multiple styles:
   ```plantuml
   Person(admin, "Admin User", $tags="admin+nhs")
   ```

### Custom Organization Branding

1. Start with the "Custom Brand Template"
2. Replace `$PRIMARY_COLOR`, `$SECONDARY_COLOR`, etc. with your colors
3. Define element and relationship tags for your standards
4. Save as `branding-[organization].puml` for reuse

### Azure Service Mapping

- Match container technology to Azure service tags
- Example: Node.js API → `$tags="azure-appservice"` or `$tags="azure-func"`
- Example: SQL database → `$tags="azure-sql"`
- Layer branding + service tags for dual classification

---

## Best Practices

1. **Consistency Across Diagrams**: Use same branding palette for all Context/Container/Component diagrams
2. **Meaningful Tag Combinations**: `$tags="prod+critical"` documents both environment and priority
3. **Legend Clarity**: Auto-generated legends show all tag meanings—avoid redundant documentation
4. **Color Contrast**: Ensure 4.5:1 WCAG contrast ratio between text and background
5. **Sprite Attribution**: Include library sources (DevIcons, FontAwesome, Azure) in diagram footer

---

## File References

- **NHS Branding**: Healthcare Blue (#005EB8) aligned with [NHS Digital Service Manual](https://www.england.nhs.uk/)
- **Welsh Branding**: Public Sector Green (#007A33) aligned with [Welsh Government Brand Book](https://gov.wales/)
- **GPW Branding**: Enterprise Purple (#4F46E5) – generic corporate standard
- **C4-PlantUML Documentation**: [Official Repository](https://github.com/plantuml-stdlib/C4-PlantUML)
- **Azure Icons**: [Official Azure Architecture Icons](https://learn.microsoft.com/en-us/azure/architecture/icons/)

---

## Tips & Troubleshooting

**Q: Custom colors not rendering?**

- A: Ensure hex color codes are wrapped in quotes: `$bgColor="#FFFFFF"`

**Q: Tags not appearing in legend?**

- A: Use `SHOW_LEGEND()` instead of `LAYOUT_WITH_LEGEND()` for custom tags

**Q: Sprite not found?**

- A: Verify sprite library is included before sprite reference; use full URL for remote sprites

**Q: How to combine branding + environment tags?**

- A: Use `+` separator: `$tags="nhs+prod+critical"` (order doesn't matter)
