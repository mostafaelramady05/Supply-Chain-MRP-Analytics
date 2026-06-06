# 🔄 Total Net Material Requirement (Iterator)

**Description:**
A classic DAX pattern used to calculate accurate grand totals in matrices. By iterating over the `Materials_D` table using `SUMX`, this measure evaluates the shortage at the granular material level. It strictly returns shortages (`> 0`) and ignores surpluses, ensuring that a surplus in one material does not artificially offset the shortage of another in the Grand Total row.

```dax
Total Material Requirement = 
SUMX(
    // Iterate row by row over the unique materials
    VALUES('Materials_D'[Material Code]),
    
    // Evaluate demand and stock within the current row context
    VAR MatDemand = [Material Demand for Selected Item]
    VAR MatStock = CALCULATE(SUM('Material_Inv_F'[Quantity]))
    VAR Shortage = MatDemand - MatStock
    
    // Ignore excess stock for requirement planning
    RETURN 
        IF(Shortage > 0, Shortage, 0)
)
