# CS2 knife prices after a supply shock

How did the prices of Counter-Strike 2 knives react when the October 2025 game update let players trade up covert weapon skins into knives? The update suddenly made a large stock of existing items convertible into knives. This project measures each knife's price change over the two days around the update and asks which knives lost more value.

Python: Selenium, BeautifulSoup, pandas, duckdb, statsmodels.

## Data collection (`01_data_collection.ipynb`)

The data come from pricempire.com, a price aggregator that collects listings from many CS2 marketplaces. The pages are rendered with JavaScript, so they are scraped with Selenium.

1. **Listing pages.** All covert knives and guns, cases and keys, with name, type, wear, StatTrak/Souvenir status, listed stock and average price (about 4,800 items).
2. **Item pages.** Each item page is opened with Chrome's performance logging switched on. The price chart on the page is loaded from an internal API endpoint, and that request is picked out of the browser's network log, so the full price history can later be called directly. The scraper saves its progress to parquet and resumes where it stopped after a driver error or a bot check. Requests are spaced 2.5 seconds apart, so the full run takes about seven hours.
3. **Item-page variables.** The saved html is processed with duckdb in batches of 1,000 items: share of sales on western marketplaces, stock on each marketplace (summarised as a Herfindahl-Hirschman index), liquidity score, and the cases each item can be obtained from.
4. **Price history.** Price and stock on 22 October 2025 and price on 24 October 2025, from the API.

`test = True` runs the whole pipeline on a small sample (four listing pages, ten item pages) in a few minutes.

The data were collected in October 2025. pricempire.com has changed its page layout since then, so the CSS selectors no longer match and the scraper does not run as it is; the notebook documents how the data were collected.

## Analysis (`02_analysis.ipynb`)

**Dependent variable:** log change of a knife's price between 22 and 24 October 2025.

**Explanatory variables:**

| Variable | Meaning |
|---|---|
| `HHI` | Concentration of the knife's stock across marketplaces |
| `liq` | Liquidity score of the knife |
| `west2` | Squared share of sales on western marketplaces |
| `gun_relstock` | Stock of convertible covert guns (same cases, same StatTrak status) relative to the knife's own stock |
| `sqrt_gun_relval` | Square root of the mean value of those guns relative to the knife's price |
| `log_case_relstock` | Log stock of the knife's cases relative to the knife's own stock |
| `log_case_relval` | Log mean value of those cases relative to the knife's price |

The sample is the 1,129 non-Souvenir knives with more than 100 items in stock and complete data. The functional forms were chosen from scatter plots and a RESET test. Five interactions were added after screening all pairwise products by their correlation with the price change, and the final model keeps the terms significant at 5% with heteroscedasticity-robust (HC0) standard errors. Nested models are compared with F-tests.

## Results

The final model explains about 56% of the variation in price changes (R² = 0.56, N = 1,129). Knives with more convertible gun stock relative to their own stock lost more value, which is what the supply-shock mechanism predicts. Knives traded on more concentrated markets also lost more.

## Limitations

- The interactions were selected by looking at their correlation with the outcome, and the final model by p-values, so the reported p-values are not valid for formal inference. The model is exploratory.
- The final model keeps some interactions without their main effects.
- The RESET test still rejects the final specification, although less strongly than for the linear baseline.
- The price change covers two days, so it captures the immediate reaction, not the new equilibrium.

## Data

The scraped data are not included. The analysis notebook expects the output of the full run as `final.csv`.

Python packages: `selenium`, `beautifulsoup4`, `pandas`, `numpy`, `duckdb`, `fastparquet`, `statsmodels`, `matplotlib`, `tqdm`. Chrome and a matching ChromeDriver are needed for the scraping.

## Author

Kolos Mohácsi
