---
name: company-brief
description: Collect and format the company information that the Caeros investor matcher needs (name, description, website, keywords, country, cap table). Use before caeros_investor_match or caeros_family_office_search, or when the user gives a company to research, a deck, a website, or a thin one-line description.
---

# Company Brief

The matcher embeds your `description`, `keywords`, and `country` into a single
vector and retrieves against it. Everything downstream, the firms, the scores,
the contacts, is decided by that one piece of text. A vague brief does not
produce a vague list, it produces a confidently wrong one. This skill is how you
build the input.

## The field contract

| Field | Required | What good looks like |
|---|---|---|
| `company_name` | yes | Legal or trading name of the company raising. Not the investor, not the parent. |
| `description` | yes | 3 to 6 sentences on what the company does. See below. |
| `website` | strongly preferred | Bare domain, lowercase, no scheme or `www` (`acme.com`). Also links the company to the Vault graph if it is already known there. |
| `keywords` | preferred | Comma-separated sector, technology, and business-model terms. |
| `country` | preferred | Primary market or headquarters country. |
| `cap_table_domains` | when known | Domains of investors already on the cap table. |

## Writing the description

Write what the company **does**, for whom, and how it makes money. Plain, dense,
specific. You are describing the company to a system that will look for other
companies like it.

Include, when known:

- The product and what it actually does.
- The customer: who buys it, in what industry, at what size.
- The business model: subscription, usage, marketplace take rate, hardware.
- The stage and traction, in facts (revenue band, customer count, headcount).
- The technology, if it is a differentiator.
- The geography served.

Leave out:

- Fundraising language. "Raising a $5M Series A" describes the round, not the
  company, and it pollutes the embedding with the vocabulary of every other
  company that is also raising.
- Superlatives and pitch adjectives: leading, innovative, revolutionary,
  best-in-class, next-generation. They carry no retrieval signal.
- Buzzword stacks with no object. "AI-powered platform" matches everything, and
  matching everything means matching nothing.
- Investor preferences. Where the company wants to raise from belongs in the
  filter arguments, not in the description.

Bad: "Acme is an innovative AI-powered fintech platform raising a Series A to
disrupt payments."

Good: "Acme provides accounts-payable automation for mid-market construction
firms in the UK and Ireland. Its software ingests supplier invoices, matches
them to purchase orders and delivery notes, and pushes approved payments into
Sage and Xero. Customers are finance teams at companies with 50 to 500 staff,
paying an annual subscription tiered by invoice volume. Roughly 180 customers
and 2.4M GBP ARR, 31 staff, headquartered in Manchester."

## When information is missing

In order:

1. **Ask the user first.** They usually have the deck, the site, or the numbers.
   One short question beats ten minutes of research.
2. **If they ask you to research it, or hand you a website and nothing else,**
   use `web_search` and `web_fetch` against the company's OWN primary sources:
   their website, product and pricing pages, and their public profiles. Prefer
   what the company says about itself over commentary about it.
3. **State what you filled in.** Show the assembled brief and mark which fields
   came from research rather than from the user, before you spend the match.
4. **Never invent.** If you cannot establish the business model or the traction,
   leave it out of the description. An omission weakens the embedding a little.
   A fabrication corrupts it, and it will be indistinguishable from fact in the
   result set.

**Hard boundary on research:** the web is allowed ONLY to describe the company
being matched. Investors, firms, and contacts come exclusively from the Vault
via `caeros_investor_match` or Vault SQL. Never search the web for investors,
never assemble a target list from model knowledge, and never guess or
pattern-match a contact email or LinkedIn URL. If the Vault has nothing, the
answer is that the Vault has nothing.

## Existing investors and the cap table

Collect the domains of investors already on the cap table and pass them as
`cap_table_domains`. This is worth doing properly:

- The matcher pins them as existing investors instead of presenting them as
  fresh targets, so `is_existing_investor` is accurate.
- They feed the warm-intro paths, which is often the most useful column in the
  output.

Use bare investor domains (`sequoiacap.com`), not firm names, not portfolio
company domains. If the user gives firm names only, ask for the domains, or
confirm the ones you resolve before using them.

If the user wants specific firms kept OUT of the results entirely (an existing
investor they will not go back to, a competitor's backer), that is
`exclude_investors`, not `cap_table_domains`. The two do opposite things.

## If the user gives you a file

A CSV, TSV, or XLSX of companies or investors goes through
`analyst_read_table_file` first. Do not read a spreadsheet with a text file
tool. Extract the names, domains, or people, then build the brief from them.

## Pre-flight checklist

Before calling the matcher, confirm all of these:

- [ ] `company_name` is the company raising.
- [ ] `description` is 3 or more sentences and says what the company does.
- [ ] No fundraising language, no pitch adjectives in the description.
- [ ] `website` is a bare lowercase domain, if known.
- [ ] `keywords` are comma-separated and specific.
- [ ] `country` is set, if known.
- [ ] `cap_table_domains` are investor domains, if any exist.
- [ ] Any field you researched yourself has been shown to the user.
- [ ] Filters (`top_k`, `min_score`, exclusions, contact locations) reflect what
      the user actually asked for, and nothing they did not.

Then load `investor-match` and make the single call.

## Reusing a brief

The same brief drives `caeros_family_office_search`, which takes the description
as its `query`. Build it once per company. If the user runs both, do not rewrite
the description between them, or the two result sets stop being comparable.
