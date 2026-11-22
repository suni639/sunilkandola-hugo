+++
title = "Where It Started: My First Small Projects from 2024"
date = 2024-06-20
lastmod = 2026-11-22
draft = false
description = "A roundup of the small projects I built in spring 2024 while learning data science and blockchain: house prices, a supermarket loyalty model, article scrapers, a first smart contract, a game and a childcare calculator."
summary = "Before the bigger projects on this site, I spent the spring of 2024 building small things to learn: a look at 25 years of London house prices, a dozen models predicting supermarket spend, scrapers for digital asset news, a first smart contract and a game. Several of them led straight to bigger work."

featureImage = "feature.png"
featured = false

tags = ["Python", "Data Analysis", "Machine Learning", "Web Scraping", "Blockchain", "Learning"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
showDateUpdated = true
+++

{{< lead >}}
Most of the projects on this site started as something much smaller. Between April and June 2024 I built a string of quick projects to learn data science, web scraping and blockchain. None of them deserves its own write-up, but together they show where the bigger work came from.
{{< /lead >}}

## Learning to work with data

### London house prices, 1995 to 2020

One of my first notebooks took a public dataset of monthly average house prices for every London borough and asked a simple question: how much have prices really moved?

{{< figure src="feature.png" alt="Line chart of average house prices from January 1995 to January 2020. London rises from about £74,000 to £477,000, England from about £53,000 to £247,000, with both dipping after 2008" caption="Average house prices in London and England, from the dataset I analysed." >}}

| | January 1995 | January 2020 | Rise |
|---|---|---|---|
| London average | £74,000 | £477,000 | 6.4 times |
| England average | £53,000 | £247,000 | 4.6 times |
| Fastest-rising borough | Hackney | | 9.3 times |
| Slowest-rising borough | Bexley | | 5.1 times |

By 2020 there was a huge gap between boroughs, from about £1.18 million in Kensington and Chelsea to about £301,000 in Barking and Dagenham. The notebook also broke the price series into its long-term trend and seasonal pattern, and the dip after the 2008 financial crisis stands out clearly in both lines.

### Predicting supermarket spend

Next came a practice dataset from DataCamp: a fictional supermarket's loyalty programme, with each customer's region, years in the scheme, joining month and promotions. The task was to predict how much each customer would spend in a year.

I trained twelve different models, from simple linear regression to XGBoost, and compared them on data they hadn't seen. XGBoost and gradient boosting did best, but I learned more from one of the weaker models. A single decision tree scored a perfect zero error on its training data and then made errors about twice as large as XGBoost's on new customers, because it had memorised the training set instead of learning general patterns from it. This is called **overfitting**.

---

## Tracking digital asset news

I was following tokenisation closely, so I tried to automate the reading.

1. **Two scrapers.** The first version had separate scripts for two news sites, Ledger Insights and Blockworks, saving each article's headline, link and date to a spreadsheet. Each site needed its own workarounds, such as clicking a button repeatedly to load older articles.
2. **One scraper with settings.** Version 2 combined them, moved the sites and keywords into a settings file, and appended new articles to a single file each week. By the end it held about 290 articles.
3. **Finding the themes.** I then ran simple text analysis on the headlines: word counts, a word cloud, and a **topic model**, which groups words that tend to appear together. It found five themes: bond issuance and security tokens (with Hong Kong and central bank digital currencies prominent), fund tokenisation, regulatory sandboxes including the UK's, exchanges and custody, and partnerships around tokenised deposits.

A third version never got past its first commit, and the idea later grew into the [automated weekly digital asset briefing](/projects/da-intel-01/) that now runs on this site every week.

---

## Building things

### A first smart contract

A "hello world" for blockchains: a tiny Solidity contract that stores a message and lets you change it, deployed on a test blockchain on my laptop. It taught me the tools, Truffle and Ganache, that I used a week later to log model updates on a blockchain in [FraudBusters](/projects/fraudbusters/).

### A first game

A simple arcade shooter in Python with Pygame, modelled on Space Invaders: move left and right, shoot the rows of aliens before they reach you, press R to play again. It was my first time handling a game loop, collisions and keyboard input.

### A childcare calculator

A small desktop app for a question many families face: would both parents going back to work actually pay, once childcare is counted? You enter both take-home incomes, the childcare bill and other costs, and it shows what's left, with a pie chart and saved scenarios.

Looking back, the maths was too simple, since it only took costs away from income. The real answer depends on free childcare hours, Tax-Free Childcare, the tax on the second income and the long-term value of staying in work, and a proper version would need all of those built in.

### The start of FraudBusters

I also explored a dataset of 6.4 million simulated online payments, of which only a tiny fraction were fraud. My first attempt to train a federated model on it ran out of memory, which pushed me to a smaller dataset and a simpler model. That became [FraudBusters](/projects/fraudbusters/).

---

## Key takeaways

1. **Several small projects led to bigger ones.** The scrapers turned into a weekly briefing, the smart contract became part of FraudBusters, and the payments dataset started the fraud work.
2. **Test on data the model hasn't seen.** A perfect score on training data should make you suspicious of the model.
3. **The projects that went furthest started from real questions.** They came from things I actually wanted to know, rather than from a tutorial.
4. **A simple version is useful for getting started.** The childcare calculator worked, but answering the real question would take much more than subtraction.

Most of the larger projects elsewhere on this site grew out of one of these.
