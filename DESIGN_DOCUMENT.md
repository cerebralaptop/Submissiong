# Green Star Submission Assistant - Design Document

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture](#architecture)
3. [Current Features](#current-features)
4. [Data Model](#data-model)
5. [User Flows](#user-flows)
6. [Proposed Features](#proposed-features)
7. [Technical Specifications](#technical-specifications)
8. [Security](#security)
9. [Deployment](#deployment)

---

## 1. System Overview

### Purpose
The Green Star Submission Assistant is a web-based application designed to help project teams manage and track their Green Star Buildings v1.1 certification submissions. It provides tools for credit selection, documentation tracking, progress monitoring, and team collaboration.

### Target Users
- **Green Star Users**: Project team members tracking credits and documentation
- **Administrators**: GBCA staff or project leads managing credit data, templates, and FAQs

### Key Goals
- Simplify the Green Star submission process
- Provide clear visibility into credit requirements and progress
- Enable offline access for site-based work
- Allow administrators to maintain and update credit information without code changes

---

## 2. Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐         ┌──────────────────────────────┐  │
│  │   Main App       │         │      Admin Panel             │  │
│  │   (index.html)   │◄───────►│      (admin.html)            │  │
│  │                  │         │                              │  │
│  │  - Credit View   │         │  - Data Editors              │  │
│  │  - Progress      │  Shared │  - Import/Export             │  │
│  │  - Filters       │  Local  │  - Password Auth             │  │
│  │  - Export        │ Storage │                              │  │
│  └────────┬─────────┘         └──────────────┬───────────────┘  │
│           │                                   │                  │
│           ▼                                   ▼                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    localStorage                              ││
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ ││
│  │  │ greenstar_      │  │ admin_*         │  │ greenstar-   │ ││
│  │  │ admin_data      │  │ (categories,    │  │ project      │ ││
│  │  │ (shared data)   │  │  credits, etc)  │  │ (user data)  │ ││
│  │  └─────────────────┘  └─────────────────┘  └──────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Static Files                              ││
│  │  ┌─────────────────┐  ┌─────────────────────────────────┐   ││
│  │  │ FAQs Excel      │  │ GreenStar_Admin_Data.xlsx       │   ││
│  │  │ (read-only)     │  │ (import/export template)        │   ││
│  │  └─────────────────┘  └─────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| UI Framework | React 18 | Component-based UI |
| Styling | Tailwind CSS | Utility-first styling |
| Transpilation | Babel (browser) | JSX support |
| Icons | Font Awesome 6 | UI icons |
| Fonts | Plus Jakarta Sans | GBCA brand typography |
| Excel Processing | SheetJS (xlsx) | Excel import/export |
| Storage | localStorage | Client-side persistence |
| PDF Generation | html2pdf.js | Export to PDF |

### File Structure

```
/Submissiong
├── index.html                    # Main application (single-page React app)
├── admin.html                    # Admin panel (single-page React app)
├── FAQs for Green Star Buildings copy.xlsx  # FAQ data source
├── GreenStar_Admin_Data.xlsx     # Admin data template
├── create_admin_spreadsheet.py   # Python script to generate Excel template
├── DESIGN_DOCUMENT.md            # This document
└── /assets (future)
    ├── /templates                # Document templates
    └── /icons                    # Custom icons
```

---

## 3. Current Features

### Main Application (index.html)

#### Credit Management
| Feature | Description |
|---------|-------------|
| Category View | 8 categories displayed with color coding and icons |
| Credit Cards | Expandable cards showing credit details |
| Performance Levels | Minimum/Credit/Exceptional level selection |
| Progress Tracking | Real-time points calculation and star rating |
| Credit Selection | Track which level is targeted for each credit |

#### Filtering & Views
| Feature | Description |
|---------|-------------|
| Category Filter | Filter credits by category |
| Phase Filter | Filter by project phase (Concept, Schematic, etc.) |
| Quick Wins | Show only "Easy" difficulty credits |
| Required Only | Show only mandatory credits |
| Search | Full-text search across credits |
| View Modes | Cards, Timeline, Matrix, Docs views |

#### Credit Detail Modal
| Tab | Content |
|-----|---------|
| Overview | Description, aims, tips, related credits |
| Levels | Performance level requirements |
| Checklist | Interactive criteria checklist |
| Notes | User notes per credit |
| Criteria | Detailed criteria with evidence types |
| FAQs | Related FAQs from Excel file |

#### Additional Features
- Dark mode toggle
- Project templates (pre-configured credit selections)
- Gap analysis (identify missing points for target rating)
- Synergies view (credit relationships)
- PDF export
- Multi-project support

### Admin Panel (admin.html)

#### Authentication
- SHA-256 password hashing
- 24-hour session management
- Logout functionality

#### Data Editors
| Editor | Fields |
|--------|--------|
| Categories | ID, Name, Description, Icon, Color, Order |
| Credits | ID, Name, Category, Description, Aims, Tips, Required, Max Points, Difficulty, Project Phases, Order |
| Levels | Credit ID, Level Type, Points, Summary, Description |
| Criteria | Credit ID, Level, Criteria #, Text, Mandatory, Evidence Type |
| Documentation | Credit ID, Level, Name, Type, Description, Template Available, Template Link |
| Templates | ID, Name, Building Type, Description, Target Rating, Credits, Active |
| Synergies | ID, Credit 1, Credit 2, Type, Description, Recommendation |

#### Import/Export
- Import from Excel (all sheets)
- Export to Excel (all data)
- Column mapping for data transformation

---

## 4. Data Model

### Entity Relationship Diagram

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│  Category   │       │   Credit    │       │   Level     │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id (PK)     │──1:N──│ id (PK)     │──1:N──│ creditId(FK)│
│ name        │       │ category(FK)│       │ level       │
│ description │       │ name        │       │ points      │
│ icon        │       │ description │       │ summary     │
│ color       │       │ aims        │       │ description │
│ order       │       │ tips        │       └─────────────┘
└─────────────┘       │ required    │
                      │ maxPoints   │       ┌─────────────┐
                      │ difficulty  │──1:N──│  Criteria   │
                      │ projectPhases       ├─────────────┤
                      │ order       │       │ creditId(FK)│
                      └─────────────┘       │ level       │
                             │              │ criteriaNum │
                             │              │ text        │
                      ┌──────┴──────┐       │ mandatory   │
                      │             │       │ evidenceType│
               ┌──────┴─────┐ ┌─────┴─────┐ └─────────────┘
               │Documentation│ │  Synergy  │
               ├────────────┤ ├───────────┤
               │ creditId   │ │ id        │
               │ level      │ │ credit1   │
               │ name       │ │ credit2   │
               │ type       │ │ type      │
               │ description│ │ description
               │ template   │ │ recommend │
               └────────────┘ └───────────┘
```

### localStorage Keys

| Key | Used By | Purpose |
|-----|---------|---------|
| `greenstar_admin_data` | Both | Shared data from admin to main app |
| `greenstar-project` | Main | Current project state (selections, notes) |
| `greenstar-projects` | Main | List of saved projects |
| `admin_categories` | Admin | Category data (auto-save) |
| `admin_credits` | Admin | Credit data (auto-save) |
| `admin_levels` | Admin | Level data (auto-save) |
| `admin_criteria` | Admin | Criteria data (auto-save) |
| `admin_documentation` | Admin | Documentation data (auto-save) |
| `admin_templates` | Admin | Template data (auto-save) |
| `admin_synergies` | Admin | Synergy data (auto-save) |
| `admin_session` | Admin | Authentication session |
| `admin_password_hash` | Admin | Hashed password |

### Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         DATA FLOW                                │
└─────────────────────────────────────────────────────────────────┘

1. ADMIN EDITING FLOW
   ┌─────────┐    ┌─────────┐    ┌─────────────┐    ┌─────────┐
   │ Admin   │───►│ Edit    │───►│ Auto-save   │───►│ Save    │
   │ Login   │    │ Tables  │    │ to admin_*  │    │ Changes │
   └─────────┘    └─────────┘    └─────────────┘    └────┬────┘
                                                         │
                                                         ▼
                                              ┌─────────────────┐
                                              │ greenstar_      │
                                              │ admin_data      │
                                              └─────────────────┘

2. EXCEL IMPORT FLOW
   ┌─────────┐    ┌─────────┐    ┌─────────────┐    ┌─────────┐
   │ Upload  │───►│ SheetJS │───►│ Transform   │───►│ Update  │
   │ .xlsx   │    │ Parse   │    │ to Objects  │    │ State   │
   └─────────┘    └─────────┘    └─────────────┘    └─────────┘

3. MAIN APP LOADING FLOW
   ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐
   │ Page Load   │───►│ Check       │───►│ Transform to    │
   │             │    │ admin_data  │    │ categoriesData  │
   └─────────────┘    └──────┬──────┘    └─────────────────┘
                             │
                      ┌──────┴──────┐
                      │ Not Found   │
                      ▼             ▼
              ┌─────────────┐ ┌─────────────┐
              │ Use Default │ │ Use Admin   │
              │ Data        │ │ Data        │
              └─────────────┘ └─────────────┘

4. FAQ LOADING FLOW
   ┌─────────────┐    ┌─────────┐    ┌─────────────┐
   │ Fetch Excel │───►│ SheetJS │───►│ Filter by   │
   │ File        │    │ Parse   │    │ Credit Name │
   └─────────────┘    └─────────┘    └─────────────┘
```

---

## 5. User Flows

### Green Star User Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    GREEN STAR USER JOURNEY                       │
└─────────────────────────────────────────────────────────────────┘

1. PROJECT SETUP
   ┌─────────┐    ┌─────────────┐    ┌─────────────┐
   │ Open    │───►│ Name        │───►│ Select      │
   │ App     │    │ Project     │    │ Template    │
   └─────────┘    └─────────────┘    └──────┬──────┘
                                            │
                              ┌─────────────┴─────────────┐
                              ▼                           ▼
                       ┌─────────────┐           ┌─────────────┐
                       │ Use         │           │ Start       │
                       │ Template    │           │ Fresh       │
                       └─────────────┘           └─────────────┘

2. CREDIT SELECTION
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Browse      │───►│ View Credit │───►│ Select      │
   │ Categories  │    │ Details     │    │ Level       │
   └─────────────┘    └─────────────┘    └──────┬──────┘
                                                │
                                                ▼
                                         ┌─────────────┐
                                         │ Track       │
                                         │ Progress    │
                                         └─────────────┘

3. DOCUMENTATION
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Open Credit │───►│ View        │───►│ Check Off   │
   │ Modal       │    │ Checklist   │    │ Items       │
   └─────────────┘    └─────────────┘    └──────┬──────┘
                                                │
                                                ▼
                                         ┌─────────────┐
                                         │ Add Notes   │
                                         │ & Docs      │
                                         └─────────────┘

4. EXPORT & REVIEW
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Gap         │───►│ Generate    │───►│ Export      │
   │ Analysis    │    │ Report      │    │ PDF/Excel   │
   └─────────────┘    └─────────────┘    └─────────────┘
```

### Administrator Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                      ADMIN USER JOURNEY                          │
└─────────────────────────────────────────────────────────────────┘

1. AUTHENTICATION
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Open Admin  │───►│ Enter       │───►│ Access      │
   │ Panel       │    │ Password    │    │ Dashboard   │
   └─────────────┘    └─────────────┘    └─────────────┘

2. DATA MANAGEMENT
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Select      │───►│ Edit Data   │───►│ Auto-Save   │
   │ Data Type   │    │ in Table    │    │ to Browser  │
   └─────────────┘    └─────────────┘    └──────┬──────┘
                                                │
                                                ▼
                                         ┌─────────────┐
                                         │ Save to     │
                                         │ Main App    │
                                         └─────────────┘

3. EXCEL WORKFLOW
   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
   │ Export      │───►│ Edit in     │───►│ Import      │
   │ to Excel    │    │ Excel       │    │ Updated     │
   └─────────────┘    └─────────────┘    └─────────────┘
```

---

## 6. Proposed Features

### 6.1 Data & Content Enhancements

#### 6.1.1 Editable Project Templates
**Current State**: Project templates are hardcoded in `index.html` as `projectTemplates` array.

**Proposed Solution**:
```
┌─────────────────────────────────────────────────────────────────┐
│                 PROJECT TEMPLATES FLOW                           │
└─────────────────────────────────────────────────────────────────┘

Admin Panel                              Main App
┌─────────────────┐                     ┌─────────────────┐
│ Templates       │     Save Changes    │ loadTemplates() │
│ Editor          │────────────────────►│                 │
│                 │                     │ Check localStorage
│ - Add/Edit/     │                     │ for admin data  │
│   Delete        │                     │                 │
│ - Building Type │                     │ Fall back to    │
│ - Target Rating │                     │ defaults        │
│ - Credit List   │                     └─────────────────┘
└─────────────────┘
```

**Implementation**:
1. Add `templates` to `greenstar_admin_data` on Save
2. Add `loadTemplatesData()` function in main app
3. Replace hardcoded `projectTemplates` with dynamic loading

#### 6.1.2 Editable Calculators List
**Current State**: Calculators are hardcoded in `calculatorsData` array.

**Proposed Solution**:
- Add "Calculators" tab to Admin Panel
- Fields: ID, Name, Description, Icon, Type, Credit Link, External URL
- Load dynamically in main app

**Data Structure**:
```javascript
{
  id: 'responsible-products',
  name: 'Responsible Products Calculator',
  description: 'Calculate responsible products percentage',
  icon: 'fa-boxes-stacked',
  type: 'products',
  creditId: 'resp-struct',  // Link to related credit
  externalUrl: ''           // Optional external calculator link
}
```

#### 6.1.3 Common Mistakes Editing
**Current State**: Common mistakes are embedded in credit definitions.

**Proposed Solution**:
- Add "Common Mistakes" column to Credits editor
- Store as comma-separated or JSON array
- Display in credit detail modal

---

### 6.2 User Experience Enhancements

#### 6.2.1 Reload Data Button
**Purpose**: Allow users to reload admin data without full page refresh.

**Implementation**:
```
┌─────────────────────────────────────────────────────────────────┐
│                      RELOAD DATA FLOW                            │
└─────────────────────────────────────────────────────────────────┘

┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ User Clicks │───►│ Re-run      │───►│ Update      │
│ "Reload"    │    │ loadData()  │    │ React State │
└─────────────┘    └─────────────┘    └─────────────┘
                                             │
                                             ▼
                                      ┌─────────────┐
                                      │ UI Updates  │
                                      │ Automatically│
                                      └─────────────┘
```

**UI Location**: Header toolbar, next to dark mode toggle

**Component**:
```jsx
<button onClick={reloadAdminData} title="Reload admin data">
  <i className="fa-solid fa-rotate"></i>
</button>
```

#### 6.2.2 Admin Data Notification
**Purpose**: Show users when admin-managed data is being used.

**Implementation**:
- Check if `greenstar_admin_data` exists on load
- Display subtle banner: "Using custom data from Admin Panel"
- Include timestamp of last admin save
- Option to "Reset to defaults"

**UI Mockup**:
```
┌─────────────────────────────────────────────────────────────────┐
│ ℹ️ Using custom data (Last updated: Jan 31, 2026 2:30 PM)       │
│                                          [Reset to Defaults]    │
└─────────────────────────────────────────────────────────────────┘
```

#### 6.2.3 Admin Panel Search/Filter
**Purpose**: Find data quickly in large tables.

**Implementation**:
```
┌─────────────────────────────────────────────────────────────────┐
│                    ADMIN SEARCH UI                               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Credits                                                          │
├─────────────────────────────────────────────────────────────────┤
│ 🔍 [Search credits...        ] [Category: All ▼] [44 results]   │
├─────────────────────────────────────────────────────────────────┤
│ ID        │ Name              │ Category    │ Difficulty │ ...  │
│───────────│───────────────────│─────────────│────────────│──────│
│ ind-dev   │ Industry Dev...   │ Responsible │ Easy       │      │
│ resp-const│ Responsible Con...│ Responsible │ Medium     │      │
└─────────────────────────────────────────────────────────────────┘
```

**Features**:
- Real-time text search across all columns
- Dropdown filters for category, difficulty, required status
- Result count display
- Highlight matching text

#### 6.2.4 Bulk Edit Capabilities
**Purpose**: Edit multiple rows at once.

**Implementation**:
```
┌─────────────────────────────────────────────────────────────────┐
│                    BULK EDIT FLOW                                │
└─────────────────────────────────────────────────────────────────┘

1. Select Mode
   ┌─────────────────────────────────────────────────────────────┐
   │ [☑] Select All    [3 selected]    [Bulk Edit] [Delete]      │
   ├─────────────────────────────────────────────────────────────┤
   │ [☑] │ ind-dev   │ Industry Development    │ Easy       │    │
   │ [☑] │ resp-const│ Responsible Construction│ Medium     │    │
   │ [☑] │ verify... │ Verification and Hand...│ Medium     │    │
   │ [ ] │ resp-res..│ Responsible Resource... │ Medium     │    │
   └─────────────────────────────────────────────────────────────┘

2. Bulk Edit Modal
   ┌─────────────────────────────────────────────────────────────┐
   │ Bulk Edit (3 credits selected)                              │
   ├─────────────────────────────────────────────────────────────┤
   │ Change Difficulty:  [Easy ▼]                                │
   │ Change Category:    [-- No Change -- ▼]                     │
   │ Change Required:    [-- No Change -- ▼]                     │
   │                                                              │
   │                              [Cancel]  [Apply to 3 items]   │
   └─────────────────────────────────────────────────────────────┘
```

---

### 6.3 Export & Reporting

#### 6.3.1 Export Project Progress to Excel/PDF
**Purpose**: Generate comprehensive project status report.

**Excel Export Structure**:
```
Sheet 1: Summary
┌─────────────────────────────────────────────────────────────────┐
│ Project: My Office Building                                      │
│ Date: January 31, 2026                                          │
│ Target: 5 Star (60 points)                                      │
│ Current: 52 points (87%)                                        │
│ Gap: 8 points needed                                            │
└─────────────────────────────────────────────────────────────────┘

Sheet 2: Credit Status
┌──────────────┬─────────────────────┬────────┬────────┬─────────┐
│ Category     │ Credit              │ Target │ Points │ Status  │
├──────────────┼─────────────────────┼────────┼────────┼─────────┤
│ Responsible  │ Industry Dev        │ Credit │ 1      │ ✓       │
│ Responsible  │ Responsible Const.  │ Except.│ 5      │ ✓       │
│ Healthy      │ Clean Air           │ Credit │ 4      │ Pending │
└──────────────┴─────────────────────┴────────┴────────┴─────────┘

Sheet 3: Documentation Checklist
┌──────────────┬─────────────────────┬──────────────┬────────────┐
│ Credit       │ Document            │ Required     │ Status     │
├──────────────┼─────────────────────┼──────────────┼────────────┤
│ Resp. Const. │ Environmental Mgmt  │ Yes          │ Uploaded   │
│ Resp. Const. │ Waste Report        │ Yes          │ Pending    │
└──────────────┴─────────────────────┴──────────────┴────────────┘

Sheet 4: Notes
┌──────────────┬─────────────────────────────────────────────────┐
│ Credit       │ Notes                                           │
├──────────────┼─────────────────────────────────────────────────┤
│ Clean Air    │ Waiting on HVAC consultant report. Due Feb 15.  │
└──────────────┴─────────────────────────────────────────────────┘
```

**PDF Export**:
- Professional layout with GBCA branding
- Executive summary on first page
- Category-by-category breakdown
- Visual progress charts
- Print-optimized formatting

#### 6.3.2 Submission Checklist Document
**Purpose**: Generate a printable checklist for submission preparation.

**Format**:
```
┌─────────────────────────────────────────────────────────────────┐
│            GREEN STAR SUBMISSION CHECKLIST                       │
│                    Project: My Office Building                   │
│                    Target: 5 Star Rating                         │
└─────────────────────────────────────────────────────────────────┘

□ RESPONSIBLE CATEGORY
  □ Industry Development (Credit Level - 1 point)
    □ Evidence of professional development completion
    □ Case study or presentation materials

  □ Responsible Construction (Exceptional - 5 points)
    □ Environmental Management Plan
    □ Waste Management Report (95% diversion)
    □ Site inspection photos

□ HEALTHY CATEGORY
  □ Clean Air (Credit Level - 4 points)
    □ Ventilation calculations
    □ HVAC design report
    □ Air quality monitoring plan

[...continues for all selected credits...]

───────────────────────────────────────────────────────────────────
SUBMISSION NOTES:
• All documents must be in PDF format
• Maximum file size: 50MB per document
• Naming convention: [ProjectID]_[CreditID]_[DocName].pdf
───────────────────────────────────────────────────────────────────
```

#### 6.3.3 Print-Friendly Credit Summary
**Purpose**: One-page summary of each credit for team reference.

**Layout**:
```
┌─────────────────────────────────────────────────────────────────┐
│ CREDIT: Clean Air                              Category: Healthy │
│ Points: 6 max │ Required: Yes │ Difficulty: Medium              │
├─────────────────────────────────────────────────────────────────┤
│ DESCRIPTION                                                      │
│ Ensuring high indoor air quality through ventilation and        │
│ material selection.                                              │
├─────────────────────────────────────────────────────────────────┤
│ PERFORMANCE LEVELS                                               │
│ ┌─────────────┬────────┬───────────────────────────────────────┐│
│ │ Level       │ Points │ Requirements                          ││
│ ├─────────────┼────────┼───────────────────────────────────────┤│
│ │ Minimum     │ 0      │ Meet AS 1668.2 ventilation rates      ││
│ │ Credit      │ 4      │ 50% improvement on code minimum       ││
│ │ Exceptional │ 6      │ 100% improvement + air monitoring     ││
│ └─────────────┴────────┴───────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│ REQUIRED DOCUMENTATION                                           │
│ • Ventilation Calculations                                       │
│ • HVAC Design Report                                             │
│ • Air Quality Monitoring Plan                                    │
├─────────────────────────────────────────────────────────────────┤
│ TIPS                                                             │
│ Early engagement with HVAC consultant is critical. Consider     │
│ demand-controlled ventilation for energy efficiency.             │
├─────────────────────────────────────────────────────────────────┤
│ PROJECT PHASES: Schematic, Design Dev, Documentation             │
│ RELATED CREDITS: Light Quality, Amenity and Comfort              │
└─────────────────────────────────────────────────────────────────┘
```

---

### 6.4 Technical Enhancements

#### 6.4.1 Offline Mode (PWA)
**Purpose**: Allow app to work without internet connection.

**Implementation Architecture**:
```
┌─────────────────────────────────────────────────────────────────┐
│                    PWA ARCHITECTURE                              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│   Browser   │◄──►│ Service Worker  │◄──►│    Cache Storage    │
│             │    │                 │    │                     │
│  index.html │    │ - Intercept     │    │ - HTML/CSS/JS       │
│  admin.html │    │   requests      │    │ - Excel files       │
│             │    │ - Serve from    │    │ - Font files        │
│             │    │   cache         │    │ - Icons             │
└─────────────┘    │ - Background    │    └─────────────────────┘
                   │   sync          │
                   └─────────────────┘
```

**Required Files**:
```
/manifest.json          # PWA manifest
/service-worker.js      # Caching logic
/icons/
  icon-192.png         # App icon
  icon-512.png         # App icon large
```

**manifest.json**:
```json
{
  "name": "Green Star Submission Assistant",
  "short_name": "Green Star",
  "description": "Track and manage Green Star building certification",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#1B4D3E",
  "theme_color": "#1B4D3E",
  "icons": [
    { "src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

**Service Worker Strategy**:
- Cache-first for static assets (HTML, CSS, JS, fonts)
- Network-first for Excel files (FAQ data)
- Background sync for localStorage changes

#### 6.4.2 Auto-Sync Between Browser Tabs
**Purpose**: Keep multiple tabs in sync when data changes.

**Implementation**:
```javascript
// Listen for storage changes from other tabs
window.addEventListener('storage', (event) => {
  if (event.key === 'greenstar_admin_data') {
    // Reload categories data
    const newData = loadCategoriesData();
    setCategoriesData(newData);
    showNotification('Data updated from another tab');
  }

  if (event.key === 'greenstar-project') {
    // Reload project data
    loadProjectData();
  }
});
```

**Flow Diagram**:
```
┌─────────────────────────────────────────────────────────────────┐
│                    CROSS-TAB SYNC                                │
└─────────────────────────────────────────────────────────────────┘

   Tab 1 (Admin)              localStorage              Tab 2 (Main App)
   ┌─────────────┐           ┌─────────────┐           ┌─────────────┐
   │ Save        │──────────►│ Update      │──────────►│ storage     │
   │ Changes     │           │ admin_data  │           │ event       │
   └─────────────┘           └─────────────┘           └──────┬──────┘
                                                              │
                                                              ▼
                                                       ┌─────────────┐
                                                       │ Reload      │
                                                       │ Data        │
                                                       └─────────────┘
```

#### 6.4.3 Data Backup to File
**Purpose**: Allow users to backup and restore all data.

**Backup Format** (JSON):
```json
{
  "version": "1.0",
  "exportedAt": "2026-01-31T14:30:00.000Z",
  "data": {
    "adminData": { /* greenstar_admin_data */ },
    "projectData": { /* greenstar-project */ },
    "projects": [ /* greenstar-projects */ ]
  }
}
```

**UI**:
```
┌─────────────────────────────────────────────────────────────────┐
│ Settings                                                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ Data Backup                                                      │
│ ────────────────────────────────────────────────────────────    │
│                                                                  │
│ [📥 Export Backup]     Download all data as JSON file            │
│                                                                  │
│ [📤 Import Backup]     Restore from backup file                  │
│                        ⚠️ This will overwrite current data       │
│                                                                  │
│ [🗑️ Clear All Data]    Remove all saved data                     │
│                        ⚠️ This cannot be undone                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 6.4.4 Undo/Redo in Admin Panel
**Purpose**: Allow administrators to undo mistakes.

**Implementation**:
```
┌─────────────────────────────────────────────────────────────────┐
│                    UNDO/REDO SYSTEM                              │
└─────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────────────┐
                    │      History Stack          │
                    ├─────────────────────────────┤
                    │ [State 3] ◄── Current       │
                    │ [State 2]                   │
                    │ [State 1]                   │
                    │ [State 0] (Initial)         │
                    └─────────────────────────────┘
                              │
               ┌──────────────┴──────────────┐
               │                             │
               ▼                             ▼
        ┌─────────────┐              ┌─────────────┐
        │   Undo      │              │    Redo     │
        │ (Ctrl+Z)    │              │  (Ctrl+Y)   │
        └─────────────┘              └─────────────┘
```

**State Management**:
```javascript
const [history, setHistory] = useState([initialState]);
const [historyIndex, setHistoryIndex] = useState(0);

const undo = () => {
  if (historyIndex > 0) {
    setHistoryIndex(historyIndex - 1);
    applyState(history[historyIndex - 1]);
  }
};

const redo = () => {
  if (historyIndex < history.length - 1) {
    setHistoryIndex(historyIndex + 1);
    applyState(history[historyIndex + 1]);
  }
};

const recordChange = (newState) => {
  const newHistory = history.slice(0, historyIndex + 1);
  newHistory.push(newState);
  setHistory(newHistory);
  setHistoryIndex(newHistory.length - 1);
};
```

**UI**:
```
┌─────────────────────────────────────────────────────────────────┐
│ [⟲ Undo] [⟳ Redo]    │ Categories │ Last saved: 2 minutes ago  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Technical Specifications

### Browser Support
| Browser | Minimum Version |
|---------|-----------------|
| Chrome | 90+ |
| Firefox | 88+ |
| Safari | 14+ |
| Edge | 90+ |

### Performance Targets
| Metric | Target |
|--------|--------|
| First Contentful Paint | < 1.5s |
| Time to Interactive | < 3s |
| Lighthouse Score | > 90 |
| Bundle Size | < 500KB (excluding CDN) |

### localStorage Limits
- Total storage: ~5-10MB per origin
- Current usage estimate: ~500KB max
- Mitigation: Compress large data, warn on limits

### Accessibility (WCAG 2.1 AA)
- Keyboard navigation for all features
- ARIA labels on interactive elements
- Color contrast ratio > 4.5:1
- Screen reader compatible

---

## 8. Security

### Current Security Measures
| Measure | Implementation |
|---------|----------------|
| Admin Authentication | SHA-256 password hashing |
| Session Management | 24-hour expiry, localStorage |
| Data Isolation | Per-origin localStorage |

### Recommendations for Production
| Enhancement | Priority | Description |
|-------------|----------|-------------|
| HTTPS | Critical | Encrypt all traffic |
| CSP Headers | High | Prevent XSS attacks |
| Server-side Auth | High | Move auth to backend |
| Data Encryption | Medium | Encrypt sensitive localStorage |
| Audit Logging | Medium | Track admin changes |

### Password Security Flow
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ User Input  │───►│ SHA-256     │───►│ Compare to  │
│ Password    │    │ Hash        │    │ Stored Hash │
└─────────────┘    └─────────────┘    └──────┬──────┘
                                             │
                                      ┌──────┴──────┐
                                      │             │
                                      ▼             ▼
                               ┌─────────────┐ ┌─────────────┐
                               │ Grant       │ │ Deny        │
                               │ Access      │ │ Access      │
                               └─────────────┘ └─────────────┘
```

---

## 9. Deployment

### Current Deployment
- Static files served directly
- No build process required
- Dependencies loaded from CDN

### Recommended Production Setup
```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION ARCHITECTURE                       │
└─────────────────────────────────────────────────────────────────┘

                         ┌─────────────────┐
                         │   CloudFlare    │
                         │   (CDN + WAF)   │
                         └────────┬────────┘
                                  │
                         ┌────────┴────────┐
                         │   Web Server    │
                         │   (nginx)       │
                         └────────┬────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
     ┌────────┴────────┐ ┌───────┴───────┐ ┌────────┴────────┐
     │   index.html    │ │  admin.html   │ │  Static Assets  │
     │   (Main App)    │ │  (Admin)      │ │  (Excel, etc)   │
     └─────────────────┘ └───────────────┘ └─────────────────┘
```

### Deployment Checklist
- [ ] Minify HTML/CSS/JS
- [ ] Enable gzip compression
- [ ] Set cache headers
- [ ] Configure HTTPS
- [ ] Set up error monitoring
- [ ] Configure backup schedule
- [ ] Document admin password reset procedure

---

## Appendix A: Component Reference

### Main App Components
| Component | Purpose | Props |
|-----------|---------|-------|
| `App` | Root component | - |
| `Header` | Top navigation bar | projectName, onMenuClick |
| `Sidebar` | Category navigation | categories, selectedCategory |
| `CreditCard` | Individual credit display | credit, category, onSelect |
| `CreditDetailModal` | Full credit details | credit, isOpen, onClose |
| `ProgressBar` | Points progress display | current, target |
| `StarRating` | Star rating display | rating |
| `FilterBar` | Credit filters | filters, onChange |

### Admin Panel Components
| Component | Purpose | Props |
|-----------|---------|-------|
| `LoginScreen` | Authentication | onLogin |
| `Sidebar` | Navigation | activeTab, setActiveTab |
| `Dashboard` | Overview statistics | stats |
| `TableEditor` | Generic data editor | data, columns, onUpdate |
| `CategoriesEditor` | Category management | data, onUpdate |
| `CreditsEditor` | Credit management | data, onUpdate, categories |
| `ImportModal` | Excel import | isOpen, onImport |

---

## Appendix B: Data Validation Rules

### Categories
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| id | string | Yes | Unique, lowercase, alphanumeric + hyphen |
| name | string | Yes | 1-50 characters |
| description | string | No | Max 500 characters |
| icon | string | No | Font Awesome class |
| color | string | Yes | Valid Tailwind color |
| order | number | Yes | Positive integer |

### Credits
| Field | Type | Required | Validation |
|-------|------|----------|------------|
| id | string | Yes | Unique, format: xxx-xxx |
| name | string | Yes | 1-100 characters |
| category | string | Yes | Valid category ID |
| maxPoints | number | Yes | 0-20 |
| difficulty | string | Yes | Easy, Medium, Hard |
| required | boolean | Yes | - |

---

## Appendix C: Keyboard Shortcuts

### Main App
| Shortcut | Action |
|----------|--------|
| `/` | Focus search |
| `Escape` | Close modal |
| `D` | Toggle dark mode |
| `?` | Show help |

### Admin Panel (Proposed)
| Shortcut | Action |
|----------|--------|
| `Ctrl+S` | Save changes |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |
| `Ctrl+F` | Focus search |
| `Escape` | Cancel/Close |

---

*Document Version: 1.0*
*Last Updated: January 31, 2026*
*Author: Green Star Development Team*
