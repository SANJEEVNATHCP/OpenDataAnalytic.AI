# InsightOS

**AI-Powered Analytics Copilot**

InsightOS is an enterprise-grade analytics workspace that unifies data ingestion, cleaning, transformation, visualization, AI-assisted analytics, reporting, collaboration, governance, and knowledge management into a single platform.

## Overview

InsightOS empowers analysts, business users, researchers, and organizations to perform end-to-end analytics without switching between multiple tools. Unlike traditional dashboard tools like Power BI or Tableau, InsightOS is an **AI Copilot** that assists users throughout the complete analytics lifecycle while keeping humans in control.

### Core Philosophy

- **Human-in-the-Loop AI**: AI detects problems, suggests improvements, explains recommendations, and drafts outputs. Humans approve final actions.
- **Explainability**: Every AI action must be explainable, reviewable, auditable, reversible, and safe.
- **User Empowerment**: AI assists users—it does not replace human judgment.

## Key Features

### Data Management
- **Dataset Management** - Centralized data source management
- **Automatic Data Profiling** - AI-powered data discovery and analysis
- **Data Cleaning** - Intelligent data quality improvements with human oversight
- **Undo/Redo** - Full operation reversibility
- **Data Quality Scoring** - Automated quality metrics

### Analytics & Insights
- **Natural Language Analytics** - Query data using plain English
- **SQL Generation** - AI-assisted SQL query creation
- **Power BI DAX Generation** - Generate DAX expressions automatically
- **Tableau Calculation Generation** - Create Tableau calculations
- **Visualization Recommendations** - AI-suggested chart types
- **Insight Generation** - Automated pattern detection and insights
- **Executive Summary Generation** - Auto-generated business summaries

### Enterprise Features
- **Dashboard Builder** - Drag-and-drop dashboard creation
- **Knowledge Hub** - Centralized analytics knowledge base
- **Version Control** - Track data and configuration changes
- **Data Lineage** - Understand data transformations
- **Semantic Layer** - Unified business metrics layer
- **Team Collaboration** - Real-time collaboration features
- **Role-Based Access Control** - Fine-grained permissions
- **Audit Trail** - Complete action logging
- **Report Generator** - Professional report creation

### Advanced Capabilities
- **Plugin System** - Extend functionality with custom plugins
- **Offline Mode** - Work without internet connectivity
- **Local AI Support** - Run AI models locally
- **Cloud AI Support** - Leverage cloud-based AI services
- **API Provider Manager** - Integrate with multiple data sources
- **Anomaly Detection** - Automatic outlier detection
- **Business KPI Explorer** - KPI tracking and monitoring

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/insightos/insightos.git
cd insightos

# Install dependencies
npm install
pip install -r backend/requirements.txt

# Start the development server
npm run dev
```

### Basic Usage

1. **Create a Project** - Start a new analytics project
2. **Upload Data** - Import datasets from various sources
3. **Profile Data** - AI automatically profiles your data
4. **Clean Data** - Use AI suggestions to clean your data
5. **Analyze** - Ask questions in natural language
6. **Visualize** - Get AI-recommended visualizations
7. **Collaborate** - Share insights with your team
8. **Report** - Generate professional reports

## Documentation

Complete documentation is available in the `docs/` directory:

- **[01_Product](docs/01_Product/)** - Product vision, roadmap, and market analysis
- **[02_Requirements](docs/02_Requirements/)** - Functional and non-functional requirements
- **[03_Architecture](docs/03_Architecture/)** - System and component architecture
- **[04_AI](docs/04_AI/)** - AI agents and strategies
- **[05_Database](docs/05_Database/)** - Data model and database design
- **[06_API](docs/06_API/)** - REST API documentation
- **[07_UI](docs/07_UI/)** - User interface specifications
- **[08_Security](docs/08_Security/)** - Security architecture and compliance
- **[09_Testing](docs/09_Testing/)** - Testing strategies and procedures
- **[10_Development](docs/10_Development/)** - Development standards and workflows
- **[11_Open_Source](docs/11_Open_Source/)** - Community and governance guidelines
- **[12_Diagrams](docs/12_Diagrams/)** - Architecture and system diagrams

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│              InsightOS Desktop (Tauri)              │
│  ┌──────────────────────────────────────────────┐   │
│  │         React Frontend (TypeScript)          │   │
│  │  - Dashboard | Projects | Dataset Manager   │   │
│  │  - Data Cleaner | Visualizations | Reports  │   │
│  └────────────────┬─────────────────────────────┘   │
└───────────────────┼─────────────────────────────────┘
                    │ IPC / HTTP
┌───────────────────┼─────────────────────────────────┐
│  ┌────────────────▼──────────────────────────────┐  │
│  │   Python Backend (FastAPI)                   │  │
│  │  - Data Processing | AI Agents | Database    │  │
│  │  - API Endpoints | Authentication            │  │
│  └────────────┬───────────────────────────────┬─┘  │
│               │                               │     │
│  ┌────────────▼────────┐    ┌────────────────▼─┐   │
│  │  SQLite Database    │    │  AI Services     │   │
│  │  - Metadata         │    │  - Local LLMs    │   │
│  │  - Audit Logs       │    │  - Cloud APIs    │   │
│  └─────────────────────┘    └──────────────────┘   │
└─────────────────────────────────────────────────────┘
```

## AI Agents

InsightOS includes specialized AI agents for different analytics tasks:

| Agent | Responsibility |
|-------|-----------------|
| **Data Profiling AI** | Analyze data structure, types, and quality |
| **Cleaning AI** | Suggest and apply data cleaning transformations |
| **Query AI** | Generate SQL and other query languages |
| **Visualization AI** | Recommend appropriate chart types |
| **Insight AI** | Detect patterns and generate insights |
| **Reporting AI** | Create executive summaries and reports |
| **Knowledge AI** | Manage and retrieve analytics knowledge |
| **Validation AI** | Verify data quality and consistency |
| **Forecasting AI** | (Future) Predict future trends |
| **Governance AI** | (Future) Ensure compliance and best practices |

## Technology Stack

### Frontend
- **React 18+** - UI framework
- **TypeScript** - Type safety
- **Tauri** - Desktop application framework
- **Tailwind CSS** - Styling
- **Vite** - Build tool

### Backend
- **Python 3.10+** - Server-side runtime
- **FastAPI** - Web framework
- **SQLAlchemy** - ORM
- **SQLite** - Default database
- **Langchain** - AI framework
- **Anthropic Claude / OpenAI** - LLM providers

### AI/ML
- **Langchain** - LLM orchestration
- **Ollama** - Local LLM support
- **Scikit-learn** - ML algorithms
- **Pandas** - Data processing
- **NumPy** - Numerical computing

### DevOps
- **Docker** - Containerization
- **GitHub Actions** - CI/CD
- **GitHub** - Version control

## Getting Involved

We welcome contributions from developers, designers, data scientists, and product managers.

### For Contributors
- Read [CONTRIBUTING.md](CONTRIBUTING.md)
- Check [ROADMAP.md](ROADMAP.md) for priorities
- Review [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)

### For Community
- Join our [GitHub Discussions](https://github.com/insightos/insightos/discussions)
- Report issues on [GitHub Issues](https://github.com/insightos/insightos/issues)
- Follow us on [Twitter/X](https://twitter.com/insightos)

## License

InsightOS is released under the [MIT License](LICENSE).

## Security

If you discover a security vulnerability, please email security@insightos.dev instead of using the issue tracker. See [SECURITY.md](SECURITY.md) for details.

## Roadmap

### Current Release (v1.0)
- Core data management
- Basic AI agents (Data Profiling, Cleaning, Query)
- Dashboard builder
- Team collaboration

### Upcoming (v1.1 - v2.0)
- Advanced forecasting
- Governance AI
- Custom plugin development
- Enhanced reporting
- Mobile support

See [ROADMAP.md](ROADMAP.md) for detailed timeline.

## Support

- 📖 **Documentation**: [docs/](docs/)
- 💬 **Community**: [GitHub Discussions](https://github.com/insightos/insightos/discussions)
- 🐛 **Issues**: [GitHub Issues](https://github.com/insightos/insightos/issues)
- 📧 **Email**: support@insightos.dev

---

**InsightOS** - Making analytics accessible to everyone.
