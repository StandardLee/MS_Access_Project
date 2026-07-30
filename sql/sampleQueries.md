# Sample Queries

This document highlights representative queries used in the Operations Reporting Dashboard project. These queries support quality assurance review, issue monitoring, and monthly reporting workflows in Microsoft Access.

The goal of this file is to document the logic behind the most important queries used in forms and reports.

## QA_Exceptional

### Purpose
Returns daily metric records that require QA review, especially records not marked as `Pass`.

### Logic
This query filters the daily metrics table to show only records with unresolved QA status. It is intended to help reviewers focus on exceptions rather than scanning all daily records.

### Key Fields
- MetricDate
- EnteredBy
- SiteName
- MetricName
- Value
- QAStatus
- Notes

### Used In
- `frmQA_Exceptional`

```
SELECT
    A.MetricDate,
    E.FullName AS EnteredBy,
    C.SiteName,
    D.MetricName,
    A.Value,
    A.QAStatus,
    A.Notes
FROM
    (
        (
            (
                TblDailyMetrics AS A
                INNER JOIN TblProjects AS B ON A.ProjectID = B.ProjectID
            )
            INNER JOIN TblSites AS C ON A.SiteID = C.SiteID
        )
        INNER JOIN TblMetricTypes AS D ON A.MetricTypeID = D.MetricTypeID
    )
    INNER JOIN TblStaff AS E ON A.EnteredBy = E.StaffID
WHERE
    A.QAStatus NOT IN ('Pass')
ORDER BY
    A.MetricDate DESC,
    B.ProjectName,
    E.FullName;
```

## OpenIssueSummary

### Purpose
Summarizes unresolved issues by location and severity.

### Logic
This query groups issue records by site/location and severity, then counts open issues for each group. It supports quick operational monitoring and helps identify where unresolved issues are concentrated.

### Key Fields
- Location
- Severity
- OpenIssueCount

### Used In
- `frmOpenIssueSummary`

### Example Notes
This query works well as the record source for a card-style continuous form because it reduces issue-level detail into a compact summary view.

```
SELECT
    B.SiteName AS location,
    A.Severity,
    Count(*) AS OpenIssueCount
FROM
    TblIssueLog AS A
    INNER JOIN TblSites AS B ON A.SiteID = B.SiteID
WHERE
    A.Status <> 'closed'
GROUP BY
    B.SiteName,
    A.Severity
ORDER BY
    B.SiteName,
    A.Severity;
```

---

## MonthlyReport

### Purpose
Provides monthly summary metrics for each project.

### Logic
This query groups records by reporting month and project, then calculates monthly totals and counts such as files processed, average maximum temperature, pending QA counts, and pass counts.

### Key Fields
- ReportMonth
- ProjectName
- MonthlyFilesProcessed
- MonthlyAvgMaxTemp
- PendingCount
- PassCount

### Used In
- `rptMonthlyProjectPerformance`

### Example Notes
This query is designed as a reporting source rather than a data entry source. It supports grouped output in an Access report.
```
SELECT
    Format(A.MetricDate, 'yyyy-mm') AS ReportMonth,
    B.ProjectName,
    sum(iif(C.MetricName = 'Files Processed', A.Value, 0)) AS MonthlyFilesProcessed,
    Round(
        avg(iif(C.MetricName = 'Max Temp', A.Value, NULL)),
        2
    ) AS MonthlyAvgMaxTemp,
    Sum(IIf(A.QAStatus = 'Pending', 1, 0)) AS PendingCount,
    Sum(IIf(A.QAStatus = 'Pass', 1, 0)) AS PassCount
FROM
    (
        TblDailyMetrics AS A
        INNER JOIN TblProjects AS B ON A.ProjectID = B.ProjectID
    )
    INNER JOIN TblMetricTypes AS C ON A.MetricTypeID = C.MetricTypeID
GROUP BY
    Format(A.MetricDate, 'yyyy-mm'),
    B.ProjectName
ORDER BY
    Format(A.MetricDate, 'yyyy-mm'),
    B.ProjectName;
```
---
