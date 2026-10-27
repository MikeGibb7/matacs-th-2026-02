Introduction:

This proposal outlines using AI to extract facts from unstructured paragraph or pdf inputs. The process still uses deterministic rules, computation and ranks.
Errors and edge cases would be flagged and reviewed by a human.
The sample set in ranking.json is based on how they would be presented after the prompt is complete. The number result is an estimation based on how the rankings would translate through the point system. It also displays possible risks and for what we would return null outputs for.

Questions to Answer:

1. AI vs rules.

In my flow the AI tool is simply used for data extraction. The purpose is to simply the proposal text it receives into a JSON data schema. it specifically pulls raw numbers and verifies that the required qualities are present, for example, the Ontario electrical contractor licence being held by the company. It also would check for suspected prompt injection.

plain code or rules are present in a pre scan of the text to prevent obvious injection attempts. It is then also used for mandatory pass or fail fields, for example ensuring project timeline specifications meet requirements. Or project proposals have all required licences. It would also be used to check for null rankings or flagged submissions in order to seperate those and send them on the right path. The final ranking would also be calculated using deterministic code to ensure the ranks are all fair and consistent.

I chose to draw the line at extraction to avoid issues with any subjective interpretation and to minimize the chance of LLM bias affecting the rankings. It also ensures consistency with the calculations and the ranking calculations correctness as the math is performed using code and not an LLM so there is no chance of any arthmetic mistakes. 

2. consistency and explainabilty.

to gaurentee repeatability in our outputs, we would pin the exact model and use temperature: 0, this ensures for the same prompt it will always return the same output. So rerunning the same proposal with the same criteria will always return the same output. 

for explainabilty every field would be paired with a quote from the text. The criteria engine would also save the score breakdown along with a generated summary string stored in reason. If implemented into a UI you could click on a field and see the quote it pulled, the score breakdown and the generated reason all in one place. 

3. Human in the loop.

Humans would be used in the first step to design and update categories and criteria. They would also be used to review flagged submissions and can also review the ineligigle disqualifications. And they would be allowed to override any AI extracted field or score, though these overrides would be logged for review and could be used to improve the system. All changes made by the admin would be logged so we can review the process.
A human would also make the final award decision.

4. Changing criteria

To prevent proposals from being compared on different criterea I believe we should have any updates to criteria update the criteria version number. this should be tracked till the end and then the proposals should be ranked against each other using the same criteria version number. I believe that any proposals which had been ranked by version 1 of the criteria should be re ranked by version 2 of the criteria for the most fair comparison. Although, if some proposals were ineligible originally and the criteria for their cretain eligibilty failures did not change, we could keep those ineligible to prevent re running these and using more resources for this. But overall I believe re running the same proposals with the updated criteria would be the most fair and consistent approach.

5. Risks.

Proposals may have unusual layouts, multi-column tables, or ambiguous phrasing (such as Proposal B deferring its electrical licence to an unnamed subcontractor, or Proposal C citing installs "since 2020" which stretches the 5-year window). 
Use a layout-aware parser (e.g., converting PDF tables to Markdown before LLM ingestion) and break complex criteria into granular schema fields (`met_directly_by_bidder`, `earliest_install_year_stated`). In the UI, pair every extracted value with its verbatim quote highlighted directly in the source PDF so a human reviewer can quickly spot-check errors.

Prompt injection: Proposals may attempt to override the system's instructions or manipulate the output. In addition to our regex pre-scan and having the AI engine flag suspicious instructions (`prompt_injection_detected: true`), we sandbox the proposal text inside `<proposal_text>` XML tags treated strictly as untrusted data. Most importantly, because the LLM only extracts facts and plain code calculates all scores and ranks, an injection attack cannot directly alter the final ranking math. Flagged proposals are halted and routed to human security review.

Hallucinations: When a proposal omits or defers critical information (such as Proposal D stating its timeline will be "determined after a detailed site assessment"), an LLM might guess or fabricate a number to satisfy the prompt. We enforce strict JSON schema validation where all metric fields are explicitly nullable (`number | null`), paired with system instructions requiring `null` whenever a fact is missing or deferred. We require a `verbatim_quote` for every extracted metric and run a deterministic substring check in code to confirm the quote actually exists in the raw PDF text; any proposal with a `null` required field is automatically set to `flagged` for human follow-up.

Bias: An LLM could favor well-known company names, persuasive marketing language, or specific writing styles over actual project qualifications. We could redact or mask vendor names and branding before sending text to the LLM, and restrict the AI to extracting objective, verifiable facts (counts, dollar amounts, weeks, and boolean checklist items) rather than asking it for holistic subjective ratings.

Costs: Uploading massive PDFs, duplicate documents, or repeatedly tweaking admin weights could trigger expensive, redundant LLM API calls. Set page and file-size limits on uploads, and save (cache) the AI's extracted data in the database the first time a PDF is read. If an admin changes the scoring weights later, the system simply re-calculates the math using the saved data instead of paying to run the AI all over again.