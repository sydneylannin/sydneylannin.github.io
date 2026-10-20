---
# ---- The card on the Projects page ----
title: Predicting Customer Churn for a Telecom Company
tag: Machine Learning · Classification
card-summary: Building a model to flag which customers are about to leave, and finding out why they leave.
card-image: images/projects/telco-churn/card.png
color: blue

# ---- The top of the project page ----
eyebrow: INFO 4604 Applied Machine Learning · Solo
summary: Which customers are likely to leave (churn) next month, based on information the company already has about them?

meta: Role | Solo project
meta: Methods | Random Forest classification, KMeans clustering
meta: Tools | Python (pandas, scikit-learn), Google Colab

# ---- The left side: visual summary of the project ----
# (the visual itself lives in files/telco-churn.html; charts in images/projects/telco-churn/)
embed: files/telco-churn.html
link: Open the full notebook in Colab | https://colab.research.google.com/drive/1o4Cp1ICySV-m9VIpkM9TYm5G7yk5oq6n?usp=sharing
link: Dataset on Kaggle | https://www.kaggle.com/blastchar/telco-customer-churn
---

## The question and its importance

Which customers are likely to leave (churn) next month, based on information the company already has about them? This matters to a telecommunications company because keeping an existing customer is usually cheaper than winning a new one. If the company can flag at-risk customers early, it can target them with retention offers before they cancel. The model also uses feature importance to rank which factors drive churn, which tells the company what to fix, past just who to target.

## The data

The Telco Customer Churn dataset is a widely used benchmark dataset containing customer information and churn status for a fictional telecommunications company. Each row is a customer and the target is Churn (yes/no): did they cancel in the last month?

- **Demographics:** gender, SeniorCitizen, Partner, Dependents
- **Account:** tenure (months as a customer), Contract (month-to-month, one year, two year), PaperlessBilling, PaymentMethod (four types)
- **Services:** PhoneService, MultipleLines, InternetService (DSL, fiber optic, no) and six add-ons
- **Money:** MonthlyCharges, TotalCharges

## Context: why do telecom customers leave?

Churn "is a customer satisfaction indicator that represents the proportion of consumers or subscribers who stop doing business with a company during a given period." It is an issue because "losing a customer can generate costs for telecommunications companies, as they must invest in acquiring a new one," and "acquiring new customers in telecom costs 6-7 times more than retaining existing ones."

<details>
<summary>Reasons for high churn, from my research</summary>

1. Little to no automation in recurrent billing and billing issues. Current customers do not want to receive paper invoices, and even less so, to travel to pay a bill at an office. Complex or erroneous bills frustrate customers. Unexpected charges on postpaid plans trigger churn.
2. Problems qualifying and generating leads. To have an engaged customer base, a company needs to understand which type of customers are their best prospects.
3. Difficulty in resolving technical issues and customer service issues. No one wants intermittent service with their phone plan or internet speed. Companies need a team of empathetic, observant agents capable of finding and resolving problems. Poor support, long wait times, and unresolved issues is a huge churn driver.
4. Price Sensitivity: Customers switch to competitors offering lower prices or better value.
5. Competitor offers: promotions lure customers away.
6. Network reliability: service disruptions like call drops or slow internet.
7. Lack of personalization: generic plans failing to meet individual needs.
8. Poor onboarding experience. A confusing or overwhelming onboarding process can result in early abandonment.
9. Evolving customer needs: If a business doesn't evolve to fit new customer needs, it risks losing customers.

Sources: [Churn In Telecom: What It Is And Its Impacts](https://www.sydle.com/blog/churn-telecom-66b4c9e8b2713612fa17da85) · [Why do Telecom Customers Churn and How to Measure it](https://tridenstechnology.com/telecom-churn/) · [What causes churn and how businesses can minimize it](https://stripe.com/resources/more/what-causes-churn-and-how-businesses-can-minimize-it)

</details>

## Hypothesis

The three biggest factors that matter most for whether a customer churns, based on readings:

1. Tenure
2. Monthly Charges
3. Senior Citizen

My hypothesis chooses the three most likely factors from the existing data. The data, however, doesn't account for likely loss variables noted in the readings, like competitor offers, poor onboarding experience, network reliability, or customer experience encounters. So, this data may be limited.

## Biggest takeaways before the model

- From manual look throughs of the data I found that 11 customers had blank TotalCharges and all 11 had tenure = 0. This means that the 11 blanks are people who hadn't been billed yet, so the blanks were not errors, and could be filled in with 0.
- Several columns say "No internet service" instead of "No." (in OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies). This is repeating information already given by the InternetService column, so I collapsed it into "No."
- I found in the heat map that TotalCharges was a redundant column. It correlated highly with tenure, because it's calculated by roughly tenure x MonthlyCharges. I dropped this column to prevent feature importance being split between these two columns that say the same thing.
- I found with the KMeans clustering on tenure and MonthlyCharges that customers fall into four groups, the largest one being new customers paying high prices. This gives the company a good understanding of their customer base.
- In the correlation heatmap I found that Contract has the strongest relationship with churn, so longer contracts mean less churn.

## How did the model do?

From the confusion matrix, I can tell the model's accuracy is ~78% ((919 + 184) / 1409). This beats the baseline of always predicting "stayed" 73%, but not by very much. It also only caught 184 of 374 customers who churned, and a model that misses half of churners isn't very reliable.

## Hypothesis reflection

I predicted that the three main factors of churn would be tenure, MonthlyCharges, and SeniorCitizen. Feature importance showed that the actual three main factors are tenure, MonthlyCharges, and Contract.

## Overall

Overall the model is moderately effective, but not something the company should solely rely on. Its biggest value is explaining why customers leave (short tenure, high monthly bills, and easy outs with month-to-month contracts). This tells the company what to fix, but doesn't tell them which customers to call to get them to stay.

## My recommendation

If I were advising this company, I would tell them to focus on maintaining the newest customers via newbie deals, good customer service, and an easy onboarding experience. I would also advise them to use strategies like promotions, discounts, or add ons to get customers to switch from monthly plans to annual or two year plans, because customers on those plans tend to have lower churn.
