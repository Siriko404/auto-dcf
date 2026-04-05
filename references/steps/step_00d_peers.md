# Step 0d: Peer Company Research

## Mandate
Identify 5+ comparable companies and extract market data for WACC calculation (Hamada unlevering/re-levering).

## Output
- `data/peers.csv` — one row per data point per peer
- `data/artifacts/{peer_ticker}/{data_type}.txt` — raw source excerpts

## Required Data Per Peer
- company_name, ticker, exchange
- market_cap (equity market value)
- total_debt (interest-bearing, excl lease liabilities where possible)
- cash_and_equivalents
- beta_levered (from market data provider)
- tax_rate (statutory rate for peer's jurisdiction)
- shares_outstanding

## Peer Selection Criteria
- Same industry/sector as target
- Similar business model (industrial distribution, equipment, etc.)
- Publicly traded with available market data
- Mix of direct competitors and adjacent players
- Minimum 5 peers, aim for 5-7

## Derived Fields (computed in CSV, marked confidence=derived)
- enterprise_value = market_cap + total_debt - cash
- debt_to_equity = total_debt / market_cap
- beta_unlevered = beta_levered / (1 + (1 - tax_rate) * debt_to_equity) [Hamada]

## Audit Checklist
- [ ] 1. At least 5 peers identified with justification for selection
- [ ] 2. Every peer data point has source_doc, source_page, source_quote
- [ ] 3. Market cap values are current (within last 30 days)
- [ ] 4. Beta values sourced from actual market data (NOT fabricated)
- [ ] 5. Debt excludes lease liabilities where data is available
- [ ] 6. Derived fields (EV, D/E, unlevered beta) computed correctly
- [ ] 7. At least 1 artifact file per peer saved
