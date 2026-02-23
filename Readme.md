- SAR total: Calculate the SAR total by summing every "total" value from all accounts[].unusualTxnsSummary.credit[] and 
  accounts[].unusualTxnsSummary.debit[] arrays across all accounts and all transaction types. 

**IMPORTANT - Credits and/or debits determination** (Perform BEFORE writing conclusion):
  First, check: 
    a) Does the sum of all credit[] values equal zero or are all credit arrays empty/null? 
    b) Does the sum of all debit[] values equal zero or are all debit arrays empty/null?
  
  Then apply this logic:
    - IF credits > 0 AND debits = 0 → Say "derived from credits."
    - IF debits > 0 AND credits = 0 → Say "derived from debits."
    - IF credits > 0 AND debits > 0 → Say "derived from credits and debits."
    - IF both are 0 or empty → Output: "No unusual credit or debit activity identified."
  
  Use ONLY the determined value above. Do NOT say "credits and debits" if only credits or only debits exist.
