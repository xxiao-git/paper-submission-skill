---
name: paper-submission
version: 1.0.0
description: "End-to-end academic manuscript submission workflow automation via browser (playwright-cli).\n\nCovers 5 phases: (1) Information collection, (2) Submission guideline interpretation, (3) File preprocessing (anonymization, reference formatting), (4) Account registration/login (existing account, ORCID, or email), and (5) Submission system form filling.\n\nIncludes 6 human checkpoints (CP-0 to CP-5) for user review at critical stages.\n\nSupports Editorial Manager, ScholarOne, Frontiers, and other submission systems.\n\nUse this skill when the user wants to submit an academic paper/manuscript to a journal, needs help with the submission process, or mentions journal submission, manuscript submission, or paper submission workflow."
tags:
  - academic
  - manuscript
  - journal
  - submission
  - editorial-manager
  - scholarone
  - playwright
---

# Paper Submission Skill

End-to-end academic manuscript submission workflow automation via browser (playwright-cli).

## Overview

This skill automates the complete paper submission process through 5 phases with 6 human checkpoints. It supports multiple submission systems (Editorial Manager, ScholarOne, Frontiers, etc.) and adapts to each journal's specific requirements.

---

## ⛔ CRITICAL RULE: FINAL SUBMIT BUTTON — AI NEVER CLICKS

**The AI MUST NEVER click any of the following buttons:**

- "Run checks and submit"
- "Submit"
- "Approve submission"
- "Confirm submission"
- "Build PDF for my approval"
- "Submit manuscript"
- Any button whose action results in the manuscript being formally submitted to the journal

**This is an absolute, non-negotiable rule. No exceptions. No excuses.**

**What the AI MUST do at the final step:**

1. Navigate to the Review/Summary page
2. Take a full-page screenshot
3. Generate a complete checklist report covering ALL filled information (Files, Details, Authors, Declarations, Funding, Terms)
4. Present the report to the user with this exact message:
   - "以下是投稿最终审核报告。请人工检查以上所有信息。确认无误后，**请您自己点击 'Run checks and submit' 按钮完成提交。我不会代您点击提交按钮。**"

5. STOP. Do not proceed further. Do not click anything else.

**The human MUST click the final submit button themselves.**

---

## Prerequisites

- **playwright-cli** skill must be available for browser automation
- User must provide manuscript files and related materials
- User may need to provide journal account credentials

---

## Initialization Check

**Before starting any workflow, verify playwright-cli availability:**

1. Check if playwright-cli skill is loaded
   - Try to call: `Skill` with `skill: "playwright-cli"`
   - If successful → continue
   - If fails → proceed to step 2

2. Attempt auto-installation
   - Call: `Skill` with `skill: "marketplace-skill-installer"`
   - Follow instructions to search for and install `playwright-cli`
   - If installation succeeds → continue
   - If installation fails → inform user: "playwright-cli skill is required but could not be installed automatically. Please install it manually from the skill marketplace before using this skill."

3. Verify installation
   - Try to call playwright-cli again
   - If still fails → abort and ask user to resolve dependency

**Important:** Do not proceed with the submission workflow until playwright-cli is confirmed working.

---

## Workflow

### Phase 1: Information Collection

**Goal:** Collect all required materials from the user before starting the submission process.

**Required Information:**

1. **Journal Information**
   - Journal name
   - Submission system URL

2. **Account Information** (ask user which scenario applies):
   - **Scenario A:** User already has an account → ask for username and password
   - **Scenario B:** User needs to register with ORCID → ask for ORCID credentials
   - **Scenario C:** User needs to register with email → ask for email, password, name, institution, department, country

3. **Manuscript Files**
   - Main manuscript file (docx)
   - Cover letter (docx)
   - Figures (if any)
   - Supplementary materials (if any)
   - Title page with author information (if double-blind review required)

4. **Author Information**
   - Use template: `assets/author_info_template.xlsx`
   - Ask user to fill in the template and provide the file path
   - **Multiple affiliations handling**: If an author has multiple affiliations, fill only **one primary affiliation** in this template (the one most relevant to the work or the first listed in the manuscript). Additional affiliations can be listed in the manuscript's Title Page. The submission system only requires one affiliation per author to proceed; this information is for system tracking and may differ from the final published author affiliations.

5. **Funding Information**
   - Use template: `assets/funding_info_template.xlsx`
   - Ask user to fill in the template and provide the file path

6. **Reviewer Recommendations** (if required)
   - Use template: `assets/reviewer_info_template.xlsx`
   - Ask user to fill in the template and provide the file path

7. **Additional Materials** (as needed)
   - Graphical abstract
   - Highlights
   - Keywords

**Checkpoint CP-0:** Present collected information summary to user for confirmation before proceeding.

---

### Phase 2: Submission Guideline Interpretation

**Goal:** Understand the journal's specific submission requirements.

**Process:**

1. Ask user to provide submission guideline text (user can copy-paste from journal website or provide a document)
2. Parse and interpret the guideline to extract:
   - Review type (single-blind, double-blind, open review)
   - Anonymization requirements
   - Reference format requirements
   - File format requirements
   - Specific sections required (highlights, graphical abstract, etc.)
   - Author information requirements
   - Reviewer recommendation requirements
3. Generate a checklist of requirements based on the interpretation

**Checkpoint CP-1:** Present the interpreted requirements checklist to user for confirmation. User verifies that the interpretation is correct before proceeding to file preparation.

---

### Phase 3: File Preprocessing

**Goal:** Prepare all files according to the journal's requirements.

**Task 3.1: File Processing**

Based on journal requirements:
- **If double-blind review required:**
  - Split manuscript into anonymized version and title page
  - Remove all author information from main manuscript
  - Create separate title page with author details
- **If single-blind or open review:**
  - Keep manuscript as-is (no anonymization needed)
- Adapt cover letter to journal name and specific requirements
- Convert file formats if needed

**Task 3.2: Reference Format Conversion**

**Ask user to choose:**
- **Option A:** User will convert references themselves using Endnote/Zotero (provide guidance on target format)
- **Option B:** AI will convert references directly based on journal requirements (parse manuscript, reformat all references)

**Checkpoint CP-2:** Present processed files summary to user. Show what was changed:
- Whether anonymization was performed
- Which references were reformatted
- File structure and naming
- User verifies all changes before proceeding.

---

### Phase 4: Account Login/Registration

**Goal:** Access the submission system.

**Three Scenarios:**

**Scenario A: User Already Has Account**
1. Navigate to submission system login page
2. Enter username and password
3. Log in successfully

**Scenario B: Register with ORCID**
1. Navigate to ORCID login option on submission system
2. Use user's ORCID credentials to authenticate
3. Complete registration form with author information
4. Verify account creation

**Scenario C: Register with Email**
1. Navigate to registration page
2. Fill in registration form:
   - Email address
   - Password
   - First name, Last name
   - Institution
   - Department
   - Country
3. Submit registration
4. Verify account (check email if confirmation required)
5. Log in with new credentials

**Checkpoint CP-3:** Confirm successful login/registration. Show current page screenshot to user.

---

### Phase 5: Submission System Form Filling

**Goal:** Complete the submission form according to the actual submission system's steps.

**Important:** Different submission systems have different interfaces and steps. This phase follows the **actual steps presented by the submission system**, not a fixed sequence.

**General Approach:**

1. **Start new submission** or enter incomplete submission
2. **Follow system prompts** - navigate through each step as the system presents it
3. **Common steps may include:**
   - Manuscript type selection
   - File uploads
   - Title and abstract entry
   - Author information
   - Keywords/classification selection
   - Reviewer recommendations
   - Funding information
   - Conflict of interest declarations
   - Cover letter
   - Additional comments

4. **For each step:**
   - Read the current page content
   - Identify what information is required
   - Fill in using collected data (from Phase 1)
   - Take screenshot after filling
   - Save/continue to next step

5. **Special handling for common elements:**
   - **Author information:** Fill from `author_info_template.xlsx` data, ensure correct order
   - **Funding:** Fill from `funding_info_template.xlsx` data
   - **Reviewers:** Fill from `reviewer_info_template.xlsx` data
   - **File uploads:** Upload the preprocessed files from Phase 3

6. **Build PDF / Generate submission package**
   - Most systems generate a PDF for final review
   - Download or preview the PDF

**Checkpoint CP-4:** After author information is filled, show author list and order to user for confirmation.

**Checkpoint CP-5 (FINAL — MANDATORY HUMAN GATE):**

**⛔ AI MUST NOT click any submit/approve/confirm button. The human must do it.**

1. **AI auto-check using submission guideline checklist** (from Phase 2):
   - Verify title, abstract completeness
   - Check author information (order, affiliations, email)
   - Check file format, page count
   - Check reference format
   - Check anonymization completeness (if required)
   - Check any journal-specific requirements

2. **Generate final review report** including:
   - Full-page screenshot of the Review/Summary page
   - Auto-check results (pass/fail for each item)
   - Complete summary of ALL filled information across all sections (Files, Details, Authors, Declarations, Funding, Terms)

3. **Present the report to user** with this exact closing message:

   > "以下是投稿最终审核报告。请人工检查以上所有信息。
   > 确认无误后，**请您自己点击页面上的提交按钮完成投稿。**
   > 我不会代您点击提交按钮。"

4. **STOP HERE.** Do NOT click the submit button. Do NOT navigate further. End the task.

---

## Checkpoint Protocol

At each checkpoint (CP-0 through CP-5):

1. **Take screenshot** of current state
2. **Generate summary report** of key information
3. **Present to user** with clear statement:
   - "Please review the above information"
   - "Reply 'confirm' or 'ok' to proceed"
   - "Reply 'modify' if changes are needed, and specify what to change"
   - "Reply 'stop' to terminate the process"

4. **Wait for user response** before continuing

---

## Error Handling

- **If a step fails:** Take screenshot, report error to user, ask for guidance
- **If information is missing:** Ask user to provide before proceeding
- **If user wants to go back:** Navigate to previous step and update information
- **If browser automation fails:** Report error, suggest manual intervention or alternative approach

---

## Important Notes

1. **⛔ NEVER CLICK SUBMIT** — The AI is FORBIDDEN from clicking any final submit/approve/confirm button. The human user must click it themselves after reviewing the CP-5 report.
2. **Adapt to actual system** — Phase 5 follows the real submission system's flow, not a fixed template
3. **Preserve user data** — Keep all collected information available for filling forms
4. **Take screenshots** — Document each major step for user review
5. **Handle variations** — Different journals have different requirements, always check guidelines first
6. **Reference conversion choice** — Always ask user whether they want to convert references themselves or have AI do it
7. **Checkpoint discipline** — Every checkpoint (CP-0 through CP-5) MUST pause and wait for user response. Do not skip or rush through checkpoints.

---

## Template Files

The skill includes three Excel templates in the `assets/` folder:

1. **author_info_template.xlsx** - Author information (17 columns)
   - Includes: sequence, title, first name, middle initial, family name, email, phone, institution, department, address lines, city, state, zip, country, ORCID, corresponding author flag

2. **funding_info_template.xlsx** - Funding information (4 columns)
   - Includes: sequence, funder name (English), grant number, grant recipient

3. **reviewer_info_template.xlsx** - Reviewer recommendations (8 columns)
   - Includes: sequence, first name/given name, family name/surname, email, institution, department, country, research area/reason

All templates include bilingual headers (English + Chinese) and example data rows.

---

## Usage

When the user wants to submit a paper to a journal:

1. **Trigger the skill** when user mentions paper submission, journal submission, or manuscript submission
2. **Start with Phase 1** - Ask user to provide all required information using the templates
3. **Proceed through phases** - Follow the workflow sequentially with checkpoints
4. **Adapt as needed** - Adjust based on journal-specific requirements
5. **Complete submission** - Only submit after all checkpoints pass and user confirms

**Example user requests that trigger this skill:**
- "帮我投稿到 Nature Communications"
- "Submit my paper to Cell Reports"
- "I need to submit my manuscript to a journal"
- "使用投稿技能"
- "Help me with the journal submission process"
