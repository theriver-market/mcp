# The River MCP server

[The River](https://app.theriver.markets) is the best place to trade and invest in tokenized equities: stock tokens from ST0x, Coinbase and Robinhood on Base and Robinhood Chain, compared in one place so each buy goes to the lowest price you may use. This is its remote [Model Context Protocol](https://modelcontextprotocol.io) server. An agent can read prices, holdings, history and P&L, and build market trades, limit orders and recurring buys that the user signs in their own wallet.

It answers questions an agent cannot easily answer elsewhere:

- **Which issuer's token of a stock is cheapest right now?** The same share is sold as a token by several issuers (ST0x, Coinbase, Robinhood) at different prices.
- **How far is the token from the real stock price?** Including nights and weekends, when the US market is shut.
- **How do I hand my user a trade they can check and sign themselves?** A link that opens a pre-filled ticket on The River, or the unsigned transactions themselves.

No account, no API key. The server never takes keys and never signs or sends anything: the build tools return unsigned transactions, each with a plain description, and the person signs them in their own wallet (or reviews a pre-filled ticket on [app.theriver.markets](https://app.theriver.markets)).

| | |
|---|---|
| Endpoint | `https://theriver.markets/mcp` |
| Transport | Streamable HTTP |
| Auth | none |
| Official MCP Registry | `markets.theriver/the-river` |
| Plain-text guide for agents | <https://theriver.markets/llms.txt> |
| Human page | <https://app.theriver.markets/agents> |
| Same features over plain HTTP + JSON | <https://app.theriver.markets/docs> (`/api/v1`, no key) |

## Connect

Claude Code:

```sh
claude mcp add --transport http the-river https://theriver.markets/mcp
```

Any client that takes a JSON config:

```json
{
  "mcpServers": {
    "the-river": { "type": "http", "url": "https://theriver.markets/mcp" }
  }
}
```

Check it answers:

```sh
curl -s https://theriver.markets/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## Tools

| Tool | What it returns |
|---|---|
| `compare_issuers` | Each issuer's token priced per share for a $100 buy, against the stock's own price, with the gap in % and the lowest issuer. Up to 12 tickers. From quotes taken every 5 minutes (biggest stocks) or hourly; the time is in the answer. Not a live quote. |
| `fair_value` | Last US market price and market status of listed stocks. Up to 12 tickers. |
| `list_stocks` | Listed tokenized stocks with ticker, token symbol, issuer, chain and trade link. Filter by chain or search text. |
| `trade_link` | A pre-filled trade link. A buy by ticker with `usd` opens the cheapest issuer the user may use. Give exactly one of `usd` or `qty`. |
| `strategy_link` | A pre-filled strategy card link (Base, ST0x stocks): buy the dip, recurring buy, take profit, stop, sell ladder and others. Fields are listed in the tool description. |
| `wallet_holdings` | Any wallet's stock-token and cash balances on the chains The River serves (public onchain data). |
| `wallet_pnl` | Any wallet's profit and loss on Base from public chain data, per token and per strategy. |
| `wallet_history` | Any wallet's stock-token buys, sells and transfers on Base and Robinhood Chain, newest first, with price and transaction. |
| `market_trade` | Build a market buy or sell on The River's best-execution route (the site's own route). Returns the quote (price, minimum out, fee, venue) and the unsigned approve + swap. |
| `place_limit_order` | Build a limit buy or sell against USDC (Base) or USDG (Robinhood Chain). Exact approve only, never unlimited. No fee to place. Unsigned transactions. |
| `list_limit_orders` | A wallet's limit orders on one chain: open, partially filled, filled or cancelled, with filled and remaining amounts. |
| `cancel_limit_order` | One unsigned transaction: withdraw what the order still holds, then remove it. |
| `withdraw` | Withdraw an exact amount or everything from one of the owner's order vaults. Unsigned. |
| `start_recurring_buy` | Build a recurring buy of an ST0x stock token: budget deposited now, spent over days or hours in buys of at least $25, never above your max price. No fee to start or stop. Unsigned. |
| `stop_recurring_buy` | One unsigned transaction: withdraw what the plan has not spent and what it bought, then remove it. |

Every build tool returns unsigned transactions for the user to sign in their own wallet, in order; nothing is sent or signed by the server.

## Example

Request:

```json
{"jsonrpc":"2.0","id":2,"method":"tools/call",
 "params":{"name":"compare_issuers","arguments":{"tickers":["AMZN","TSLA"]}}}
```

Answer, as returned on Sunday 11 October 2026 with the US market closed:

```text
- AMZN (stock last close $262.43; quoted 2026-10-11 01:45 UTC): ST0x: no fair quote (more than 5% off the US price); Coinbase $262.67/share (+0.09% vs the stock); Robinhood: no route for a $100 buy. Lowest: Coinbase
- TSLA (stock last close $382.7; quoted 2026-10-11 01:45 UTC): ST0x: no fair quote (more than 5% off the US price); Coinbase $382.8/share (+0.02% vs the stock); Robinhood: no route for a $100 buy. Lowest: Coinbase
Prices per share for a $100 buy at the time shown; not live. For an executable best-price buy, call trade_link with usd.
```

## What to know before relying on it

- **Prices are snapshots, not live quotes.** Every answer carries its time. The executable price is the one on the ticket the user opens.
- **Who may hold a token is set by its issuer.** Coinbase's stock tokens, for example, are not offered to US persons. The ticket shows the issuer's terms before a buy; the tools do not check the user's eligibility.
- **A token is not the share.** Each issuer owes the holder its value on its own terms.
- **Fees.** A market trade may carry The River's fee of at most 0.10%, inside the quoted amounts and named in the quote, plus network gas. Placing and cancelling limit orders and recurring plans has no River fee.
- **Rate limits.** Please stay under 60 requests a minute per address.
- **No warranty.** Nothing here is investment advice.

## This repository

It holds this description and `server.json` (the entry published to the official MCP Registry). The server itself runs at the endpoint above; there is nothing to install.

Problems or requests: open an issue here, or use "Report a problem" on the site.
