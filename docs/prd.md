# Product Requirements Document (PRD)
## Household Electricity Consumption Forecasting Using Machine Learning

**Document status:** Initial draft for team review  
**Version:** 0.1  
**Last updated:** 12 October 2026  
**Project type:** College mini-project  
**Team:** A13  
**Team members:** Jnanesh, Akanksha K., Akshatha K., Abhinav N. G.  
**Project guide:** Prof. Shrinidhi A.

> **Important:** This PRD consolidates the current project description, review feedback from the guide/judges, and ideas raised by team members. Items marked **TBD**, **Proposed**, or **Needs validation** are not final decisions. Verify them before treating this document as a locked specification.

---

## 1. Product Overview

The proposed product is a web-based household electricity-consumption forecasting system. It aims to estimate electricity consumption at both appliance and household levels, identify periods of high predicted demand, and present results through understandable charts and insights.

The system is expected to consider historical consumption, time, weather/environmental conditions, household information, calendar events, and explicitly supplied exceptional conditions where suitable data is available.

The minimum appliance scope specified during the synopsis presentation/review is:

1. Refrigerator
2. Washing machine
3. Air conditioner (AC)
4. Lights
5. Fan

The system must support two forecast views:

- **Hourly forecast:** predicted consumption by hour, including identification of the peak-demand hour.
- **Whole-day forecast:** predicted consumption for the day, with an appliance-wise breakdown where supported by the data.

The system should also address exceptional cases such as appliance failure or maintenance, weather-related events, festivals, holidays, and family functions. The exact way these cases are represented and modelled remains to be defined during feasibility analysis.

## 2. Problem Statement

Households commonly learn their total electricity consumption after usage has occurred, such as when reviewing an electricity bill. A bill provides a retrospective total, but by itself it does not forecast future consumption, identify when demand is likely to peak, or explain how individual appliances may contribute to the expected usage.

Consumption can vary with time, weather, household routines, appliance availability, and special events. A forecast based only on a normal historical pattern may be less useful when the upcoming day differs from a typical day.

The project aims to investigate whether historical electricity-consumption data and relevant contextual information can be used to forecast appliance-wise and whole-household consumption, identify peak-demand periods, and present the results in a way that helps users understand and manage their usage.

**Problem statement to validate with the team:**  
“How can we forecast hourly and whole-day household electricity consumption, including the expected contribution of at least five appliance categories, and identify peak demand while accounting for relevant weather, calendar, and exceptional conditions?”

## 3. Product Goals

### 3.1 Primary goals

- Forecast electricity consumption for the five required appliance categories, subject to dataset feasibility.
- Provide hourly predictions and identify the hour with the highest predicted demand.
- Provide a whole-day consumption forecast.
- Consider relevant weather, time, household, calendar, and exceptional-condition information when available.
- Present forecasts in a clear, understandable interface.
- Help users review historical consumption and understand appliance contributions where the required data exists.

### 3.2 Secondary goals proposed by the team

- Provide graphs for historical and predicted consumption.
- Offer consumption insights and practical suggestions.
- Allow users to set a consumption threshold and receive an alert when usage or predicted usage exceeds it.
- Support user registration and login.

These secondary goals should be prioritized against time, data availability, and mini-project scope before implementation.

## 4. Target Users

### 4.1 Primary user — Household electricity consumer

A person who wants to understand past consumption, estimate upcoming electricity usage, identify high-demand periods, and make more informed energy-use decisions.

### 4.2 Potential user — Household member managing electricity usage

A household member who tracks consumption, compares actual and predicted usage, and looks for opportunities to reduce unnecessary electricity use.

**Needs validation:** Whether the product is designed for a single household profile, multiple household members, or multiple households per account.

## 5. User Problems and Needs

The product is intended to address the following needs:

- Users cannot directly know their future electricity consumption from a past electricity bill.
- Users may not know which supported appliance contributes most to predicted consumption.
- Users may not know which hour is expected to have the highest demand.
- Users may want to compare historical, predicted, and actual consumption.
- Users may want to understand how holidays, festivals, functions, weather, or appliance unavailability could change expected usage.
- Users may want warnings when consumption is above a user-defined limit.

These are problem hypotheses and should be checked with the guide and team; the project does not yet establish that every household will have appliance-level measurement data.

## 6. Scope

### 6.1 Minimum expected scope

Based on the guide/judges' review feedback:

- Forecast the consumption of at least five appliance categories: refrigerator, washing machine, AC, lights, and fan.
- Produce an hourly forecast.
- Identify and display the peak-demand hour.
- Produce a whole-day forecast.
- Consider exceptional conditions, including appliance failure, maintenance, weather-related events, festivals, holidays, and functions.
- Show the results to the user.

### 6.2 Proposed supporting scope

Subject to feasibility and team agreement:

- User registration/login.
- Household profile inputs such as region and number of occupants.
- Appliance inventory or appliance details.
- Weather-data retrieval for the selected region.
- Historical consumption tracking.
- Charts and comparisons.
- Appliance-wise insights and energy-saving suggestions.
- User-defined consumption limits and alerts.

### 6.3 Out of scope unless explicitly approved

To keep the mini-project manageable, the following are not assumed to be part of the first version:

- Real-time electricity control or switching appliances on/off.
- Guaranteed detection of physical appliance faults using sensors.
- Direct integration with utility-company smart meters.
- IoT hardware installation.
- Automated repair/maintenance scheduling.
- Exact electricity-bill or tariff prediction, unless the team explicitly adds it to the target.
- Production-scale deployment or support for all households in India.

## 7. Functional Requirements

Requirement IDs are provisional. Priority labels: **Must** = essential to the agreed minimum target; **Should** = valuable if feasible; **Could** = optional.

### FR-01: Household profile — Proposed, Should

The system should allow a user to provide relevant household information, potentially including region, number of occupants, and appliance details.

**Open point:** Determine which profile fields are genuinely required by the selected dataset and forecasting model.

### FR-02: Historical consumption data — Must

The forecasting pipeline must use suitable historical electricity-consumption data for model development and prediction.

**Dependency:** The team must identify a dataset with sufficient temporal coverage and appropriate appliance-level labels or measurements.

### FR-03: Five appliance categories — Must

The system must support predictions for the following minimum categories:

- Refrigerator
- Washing machine
- Air conditioner
- Lights
- Fan

The team must verify that each category can be predicted from the selected data. If the dataset does not measure these categories individually, the team must not present inferred values as measured ground truth without a defensible method.

### FR-04: Hourly forecast — Must

The system must produce an hourly consumption forecast for the forecast day, at the resolution supported by the selected dataset and model.

**Output to define:** Whether each hour is represented by energy consumed during that hour (for example, kWh) or by an instantaneous/average power value (for example, W or kW).

### FR-05: Peak-demand identification — Must

The system must identify and display the hour with the highest predicted demand for the selected forecast period.

The team must define whether “peak demand” means the hour with the highest predicted total household energy consumption or the highest predicted power demand. These are related but not identical quantities.

### FR-06: Whole-day forecast — Must

The system must provide a predicted whole-day consumption value. Where the model supports appliance-level forecasts, it should also show the contribution of each of the five required categories and the total.

### FR-07: Exceptional appliance conditions — Must, approach TBD

The system must account for relevant appliance conditions such as failure or maintenance when generating predictions.

The team must decide how users or the system specify the condition, how it changes the prediction, and what evidence or assumptions justify the adjustment. A manually supplied status flag may be a feasible initial approach, but it is not yet a locked decision.

### FR-08: Weather-related exceptional conditions — Must, approach TBD

The system must consider relevant weather-related conditions, including severe weather where suitable data is available.

The team must clarify whether “thunder” refers to a weather forecast, an electrical outage/surge, an appliance issue, or another event. The system must not claim to predict an electrical fault solely from thunder data unless this is supported by the data and method.

### FR-09: Calendar and special events — Must, approach TBD

The system must account for relevant calendar conditions such as weekends, holidays, festivals, and user-declared functions when generating predictions, where suitable features and data are available.

The team must decide how official holidays/festivals are sourced and how a private function is entered by a user. Event effects should be learned from suitable data where possible; unsupported fixed assumptions must be clearly identified.

### FR-10: Historical tracking — Should

The system should allow users to view available historical consumption and compare it with predictions and actual outcomes, where actual data is available.

### FR-11: Charts and visualizations — Should

The system should present relevant historical and forecast results through clear visualizations, such as hourly charts and appliance-wise daily breakdowns.

### FR-12: Insights and suggestions — Should

The system should identify notable consumption patterns and, where supported by the data, provide understandable energy-saving suggestions.

Suggestions must be based on available evidence and should not claim that a particular appliance caused excess consumption unless the data supports that conclusion.

### FR-13: Consumption threshold and alerts — Could

The system could allow users to set a daily or hourly consumption threshold and display an alert when the forecast or observed value exceeds it.

The team must distinguish a forecast-based warning from an alert based on real-time measured usage.

### FR-14: User authentication — Proposed, Should

The system may provide registration and login if user-specific profiles and saved history are included in the first version.

Authentication is not assumed to be necessary for the ML experiment itself.

### FR-15: Prediction history — Proposed, Should

The system should save forecasts with their generation time and forecast date if tracking, comparison, or evaluation requires persistent history.

### FR-16: Actual-versus-predicted comparison — Proposed, Should

Where actual consumption data becomes available, the system should support comparing predicted and actual values to help evaluate model performance and identify errors.

Storing new data alone does not automatically improve the model. Any retraining process must be separately designed and validated.

## 8. Inputs and Data Requirements

Potential inputs include the following. The final input list must be based on the chosen dataset and the features that are available at prediction time.

### 8.1 Historical electrical data

- Timestamp
- Household aggregate consumption, if available
- Appliance-level consumption for the five required categories, if available
- Data-quality or missing-reading indicators, where available

### 8.2 Time and calendar features

- Hour of day
- Day of week
- Weekend indicator
- Date/month or seasonal indicator, where useful
- Holiday/festival indicator
- User-declared function or special event, if supported

### 8.3 Environmental features

- Temperature
- Humidity, if available
- Weather condition or severe-weather indicator, if relevant and available

### 8.4 Household and appliance information

- Region
- Number of occupants, if available and useful
- Supported appliance inventory or appliance characteristics, if available
- Appliance status such as available, under maintenance, or failed, if manually supplied or otherwise reliably known

### 8.5 Data feasibility requirements

Before choosing a dataset, the team must verify:

1. Whether all five required appliance categories are represented.
2. Whether timestamps are sufficiently detailed to derive hourly consumption.
3. Whether the dataset has enough continuous history to support daily forecasting.
4. Whether weather and household/contextual data can be aligned with consumption.
5. Whether special events and failures are recorded, can be supplied as scenario inputs, or must be treated as limitations.
6. Whether the data represents one or many households and how that affects generalization.
7. Whether licensing and access conditions permit the intended academic use.

A dataset containing only aggregate household consumption cannot directly train five independently supervised appliance-consumption predictors unless additional suitable labels or a defensible disaggregation method are available.

## 9. Outputs

The proposed user-facing outputs are:

### 9.1 Hourly forecast

For each hour of the selected forecast day, show the supported predicted consumption values for each appliance and, if available, the household total.

### 9.2 Peak-demand information

Show the hour identified as the peak and the corresponding predicted demand value. Clearly state the unit and whether the value represents energy consumed over an hour or power demand.

### 9.3 Whole-day forecast

Show the predicted daily total and an appliance-wise breakdown for the five required categories, where supported.

### 9.4 Exceptional-condition scenario

When a supported exceptional condition is supplied, show the resulting forecast or scenario comparison. The interface should make clear which condition was considered and avoid presenting an unvalidated scenario estimate as a guaranteed outcome.

### 9.5 Historical view

If historical records are available, show previous consumption and comparisons with forecasts and actual values.

### 9.6 Insights and alerts

If included in the first version, show evidence-based usage insights, suggestions, and threshold warnings.

## 10. User Flows

### 10.1 Forecast generation — Proposed

1. User opens the application.
2. User selects or enters the relevant household profile.
3. User selects a forecast date or accepts the default forecast day.
4. System obtains the required historical and contextual data.
5. User supplies any known exceptional conditions or special events, if the system supports manual entry.
6. System validates and prepares the inputs.
7. Forecasting pipeline generates hourly and whole-day predictions.
8. System identifies the peak-demand hour.
9. System displays the results and any supported insights.

### 10.2 Historical tracking — Proposed

1. User opens the consumption history view.
2. System displays available historical records.
3. User selects a date range or view.
4. System displays graphs and, where possible, actual-versus-predicted comparisons.

### 10.3 Exceptional-condition scenario — Proposed

1. User selects or enters a relevant condition, such as appliance maintenance or a special event.
2. System validates whether the condition can be represented by the available model or rule.
3. System generates a forecast using the supported inputs.
4. System identifies the condition used and displays the resulting forecast.
5. If the condition cannot be modelled reliably, the system communicates the limitation instead of implying a validated prediction.

## 11. Machine-Learning Requirements

### 11.1 Prediction targets

The proposed targets are:

- Hourly consumption for each required appliance category.
- Whole-day consumption, either predicted directly or derived from hourly forecasts using a method validated by the team.
- Peak-demand hour derived from the hourly prediction.

The team must decide whether to train separate models per appliance, a multi-output model, or another approach after inspecting the dataset.

### 11.2 Candidate approaches

Potential baseline and candidate models include:

- Linear Regression
- Random Forest
- XGBoost
- Time-series or sequence models, if justified by the amount and structure of data

No model is finalized by this PRD. Model selection must be based on data suitability, validation results, complexity, and available project time.

### 11.3 Data preparation

The pipeline should examine and appropriately handle:

- Missing values
- Duplicate records
- Invalid or inconsistent readings
- Timestamp alignment and sampling intervals
- Outliers and sensor errors
- Unit consistency
- Feature creation from timestamps and calendar information

Any removed or transformed data should be documented.

### 11.4 Evaluation

The team should compare suitable models against a simple baseline and use a time-aware validation strategy to avoid training on future data when evaluating forecasts.

Potential regression metrics include:

- MAE (Mean Absolute Error)
- RMSE (Root Mean Squared Error)
- R², where appropriate

The team should report performance separately for relevant appliances and forecast horizons where enough test data exists. The meaning of “accuracy” should be defined using these measurable results rather than a vague claim.

### 11.5 Exceptional-case evaluation

The team should test whether predictions behave reasonably under supported exceptional scenarios. If the training dataset contains no examples of failures, maintenance, or certain events, the team must document that limitation and distinguish rule-based scenario handling from patterns learned by the model.

### 11.6 Retraining and feedback

Actual future consumption may be stored for later evaluation. Incorporating it into training requires data validation, versioning, retraining, and re-evaluation. Automatic online learning is not assumed to be part of the first version.

## 12. UI/UX Requirements

Based on the team's initial suggestions:

- The interface should be simple and familiar to users.
- Navigation and actions should be consistent and clearly labelled.
- Forecasts should be understandable without requiring ML knowledge.
- Hourly results should make the peak hour easy to identify.
- Daily results should show total consumption and the appliance-wise contribution where available.
- Graphs should have readable labels, dates, and units.
- Exceptional conditions considered in a forecast should be visible to the user.
- Alerts and suggestions should explain what triggered them.
- Errors, missing data, and unsupported scenarios should be communicated clearly.

The exact screens and navigation structure remain to be designed.

## 13. Non-Functional Requirements

### NFR-01: Usability

Users should be able to locate the forecast, peak hour, and historical view without needing technical knowledge.

### NFR-02: Performance

The application should generate and display forecasts within a practical response time for the selected dataset and deployment environment. A numeric response-time target is TBD.

### NFR-03: Reliability

The system should handle missing or invalid inputs without silently presenting them as valid predictions.

### NFR-04: Security and privacy

If accounts and household data are stored, access should be restricted appropriately and only necessary personal information should be collected.

### NFR-05: Explainability

The system should clearly distinguish predicted values, actual measured values, and scenario estimates.

### NFR-06: Maintainability

Data preparation, model training, prediction logic, and presentation should be structured so that the team can test and update them independently.

### NFR-07: Reproducibility

The team should record the dataset version, preprocessing choices, model configuration, and evaluation method used for reported results.

Numeric thresholds for performance, reliability, and evaluation remain TBD until the team examines the data and implementation constraints.

## 14. Constraints and Assumptions

### 14.1 Constraints

- This is a college mini-project with limited development time and computing resources.
- The forecasting scope depends on publicly or otherwise legitimately accessible data.
- Appliance-level forecasting requires suitable appliance-level labels/measurements or a separately justified method.
- Weather and calendar data must be aligned with the forecast date and region.
- Data collected outside India may not fully represent Indian household routines, appliance mixes, weather, tariffs, or electricity reliability.

### 14.2 Assumptions to validate

- Users can provide or the system can obtain enough information to identify the household's relevant appliance/context.
- Historical data has patterns useful for future forecasts.
- At least one feasible dataset supports the required appliance categories and hourly/daily targets.
- Calendar and exceptional conditions can be represented as input features, scenario controls, or clearly documented rules.
- A useful academic prototype can be developed without direct access to real-time utility smart meters.

These are assumptions, not established facts.

## 15. Success Criteria

The project will be considered to meet its minimum functional target when:

1. The system produces hourly predictions for the five required appliance categories, using a documented and defensible data/model approach.
2. The system produces a whole-day consumption forecast.
3. The system identifies the peak-demand hour from the hourly output.
4. The team demonstrates supported exceptional-condition scenarios, including the conditions that can actually be represented by its data or rules.
5. The evaluation reports model performance on held-out data using appropriate metrics.
6. The interface presents predictions, units, and relevant limitations clearly.
7. The team documents unsupported conditions and dataset limitations rather than claiming capabilities that were not tested.

Numeric model-performance acceptance thresholds are TBD and should be chosen after baseline experiments.

## 16. Future Enhancements

Potential future enhancements, not commitments for the first version:

- Real-time smart-meter integration.
- IoT-based appliance monitoring.
- More appliance categories.
- Personalized model adaptation using a household's own history.
- More sophisticated weather/event scenario modelling.
- Notifications through email or messaging platforms.
- Automatic retraining workflows.
- Electricity tariff and bill-cost forecasting.
- Household-to-household comparisons, where privacy and data quality permit.

## 17. Open Questions and Decisions Required

The team must resolve these before finalizing system design and implementation:

1. **Dataset:** Which dataset provides usable data for all five required appliance categories?
2. **Forecast horizon:** Does “hourly prediction” mean the next 24 hours, each hour of the following calendar day, or both?
3. **Daily target:** Is daily consumption the sum of the 24 hourly predictions, a separate daily model output, or should both be compared?
4. **Units:** Will the system predict hourly energy (kWh), average/peak power (W/kW), or both?
5. **Peak definition:** Is the required peak the highest-demand hour of the household total, or should appliance-level peaks also be shown?
6. **Appliance data:** Are lights represented as one category, multiple circuits, or a proxy? Can the chosen dataset distinguish lights and fans?
7. **Exceptional conditions:** Which conditions must be model-learned and which may be represented by user-provided scenario inputs or explicit rules?
8. **Appliance failure:** What exactly should be predicted when an appliance fails—its consumption, the household's adjusted consumption, a fault occurrence, or a scenario?
9. **Thunder/weather:** What event is intended, and what data source or measurable variable represents it?
10. **Events:** Which region's holiday/festival calendar is needed? How will private functions be entered?
11. **Household input:** Is the number of appliances alone sufficient, or are appliance type, rating, usage schedule, and status also needed?
12. **Weather service:** Which weather source is available and permitted for academic use?
13. **Historical tracking:** Where will actual consumption history come from if the user has no smart meter or appliance-level sensors?
14. **Authentication:** Is login required for the first working prototype?
15. **Suggestions:** Will suggestions be simple rule-based recommendations or derived from model/data analysis?
16. **Alerts:** Are alerts based on forecast values, actual measured values, or both?
17. **Model evaluation:** What baseline and acceptance thresholds will be used?
18. **Scope priority:** If the selected dataset cannot support every exceptional case, what minimum demonstrable handling will the guide accept?

## 18. Glossary

- **Consumption:** Electrical energy used over a period, commonly measured in kWh.
- **Power/demand:** The rate at which electrical energy is being used, commonly measured in W or kW.
- **Hourly forecast:** A prediction for each hour in the selected forecast period.
- **Whole-day forecast:** A prediction of total energy consumption across the forecast day.
- **Peak-demand hour:** The hour identified as having the highest predicted demand according to the project's chosen definition.
- **Feature:** An input variable used by the model.
- **Target:** The value the model is trained to predict.
- **Exceptional condition:** A condition that may cause consumption to differ from a typical pattern, such as maintenance, failure, severe weather, or a special event.
- **Ground truth:** The actual observed value used to train or evaluate a prediction.

---

## Change Log

- **v0.1 — 12 October 2026:** Initial consolidated draft based on the project description, guide/judges' review requirements, and team brainstorming. Open decisions remain intentionally unresolved.
