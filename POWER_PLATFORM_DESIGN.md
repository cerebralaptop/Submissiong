# Green Star Submission Assistant - Power Platform Design Document

## Executive Summary

This document provides technical specifications for implementing the Green Star Submission Assistant on Microsoft Power Platform. It is intended for development teams building an enterprise-grade version integrated with Dataverse, Power Apps, Power Automate, and Power BI.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Dataverse Data Model](#2-dataverse-data-model)
3. [Power Apps Design](#3-power-apps-design)
4. [Power Automate Flows](#4-power-automate-flows)
5. [Power BI Reports](#5-power-bi-reports)
6. [SharePoint Integration](#6-sharepoint-integration)
7. [Security & Access Control](#7-security--access-control)
8. [Deployment Guide](#8-deployment-guide)
9. [Migration from Static Version](#9-migration-from-static-version)

---

## 1. Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POWER PLATFORM ARCHITECTURE                          │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────┐
                              │   Azure AD      │
                              │   (Identity)    │
                              └────────┬────────┘
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        │                              │                              │
        ▼                              ▼                              ▼
┌───────────────┐            ┌─────────────────┐            ┌───────────────┐
│  Power Apps   │            │  Power Automate │            │   Power BI    │
│               │            │                 │            │               │
│ ┌───────────┐ │            │ ┌─────────────┐ │            │ ┌───────────┐ │
│ │ Main App  │ │◄──────────►│ │ Workflows   │ │◄──────────►│ │ Dashboards│ │
│ │ (Canvas)  │ │            │ │             │ │            │ │           │ │
│ └───────────┘ │            │ │ - Approvals │ │            │ │ - Progress│ │
│ ┌───────────┐ │            │ │ - Notifs    │ │            │ │ - KPIs    │ │
│ │ Admin App │ │            │ │ - Sync      │ │            │ │ - Trends  │ │
│ │ (Model)   │ │            │ └─────────────┘ │            │ └───────────┘ │
│ └───────────┘ │            └────────┬────────┘            └───────┬───────┘
└───────┬───────┘                     │                             │
        │                             │                             │
        └─────────────────────────────┼─────────────────────────────┘
                                      │
                              ┌───────┴───────┐
                              │   Dataverse   │
                              │   (Database)  │
                              │               │
                              │ - Projects    │
                              │ - Credits     │
                              │ - Submissions │
                              │ - Users       │
                              └───────┬───────┘
                                      │
                              ┌───────┴───────┐
                              │  SharePoint   │
                              │  (Documents)  │
                              │               │
                              │ - Evidence    │
                              │ - Templates   │
                              │ - Reports     │
                              └───────────────┘
```

### 1.2 Component Responsibilities

| Component | Purpose | Key Features |
|-----------|---------|--------------|
| **Power Apps (Canvas)** | Main user application | Credit selection, progress tracking, document upload |
| **Power Apps (Model-Driven)** | Admin configuration | Manage credits, categories, templates, users |
| **Dataverse** | Central database | All structured data, relationships, business rules |
| **Power Automate** | Workflow automation | Notifications, approvals, document generation |
| **Power BI** | Reporting & analytics | Dashboards, progress reports, portfolio views |
| **SharePoint** | Document management | Evidence files, templates, generated reports |
| **Azure AD** | Identity & access | SSO, role-based access, security groups |

### 1.3 Environment Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ENVIRONMENT STRATEGY                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Development   │────►│     Test/UAT    │────►│   Production    │
│                 │     │                 │     │                 │
│ - Feature dev   │     │ - User testing  │     │ - Live system   │
│ - Rapid changes │     │ - Integration   │     │ - Managed       │
│ - Sample data   │     │ - Real data     │     │ - Monitored     │
└─────────────────┘     └─────────────────┘     └─────────────────┘

Solution: GreenStarAssistant (Managed in Prod, Unmanaged in Dev)
Publisher: GBCA or Organization Name
```

---

## 2. Dataverse Data Model

### 2.1 Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DATAVERSE ENTITY MODEL                               │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│  gs_category    │       │   gs_credit     │       │   gs_level      │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ PK categoryid   │──1:N──│ PK creditid     │──1:N──│ PK levelid      │
│    name         │       │ FK categoryid   │       │ FK creditid     │
│    description  │       │    name         │       │    leveltype    │
│    icon         │       │    description  │       │    points       │
│    color        │       │    maxpoints    │       │    summary      │
│    displayorder │       │    difficulty   │       │    description  │
│    statecode    │       │    required     │       └─────────────────┘
└─────────────────┘       │    projectphases│
                          │    displayorder │       ┌─────────────────┐
                          └────────┬────────┘       │  gs_criteria    │
                                   │                ├─────────────────┤
                          ┌────────┴────────┐       │ PK criteriaid   │
                          │                 │───1:N─│ FK creditid     │
                          │                 │       │ FK levelid      │
                          │                 │       │    criteriatext │
                          │                 │       │    mandatory    │
                          │                 │       │    evidencetype │
                          │                 │       └─────────────────┘
                          │
┌─────────────────┐       │       ┌─────────────────┐
│   gs_project    │       │       │ gs_projectcredit│
├─────────────────┤       │       ├─────────────────┤
│ PK projectid    │──1:N──┼───────│ PK id           │
│ FK ownerid      │       │       │ FK projectid    │
│ FK templateid   │       │       │ FK creditid     │
│    name         │       └───────│ FK levelid      │
│    targetrating │               │    status       │
│    status       │               │    points       │
│    createdon    │               │    notes        │
│    isscenario   │               │    assignedto   │
│ FK parentproject│               │    duedate      │
└────────┬────────┘               └────────┬────────┘
         │                                 │
         │                                 │
         │       ┌─────────────────┐       │
         │       │ gs_documentation│       │
         └──1:N──├─────────────────┤──N:1──┘
                 │ PK documentid   │
                 │ FK projectid    │
                 │ FK creditid     │
                 │    filename     │
                 │    sharepointurl│
                 │    uploadedby   │
                 │    uploadedon   │
                 │    status       │
                 └─────────────────┘

┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│  gs_template    │       │   gs_synergy    │       │ gs_commonmistake│
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ PK templateid   │       │ PK synergyid    │       │ PK mistakeid    │
│    name         │       │ FK credit1id    │       │ FK creditid     │
│    buildingtype │       │ FK credit2id    │       │    description  │
│    description  │       │    synergytype  │       │    impact       │
│    targetrating │       │    description  │       │    prevention   │
│    creditlist   │       │    recommendation│      └─────────────────┘
│    active       │       └─────────────────┘
└─────────────────┘
```

### 2.2 Table Definitions

#### gs_category (Category)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_categoryid | Unique Identifier | Auto | Primary key |
| gs_name | Single Line Text (100) | Yes | Category name (e.g., "Responsible") |
| gs_description | Multiple Lines Text | No | Category description |
| gs_icon | Single Line Text (50) | No | Icon class name |
| gs_color | Single Line Text (20) | Yes | Color theme (amber, green, blue, etc.) |
| gs_displayorder | Whole Number | Yes | Sort order |
| statecode | State | Auto | Active/Inactive |

**Business Rules:**
- Display order must be unique
- Name must be unique

---

#### gs_credit (Credit)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_creditid | Unique Identifier | Auto | Primary key |
| gs_name | Single Line Text (100) | Yes | Credit name |
| gs_categoryid | Lookup (gs_category) | Yes | Parent category |
| gs_shortdescription | Single Line Text (200) | No | Brief description |
| gs_description | Multiple Lines Text | No | Full description |
| gs_aims | Multiple Lines Text | No | Credit aims |
| gs_tips | Multiple Lines Text | No | Implementation tips |
| gs_maxpoints | Whole Number | Yes | Maximum achievable points |
| gs_difficulty | Choice | Yes | Easy, Medium, Hard |
| gs_required | Yes/No | Yes | Is credit mandatory? |
| gs_projectphases | Multi-Select Choice | No | Applicable project phases |
| gs_displayorder | Whole Number | Yes | Sort order within category |
| statecode | State | Auto | Active/Inactive |

**Choice: gs_difficulty**
```
100000000 = Easy
100000001 = Medium
100000002 = Hard
```

**Multi-Select Choice: gs_projectphases**
```
100000000 = Concept
100000001 = Schematic
100000002 = Design Development
100000003 = Documentation
100000004 = Construction
100000005 = Handover
```

---

#### gs_level (Performance Level)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_levelid | Unique Identifier | Auto | Primary key |
| gs_creditid | Lookup (gs_credit) | Yes | Parent credit |
| gs_leveltype | Choice | Yes | Minimum, Credit, Exceptional |
| gs_points | Whole Number | Yes | Points for this level |
| gs_summary | Single Line Text (200) | Yes | Brief summary |
| gs_description | Multiple Lines Text | No | Detailed requirements |

**Choice: gs_leveltype**
```
100000000 = Minimum Expectation
100000001 = Credit Achievement
100000002 = Exceptional Performance
```

---

#### gs_project (Project)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_projectid | Unique Identifier | Auto | Primary key |
| gs_name | Single Line Text (200) | Yes | Project name |
| ownerid | Lookup (User/Team) | Yes | Project owner |
| gs_templateid | Lookup (gs_template) | No | Template used |
| gs_targetrating | Choice | Yes | Target star rating |
| gs_status | Choice | Yes | Project status |
| gs_currentpoints | Whole Number | Calc | Calculated total points |
| gs_isscenario | Yes/No | No | Is this a scenario copy? |
| gs_parentprojectid | Lookup (gs_project) | No | Parent if scenario |
| createdon | DateTime | Auto | Creation timestamp |
| modifiedon | DateTime | Auto | Last modified |

**Choice: gs_targetrating**
```
100000000 = 4 Star (45 points)
100000001 = 5 Star (60 points)
100000002 = 6 Star (75 points)
100000003 = World Leadership (90+ points)
```

**Choice: gs_status**
```
100000000 = Planning
100000001 = In Progress
100000002 = Under Review
100000003 = Submitted
100000004 = Certified
100000005 = Archived
```

---

#### gs_projectcredit (Project Credit Selection)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_projectcreditid | Unique Identifier | Auto | Primary key |
| gs_projectid | Lookup (gs_project) | Yes | Parent project |
| gs_creditid | Lookup (gs_credit) | Yes | Selected credit |
| gs_levelid | Lookup (gs_level) | No | Selected level |
| gs_status | Choice | Yes | Credit status |
| gs_points | Whole Number | Calc | Points from selected level |
| gs_notes | Multiple Lines Text | No | User notes |
| gs_assignedto | Lookup (User) | No | Assigned team member |
| gs_duedate | Date Only | No | Target completion date |

**Choice: gs_status**
```
100000000 = Not Started
100000001 = In Progress
100000002 = Documentation Ready
100000003 = Under Review
100000004 = Approved
100000005 = Not Pursuing
```

---

#### gs_documentation (Evidence Documents)

| Column | Type | Required | Description |
|--------|------|----------|-------------|
| gs_documentationid | Unique Identifier | Auto | Primary key |
| gs_projectid | Lookup (gs_project) | Yes | Parent project |
| gs_creditid | Lookup (gs_credit) | Yes | Related credit |
| gs_name | Single Line Text (255) | Yes | Document name |
| gs_sharepointurl | URL | Yes | SharePoint file URL |
| gs_documenttype | Choice | Yes | Document type |
| gs_status | Choice | Yes | Review status |
| gs_uploadedby | Lookup (User) | Auto | Uploader |
| gs_uploadedon | DateTime | Auto | Upload timestamp |
| gs_reviewedby | Lookup (User) | No | Reviewer |
| gs_reviewedon | DateTime | No | Review timestamp |
| gs_comments | Multiple Lines Text | No | Review comments |

**Choice: gs_documenttype**
```
100000000 = Calculation
100000001 = Report
100000002 = Drawing
100000003 = Specification
100000004 = Certificate
100000005 = Photo Evidence
100000006 = Other
```

---

### 2.3 Calculated Fields & Rollups

#### gs_project.gs_currentpoints (Rollup)

```
Rollup Configuration:
- Related Entity: gs_projectcredit
- Aggregation: SUM
- Source Attribute: gs_points
- Filter: gs_status != "Not Pursuing"
```

#### gs_project.gs_progresspercentage (Calculated)

```
Formula:
IF(gs_targetrating == 100000000, gs_currentpoints / 45 * 100,
IF(gs_targetrating == 100000001, gs_currentpoints / 60 * 100,
IF(gs_targetrating == 100000002, gs_currentpoints / 75 * 100,
gs_currentpoints / 90 * 100)))
```

#### gs_projectcredit.gs_points (Calculated)

```
Formula:
IF(gs_levelid != null, gs_levelid.gs_points, 0)
```

---

### 2.4 Business Rules

#### BR-001: Project Credit Unique Constraint

**Table:** gs_projectcredit
**Condition:** On Create
**Rule:** Prevent duplicate credit selections per project

```
// Alternate Key on: gs_projectid + gs_creditid
CREATE ALTERNATE KEY ak_projectcredit_unique
ON gs_projectcredit (gs_projectid, gs_creditid)
```

#### BR-002: Required Credit Validation

**Table:** gs_project
**Condition:** On Status Change to "Submitted"
**Rule:** All required credits must be selected

```javascript
// Power Automate or Plugin
var requiredCredits = fetchRelated("gs_credit", "gs_required == true");
var selectedCredits = fetchRelated("gs_projectcredit", "gs_projectid == currentProject");

foreach (credit in requiredCredits) {
    if (!selectedCredits.contains(credit.gs_creditid)) {
        throw Error("Required credit not selected: " + credit.gs_name);
    }
}
```

#### BR-003: Points Calculation

**Table:** gs_projectcredit
**Condition:** On Level Selection
**Rule:** Auto-calculate points from level

```
// Real-time workflow or calculated field
gs_points = RELATED(gs_levelid).gs_points
```

---

## 3. Power Apps Design

### 3.1 Application Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POWER APPS STRUCTURE                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        CANVAS APP: Green Star Main                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐  ┌────────────────────────────────────────────────────┐    │
│  │   Sidebar   │  │                    Main Content                     │    │
│  │             │  │                                                     │    │
│  │ ┌─────────┐ │  │  ┌───────────────────────────────────────────────┐ │    │
│  │ │ Logo    │ │  │  │ Header: Project Name, Score, Target Rating   │ │    │
│  │ └─────────┘ │  │  └───────────────────────────────────────────────┘ │    │
│  │             │  │                                                     │    │
│  │ ┌─────────┐ │  │  ┌───────────────────────────────────────────────┐ │    │
│  │ │ Project │ │  │  │ Tabs: Credits | Timeline | Docs | Analysis   │ │    │
│  │ │Dropdown │ │  │  └───────────────────────────────────────────────┘ │    │
│  │ └─────────┘ │  │                                                     │    │
│  │             │  │  ┌───────────────────────────────────────────────┐ │    │
│  │ Categories  │  │  │                                               │ │    │
│  │ ┌─────────┐ │  │  │            Credit Gallery / List              │ │    │
│  │ │Responsib│ │  │  │                                               │ │    │
│  │ │Healthy  │ │  │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐         │ │    │
│  │ │Resilient│ │  │  │  │Credit 1 │ │Credit 2 │ │Credit 3 │         │ │    │
│  │ │Positive │ │  │  │  │         │ │         │ │         │         │ │    │
│  │ │Places   │ │  │  │  │ 4 pts   │ │ 6 pts   │ │ 3 pts   │         │ │    │
│  │ │People   │ │  │  │  └─────────┘ └─────────┘ └─────────┘         │ │    │
│  │ │Nature   │ │  │  │                                               │ │    │
│  │ │Leadership│ │  │  └───────────────────────────────────────────────┘ │    │
│  │ └─────────┘ │  │                                                     │    │
│  │             │  │                                                     │    │
│  │ ┌─────────┐ │  │                                                     │    │
│  │ │ Tools   │ │  │                                                     │    │
│  │ │- Compare│ │  │                                                     │    │
│  │ │- What-If│ │  │                                                     │    │
│  │ │- Export │ │  │                                                     │    │
│  │ └─────────┘ │  │                                                     │    │
│  └─────────────┘  └────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

SCREENS:
1. HomeScreen           - Main dashboard with credits
2. CreditDetailScreen   - Credit details, levels, checklist
3. ProjectsScreen       - Project list and management
4. ComparisonScreen     - Side-by-side project comparison
5. WhatIfScreen         - Scenario analysis tool
6. DocumentsScreen      - Document upload and management
7. ReportsScreen        - Export and reporting options
8. SettingsScreen       - User preferences
9. OnboardingScreen     - First-time user tour
```

### 3.2 Screen Specifications

#### HomeScreen

**Components:**
| Component | Type | Data Source | Notes |
|-----------|------|-------------|-------|
| galCategories | Gallery (Vertical) | gs_category | Category navigation |
| galCredits | Gallery (Flexible) | gs_credit | Credit cards grid |
| lblCurrentScore | Label | varCurrentProject.gs_currentpoints | Live score |
| drpTargetRating | Dropdown | Choices | Target selection |
| cmpProgressBar | Component | Calculated | Visual progress |
| txtSearch | Text Input | - | Credit search |
| drpFilter | Dropdown | - | Category/Phase filter |

**Key Formulas:**

```powerapps
// Filter credits by category and search
Filter(
    gs_credit,
    (IsBlank(varSelectedCategory) || gs_categoryid.gs_categoryid = varSelectedCategory.gs_categoryid) &&
    (IsBlank(txtSearch.Text) || txtSearch.Text in gs_name || txtSearch.Text in gs_description)
)

// Calculate current points
Sum(
    Filter(gs_projectcredit, gs_projectid.gs_projectid = varCurrentProject.gs_projectid),
    gs_points
)

// Get star rating from points
Switch(
    true,
    varCurrentPoints >= 90, "World Leadership",
    varCurrentPoints >= 75, "6 Star",
    varCurrentPoints >= 60, "5 Star",
    varCurrentPoints >= 45, "4 Star",
    "Below 4 Star"
)
```

---

#### CreditDetailScreen

**Layout:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  [← Back]                    Credit Name                      [Required]     │
├─────────────────────────────────────────────────────────────────────────────┤
│  Category: Responsible    │ Max Points: 6    │ Difficulty: Medium           │
├─────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  TABS: Overview | Levels | Checklist | Documents | Notes            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  [Overview Tab]                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Description                                                          │    │
│  │ ────────────────────────────────────────────────────────────────────│    │
│  │ Supporting the development of the green building industry through   │    │
│  │ training, research, and knowledge sharing.                          │    │
│  │                                                                      │    │
│  │ Aims                                                                 │    │
│  │ ────────────────────────────────────────────────────────────────────│    │
│  │ • Encourage professional development                                 │    │
│  │ • Share knowledge across the industry                                │    │
│  │                                                                      │    │
│  │ Tips                                                                 │    │
│  │ ────────────────────────────────────────────────────────────────────│    │
│  │ Engage early with training providers...                              │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  [Levels Tab]                                                                │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐             │
│  │ ○ Minimum (0pt)  │ │ ● Credit (4pt)   │ │ ○ Exceptional(6pt│             │
│  │                  │ │   [SELECTED]     │ │                  │             │
│  │ Meet basic code  │ │ 50% improvement  │ │ 100% improvement │             │
│  │ requirements     │ │ on code minimum  │ │ + monitoring     │             │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘             │
│                                                                              │
│                                             [Save Selection]                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Key Formulas:**

```powerapps
// Get levels for current credit
Sort(
    Filter(gs_level, gs_creditid.gs_creditid = varSelectedCredit.gs_creditid),
    gs_leveltype
)

// Save credit selection
Patch(
    gs_projectcredit,
    LookUp(gs_projectcredit,
        gs_projectid.gs_projectid = varCurrentProject.gs_projectid &&
        gs_creditid.gs_creditid = varSelectedCredit.gs_creditid
    ),
    {
        gs_levelid: varSelectedLevel,
        gs_points: varSelectedLevel.gs_points,
        gs_status: 'gs_status (Project Credits)'.InProgress
    }
);
Notify("Selection saved", NotificationType.Success)
```

---

### 3.3 Component Library

Create a reusable component library for consistent UI:

| Component | Purpose | Properties |
|-----------|---------|------------|
| cmpCreditCard | Credit display card | Credit record, IsSelected, OnSelect |
| cmpProgressBar | Visual progress | CurrentValue, MaxValue, Color |
| cmpStarRating | Star rating display | Rating (4-6), Size |
| cmpLevelSelector | Level radio buttons | Levels collection, SelectedLevel |
| cmpDocumentUploader | File upload | ProjectId, CreditId, OnUpload |
| cmpCategoryIcon | Category icon | CategoryName, Size, Color |
| cmpTourStep | Onboarding step | StepNumber, Title, Content, Position |

**Example: cmpCreditCard**

```powerapps
// Component Properties
Credit: Record
IsSelected: Boolean
OnSelect: Behavior

// Component Body
Rectangle (Container)
├── Icon (Category icon)
├── Label (Credit name)
├── Label (Short description)
├── Label (Points: X max)
├── Icon (Required indicator if applicable)
└── Rectangle (Selection indicator)

// OnSelect of container
Select(Parent); cmpCreditCard.OnSelect()

// Fill color based on selection
If(cmpCreditCard.IsSelected,
   RGBA(47, 107, 63, 0.1),
   RGBA(255, 255, 255, 1))
```

---

### 3.4 Model-Driven App (Admin)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MODEL-DRIVEN APP: Green Star Admin                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SITE MAP:                                                                   │
│  ├── Dashboard                                                               │
│  │   └── Admin Overview (Power BI embedded)                                  │
│  │                                                                           │
│  ├── Reference Data                                                          │
│  │   ├── Categories (gs_category)                                            │
│  │   ├── Credits (gs_credit)                                                 │
│  │   ├── Performance Levels (gs_level)                                       │
│  │   ├── Criteria (gs_criteria)                                              │
│  │   └── Documentation Requirements (gs_documentrequirement)                 │
│  │                                                                           │
│  ├── Configuration                                                           │
│  │   ├── Project Templates (gs_template)                                     │
│  │   ├── Credit Synergies (gs_synergy)                                       │
│  │   └── Common Mistakes (gs_commonmistake)                                  │
│  │                                                                           │
│  ├── Projects                                                                │
│  │   ├── All Projects (gs_project)                                           │
│  │   ├── Project Credits (gs_projectcredit)                                  │
│  │   └── Documentation (gs_documentation)                                    │
│  │                                                                           │
│  └── Settings                                                                │
│      ├── Users & Security                                                    │
│      └── Import/Export                                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Form Customizations:**

| Entity | Main Form | Quick Create | Views |
|--------|-----------|--------------|-------|
| gs_credit | Full details with tabs | Name, Category, MaxPoints | By Category, By Difficulty, Required Only |
| gs_project | Project summary + related | Name, Template, Target | Active, My Projects, By Status |
| gs_projectcredit | Selection with documents | - | By Status, Overdue, By Assignee |

---

## 4. Power Automate Flows

### 4.1 Flow Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POWER AUTOMATE FLOWS                                 │
└─────────────────────────────────────────────────────────────────────────────┘

TRIGGER-BASED FLOWS:
┌─────────────────────────────────────────────────────────────────────────────┐
│ Flow Name                    │ Trigger                │ Action              │
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-001: New Project Created  │ gs_project Created     │ Create SharePoint   │
│                              │                        │ folder, send welcome│
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-002: Credit Status Change │ gs_projectcredit       │ Notify assignee,    │
│                              │ Modified (status)      │ update project      │
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-003: Document Uploaded    │ gs_documentation       │ Move to SharePoint, │
│                              │ Created                │ notify reviewer     │
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-004: Due Date Approaching │ Scheduled (Daily)      │ Send reminders for  │
│                              │                        │ credits due in 7 days│
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-005: Project Submitted    │ gs_project Modified    │ Generate PDF report,│
│                              │ (status = Submitted)   │ notify stakeholders │
├──────────────────────────────┼────────────────────────┼─────────────────────┤
│ FL-006: Weekly Summary       │ Scheduled (Weekly)     │ Email project owners│
│                              │                        │ with progress report│
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Flow Specifications

#### FL-001: New Project Created

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ FLOW: FL-001 New Project Created                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TRIGGER: When a row is added (gs_project)                                   │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Create SharePoint Folder                                            │
│     │    Site: GreenStarProjects                                             │
│     │    Library: Project Documents                                          │
│     │    Folder: {ProjectName}_{ProjectId}                                   │
│     │                                                                        │
│     ├──► Create subfolder: Evidence                                          │
│     ├──► Create subfolder: Reports                                           │
│     └──► Create subfolder: Templates                                         │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Update project record                                               │
│     │    gs_sharepointfolderurl = folder URL                                 │
│     │                                                                        │
│     ▼                                                                        │
│  CONDITION: Template selected?                                               │
│     │                                                                        │
│     ├── YES ──► Apply template credits                                       │
│     │           Loop through template.creditlist                             │
│     │           Create gs_projectcredit for each                             │
│     │                                                                        │
│     └── NO ───► Skip                                                         │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Send email to owner                                                 │
│          Subject: "Your Green Star project has been created"                 │
│          Body: Welcome message + link to app                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### FL-003: Document Uploaded

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ FLOW: FL-003 Document Uploaded                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TRIGGER: When a row is added (gs_documentation)                             │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Get file content from temporary location                            │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Get project details                                                 │
│     │    Expand: gs_projectid                                                │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Create file in SharePoint                                           │
│     │    Site: GreenStarProjects                                             │
│     │    Folder: {Project.gs_sharepointfolderurl}/Evidence/{CreditId}        │
│     │    Filename: {timestamp}_{originalname}                                │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Update documentation record                                         │
│     │    gs_sharepointurl = file URL                                         │
│     │    gs_status = Uploaded                                                │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Post to Teams channel (optional)                                    │
│          Channel: Project-{ProjectName}                                      │
│          Message: "New document uploaded for {CreditName}"                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### FL-005: Project Submitted

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ FLOW: FL-005 Project Submitted                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TRIGGER: When a row is modified (gs_project)                                │
│     │    Filter: gs_status = Submitted                                       │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Get all project credits                                             │
│     │    Filter: gs_projectid = trigger.gs_projectid                         │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Get all documentation                                               │
│     │    Filter: gs_projectid = trigger.gs_projectid                         │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Generate Word document from template                                │
│     │    Template: SubmissionReport.docx                                     │
│     │    Data: Project details, credits, documents                           │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Convert to PDF                                                      │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Save to SharePoint                                                  │
│     │    Folder: {Project}/Reports                                           │
│     │    Filename: SubmissionReport_{date}.pdf                               │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Send email with attachment                                          │
│     │    To: Project owner, Stakeholders                                     │
│     │    Subject: "Green Star Submission Ready: {ProjectName}"               │
│     │    Attachment: PDF report                                              │
│     │                                                                        │
│     ▼                                                                        │
│  ACTION: Start approval flow (optional)                                      │
│          Approver: GBCA Reviewer                                             │
│          Outcome updates gs_status                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Power BI Reports

### 5.1 Report Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         POWER BI REPORTS                                     │
└─────────────────────────────────────────────────────────────────────────────┘

REPORT: Green Star Analytics
├── Page 1: Executive Dashboard
│   ├── Total Projects (Card)
│   ├── Projects by Status (Pie Chart)
│   ├── Average Progress % (Gauge)
│   ├── Points Distribution (Histogram)
│   └── Recent Activity (Table)
│
├── Page 2: Project Deep Dive
│   ├── Project Selector (Slicer)
│   ├── Progress by Category (Bar Chart)
│   ├── Credit Status Breakdown (Stacked Bar)
│   ├── Document Completion (Donut)
│   └── Timeline to Submission (Gantt)
│
├── Page 3: Portfolio View
│   ├── All Projects Matrix
│   ├── Points Trend Over Time (Line)
│   ├── Team Performance (Table)
│   └── Geographic Distribution (Map)
│
├── Page 4: Credit Analytics
│   ├── Most Popular Credits (Bar)
│   ├── Average Points by Category (Column)
│   ├── Difficulty vs Selection Rate (Scatter)
│   └── Common Mistakes Impact (Table)
│
└── Page 5: Comparison
    ├── Multi-Project Selector
    ├── Side-by-Side Points (Clustered Column)
    ├── Category Comparison (Radar)
    └── Status Comparison (Matrix)
```

### 5.2 Key Measures (DAX)

```dax
// Total Projects
Total Projects = COUNTROWS(gs_project)

// Active Projects
Active Projects =
CALCULATE(
    COUNTROWS(gs_project),
    gs_project[gs_status] IN {"Planning", "In Progress", "Under Review"}
)

// Average Progress
Avg Progress % =
AVERAGEX(
    gs_project,
    DIVIDE(gs_project[gs_currentpoints],
           SWITCH(gs_project[gs_targetrating],
                  100000000, 45,
                  100000001, 60,
                  100000002, 75,
                  90),
           0) * 100
)

// Points by Category
Points by Category =
SUMX(
    RELATEDTABLE(gs_projectcredit),
    gs_projectcredit[gs_points]
)

// Credit Completion Rate
Credit Completion % =
DIVIDE(
    CALCULATE(COUNTROWS(gs_projectcredit),
              gs_projectcredit[gs_status] = "Approved"),
    COUNTROWS(gs_projectcredit),
    0
) * 100

// Documents Pending
Documents Pending =
CALCULATE(
    COUNTROWS(gs_documentation),
    gs_documentation[gs_status] = "Pending Review"
)

// Days to Due Date
Days to Due =
DATEDIFF(TODAY(), gs_projectcredit[gs_duedate], DAY)

// Overdue Credits
Overdue Credits =
CALCULATE(
    COUNTROWS(gs_projectcredit),
    gs_projectcredit[gs_duedate] < TODAY(),
    gs_projectcredit[gs_status] <> "Approved"
)
```

### 5.3 Embedded Reports in Apps

**Canvas App Integration:**

```powerapps
// Add Power BI tile to canvas app
PowerBIIntegration.Refresh()

// Filter context
PowerBIIntegration.Data =
    Filter(
        PowerBITile.Data,
        ProjectId = varCurrentProject.gs_projectid
    )
```

**Model-Driven App Integration:**

1. Create Power BI Dashboard
2. Add to Model-Driven App via sitemap
3. Configure contextual filtering

---

## 6. SharePoint Integration

### 6.1 Site Structure

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SHAREPOINT SITE STRUCTURE                               │
└─────────────────────────────────────────────────────────────────────────────┘

SharePoint Site: GreenStarProjects
│
├── Document Libraries
│   │
│   ├── Project Documents
│   │   └── {Project Name}_{ID}/
│   │       ├── Evidence/
│   │       │   ├── Responsible/
│   │       │   ├── Healthy/
│   │       │   ├── Resilient/
│   │       │   └── ...
│   │       ├── Reports/
│   │       │   ├── SubmissionReport.pdf
│   │       │   └── ProgressReport.pdf
│   │       └── Templates/
│   │
│   ├── Reference Documents
│   │   ├── Green Star Guidelines/
│   │   ├── Documentation Templates/
│   │   └── Training Materials/
│   │
│   └── Admin Exports
│       └── DataExports/
│
├── Lists
│   ├── Project Registry (linked to Dataverse)
│   └── Document Reviews (workflow tracking)
│
└── Pages
    ├── Home (links to apps)
    ├── Getting Started Guide
    └── FAQ
```

### 6.2 Document Metadata

**Content Type: Green Star Evidence**

| Column | Type | Purpose |
|--------|------|---------|
| ProjectId | Lookup | Link to Dataverse project |
| CreditId | Lookup | Link to Dataverse credit |
| DocumentType | Choice | Calculation, Report, Photo, etc. |
| UploadedBy | Person | Auto-populated |
| ReviewStatus | Choice | Pending, Approved, Rejected |
| ReviewedBy | Person | Reviewer |
| ReviewDate | DateTime | When reviewed |
| Comments | Multiple Lines | Review notes |

### 6.3 Permissions Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SHAREPOINT PERMISSIONS                                  │
└─────────────────────────────────────────────────────────────────────────────┘

Site Level:
├── Site Owners: IT Admins, GBCA Admins
├── Site Members: All Green Star Users
└── Site Visitors: Read-only stakeholders

Library: Project Documents
├── Unique permissions per project folder
├── Project Owner: Full Control
├── Project Team: Contribute
└── Reviewers: Edit (specific folders)

Library: Reference Documents
├── Inherit from site
└── Members: Read Only

Library: Admin Exports
├── Admins Only: Full Control
└── All others: No Access
```

---

## 7. Security & Access Control

### 7.1 Security Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SECURITY ARCHITECTURE                                │
└─────────────────────────────────────────────────────────────────────────────┘

                         ┌─────────────────────┐
                         │     Azure AD        │
                         │                     │
                         │  Security Groups:   │
                         │  - GS_Admins        │
                         │  - GS_Users         │
                         │  - GS_Reviewers     │
                         │  - GS_ReadOnly      │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼
           ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
           │ Dataverse   │  │ Power Apps  │  │ SharePoint  │
           │             │  │             │  │             │
           │ Security    │  │ App Sharing │  │ Permissions │
           │ Roles       │  │             │  │ Groups      │
           └─────────────┘  └─────────────┘  └─────────────┘
```

### 7.2 Dataverse Security Roles

| Role | Tables | Access Level |
|------|--------|--------------|
| GS System Admin | All | Organization |
| GS Admin | gs_category, gs_credit, gs_level, gs_criteria, gs_template, gs_synergy, gs_commonmistake | Organization (CRUD) |
| GS Admin | gs_project, gs_projectcredit, gs_documentation | Organization (Read) |
| GS User | gs_project (own) | User (CRUD) |
| GS User | gs_projectcredit, gs_documentation (own project) | User (CRUD) |
| GS User | gs_category, gs_credit, gs_level, gs_criteria | Organization (Read) |
| GS Reviewer | gs_project, gs_projectcredit, gs_documentation | Business Unit (Read/Write) |
| GS Viewer | All | Organization (Read Only) |

### 7.3 Row-Level Security

**gs_project:**
```
// Users can only see projects they own or are team members of
ownerid = CurrentUser OR
gs_projectid IN (SELECT gs_projectid FROM gs_projectteam WHERE userid = CurrentUser)
```

**gs_projectcredit & gs_documentation:**
```
// Inherits from parent project
gs_projectid.ownerid = CurrentUser OR
gs_projectid IN (SELECT gs_projectid FROM gs_projectteam WHERE userid = CurrentUser)
```

### 7.4 Field-Level Security

| Field | Restriction |
|-------|-------------|
| gs_project.gs_internalrating | Visible to Admins/Reviewers only |
| gs_documentation.gs_reviewcomments | Visible to Reviewers/Owner only |
| gs_project.gs_confidentialnotes | Visible to Owner only |

---

## 8. Deployment Guide

### 8.1 Prerequisites

| Requirement | Details |
|-------------|---------|
| Power Platform License | Per-user or per-app licensing |
| Dataverse Capacity | Minimum 1GB database storage |
| SharePoint Site | Modern team site with document libraries |
| Azure AD Groups | Security groups for each role |
| Power BI Pro | For report sharing (or Premium capacity) |

### 8.2 Deployment Steps

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DEPLOYMENT CHECKLIST                                    │
└─────────────────────────────────────────────────────────────────────────────┘

PHASE 1: Environment Setup (Week 1)
□ 1.1  Create Development environment
□ 1.2  Create Test environment
□ 1.3  Create Production environment
□ 1.4  Configure Azure AD security groups
□ 1.5  Set up SharePoint site and libraries
□ 1.6  Configure Dataverse connection

PHASE 2: Data Model (Week 2)
□ 2.1  Import solution with tables
□ 2.2  Create relationships and lookups
□ 2.3  Configure business rules
□ 2.4  Set up calculated/rollup fields
□ 2.5  Create views and forms
□ 2.6  Import reference data (categories, credits)

PHASE 3: Power Apps (Weeks 3-4)
□ 3.1  Build Canvas App screens
□ 3.2  Create component library
□ 3.3  Implement data connections
□ 3.4  Build Model-Driven Admin app
□ 3.5  Configure app security
□ 3.6  Add onboarding tour

PHASE 4: Automation (Week 5)
□ 4.1  Build Power Automate flows
□ 4.2  Configure SharePoint integration
□ 4.3  Set up email notifications
□ 4.4  Test workflow triggers
□ 4.5  Configure error handling

PHASE 5: Reporting (Week 6)
□ 5.1  Create Power BI dataset
□ 5.2  Build report pages
□ 5.3  Configure refresh schedule
□ 5.4  Embed in apps
□ 5.5  Set up workspace security

PHASE 6: Testing & Validation (Weeks 7-8)
□ 6.1  Unit testing
□ 6.2  Integration testing
□ 6.3  User acceptance testing
□ 6.4  Performance testing
□ 6.5  Security testing
□ 6.6  Fix identified issues

PHASE 7: Go-Live (Week 9)
□ 7.1  Final data migration
□ 7.2  Deploy to Production
□ 7.3  Configure monitoring
□ 7.4  User training
□ 7.5  Go-live support
□ 7.6  Documentation handover
```

### 8.3 Solution Packaging

```
Solution: GreenStarAssistant
Publisher: GreenBuildingCouncil
Version: 1.0.0.0

Components:
├── Tables (14)
│   ├── gs_category
│   ├── gs_credit
│   ├── gs_level
│   ├── gs_criteria
│   ├── gs_project
│   ├── gs_projectcredit
│   ├── gs_documentation
│   ├── gs_template
│   ├── gs_synergy
│   ├── gs_commonmistake
│   ├── gs_projectteam
│   ├── gs_calculator
│   ├── gs_faq
│   └── gs_auditlog
│
├── Apps (2)
│   ├── Green Star Main (Canvas)
│   └── Green Star Admin (Model-Driven)
│
├── Flows (6)
│   ├── FL-001 to FL-006
│   └── Child flows
│
├── Security Roles (4)
│   ├── GS System Admin
│   ├── GS Admin
│   ├── GS User
│   └── GS Reviewer
│
├── Connection References (3)
│   ├── Dataverse
│   ├── SharePoint
│   └── Office 365 Outlook
│
└── Environment Variables (8)
    ├── SharePointSiteUrl
    ├── AdminEmailGroup
    ├── DefaultTargetRating
    ├── NotificationEnabled
    ├── ReviewReminderDays
    ├── MaxFileUploadSizeMB
    ├── PowerBIWorkspaceId
    └── SupportEmailAddress
```

---

## 9. Migration from Static Version

### 9.1 Data Migration Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      DATA MIGRATION FLOW                                     │
└─────────────────────────────────────────────────────────────────────────────┘

SOURCE: Static HTML App (localStorage + JSON)
│
├── Export from browser
│   │
│   ├── Categories ──────────────────────────► gs_category
│   │   localStorage: admin_categories
│   │
│   ├── Credits ─────────────────────────────► gs_credit
│   │   localStorage: admin_credits
│   │
│   ├── Levels ──────────────────────────────► gs_level
│   │   localStorage: admin_levels
│   │
│   ├── Criteria ────────────────────────────► gs_criteria
│   │   localStorage: admin_criteria
│   │
│   ├── Templates ───────────────────────────► gs_template
│   │   localStorage: admin_templates
│   │
│   ├── Synergies ───────────────────────────► gs_synergy
│   │   localStorage: admin_synergies
│   │
│   └── Projects ────────────────────────────► gs_project + gs_projectcredit
│       localStorage: greenstar-projects
│
TARGET: Dataverse (via Power Automate or Excel import)
```

### 9.2 Migration Script (JavaScript to JSON Export)

Add this to the static app for data export:

```javascript
function exportForPowerPlatform() {
    const exportData = {
        metadata: {
            exportedAt: new Date().toISOString(),
            sourceVersion: '1.3',
            targetPlatform: 'Power Platform'
        },
        categories: JSON.parse(localStorage.getItem('admin_categories') || '[]'),
        credits: JSON.parse(localStorage.getItem('admin_credits') || '[]'),
        levels: JSON.parse(localStorage.getItem('admin_levels') || '[]'),
        criteria: JSON.parse(localStorage.getItem('admin_criteria') || '[]'),
        templates: JSON.parse(localStorage.getItem('admin_templates') || '[]'),
        synergies: JSON.parse(localStorage.getItem('admin_synergies') || '[]'),
        commonMistakes: JSON.parse(localStorage.getItem('admin_mistakes') || '[]'),
        projects: JSON.parse(localStorage.getItem('greenstar-projects') || '[]')
    };

    // Transform IDs to GUID format for Dataverse
    exportData.categories = exportData.categories.map(c => ({
        ...c,
        gs_categoryid: generateGuid(),
        gs_name: c.name,
        gs_description: c.description,
        gs_icon: c.icon,
        gs_color: c.color,
        gs_displayorder: c.order
    }));

    // ... similar transforms for other entities

    const blob = new Blob([JSON.stringify(exportData, null, 2)],
                          {type: 'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'GreenStar_PowerPlatform_Export.json';
    a.click();
}

function generateGuid() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        const r = Math.random() * 16 | 0;
        const v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

### 9.3 Feature Mapping

| Static App Feature | Power Platform Implementation |
|-------------------|------------------------------|
| localStorage persistence | Dataverse tables |
| React components | Power Apps Canvas/Model-Driven |
| CSS design tokens | Power Apps themes + component library |
| Excel import/export | Dataverse Excel Online integration |
| Progress calculation | Rollup fields + calculated columns |
| PDF export | Power Automate + Word templates |
| Project switching | Dataverse record selection |
| Scenario cloning | Duplicate detection + flow |
| Onboarding tour | Canvas app component with tutorial screens |
| Dark mode | Power Apps theming (limited) |
| Offline support | Power Apps offline capabilities |

---

## Appendix A: Environment Variables

| Variable | Default Value | Purpose |
|----------|---------------|---------|
| SharePointSiteUrl | https://tenant.sharepoint.com/sites/GreenStar | Base SharePoint URL |
| AdminEmailGroup | greenstar-admins@org.com | Admin notification group |
| DefaultTargetRating | 5 Star | Default for new projects |
| NotificationEnabled | true | Toggle email notifications |
| ReviewReminderDays | 7 | Days before due date to remind |
| MaxFileUploadSizeMB | 50 | Max document size |
| PowerBIWorkspaceId | {guid} | Embedded reports workspace |
| SupportEmailAddress | support@org.com | Help desk contact |

---

## Appendix B: Error Codes

| Code | Message | Resolution |
|------|---------|------------|
| GS-001 | Required credit not selected | Select all mandatory credits before submission |
| GS-002 | Document upload failed | Check file size and format |
| GS-003 | Permission denied | Contact admin for access |
| GS-004 | SharePoint folder creation failed | Verify SharePoint connection |
| GS-005 | Workflow timeout | Retry or contact support |
| GS-006 | Invalid data format | Check import file structure |
| GS-007 | Duplicate record | Record already exists |
| GS-008 | Connection lost | Check network and retry |

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| Dataverse | Microsoft's cloud database for Power Platform |
| Canvas App | Custom-designed Power App with pixel-perfect control |
| Model-Driven App | Data-first Power App based on table structure |
| Power Automate | Workflow automation service (formerly Flow) |
| Solution | Packaged container for Power Platform components |
| Environment | Isolated space for Power Platform development |
| Security Role | Set of table-level permissions |
| Business Rule | Client-side validation and field behavior |
| Rollup Field | Aggregated calculation across related records |

---

*Document Version: 1.0*
*Created: February 1, 2026*
*Author: Green Star Development Team*
*Target Platform: Microsoft Power Platform*

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 1, 2026 | Development Team | Initial Power Platform design document |
