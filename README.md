# capitol-cli

Track U.S. House members' stock trades straight from the official STOCK Act filings — no API key, no third-party data broker.

```
$ capitol trades -t NVDA -n 2
2026-10-02  sell  NVDA   $15K–$50K    Pete Sessions TX17 [spouse]
                   NVIDIA Corporation - Common Stock (NVDA)
2026-10-01  buy   NVDA   $1K–$15K     Cleo Fields LA06
                   NVIDIA Corporation - Common Stock (NVDA)
```

- Downloads every Periodic Transaction Report from the [House Clerk](https://disclosures-clerk.house.gov/) and parses the PDFs into a local SQLite database.
- Cross-references [committee assignments](https://github.com/unitedstates/congress-legislators) so you can see who's trading what they oversee.
- Flags trades **filed late** (more than 45 days after the trade — the STOCK Act limit).
- `watch` mode for cron, with push alerts via [ntfy](https://ntfy.sh) or any webhook.

## Install

Needs Python 3.8+ and `pdftotext` (poppler):

```sh
brew install poppler        # macOS   (Arch: pacman -S poppler · Debian: apt install poppler-utils)
git clone https://github.com/NPCmillionaire/capitol-cli.git
ln -s "$PWD/capitol-cli/capitol" ~/.local/bin/capitol
capitol sync                # last year + this year (~1 min). Go further back: --since 2020
```

## Usage

```sh
capitol trades -t NVDA,TSLA --since 2026-01-01      # trades in specific tickers
capitol trades -m pelosi --type buy                 # a member's buys
capitol trades --late --min 50000                   # big trades filed late
capitol trades --state TX --stocks --sort amount    # Texas delegation, stocks only, biggest first
capitol top --by ticker --type buy --since 2026-07-01   # most-bought tickers
capitol top --by member --sort volume               # most active traders
capitol member moskowitz                            # committees + trading profile
capitol paper                                       # scanned filings that can't be parsed
capitol status
```

`trades` and `watch` take `--format json|csv`.

### Alerts

```sh
capitol watch -t NVDA,MSFT --notify ntfy:pick-a-secret-topic
```

The first run sets a baseline; later runs print (and push) only new trades. Put it in cron:

```cron
0 */6 * * *  ~/.local/bin/capitol watch --name tech -t NVDA,MSFT,AAPL --notify ntfy:pick-a-secret-topic
```

## Caveats

- **House only.** The Senate's eFD site sits behind bot protection that blocks non-browser clients.
- Amounts are disclosed as ranges (`$1,001 – $15,000`); "est. volume" sums range midpoints.
- ~10% of filings are scanned paper forms (images). They're listed by `capitol paper` with links, not parsed.
- Parsing is heuristic. If a trade looks wrong, check the source PDF: `capitol pdf <doc_id>`.

Data lives in `~/.local/share/capitol/` (override with `CAPITOL_HOME`).

## License

MIT
