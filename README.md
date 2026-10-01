# Tour Price in AMD Agent (LangGraph)

A ReAct agent built with **LangGraph** and **Gemini** for a travel agency in Armenia. Tour operators send package prices in foreign currency, and the agency quotes the tourist in AMD using the partner bank's rule:

**price in AMD = package price × (Armeconombank sell rate + 2 AMD)**

The agent gets today's sell rate live from the Armeconombank website and calculates the price with a tool, so it never uses a rate from memory or calculates in its head.

Module 9 homework: Frameworks for AI Agents (LangChain, LangGraph).

## How it works

```
START → assistant → (tool call?) → tools → assistant → … → END
```

- **State**: `messages` (full conversation history)
- **assistant node**: the LLM with bound tools decides to answer, ask, or call a tool
- **tools node** (`ToolNode`): runs the requested tool and returns the result
- **tools_condition**: routes to `tools` or to `END`

## Tools

| Tool | What it does |
|---|---|
| `get_aeb_rate(currency)` | Scrapes today's buy / sell / Central Bank rates from aeb.am (requests + BeautifulSoup) |
| `quote_price(price, sell_rate, markup=2)` | Applies the rule: price × (sell rate + markup), rounded to whole AMD |

## Example run

**Question:** The tour operator sent a package to Dubai for 1350 USD. How much should we quote the tourist in AMD?

1. `get_aeb_rate("USD")` → sell rate 364.5
2. `quote_price(1350, 364.5, 2)` → applied rate 366.5
3. **Answer:** 494,775 AMD

## Evaluation (before / after)

| Test | Before prompt fix | After prompt fix |
|---|---|---|
| "Package to Paris costs 980 EUR" | ✅ 408,660 AMD | ✅ 408,660 AMD |
| "How much is 1350 in drams?" (no currency) | ❌ assumed USD and calculated | ✅ no tools called, asked which currency |

The fix added an explicit input check to the system prompt: a number without a currency is not USD, and no tool is called until both price and currency are known.

## Tech stack

Google Colab · LangGraph · LangChain (`langchain-google-genai`) · Gemini 3.5 Flash Lite · requests · BeautifulSoup · Colab Secrets

## Notes

- Rates change during the day, so results differ from the example above.
- Only the default rates table on the homepage (cash) is read; other tabs are loaded separately by the site.
- The scraper depends on the page structure (`__currency-lbl-<code>` labels in a table row). If the website changes, the tool returns an error instead of a wrong number.
- Data source: [Armeconombank](https://www.aeb.am). The homepage is not disallowed in the site's robots.txt.
