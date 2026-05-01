# GEMINI Instructions - 20261sre-vinicius-trentino

## Project Overview
This project aims to implement a **Reliable ETL (Extract, Transform, Load) Pipeline** for the Olist Marketplace. The primary objective is to process approximately 100,000 daily orders from CSV source files into a PostgreSQL analytical database while ensuring high reliability, data integrity, and observability.

### Main Technologies (Inferred/Planned)
- **Source:** CSV files
- **Destination:** PostgreSQL
- **Monitoring/Observability:** Grafana
- **Infrastructure:** AWS (EC2)

## Directory Overview
The project is currently in its early stages, primarily focused on requirements and problem definition.

- **`specs/`**: Contains documentation regarding the project's specifications, goals, and constraints.
- **`README.md`**: Basic entry point for the project.
- **`GEMINI.md`**: This file, providing context and instructions for AI interactions.

## Key Files
- **`specs/00_problem.md`**: The core specification document. it defines:
    - **Stakeholders**: Operation, Data Team, Dashboard Consumers, and SRE/Platform.
    - **Critical Flows**: Daily ingestion, Grafana dashboards, and SLA monitoring.
    - **Failure Modes**: Corrupted files, duplication (lack of idempotency), infrastructure failures, and database unavailability.
    - **Systemic Risks**: Silent data loss, stale data, and cost spikes.

## Development Strategy (TODO)
As the project evolves, the following areas will need to be addressed:
- **Ingestion Script**: Implementation of the ETL logic (likely in Python or similar).
- **Database Schema**: Definition of the PostgreSQL tables.
- **Idempotency**: Mechanisms to handle re-runs without duplicating data.
- **Observability**: Integration with Grafana and alerting for SLA breaches.
- **CI/CD & Infrastructure**: Deployment scripts (e.g., Terraform, CloudFormation) and automation.

## Building and Running
*No code implemented yet.*
- **TODO**: Define the commands for environment setup, running the ETL job, and executing tests.

## Development Conventions
*To be defined.*
- **TODO**: Establish coding standards, testing frameworks, and documentation practices once implementation begins.
