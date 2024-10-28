---
date: 2023-09-21
updated: 2023-12-26
state: 🌿
---
## learning full-stack javascript

This week at [[recurse center|Recurse Center]] (RC) I worked on a [Coding Black Females](https://codingblackfemales.com/) course [project](https://github.com/sassela/cbf-js-fraud-detection) to learn React, Node.js and ElasticSearch fundamentals. The project was a dummy fraud detection system for an internal team at a bank. It ingests data via ElasticSearch and allows users to set rules to identify potentially fraudulent transactions. I built the project in React, Node and TypeScript, and continuously deployed to Heroku.

Here are some of the things I learned!

## if something's difficult, there's probably a framework for it
A week or so into the project, I decided to port the the front- and back-end to TypeScript from JavaScript. I anticipated that TypeScript would help me to identify and debug type errors faster. I assumed (!!) the effort would be trivial and worth it for a smoother development experience.

It took me the best part of a week to port from JavaScript to TypeScript. I went about it clumsily and thoughtlessly; incrementally fixing errors by copy-pasting from different blog posts or Stack Overflow solutions without understanding what I was doing. The result was a Frankenstein project that either didn't run, test or deploy at various steps down the rabbit hole.

I eventually asked for help and paired with a fellow Recurser, Kathrin Ayer. Kathrin quickly identified all the wonderful ways my project had become misconfigured, like missing a crucial build step in a copy-pasted `project.json` file. She also mentioned that modern frameworks exist that made deploying full-stack TypeScript and JavaScript apps trivial.

If I were to do a project like this again, I'd either stick to the learning goals of the project (which were React and Node.js, not TypeScript) or look for alternative frameworks before wasting so much time.

## understand the data requirements
I wish I'd thought more about how I'd use my data before picking a data source to work with. I wanted one or more datasets representing banking transactions and flipped between two candidate sources:
1. [public charity donations from the ElasticSearch docs](https://github.com/elastic/examples/tree/master/Exploring%20Public%20Datasets/donorschoose).
2. [obfuscated fraudulent credit card transactions from Kaggle](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud).

The Kaggle dataset was much easier to integrate using [the Elastic File Data Visualiser tool](https://www.elastic.co/guide/en/kibana/6.5/release-highlights-6.5.0.html), so I started with that. Next, I wanted to use [Elastic machine learning tools](https://www.elastic.co/elasticsearch/machine-learning) to identify "similar", potentially fraudulent transactions. However, the transactions data didn't contain a `timestamp` property, and all its properties were obfuscated, which made it hard to work with any of the machine learning tools available.

In hindsight, the charity donations dataset would've been easier to use with ML tools; it contained timestamps and several non-obfuscated properties. But by the time I'd realised that I was close to the project submission date. I stuck with the credit card data and a quick notion of "similarity" based on numeric ranges.

## elasticsearch is good, actually
In the course of trying to work with different datasets, specifically [this charity donations one](https://github.com/elastic/examples/tree/master/Exploring%20Public%20Datasets/donorschoose), I went down a rabbit hole to reindex a data snapshot from 2017 to work with a newer version of ElasticSearch. I found the ElasticSearch docs clear and nice to find answers and, through them, experimented and learned about indexing, snapshots, mappings and analysers. Even though I didn't end up working with this dataset, I have no regrets about this diversion!

## in summary
Optimistically, I'm glad I learned these things in my first week at RC because I think it will help to make better use of my time here, and that documenting mistakes and what I've learned will help me to become a better software engineer.
