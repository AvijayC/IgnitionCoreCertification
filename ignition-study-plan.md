# Ignition Core Certification Study Plan

## Study Strategy Overview

### Time Allocation
- **70% Hands-On Practice**: Build real projects in your Docker environment
- **20% Video Learning**: Inductive University courses
- **10% Documentation**: Reference and note-taking

## 6-Week Study Schedule

### Week 1-2: Foundation (Must Master First)
#### Focus Areas
1. **Ignition Platform/Gateway Basics**
   - Install and configure Ignition (practice with Docker setup)
   - Gateway and project backups (automate with backup_volumes.sh)
   - Database connections (PostgreSQL setup)
   - Device connections (simulate with OPC-UA)
   - Licensing concepts
   - Gateway Network setup between gw1 and gw2

2. **Tags**
   - Create all tag types (OPC, Memory, Expression, Query)
   - Build UDTs for a production line simulation
   - Practice tag paths and provider scopes
   - Configure tag groups with different scan rates
   - Implement tag security

3. **Security**
   - Set up multiple user sources
   - Create security levels (Operator, Supervisor, Admin)
   - Configure security zones
   - Practice project security inheritance

#### Hands-On Project
Build a "Production Line Simulator" with:
- UDTs for machines (Motor, Valve, Sensor)
- Nested UDTs for production cells
- Security levels for different operations
- Gateway Network communication

### Week 3-4: Visualization Modules
#### Vision (Gateway 1: Port 8188)
1. **Components & Bindings**
   - Master all 4 binding types (Tag, Property, Expression, Indirect)
   - Create templates with parameters
   - Build navigation strategy (Tab Strip, Tree View, Buttons)
   - Practice window management

2. **Vision Projects**
   - Main dashboard with templates
   - Popup windows for details
   - Client tags for session data
   - Custom properties on windows

#### Perspective (Gateway 2: Port 8288)
1. **Components & Views**
   - All binding types plus Transforms
   - View parameters and embedded views
   - Session properties vs page properties
   - Flex containers and responsive design

2. **Perspective Features**
   - Style classes and themes
   - Page URLs and navigation
   - Docked views for navigation
   - Mobile-responsive layouts

#### Comparison Project
Build the SAME dashboard in both Vision and Perspective to understand:
- Component differences
- Binding approach variations
- Security implementation
- Performance considerations

### Week 5: Data Management
#### Tag History
1. Configure history on multiple tags
2. Set up different sample modes (On Change, Periodic, Min/Max)
3. Create trend displays in Vision and Perspective
4. Practice history bindings with aggregation modes

#### Transaction Groups
1. **Historical Groups**: Log data every minute
2. **Trigger-Based Groups**: Log on events
3. **Block Groups**: Multiple items at once
4. **Stored Procedure Groups**: Call database procedures

#### Reporting
1. Design reports with dynamic data
2. Schedule reports (daily, weekly)
3. Email distribution setup
4. Parameter passing from Vision/Perspective

#### Database Project
Create a "Production Tracking System":
- Historical data collection via Transaction Groups
- Tag History for real-time trends
- Reports for shift summaries
- SQL queries for custom displays

### Week 6: Advanced Features & Review
#### Alarming
1. **Configuration**
   - Tag alarms with setpoints
   - Dynamic alarms on UDTs
   - Alarm priorities and delays

2. **Notification Pipeline**
   - Email notifications
   - Escalation chains
   - On-call rosters
   - Acknowledgment requirements

3. **Visualization**
   - Alarm Status Table configuration
   - Alarm Journal queries
   - Filtering and sorting

#### Final Project
Build a complete "Plant Monitoring System":
- 10+ UDT instances with alarms
- Vision and Perspective interfaces
- Historical trending
- Alarm management
- Scheduled reports
- Multi-level security

## Daily Study Routine (2.5 hours)

### Morning Session (1.5 hours)
```
30 min: Watch Inductive University video on topic
60 min: Implement what you learned in Docker environment
```

### Evening Session (1 hour)
```
30 min: Document learnings and create notes
30 min: Review previous topics (spaced repetition)
```

## Key Concepts to Master

### Expression Functions (Memorize These)
- String manipulation: `concat()`, `substring()`, `len()`
- Logic: `if()`, `case()`, `switch()`
- Date/Time: `now()`, `dateFormat()`, `addDays()`
- Tag functions: `tag()`, `tagPath()`, `isGood()`
- Math: `round()`, `floor()`, `ceil()`, `abs()`

### Common Binding Patterns
1. **Indirect Tag Binding**: `[default]Line{1}/Machine{2}/Status`
2. **Expression with Tag**: `if({[default]MyTag} > 100, "High", "Normal")`
3. **Transform**: Map values, format strings, add logic

### Critical Differences to Understand
| Feature | Vision | Perspective |
|---------|---------|-------------|
| Client Tags | Yes | No (use Session Props) |
| Templates | Yes | No (use Views) |
| Scripting | Client & Gateway | Gateway Only |
| Mobile | Limited | Full Support |
| Components | Swing-based | Web-based |

## Testing Your Knowledge

### Self-Assessment Checkpoints

#### Week 2 Checkpoint
- [ ] Create UDT with 3 levels of nesting
- [ ] Set up Gateway Network with bidirectional communication
- [ ] Configure 3 security levels with different component access

#### Week 4 Checkpoint
- [ ] Build same interface in Vision and Perspective
- [ ] Create template/view with 5+ parameters
- [ ] Implement navigation with security restrictions

#### Week 6 Checkpoint
- [ ] Configure complete alarm pipeline with escalation
- [ ] Create report with parameters from UI
- [ ] Build transaction group for recipe management

## Exam Preparation Tips

### Two Weeks Before Exam
1. **Stop learning new content** - Focus on reinforcement
2. **Rebuild projects from memory** - Tests understanding
3. **Time yourself** on common tasks:
   - Creating a UDT: < 5 minutes
   - Building a basic view: < 10 minutes
   - Configuring an alarm: < 3 minutes

### Common Exam Scenarios
- Troubleshooting binding errors
- Identifying correct binding type for scenario
- Security level inheritance issues
- Gateway Network architecture questions
- Performance optimization choices

### Quick Reference Sheet (Create Your Own)
- Tag path syntax
- Expression function signatures
- System function categories
- Binding type decision tree
- Component property names

## Resources

### Primary Resources
1. **Inductive University** (Free)
   - Core Certification Learning Path
   - Practice exercises included

2. **Your Docker Environment**
   - Gateway 1: http://localhost:8188
   - Gateway 2: http://localhost:8288
   - Database: PostgreSQL on 5432

3. **Documentation**
   - https://docs.inductiveautomation.com
   - Keep open during practice

### Community Resources
- Ignition Forum for questions
- YouTube tutorials for specific topics
- GitHub for example projects

## Success Metrics

Track your progress:
- [ ] Completed all 115 subtopics in checklist
- [ ] Built 5+ complete projects
- [ ] Can explain Vision vs Perspective tradeoffs
- [ ] Comfortable with all binding types
- [ ] Understand Tag History vs Transaction Groups
- [ ] Can design multi-level security system
- [ ] Created working alarm notification pipeline

## Final Tips

1. **Learn by Building**: Don't just watch videos - implement immediately
2. **Use Both Gateways**: Practice distributed architecture
3. **Document Everything**: Your notes are exam gold
4. **Focus on Fundamentals**: Tags and bindings are 40% of exam
5. **Practice Speed**: Exam is timed, efficiency matters

Remember: The exam tests practical application, not memorization. Build real projects!