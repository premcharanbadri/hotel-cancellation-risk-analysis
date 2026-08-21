#  Predicting Hotel Booking Cancellations Risk

## [Executive Report](https://docs.google.com/presentation/d/1YyZr4baLn2duZQ8OOYHvxDd3QPLk3Se81mP5rMuG2U8/edit?usp=sharing)
## 1) Problem Statement

Hotels confirm thousands of reservations that never turn into actual stays. In our dataset, **37.1%** of all bookings were canceled before arrival *(Figure 1)*, which makes it hard for a hotel to plan staffing, set prices, and forecast revenue. The booked room value attached to those cancellations came to roughly EUR 16.7 million. While we do not treat that figure as lost revenue since many rooms were likely resold, it shows exactly how much money was exposed to cancellation risk.

Our question was simple: **at the moment a booking is confirmed, can the hotel tell which reservations are most likely to cancel, using only the information it has at that point?** The goal is to rank bookings by risk so staff can prioritize reminders, reconfirmation calls, and wait-list planning for the riskiest bookings. The score is not meant to reject guests or trigger automatic overbooking.

---

## 2) Data Acquisition & Cleaning

We used a public dataset containing 119,390 reservations from two real Portuguese hotels (a resort and a city hotel), with arrival dates spanning from July 2015 to August 2017.

- **Removed invalid data:** We dropped 180 bookings that listed zero guests, as well as rows with negative or impossibly high daily room rates above EUR 1,000.
- **Handled missing values:** We left blank "company" and "travel agent" fields as an explicit "not applicable" category, since most everyday guests simply book directly without an agent.
- **Retained Duplicates:** We kept 31,994 identical-looking rows. Because personal IDs were removed from the public dataset for privacy, these likely represent real group reservations (like a tour group booking multiple identical rooms) rather than data entry errors.

*(The principal data-quality issues and their treatment are summarized in Table 3).*

---

## 3) Exploratory Data Analysis (EDA)

- **Lead Time:** The earlier a booking is made, the riskier it is. Bookings made over a year in advance had a massive 67.7% cancellation rate, while last-minute bookings (0–7 days) canceled only 9.6% of the time *(Figure 2)*.
- **Guest History:** Guests with no special requests canceled at 47.7%, while repeat guests rarely canceled (14.7%). Furthermore, if a guest had canceled in the past, they canceled again 94.4% of the time.
- **Booking Route:** Travel agents and tour operators saw far more cancellations (41.1%) than guests who booked directly with the hotel (17.5%). Transient customers canceled at 40.8%, compared to just 10.1% for group customers *(Figure 3)*.
- **Hotel Type:** The city hotel experienced more cancellations overall (41.8%) compared to the resort (27.8%). Monthly cancellation patterns also differed between the city and resort hotels *(Figure 4)*.
- **Ignored misleading data:** We ignored fields like "requested parking" (none of which canceled) or "non-refundable deposits" (which had a 99.4% cancellation rate). While they perfectly predicted cancellations, they reflect strict hotel policies or post-booking data collection rather than natural guest behavior.

---

## 4) Feature Engineering

- **Total Nights:** We combined weekend and weekday stay columns into a single metric representing the full length of the trip.
- **Total Guests:** We merged adults, children, and babies into one single party-size number.
- **Arrival Weekday:** We created a feature to track the specific day of the week a guest arrives, helping to separate weekend vacationers from weekday travelers.

### Most Important Features - with Highest Signal:
- **Country, Agent, and Company:** These commercial identifiers were the strongest predictors of cancellation. Removing them caused the model's performance (PR-AUC) to drop significantly from 0.814 to 0.741.
- **Lead Time:** The number of days between the booking and the arrival date carried immense predictive power.
- **Special Requests:** Small effort signals, like asking for a specific room or amenity, indicated a real commitment to the trip.
- **Market Segment & Customer Type:** How the booking was categorized (e.g., transient vs. group) strongly signaled the likelihood of a cancellation.

### The Rest of the Features:
Hotel type, arrival date, length of stay, party size, meal plan, distribution channel, guest history, reserved room type, wait-list days, Average Daily Rate (ADR), and parking requests.

> **Note:** We removed any data that wouldn't be definitively known on the day of booking. This included the final reservation status, status dates, assigned room type, post-booking changes, and deposit type. Keeping these would mean the model was peeking into the future.

---

## 5) Modeling

Because cancellation behavior changed over time, we used a chronological evaluation instead of randomly shuffling the data. Models were trained on 2015 arrivals, validated on 2016 arrivals, and tested on 40,619 unseen bookings from 2017. 

We compared a dummy baseline, Logistic Regression, Histogram Gradient Boosting, and Random Forest. Validation results are reported in Table 1 and Figure 5.

Random Forest achieved the highest validation PR-AUC at 0.748, while Logistic Regression was practically tied at 0.746 and achieved a higher ROC-AUC of 0.814 versus 0.793. Based on our predefined tie-breaking rule and its greater interpretability, we selected **Logistic Regression**. 

On the 2017 test set, it achieved:
- **78.0%** accuracy
- **ROC-AUC:** 0.868
- **PR-AUC:** 0.814

The highest-risk 20% of bookings contained 44.3% of all cancellations and had 85.8% precision. Complete results appear in Table 2, with classification and calibration results in Figures 6 and 7.

---

## 6) Recommendations

1. **Target the Top 20%:** Have the front desk focus their outreach solely on the 20% of bookings the model ranks as highest risk. This approach identifies 44.3% of all cancellations while limiting staff outreach to only one-fifth of bookings *(Table 2)*.
2. **Offer Targeted Interventions:** Proactively secure high-risk bookings by offering incentives. This includes free room upgrades to higher tiers, larger beds, complimentary breakfast or lunch, spa coupons, or discounted rates for additional member stays.
3. **Keep it Friendly:** Use the risk scores to trigger helpful reminders and easy reconfirmation emails. Never use this score to automatically reject or penalize a guest.
4. **Smarter Wait-lists:** If the model identifies an unusually high concentration of high-risk bookings for a particular week, management can review wait-list capacity and inventory plans more closely. The prediction should support managerial judgment and should not be used by itself to authorize overbooking.
5. **Monitor Monthly:** Guest habits and market conditions change over time. The hotel should track the model’s accuracy by segment every month and retrain it when performance slips.

---

## 7) Limitations & Future Scope

The data ends in 2017 and covers only two anonymous hotels in Portugal, so the findings should not be applied blindly to other properties. Because booking IDs were removed, we could not track group identities or confirm whether identical rows represented separate reservations. Features such as country and agent identify predictive customer segments but do not establish the causes of cancellation. Future work should use newer data and incorporate room-resale information to estimate realized revenue impact rather than risk exposure.

---

## Web Application Demo

We deployed the selected Logistic Regression model as a browser-based prototype that allows hotel staff to enter booking-time information and receive an estimated cancellation probability and risk category. 

🔗 **[Live Demo: StaySure Hotel Risk Predictor](https://staysure-hotel-risk.hotel-cancellation-project.workers.dev/)**

---

## Appendix

### Figures (Placeholders)
* **Figure 1.** Overall cancellation proportion and annual trend.
* **Figure 2.** Canceled Bookings vs Lead Times across Hotel types.
* **Figure 3.** Canceled Bookings - Distribution Channels and Risk Profiles.
* **Figure 4.** Seasonality trends.
* **Figure 5.** ROC-AUC and PR curves.
* **Figure 6.** Confusion Matrix - Actual vs Predicted.
* **Figure 7.** Cancellation Rate vs Predicted Probability.

### Data Tables

**Table 1. Model comparison on 2016 validation data**

| Model | ROC-AUC | PR-AUC | Accuracy | Precision | Recall |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Random forest | 0.793 | 0.748 | 0.777 | 0.849 | 0.460 |
| Logistic regression | 0.814 | 0.746 | 0.759 | 0.831 | 0.411 |
| Gradient boosting | 0.802 | 0.722 | 0.736 | 0.844 | 0.323 |
| Dummy baseline | 0.500 | 0.359 | 0.641 | 0.000 | 0.000 |

**Table 2. Final 2017 test metrics**

| Metric | Result |
| :--- | :---: |
| ROC-AUC | 0.868 |
| PR-AUC | 0.814 |
| Accuracy | 78.0% |
| Balanced accuracy | 74.5% |
| Precision | 78.8% |
| Recall | 59.1% |
| F1 | 0.676 |
| Top-20% precision | 85.8% |
| Cancellation capture in top 20% | 44.3% |

**Table 3. Principal data-quality findings and treatment**

| Finding | Records | Treatment |
| :--- | :--- | :--- |
| Company missing | 112,593 | Not applicable category |
| Agent missing | 16,340 | Not applicable category |
| Country missing | 488 | Unknown / imputed in pipeline |
| Children missing | 4 | Median imputation in pipeline |
| Duplicate-looking rows | 31,994 | Retained; identity fields removed |
| Zero recorded guests | 180 | Excluded from modeling |
| Negative ADR | 1 | Excluded from modeling |
| ADR above EUR 1,000 | 1 | Excluded from modeling |

---

## References

* António, N., de Almeida, A., & Nunes, L. (2019). Hotel booking demand datasets. *Data in Brief*, 22, 41-49. [https://doi.org/10.1016/j.dib.2018.11.126](https://doi.org/10.1016/j.dib.2018.11.126)
* António, N., de Almeida, A., & Nunes, L. (2017). Predicting hotel booking cancellations with a machine learning classification model. *16th IEEE International Conference on Machine Learning and Applications*, 1049-1054. [https://doi.org/10.1109/ICMLA.2017.00-11](https://doi.org/10.1109/ICMLA.2017.00-11)
* Scikit-learn developers. scikit-learn: Machine learning in Python. [https://scikit-learn.org/](https://scikit-learn.org/)
