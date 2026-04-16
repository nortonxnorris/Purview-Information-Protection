# Strategy for leveraging built-in classifiers

Corporate espionage is no longer the stuff of spy novels; it is an everyday concern for modern enterprises. Insider Risk Management (IRM) in Microsoft Purview helps you spot suspicious activity quickly, curbing data theft or sabotage before it damages the business. Below is a practical walkthrough for turning IRM insights into action.

## Why Insider Risk Management?
- IRM ships with Microsoft 365 E5 and Compliance E5, letting you leverage existing licensing.
- It ingests signals from Microsoft DLP, Communication Compliance, and other Purview solutions for broader context.
- Built-in analytics surface recommended policies so you can move from data to action fast.

## Step 1 - Turn On Analytics
Start with **Settings -> Insider Risk Management -> Analytics**. Analytics mines historical audit logs to uncover risky patterns and recommends starter policies. Wait for the initial scan to complete; your recommendations list is the perfect launchpad for tuning IRM to your environment.

## Step 2 - Define Priority Users and Locations
Identify the people and places that matter most:
- **Priority users:** Executives, engineers, departing employees, or anyone with access to sensitive IP.
- **Priority locations:** Sites storing trade secrets, financial data, or regulated content.

Update the IRM settings with these indicators so alerts focus on what truly matters.

## Step 3 - Run a Proof of Concept
When piloting IRM:
1. Enable **all indicators** (exfiltration, device signals, physical access, and more).
2. Make sure devices are onboarded to Microsoft 365 Purview or Defender to capture device indicators.
3. **Disable anonymization** temporarily so your PoC team can quickly differentiate false positives from genuine incidents.

## Step 4 - Connect HR and Physical Security Data
Use Microsoft Purview connectors to bring in HR events (departures, performance flags) or physical badge data. These feeds help correlate digital signals with real-world behavior, such as data theft right before an exit interview.

## Step 5 - Create Your First Policy
With analytics recommendations in hand:
- Target a **pilot group** or roll out to all users.
- Re-enable **anonymization** before broader investigations to remove bias.
- Iterate quickly; policies can be fine-tuned without redeploying infrastructure.

## Step 6 - Tune Detection Thresholds
Too many false positives? Lower sensitivity. Missing risky behavior? Raise thresholds. Adjustments are iterative; revisit thresholds whenever you add new signals or extend IRM to more teams.

## Layer In Communications Compliance
Microsoft Purview Communications Compliance spots toxic or harmful language in chats, email, and collaboration tools. When those signals feed IRM, you get richer context around risky users, especially when inappropriate communication precedes data loss.

## Final Thoughts
Start early, iterate often, and let analytics guide you. Combining IRM with DLP, Communications Compliance, and HR data delivers a comprehensive insider risk program without bolting together disparate tools.
