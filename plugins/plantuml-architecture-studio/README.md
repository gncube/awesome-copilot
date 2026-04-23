# PlantUML Architecture Studio

Professional architecture diagram generation and enterprise documentation plugin for GitHub Copilot, powered by C4-PlantUML with pre-built branding templates and enterprise standards.

## 🎯 Overview

**PlantUML Architecture Studio** is a comprehensive plugin combining specialized agents, reusable skills, and integration tools to enable rapid, consistent, presentation-quality architecture diagram creation across all organizational branding standards.

### Core Components

| Component                      | Purpose                                                                                |
| ------------------------------ | -------------------------------------------------------------------------------------- |
| **PlantUML Architect Agent**   | Expert diagram generation: all C4 types, Azure integration, custom branding            |
| **Architecture Expert Agents** | Senior architects (High-Level Big Picture, Architecture Design) for strategic planning |
| **C4 Branding Skill**          | Pre-configured tag libraries for NHS, Welsh Gov, GPW, and custom brands                |
| **Azure Visualizer Skill**     | Real-time Azure resource topology mapping and visualization                            |

## 🚀 Quick Start

### Create an Azure Architecture Diagram

```prompt
@plantuml-architect
Create a C4 Container diagram for our microservices platform:
- Web Portal (React)
- API (Node.js on App Service)
- Worker (Python in AKS)
- SQL Database
- Service Bus for async messaging

Use GPW branding (enterprise purple).
```

### Generate System Context with Custom Brand

```prompt
@plantuml-architect
System Context diagram for our healthcare platform using our custom branding:
- Brand color: #1a5490 (primary)
- Accent: #f39c12 (orange)
- Include: Patients, Doctors, GP Systems, NHS Integration
- Azure hosted (show cloud boundary)
```

### Multi-Level Architecture Flow

```prompt
@plantuml-architect
I need Context → Container → Component views for my e-commerce system.
Maintain consistency across all three diagrams.
Use NHS branding for the healthcare vendor component.
```

## 📊 Supported Diagram Types

- **System Context** – High-level system landscape
- **Container** – Application, database, queue, web tier breakdown
- **Component** – Internal microservice architecture
- **Dynamic** – Interaction sequences and workflows
- **Deployment** – Infrastructure, Kubernetes, cloud topology
- **Sequence** – C4-styled interaction flows

## 🎨 Pre-built Branding Systems

### NHS (Healthcare Blue)

Primary: `#005EB8` | Dark: `#003087`  
**Usage**: Healthcare platforms, patient portals, clinical systems

### Welsh Government (Public Sector Green)

Primary: `#007A33` | Dark: `#005A24`  
**Usage**: Government services, civic platforms, public sector

### GPW (Enterprise Purple)

Primary: `#4F46E5` | Dark: `#3730A3`  
**Usage**: Corporate SaaS, enterprise applications, business systems

### Custom Organization Branding

Define your own colors and style system  
**Usage**: Company-specific guidelines, proprietary branding

## 🔧 Branding Skill Features

The **plantuml-c4-branding** skill provides:

✅ **Complete tag libraries** for NHS, Welsh, GPW, and custom brands  
✅ **Azure service icons** with consistent styling (25+ services)  
✅ **Element type templates** (persons, containers, environments)  
✅ **Starter templates** for rapid diagram creation  
✅ **Best practices documentation**

### Quick Tag Application

```plantuml
Container(api, "API", "Node.js", $tags="azure-appservice+gpw+prod")
Container(db, "Database", "SQL", $tags="azure-sql+prod")
```

## 🏗️ Architecture Consistency

**Cross-diagram consistency** maintained automatically:

- **Naming Conventions**: Consistent aliases across Context → Container → Component
- **Visual Identity**: Same branding palette throughout documentation
- **Relationship Tracking**: Document how diagram layers connect
- **Version Control**: Tag elements for traceability across versions
- **Asset References**: Preserve architectural decisions and component roles

## 🔗 Azure Integration

Native integration with Azure services:

- **25+ Azure service icons** (Functions, AKS, SQL, Cosmos, Service Bus, etc.)
- **Real-time topology mapping** via Azure Resource Visualizer skill
- **Resource group visualization** for deployment diagrams
- **Managed Identity & RBAC** documentation

Example: Visualize existing resource group, then create deployment diagram:

```prompt
@azure-visualizer
Show me the topology of my prod resource group

@plantuml-architect
Now create a C4 Deployment diagram matching this topology with enterprise branding
```

## 📚 Common Use Cases

### 1. **Stakeholder Presentations**

Create Context diagrams with organizational branding for board/investor presentations.

### 2. **Technical Documentation**

Multi-level architecture (Context → Container → Component) with Azure components.

### 3. **Migration Planning**

Deployment diagrams showing current state (on-premises) → target state (Azure).

### 4. **Team Onboarding**

System Context + Container diagrams explaining platform architecture to new engineers.

### 5. **Compliance & Audit**

Diagrams with security zones, data flows, and compliance-tagged components.

### 6. **Solution Design**

Rapid prototyping of architectures during design sessions (sketch mode available).

## 🛠️ Agent Capabilities

### PlantUML Architect

- All C4 diagram types (Context, Container, Component, Dynamic, Deployment, Sequence)
- Branding system selection or custom color application
- Azure service icon integration
- Multi-diagram consistency management
- Sprite and custom styling

### High-Level Big Picture Architect

- Strategic architecture planning
- Enterprise-scale system design
- Compliance and security considerations
- Documentation review and refinement

### Architecture Expert

- Detailed design validation
- Technical debt assessment
- Modernization strategies
- Performance and scalability review

## 📖 Example Workflows

### Workflow 1: From Scratch to Presentation

```
1. @high-level-big-picture-architect: Plan the architecture
2. @plantuml-architect: Create System Context with GPW branding
3. @plantuml-architect: Create Container diagram
4. @arch-expert: Validate design decisions
5. Export diagrams for stakeholder presentation
```

### Workflow 2: Existing Azure Resources → Diagrams

```
1. @azure-visualizer: Map existing resource group
2. @plantuml-architect: Generate Deployment diagram matching topology
3. Apply NHS branding for healthcare compliance
4. Document data flows and integrations
```

### Workflow 3: Multi-Diagram Consistency

```
1. Create System Context (overall scope)
2. Create Container diagram (decompose into services)
3. Create Component diagram for critical service
4. Agent maintains naming, branding, and relationship consistency
5. All three linked and documented
```

## 🎓 Learning & Documentation

- **[C4-PlantUML Official Docs](https://github.com/plantuml-stdlib/C4-PlantUML)** – Full library reference
- **[C4 Model Guide](https://c4model.com/)** – Architecture patterns and examples
- **[PlantUML Syntax](https://plantuml.com/)** – Core diagram language
- **In-plugin Skill**: plantuml-c4-branding with templates and best practices

## ⚙️ Configuration & Customization

### Branding Template

Store organizational branding in a reusable file:

```plantuml
' branding-myorg.puml
!$PRIMARY_COLOR = "#1a5490"
!$SECONDARY_COLOR = "#f39c12"

AddElementTag("myorg", $bgColor=$PRIMARY_COLOR, $fontColor="#FFFFFF", $borderColor="...")
```

Include in diagrams:

```plantuml
!include branding-myorg.puml
```

### Custom Sprite Library

Define organization-specific icons:

```plantuml
!define COMPANY_ICONS https://internal-cdn.company.com/plantuml-icons/

!include COMPANY_ICONS/database-secured.puml
!include COMPANY_ICONS/workflow-approved.puml
```

## 🔒 Best Practices

✅ **Maintain naming consistency** across diagram levels  
✅ **Use standardized tags** (prod, dev, deprecated, critical)  
✅ **Document architectural decisions** in diagram descriptions  
✅ **Version diagrams** alongside code (commit to repo)  
✅ **Layer Azure services** with branding tags for dual classification  
✅ **Export to SVG** for presentation flexibility  
✅ **Review legends** to ensure all tags are meaningful

## 📝 License

MIT License – See [LICENSE](../../LICENSE) for details

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](../../CONTRIBUTING.md) for guidelines.

## 📞 Support

- **GitHub Issues**: Report bugs or request features
- **Discussions**: Share architecture patterns and branding templates
- **Documentation**: Check in-plugin skill references for detailed examples

---

**PlantUML Architecture Studio** – Architecture Diagrams, Enterprise Standards, Production Quality.
