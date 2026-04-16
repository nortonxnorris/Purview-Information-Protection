# Strategy for leveraging built-in classifiers

A key decision with classifiers in Microsoft Purview, which type best fits your needs:
Are you looking at pattern or keyword based matches, you would look at Sensitive Info types.
You may be interested in files that match a standard template, consider Document Fingerprinting.
You may be interested in file matches based on machine learning, consider Trainable classifiers.
You may be interested in file matches against a table of data supplied to Purview, consider Exact Data Match

- Let's focus on the **Sensitive Info Types (SIT)** based on patterns and keywords.

## Step 1 - Relevant Classifiers
Microsoft Purview has **300+ built in classifiers** as of today. However, not all classifiers may be relevant to you.
The first step is to understand which classifiers are of relevance to your organization or industry. You may be interested in prioritizing design controls around PII, or let's use another example, where we want to prioritize identifying exposed credentials. We are going with the 'All credentials' SIT.

For reference, here is a list of classifiers and their definitions: https://learn.microsoft.com/en-us/purview/sit-sensitive-information-type-entity-definitions

## Step 2 - Understand your level of tolerance
Next, take into consideration the volume of matches you want to prioritize:
- **Strict:** I am interested in any number of credentials that are externally exposed
- **Volume:** I am interested if we see 10 or more credentials in a file that is externally shared

## Step 3 - Consider first looking at data shared externally
The Data Explorer gives you a count of the matches in files and email. This may be an overwhelming place to start.

Instead, consider first looking for any matches, where this content is shared externally.
We can do this using a DLP policy:
1. Create a new DLP policy scoped to Exchange, SharePoint and OneDrive
2. Design your policy rules looking for 'Content contains: All credentials' and 'Shared with: People outside my organization'
3. From step 2, you may want to adjust the matches: A: Default: 1 to Any, OR B: 10 to Any
4. Do not configure any actions, or user notifications, or incident reports at this time
5. Run the policy in simulation mode
6. DLP **Activity Explorer**: Wait till you start seeing policy matches in the Activity Explorer, and start investigating the policy matches.
This stage is important, as it gives you visibility to potential true matches that should be remediated immediately.

## Step 4 - Built-in versus Custom
Use the built-in classifiers as much as possible, however, if the built-in classifiers do not meet your needs, or if you need to further reduce the matched content, this is where you consider a custom classifier.
Either start from scratch, or, I would receommend creating a copy of a built-in classifier, and customize the copy. (Note: You cannot modify a built-in classifier. There are certain classifiers that cannot be copied)

For reference: https://learn.microsoft.com/en-us/purview/sit-create-a-custom-sensitive-information-type

## Step 5 - Update and iterate
Now that you have visibility into externally shared sensitive content, you could pivot your strategy: If you created a custom SIT in Step 4, update your DLP policy to include the new custom SIT.
You may also want to start a parallel effort looking for sensitive data shared within your tenant.


## Final Thoughts
It is not feasible to validate all built-in classifiers, however, you now have a strategy to prioritize, modify and implement a classifier that best meets your needs.
