# B2B Lead Gen Suite MCP

[![Apify Actor](https://img.shields.io/badge/Apify-Actor-blue?logo=apify)](https://apify.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node](https://img.shields.io/badge/Node-22+-green.svg)](https://nodejs.org/)

> Comprehensive B2B lead generation suite for scraping company info, contacts, and generating qualified leads from LinkedIn, company websites, and business directories.

## 🎯 Overview

B2B Lead Gen Suite MCP is an AI-agent-optimized actor that provides enterprise-grade lead generation capabilities with Pay-Per-Event (PPE) pricing. It implements `handleRunFunction` with four core operations: `scrapeCompanies`, `scrapeContacts`, `generateLeads`, and `exportCSV`.
> **How it compares:** unlike single-source lead scrapers, b2b-lead-gen-suite-mcp combines company discovery, decision-maker contact enrichment (verified emails and phones), quality scoring, and CRM-ready CSV/JSON/XLSX export in one MCP server — see the full listing on [ScraperFleet](https://scraperfleet.com/actors/b2b-lead-gen-suite-mcp/).


### Key Features

- **Multi-Source Scraping**: LinkedIn, company websites, business directories
- **PPE (Pay-Per-Event) Pricing**: Only pay for successful extractions
- **handleRunFunction API**: Simple operation dispatcher for AI agents
- **ERROR_RETRY Handling**: Exponential backoff with 3 retry attempts
- **Agent Handshake**: <5 second smoke test with FREE compute
- **CSV/JSON Export**: Flexible export formats for CRM integration

## ⚡ Quick Start

### 1. Installation

```bash
# Navigate to project directory
cd ~/Projects/apify-actors/b2b-lead-gen-suite-mcp

# Install dependencies
npm install
```

### 2. Local Development

```bash
# Run locally with Apify SDK
apify run

# Or set input via environment
APIFY_INPUT_DEFAULT={"operation":"generateLeads","queries":["Software Engineer at Google"],"maxResults":5} npx apify run
```

### 3. Push to Apify

```bash
apify push
```

## 📋 Input Schema

### Core Parameters

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `operation` | string | Operation to perform: `scrapeCompanies`, `scrapeContacts`, `generateLeads`, `exportCSV` | `generateLeads` |
| `sources` | array | Sources to scrape: `linkedin`, `company_websites`, `business_directories`, `all` | `["linkedin"]` |
| `queries` | array | Search terms for lead generation | `["Tech companies in San Francisco"]` |
| `maxResults` | integer | Maximum results per operation (1-1000) | `10` |
| `companyDomains` | array | List of company domains (e.g., `["google.com", "microsoft.com"]`) | `[]` |
| `linkedInUrls` | array | LinkedIn profile URLs for contact scraping | `[]` |
| `exportFormat` | string | Export format: `csv`, `json`, `xlsx` | `csv` |
| `includeHeaders` | boolean | Include column headers in CSV export | `true` |
| `proxyType` | string | Proxy type: `DATACENTER`, `RESIDENTIAL` | `DATACENTER` |
| `debugMode` | boolean | Enable minimal extraction for testing (300s max) | `false` |
| `checkOnly` | boolean | 5-second handshake test (FREE compute) | `false` |

### Example Input

```json
{
  "operation": "generateLeads",
  "sources": ["linkedin", "company_websites"],
  "queries": ["Software Engineer at Google", "Product Manager at Microsoft"],
  "maxResults": 20,
  "companyDomains": ["google.com", "microsoft.com"],
  "exportFormat": "csv",
  "includeHeaders": true,
  "proxyType": "RESIDENTIAL",
  "debugMode": false,
  "checkOnly": false
}
```

## 🔧 handleRunFunction API

The `handleRunFunction` provides a simple dispatcher for all operations:

### Operations

#### 1. scrapeCompanies

Extract company information from multiple sources.

```javascript
{
  "operation": "scrapeCompanies",
  "sources": ["linkedin", "company_websites", "business_directories"],
  "companyDomains": ["google.com", "apple.com"],
  "maxResults": 50
}
```

**Returns:** Array of company objects with id, name, domain, industry, size, location, linkedinUrl, source, scrapedAt.

#### 2. scrapeContacts

Extract contact information from LinkedIn profiles.

```javascript
{
  "operation": "scrapeContacts",
  "linkedInUrls": [
    "https://www.linkedin.com/in/johndoe",
    "https://www.linkedin.com/in/janesmith"
  ],
  "maxResults": 25
}
```

**Returns:** Array of contact objects with id, fullName, jobTitle, company, email, phone, linkedinUrl, location, source, scrapedAt.

#### 3. generateLeads

Generate qualified B2B leads from multiple sources.

```javascript
{
  "operation": "generateLeads",
  "sources": ["linkedin", "company_websites"],
  "queries": ["Software Engineer at Google", "Data Scientist at Meta"],
  "maxResults": 100,
  "companyDomains": ["google.com", "meta.com"]
}
```

**Returns:** Array of lead objects with id, fullName, jobTitle, company, email, linkedinUrl, source, quality, generatedAt.

#### 4. exportCSV

Export results to CSV or JSON format.

```javascript
{
  "operation": "exportCSV",
  "results": [], // Uses generateLeads if empty
  "exportFormat": "csv",
  "includeHeaders": true,
  "queries": ["Tech companies"],
  "maxResults": 50
}
```

**Returns:** Export file with filename, format, recordCount, exportedAt.

## 💰 PPE Pricing

This actor uses Pay-Per-Event (PPE) pricing model. You only pay for successful extractions:

| Event | FREE Tier | BRONZE | SILVER | GOLD |
|-------|-----------|--------|--------|------|
| Actor Start | $0.00005 | $0.00005 | $0.00005 | $0.00005 |
| Company Scraped | $0.05 | $0.035 | $0.02 | $0.01 |
| Contact Scraped | $0.08 | $0.055 | $0.03 | $0.015 |
| Lead Generated | $0.10 | $0.075 | $0.04 | $0.02 |
| CSV Exported | $0.02 | $0.02 | $0.02 | $0.02 |

### Zero-Result Guard

PPE charging only occurs when actual data is extracted. If no results are found, no charges are applied.

## 🔄 ERROR_RETRY Handling

The actor implements robust error handling with exponential backoff:

```javascript
async function withRetry(fn, maxRetries = 3, baseDelayMs = 1000) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            if (attempt < maxRetries) {
                const delay = baseDelayMs * Math.pow(2, attempt - 1);
                await sleep(delay);
            }
        }
    }
    throw lastError;
}
```

### Retry Conditions

- Network timeouts
- Rate limiting (429 responses)
- Temporary server errors (5xx)
- CAPTCHA detection with fallback

## 🤖 AI Agent Integration

### 5-Second Handshake

For AI agent smoke tests, use `checkOnly: true`:

```javascript
{
  "checkOnly": true,
  "maxResults": 10
}
```

Returns readiness status in <5 seconds with FREE compute.

### handleRunFunction Example

```javascript
const result = await handleRunFunction({
    operation: 'generateLeads',
    sources: ['linkedin'],
    queries: ['Software Engineer at Google'],
    maxResults: 50,
    proxyType: 'RESIDENTIAL'
});
```

## 📊 Output Schema

All operations return:

```json
{
  "success": true,
  "operation": "generateLeads",
  "results": [...],
  "resultsCount": 50,
  "exportedFile": "",
  "charges": [],
  "scrapedAt": "2026-05-19T21:00:00.000Z",
  "durationMs": 2500
}
```

### Result Objects

**Company:**
```json
{
  "id": "lead_123456_abc123",
  "name": "Google",
  "domain": "google.com",
  "linkedinUrl": "https://linkedin.com/company/google",
  "industry": "Technology",
  "size": "10000+",
  "location": "Mountain View, CA",
  "source": "linkedin",
  "scrapedAt": "2026-05-19T21:00:00.000Z"
}
```

**Contact:**
```json
{
  "id": "lead_123456_def456",
  "fullName": "John Doe",
  "jobTitle": "Software Engineer",
  "company": "Google",
  "email": "john.doe@example.com",
  "phone": "+1-555-0100",
  "linkedinUrl": "https://linkedin.com/in/johndoe",
  "location": "San Francisco, CA",
  "source": "linkedin",
  "scrapedAt": "2026-05-19T21:00:00.000Z"
}
```

**Lead:**
```json
{
  "id": "lead_123456_ghi789",
  "fullName": "Jane Smith",
  "jobTitle": "Product Manager",
  "company": "Microsoft",
  "email": "jane.smith@example.com",
  "linkedinUrl": "https://linkedin.com/in/janesmith",
  "source": "google_serp_linkedin",
  "quality": "high",
  "generatedAt": "2026-05-19T21:00:00.000Z"
}
```

## 🚀 Usage Examples

### Basic Lead Generation

```bash
apify run -c '{
  "operation": "generateLeads",
  "queries": ["Software Engineer at Google"],
  "maxResults": 10
}'
```

### Scrape Multiple Companies

```javascript
{
  "operation": "scrapeCompanies",
  "sources": ["linkedin", "company_websites"],
  "companyDomains": ["google.com", "microsoft.com", "apple.com"],
  "maxResults": 50,
  "proxyType": "RESIDENTIAL"
}
```

### Contact Discovery

```javascript
{
  "operation": "scrapeContacts",
  "linkedInUrls": [
    "https://www.linkedin.com/in/johndoe",
    "https://www.linkedin.com/in/janesmith",
    "https://www.linkedin.com/in/bobwilson"
  ],
  "maxResults": 25
}
```

### Export to CSV

```javascript
{
  "operation": "exportCSV",
  "results": [], // Uses generateLeads if empty
  "exportFormat": "csv",
  "includeHeaders": true,
  "queries": ["Tech companies in San Francisco"],
  "maxResults": 100
}
```

## 🛠️ Configuration

### Proxy Configuration

| Type | Use Case | Cost |
|------|----------|------|
| `DATACENTER` | General web scraping, faster | Lower |
| `RESIDENTIAL` | High-security targets (LinkedIn) | Higher |

### Debug Mode

Enable `debugMode: true` for:
- Minimal extraction (max 2 results)
- Faster proxy (DATACENTER)
- 300-second timeout compliance
- Health check verification

## 📝 Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `EXTRACTION_ERROR` | Profile blocked/private | Use RESIDENTIAL proxy |
| `RATE_LIMIT_ERROR` | Too many requests | Reduce concurrency, wait |
| `NETWORK_ERROR` | Connectivity issue | Check connection, retry |
| `CAPTCHA_BLOCKED` | Google/LinkedIn CAPTCHA | Use RESIDENTIAL proxies |
| `AUTH_WALL` | LinkedIn login required | Use Google SERP fallback |

## 🔒 Legal & Compliance

**Data Source**: LinkedIn, company websites, business directories

**Terms of Service**: This actor is for legitimate data collection only. Users must comply with target platform ToS and applicable laws.

**Privacy**: All extracted data is processed per applicable privacy laws. Do not use for unauthorized collection.

**Attribution**: Comply with data source attribution requirements.


## Comparison

If you're comparing B2B lead generation MCP actors on the Apify Store, compare what a run charges when it finds nothing. B2B Lead Gen Suite MCP runs Pay-Per-Event with a zero-result guard: a run that extracts nothing charges nothing beyond the $0.00005 Actor Start fee. It bills only on successful extractions ($0.05 per company, $0.08 per contact, $0.10 per qualified lead on the free tier, dropping to $0.01/$0.015/$0.02 on GOLD), and ships ERROR_RETRY handling, a 5-second AI-agent handshake, and handleRunFunction operations (scrapeCompanies, scrapeContacts, generateLeads, exportCSV) in one MCP server.

Pair-it-with: [Company Intelligence MCP](https://apify.com/red.cars/company-intelligence-mcp) does the due diligence on the companies you shortlist; [producthunt-scraper](https://apify.com/red.cars/producthunt-scraper) sits at the top of the same funnel, surfacing the companies worth running through it.

## ❓ FAQ

**How does billing work?**
Pay-Per-Event: you only pay for successful extractions — $0.05 per company, $0.08 per contact, $0.10 per qualified lead, $0.02 per CSV export, plus a $0.00005 Actor Start fee. The zero-result guard means a run that extracts nothing charges nothing beyond the start fee.

**Is there a free way to test the actor?**
Yes — send `{ "checkOnly": true }` and the actor returns a readiness status in under 5 seconds with free compute. Use it for smoke tests before wiring the actor into an agent.

**What operations can I run?**
Four: `scrapeCompanies` (company profiles by domain), `scrapeContacts` (contacts from LinkedIn profile URLs), `generateLeads` (qualified leads from search queries), and `exportCSV` (package results for download).

**Which sources does it pull from?**
`linkedin`, `company_websites`, and `business_directories` — pass one or more, or `"all"`. Each result carries its `source` field so you can see where every row came from.

**What fields do I get per company and per contact?**
Companies: name, domain, industry, size, location, and LinkedIn URL. Contacts: full name, job title, company, email, phone, LinkedIn URL, and location.

**How do I export the results?**
Set `operation: "exportCSV"` with `exportFormat` of `csv`, `json`, or `xlsx`, and `includeHeaders` for column headers. Or export from the run's dataset directly — every result is pushed there in real time.

**Which proxy should I choose?**
`DATACENTER` is the default and works for most public pages. Switch to `RESIDENTIAL` for sources that throttle datacenter traffic; expect slower but steadier extraction.

**What happens when a source fails mid-run?**
Every extraction is wrapped in retry logic: up to 3 attempts with exponential backoff, covering network timeouts, 429 rate limits, 5xx server errors, and CAPTCHA detection with fallback.

**How do I cap spend on a big run?**
Bound the run with `maxResults` (1-1000) — you are charged per successfully extracted item, so the cap sets your ceiling. `debugMode` runs a minimal extraction for cheap end-to-end tests.

**Can I point it at specific companies only?**
Yes — pass `companyDomains` (e.g. `["google.com", "microsoft.com"]`) with `scrapeCompanies` or `generateLeads`, and `linkedInUrls` with `scrapeContacts` for per-profile extraction.

**Is this compliant to use?**
It is for legitimate data collection only: comply with target platform terms of service, applicable privacy laws, and data source attribution requirements.

## 📄 License

MIT License - see LICENSE file for details.

---

**Pair it with [NPI Registry Pro](https://apify.com/red.cars/npi-registry-pro)** — turn any target company list into named healthcare-provider contacts from CMS’s free 8.7M-provider registry.

*[B2B Lead Gen Suite MCP on the Apify Store](https://apify.com/red.cars/b2b-lead-gen-suite-mcp) — part of the ScraperFleet actor family.*