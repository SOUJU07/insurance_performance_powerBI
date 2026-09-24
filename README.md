# Insurance Agency Performance — Power BI

## Overview

This project is an end-to-end Power BI solution for a regional life insurance agency distribution team.

The dashboard focuses on:

- Agent performance against APE targets
- Monthly sales performance
- Product contribution
- Policy persistency
- Agent-level performance analysis
- Territory-based data security

The solution was developed in Power BI Project (PBIP) format and uses a star-schema-based semantic model.

---

## Report Pages

### 1. Executive Summary

Provides a management-level overview of agency performance.

Includes:

- APE YTD
- APE vs Target %
- Active Agent Count
- Persistency Rate
- Monthly APE vs Target trend
- Top 10 Agents by APE
- Region, Channel and Year filters
- Mobile layout

### 2. Agent Performance Grid

Built using the Deneb custom visual with Vega-Lite.

Each agent is compared against their monthly APE target.

Performance is colour-coded as:

- Above Target: >= 100%
- On Track: 90% to < 100%
- Below Target: < 90%

The visual supports navigation to the Agent Profile page using agent-level filter context.

### 3. Agent Profile

Built using a dynamic HTML custom visual.

Displays:

- Agent name and territory
- YTD APE
- Agent rank
- Month-on-month trend
- Top product by APE
- Persistency rate

The page receives the selected agent through drillthrough context.

### 4. Persistency Cohort

Analyzes renewal performance by policy issue cohort.

The page includes:

- Persistency Rate
- Active Policy Count
- Due Policy Count
- Issue Year vs Renewal Year cohort matrix
- Conditional formatting for renewal performance

---

## Data Model

The model follows a star-schema approach with dimension tables filtering fact tables.

Main dimensions:

- dim_date
- dim_agent
- dim_product

Fact tables:

- fact_sales
- fact_target
- fact_persistency

### Date Relationships

For fact_persistency, renewal_due_date_key is the active relationship to dim_date.

This was selected because persistency is primarily measured when a policy becomes due for renewal.

The issue_date_key relationship is inactive and is used explicitly when issue-date context is required.

### Target Modelling

fact_target originally contained year and month instead of a date surrogate key.

A target_date_key was created to connect the monthly target data to the date dimension.

### Missing Agent Handling

Some fact_sales records contained agent IDs that were missing from the supplied agent dimension.

Instead of dropping these fact records, inferred agent members were created so that sales values were preserved and referential integrity was maintained.

---

## DAX Measures

Core measures include:

- Total APE
- APE YTD
- APE MoM Growth %
- APE Rolling 3 Months
- Total APE Target
- APE vs Target %
- Persistency Rate
- Active Policy Count
- Active Agent Count
- Agent Rank by APE

---

## Semi-Additive Active Policy Count

Active Policy Count is treated as a snapshot measure rather than an additive measure.

Policy counts from different dates cannot be summed because the same policy may remain active across multiple reporting periods.

The measure evaluates policy status as of the selected snapshot date using:

- Policy issue date
- Renewal due date
- Renewal status

This ensures that the measure answers:

"How many policies were active as of this date?"

rather than summing historical policy counts.

---

## Persistency Logic

Persistency Rate is calculated as:

Renewed Policies / Policies Due for Renewal

Only these statuses are included in the denominator:

- renewed
- lapsed
- grace

Policies that are not yet due for renewal are excluded.

The renewal due date is therefore used as the primary date context for persistency reporting.

---

## Row-Level Security

A Territory Manager role was implemented on the agent dimension.

A Territory_Access mapping table maps manager UPNs to territories.

USERPRINCIPALNAME() is used to determine the current manager and restrict dim_agent to the permitted territory.

Because dim_agent filters the fact tables, the security context propagates to sales, targets and persistency data.

RLS was validated using Power BI Desktop's View As functionality.

---

## Design Process

Before building the report, an HTML wireframe was created for the Executive Summary page.

The wireframe defined:

- Header
- Filter area
- KPI strip
- Trend area
- Agent ranking area

The wireframe is available at:

/mockups/page1_wireframe.html

---

## Data Quality Decisions

During profiling and transformation:

- Data types were validated
- Fact and dimension keys were checked
- Missing agent dimension members were identified
- Inferred agent records were created rather than dropping valid sales
- Renewal status values were validated
- Target month/date alignment was created explicitly

---

## Improvements With More Time

With additional development time I would:

- Externalize the RLS mapping to a governed user-access source
- Add automated data-quality validation
- Add additional report tooltips and navigation
- Extend the persistency cohort analysis across policy duration
- Enhance accessibility and mobile layouts across all pages
- Add deployment pipelines and environment-specific parameters
- Perform further DAX and semantic-model performance optimization

---

## Technology

- Power BI Desktop
- Power Query
- DAX
- Deneb / Vega-Lite
- HTML Content custom visual
- Git / GitHub
- PBIP format