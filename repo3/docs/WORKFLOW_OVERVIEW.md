# Workflow Overview: AI Employee for YouTube & LinkedIn Analysis

A node-by-node breakdown of `AI_Employee_YouTube_LinkedIn_Analysis.json`.

---

## Trigger & Setup

| Node | Type | Purpose |
|---|---|---|
| **Monthly Review with Date** | Schedule Trigger | Fires on the 30th of every month at 9 AM — the main entry point |
| **Manual Trigger** + **Get all Channel ID linked to account** | Manual Trigger / YouTube node | A one-time helper flow — run once after setup to fetch the Channel ID tied to your authenticated YouTube account |
| **Enter Channel ID** / **Enter Channel ID1** | Set | Holds the channel ID used by the main YouTube data-collection branch (paste in your own after running the helper flow above) |

---

## YouTube Branch

| Node | Type | Purpose |
|---|---|---|
| **Get Channel details** | YouTube node | Retrieves subscriber count, total views, video count, creation date for the configured channel |
| **Get last week videos** | YouTube node | Searches for all videos published in the last 7 days on the channel |
| **Loop for many videos** | Split In Batches | Iterates through each video found, one at a time |
| **Get YT Video details** | YouTube node | Pulls per-video stats: views, likes, comments, favorites |
| **Update YT video Database** | Google Sheets | Appends/updates each video's stats in the tracking sheet |
| **Add in YT channel Database** | Google Sheets | Appends the channel-level snapshot for this run |

---

## LinkedIn Branch

| Node | Type | Purpose |
|---|---|---|
| **Linkedin_account_scraping** | Apify node (`linkedin-profile-scraper`) | Scrapes profile-level stats: followers, connections, location, account info |
| **Linkedin_post_scraper** | Apify node (`linkedin-profile-posts`) | Scrapes the profile's posts from the last month: likes, comments, content type, day/hour posted |
| **Records data for LinkedIn Profile** | Google Sheets | Logs the profile snapshot |
| **Add in Linkedin post scraper database** | Google Sheets | Logs each scraped post |

Both Apify actors are configured to run **without requiring LinkedIn login cookies** — they use third-party scraping services available on the Apify Store. The scraping target (profile URL) is set directly in each node's request body and should be swapped for your own profile.

---

## Merge, Clean & Route

| Node | Type | Purpose |
|---|---|---|
| **Collect all output** | Merge | Combines the YouTube and LinkedIn data streams once both branches finish |
| **Combine it for final** | Aggregate | Consolidates collected items |
| **Prepare the Data** | Code | Custom JS to reshape/normalize the combined payload |
| **Separate it for YT and LinkedIn** | Code | Splits the unified payload back into two distinct objects, one per platform, for the two AI agents |

---

## AI Analysis

| Node | Type | Purpose |
|---|---|---|
| **YT Data Analyzer** | LangChain Agent | Acts as a "YouTube analytics analyst" — computes aggregated and per-video stats, engagement metrics, and generates structured insight text |
| **LinkedIn Data Analyzer** | LangChain Agent | Acts as a "LinkedIn analytics analyst" — computes per-post engagement scores, ranks content performance, and generates structured insight text |
| **OpenAI Chat Model** | LangChain LM | The shared underlying model for both agents |
| **Calculator** / **Calculator1** | LangChain Tool | Gives each agent a calculator tool for accurate metric math (averages, engagement scores, etc.) |
| **Clean the YT data from Agent** / **Clean the LinkedIn data from agent** | Code | Post-processes each agent's raw output into a clean structure for report rendering |

### YT Data Analyzer — role summary

> *"You are a YouTube analytics analyst AI agent. Your job is to analyze structured YouTube channel and video data and generate a comprehensive analysis dataset... extract meaningful metrics, evaluate performance indicators, and generate structured insights about channel performance, engagement, and growth potential."*

### LinkedIn Data Analyzer — role summary

> *"You are a LinkedIn social media analytics analyst AI agent... extract meaningful metrics, evaluate content performance, identify engagement patterns, and generate structured insights about the LinkedIn profile's content strategy and audience interaction."*

---

## Report Generation & Delivery

| Node | Type | Purpose |
|---|---|---|
| **Prepare HTML Code for YT** | Code | Builds the styled HTML for the YouTube report |
| **Prepare HTML Report Code for LinkedIn** | Code | Builds the styled HTML for the LinkedIn report |
| **Send the YT Report Email** | Gmail | Sends the YouTube report, subject line includes the current month/year |
| **Send the LinkedIn Report in Email** | Gmail | Sends the LinkedIn report, subject line includes the current month/year |

---

## Data & Privacy Notes

The original workflow export contained test data captured during development, including:

- A personal Gmail address as the hardcoded report recipient
- A real LinkedIn profile URL used as the scraping target during testing
- A real, private Google Sheet ID used as the tracking database

All of the above have been **replaced with placeholders** in the version published in this repo. If you import this workflow, you'll need to add your own channel ID, LinkedIn profile URL, Google Sheet, and recipient email before it will run end-to-end.
