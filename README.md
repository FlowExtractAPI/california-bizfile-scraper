# California Business & UCC Lien Search — bizfileOnline Scraper

Search the **official California Secretary of State** registers at [bizfileOnline](https://bizfileonline.sos.ca.gov) and get structured JSON back.

**Choose your registers — both are selected by default, so one search term returns both sides:**

- **Business entities** — every corporation, LLC and LP in California, with status, standing, entity type, formation date and the **registered agent for service of process**.
- **UCC liens** — filings recorded against businesses: **financing statements** (who lent against a company's assets), plus **judgment liens**, **state and federal tax liens** and **attachments**, each with debtor, secured party, file date and lapse date.

So one search for a company tells you both *what it is* and *who has a claim on it*. Untick either register to search only the other.

The business side has a second mode of its own, exactly as the registry's own Business tab does: **publicly traded disclosure** filings, searchable by director or officer, board composition, compensation range and auditor.

**Free to run. No login, no California state API key, nothing to configure.**

## Why this Actor

Most business-registry tools only let you *look up a company you already know*. This one also lets you **discover**, because it exposes the registry's own advanced filters:

- Every **LLC formed in California last month** → entity type + formation-date range
- Every **federal tax lien filed in a date range** → lien type + date range
- Every **suspended entity** whose name contains a keyword → status filter

## What data can this California business search scraper extract?

### Business entity record

```json
{
  "register": "business",
  "name": "TESLA MOTORS CLUB LLC",
  "entityNumber": "201313510058",
  "status": "Terminated",
  "standing": "Good Standing",
  "entityType": "Limited Liability Company - CA",
  "formedIn": "CALIFORNIA",
  "initialFilingDate": "05/13/2013",
  "registeredAgent": "CALIFORNIA CORPORATE AGENTS, INC.",
  "jurisdiction": "California, USA",
  "searchQuery": "Tesla Motors"
}
```

### UCC lien record

```json
{
  "register": "ucc",
  "fileNumber": "187651573053",
  "uccType": "Notice of Federal Tax Lien",
  "status": "Active",
  "filingDate": "06/01/2018",
  "lapseDate": "07/01/2028",
  "debtorName": "TESLA ENERGY OPERATIONS INC A CORPORATION",
  "debtorLocation": "SAN MATEO, CA",
  "securedPartyName": "IRS/OHIO",
  "securedPartyLocation": "CINCINNATI, OH"
}
```

### With **Full profile**, every record also carries its addresses

```json
{
  "principalAddress": "25 ISABELLA AVE, ATHERTON, CA 94027",
  "mailingAddress": "25 ISABELLA AVE, ATHERTON, CA 94027",
  "agentType": "Individual",
  "agentName": "Weina Yu",
  "agentAddress": "25 ISABELLA AVE, ATHERTON, CA 94027",
  "standingSos": "Good", "standingFtb": "Good",
  "standingAgent": "Good", "standingVcfcf": "Good",
  "statementOfInfoDueDate": "07/31/2028"
}
```

and every UCC party gets a street address — often naming an **individual** behind a trading name that the search rows never show:

```json
{
  "debtorsDetailed": [
    { "name": "ARAUJO, FAUSTO",   "address": "625 S RIVERSIDE AVE STE P, RIALTO, CA 923767072" },
    { "name": "LA CASA DEL TACO", "address": "625 S RIVERSIDE AVE STE P, RIALTO, CA 923767072" }
  ],
  "securedPartiesDetailed": [
    { "name": "EMPLOYMENT DEVELOPMENT DEPARTMENT", "address": "722 CAPITOL MALL, SACRAMENTO, CA 95814" }
  ]
}
```

### With **Download documents**, the filings themselves come back as PDFs

```json
{
  "documentCount": 3,
  "documents": [
    {
      "type": "Statement of Information",
      "number": "BA20261385813",
      "date": "7/4/2026",
      "key": "document-business-BA20261385813",
      "contentType": "application/pdf",
      "bytes": 129548
    }
  ]
}
```

Each `key` is a record in the run's **key-value store**, under the *Filing documents* collection.

Records are written to the dataset **as they are extracted**, so a large search starts filling in immediately instead of arriving in one batch at the end.

## How to use it

Search a company in both registers — this is the default:

```json
{ "searchQueries": ["DEL TACO"] }
```

Only the lien side:

```json
{ "registers": ["ucc"], "searchQueries": ["DEL TACO"] }
```

One exact entity, by its California entity/file number:

```json
{ "searchQueries": ["3814816"] }
```

Every LLC formed in January 2024 — a new-business lead list:

```json
{
  "searchQueries": ["California"],
  "entityType": "3",
  "filedFrom": "2024-01-01",
  "filedTo": "2024-01-31"
}
```

Only federal tax liens on the UCC side:

```json
{ "searchQueries": ["Tesla"], "uccType": "2174" }
```

Full profiles with addresses, ready for outreach:

```json
{ "searchQueries": ["DEL TACO"], "detailLevel": "full", "maxResultsPerQuery": 25 }
```

## What the UCC lien types mean

| Lien type | What it tells you |
|---|---|
| **Financing Statement** | A lender claims a security interest in the business's assets. Normal borrowing — and it names the lender. Lapses 5 years after filing unless renewed. |
| **Judgment Lien** | Someone sued the business and won, then recorded the judgment against its property. |
| **State Tax Lien** | California claims unpaid state taxes. |
| **Federal Tax Lien** | The IRS claims unpaid federal taxes. |
| **Attachment** | A court froze assets while a lawsuit is still pending. |

Only the first is routine. The other four are distress signals.

## Input options

| Section | Fields |
|---|---|
| **Top** | `registers` — Business entities, UCC liens, or both (default). `searchQueries` — company names, entity numbers, or debtor / secured-party names |
| **Business entity filters** | Mirrors the registry's own Business tab in the same order: `matchType` (Contains / Starts with), `searchType` (Entity Information / Publicly Traded Disclosure), `entityType` (35 types), `entityStatus` (24 statuses), `filedFrom`, `filedTo` — then the disclosure-only fields: corporation bankruptcy and legal proceedings, director / officer first, middle and last name, board composition, compensation range, the five Disclosures checkboxes (Shares, Options, Bankruptcy, Fraud, Loans to Directors) and independent auditor |
| **UCC lien filters** | `uccStatus`, `uccType` (All, Financing Statement, Judgment Lien, State Tax Lien, Federal Tax Lien, Attachment), `uccFiledFrom`, `uccFiledTo`, `uccLapseFrom`, `uccLapseTo` |
| **Limits & connection** | `detailLevel`, `maxResultsPerQuery`, `proxyConfiguration` |

Every dropdown lists its options in the registry's own order and wording, so a filter you know from the site is where you expect it. Search terms are limited to **49 characters** — the registry's own search box truncates anything longer, so a longer term would quietly run a different search than the one you typed.

### Detail level — the main speed lever

| Level | What you get | Cost |
|---|---|---|
| **Search results only** | The registers' own columns | Fastest — one request per search |
| **+ Full profile** | Addresses, agent record, all four standings, SOI due date | One extra request per record |
| **+ Filing history** | Every filing, its document link, and field-level change history | Two extra requests per record |
| **+ Download documents** | Each filing fetched as a PDF into the key-value store | Slowest — a single company can have ten or more filings |

Keep `maxResultsPerQuery` low when downloading documents.

`maxResultsPerQuery` applies **per register**, so `10` returns up to 10 business records *and* up to 10 UCC records for each search term.

## Why does this Actor need a residential proxy?

The registry refuses datacenter addresses — verified against both plain datacenter egress and Apify's datacenter proxy. So runs go out through a **US residential proxy** by default, and you should leave that setting alone. The Actor is free; you pay only your own Apify platform and proxy usage.

## What can you do with California registry data?

- **KYB / KYC onboarding** — verify an entity exists, is active and is in good standing before you onboard it.
- **Registered-agent intelligence** — get the agent for service of process, for legal, compliance or sales work.
- **Lending and credit risk** — check existing liens before lending, and see who already has a claim on the collateral.
- **Distress and collections** — tax liens, judgment liens and attachments identify businesses in trouble, and full profiles name the individuals behind them.
- **Competitive intelligence for lenders** — a fresh financing statement shows a company just borrowed, and names its current lender.
- **Lead generation** — pull newly formed companies by type and formation month, with mailing addresses.

## Frequently asked questions

**Is it legal to scrape California business records?**
California business-entity and UCC records are **public records**. This Actor reads the open public searches at [bizfileonline.sos.ca.gov](https://bizfileonline.sos.ca.gov) — it does not log in and does not pass any paywall. Use the data lawfully for KYB/AML, due-diligence and B2B purposes.

**Do I need a California state API key?**
No. The state's own bulk API is key-gated and capped. This Actor reads the open public search, so there is nothing to obtain or configure.

**Why did my business search return exactly 500 records?**
That is the register's own hard per-term ceiling. The run warns you when a search hits it. Narrow the term, or add an entity type, status or formation-date filter.

**Why did I get a message about more than 1,000 UCC filings?**
The UCC register refuses to answer a term matching more than 1,000 filings rather than returning a partial list — common words like "Tesla" or "California" hit this easily. Your business-entity results are unaffected. Search a fuller name, or narrow the UCC side with a lien type, status or date range.

**Why did the UCC side get skipped entirely?**
Either you unticked it under *Which registers to search*, or the term is shorter than 3 characters — the UCC register's own minimum. Shorter terms search the business register only.

**Why is my search term limited to 49 characters?**
Because the registry's own search box truncates anything past 49 characters. Accepting a longer term would run a silently different search than the one you typed.

**How do I search publicly traded disclosures?**
Set *Search type* to *Publicly traded disclosure search* in the business filters. That switches the business side to the disclosure register and enables the director/officer, board-composition, compensation and auditor fields.

**How do I get only active LLCs?**
Set `entityType` to *Limited Liability Company - CA* and `entityStatus` to *Active*.

**Where do downloaded documents go?**
Into the run's key-value store, under the *Filing documents* collection. Each dataset record lists its documents with the key to look up.

**How much does it cost?**
The Actor is free. You pay only your own Apify platform usage and residential proxy traffic.

---

## Support

Found a bug, or need a field this Actor does not return yet? Open an issue on the Actor's **Issues** tab and it will be looked at.

## Legal & compliance

This Actor collects publicly available business-registry information only. You are responsible for using the extracted data in compliance with applicable laws, including data-protection rules in your jurisdiction, and with the source's terms.

---

*Built and maintained by **FlowExtractAPI**.*
