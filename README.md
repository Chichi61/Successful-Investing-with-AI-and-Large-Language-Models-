# Financial Sentiment Analysis Using LLMs for Investment Signals

Using **GPT-4.1-mini** to extract short-term sentiment signals from financial news headlines and testing whether prompt design can materially change investment performance.

This was a team project completed at the Technical University of Munich. We were interested in a simple question: if an LLM is used to interpret the same financial news, how much does the way we prompt the model actually matter?

Rather than treating prompt engineering as a cosmetic step, we tested different prompt designs systematically and evaluated the resulting sentiment signals through out-of-sample portfolio backtesting and standard asset-pricing models.

For detailed prompt designs, portfolio performance analysis, factor model results, and visualizations, please refer to the presentation slides:

📂 `Campus Challenge_group 6.pdf`

---

## Project Overview

Financial markets process enormous amounts of textual information every day. Large language models offer a new way to turn that unstructured information into structured signals, but the quality of those signals can depend heavily on how the model is instructed.

Our main research question was:

> Can LLM-generated sentiment produce abnormal returns, and how sensitive is this performance to prompt design?

We therefore focused on two things:

- designing and comparing economically meaningful prompt specifications;
- testing whether the resulting sentiment signals remained useful after controlling for conventional market risk factors.

The project follows and extends the framework of Lopez-Lira and Tang (2023), with particular emphasis on **prompt robustness and systematic evaluation**.

---

## Data

We worked with firm-level financial news headlines and corresponding stock returns.

For prompt development, the news sample covered **January 2020 to June 2021**. After removing duplicate news events, the dataset was reduced from **13,570 to 10,824 unique headlines**.

For the final out-of-sample evaluation, we used sentiment signals for **128 trading days from July to December 2024**.

The training and evaluation periods were deliberately non-overlapping to create a clean out-of-sample design.

We also used standard asset-pricing factors from the **Kenneth R. French Data Library** for the regression analysis.

---

## Data Preparation

Before running the LLM analysis, we cleaned the news dataset to avoid overweighting repeated information events.

A unique `news_id` was created using:

- ticker
- date
- title
- content

Duplicate observations referring to the same underlying news event were then removed.

This reduced the dataset from 13,570 raw observations to 10,824 unique news items.

---

## Prompt Design

One of the most interesting parts of the project was testing how different prompt choices changed the resulting trading signals.

We varied several dimensions, including:

### Role Framing

We compared prompts that asked the model to think more like:

- a financial analyst focusing on fundamentals and valuation;
- a short-term trader focusing on immediate market reactions.

### Reasoning Guidance

We experimented with different levels of guidance, ranging from relatively simple classification instructions to more structured financial reasoning.

### Output Design

We also tested different sentiment structures, including:

- binary signals: `1 / -1`
- ternary signals: `1 / 0 / -1`

All prompt experiments were run with **temperature = 0** to improve reproducibility.

---

## Prompt Selection Process

Running every prompt specification on the entire dataset would have been expensive and inefficient, so we used a staged selection process.

Our workflow was:

```text
Prompt Design
      ↓
Controlled Subset Testing
      ↓
Generate GPT Sentiment Scores
      ↓
Construct Trading Signals
      ↓
Compare Sharpe Ratios
      ↓
Shortlist Prompts
      ↓
Full-Sample Validation
      ↓
Final Prompt Selection
```

We first evaluated prompts on identical randomly selected subsets of the data.

The strongest candidates were then re-tested on the complete training sample.

The final prompt was selected based on **risk-adjusted performance and consistency across samples**, rather than performance on a single subset.

---

## From Sentiment to Trading Signals

GPT-4.1-mini was used to classify financial headlines into sentiment-based trading signals.

The selected prompt focused on the **short-term market reaction** to corporate news and produced a binary trading signal:

```text
1  → positive expected short-term reaction
-1 → negative expected short-term reaction
```

These sentiment signals were then translated into investment positions.

---

## Portfolio Construction

We evaluated two simple strategies:

### Long-Only Portfolio

Stocks receiving positive sentiment signals were included in the portfolio.

### Long-Short Portfolio

Stocks with positive sentiment were held long, while stocks with negative sentiment were held short.

Portfolios were:

- equally weighted;
- rebalanced daily during the final evaluation;
- constructed using sentiment at time `t`;
- evaluated using returns at time `t+1`.

The one-period return shift was important because it avoids using future price information when constructing the portfolio.

---

## Performance Evaluation

We did not want to judge the strategy based only on raw returns.

Portfolio performance was evaluated using:

- Cumulative Return
- Annualized Return
- Annualized Volatility
- Sharpe Ratio
- Maximum Drawdown
- Skewness
- Excess Kurtosis

We also tested whether the returns could simply be explained by traditional systematic risk factors.

For this, we estimated:

- CAPM
- Fama–French Three-Factor Model
- Fama–French Five-Factor Model
- Fama–MacBeth regressions

HAC / Newey–West standard errors were used where appropriate to account for heteroskedasticity and autocorrelation.

---

## Results

The two portfolio strategies showed quite different characteristics.

| Strategy | Cumulative Return | Annualized Return | Volatility | Sharpe Ratio |
|---|---:|---:|---:|---:|
| Long-Only | 17.9% | 38.3% | 9.66% | 3.41 |
| Long-Short | 13.0% | 27.2% | 5.81% | **4.18** |

The long-only strategy generated higher absolute returns, but the long-short strategy had substantially lower volatility and therefore produced stronger **risk-adjusted performance**.

The long-short portfolio also had a near-zero market beta, suggesting that much of its performance was not simply driven by broad market movements.

---

## Factor Model Results

A particularly interesting result was that the long-short portfolio continued to generate positive and statistically significant alpha after controlling for traditional risk factors.

Approximate annualized alpha remained around **19%** across:

- CAPM
- Fama–French 3-Factor Model
- Fama–French 5-Factor Model

The corresponding t-statistics remained above 2 across the three specifications.

This suggests that, within our sample, the LLM-generated sentiment signal captured information that was not fully explained by standard market, size, value, profitability, or investment factors.

The long-only portfolio, in contrast, generated positive but statistically insignificant alpha after factor controls.

---

## What We Found About Prompt Engineering

One of the main lessons from the project was that **prompt design materially affects financial outcomes**.

Prompts were not simply different ways of asking the same question. Different role assumptions, reasoning instructions, and output structures changed the sentiment classifications and therefore the resulting portfolio performance.

In our experiments, prompts framed around **short-term trader reactions** produced stronger abnormal performance than more traditional fundamental-analysis framing.

This made prompt engineering itself an important part of the empirical model design.

---

## Risks and Limitations

The strong headline performance should be interpreted carefully.

The long-short strategy showed:

- substantial negative skewness;
- excess kurtosis;
- meaningful tail risk.

Daily rebalancing would also create transaction costs and market-impact concerns that were not fully incorporated into the backtest.

More broadly, LLM-based financial analysis raises several methodological issues, including:

- prompt-selection bias;
- overfitting;
- possible look-ahead effects from pretrained models;
- hallucinations;
- sensitivity to market regime and sample period.

For this reason, we see the project primarily as an empirical study of **LLM-generated financial signals and prompt robustness**, rather than a production-ready trading strategy.

---

## What I Learned

What surprised me most in this project was how much the prompt actually mattered.

Before working on this project, I thought of prompt engineering mainly as a way to give an LLM clearer instructions. But when we tested different prompts on the same financial news, we saw that relatively small changes in how we framed the task could lead to different sentiment signals and, eventually, very different investment results.

I also really enjoyed seeing the whole process come together: starting with unstructured financial news, turning it into sentiment signals with an LLM, building portfolios from those signals, and finally testing whether the returns still held up after controlling for traditional risk factors.

For me, this was probably the most interesting part of the project — seeing how an AI tool can be connected to a real quantitative finance question.

---

## Technologies & Methods

- GPT-4.1-mini
- Prompt Engineering
- Natural Language Processing
- Financial Sentiment Analysis
- Python
- Portfolio Backtesting
- Sharpe Ratio Analysis
- CAPM
- Fama–French 3-Factor Model
- Fama–French 5-Factor Model
- Fama–MacBeth Regression
- HAC / Newey–West Standard Errors

---

## Additional Resources

This README provides a high-level overview of the project.

For the complete analysis, including prompt specifications, methodology, regression results, visualizations, robustness tests, and detailed interpretation, please refer to the project report and presentation slides included in this repository.

📄 `report_group6.pdf`

📊 `Campus Challenge_group 6.pdf`
