# ProfilePulse — Product Requirements Document (PRD)

**Version:** 1.0  
**Date:** May 2026  
**Author:** Sanjay (Product Owner)  
**Status:** Ready for Development

---

## Table of Contents

1. [Product Overview](#1-product-overview)
2. [Problem Statement](#2-problem-statement)
3. [Target Users](#3-target-users)
4. [Legal and Privacy Foundation](#4-legal-and-privacy-foundation)
5. [Data Sources](#5-data-sources)
6. [Page Layout and Screen Design](#6-page-layout-and-screen-design)
7. [KPI Definitions](#7-kpi-definitions)
8. [Charts and Visualizations](#8-charts-and-visualizations)
9. [User Flows](#9-user-flows)
10. [Input Validations and Edge Cases](#10-input-validations-and-edge-cases)
11. [Error Handling](#11-error-handling)
12. [Technical Specifications](#12-technical-specifications)
13. [GitHub API Reference](#13-github-api-reference)
14. [LinkedIn PDF Parsing Reference](#14-linkedin-pdf-parsing-reference)
15. [Profile Strength Score Algorithm](#15-profile-strength-score-algorithm)
16. [Design System](#16-design-system)
17. [Responsive Design Rules](#17-responsive-design-rules)
18. [Success Metrics](#18-success-metrics)
19. [Out of Scope](#19-out-of-scope)
20. [Glossary](#20-glossary)

---

## 1. Product Overview

**Product Name:** ProfilePulse  
**Tagline:** Your Professional Activity at a Glance  
**Type:** Free, browser-based, single-page web application  
**Delivery:** Single HTML file — no installation, no backend, no login

ProfilePulse is a professional activity intelligence dashboard that combines a person's GitHub public data and LinkedIn exported data into one clean, visual report. It helps recruiters evaluate candidates faster and helps job seekers understand how their profile looks from the outside.

---

## 2. Problem Statement

Today, a recruiter checking a candidate must:
- Open LinkedIn manually and read through the profile
- Open GitHub separately and scroll through repositories
- Mentally calculate activity level, experience years, tech stack
- Repeat this for every candidate

This takes 10–15 minutes per candidate. There is no single tool that combines both platforms into a clean, structured dashboard — for free, without login, without legal risk.

**ProfilePulse solves this by:**
- Letting the recruiter enter a GitHub username and instantly see a visual activity report
- Letting candidates upload their own LinkedIn export PDF and see a structured profile summary
- Combining both into one Profile Strength Score

---

## 3. Target Users

| User Type | Goal | How ProfilePulse Helps |
|---|---|---|
| **Recruiter** | Evaluate candidate activity quickly | GitHub KPIs + LinkedIn summary in one view |
| **Hiring Manager** | Check tech stack and project quality | Language charts, repo list, experience timeline |
| **Job Seeker** | Audit their own profile before applying | See exactly what recruiters see |
| **College Student** | Understand their profile strength | Profile Strength Score with gap analysis |
| **Freelancer** | Share proof of work | Generate a clean shareable report |

---

## 4. Legal and Privacy Foundation

This section is non-negotiable. Every feature must comply with these rules.

### 4.1 What Is Allowed

| Action | Legal Basis |
|---|---|
| Fetch GitHub public profiles via GitHub REST API | GitHub's public API terms allow this |
| Parse user-uploaded LinkedIn PDF export | User owns their own data; they choose to upload it voluntarily |
| Display parsed data in the browser | No data leaves the user's device |

### 4.2 What Is Strictly Prohibited

| Action | Reason |
|---|---|
| Accepting a LinkedIn profile URL as input | LinkedIn prohibits automated data access via URL |
| Scraping any website (LinkedIn or GitHub) without API | Violates Terms of Service |
| Storing user data on a server | Privacy violation; not needed for this app |
| Sending LinkedIn PDF content to any external server | User data must stay in the browser |
| Tracking user behavior with cookies or analytics | Not permitted in this product |

### 4.3 Privacy Statement (shown in app footer)

> "ProfilePulse does not store, transmit, or share any data. GitHub data is fetched in real time from GitHub's public API. LinkedIn PDF data is processed entirely in your browser and never sent anywhere. Refreshing the page clears all data."

---

## 5. Data Sources

### 5.1 GitHub Data Source

- **Method:** GitHub REST API (unauthenticated, public endpoints)
- **Base URL:** `https://api.github.com`
- **Rate Limit:** 60 requests per hour per IP (unauthenticated)
- **Auth Required:** No (for public profiles)
- **Trigger:** User enters a GitHub username and clicks "Fetch Profile"

**API Endpoints Used:**

| Endpoint | Data Fetched |
|---|---|
| `GET /users/{username}` | Name, bio, location, followers, following, public repos, account created date, avatar URL, blog, company |
| `GET /users/{username}/repos?per_page=100&sort=updated` | All public repos: name, description, language, stars, forks, created date, updated date, topics |
| `GET /users/{username}/events/public?per_page=100` | Recent activity: push events, PR events, issue events |

### 5.2 LinkedIn Data Source

- **Method:** User uploads their own LinkedIn data export file (PDF format)
- **Parsing:** PDF.js library, runs entirely in the browser (client-side)
- **Trigger:** User clicks "Upload LinkedIn PDF" and selects a file
- **File Type Accepted:** PDF only (`.pdf`)
- **No URL input field exists anywhere in the app**

**How to Get a LinkedIn Export (shown as tooltip in app):**
1. Go to LinkedIn → Settings & Privacy
2. Click "Data Privacy" → "Get a copy of your data"
3. Select "Want something in particular?" → choose Profile
4. Download and save the PDF file
5. Upload that file to ProfilePulse

**Fields Extracted from LinkedIn PDF:**

| Field | Source Section in PDF | Fallback if Missing |
|---|---|---|
| Full name | Top of profile | "Name not found" |
| Headline | Below name | "Not available" |
| Location | Profile header | "Not available" |
| About/Summary | About section | "Not available" |
| Connections count | Profile header | "Not available" |
| Positions / Work experience | Experience section | Show 0 positions |
| Education | Education section | "Not available" |
| Skills | Skills section | Show 0 skills |
| Certifications | Licenses & Certifications section | Show 0 certifications |
| Languages | Languages section | "Not available" |
| Volunteer experience | Volunteering section | Skip if missing |
| Profile creation date | Not reliably in export — do not show | N/A |

---

## 6. Page Layout and Screen Design

The app is a single HTML page. No routing. No page changes. Everything renders on one screen.

### 6.1 Overall Page Structure (Top to Bottom)

```
+--------------------------------------------------+
|  HEADER — App name, tagline, nav links           |
+--------------------------------------------------+
|  INPUT SECTION                                   |
|  [GitHub Input Card]   [LinkedIn Input Card]     |
+--------------------------------------------------+
|  DASHBOARD SECTION (renders after data loads)    |
|  [Profile Summary Bar]                           |
|  [Profile Strength Score — large number]         |
|  [KPI Cards Row — GitHub]                        |
|  [KPI Cards Row — LinkedIn]                      |
|  [Charts Row 1 — Language + Contribution]        |
|  [Charts Row 2 — Career Timeline + Skills]       |
|  [Repositories List]                             |
|  [LinkedIn Experience Timeline]                  |
+--------------------------------------------------+
|  FOOTER — Privacy note, credits                  |
+--------------------------------------------------+
```

### 6.2 Header

- **App name:** "ProfilePulse" — large, bold, blue (`#0A66C2`)
- **Tagline:** "Your Professional Activity at a Glance" — grey subtitle
- **Right side:** Two small links: "How to export LinkedIn data" (opens tooltip) and "About"
- **Sticky:** Yes — header stays at top on scroll

### 6.3 Input Section

Two side-by-side cards on desktop, stacked on mobile.

**GitHub Input Card:**
```
+---------------------------------+
| GitHub logo icon                |
| "Enter GitHub Username"         |
| [_________________________]     |  ← text input, placeholder: "e.g. torvalds"
| [   Fetch GitHub Profile   ]    |  ← blue button
| Small text: "Public profiles only" |
+---------------------------------+
```

- Input field accepts only alphanumeric characters and hyphens (GitHub username rules)
- No @ symbol needed
- On Enter key press — same action as clicking Fetch button
- If already fetched — show a "Clear" button to reset

**LinkedIn Input Card:**
```
+---------------------------------+
| LinkedIn logo icon              |
| "Upload Your LinkedIn Export"   |
| [  📄 Choose PDF File  ]        |  ← file input, accept=".pdf" only
| Small text: "How to export?" → tooltip |
| Small text: "Your file never leaves your browser" |
+---------------------------------+
```

- Only `.pdf` files accepted — reject all other formats with error message
- No URL field — no text input in this card at all
- After file selected: show filename + green checkmark
- "Remove file" link appears after upload to clear selection

### 6.4 Dashboard Section

Hidden by default. Renders progressively as data loads:
- GitHub section renders as soon as GitHub API call completes
- LinkedIn section renders as soon as PDF is parsed
- Combined score renders when both sections have data; if only one is available, shows partial score with note

**Profile Summary Bar (top of dashboard):**
```
[Avatar]  Name (from GitHub or LinkedIn)
          GitHub: @username  |  LinkedIn: Headline
          Location  |  Account Age
```

**Profile Strength Score:**
- Large circular gauge (0–100)
- Color: Red (0–40), Orange (41–70), Green (71–100)
- Label below: "Profile Strength Score"
- Sub-label: "Based on GitHub activity + LinkedIn completeness"
- If only one source: "Partial score — add [GitHub/LinkedIn] for full score"

### 6.5 KPI Cards

Two rows of cards. Each card has:
- Icon (relevant to the metric)
- Large number (the KPI value)
- Label below the number
- Subtle border and shadow
- Hover: slight lift animation

**GitHub KPI Row (6 cards):**

| Card | Value | Icon |
|---|---|---|
| Public Repositories | Count | 📁 |
| Total Stars | Sum of all repo stars | ⭐ |
| Top Language | Most used language name | 💻 |
| Followers | GitHub followers count | 👥 |
| Account Age | Years since account created | 📅 |
| Last Active | Days since last public event | 🟢 |

**LinkedIn KPI Row (6 cards):**

| Card | Value | Icon |
|---|---|---|
| Connections | Count from export | 🔗 |
| Years Experience | Calculated from positions | 💼 |
| Skills Listed | Count of skills | 🛠️ |
| Certifications | Count | 🏆 |
| Positions Held | Count of job entries | 🏢 |
| Education Level | Highest degree found | 🎓 |

### 6.6 Charts Section

**Row 1 — Two charts side by side:**

Chart A — Language Distribution (Donut Chart)
- Shows percentage breakdown of programming languages across all repos
- Calculated by: count of repos per language / total repos with language
- Top 6 languages shown; rest grouped as "Others"
- Legend on the right
- Hover: shows language name + percentage + repo count

Chart B — GitHub Contribution Activity (Bar Chart)
- X-axis: Last 12 months (month labels)
- Y-axis: Number of public events (pushes, PRs, issues)
- Data source: `/users/{username}/events/public`
- Grouped by month from event timestamps
- Hover: shows month + event count

**Row 2 — Two charts side by side:**

Chart C — Skills Bar Chart (Horizontal Bar)
- Top 10 skills from LinkedIn export
- X-axis: not shown (just visual bars)
- Y-axis: Skill names
- All bars same color (LinkedIn blue), no ranking implied

Chart D — Profile Completeness Radar (Radar/Spider Chart)
- 6 axes: GitHub Activity, Repos Quality, LinkedIn Experience, Skills, Certifications, Education
- Filled polygon showing score per axis (0–100 each)
- Helps identify which area needs improvement

### 6.7 Repositories List

Below charts. Shows top 6 repositories sorted by stars descending.

Each repo card:
```
+----------------------------------------------+
| Repo Name (clickable → opens GitHub)         |
| Description (truncated to 100 chars)         |
| [Language badge] [⭐ Stars] [🍴 Forks]       |
| Last updated: X days ago                     |
+----------------------------------------------+
```

- "View all repositories on GitHub" link at the bottom → opens GitHub profile

### 6.8 LinkedIn Experience Timeline

Horizontal timeline (vertical on mobile) showing career history.

Each position:
```
[Company Logo placeholder] → Company Name
                              Job Title
                              Start Date – End Date (Duration calculated)
                              Location (if available)
```

- Sorted newest to oldest
- Duration auto-calculated: "2 years 3 months"
- Current roles show "Present" as end date

### 6.9 Footer

```
ProfilePulse — Free, open, privacy-first.
We do not store any data. Everything runs in your browser.
Built with GitHub API + PDF.js + Chart.js
[GitHub Repo link if published]
```

---

## 7. KPI Definitions

Every KPI must be precisely defined so the calculation is consistent.

### GitHub KPIs

**Total Public Repositories**
- Source: `user.public_repos` from `/users/{username}`
- Display: plain number, e.g. "42"
- Edge case: if 0, show "0" with note "No public repos yet"

**Total Stars**
- Source: sum of `stargazers_count` across all repos fetched
- Note: GitHub API returns max 100 repos per page — if user has more than 100 repos, paginate using `?page=2`, `?page=3` until empty response
- Display: "1.2k" format for numbers above 1000

**Top Language**
- Source: group repos by `language` field, count per language, return language with highest count
- Ignore repos where `language` is null
- If all repos have null language: show "Not detected"
- Display: language name as text, e.g. "Python"

**Followers**
- Source: `user.followers` from `/users/{username}`
- Display: plain number

**Account Age**
- Source: `user.created_at` from `/users/{username}`
- Calculation: `(today - created_at) / 365` rounded to 1 decimal
- Display: "3.5 years" or "8 months" if less than 1 year

**Last Active**
- Source: most recent event timestamp from `/users/{username}/events/public`
- Calculation: days since that event
- Display: "2 days ago" / "Today" / "3 weeks ago"
- Edge case: if events list is empty, show "No recent activity"

### LinkedIn KPIs

**Total Connections**
- Source: parse text "500+ connections" or "X connections" from PDF
- Regex pattern: look for number followed by "connections" or "connection"
- If "500+" found: display "500+" exactly
- Edge case: if not found, show "Not in export"

**Years of Experience**
- Source: all positions in Experience section
- Calculation: for each position, parse start and end date. Sum all durations. Convert to years.
- Handle "Present" as today's date
- Handle month-only dates (e.g. "Jan 2020 – Mar 2022")
- Display: "5.5 years total experience"
- Edge case: if dates not parseable, show "Not calculated"

**Skills Listed**
- Source: Skills section of PDF
- Count: total number of skill entries found
- Display: plain number, e.g. "18 skills"

**Certifications**
- Source: Licenses & Certifications section
- Count: number of certification entries
- Display: plain number

**Positions Held**
- Source: Experience section
- Count: number of job entries (each company/role = 1 position)
- Display: plain number

**Education Level**
- Source: Education section
- Logic: find highest degree keyword in text
  - PhD / Doctorate → "Doctorate"
  - Master's / M.Tech / MBA / MSc → "Master's Degree"
  - Bachelor's / B.Tech / BE / BSc / BCA → "Bachelor's Degree"
  - Diploma → "Diploma"
  - If none found → "Not available"
- Display: degree label as text

---

## 8. Charts and Visualizations

All charts use Chart.js (loaded from CDN). All charts must render within 1 second of data availability.

### 8.1 Language Distribution — Donut Chart

```
Type: doughnut
Library: Chart.js
Data: { labels: ["Python", "JavaScript", ...], datasets: [{ data: [40, 30, ...] }] }
Colors: Use a fixed palette of 7 colors:
  ["#0A66C2", "#21C55D", "#F59E0B", "#EF4444", "#8B5CF6", "#06B6D4", "#F97316"]
  Last slice (Others): "#94A3B8"
Options:
  cutout: "65%"
  plugins.legend.position: "right"
  plugins.tooltip: show label + percentage + count
```

Edge cases:
- If only 1 language: show single color donut with 100%
- If no language data: show empty state message "No language data available"

### 8.2 GitHub Activity — Bar Chart

```
Type: bar
Library: Chart.js
Data: Last 12 months, event count per month
X-axis labels: ["Jun '25", "Jul '25", ..., "May '26"] — last 12 months
Y-axis: integer, min 0
Bar color: #0A66C2 with 0.8 opacity
Options:
  responsive: true
  scales.y.beginAtZero: true
  plugins.tooltip: show "Month: X events"
```

Event types to count: `PushEvent`, `PullRequestEvent`, `IssuesEvent`, `CreateEvent`, `ForkEvent`

Edge cases:
- If no events in a month: bar height = 0 (still show month on X-axis)
- If events API returns empty: show "No public activity data available"

### 8.3 Skills Horizontal Bar Chart

```
Type: bar (indexAxis: 'y')
Library: Chart.js
Data: top 10 skills from LinkedIn
Y-axis: skill names (truncated to 20 chars if longer)
X-axis: hidden (bars are decorative, not scored)
Bar color: #0A66C2
All bars same fixed width representing "Listed" status
Options:
  indexAxis: 'y'
  scales.x: { display: false }
  plugins.legend: { display: false }
```

Edge cases:
- If fewer than 10 skills: show however many exist
- If 0 skills: show "No skills data in export"

### 8.4 Profile Completeness — Radar Chart

```
Type: radar
Library: Chart.js
Axes (6): 
  "GitHub Activity" — score 0–100 based on event frequency
  "Repo Quality" — score based on avg stars + descriptions
  "Work Experience" — score based on years
  "Skills" — score based on count
  "Certifications" — score based on count
  "Education" — score based on level detected

Fill: rgba(10, 102, 194, 0.2)
Border: #0A66C2
Point: filled blue dots
Options:
  scales.r.min: 0
  scales.r.max: 100
  scales.r.ticks: display: false
```

Axis score calculations:
- GitHub Activity: min(events_last_30_days * 5, 100)
- Repo Quality: min((avg_stars * 10) + (repos_with_description_pct), 100)
- Work Experience: min(years_experience * 10, 100)
- Skills: min(skills_count * 5, 100)
- Certifications: min(cert_count * 20, 100)
- Education: Doctorate=100, Master's=80, Bachelor's=60, Diploma=40, None=0

If data for an axis is unavailable (source not loaded): axis score = 0, show tooltip "Add [source] to score this axis"

---

## 9. User Flows

### 9.1 Flow A — GitHub Only

1. User opens the app
2. User types a GitHub username in the GitHub input field
3. User clicks "Fetch GitHub Profile" or presses Enter
4. Loading spinner appears on the GitHub card
5. App calls `GET /users/{username}` — if 404 returned: show error "Profile not found"
6. If success: app calls `GET /users/{username}/repos?per_page=100`
7. App calls `GET /users/{username}/events/public?per_page=100`
8. All API calls complete → GitHub KPI cards animate in
9. Language donut chart renders
10. Activity bar chart renders
11. Repositories list renders
12. Profile Strength Score renders (partial, GitHub-only)
13. LinkedIn section shows "Upload LinkedIn PDF to see full score"

### 9.2 Flow B — LinkedIn Only

1. User opens the app
2. User clicks "Choose PDF File" on the LinkedIn card
3. File picker opens — only PDF accepted
4. User selects file
5. Filename appears with checkmark
6. "Parsing PDF..." message appears
7. PDF.js loads and parses the file in browser
8. LinkedIn KPI cards animate in
9. Skills bar chart renders
10. Career timeline renders
11. Profile Strength Score renders (partial, LinkedIn-only)
12. GitHub section shows "Enter a GitHub username to see full score"

### 9.3 Flow C — Both Sources (Full Dashboard)

1. User loads GitHub profile (Flow A steps 1–12)
2. User uploads LinkedIn PDF (Flow B steps 3–10)
3. Combined Profile Strength Score updates to full score
4. Radar chart renders with all 6 axes filled
5. Profile Summary Bar shows combined info (GitHub avatar + LinkedIn headline)

### 9.4 Flow D — Reset / Clear

1. User clicks "Clear" on GitHub input → GitHub data clears, GitHub KPIs hide, charts hide
2. User clicks "Remove file" on LinkedIn card → LinkedIn data clears, LinkedIn KPIs hide
3. If both cleared → Dashboard section hides completely

---

## 10. Input Validations and Edge Cases

### GitHub Username Validation

| Rule | Message Shown |
|---|---|
| Empty input on submit | "Please enter a GitHub username" |
| Contains spaces | "GitHub usernames cannot contain spaces" |
| Contains special chars (not `-`) | "GitHub usernames can only contain letters, numbers, and hyphens" |
| Starts or ends with hyphen | "GitHub username cannot start or end with a hyphen" |
| Longer than 39 characters | "GitHub usernames are max 39 characters" |
| Valid but profile not found (404) | "GitHub profile not found. Please check the username." |
| Rate limit exceeded (403) | "GitHub API rate limit reached. Please try again in 1 hour." |
| Network error | "Could not connect to GitHub. Please check your internet connection." |

### LinkedIn PDF Validation

| Rule | Message Shown |
|---|---|
| Non-PDF file selected | "Only PDF files are accepted. Please upload your LinkedIn export PDF." |
| PDF is too large (>10MB) | "File is too large. LinkedIn export PDFs are usually under 2MB." |
| PDF is password protected | "This PDF is password protected. Please upload an unprotected export." |
| PDF parses but no recognizable LinkedIn data found | "This does not look like a LinkedIn export PDF. Please check the file." |
| PDF.js fails to load | "Could not parse the PDF. Please try a different file or browser." |

### LinkedIn Data Field Edge Cases

| Field | Edge Case | Handling |
|---|---|---|
| Connections | Shows "500+" | Display as "500+" — do not try to parse as number |
| Date ranges | "Present" as end date | Use today's date for calculation |
| Date ranges | Month missing (just year) | Use January of that year as start |
| Positions | Overlapping dates | Sum durations as-is, no deduplication |
| Skills | Duplicate skills | Count each occurrence (LinkedIn may list duplicates) |
| Education | Multiple degrees | Show highest found; list all in timeline section |
| Name | Multiple names / changed name | Show first name found at top of PDF |

---

## 11. Error Handling

### General Rules

- Never show a blank card — always show either data or a fallback message
- Never show a JavaScript error to the user — all errors caught and shown as friendly messages
- Never show "undefined", "null", or "NaN" in the UI
- All error messages shown inside the relevant card/section, not as browser alerts

### Error States for Each Section

**GitHub Fetch Error:**
- Show a red-bordered card with error icon and message
- Show "Try Again" button that re-triggers the fetch
- Do not show any GitHub KPI cards in error state

**LinkedIn Parse Error:**
- Show a yellow-bordered card with warning icon and message
- Show "Try a different file" button
- Do not show any LinkedIn KPI cards in error state

**Partial Data (some fields missing):**
- Show the card with available data
- Missing fields show grey text: "Not available in export"
- Card does not disappear — it renders with what it has

**Both Sources Empty (no input yet):**
- Dashboard section is hidden
- Show a welcoming prompt: "Enter a GitHub username or upload a LinkedIn export PDF to get started"

---

## 12. Technical Specifications

### File Structure

```
profilepulse.html       ← single file, all HTML + CSS + JS inline
```

No external files. No build process. Open in any browser and it works.

### Libraries (CDN only)

```html
<!-- Chart.js for visualizations -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>

<!-- PDF.js for LinkedIn PDF parsing -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script>
  pdfjsLib.GlobalWorkerOptions.workerSrc = 
    'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
</script>
```

No other external dependencies. No frameworks. Vanilla JS only.

### State Management

All app state stored in a single JavaScript object in memory:

```javascript
const state = {
  github: {
    loaded: false,
    username: null,
    user: null,        // raw API response from /users/{username}
    repos: [],         // array of repo objects
    events: [],        // array of event objects
    error: null
  },
  linkedin: {
    loaded: false,
    filename: null,
    rawText: null,     // full extracted text from PDF
    parsed: {          // structured parsed data
      name: null,
      headline: null,
      location: null,
      about: null,
      connections: null,
      positions: [],
      education: [],
      skills: [],
      certifications: [],
      languages: []
    },
    error: null
  }
};
```

State resets when "Clear" is clicked. No persistence between page refreshes.

### GitHub API Calls

```javascript
// Base function for all GitHub API calls
async function githubFetch(endpoint) {
  const url = `https://api.github.com${endpoint}`;
  const response = await fetch(url, {
    headers: { 'Accept': 'application/vnd.github.v3+json' }
  });
  if (response.status === 404) throw new Error('NOT_FOUND');
  if (response.status === 403) throw new Error('RATE_LIMITED');
  if (!response.ok) throw new Error('API_ERROR');
  return response.json();
}

// Fetch all repos with pagination
async function fetchAllRepos(username) {
  let repos = [];
  let page = 1;
  while (true) {
    const batch = await githubFetch(`/users/${username}/repos?per_page=100&page=${page}&sort=updated`);
    repos = repos.concat(batch);
    if (batch.length < 100) break;
    page++;
  }
  return repos;
}
```

### PDF Parsing

```javascript
async function parseLinkedInPDF(file) {
  const arrayBuffer = await file.arrayBuffer();
  const pdf = await pdfjsLib.getDocument({ data: arrayBuffer }).promise;
  
  let fullText = '';
  for (let i = 1; i <= pdf.numPages; i++) {
    const page = await pdf.getPage(i);
    const textContent = await page.getTextContent();
    const pageText = textContent.items.map(item => item.str).join(' ');
    fullText += pageText + '\n';
  }
  
  return parseLinkedInText(fullText);
}
```

### Rendering

- All DOM manipulation via `document.getElementById` and `innerHTML`
- Charts created with `new Chart(ctx, config)` — destroy old chart before re-creating
- Animations: CSS transitions for card fade-in (`opacity 0.4s ease, transform 0.4s ease`)
- Loading spinner: CSS-only rotating border animation

### Browser Support

Must work on:
- Chrome 90+
- Firefox 88+
- Edge 90+
- Safari 14+
- Chrome for Android (mobile)
- Safari iOS (mobile)

---

## 13. GitHub API Reference

### Endpoint 1: User Profile

`GET https://api.github.com/users/{username}`

**Fields used:**

| Field | Used For |
|---|---|
| `login` | Display username |
| `name` | Display name |
| `avatar_url` | Profile photo |
| `bio` | Bio text |
| `location` | Location KPI |
| `blog` | Website link |
| `company` | Company info |
| `public_repos` | Repos KPI |
| `followers` | Followers KPI |
| `following` | Following count |
| `created_at` | Account age calculation |

### Endpoint 2: Repositories

`GET https://api.github.com/users/{username}/repos?per_page=100&sort=updated`

**Fields used per repo:**

| Field | Used For |
|---|---|
| `name` | Repo name |
| `description` | Repo description |
| `language` | Language distribution chart |
| `stargazers_count` | Stars KPI + repo card |
| `forks_count` | Forks on repo card |
| `created_at` | Repo age |
| `updated_at` | Last updated on repo card |
| `html_url` | Clickable link to GitHub |
| `topics` | Tags on repo card |

### Endpoint 3: Public Events

`GET https://api.github.com/users/{username}/events/public?per_page=100`

**Fields used per event:**

| Field | Used For |
|---|---|
| `type` | Filter for relevant events |
| `created_at` | Group by month for activity chart |

**Event types counted:** `PushEvent`, `PullRequestEvent`, `IssuesEvent`, `CreateEvent`, `WatchEvent`, `ForkEvent`

---

## 14. LinkedIn PDF Parsing Reference

LinkedIn export PDFs follow a consistent structure. Use text pattern matching to extract fields.

### Section Detection

Detect section headers by looking for these exact phrases in the extracted text:

```
"Contact" or "Contact Info"         → contact section
"Summary" or "About"                → about section  
"Experience"                        → work experience section
"Education"                         → education section
"Skills"                            → skills section
"Licenses & Certifications"         → certifications section
"Languages"                         → languages section
"Volunteer Experience"              → volunteer section
"Accomplishments" or "Projects"     → projects section
```

### Name Extraction

The name is typically the first large text block at the very beginning of the PDF text.

```javascript
function extractName(text) {
  // Name is usually the first 1-3 words before any section header
  const lines = text.split('\n').filter(l => l.trim().length > 0);
  // First non-empty line is typically the name
  return lines[0]?.trim() || null;
}
```

### Connections Extraction

```javascript
function extractConnections(text) {
  // Patterns: "500+ connections", "423 connections", "1 connection"
  const match = text.match(/(\d+\+?)\s+connections?/i);
  return match ? match[1] : null;
}
```

### Experience Section Parsing

Each position in the experience section typically appears as:
```
Job Title
Company Name
Start Date – End Date
Duration (sometimes shown)
Location (sometimes shown)
```

```javascript
function extractPositions(text) {
  // Find the experience section
  const expStart = text.indexOf('Experience');
  const expEnd = text.indexOf('Education'); // next major section
  const expText = text.substring(expStart, expEnd);
  
  // Parse individual positions
  // Date patterns: "Jan 2020 – Mar 2022", "2019 – Present", "March 2021 – Present"
  const datePattern = /([A-Za-z]+\s+\d{4}|\d{4})\s*[–\-]\s*([A-Za-z]+\s+\d{4}|\d{4}|Present)/gi;
  // ... extract around date matches to get title and company
}
```

### Date Calculation for Experience

```javascript
function calculateDuration(startStr, endStr) {
  const parseDate = (str) => {
    if (!str || str.toLowerCase() === 'present') return new Date();
    // Try "Jan 2020" format
    const monthYear = str.match(/([A-Za-z]+)\s+(\d{4})/);
    if (monthYear) {
      return new Date(`${monthYear[1]} 1, ${monthYear[2]}`);
    }
    // Try year only "2020"
    const yearOnly = str.match(/(\d{4})/);
    if (yearOnly) return new Date(`January 1, ${yearOnly[1]}`);
    return null;
  };
  
  const start = parseDate(startStr);
  const end = parseDate(endStr);
  if (!start || !end) return null;
  
  const months = (end.getFullYear() - start.getFullYear()) * 12 
                 + (end.getMonth() - start.getMonth());
  return { months, years: (months / 12).toFixed(1) };
}
```

### Skills Extraction

```javascript
function extractSkills(text) {
  const skillsStart = text.indexOf('Skills');
  const nextSection = /* find next section header after skills */;
  const skillsText = text.substring(skillsStart, nextSection);
  
  // Skills are typically listed one per line or separated by bullets
  const skills = skillsText
    .split(/[\n•·,]/)
    .map(s => s.trim())
    .filter(s => s.length > 1 && s.length < 60) // filter noise
    .filter(s => !['Skills', 'Top Skills', 'Industry Knowledge'].includes(s));
  
  return [...new Set(skills)]; // deduplicate
}
```

---

## 15. Profile Strength Score Algorithm

The score is a number from 0 to 100. It measures how complete and active the professional profile is.

### Score Breakdown (100 points total)

**GitHub Component (50 points max):**

| Sub-metric | Max Points | Calculation |
|---|---|---|
| Has public repos | 10 | repos > 0: 10 pts, else 0 |
| Repo count | 10 | min(repos / 10 * 10, 10) — 10 repos = full score |
| Total stars | 10 | min(total_stars / 50 * 10, 10) — 50 stars = full score |
| Recent activity | 10 | Events in last 30 days: min(events * 2, 10) |
| Profile completeness | 10 | bio filled: 3 pts, location: 3 pts, website/blog: 4 pts |

**LinkedIn Component (50 points max):**

| Sub-metric | Max Points | Calculation |
|---|---|---|
| Experience present | 10 | at least 1 position: 10 pts, else 0 |
| Years of experience | 10 | min(years / 5 * 10, 10) — 5 years = full score |
| Skills listed | 10 | min(skills_count / 10 * 10, 10) — 10 skills = full score |
| Certifications | 10 | min(cert_count / 2 * 10, 10) — 2 certs = full score |
| Education present | 10 | degree found: 10 pts, else 0 |

**If only one source loaded:**
- GitHub only: score = (GitHub points / 50) * 100, shown as "GitHub Score"
- LinkedIn only: score = (LinkedIn points / 50) * 100, shown as "LinkedIn Score"
- Both loaded: score = GitHub points + LinkedIn points (out of 100)

**Score Labels:**

| Range | Label | Color |
|---|---|---|
| 0–40 | "Needs Improvement" | Red `#EF4444` |
| 41–70 | "Good Profile" | Orange `#F59E0B` |
| 71–85 | "Strong Profile" | Blue `#0A66C2` |
| 86–100 | "Exceptional Profile" | Green `#21C55D` |

---

## 16. Design System

### Colors

```css
:root {
  --primary: #0A66C2;       /* LinkedIn blue — primary brand color */
  --primary-dark: #004182;  /* Darker blue for hover states */
  --primary-light: #EBF3FB; /* Light blue for card backgrounds */
  --github: #24292F;        /* GitHub dark — used for GitHub section */
  --github-light: #F6F8FA;  /* GitHub light grey background */
  --success: #21C55D;       /* Green for positive indicators */
  --warning: #F59E0B;       /* Orange for medium scores */
  --danger: #EF4444;        /* Red for errors and low scores */
  --text-primary: #1D2226;  /* Main body text */
  --text-secondary: #666666;/* Subdued labels */
  --text-muted: #999999;    /* Placeholder, fallback text */
  --border: #E0E0E0;        /* Card borders */
  --shadow: 0 2px 8px rgba(0,0,0,0.08); /* Card shadows */
  --bg: #F3F6F8;            /* Page background */
  --white: #FFFFFF;         /* Card backgrounds */
}
```

### Typography

```css
font-family: 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;

--font-xl: 2.5rem;    /* App title */
--font-lg: 1.5rem;    /* Section headers */
--font-md: 1.125rem;  /* KPI values */
--font-sm: 0.875rem;  /* Labels, captions */
--font-xs: 0.75rem;   /* Footer, micro text */
```

### Spacing

```css
--space-xs: 4px;
--space-sm: 8px;
--space-md: 16px;
--space-lg: 24px;
--space-xl: 32px;
--space-2xl: 48px;
```

### Cards

```css
.card {
  background: var(--white);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: var(--space-lg);
  box-shadow: var(--shadow);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0,0,0,0.12);
}
```

### Buttons

```css
.btn-primary {
  background: var(--primary);
  color: white;
  border: none;
  border-radius: 6px;
  padding: 10px 20px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}
.btn-primary:hover { background: var(--primary-dark); }
.btn-primary:disabled { background: #94A3B8; cursor: not-allowed; }
```

### Loading Spinner

```css
.spinner {
  width: 24px; height: 24px;
  border: 3px solid var(--primary-light);
  border-top: 3px solid var(--primary);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
```

### Card Entrance Animation

```css
.card-enter {
  opacity: 0;
  transform: translateY(12px);
}
.card-enter-active {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 0.4s ease, transform 0.4s ease;
}
```

---

## 17. Responsive Design Rules

### Breakpoints

```css
/* Mobile: < 640px */
/* Tablet: 640px – 1024px */
/* Desktop: > 1024px */
```

### Layout Changes by Breakpoint

| Element | Desktop | Tablet | Mobile |
|---|---|---|---|
| Input cards | Side by side (50/50) | Side by side (50/50) | Stacked (100%) |
| KPI cards | 6 in a row | 3 per row | 2 per row |
| Charts Row 1 & 2 | 2 per row | 2 per row | 1 per row (full width) |
| Repositories | 3-column grid | 2-column grid | 1-column |
| Career timeline | Horizontal | Horizontal | Vertical |

### Mobile-Specific Rules

- Touch target minimum size: 44x44px for all buttons
- Font size minimum: 14px (no text smaller than 14px on mobile)
- Horizontal scroll: never allowed
- Charts: reduce height to 250px on mobile (300px+ on desktop)

---

## 18. Success Metrics

| Metric | Target |
|---|---|
| GitHub profile loads (API + render) | Under 3 seconds on standard broadband |
| LinkedIn PDF parses and renders | Under 5 seconds for a typical 2MB export |
| All KPI cards populate with data | 100% — no blank cards |
| Graceful fallback for missing fields | 100% — no "undefined" or "null" visible |
| Works across target browsers | Chrome, Firefox, Edge, Safari — all pass |
| Mobile usability | Full functionality on 375px wide screen |
| Zero JavaScript errors in console | For all valid inputs |
| File loads offline after first open | Not required in v1 |

---

## 19. Out of Scope (Version 1)

The following features are intentionally excluded from v1. They may be considered for future versions.

| Feature | Reason Excluded |
|---|---|
| LinkedIn URL input | Legal — not permitted per LinkedIn ToS |
| User authentication / login | Unnecessary complexity for v1 |
| Saving or sharing dashboard | No backend in v1 |
| PDF export of the dashboard | Future enhancement |
| Comparing two profiles side by side | Future enhancement |
| GitHub private repo data | Requires OAuth — out of scope |
| Twitter / X profile data | Out of scope |
| Portfolio website analysis | Out of scope |
| AI-generated profile improvement tips | Future enhancement |
| Dark mode | Future enhancement |

---

## 20. Glossary

| Term | Definition |
|---|---|
| **GitHub REST API** | GitHub's official free API for accessing public profile and repo data |
| **LinkedIn Data Export** | A PDF file LinkedIn allows any user to download containing their own profile data |
| **PDF.js** | Mozilla's open-source JavaScript library for parsing PDF files in the browser |
| **Chart.js** | Open-source JavaScript charting library |
| **KPI** | Key Performance Indicator — a measurable value shown as a metric card |
| **Profile Strength Score** | A calculated 0–100 score representing overall profile completeness and activity |
| **Client-side** | Processing that happens in the user's browser, not on a server |
| **Unauthenticated API** | API calls made without a login token — limited to public data and 60 requests/hour |
| **State** | The JavaScript object holding all currently loaded data in memory |
| **Edge case** | An unusual input or condition that the app must handle without breaking |

---

*End of ProfilePulse PRD v1.0*  
*This document is the single source of truth for building the app. Any developer or AI coding tool should be able to build the complete app from this document alone.*
