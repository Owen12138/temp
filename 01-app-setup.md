# BOA Canvas App — Setup (App + Theme + Sample Data)

This is the first file to set up after you create a new blank Canvas app
in Power Apps Studio.

## 1. App settings

Power Apps Studio → File → Settings:

- **Name**: BOA – Business Opportunity Assessment
- **Screen size + orientation**: Tablet (Landscape), Size 16:9, Width 1366 / Height 768
- **Scale to fit**: OFF
- **Lock aspect ratio**: OFF
- **Lock orientation**: ON
- **Show mobile status bar**: OFF

Turn ON/OFF these features in Settings → Upcoming features:

- Modern controls and themes (Preview): **OFF** — modern buttons remove the
  `Fill`, `HoverFill`, `Color`, and `BorderColor` properties and replace them
  with a theme system that is hard to override with exact hex colors. Keep this
  off so every button exposes the full classic property set.
- Named formulas: **ON**
- Enhanced delegation (if shown): **ON**
- **Allow canvas components to navigate: ON** — without this, any
  `Navigate(scrHome, ...)` call inside a component errors because the
  component cannot see screen objects from the main app.

## 2. Screen order in the tree view

**Do this before pasting any formulas.**

In the left Tree view panel, make sure `scrHome` is the topmost screen
(drag it or right-click → Move up). Canvas always renders the first screen
in the tree while `App.OnStart` is still running. If `scrDetail` or another
screen is on top, its controls try to evaluate `selectedUC.Name`,
`theme.Maroon`, etc. before those variables exist — causing red formula
errors in the App Checker.

## 3. App.OnStart

Select the `App` node in the tree view, set `OnStart` to:

```powerfx
// ──────────────────────────────────────────────────────────────
// THEME (matches the HTML prototype's --maroon palette)
// ──────────────────────────────────────────────────────────────
Set(theme, {
    Maroon:       RGBA(122, 26, 46, 1),       // #7A1A2E
    MaroonDeep:   RGBA(94, 19, 34, 1),        // #5E1322
    MaroonLight:  RGBA(242, 230, 233, 1),     // #F2E6E9
    CIBCRed:      RGBA(196, 31, 62, 1),       // #C41F3E

    Bg:           RGBA(245, 245, 245, 1),     // #F5F5F5
    Surface:      RGBA(255, 255, 255, 1),
    Border:       RGBA(225, 225, 225, 1),     // #E1E1E1
    BorderStrong: RGBA(207, 207, 207, 1),     // #CFCFCF

    Ink:          RGBA(37,  37,  37,  1),     // #252525
    Ink2:         RGBA(74,  74,  74,  1),     // #4A4A4A
    Ink3:         RGBA(110, 110, 110, 1),     // #6E6E6E
    Ink4:         RGBA(154, 154, 154, 1),     // #9A9A9A

    Ok:           RGBA(45, 125,  63, 1),      // #2D7D3F
    Warn:         RGBA(197, 139, 26, 1),      // #C58B1A
    Info:         RGBA(31, 111, 178, 1),      // #1F6FB2
    Monitor:      RGBA(74, 124, 140, 1),
    Decom:        RGBA(176, 176, 176, 1),

    FontFamily:   "'Segoe UI', Arial, sans-serif"
});

// ──────────────────────────────────────────────────────────────
// LOOKUPS (will become Dataverse choice columns later)
// ──────────────────────────────────────────────────────────────
ClearCollect(colStatus,
    { Code: "Rationale",      Label: "Rationale",       Color: theme.Ink3 },
    { Code: "DataPrep",       Label: "Data Prep",       Color: theme.Ink3 },
    { Code: "Development",    Label: "Development",     Color: theme.Info },
    { Code: "Testing",        Label: "Testing",         Color: theme.Warn },
    { Code: "Deployment",     Label: "Deployment",      Color: theme.Ok },
    { Code: "Monitoring",     Label: "Monitoring",      Color: theme.Monitor },
    { Code: "Decommissioning",Label: "Decommissioning", Color: theme.Decom }
);

ClearCollect(colSBU,
    "PBB", "Capital Markets", "Wealth", "Commercial", "Direct Banking"
);

ClearCollect(colUseCaseType,
    "AI — Predictive", "AI — Generative", "Non-AI Analytics"
);

ClearCollect(colRefreshFreq,
    "Real-time", "Daily", "Weekly", "Monthly", "Quarterly", "One-time"
);

ClearCollect(colBenefitType,
    "Cost Reduction",
    "Cost/Risk Avoidance",
    "Revenue Enhancement",
    "Capacity Creation",
    "Client/Employee Experience",
    "Balance Sheet"
);

// ──────────────────────────────────────────────────────────────
// SAMPLE USE CASES (replace with Dataverse source later)
// ──────────────────────────────────────────────────────────────
ClearCollect(colUseCases,
    { UCID: "UC-0142", Name: "Credit Risk Churn Predictor",
      SBU: "PBB", Owner: "Jasmine Lee", OwnerInitials: "JL",
      Status: "Development", FY: "F26", RealizedValue: 1800000,
      EstimatedValue: 3600000, LastUpdated: DateAdd(Today(), -2),
      ProblemStatement: "Retail credit attrition in PBB has trended up 2.4pp YoY...",
      AISolution: "Gradient-boosted classifier on 90-day client activity windows...",
      Type: "AI — Predictive", LOB: "Everyday Banking — Credit Cards",
      TargetDate: Date(2026,9,30), RefreshFreq: "Weekly" },
    { UCID: "UC-0141", Name: "Small Business Lending — Next Best Action",
      SBU: "PBB", Owner: "Aanya Roy", OwnerInitials: "AR",
      Status: "Testing", FY: "F26", RealizedValue: 600000,
      EstimatedValue: 1500000, LastUpdated: DateAdd(Today(), -5),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "Small Business Banking", TargetDate: Date(2026,12,31),
      RefreshFreq: "Daily" },
    { UCID: "UC-0140", Name: "Mortgage Default Early Warning",
      SBU: "PBB", Owner: "Daniel Mendoza", OwnerInitials: "DM",
      Status: "Deployment", FY: "F26", RealizedValue: 4200000,
      EstimatedValue: 5000000, LastUpdated: DateAdd(Today(), -7),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "Mortgages", TargetDate: Date(2026,8,15), RefreshFreq: "Weekly" },
    { UCID: "UC-0139", Name: "Contact Centre Sentiment Triage",
      SBU: "Direct Banking", Owner: "Priya Kapoor", OwnerInitials: "PK",
      Status: "Monitoring", FY: "F26", RealizedValue: 3100000,
      EstimatedValue: 3100000, LastUpdated: DateAdd(Today(), -7),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "Contact Centre", TargetDate: Date(2026,3,31), RefreshFreq: "Real-time" },
    { UCID: "UC-0138", Name: "Wealth Client Segmentation v3",
      SBU: "Wealth", Owner: "Marcus Okafor", OwnerInitials: "MO",
      Status: "DataPrep", FY: "F26", RealizedValue: 0,
      EstimatedValue: 2200000, LastUpdated: DateAdd(Today(), -14),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "Wealth Management", TargetDate: Date(2027,3,31), RefreshFreq: "Monthly" },
    { UCID: "UC-0137", Name: "FX Trade Surveillance Anomaly Detection",
      SBU: "Capital Markets", Owner: "Sara Hassan", OwnerInitials: "SH",
      Status: "Development", FY: "F26", RealizedValue: 900000,
      EstimatedValue: 2800000, LastUpdated: DateAdd(Today(), -14),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "FX Trading", TargetDate: Date(2026,11,30), RefreshFreq: "Real-time" },
    { UCID: "UC-0136", Name: "Commercial Credit Document Extraction",
      SBU: "Commercial", Owner: "Thomas Nguyen", OwnerInitials: "TN",
      Status: "Testing", FY: "F26", RealizedValue: 2400000,
      EstimatedValue: 3000000, LastUpdated: DateAdd(Today(), -21),
      ProblemStatement: "", AISolution: "", Type: "AI — Generative",
      LOB: "Commercial Lending", TargetDate: Date(2026,10,1), RefreshFreq: "Daily" },
    { UCID: "UC-0135", Name: "Fraud Detection v3 — Card-Not-Present",
      SBU: "PBB", Owner: "Léa Fortier", OwnerInitials: "LF",
      Status: "Monitoring", FY: "F25", RealizedValue: 8700000,
      EstimatedValue: 8700000, LastUpdated: DateAdd(Today(), -30),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "Cards", TargetDate: Date(2025,12,31), RefreshFreq: "Real-time" },
    { UCID: "UC-0134", Name: "Retail Branch Staffing Forecast",
      SBU: "PBB", Owner: "Ravi Venkat", OwnerInitials: "RV",
      Status: "Rationale", FY: "F26", RealizedValue: 0,
      EstimatedValue: 1100000, LastUpdated: DateAdd(Today(), -30),
      ProblemStatement: "", AISolution: "", Type: "Non-AI Analytics",
      LOB: "Retail Operations", TargetDate: Date(2027,6,30), RefreshFreq: "Monthly" },
    { UCID: "UC-0133", Name: "AML Transaction Monitoring — Tier 2",
      SBU: "Capital Markets", Owner: "Kim Baptiste", OwnerInitials: "KB",
      Status: "Decommissioning", FY: "F25", RealizedValue: 2100000,
      EstimatedValue: 2100000, LastUpdated: DateAdd(Today(), -60),
      ProblemStatement: "", AISolution: "", Type: "AI — Predictive",
      LOB: "AML", TargetDate: Date(2025,6,30), RefreshFreq: "Daily" }
);

// ──────────────────────────────────────────────────────────────
// VALUE ENTRIES (one-to-many child of UseCase)
// ──────────────────────────────────────────────────────────────
ClearCollect(colValueEntries,
    { UCID: "UC-0142", Period: "F26 · Q1", Driver: "Cost Reduction",
      Amount: 900000, Frequency: "Quarterly", Signoff: "Signed off" },
    { UCID: "UC-0142", Period: "F26 · Q2", Driver: "Cost Reduction",
      Amount: 900000, Frequency: "Quarterly", Signoff: "Pending" },
    { UCID: "UC-0142", Period: "F26 · Q3", Driver: "Revenue Enhancement",
      Amount: 0, Frequency: "Quarterly", Signoff: "Not started" }
);

// ──────────────────────────────────────────────────────────────
// GOVERNANCE CHECKS
// ──────────────────────────────────────────────────────────────
ClearCollect(colGovernance,
    { UCID: "UC-0142", Name: "AIRAP — AI Risk Assessment",
      Meta: "Approved · AIRAP-2025-1184", Done: true },
    { UCID: "UC-0142", Name: "CIRA — Compliance & Information Risk",
      Meta: "Approved · CIRA-2025-0742", Done: true },
    { UCID: "UC-0142", Name: "Enterprise AI Inventory",
      Meta: "Registered · MAP-4421", Done: true },
    { UCID: "UC-0142", Name: "Model Validation Approval",
      Meta: "In review — expected by F26 W14", Done: false },
    { UCID: "UC-0142", Name: "Technology Triage",
      Meta: "Submit triage number once environment is confirmed", Done: false }
);

ClearCollect(colTechReview,
    { UCID: "UC-0142", Name: "Code Review",
      Meta: "Completed by Tech Lead · 2026-04-22", Done: true },
    { UCID: "UC-0142", Name: "Algorithm Review",
      Meta: "Reviewed by Sr. DS", Done: true },
    { UCID: "UC-0142", Name: "Folder Structure adheres to standards",
      Meta: "Verified against AAI repo template v3.2", Done: true },
    { UCID: "UC-0142", Name: "Model Monitoring in place",
      Meta: "PSI + decile-rank dashboards pending", Done: false }
);

// ──────────────────────────────────────────────────────────────
// APP STATE
// ──────────────────────────────────────────────────────────────
Set(currentUser, { FullName: "Owen Huang", Initials: "OH" });

// Initialize selectedUC as a TYPED blank — NOT Blank() — so Power Apps
// can resolve selectedUC.Name, selectedUC.UCID etc. on all screens
// before a real record is selected. Using Blank() leaves the type
// unknown and causes App Checker errors on scrDetail controls.
Set(selectedUC, {
    UCID: "", Name: "", SBU: "", Owner: "", OwnerInitials: "",
    Status: "", FY: "", RealizedValue: 0, EstimatedValue: 0,
    LastUpdated: Today(), ProblemStatement: "", AISolution: "",
    Type: "", LOB: "", TargetDate: Today(), RefreshFreq: ""
});

Set(currentSection, "Info");
Set(sideCollapsed, true);
Set(filterSearch, "");
Set(filterStatus, "All Statuses");
Set(filterSBU, "All SBUs");
Set(filterFY, "F26");

// Navigate to landing
Navigate(scrHome, ScreenTransition.None);
```

## 3. Named formulas (App.Formulas)

These don't run every time — they're reactive calculations. Put them in
the `Formulas` property on the `App` node:

```powerfx
// Filtered list driven by the filter card
filteredUseCases =
    Filter(
        colUseCases,
        (filterSearch = "" ||
         filterSearch in UCID ||
         filterSearch in Name ||
         filterSearch in Owner) &&
        (filterStatus = "All Statuses" || Status = filterStatus) &&
        (filterSBU    = "All SBUs"    || SBU    = filterSBU) &&
        (filterFY     = ""            || FY     = filterFY)
    );

// Value rollups for the currently-selected use case
ucValueRows =
    Filter(colValueEntries, UCID = selectedUC.UCID);

ucGovRows =
    Filter(colGovernance, UCID = selectedUC.UCID);

ucTechRows =
    Filter(colTechReview, UCID = selectedUC.UCID);

govDoneCount =
    CountIf(ucGovRows, Done);

govTotalCount =
    CountRows(ucGovRows);

govProgressPct =
    If(govTotalCount = 0, 0, govDoneCount / govTotalCount);

realizedYTD =
    Sum(
        Filter(ucValueRows, Signoff = "Signed off"),
        Amount
    );
```

## 4. App-wide reusable component (recommended)

Create one component called `cmpStatusPill` that takes a Status code and
renders the colored dot + label. You'll reuse it on the list, detail rail,
and value table. The build guide has the exact spec.

Once steps 1–3 are saved, move on to `02-build-guide.md`.
