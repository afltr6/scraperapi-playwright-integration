# How to Use ScraperAPI with Playwright: Complete Integration Guide — API Endpoint vs Proxy Mode, JavaScript Rendering, Optional Parameters, and Plan Comparison (With Setup Code for Node.js and Python)

So you've got Playwright running. Pages load, selectors work, life is good. Then you try to scrape Amazon, or you spin up fifty concurrent sessions, and suddenly you're staring at a wall of CAPTCHAs and blocked IPs.

That's the moment most people start Googling "how to use ScraperAPI with Playwright."

This guide walks you through exactly that — step by step, with real working code, honest notes on what works and what doesn't, and a full breakdown of ScraperAPI's plans so you know what you're actually paying for before you commit.

---

## **Why Playwright Alone Isn't Enough for Production Scraping**

Playwright is genuinely excellent at what it does. It can render JavaScript, interact with dynamic pages, handle complex SPAs, and simulate real browser behavior better than almost any other tool. But it has one big blind spot: it's still coming from *your* IP address.

Run the same IP against a rate-limited endpoint a hundred times and you're done. No amount of `waitForSelector` magic is going to fix a ban.

What you actually need is:

- **Proxy rotation** — a different IP for every (or every few) requests
- **Automatic CAPTCHA handling** — so you don't have to implement solvers yourself
- **Geotargeting** — when the content you want is region-locked
- **Retry logic** — because the web is flaky and production scrapers need resilience

That's ScraperAPI's whole pitch. It sits between your Playwright script and the target website, handling all the messy infrastructure so you can focus on the data extraction logic.

---

## **How ScraperAPI Works with Playwright (The Two Approaches)**

There are two ways to plug ScraperAPI into a Playwright project. One works reliably. The other doesn't — and it's the one that sounds most intuitive, which is why it trips people up.

### **The API Endpoint Method (Recommended)**

Instead of navigating Playwright directly to `https://target-site.com`, you navigate it to a ScraperAPI URL that wraps the target:


https://api.scraperapi.com?api_key=YOUR_KEY&url=https://target-site.com


ScraperAPI intercepts the request, rotates proxies, solves CAPTCHAs if needed, and returns the rendered HTML to your Playwright browser. Simple, reliable, and properly authenticated.

### **The Proxy Mode (Why It Fails)**

You might be tempted to set ScraperAPI's proxy server (`proxy-server.scraperapi.com:8001`) in Playwright's `launch()` options. Don't. Playwright's proxy configuration expects Basic Auth or IP-based auth — not query string API keys. You'll hit `Proxy Authentication Required` errors immediately.

ScraperAPI themselves flag this in their documentation: proxy mode fails with Playwright because of how Playwright handles proxy authentication. The API endpoint method is the officially supported approach.

---

## **Setting Up ScraperAPI with Playwright (Node.js)**

Here's the full working setup. You'll need Node.js v18+, the `playwright` package, and `dotenv` for keeping your API key out of source code.

**Step 1: Initialize your project**

bash
npm init -y
npm install playwright dotenv


**Step 2: Create your `.env` file**


SCRAPERAPI_KEY=your_api_key_here


No quotes around the key — just the raw value.

**Step 3: Write your scraper**

Create `scraperapi-playwright.js`:

javascript
const { chromium } = require('playwright');
require('dotenv').config();

const SCRAPERAPI_KEY = process.env.SCRAPERAPI_KEY;
const targetUrl = 'https://httpbin.org/ip';
const scraperApiUrl = `https://api.scraperapi.com?api_key=${SCRAPERAPI_KEY}&url=${encodeURIComponent(targetUrl)}`;

(async () => {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  await page.goto(scraperApiUrl, { waitUntil: 'domcontentloaded' });

  const content = await page.textContent('body');
  console.log('IP Details:', content);

  await browser.close();
})();


**Step 4: Run it**

bash
node scraperapi-playwright.js


If it's working, your terminal will show an IP address that is *not* your home IP — ScraperAPI's proxy infrastructure at work.

---

## **Setting Up ScraperAPI with Playwright (Python)**

Python users aren't left out. You'll need `playwright` and `python-dotenv`:

bash
pip install playwright python-dotenv
playwright install chromium


Your `.env` file stays the same. The script:

python
import asyncio
import os
from urllib.parse import quote
from playwright.async_api import async_playwright
from dotenv import load_dotenv

load_dotenv()

SCRAPERAPI_KEY = os.getenv("SCRAPERAPI_KEY")
target_url = "https://httpbin.org/ip"
scraper_api_url = f"https://api.scraperapi.com?api_key={SCRAPERAPI_KEY}&url={quote(target_url)}"

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        context = await browser.new_context()
        page = await context.new_page()

        await page.goto(scraper_api_url, wait_until="domcontentloaded")

        content = await page.text_content("body")
        print("IP Details:", content)

        await browser.close()

asyncio.run(main())


Same logic, same result. ScraperAPI handles the heavy lifting; Playwright handles the browser automation.

👉 [Get your free ScraperAPI key and start scraping](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Optional Parameters That Actually Matter**

ScraperAPI lets you fine-tune each request with query parameters tacked onto the API URL. Here are the ones worth knowing:

| Parameter | What It Does | Extra Credits |
|-----------|-------------|---------------|
| `render=true` | Enables JavaScript rendering (headless Chromium on ScraperAPI's end) | +10 credits/req |
| `country_code=us` | Routes through a US-based IP | No extra cost |
| `session_number=123` | Sticks to the same proxy IP across multiple requests | No extra cost |
| `premium=true` | Uses premium residential proxies | +10 credits/req |
| `ultra_premium=true` | Uses ultra-premium proxies for hardest targets | +30 credits/req |
| `screenshot=true` | Returns a screenshot of the rendered page | +10 credits/req |
| `device_type=mobile` | Simulates a mobile device | No extra cost |

A URL with multiple parameters looks like this:

javascript
const scraperApiUrl = `https://api.scraperapi.com?api_key=${SCRAPERAPI_KEY}&render=true&country_code=us&session_number=123&url=${encodeURIComponent(targetUrl)}`;


One thing to know before you start stacking parameters: costs are not always additive. Combining `premium=true` and `render=true` costs **+25 credits**, not +20. Combining `ultra_premium=true` and `render=true` costs **+75 credits**, not +40. This non-linear stacking is real and it's documented, but it's easy to miss if you don't read the fine print.

---

## **Understanding the Credit System (The Part That Trips Everyone Up)**

ScraperAPI's pricing is built around an API credit system. One credit = one standard request. Except it's rarely one credit in practice, because different domains and different parameters cost different amounts.

Here's the base cost by domain type:

| Domain Type | Credits Per Request | Examples |
|-------------|-------------------|---------|
| Normal websites | 1 | Blogs, news, static HTML |
| E-commerce | 5 | Amazon, eBay |
| Search engines | 25 | Google, Bing |
| Social media | 30 | LinkedIn |

These domain multipliers are **automatic** — you don't opt in. The moment ScraperAPI detects you're hitting Amazon, it's 5 credits. Hit Google, it's 25. You don't get to choose.

The same goes for anti-bot bypass costs: if ScraperAPI detects Cloudflare, DataDome, or PerimeterX on your target site, it automatically adds +10 credits per request for the bypass mechanism.

This is why people open their dashboards three days after starting a scrape and find 80% of their monthly credits gone.

> **Pro tip**: Use ScraperAPI's [Domain Multiplier tool](https://www.scraperapi.com/?fp_ref=coupons) in the dashboard to look up the credit cost for any URL before you run a large batch. Saves a lot of expensive surprises.

---

## **Best Practices for Using ScraperAPI with Playwright**

A few things that'll save you headaches:

**1. Always use environment variables for your API key.** Never hardcode it in your script. `.env` + `dotenv` is the standard pattern and ScraperAPI recommends it explicitly.

**2. Use `render=true` only when you actually need it.** If you're scraping a standard HTML page, render mode costs you 10 extra credits per request for no benefit. Save it for SPAs and JS-heavy pages.

**3. Use `session_number` for multi-step scraping.** If your scrape involves navigating through several pages on the same site (say, a product listing, then individual product pages), set a `session_number` so ScraperAPI routes all those requests through the same proxy IP. Helps avoid detection from IP switching mid-session.

**4. Check success rates on your specific targets before committing to a paid plan.** ScraperAPI performs excellently on Amazon (98% success rate), Zillow (100%), and Etsy (99%). But it hits 0% on Instagram, Twitter/X, and Booking.com. Your mileage will vary by target.

**5. Monitor your credit usage daily for the first month.** There are no proactive usage alerts in ScraperAPI — no email warning when you're at 80% of your monthly limit. You have to check the dashboard manually.

**6. Avoid the Playwright proxy mode.** It doesn't work with ScraperAPI's authentication model. Use the API endpoint URL method described above.

👉 [Start free with 5,000 trial credits — no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

## **When JavaScript Rendering Actually Helps**

Here's a question worth thinking through before you add `render=true` to every request: does your target site actually need it?

If the content you want is in the raw HTML source — visible when you do View Source in your browser — then no, you don't need JS rendering. ScraperAPI's standard request will get it, and you save 10 credits per call.

If the content appears only after JavaScript executes — think dynamically loaded product grids, infinite scroll, React/Vue/Angular SPAs — then yes, you need `render=true`.

With Playwright + ScraperAPI together, you have two layers of JavaScript rendering happening: ScraperAPI renders on its end when you pass `render=true`, and Playwright is also a browser that renders JS. For most use cases with ScraperAPI's API endpoint method, you may only need Playwright's own rendering (since the page content is delivered via the ScraperAPI endpoint) without adding the `render=true` parameter. Test your specific target to see which combination gives you the content you need.

---

## **ScraperAPI Plan Comparison: Which Tier Makes Sense for Your Project**

ScraperAPI currently offers six tiers, from a free starter level to enterprise-scale custom plans. Here's the full breakdown:

| Plan | Monthly Price | Annual Price (per mo) | API Credits | Concurrent Threads | Geotargeting | Key Features |
|------|-------------|----------------------|-------------|-------------------|--------------|-------------|
| **Free** | $0 | — | 1,000/mo | 5 | ✗ | Good for testing; 5,000 bonus credits for first 7 days |
| **Hobby** | $49 | $44.10 | 100,000/mo | 20 | US & EU only | Structured data endpoints, standard proxies |
| **Startup** | $149 | $134.10 | 1,000,000/mo | 50 | US & EU only | All Hobby features, 10x credit volume |
| **Business** | $299 | $269.10 | 3,000,000/mo | 100 | 50+ countries | Country-level geotargeting, full global proxy pool |
| **Scaling** | $475 | $427.50 | 5,000,000/mo | 200 | 50+ countries | Pay-As-You-Go when credits run out |
| **Enterprise** | Custom | Custom | 5,000,000+ | 200+ | 50+ countries | Custom terms, dedicated support, SLA |

A few things worth noting that the plan names don't tell you:

- **Geotargeting beyond US and EU requires the Business plan ($299/mo) or above.** If your target audience is in Asia, Latin America, or elsewhere, the Hobby and Startup tiers won't get you there.
- **Pay-As-You-Go (PAYG) is only available on Scaling and above.** On Hobby, Startup, and Business, if you burn through your monthly credits before the billing cycle resets, your scraping just stops. No overage option. You either upgrade or wait.
- **Credits don't roll over.** Unused credits at the end of a billing period expire. There's no accumulation between months.
- **The 7-day free trial gives you 5,000 credits** — significantly more useful than the 1,000/month free tier for actually testing your target URLs at scale.

| Plan | Purchase Link |
|------|-------------|
| Free Trial |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby ($49/mo) |  [Get Hobby Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup ($149/mo) |  [Get Startup Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Business ($299/mo) |  [Get Business Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling ($475/mo) |  [Get Scaling Plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

---

## **Real Credit Math: What Your Plan Actually Gets You**

The headline credit numbers are a starting point, not the full picture. Here's what each plan actually delivers across common scraping scenarios — after multipliers:

| Plan | Simple HTML (1 credit) | E-commerce like Amazon (5 credits) | SERP like Google (25 credits) | Premium + JS Rendering (25 credits) |
|------|----------------------|------------------------------------|-----------------------------|-------------------------------------|
| Hobby ($49) | 100,000 pages | 20,000 pages | 4,000 pages | 4,000 pages |
| Startup ($149) | 1,000,000 pages | 200,000 pages | 40,000 pages | 40,000 pages |
| Business ($299) | 3,000,000 pages | 600,000 pages | 120,000 pages | 120,000 pages |
| Scaling ($475) | 5,000,000 pages | 1,000,000 pages | 200,000 pages | 200,000 pages |

If you're building a Playwright scraper targeting standard HTML blogs or news sites, the Hobby plan at $49/month goes a long way. If you're targeting Amazon or running Google SERP scrapes, the math gets expensive fast.

---

## **What ScraperAPI Is Genuinely Good At (And Where It Struggles)**

Based on independent benchmarks, ScraperAPI's performance is strongly site-dependent.

**It shines on:**
- Amazon (98% success rate, comprehensive structured data endpoints)
- Zillow (100% success rate)
- Etsy (99% success rate)
- Google SERPs (strong, with dedicated structured data endpoint)
- Walmart (93% success rate)

**It struggles with:**
- Instagram — 0% success rate
- Twitter/X — 0% success rate
- Booking.com — 0% success rate
- Realtor.com — only 12% success rate

**A hard no:** ScraperAPI explicitly forbids scraping data behind login walls in its Terms of Service. The `session_number` parameter enables same-IP persistence across a session, but it cannot handle form fills, authentication flows, or two-factor auth. If your target requires a login, this tool isn't the right fit.

The overall industry average success rate across all providers is around 58–59%. ScraperAPI sits slightly above at around 62–63%, with average response times between 5–7 seconds. Not the fastest provider out there, but solid for the supported targets.

---

## **ScraperAPI vs Writing Your Own Proxy Rotation**

One question that comes up a lot: why not just buy proxies and build your own rotation layer inside Playwright?

You can. It's a reasonable path if you have the engineering bandwidth and enjoy debugging proxy authentication issues at 2am. But for most teams, the calculation goes like this:

- Proxy pools that actually work at scale (residential, rotating) typically run $100–$500/month at baseline
- You still need to implement retry logic, CAPTCHA detection, session management, header rotation, and failure logging yourself
- That's weeks of engineering time, minimum, to get to production-quality reliability

ScraperAPI packages all of that into an API call. Whether that tradeoff is worth it depends entirely on your team's size, budget, and how many other things they have to build.

For a solo developer or a small team with a focused scraping use case, ScraperAPI's Hobby or Startup tier is almost certainly cheaper and faster than rolling your own. For a large engineering org with scraping expertise already in-house, building custom infrastructure might be more cost-effective at truly massive scale.

---

## **Getting Your Free API Key and Starting the Trial**

ScraperAPI's free tier is 1,000 credits per month with a 5-connection concurrency limit — enough to validate your integration but not enough to run a production workflow. The more useful starting point is the 7-day free trial: 5,000 credits, no credit card required, full feature access.

The setup order that makes sense:

1. Sign up for the free trial at ScraperAPI
2. Copy your API key from the dashboard
3. Use the Domain Multiplier tool to check the credit cost for your specific target URLs
4. Run the Playwright integration code above against your real targets
5. Measure how many credits you burn per 100 requests
6. Do the math: at your expected monthly request volume, which plan covers you?

That's really it. The integration itself is maybe 15 lines of code. The tricky part is understanding the credit economics for your specific use case before you commit to a monthly plan.

👉 [Start your free ScraperAPI trial — 5,000 credits, no card needed](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Frequently Asked Questions**

**Does ScraperAPI work with Playwright headless mode?**

Yes. The API endpoint method works regardless of whether Playwright is running in headless or headful mode. The `chromium.launch()` call doesn't need any special proxy configuration — you just navigate to the ScraperAPI URL.

**Can I use ScraperAPI with Playwright's Python async API?**

Yes, as shown in the Python example above. The `async_playwright` API works perfectly. Just make sure to `await` the `page.goto()` call and use `encodeURIComponent` (or Python's `urllib.parse.quote`) to properly encode the target URL.

**Do I need `render=true` when using ScraperAPI with Playwright?**

Not necessarily. Playwright itself is a full browser and renders JavaScript. The `render=true` parameter tells ScraperAPI to render the page on *its* end before returning HTML to your Playwright browser. For most Playwright integrations via the API endpoint method, try without `render=true` first — if you're getting the content you need, you don't need the extra 10 credits per request.

**What happens when I run out of credits on the Hobby plan?**

Your scraping stops until the billing cycle resets. Unlike the Scaling plan and above, lower tiers don't have a Pay-As-You-Go option. You can manually upgrade your plan mid-cycle, or wait for the reset.

**Is there an annual discount?**

Yes. Annual billing saves 10% across all paid plans. Hobby drops from $49/month to $44.10/month; Startup from $149 to $134.10; Business from $299 to $269.10; Scaling from $475 to $427.50.

**Can I get a refund if it doesn't work for my use case?**

ScraperAPI offers a 7-day no-questions-asked refund policy. If you're unhappy for any reason within the first week, contact their support team.

---

## **The Bottom Line**

Playwright is a great browser automation tool. ScraperAPI is a solid proxy and CAPTCHA management layer. Together, they cover most of what you need to scrape sites that would otherwise block you — without building proxy infrastructure from scratch.

The integration is straightforward: use the API endpoint URL method, store your key in `.env`, add optional parameters only when your target actually needs them, and check the Domain Multiplier before running any large batch.

The one thing to get right before you pick a plan: run the credit math for your specific targets. A scraper hitting Amazon or Google burns credits 5–25x faster than one hitting plain HTML pages. Know your multiplier before you sign up.

👉 [Try ScraperAPI free — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)
