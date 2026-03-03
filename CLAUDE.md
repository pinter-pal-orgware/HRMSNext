# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HRMSNext is a greenfield integrated Human Resource Management System (HRMS) designed for the Hungarian legal environment. The system handles multiple employment types under different Hungarian labor laws.

**Status**: Pre-implementation phase - currently contains design documentation only.

**Domain**: HR management with focus on Hungarian public and private sector employment regulations.

## Legal Context & Employment Types

This HRMS must support 7 distinct employment types, each governed by different Hungarian laws:

1. **Mt.** (Munka Törvénykönyve) - Private sector labor law
2. **Kjt.** (Közalkalmazottak) - Public employees in social/cultural sectors
3. **Púétv.** (Pedagógusok életpályája) - Teachers in public education
4. **Eszjtv.** (Egészségügyi szolgálat) - Healthcare workers
5. **Kit.** (Kormányzati igazgatás) - Government administration
6. **Kttv.** (Közszolgálati tisztviselők) - Local government officials
7. **Küt.** (Különleges jogállású szervek) - Special status organizations

The `employmentType` discriminator field is central to the entire system architecture - it determines applicable compensation rules, leave entitlements, work schedule types, position classifications, and qualification requirements.

## Architecture & Core Design

### Central Hub Pattern

**Employment** entity is the aggregation root of the entire HRMS. All operational entities (Compensation, Leave, TimeTracking) connect through Employment to Person and Organization. This ensures proper handling of multiple concurrent employments.

### Domain Structure (8 domains, 35 entities)

```
Person (5) ──┐
Organization (4) ─┼──> EMPLOYMENT (central hub) ──┬──> Compensation (5)
Position (4) ──┘                                   ├──> Qualification (5)
                                                   ├──> Leave (6)
                                                   └──> TimeTracking (8)
```

**Historical entities**: PersonHistory, EmploymentHistory, PositionHistory, OrgChangeEvent track temporal changes.

**Master data entities**: CompensationType, BenefitType, LeaveType, WorkScheduleTemplate, ShiftTemplate, WorkCalendar, QualificationRequirement.

### Key Design Principles

- **Multi-employment support**: Single person can have multiple concurrent employment relationships
- **Employment-type specific properties**: Employment entity contains property groups (kjt_*, puetv_*, eszjtv_*, gov_*) activated based on employmentType
- **GDPR compliance**: Strict data retention rules (employment + 3/5/8 years depending on data type)
- **Audit trail**: Historical entities track all changes with validFrom/validTo timestamps
- **Hungarian labor law compliance**: Each employment type has specific compensation calculation, leave entitlement, and work schedule rules

### Cross-domain relationships

- Leave ↔ TimeTracking: LeaveRecord connects to DailyTimeRecord for daily reconciliation
- TimeTracking → Compensation: Monthly aggregation from DailyTimeRecord generates CompensationElements (overtime, shift differentials)
- Qualification ← Position: QualificationRequirement validation against Person qualifications

## Key Legal Requirements

### Mandatory Time Tracking (Mt. 134. §)
Must record regular hours, overtime, night shifts, on-call duty, and standby time.

### Leave Management
- Base leave varies by employment type: Mt. 20+ days, Púétv. 50 days, Kit. 25 days
- Supplementary leave based on age, children, disability
- 14 consecutive days annual leave mandatory (Mt.)

### Compensation Rules
- **Kjt./Eszjtv.**: Salary grade tables based on education and seniority
- **Púétv.**: Career advancement levels (Gyakornok → Pedagógus I/II → Mesterpedagógus → Kutatótanár)
- **Kit./Kttv.**: Classification categories with salary bands
- **Mt.**: Free negotiation within minimum wage constraints

### GDPR Data Retention
- General employment data: Employment end + 5 years (tax/social security statute of limitations)
- Payroll data: Employment end + 8 years (accounting regulation)
- Time tracking: Employment end + 3 years (labor dispute statute of limitations)
- Special category data (disability, health): Enhanced protection under GDPR Art. 9

### Automatic Events (30 total)
The system must trigger automatic calculations/notifications for events like:
- Kjt./Eszjtv./Púétv. seniority-based pay increments
- Jubilee bonuses (25/30/35/40 years)
- Probation period expiration
- Leave entitlement generation (Jan 1 annually)
- Overtime limit warnings (200/250 hours)
- Professional license expiration
- Mandatory training cycle completion

## Documentation Structure

All design documentation is in `tervezes/` (Hungarian for "design"):

- `hrms-er-osszefoglalo.md` - Complete entity relationship summary
- `erintett-torvenyek.md` - Affected Hungarian laws and regulations
- `person-entity.md` - Person domain entities
- `employment-entity.md` - Employment domain (central hub)
- `organization-entity.md` - Organization and OrgUnit entities
- `position-entity.md` - Position and classification entities
- `compensation-entity.md` - Compensation and benefits entities
- `qualification-entity.md` - Education, certifications, licenses
- `leave-entity.md` - Leave and absence management
- `timetracking-entity.md` - Work schedule and time recording
- `document-entity.md` - Document management
- `performance-review-entity.md` - Performance evaluation
- `payroll-entity.md` - Payroll processing
- `workflow-entity.md` - Approval workflows
- `report-publishing-entity.md` - Reporting and analytics
- `hrms-er-diagram.mermaid` - Visual ER diagram

## Open Architecture Decisions

High-priority questions requiring resolution before implementation:

1. **Employment type transitions**: How to handle conversion from one employment law to another (e.g., Kjt. → Púétv.)?
2. **Bitemporal modeling**: Is full bitemporal support needed (transaction time + valid time)?
3. **Contract vs. employment scope**: Should temporary contracts (megbízási/vállalkozási) be included?
4. **Salary placement**: Balance between Employment (classification basis) and Compensation (actual amount) to avoid redundancy
5. **Document storage**: Metadata only vs. digital copies (considering Mt. 10. § prohibition on requiring document copies)
6. **Matrix organizations**: Dual OrgUnit references or separate Assignment entity?

## External Integrations

The system must integrate with Hungarian government systems:

- **NAV** (Tax Authority): T1041 employment reports, payroll tax submissions
- **NEAK** (Health Insurance): Sick leave, maternity/parental leave benefits
- **MÁK** (State Treasury): Public sector payroll
- **KSH** (Statistics Office): OSAP reports
- **KIR/KRÉTA**: Public education systems
- **OKFŐ**: Healthcare professional registry
- **Közigállás** (gov.hu): Job vacancy postings
- **Kamarák** (Professional chambers): MOK, MESZK, MGYK license verification

## Technology Stack

**Note**: No technology decisions have been made yet. The repository contains only design documentation.
