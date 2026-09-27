# Indian Equity Agents

A small multi-agent system for the Indian stock market (NSE/BSE) that
produces a daily recommendation brief. Runs entirely on free data and
free-tier services (Google News / Indian business-paper RSS, Yahoo Finance,
Groq, Streamlit).

| Agent | Reads | Produces |
|---|---|---|
| **Company News Agent** | Last 7 days of Google News (India) headlines for one company | Sentiment score, catalysts, risks, materiality |
| **Fundamentals Agent** | Yahoo Finance: ROE, margins, D/E, growth, P/E, P/B, 1Y trend, 50/200 DMA | Deterministic scorecard (quality, balance sheet, growth, valuation, trend) + narrative |
| **Market News Agent** | ET / Mint / Moneycontrol market feeds, RBI/FII/macro news, Nifty, Sensex, Bank Nifty, India VIX | Market mood, key themes, risks, sector tilts |
| **Daily Advisor** (orchestrator) | All of the above, for every stock on the watchlist | BUY / ACCUMULATE / HOLD / REDUCE / AVOID per stock + morning brief |

Each stock's action comes from a weighted blend — 55% fundamentals, 30%
company news, 15% market/sector view — so recommendations are reproducible.
The LLM writes the narrative but can't change actions or add stocks outside
the watchlist. Without `GROQ_API_KEY`, every agent falls back to rule-based
logic (keyword sentiment, scorecard only).

## Running it

```bash
pip install -r requirements.txt
cp .env.example .env                               # add GROQ_API_KEY (optional)

python -m equity_agents.cli daily                  # brief for equity_agents/watchlist.txt
python -m equity_agents.cli daily -s INFY TCS      # brief for chosen stocks
python -m equity_agents.cli news RELIANCE          # one agent at a time
python -m equity_agents.cli fundamentals HDFCBANK
python -m equity_agents.cli market

streamlit run app.py                        # dashboard
```

`daily` saves the brief to `reports/<date>.md` (git-ignored). Edit
`equity_agents/watchlist.txt` to change the stocks it scans.

**Daily delivery:** `.github/workflows/equity_daily_brief.yml` runs the brief
every weekday at 07:45 IST and attaches it to the Actions run (also shown in
the run summary). Add `GROQ_API_KEY` as a repository secret for the LLM
narrative.

Tests (offline, no network or keys needed): `python -m pytest tests`.

> **Not investment advice.** The agents use free, possibly delayed public
> data and are not a SEBI-registered adviser. Verify before acting.

## Project structure

```
app.py                         # Streamlit dashboard
equity_agents/
├── cli.py                     # command-line entry point
├── config.py                  # settings: st.secrets > .env
├── llm.py                     # Groq JSON helper (optional)
├── report.py                  # Markdown rendering of the daily brief
├── watchlist.txt              # stocks the daily brief scans
├── agents/
│   ├── company_news.py        # Company News Agent
│   ├── fundamentals.py        # Fundamentals Agent (scorecard)
│   ├── market_news.py         # Market News Agent
│   └── advisor.py             # Daily Advisor (orchestrator)
└── sources/
    ├── news.py                # RSS news sources
    └── market_data.py         # yfinance prices + fundamentals
tests/                         # offline tests with fake data sources
```
