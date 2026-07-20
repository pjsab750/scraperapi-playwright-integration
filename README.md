# ScraperAPI Playwright Integration: How to Scrape Dynamic Websites Without Getting Blocked — Complete Setup Guide, Plan Comparison, and Pro Tips (With Discount Info)

So here's the situation a lot of developers end up in.

You've built a decent Playwright scraper. It opens Chrome, navigates through JavaScript-heavy pages, waits for elements to load, extracts the data you need. Works great in testing. Then you throw it at the real world — Amazon product pages, Google SERPs, e-commerce sites with Cloudflare protection — and the whole thing starts falling apart. IP bans, CAPTCHAs, 403 errors, rate limiting. You spend half your day debugging infrastructure instead of actually getting data.

This is exactly the gap that combining **ScraperAPI with Playwright** fills. Playwright handles the browser automation — the clicking, scrolling, waiting, extracting. ScraperAPI handles everything underneath: proxy rotation across 40 million+ IPs, automatic CAPTCHA solving, JavaScript rendering, and retry logic. You get the best of both worlds without having to build an entire scraping infrastructure from scratch.

Let's walk through how it actually works, who it's for, and whether it's worth the price.

---

## Why Playwright Alone Isn't Enough for Production Scraping

Playwright is genuinely excellent software. Microsoft's browser automation library supports Chromium, Firefox, and WebKit, and it handles the modern web better than almost anything else. For JavaScript-heavy single-page applications, it's often the only tool that actually works.

But Playwright alone has a fundamental problem: it's too easy to detect.

Anti-bot systems have gotten remarkably sophisticated. They don't just look at your user agent string anymore — they analyze browser fingerprints, mouse movement patterns, timing behaviors, TLS handshake signatures, and hundreds of other signals. A headless Chromium instance launched by Playwright sends signals that are distinctly different from a real user browsing on their laptop, and sites like LinkedIn, Google, and most e-commerce platforms detect this within seconds.

The usual workarounds — rotating your IP manually, trying different user agents, adding random delays — help a little but don't solve the underlying problem. You end up maintaining a list of proxy providers, writing retry logic, handling CAPTCHA services, and generally spending more time on scraping infrastructure than on your actual application.

That's the problem ScraperAPI was built to solve.

---

## What ScraperAPI Actually Does (and Why It Pairs Well with Playwright)

ScraperAPI is a proxy-rotation and browser-management API that's been running since 2018. They serve over 10,000 brands — including Deloitte, Sony, and Alibaba — and process around 36 billion API requests per month. The infrastructure is substantial.

When you route a Playwright request through ScraperAPI, here's what you're actually getting:

- **Proxy rotation across 40 million+ IPs in 50+ countries** — including datacenter, residential, and mobile IPs
- **Automatic CAPTCHA solving** — built-in, no third-party service needed
- **JavaScript rendering** — ScraperAPI can handle JS-heavy pages without Playwright even needing to do it
- **Geotargeting** — depending on your plan, you can specify which country your requests appear to come from
- **Automatic retries** — failed requests get retried automatically
- **Only charges for successful requests** — 200 and 404 status codes; actual failures don't count against your credits

The combination works like this: instead of Playwright hitting a target URL directly, you point it at ScraperAPI's endpoint with your API key and the target URL as a parameter. ScraperAPI handles the proxy selection, anti-bot bypassing, and response delivery. Playwright gets back clean HTML as if it had made the request itself.

---

## The Actual Integration: How to Connect ScraperAPI with Playwright

This is the part most tutorials gloss over. There are two approaches — one that works, one that doesn't.

### The Recommended Method: API Endpoint Approach

The cleanest and most reliable way to use ScraperAPI with Playwright is to wrap your target URL inside a ScraperAPI endpoint request. You're not using Playwright's proxy settings at all — you're using Playwright's browser to visit a ScraperAPI URL that returns the target page's content.

Here's a complete working example in Node.js:

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
  console.log('Response:', content);

  await browser.close();
})();


First, set up your environment:

bash
npm init -y
npm install playwright dotenv


Create a `.env` file:


SCRAPERAPI_KEY=your_api_key_here


Then run it:

bash
node scraperapi-playwright.js


If everything's working, you'll see an IP address in the output — and it won't be yours.

### Optional Parameters That Change Behavior

ScraperAPI accepts query parameters that let you control exactly how your request gets handled:

javascript
// Enable JavaScript rendering for JS-heavy pages
const url = `https://api.scraperapi.com?api_key=${KEY}&render=true&url=${encodeURIComponent(target)}`;

// Target a specific country
const url = `https://api.scraperapi.com?api_key=${KEY}&country_code=us&url=${encodeURIComponent(target)}`;

// Stick to the same proxy across multiple requests
const url = `https://api.scraperapi.com?api_key=${KEY}&session_number=123&url=${encodeURIComponent(target)}`;

// Use premium residential proxies (higher success rate on protected sites)
const url = `https://api.scraperapi.com?api_key=${KEY}&premium=true&url=${encodeURIComponent(target)}`;


> **Important**: `render=true`, `premium=true`, and `ultra_premium=true` each add to your credit consumption. More on this in the pricing section below.

### The Approach That Doesn't Work: Proxy Port Mode

You might find tutorials suggesting you use ScraperAPI's proxy port (`proxy-server.scraperapi.com:8001`) directly in Playwright's `launch()` options. Don't bother — it fails. Playwright expects Basic Auth or IP-based proxy authentication, but ScraperAPI requires the API key as a query parameter. The result is a `Proxy Authentication Required` error every time.

Stick to the API endpoint method above.

---

## Understanding ScraperAPI's Credit System (Before You're Surprised)

This is the part that catches almost everyone off guard, and it's worth spending some time on before you commit to a plan.

ScraperAPI bills on a **credit system**. The basic premise sounds simple: 1 request = 1 credit. In reality, what you actually consume depends heavily on what you're scraping and which features you enable.

### Domain-Based Multipliers

Some domains automatically cost more credits, regardless of any settings you choose:

| Target Site Type | Credits per Request |
|---|---|
| Normal websites (blogs, news, simple HTML) | 1 |
| E-commerce (Amazon, Walmart, eBay) | 5 |
| Search engines (Google, Bing) | 25 |
| Social media (LinkedIn) | 30 |

### Feature Multipliers (Stack on Top)

| Parameter | Additional Credits |
|---|---|
| `render=true` (JavaScript rendering) | +10 |
| `screenshot=true` | +10 |
| `premium=true` (premium proxies) | +10 |
| `ultra_premium=true` | +30 |

Here's the non-obvious part: **combining premium and rendering doesn't add linearly**. Premium (+10) plus rendering (+10) should logically cost +20 extra. ScraperAPI charges +25. Ultra-premium (+30) plus rendering (+10) should be +40 — but it's actually +75.

What this means in practice: if you're scraping Cloudflare-protected sites with `ultra_premium=true` and `render=true`, each request costs **75 credits**. That 100,000-credit Hobby plan you're eyeing? At 75 credits per request, it delivers about **1,333 actual requests**, not 100,000.

Run the math for your specific use case before choosing a plan.

---

## ScraperAPI Plans: Full Comparison Table

Here's every plan currently available, with pricing for both monthly and annual billing:

| Plan | Monthly Price | Annual Price (per month) | API Credits | Concurrent Threads | Geotargeting | Notable Features |
|---|---|---|---|---|---|---|
| **Free** | $0 | — | 1,000 | 5 | No | Basic testing only |
| **Hobby** | $49/mo | $44.10/mo | 100,000 | 20 | US & EU only | JS Rendering, Premium IPs, Data Pipeline, Analytics (30 days) |
| **Startup** | $149/mo | $134.10/mo | 1,000,000 | 50 | US & EU only | Everything in Hobby + Advanced bypassing, Structured Data APIs |
| **Business** | $299/mo | $269.10/mo | 3,000,000 | 100 | Global (50+ countries) | Everything in Startup + Global geotargeting |
| **Scaling** | $475/mo | $427.50/mo | 5,000,000 | 200 | Global | Everything in Business + Pay-as-you-go, Unlimited Analytics |
| **Professional** | $975/mo | $877.50/mo | 10,500,000 | 300 | Global | Everything in Scaling + Priority Support |
| **Advanced** | Custom | Custom | 21,500,000 | 500 | Global | Everything in Professional + higher concurrency |
| **Enterprise** | Custom | Custom | 22,000,000+ | 500+ | Global | Dedicated support team, Slack support, Ultra Premium, Pay-as-you-go |

| Plan | Purchase Link |
|---|---|
| Free Trial |  [Start Free Trial](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby |  [Get Hobby Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup |  [Get Startup Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business |  [Get Business Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling |  [Get Scaling Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional |  [Get Professional Plan](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise |  [Contact Sales](https://www.scraperapi.com/?fp_ref=coupons) |

**Annual billing saves 10% across all paid plans.** ScraperAPI also offers a 7-day free trial with 5,000 requests so you can test your actual targets before committing to a paid plan. If you're unhappy for any reason, there's a 7-day no-questions-asked refund policy.

> A few things worth knowing about billing: credits do **not** roll over between billing cycles. If you're on Hobby, Startup, or Business and exhaust credits mid-cycle, your service stops until renewal — Pay-as-you-go is only available on Scaling ($475/month) and above. Geotargeting beyond US and EU requires the Business plan.

👉 [Compare all plans and start your free trial](https://www.scraperapi.com/?fp_ref=coupons)

---

## Where ScraperAPI Performs Well (and Where It Struggles)

Based on independent benchmarks from April 2026, ScraperAPI's performance is sharply split by target site:

**Strong performance:**
- **Amazon** — 98% success rate, strong structured data endpoints returning 18+ parsed fields
- **Zillow** — 100% success rate, one of the best-performing targets
- **Etsy** — 99% success rate
- **Walmart** — 93% success rate
- **Google SERPs** — works well with structured data endpoints

**Weak or zero performance:**
- **Instagram, Twitter/X, Booking.com** — 0% success rate in independent testing
- **Sites requiring login** — explicitly not supported (ScraperAPI's ToS forbids it)
- **Realtor.com** — 12% success rate

If your primary targets are e-commerce and search engines, ScraperAPI is genuinely solid. If you need social media or login-gated data, you'll need a different approach.

---

## Real User Sentiment

Across Capterra (4.6/5 from 62 reviews), G2 (4.4/5), and Trustpilot (4.5/5), the pattern is pretty consistent.

**What people like:** The setup is fast. You can go from "just signed up" to making your first successful request in under 20 minutes. The documentation is clear. The API design is clean. The proxy infrastructure is solid on well-supported targets. Support is responsive when things go wrong.

**What people complain about:** The credit multiplier system is confusing, and costs can exceed expectations once you're hitting e-commerce or search targets at scale. Credits don't roll over. Pay-as-you-go is locked to higher tiers. And the dashboard doesn't send proactive alerts when you're running low on credits — you have to check manually.

One Reddit thread surfaced a case where a user was quoted a price for 60 million credits at 1 credit per Amazon request, not knowing about the 5x Amazon multiplier — they ended up with effectively 12 million requests for their money. The multiplier system isn't hidden, but it's not prominently explained either.

---

## Tips for Getting the Most Out of ScraperAPI + Playwright

A few things that save headaches in practice:

**Always store your API key in environment variables.** Never hardcode it. Use `.env` files locally and proper secrets management in production. ScraperAPI's quick-start guide shows this pattern and it's worth following from day one.

**Test your specific targets on the free tier first.** The 5,000-credit 7-day trial is genuinely useful for this. Run your actual URLs, with the actual feature flags you'll need, and calculate what your real credit burn rate will be before committing to a paid plan.

**Disable premium features unless you actually need them.** ScraperAPI doesn't auto-enable `render=true`, `premium=true`, or `ultra_premium=true` — you have to explicitly add them. But domain-based multipliers (Amazon = 5, Google = 25) are automatic and can't be disabled. Know which targets will trigger them.

**Check your dashboard daily during the first month.** There are no automatic usage alerts. You need to build intuition for your credit consumption rate. Analytics history is limited to 30 days on Hobby and Startup plans, and 6 months on Business and above.

**Use Structured Data Endpoints for supported sites.** If you're scraping Amazon, Google, Walmart, eBay, or Redfin at scale, ScraperAPI's pre-built JSON endpoints return parsed, structured data instead of raw HTML. They cost slightly more credits, but they save significant parsing and maintenance work.

---

## Is ScraperAPI + Playwright the Right Stack for Your Project?

The combination makes most sense if:

- You're a developer or part of a technical team building a data pipeline
- Your targets include Amazon, Google, or other e-commerce/SERP sites with high ScraperAPI success rates
- You need the browser automation capabilities of Playwright (clicking, form interaction, scrolling) combined with managed proxy infrastructure
- You're scraping at medium-to-high volume (100K+ requests/month)
- You want to avoid managing your own proxy pool, CAPTCHA service, and retry infrastructure

It's probably not the right fit if your primary targets are social media platforms (0% success on Instagram and Twitter/X), sites behind login walls, or if you need time-sensitive data and can't tolerate the potential 10-minute cache on difficult targets.

For most developer teams building production scrapers against mainstream web targets, though, it's a genuinely solid combination. The infrastructure handles the ugly parts, Playwright handles the browser automation, and you can focus on the actual data extraction logic.

👉 [Start your free 7-day trial with 5,000 requests](https://www.scraperapi.com/?fp_ref=coupons) — no credit card required for the free tier, and you can test your real targets before committing to anything.

---

## Frequently Asked Questions

**Does ScraperAPI work with Python Playwright?**
The quick-start guide currently shows Node.js examples, but the API endpoint approach works with any HTTP client in any language. For Python, you'd use `playwright-python` and construct the same ScraperAPI URL — the integration is language-agnostic since it's just an HTTP request.

**Will Playwright detect ScraperAPI's proxies?**
Playwright itself doesn't detect anything — it's just a browser automation tool. The sites you're scraping might detect bot behavior, which is exactly why using ScraperAPI's residential and premium IP pools helps. The API endpoint method routes through ScraperAPI's infrastructure, giving you clean, rotated IPs.

**Can I use ScraperAPI to scrape sites that need JavaScript?**
Yes — add `render=true` to your API call. This adds 10 credits per request. For Playwright-specific workflows, you can also use Playwright's built-in JavaScript execution after ScraperAPI delivers the initial page content.

**Is there a free plan?**
Yes — 1,000 API credits per month on the free tier, plus a 7-day trial with 5,000 credits when you sign up. 👉 [Create a free account here](https://www.scraperapi.com/?fp_ref=coupons).

**What's the refund policy?**
7-day no-questions-asked refund. Contact support and they'll process it.
