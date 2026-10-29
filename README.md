# zähler.

**Is your monthly prepayment enough?** zähler. tells you before the annual utility bill arrives.

zähler. is a meter reading tracker for electricity, gas, water and district heating. You type in your meter reading once a month, and it projects your consumption over the billing year and shows whether to expect a back payment or a refund. It runs as an offline PWA, needs no account, and nothing leaves your phone.

![License: AGPL-3.0](https://img.shields.io/badge/license-AGPL--3.0-blue) ![PWA](https://img.shields.io/badge/PWA-installable-5a0fc8) ![Offline](https://img.shields.io/badge/offline-yes-2ea44f) ![No dependencies](https://img.shields.io/badge/dependencies-none-lightgrey)

**Try it:** https://zaehler.gedachtgemacht.app (works in any phone browser, install via "Add to Home Screen")

The app interface is in German, and it follows how German utility billing works (monthly prepayment, annual settlement).

Plain arithmetic, no AI, no server. Everything stays on your phone.

<p align="center">
  <img src="bilder/1-uebersicht.png" width="240" alt="Overview with the expected total back payment and one card per meter showing forecast, range and traffic light">
  <img src="bilder/2-verlauf.png" width="240" alt="History of meter readings grouped by month, with consumption and days since the previous reading">
  <img src="bilder/3-detail.png" width="240" alt="Form for a new reading with meter, date, reading, optional meter photo and note">
</p>

## Why

Every household pays a monthly prepayment (Abschlag) for electricity, gas, water or district heating. Whether that money is enough, you normally only find out with the annual bill, sometimes together with a back payment of a few hundred euros. zähler. turns this around: you type in your meter reading once a month, the app projects it over the billing year and shows you right away whether a back payment or a refund is likely, and which prepayment would fit.

## Features

- **Meters** for electricity, gas, water, district heating and other, with unit price, standing charge, prepayment, billing start and starting reading from your last bill
- **Combined service charge**: one prepayment for several meters and fixed items together, for example 150 € a month for hot water, cold water, gas, insurance and ground rent. The traffic light applies to the total, not to each meter separately
- **Gas in m³ or kWh**, converted via calorific value and correction factor (default 11.2 and 0.95)
- **Traffic light** per meter: **reicht** ("enough"), **knapp** ("tight"), **reicht nicht** ("not enough")
- **Forecast** of back payment or refund, with an honest range instead of false precision
- **Recommended prepayment**, and how much you would need from now on to close a gap by the end of the period
- **Seasonal projection**: gas and district heating by a heating profile, electricity by a household profile, water evenly
- **Charts**: cumulative consumption against what your prepayment covers, plus the projection; consumption per month
- **Readings** with date, reading, an optional photo of the meter as proof, and a note
- **Plausibility check**: warning when a reading is lower than the previous one (with meter replacement), notice on unusually high consumption
- **Meter replacement**: final reading of the old meter and starting reading of the new one, consumption carries on correctly
- **Price change from a date**: new unit price, standing charge or prepayment in the middle of the year, applied to the exact day
- **New billing period**: takes over the last reading, archives the old period with its result and sets last year's consumption
- **Reminder** as a calendar file (.ics): monthly on the day you choose, plus an event at the end of each period
- **Export** of readings as CSV for German Excel (semicolon, decimal comma)
- **Backup** as a JSON file including photos, import on another device
- Light and dark theme, works offline, installable as an app

## Privacy

Nothing leaves your device. zähler. has no server, no account, no analytics, no ads, and loads nothing from the network. Meter readings, prices and photos are kept only in your browser's local database (IndexedDB). Exports and backups only happen when you trigger them yourself.

Because the data lives only in the browser: if you clear the site data, you also delete zähler. So make a backup now and then.

## Install

zähler. is a static web app. There is no build step and there are no dependencies.

1. Put the `zaehler/` folder on any static host (your own web space, GitHub Pages, Netlify, a Raspberry Pi). Installing it as an app requires HTTPS.
2. Open the address in your phone's browser.
3. In the browser menu choose **Add to Home Screen**. After that zähler. starts like an app and also works without a connection.

Test locally:

```
cd zaehler
python3 -m http.server 8000
```

Then open `http://localhost:8000` in your browser.

## How the math works

**1. Consumption so far.** Latest reading minus starting reading. With a meter replacement, both sections count: (final reading old minus previous reading) plus (current reading minus starting reading new). For gas in m³, the value is converted to kWh with calorific value × correction factor.

**2. Seasonal share.** Consumption is not spread evenly over the year. zähler. uses monthly weights (share of annual consumption in percent), spread day by day over the days of each month:

| | Jan | Feb | Mar | Apr | May | Jun | Jul | Aug | Sep | Oct | Nov | Dec |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Gas, district heating | 15.5 | 13.5 | 12 | 8 | 4.5 | 2.5 | 2 | 2 | 3.5 | 7.5 | 12 | 17 |
| Electricity (normalised) | 9.6 | 8.6 | 8.9 | 8 | 7.7 | 7.2 | 7.3 | 7.4 | 7.5 | 8.4 | 9.1 | 10.3 |

Water and other are spread evenly by days. The sum of the weights from the billing start to the latest reading gives the share that has typically been used already.

**3. Projection.** Annual consumption = consumption so far ÷ share.
Example gas: start 1 January, reading on 30 June, 5,900 kWh. January to May is 53.5 %, plus 29 of 30 June days at 2.5 % gives 55.9 %. Projection: 5,900 ÷ 0.559 ≈ 10,550 kWh.
If the share is below 15 % and last year's consumption is known, the two are blended: weight of the projection = share ÷ 15 %, the rest comes from last year.

**4. Costs.** Annual cost = annual consumption × unit price + 12 × standing charge. Paid = 12 × prepayment. Difference = back payment (positive) or refund. Recommended prepayment = annual cost ÷ 12, rounded up to whole euros. With price changes from a date, each section is calculated with its own price.

**Combined prepayment.** Meters in a combined service charge are projected one by one, their annual costs added up and supplemented by the fixed items (amount per month × 12, or per year). From that, 12 prepayments of the combined charge are subtracted. If one meter has no estimate yet, the combined charge shows no result, so the traffic light doesn't turn green too early.

**5. Traffic light.** Refund or back payment up to 3 % of annual cost: green. Up to 12 %: yellow. Above that: red.

**6. Uncertainty.** The range gets wider the smaller the share already read (up to about ± 30 % at the start, close to zero at the end of the period). Below a 10 % share the app says explicitly that the estimate is rough.

## License

GNU Affero General Public License v3.0 (AGPL-3.0), see `LICENSE`.

**Deutsch:** zähler. ist ein Zählerstand-Rechner für Strom, Gas, Wasser und Fernwärme, der aus deinen Ablesungen hochrechnet, ob dein Abschlag reicht oder eine Nachzahlung droht, offline als PWA und ohne Konto.

---

**gedacht.gemacht.** Small, honest software. Everything stays on your device. · [More projects](https://github.com/gedachtgemacht)
