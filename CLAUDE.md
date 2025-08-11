# Ignition Core Certification Study Project

## Project Context
This workspace is set up for studying and passing the Ignition Core Certification exam. It includes a Docker-based Ignition environment with two gateways and PostgreSQL for hands-on practice.

## Environment Overview

### Docker Setup
- **Gateway 1** (gw1): Port 8188 - Primary for Vision development
- **Gateway 2** (gw2): Port 8288 - Primary for Perspective development  
- **Database**: PostgreSQL on port 5432
  - User: ignition_user
  - Password: ignition_password
  - Database: ignition_db
- **Network**: ignition_network (bridge mode)

### Access Points
- Gateway 1 Web UI: http://localhost:8188 or http://gw1.local:8188
- Gateway 2 Web UI: http://localhost:8288 or http://gw2.local:8288
- Both gateways accessible simultaneously in same browser (different ports)

### DNS Configuration
- dnsmasq configured with `address=/.local/127.0.0.1`
- Enables .local domain resolution for friendly gateway names

## Certification Scope
The Core Certification covers 9 main topics with 115 total subtopics:
1. Ignition Platform / Gateway Basics (7 subtopics)
2. Vision (13 subtopics)
3. Perspective (29 subtopics)
4. Tags (20 subtopics)
5. Security (6 subtopics)
6. Reporting (9 subtopics)
7. Tag History (7 subtopics)
8. Transaction Groups (8 subtopics)
9. Alarming (16 subtopics)

## Study Approach
- **70% Hands-On**: Build real projects in Docker environment
- **20% Video Learning**: Inductive University courses
- **10% Documentation**: Reference and notes

Recommended to use Gateway 1 for Vision projects and Gateway 2 for Perspective, then connect them via Gateway Network for distributed architecture practice.

## Key Files
- `ignition-certification-topics.md`: Complete checklist of all 115 exam subtopics
- `ignition-study-plan.md`: Detailed 6-week study plan with projects
- `docker-compose.yml`: Docker configuration for Ignition environment
- `backup_volumes.sh`: Script for backing up gateway configurations

## Study Goals
1. Master all 115 certification subtopics
2. Build 5+ complete Ignition projects
3. Understand Vision vs Perspective architecture
4. Configure distributed gateway systems
5. Implement production-ready security
6. Design efficient tag structures with UDTs
7. Create alarm pipelines and notifications
8. Build reports and transaction groups

## Important Reminders
- Always practice in both Vision and Perspective
- Focus on expression bindings and transforms (heavily tested)
- Understand the differences between Tag History and Transaction Groups
- Practice with Gateway Network for distributed questions
- Security levels and zones are critical concepts
- UDT inheritance and nesting appear frequently on exam

## Quick Commands
```bash
# Start environment
docker-compose up -d

# Stop environment  
docker-compose down

# Backup volumes
./backup_volumes.sh

# View logs
docker-compose logs -f ignition1
docker-compose logs -f ignition2
```

## Resources
- Study Guide: https://training.inductiveautomation.com/core-certification/study-guide/
- Inductive University: https://www.inductiveuniversity.com/
- Documentation: https://docs.inductiveautomation.com/
- Forum: https://forum.inductiveautomation.com/