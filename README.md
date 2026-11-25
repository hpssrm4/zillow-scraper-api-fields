# Zillow Scraper API: Which Fields You Actually Get, What It Costs, and How to Keep From Getting Blocked (Plan Lineup + Field Fill-Rate Table)

ScraperAPI handles Zillow the way it handles any hard site: you send a Zillow URL, it rotates proxies and renders the JavaScript, and HTML comes back for you to parse. A standard page request costs 1 credit, the free plan gives you 1,000 of them with 5 concurrent connections, and the paid tiers price out on the pricing page. That model suits developers who want to own the parser. It's the wrong purchase if you want a ready-made JSON Zillow feed with ten years of Zestimate history, and the alternatives that do that are compared further down.

Below: the plan lineup and the one setting that separates the tiers, what Zillow actually returns by field (with measured fill rates that explain why so many Zillow datasets have holes in them), how many listings a run can produce, and where the legal line sits.

> **Quick pick.** Start on the free 1,000 credits and run a handful of your own Zillow URLs through the Domain Cost Estimator in the dashboard before you pay for anything. If a listing page comes back at 1 credit, you're buying pages close to cost. If it comes back at 11 because the site sits behind bot protection, every plan covers roughly a tenth of what its credit count suggests. 👉 [See the live ScraperAPI plan lineup and start on the free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The plan lineup, and the setting that actually separates the tiers

ScraperAPI sells credits, and the tiers ladder from Free up to Enterprise. What changes beyond the monthly allowance is what happens at the moment you run out: some plans step you up automatically, some let you keep scraping at a fixed per-credit rate, and the top tiers only do the second.

| Plan | Auto-upgrade when credits run out | Pay-as-you-go overflow | Who it fits |
| --- | --- | --- | --- |
| Free | No | No | Pricing out a Zillow job on 1,000 credits and 5 concurrent connections |
| Hobby | Yes | No | One developer, one metro, a nightly script |
| Startup | Yes | No | A small team covering several markets |
| Business | Yes | No | A recurring feed that backs a product |
| Scaling | Yes | Yes | Bursty national pulls |
| Professional | Yes | Yes | High-volume pipelines with a fixed monthly ceiling |
| Advanced | No | Yes | Committed volume with no automatic step-ups |
| Enterprise | No | Yes | Custom volume under contract |

Credit allowances, concurrency limits and prices per tier are listed on the pricing page, and they're worth reading together rather than one at a time. Concurrency is the throughput ceiling: exceed your plan's limit and ScraperAPI returns a 429 asking you to bring concurrency back down. There's no surcharge for going over, the request is simply refused rather than queued, so a runaway job can't quietly turn into a bigger bill.

Two features do most of the work on cost control. `max_cost` caps what a single scrape is allowed to spend, and the Domain Cost Estimator gives you the real cost of any URL before you commit to it. That matters because the credit price isn't uniform.

| Request type | Credits |
| --- | --- |
| Standard page | 1 |
| Amazon | 5 |
| Google or Bing, including subdomains | 25 |
| LinkedIn | 30 |
| Site behind Cloudflare, DataDome or PerimeterX | +10 per request when bypassed |

Zillow's per-URL cost isn't on that list. ScraperAPI's own wording is that consumption "depends on the site you're scraping and the parameters you send," which is why the Estimator, not the plan page, is where a Zillow budget actually gets decided.

## What Zillow actually gives you, field by field

The distance between "Zillow lists over 100 million properties" and "your dataset has a Zestimate in it" is where most real estate data projects quietly fail. Zillow decides what to show on a given search card, and it shows different things in different markets.

The clearest numbers come from a competitor's published run measurements: an Apify Zillow actor that scraped 1,000 Austin search rows and 200 Dayton rows with home details switched on, and reported fill rates per field.

| Field | Filled on Austin search rows |
| --- | --- |
| Price, home type, days on Zillow, ZIP code | 100% |
| Beds / baths | 99.6% / 99.9% |
| Latitude and longitude, photo URLs | 99.9% |
| Living area | 94.7% |
| Lot area | 86.0% |
| Broker name | 87.1% |
| Tax assessed value | 69.9% |
| Price-change flag | 51.0% |
| Rent Zestimate | 28.1% |
| Zestimate | 18.4% |

The Zestimate number isn't a bug in the scraper. Texas search cards rarely show a Zestimate at all. On the 200 Dayton rows from the same actor, Zestimate coverage was 62.5% and rent Zestimate 89.0%. Same code, different market, four times the coverage. If your product ranks homes by price-to-Zestimate ratio, it will look healthy in Ohio and half-empty in Texas, and you won't know why unless you measure coverage before you build on a field.

Going one level deeper, into individual home detail pages, closes some gaps and opens others: description, year built, county, schools, facts and nearby homes come back at 100%, price history at 99.0%, MLS ID at 97.5%, parcel ID at 96.0%, climate risk at 95.5%, tax history at 94.5%. HOA fee arrives on 9.0% of records. If monthly HOA cost is a filter in your product, Zillow detail pages alone won't support it.

The practical rule: treat price, address, beds, baths, area, photos, status and days-on-market as reliable. Treat tax assessed value as roughly a 70/30 bet, and treat Zestimate, rent Zestimate and HOA as fields you may need to source elsewhere. 👉 [Price out your own Zillow URLs on the free credits before you build the parser](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How many listings one run can produce

A single Zillow search tops out at 20 pages of 41 results, so 820 listings before you start slicing the map into boxes. That's the wall every scraper hits, and the workarounds differ. One Apify actor recursively quarters the map and increases zoom to push a single search past 600,000 results. Oxylabs returns 500 listings from Zillow's internal async search endpoint per request and accepts up to 5,000 Zillow URLs in a batch query.

ScraperAPI doesn't sell a record cap, because it doesn't sell records. One request, one Zillow page, one charge, and the volume question becomes arithmetic you can do in advance.

The piece most Zillow tutorials skip is the crawler. ScraperAPI's Crawler accepts a job at its job endpoint with a Zillow start URL, a `crawl_budget` denominated in credits, and a `url_regexp` that tells it which links to follow. Point it at a ZIP's search page with a regex matching `/homedetails/` and it walks the listing pages itself, caps the spend at the budget you set, and streams results to a webhook. Depth can be set with `max_depth` instead of a budget if you'd rather bound it that way, and `api_params` carries per-request settings like `country_code: us`.

Run the numbers before you pick a tier. At 1 credit per page, 5,000 home detail pages cost 5,000 credits. Add 10 credits per request for bot-protection bypass and the same 5,000 pages cost 55,000. A `crawl_budget` set at 55,000 is a hard stop, not a warning. 👉 [Compare the plans that match your monthly page count](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Will Zillow block you, and what does ScraperAPI do about it

Zillow leans on JavaScript, AJAX and CAPTCHA, and it flags IPs that behave like scrapers. The DIY route means managing rotating proxies and a rendering browser yourself, which is exactly what HasData's documentation lists as the two reasons people give up and buy an API instead.

ScraperAPI takes both off your plate. Proxy rotation is the product. JavaScript rendering is available through the render instruction set, passed as JSON with the request; ScraperAPI's own Zillow walkthrough uses exactly that, then parses the returned HTML with BeautifulSoup and writes it to CSV. Rendering and proxy settings are also among the parameters that change what a request consumes, which loops back to the Estimator point: the same Zillow URL can cost different amounts depending on how you ask for it.

One honest limitation. ScraperAPI returns HTML, not a Zillow schema. You write and maintain the parser, and Zillow's markup will change. If a field filling 18% of the time bothers you, a broken CSS selector will bother you more. Budget for that maintenance or pick a service that returns structured JSON.

## The legal question, briefly

Public listing data, meaning prices, addresses, square footage and sale status, is generally treated as fair game for scraping. Personal data is not. Agent names, phone numbers, emails and owner information that surfaces in skip-trace style lookups fall under privacy rules including GDPR (EU Regulation 2016/679), and "it was on a public web page" isn't the defence people assume. Both Oxylabs and Apify describe the legality of scraping as contested and recommend legal advice for your specific situation. That's the right framing: build the listing dataset, be deliberate about the people-data layer, and get a lawyer's read before you resell anything.

## How the alternatives compare

| Service | Input | Output | Volume | Price |
| --- | --- | --- | --- | --- |
| ScraperAPI | Any Zillow URL | HTML, plus Crawler jobs with webhook delivery | Pay per request, bounded by `crawl_budget` | 1 credit per standard page, +10 for bot-protected sites |
| Oxylabs Zillow Scraper API | `source` and `url` | Structured JSON from Zillow's async search endpoint | 500 listings per request, 5,000 URLs per batch | Not published |
| Apify `maxcopell/zillow-scraper` | Location or search URL | JSON records | 820 per search, 600,000+ with map subdivision | $2 per 1,000 results |
| Apify `automation_craft/zillow-scraper` | Places, ZIPs, map areas or search links | JSON, CSV, Excel rows with optional home details | Maximum homes per run | $0.85 per 1,000 events ($0.00085 per home) |
| HasData Zillow API | `keyword` and `type` (forSale, forRent, sold) | JSON listings | Not stated | Not published |
| RealtyAPI | Address, ZPID or Zillow URL | JSON with comps, 10-year Zestimate history, ZHVI, climate risk, skip trace, MLS lookups | Hundreds per bulk run | Not published |

Read the two Apify rows as separate products, because they are: different actors, different billing units (results versus events), and the $2 figure comes from a page dated 2023. The newest number on the table is the $0.85 per 1,000 events from October 2026. Oxylabs, HasData and RealtyAPI don't publish pricing at all, which makes them hard to compare without a sales call.

Where the others win is structure. RealtyAPI's endpoint list covers comps, a decade of Zestimate history, walk and transit scores, housing market indices and agent lookups, all as parsed fields. That's months of parser work you don't do. Where ScraperAPI wins is the unit economics of raw pages and the fact that you can test it in ten minutes without talking to anyone.

## Picking a plan

Match the tier to the job rather than to the biggest number on the page.

- **A few thousand listing pages a month, one metro, you're writing the parser anyway.** Burn the free 1,000 credits first, then Hobby. Auto-upgrade means you won't stall mid-run.
- **Crawling whole ZIPs on a schedule.** Use the Crawler with a `crawl_budget`, and pick a tier whose allowance covers your worst month rather than your average one.
- **Spiky volume, unpredictable months.** Scaling or Professional, where pay-as-you-go keeps a big month from forcing a plan change.
- **You need Zestimate history, comps or skip trace as structured fields.** ScraperAPI is not the short path. Look at the endpoint-first services above and accept that most of them won't quote a price without a call.

Three things to confirm before you enter a card number. Run ten real Zillow URLs through the Domain Cost Estimator, including a home detail page, not just a search page, because that's where your real per-record cost lives. Check the concurrency that comes with your chosen tier, since it caps throughput before credits ever run out. And settle monthly versus annual billing up front; annual plans carry their own renewal terms, and early renewal for a fresh batch of credits is an option rather than a default.

👉 [Start on the free 1,000 credits and move up to a paid tier once you know your cost per Zillow page](https://www.scraperapi.com/pricing?fp_ref=coupons)
