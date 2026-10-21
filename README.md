## What does this California business search scraper do?

This Actor searches the **official California Secretary of State** registers at
[bizfileOnline](https://bizfileonline.sos.ca.gov) and returns structured JSON. It covers both
registers at once: **business entities** (status, standing, entity type, formation date and the
**registered agent for service of process**) and **UCC liens** (financing statements, judgment
liens, state and federal tax liens and attachments, each with debtor, secured party, file date
and lapse date).

So one search for a company tells you both *what it is* and *who has a claim on it*.

**No login, no California state API key, nothing to configure.** A one-company lookup costs
about **$0.05**, and a search that returns nothing is not charged for.

Most business-registry tools only let you *look up a company you already know*. This one also
lets you **discover**, because it exposes the registry's own advanced filters:

- Every **LLC formed in California last month** → entity type + formation-date range
- Every **federal tax lien filed in a date range** → lien type + date range
- Every **suspended entity** whose name contains a keyword → status filter

The business side has a second mode of its own, exactly as the registry's own Business tab does:
**publicly traded disclosure** filings, searchable by director or officer, board composition,
compensation range and auditor.

### What data can you extract from bizfileOnline?

Business entity name, entity number, status, standing, entity type, jurisdiction of formation,
initial filing date and registered agent. With **Full profile**, each record also carries its
principal and mailing address, the agent's own address, all four standings and the Statement of
Information due date. For UCC filings you get every debtor and secured party with full street
addresses — which frequently names the **individual** behind a trading name. **Filing history**
adds every filing plus the registry's field-level change history, and **Download documents**
saves each filing as a PDF.

### Running it on Apify

Because it runs on the Apify platform, you also get **scheduling** (run a watchlist every
morning), **API access** to pull results straight into your own system, **integrations** with
Make, Zapier, Google Sheets, Slack and webhooks, and **automatic proxy rotation** — the registry
refuses datacenter addresses, so runs go out through a US residential proxy by default.

## How to scrape California business records

1. Click **Try for free** to open the Actor, or add it to your Apify account.
2. Under **Which registers to search**, keep both registers selected, or pick just one.
3. Type one or more **Search terms** — a company name, a California entity number such as
   `3814816`, or a debtor name. Terms are limited to 49 characters, the registry's own limit.
4. *(Optional)* Open **Business entity filters** or **UCC lien filters** to narrow by entity
   type, status, lien type or a date range.
5. *(Optional)* Under **Limits & connection**, raise **Detail level** from *Search results only*
   to *Full profile*, *Filing history* or *Download documents*, and set **Maximum results per
   search**.
6. Click **Save & start**. Records stream into the dataset as they are extracted, so a large
   search starts filling in immediately.
7. Export from the **Output** tab as JSON, CSV, Excel or XML, or fetch it from the API.

## How much does it cost to scrape California business data?

You pay only for what a run actually does. Searching costs the same whether a term returns one
record or five hundred, so the search is priced once and the records are priced per record.

| Event | What triggers it | Price |
|---|---|---|
| Actor start | Once per run, per GB of memory | $0.005 |
| Search performed | One search term against one register that returned results | $0.025 |
| Record | One business entity or one UCC lien filing | $0.003 |
| Full profile or filing history | A record opened for addresses, agent, standings and history | $0.004 |
| Filing document | One filing PDF saved to the key-value store | $0.006 |

Volume discounts apply automatically on paid Apify plans.

**What a typical run costs**

| Run | Price |
|---|---|
| 50 business records for one search term | **$0.185** |
| One company across both registers, 50 records | ~$0.21 |
| 25 full profiles with addresses and agent | ~$0.21 |
| Two companies with all their filing PDFs | ~$0.13 |

The $0.185 figure is a real run: 50 records in 15 seconds, shown in the output screenshot below.

A search that returns nothing is not charged for. Set **Maximum cost per run** in *Run options*
to cap your spend — the Actor sizes the job to your budget before it starts, and stops rather
than exceed it.

## Input and output examples

### Input

Pick your registers, type a search term, and run. Everything else is optional.

[![California business and UCC lien search input](https://raw.githubusercontent.com/FlowExtractAPI/california-bizfile-scraper/main/docs/images/california-business-ucc-search-input.png)](https://apify.com/dz_omar/california-bizfile-scraper)

The simplest possible input searches both registers:

```json
{ "searchQueries": ["DEL TACO"] }
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

Full profiles with addresses, ready for outreach:

```json
{ "searchQueries": ["DEL TACO"], "detailLevel": "full", "maxResultsPerQuery": 25 }
```

### Output

[![California business entity results in the Apify dataset](https://raw.githubusercontent.com/FlowExtractAPI/california-bizfile-scraper/main/docs/images/california-business-entity-results.png)](https://apify.com/dz_omar/california-bizfile-scraper)

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

### Full profile adds every address

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

and every UCC party gets a street address — often naming an **individual** behind a trading name
that the search rows never show:

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

### Downloaded filing documents

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

## Other Actors you might need

| Actor | What it does |
|---|---|
| [Delaware Business Search](https://apify.com/dz_omar/delaware-corporations-scraper) | The Delaware Division of Corporations registry — file number, formation date, entity type and the registered agent's name, address and phone, many entities at once |
| [LinkedIn Company Scraper](https://apify.com/dz_omar/linkedin-company-scraper) | Public company pages — size, industry, headquarters and specialties |
| [LinkedIn Jobs Scraper](https://apify.com/dz_omar/linkedin-jobs-scraper) | Public job listings by keyword, location and date |
| [Ultimate Lead Extractor](https://apify.com/dz_omar/ai-lead-extractor) | Emails, phone numbers and social profiles from any list of websites |
| [Facebook Posts Scraper](https://apify.com/dz_omar/facebook-posts-scraper) | Posts from public pages, profiles and groups |

Many California businesses are Delaware corporations registered here as out-of-state entities —
when a record shows **Formed in: DELAWARE**, look it up in the Delaware registry for its home-state
formation details and registered agent.

A typical pipeline: pull newly formed California LLCs here, enrich the company names with
LinkedIn, then find contact details with the lead extractor.

## Frequently asked questions

### Is it legal to scrape California business records?

California business-entity and UCC records are **public records**. This Actor reads the open
public searches at [bizfileonline.sos.ca.gov](https://bizfileonline.sos.ca.gov) — it does not log
in and does not pass any paywall. Use the data lawfully for KYB/AML, due-diligence and B2B
purposes.

### Do I need a California state API key?

No. The state's own bulk API is key-gated and capped. This Actor reads the open public search, so
there is nothing to obtain or configure.

### What can I use California registry data for?

- **KYB / KYC onboarding** — verify an entity exists, is active and is in good standing.
- **Registered-agent intelligence** — the agent for service of process, for legal or sales work.
- **Lending and credit risk** — check existing liens, and see who already has a claim on the collateral.
- **Distress and collections** — tax liens, judgment liens and attachments identify businesses in
  trouble, and full profiles name the individuals behind them.
- **Competitive intelligence for lenders** — a fresh financing statement shows a company just
  borrowed, and names its current lender.
- **Lead generation** — pull newly formed companies by type and formation month, with addresses.

### What do the UCC lien types mean?

| Lien type | What it tells you |
|---|---|
| **Financing Statement** | A lender claims a security interest in the business's assets. Normal borrowing — and it names the lender. Lapses 5 years after filing unless renewed. |
| **Judgment Lien** | Someone sued the business and won, then recorded the judgment against its property. |
| **State Tax Lien** | California claims unpaid state taxes. |
| **Federal Tax Lien** | The IRS claims unpaid federal taxes. |
| **Attachment** | A court froze assets while a lawsuit is still pending. |

Only the first is routine. The other four are distress signals.

### Why does this Actor need a residential proxy?

The registry refuses datacenter addresses — verified against both plain datacenter egress and
Apify's datacenter proxy. So runs go out through a **US residential proxy** by default, and you
should leave that setting alone.

### Why did my business search return exactly 500 records?

That is the register's own hard per-term ceiling. The run warns you when a search hits it. Narrow
the term, or add an entity type, status or formation-date filter.

### Why did I get a message about more than 1,000 UCC filings?

The UCC register refuses to answer a term matching more than 1,000 filings rather than returning a
partial list — common words like "Tesla" or "California" hit this easily. Your business-entity
results are unaffected. Search a fuller name, or narrow the UCC side with a lien type, status or
date range.

### Why did the UCC side get skipped entirely?

Either you unticked it under *Which registers to search*, or the term is shorter than 3
characters — the UCC register's own minimum. Shorter terms search the business register only.

### Why is my search term limited to 49 characters?

Because the registry's own search box truncates anything past 49 characters. Accepting a longer
term would run a silently different search than the one you typed.

### How do I search publicly traded disclosures?

Set *Search type* to *Publicly traded disclosure search* in the business filters. That switches
the business side to the disclosure register and enables the director/officer, board-composition,
compensation and auditor fields.

### How do I get only active LLCs?

Set `entityType` to *Limited Liability Company - CA* and `entityStatus` to *Active*.

### Where do downloaded documents go?

Into the run's key-value store, under the *Filing documents* collection. Each dataset record lists
its documents with the key to look up.

### What does each detail level cost me in time?

| Level | What you get | Speed |
|---|---|---|
| **Search results only** | The registers' own columns | Fastest — one request per search |
| **+ Full profile** | Addresses, agent record, all four standings, SOI due date | One extra request per record |
| **+ Filing history** | Every filing, its document link, and field-level change history | Two extra requests per record |
| **+ Download documents** | Each filing fetched as a PDF into the key-value store | Slowest — a company can have ten or more filings |

Keep **Maximum results per search** low when downloading documents. The cap applies **per
register**, so `10` returns up to 10 business records *and* up to 10 UCC records per term.

### Can I use this in Make, Zapier or my own code?

Yes. Every run is available through the Apify API, and the Actor works with Apify's Make, Zapier,
Google Sheets, Slack and webhook integrations. You can also schedule it to re-run a watchlist of
companies and only act on what changed.

## Support

Found a bug, or need a field this Actor does not return yet? Open an issue on the Actor's
**Issues** tab and it will be looked at.

## Legal & compliance

This Actor collects publicly available business-registry information only. You are responsible for
using the extracted data in compliance with applicable laws, including data-protection rules in
your jurisdiction, and with the source's terms.

---

*Built and maintained by **FlowExtractAPI**.*
