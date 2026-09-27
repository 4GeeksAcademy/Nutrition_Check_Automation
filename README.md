# Workflow README



## 1\. Purpose



SnackCheck is an n8n automation that accepts a packaged food's barcode, retrieves real product information from Open Food Facts, evaluates nutrition using the supplied classification rules, and returns a friendly plain-text verdict.

The verdict includes an emoji headline, key nutrition numbers with red/amber/green markers, and a short AI-written explanation. The AI is used for wording only; the verdict is calculated by deterministic workflow logic.





## 2\. How It Works



1. **Receive input:** A Webhook accepts a `POST` request containing a barcode.
2. **Normalize and validate:** Extract the barcode as a string and check that it is non-empty and numeric.
3. **Fetch product:** Call the Open Food Facts API for the product's name, brand, Nutri-Score, and nutriments.
4. **Check API result:** Route unknown products to a not-found response.
5. **Prepare fields:** Extract values per 100 g, including energy using bracket notation for `energy-kcal\\\_100g`.
6. **Classify:** Apply the sugar, salt, and fat thresholds; count high nutrients; map Nutri-Score to a verdict level; choose the worse level.
7. **Check data sufficiency:** If required nutrient values are missing, return an insufficient-data message instead of a classification.
8. **Generate explanation:** Ask the Groq-backed AI chain for a short paragraph consistent with the computed verdict.
9. **Format response:** Assemble a plain-text message with an emoji headline, nutrition lines and markers, and the AI paragraph.
10. **Respond:** Return the text to the caller.





### Classification rules

Nutrient   Rule

\---

Sugar      Low <5 g; medium 5--22.5 g inclusive; high >22.5 g
Salt       High >1.5 g
Fat        High >17.5 g

&#x20;   High nutrient count Traffic-light level

\---

&#x20;                     0 Healthy
1 Moderate
2--3 Unhealthy



Nutri-Score       Level

\---

A or B            Healthy
C                 Moderate
D or E            Unhealthy
Unknown/missing   Fall back to traffic-light level

**Overall verdict:** The worse of the traffic-light level and Nutri-Score level. The `concern\\\_score` is the number of high nutrients
(0--3) and is not itself the verdict.





## 3\. Setup





### Requirements



* An n8n workspace.
* A Groq API key and configured Groq credential for the AI model.
* Network access to `world.openfoodfacts.org`.
* No Open Food Facts API key is required.



### Configuration



1. Create a new workflow in n8n.
2. Add the nodes and connections described in the inline node documentation.
3. Configure the Webhook as `POST`, with a path such as `/snackcheck`, and set it to respond using a Respond to Webhook node.
4. Configure the HTTP Request node to use the barcode in the API URL and request `product\\\_name,brands,nutriscore\\\_grade,nutriments`.
5. Configure HTTP error handling so 404 responses can be inspected and routed.
6. Add the classification Code node and the sufficient-data branch.
7. Configure the Basic LLM Chain and connect the Groq Chat Model credential.
8. Assemble the AI paragraph and calculated fields into a single plain-text message.
9. Configure every response node with the correct plain-text response bdy and status code.
10. Test each branch before activating the workflow.



### Suggested response status codes



* Success: `200`
* Missing/malformed barcode: `400`
* Product not found: `404`
* Insufficient nutrition data: `422`
* Unexpected internal failure: `500` (handled through an error workflow or equivalent fallback)



## 4\. Usage



Send a `POST` request to the active webhook URL with a JSON body:``` json { "barcode": "3017620422003"} ```

Example using Hopscotch:


   POST "https://YOUR\\\_N8N\\\_HOST/webhook-test/snackcheck" \\\\
   "Content-Type: application/json" \\\\
   Raw Request Body'{"barcode":"3017620422003"}'


The response is **plain text**, not JSON. Example format:

``` text
🔴 UNHEALTHY — SnackCheck Verdict

📊 NUTRITION (per 100 g)
🔴 Sugar: 56.3 g
🟢 Salt: 0.107 g
🔴 Fat: 30.9 g
⚡ Energy: 539 kcal
🏷️ Nutri-Score: E
🚦 Concern score: 2/3

💬 OUR VERDICT
This product is best enjoyed as an occasional treat. It is high in sugar and fat, so consider balancing it with other everyday choices.
```

Example values and AI wording are illustrative; live API data and
generated text may differ.



## 5\. Error Handling



\---

Condition               Handling                Expected result

\---

Empty barcode           Validation branch       Friendly usage message;
no API call



Non-numeric barcode     Validation branch       Invalid-input message;
no API call



Product absent from     API response check      Product-not-found message;
database      

&#x20;                                                   

Missing required        Data sufficiency branch  Insufficient-data message; no verdict
nutrient data   

&#x20;                               

Nutri-Score             Classification fallback Traffic-light verdict
missing/unknown,  
nutrients present



API/network failure     HTTP error handling /   Clear service-error
error workflow          response; no unhandled
crash





AI provider failure     Error handling or       Return a controlled
fallback message        message or
deterministic verdict
with a clear note; do
not fabricate AI text





All error paths should return plain text and should not expose API
credentials, stack traces, or internal implementation details.



## 6\. Limitations



* Open Food Facts coverage and completeness vary by product and
region; some entries may be outdated or incomplete.
* The thresholds reproduce the project brief and are not a medical
diagnosis or individualized dietary recommendation.
* The traffic-light calculation uses values per 100 g as specified.
Products commonly consumed by volume may require interpretation
beyond this simplified model.
* Nutri-Score is taken directly from the API and may reflect the
database's available version or data.
* Missing nutrient values are treated as insufficient data under this
workflow's rule; the workflow does not estimate missing values.
* AI-generated wording can vary. The numeric verdict must remain
determined by the Code node, not the AI.
* External API availability, rate limits, and network conditions may
affect response time.
* The workflow is not a substitute for professional nutrition or
medical advice.



