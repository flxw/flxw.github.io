---
layout: post
title: "Tracking your money made easy"
description: "An account system and the tools that allow you to track your expenses and your asset allocation easily"
category: blog
---

A couple of years ago, I started getting more and more into understanding how the financial system works,
what inflation is, what the central banks do, etc, pp.
Finally I ended getting a lot more thorough in terms of looking into where my money and time goes.
I would like to highlight the account structure and the two tools I have been using every since in this article.
I believe that the whole setup is fairly simple and can be used by anybody, so I would like to share it.
For every piece of the two-part system, I will first give a general description and then a bit of detail how I do it.

It is based on a few assumptions:
1. You got one or more bank accounts that you use mainly for spending.
2. It is possible to export the transactions from these accounts as CSV
2. The saved money goes onto the security accounts for later investing or just storage
3. You have one or more security accounts and checking accounts associated with them, and these accounts are only used for buying/selling securities

What comes out of this is what I call a cashflow system.
If you have read Robert Kiyosakis *Rich Dad, Poor Dad*, you know what I am referring to.
If not or you need a refresher, just google *Cashflow Quadrant*.

# What a cashflow system could look like
What I described in general terms above looks like this for me:

![](/assets/cashflow-system.jpg)

The income goes to an account that I refer to as *reserve account*.
It always holds 2-3 salaries that should only be touched as a last resort, e.g. when I lose my job or something serious happens.
The *reserve* feeds money onto my *budget account* for daily expenses on a monthly basis.
What's left of the income goes onto the deposit account for later investing.

Generally speaking, there are two categories here: spending and investing.\\
**Category I** is about spending. There will be high volume of spendtransactions, because I like to have every penny tracked and pay as little as possible with cash.
That's also why the CSV export is so important - you do not want to enter this by hand.
This will be covered in the first part.\\
**Category II** is about tracking your net worth. It's separate because it is irrelevant for spend analysis and the volume of transactions is comparably small.
It will be covered in the second part.

# Step 1: Spend analysis
**The tool:**
With [kmymoney](https://kmymoney.org/), it is possible to analyze your transactions in detail and categorize them easily.
Why am I recommending this instead of moneydance, homebank, lime and all the others?
Simple: CSV IMPORT AND MULTIPLE ACCOUNTS! These are simple but vital features that save you time and headache :)
You can even track securities and investments in kmymoney but I wouldn't recommend doing so.

Just import your CSV exports from your spending accounts there, enter the categories for the transactions and reconcile the amounts.
Then you can directly analyze your spend.

**How I use it:**
At the end of every month, I save the CSV exports from the accounts on my machine and import them into kmymoney.
After checking the categories and reconciling the final account balances, I can check the analysis.
What's especially important to me is the savings rate, i.e. how much of my income was not spent.

Tip #1: Do not delete the exports. They contain nice data that you can use to migrate to a different software later and keep all history.\\
Tip #2: Have the categorization done by your bank, this is why I went with N26 a few years ago.

I have recently begun to explore [beancount](http://furius.ca/beancount/) as an alternative because the work of reconciliating accounts and transactions between my accounts became too monotonous.
In combination with [Fava](https://beancount.github.io/fava/), the UI is quite nice.
It allows me to automate the whole CSV processing without needing to reconciliate a few transactions that went onto the wrong account.
The setup is very time-consuming though, so I will always recommend *kmymoney* first.

# Step 2: Security analysis
**The tool:**
Now that you know your savings rate and main cost drivers, you can think about what you want to do with the rest of the money.
Most likely you will buy something instead of leaving the money on some account, having it succumb to inflation.
I found [Portfolio Performance](https://www.portfolio-performance.info/) to be super easy to use and have very useful features.
The best is arguable the auto-update of stock prices - nowhere else have I seen it this easy.

To track something, just enter the price, fees and taxes associated with your transaction.
That's why the expected low volume of transactions mentioned above is key for this approach.
The nice thing is that you needn't track stocks.
The software allows you to create any category, so you could also see your worth in houses or cars.
Anything that's not on the budget or reserve accounts should be in here.

Just download the software.
There are example files included that give you an idea of what you could do or track with it.

**How I use it:**
After setting up the spend analysis, I put in the security transactions for the month.
Portfolio Performance allows me to easily check my asset allocation, so I can rebalance or adjust if necessary.

# Verdict
This article showed you how to structure your accounts into income/expense accounts and the tools to track accordingly.
What is a flaw, in my opinion, is that there is never a complete picture of your accounts.
Portfolio performance doesn't know about the expenses in kmymoney and vice versa.

I do not believe that this is a huge problem, however.
If you look at it, there should only ever be 2-4 months of salary inside the spend analysis.
The majority of your money should be somewhere in the security analysis.
With the level of transparency that you get here, I believe that it is a small price to always add 3x your salary onto the number that Portfolio Performance gives you.

Thanks for reading the first post since five years!
If you have any comments, please reach out to me via the contact details on the landing page!
