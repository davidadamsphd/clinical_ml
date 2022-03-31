This repo contains my opinions about using machine learning in healthcare, specifically when the goal is to supplement or replace the judgment of healthcare providers.

When I was working at Verily, I attended an AI in healthcare conference. During one of the panel discussions, a panelist expressed the belief that "If my model has a better AUC, the doctor shouldn’t object". I was deeply distrubed by that comment because it exposed a lack of understanding of the myriad of ways that machine learning fails in the real world. I found the lack of pushback from other panelists even more distressing.

That experience caused me to raise these concerns with my clinical and data science colleagues at Verily. I ended up writing what I personally thought should be the process that a team should go through before a machine learning model is used for clinical decision support. Additionally, I was invited to speak at the Harvard Medical School Clinical Informatics Lecture Series in April of 2018, where I discussed how "Machine Learning isn't Magic"



# A bare minimum checklist for using a ML tool for clinical decision support
## Section 1: Proof model is worth investing in
The research group or vender should be able to provide satisfactory answers to these questions

- Tell me why I should care/Why this is a clinically relevant endpoint
  - What narrow question are you answering and why?
  - What change in care does this answer enable?
  - What are the risks of false positive, false negative, cost of the test (spell out the costs and benefits of each possible outcome)
- Give details of retrospective study design
  - What dataset(s) did you use for training?
  - What general data cleaning did you have to do?
  - Cohort selection (who was excluded and why) [keep an eye on bias in age, gender, general disease cohorts, test/intervention specific disease cohorts, ethnicity, socioeconomic status, etc.] 
  - How survival bias was handled?
  - How were the labels constructed?
  - How did you do train/validation/test?
  - What baseline model did you make and how do the results compare to your ‘real’ model?
  - How did your features and population change over time?
  - How does your accuracy change if you train on years 1 and 2 and predict on year 3, 4, etc.?
  - Have you trained on one healthcare system and test on another?
  - What features did your model use?
  - What features were most important? (What should have been, but didn’t make the cut)
  - What proof can you provide that the response isn’t leaking into the features?
  - What type of noise (or bias) do you expect in your data?
  - Have you injected noise or bias into your data? If so, how sensitive is your model?
  - Show me your AUC, f score, etc. stratified by every relevant cohort
  - Show me your calibration curves (again, overall and stratified by every relevant cohort)
  - Show me a set of random samples in test set: features (with importance), predicted outcome(s), and actual outcomes [Spot checking as a smell test]
  - Show me cases where the prediction changed dramatically for the same patient over a short time (if any occurred)
  - Can you tell when your model is confident? [point estimate as the outcome vs. posterior probability]
  - [EHR specific]: How do you handle the ICD-9/10 transition?
  - [What causal questions should be asked?]
- Explainability
  - What is the interpretation of the outcome?
  - Can the features be interpreted?
  - How will we know if the model has “gone off the rails”?

## Section 2: Deploying in a facility for observation
Predictions must be recorded as they would be used in clinical decision support [This is not a retrospective study] 

Things to record in this period
- Feature drift over time
- Population drift over time (all important strata)
- Labeling of outcomes for all cases (new ground truth)
- Strata that shouldn’t matter: operator/tech ID, device ID, device type, hour of day, day of week, etc.
- Initial (fixed) model predictions over time
- Online model predictions over time
- [If using devices] device output drift over time

Things to evaluate in this period
- Feature drift over time including:new features entering, features leaving, changing semantics, new devices, new device types, etc. (including distributions of feature values)
- Population drift over time (all important strata)
- Ground truth drift over time
- Feature importance over time
- Accuracy and calibration of initial fixed model
  - Over time
  - For all strata: populations, features that shouldn’t matter
  - Strata over time
 - Accuracy and calibration of online model
  - Over time
  - For all strata: populations, features that shouldn’t matter
  - Strata over time

It’s very likely in this process that the model will need some level of overhaul: different features, different modeling approach, etc. 
## Section 3: Deploy for clinical decision support 
Be paranoid
- Have a process to regularly review model inputs and performance (everything from Section 2)
- Have processes to automatically determine and flag when inputs or outputs are behaving unexpectedly (e.g., input is unlike anything seen before, output strongly disagrees with simple or previous model) and have fall-back
- Always be labeling a subset of the outcomes “by hand”
- Beware of feedback, both obvious and subtle (child hand age example)
- Always be wary of software bugs, and think of (system engineering) processes to mitigate
- Be prepared for updates to inputs: new features, change of feature definition, removal of old features (ICD-9/10 transition)
- Have ‘simple’ and previous models you can compare against and fall back to in case of sustained dramatic input changes
