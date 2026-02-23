"Generate a complete SAR recommendation narrative using the following data:
```json
{{input_data}}
```

**ANTI-HALLUCINATION RULES - READ FIRST:**

1. EXTRACTION VERIFICATION - Before using any data:
   - Check if the field exists in the JSON
   - Check if the field is not null
   - Check if the field is not empty array []
   - Only extract if ALL three conditions are true
   - If ANY condition fails, SKIP that field entirely - do NOT infer, assume, or fabricate a replacement

2. EXACT MATCHING ONLY:
   - Extract ONLY the exact values present in the JSON
   - Do NOT interpret, rephrase, or add context to values
   - Do NOT combine fields to create new information
   - Do NOT use domain knowledge to fill gaps
   - Do NOT assume field values based on other fields

3. NULL/EMPTY HANDLING:
   - null = field does not exist or is explicitly null → SKIP
   - [] = empty array → SKIP
   - "" = empty string → SKIP
   - undefined → SKIP
   - 0 = valid value, use it
   - false = valid value, use it

4. NO INFERENCE ALLOWED:
   - Do NOT infer account types if accountType field is missing
   - Do NOT infer customer names if subjects[].name is missing
   - Do NOT infer dates if date fields are missing
   - Do NOT create narratives for missing sections
   - Do NOT fill gaps with assumptions

---

**CALCULATION SAFEGUARDS - CRITICAL:**

5. BEFORE ANY CALCULATION:
   - List all source values explicitly
   - Verify each value exists and is numeric
   - Count the number of items
   - Show the calculation step-by-step
   - Perform the calculation twice to verify

6. CREDIT/DEBIT CALCULATION (Apply for BOTH Section 6.3 AND Section 4):
   
   STEP 1 - EXTRACT AND VERIFY CREDIT VALUES:
   - Search for: accounts[].unusualTxnsSummary.credit[] array
   - IF field exists AND is not empty AND contains items:
     - List each item's .total value: [value1, value2, value3, ...]
     - Count items: X credit transactions
     - Sum calculation: value1 + value2 + value3 = SUM_CREDITS
     - Verify sum is numeric and > 0
   - IF field missing OR empty OR null:
     - Set: SUM_CREDITS = 0
     - Do NOT search other arrays for credit values
   
   STEP 2 - EXTRACT AND VERIFY DEBIT VALUES:
   - Search for: accounts[].unusualTxnsSummary.debit[] array
   - IF field exists AND is not empty AND contains items:
     - List each item's .total value: [value1, value2, value3, ...]
     - Count items: Y debit transactions
     - Sum calculation: value1 + value2 + value3 = SUM_DEBITS
     - Verify sum is numeric and > 0
   - IF field missing OR empty OR null:
     - Set: SUM_DEBITS = 0
     - Do NOT search other arrays for debit values
   
   STEP 3 - VERIFY NO CROSS-CONTAMINATION:
   - Confirm SUM_CREDITS is ONLY from unusualTxnsSummary.credit[]
   - Confirm SUM_DEBITS is ONLY from unusualTxnsSummary.debit[]
   - Check: notableActivity is NOT included in either sum
   - If contamination detected, RECALCULATE using correct arrays only
   
   STEP 4 - APPLY DETERMINATION LOGIC:
   - Compare SUM_CREDITS and SUM_DEBITS using this EXACT logic:
   
     IF (SUM_CREDITS > 0) AND (SUM_DEBITS = 0):
       → OUTPUT PHRASE = "derived from credits"
       → STOP - do not evaluate other conditions
     
     ELSE IF (SUM_DEBITS > 0) AND (SUM_CREDITS = 0):
       → OUTPUT PHRASE = "derived from debits"
       → STOP - do not evaluate other conditions
     
     ELSE IF (SUM_CREDITS > 0) AND (SUM_DEBITS > 0):
       → OUTPUT PHRASE = "derived from credits and debits"
       → STOP - do not evaluate other conditions
     
     ELSE IF (SUM_CREDITS = 0) AND (SUM_DEBITS = 0):
       → OUTPUT PHRASE = "with no identified credit or debit activity"
       → STOP
   
   STEP 5 - FORBIDDEN OUTPUTS:
   - If only SUM_CREDITS > 0: Do NOT say "derived from credits and debits" ❌
   - If only SUM_DEBITS > 0: Do NOT say "derived from credits and debits" ❌
   - If both are 0: Do NOT say "derived from credits" or "derived from debits" ❌
   - ONLY use the phrase determined in STEP 4

7. TRANSACTION COUNT CALCULATION:
   - Count = length of (unusualTxnsSummary.credit[] array) + length of (unusualTxnsSummary.debit[] array)
   - Example: If credit[] has 3 items and debit[] has 2 items → Count = 3 + 2 = 5 transactions
   - Do NOT count notableActivity items
   - Do NOT estimate - count actual array items only

8. TOTAL AMOUNT CALCULATION:
   - Create three separate sums:
     a) Sum of all unusualTxnsSummary.credit[].total values = CREDIT_TOTAL
     b) Sum of all unusualTxnsSummary.debit[].total values = DEBIT_TOTAL
     c) SAR_TOTAL = CREDIT_TOTAL + DEBIT_TOTAL
   - Format as: $X,XXX.XX with commas
   - Do NOT include notableActivity in totals
   - Verify sum is greater than 0

9. DATE RANGE CALCULATION:
   - Search all unusualTxnsSummary.credit[] and debit[] items
   - Find earliest minTransactionDate value → START_DATE
   - Find latest maxTransactionDate value → END_DATE
   - Format as: MM/DD/YYYY - MM/DD/YYYY
   - Do NOT use dates from other sections
   - If dates missing, do NOT infer date range

10. PRIOR SAR EXTRACTION:
    - Check: Does priorSARs array exist?
    - Check: Is priorSARs not null?
    - Check: Is priorSARs not empty []?
    - IF all three checks pass:
      - For EACH item in priorSARs array:
        - Extract: id (exact value)
        - Extract: filingDate (format as MM/DD/YYYY)
        - Extract: amountReported (format as number)
        - Extract: subject[].name (list all names)
        - Extract: reason (exact text)
        - Output: "There was a prior SAR (ID: {id}) filed on {filingDate} for {amountReported} involving subjects {names}. {reason}."
    - IF any check fails:
      - OMIT entire prior SAR section - output NOTHING

11. NOTABLE ACTIVITY EXTRACTION (SEPARATE FROM UNUSUAL ACTIVITY):
    - Check: Does notableActivity object exist?
    - Check: Is notableActivity.credit[] array not empty?
    - Check: Is notableActivity.debit[] array not empty?
    - IF notableActivity exists AND (credit[] or debit[] has items):
      - Extract from notableActivity ONLY (NOT unusualTxnsSummary)
      - List each transaction type and customLanguage
      - Calculate totals from notableActivity.credit[] + notableActivity.debit[] only
      - Output: "Other non-unusual activity identified..."
    - IF notableActivity is missing OR both credit[] and debit[] are empty:
      - OMIT entire section - output NOTHING

---

**Instructions:**

1. Use ONLY the provided JSON data. Do not infer, fabricate, or assume any values.

2. DATA SOURCE ISOLATION RULES - CRITICAL:
   - FOR CREDIT/DEBIT DETERMINATION: Use ONLY accounts[].unusualTxnsSummary.credit[] and accounts[].unusualTxnsSummary.debit[]
   - FOR NOTABLE ACTIVITY SECTION: Use ONLY accounts[].notableActivity - DO NOT include unusualTxnsSummary data
   - FOR UNUSUAL ACTIVITY SECTION: Use ONLY accounts[].unusualTxnsSummary - DO NOT include notableActivity data
   - DO NOT MIX data between unusualTxnsSummary and notableActivity

3. Section order: Alerting Activity / Reason for Review, Scope of Review, Summary of the Investigation, Conclusion (with blank lines between sections).

4. SECTION 1: Alerting Activity / Reason for Review
   - Verify alertingAccounts[] exists and is not empty BEFORE extracting
   - Output: "[itemId]: [Account Type(s) and Number(s)] alerted in [Alerting Month(s)] for [alerting reason]."
   - Account Type(s) and Number(s): Extract accountType and accountNumber from alertingAccounts[] - use exact values only
   - Alerting Month(s): Extract from alertingDates[] - convert to "Month Year" format - use EARLIEST date as reference
   - Alerting Reason: Extract customLanguage from all alertingAccounts[].alertingReason entries. Combine and deduplicate unique values only.
   - Prior SARs: Follow rule #10 above - verify existence before extracting
   - If priorSARs[] missing or empty, OMIT paragraph entirely - output NOTHING

5. SECTION 2: Scope of Review
   - Verify scopeOfReview object exists and contains fromDate and toDate BEFORE extracting
   - Output: "A review of account(s) [account numbers] was conducted from [fromDate] to [toDate]."
   - Format dates as MM/DD/YYYY - MM/DD/YYYY
   - If scopeOfReview missing or dates missing, OMIT section

6. SECTION 3: Summary of the Investigation (Red Flags, Supporting Evidence, etc.)

   6.1 Who:
   - Verify accounts[] array exists and is not empty BEFORE extracting
   - Verify accounts[].type exists and is not null/empty
   - Verify accounts[].subjects[] array exists and is not empty
   - Output: "[Account type(s) and numbers] owned by [account signer(s)] was opened on [account opening date] in [location account opened]. Per bank records, [customer name] has been a USB customer since [customer since date] and has a listed occupation in [occupation] with employer [employer name]."
   - Extract exact values only - do NOT infer missing occupation or employer
   - If any required field missing: OMIT that field from output
   - If all fields missing: OMIT entire Who section

   6.2 When/What: (CRITICAL - Use ONLY unusualTxnsSummary - DO NOT reference notableActivity)
   - Verify unusualTxnsSummary exists before extracting
   - Verify both credit[] and debit[] arrays exist (can be empty but must exist)
   - Follow calculation rules #5-9 above
   - Output: "A review of account(s) [account number(s)] identified [number] unusual activity transaction(s). The unusual activity includes [transaction types with custom language], totaling [total amount] occurring between [date range]."
   - Use exact transaction count from arrays
   - Use calculated totals from rules #8
   - Use date range from rule #9

   6.3 Unusual Activity Detail: (CRITICAL - Follow calculation rules #6)
   - Execute STEP 1-5 of rule #6 above
   - Do NOT deviate from the logic
   - Output the determined phrase ONLY
   - Output: "The [customLanguage] transactions were [derived from credits/debits/credits and debits] totaling $[amount], with transactions occurring between [date range]."

   6.4 Notable Activity: (Follow rule #11 above)
   - Verify notableActivity exists before extracting
   - Verify credit[] or debit[] has items
   - If checks pass: Output section
   - If checks fail: OMIT entire section - output NOTHING

   6.5 Where/How:
   - Verify location or entity data exists in unusualTxnsSummary
   - For CASH/CHECK: Use locations from transaction data (exact values only)
   - For WIRE/ACH/PAP/RTP/POS: Use entity names (exact values only)
   - If no locations or entities exist: OMIT this section - output NOTHING

   6.6 Why:
   - Verify analysis[] array exists and is not empty
   - For each item: Extract factor and description (exact values)
   - If analysis[] empty or missing: Output "The activity is unusual based on the patterns identified during review."

7. SECTION 4: Conclusion
   - Re-execute STEP 1-5 of rule #6 (recalculate sums independently)
   - Verify sums match previous calculation - if not, recalculate again
   - Use determined phrase from recalculation
   - Output: "In conclusion, a SAR is recommended to report unusual [unusual activity transaction type(s)] activity involving [account type(s)] [account number(s)] and subject [subject name(s)]. The unusual activity totaled $[SAR total] [credits_debits_phrase] occurring between [date range] with [number] transaction(s) from [transaction types]."

8. DO NOT fabricate, infer, or hallucinate any data. ONLY use values present in the input JSON.

9. Optional field handling: 
   - If any field is missing, null, or empty: VERIFY before using
   - If verification fails: OMIT that section entirely
   - Never output placeholder text like "N/A", "[value]", "Not provided"

10. Lists: Join multiple items with commas and 'and' before final item.

11. Currency formatting: $X,XXX.XX with commas and decimals.

12. Date formatting: MM/DD/YYYY format.

13. Never output JSON structures, field names, technical terms, or variable placeholders in the narrative. Use only human-readable language.

14. FINAL VALIDATION:
    - Do NOT output "credits and debits" if only credits exist
    - Do NOT output "credits and debits" if only debits exist
    - Do NOT mix unusual and notable activity
    - Do NOT invent missing data
    - Do NOT output instructions in narrative
