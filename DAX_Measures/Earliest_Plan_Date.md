# 📅 Earliest Plan Date

**Description:**
This measure calculates the earliest required date for a specific raw material. Since raw materials and production plans often reside in different fact tables with differing granularities, this DAX pattern uses `TREATAS` to create a virtual relationship. It retrieves the list of Finished Goods that use the selected material from the BOM table, and passes that list as a direct filter to the Production Plan table to extract the earliest date.

```dax
Earliest Plan Date = 
// 1. Retrieve a list of all Finished Goods (Items) in the BOM that use the selected material
VAR _RelatedItems = 
    CALCULATETABLE(
        VALUES('BOM_F'[Item])
    )

// 2. Pass this list as a virtual filter to the Plan table to find the minimum date
VAR _MinDate = 
    CALCULATE(
        MIN('Plan_F'[Date]),
        TREATAS(_RelatedItems, 'Plan_F'[Code]) 
    )

// 3. Return the result, handling blank values cleanly
RETURN
    IF(ISBLANK(_MinDate), BLANK(), _MinDate)
