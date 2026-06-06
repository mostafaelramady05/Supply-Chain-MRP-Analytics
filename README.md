# 🏭 Integrated MRP & Production Tracking Dashboard

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Advanced-blue?style=for-the-badge)
![Data Modeling](https://img.shields.io/badge/Data_Modeling-Star_Schema-success?style=for-the-badge)

## 📌 Project Overview
This repository showcases an advanced Power BI solution designed to streamline manufacturing operations and bridge the gap between production planning and inventory management. 

By engineering a robust Data Model and utilizing advanced DAX, this dashboard acts as a mini-MRP (Material Requirements Planning) system. It performs dynamic Bill of Materials (BOM) explosion to translate Finished Goods (FG) production into precise Raw Material requirements, instantly highlighting stock shortages and production variances with a modern, custom-built SVG interface.

---

## 🎯 Business Problem & Solution
* **The Challenge:** Manufacturing data is often siloed. Comparing the Finished Goods "Production Plan" directly with Raw Material "Stock on Hand" is nearly impossible without duplicating numbers due to Many-to-Many relationships and varying dosages.
* **The Solution:** Built a robust Star Schema data model incorporating a Bridge Table for the BOM. This allows Power BI to dynamically allocate raw material consumption based on actual FG production rates and waste percentages.

---

## 🧠 Data Modeling Architecture

*(Insert your Data Model Image here by removing the comment tags below and uploading the image to the Images folder)*
**Key Modeling Highlights:**
* **Dimensions:** `Items_D` (Finished Goods) and `Materials_D` (Raw Materials).
* **Facts:** Separated Execution (`Actual FG`), Planning (`Plan_F`), and Inventory (`Material_Inv_F`).
* **The Bridge:** `BOM_F` acts as the critical bridge resolving the granularity mismatch between FG produced and RM consumed.

<details>
<summary><b>🔍 Click here to view the Detailed Table Relationships (14 Active Links)</b></summary>

| Child Table (Fact / Bridge) | Foreign Key | Direction | Parent Table (Dimension) | Primary Key |
| :--- | :--- | :---: | :--- | :--- |
| `Actual FG` | `Code` | ⬅️ | `Items_D` | `Item` |
| `Actual FG` | `Date` | ⬅️ | `Calendar` | `Date` |
| `BOM_F` | `Item` | ⬅️ | `Items_D` | `Item` |
| `BOM_F` | `Material Code` | ⬅️ | `Materials_D` | `Material Code` |
| `Plan_F` | `Code` | ⬅️ | `Items_D` | `Item` |
| `Plan_F` | `Date` | ⬅️ | `Calendar` | `Date` |
| `Plan_Material` | `Code` | ⬅️ | `Items_D` | `Item` |
| `Plan_Material` | `Material Code` | ⬅️ | `Materials_D` | `Material Code` |
| `Safety_Stock` | `Material Code` | ⬅️ | `Demand_Material` | `Material Code` |
| `schedule_plan_material` | `Material Code` | ⬅️ | `Demand_Material` | `Material Code` |
| `Demand_FG` | `Code` | ↔️ | `Items_D` | `Item` |
| `FG_Inv_F` | `Row Labels` | ↔️ | `Items_D` | `Item` |
| `Demand_Material` | `Material Code` | ↔️ | `Materials_D` | `Material Code` |
| `Material_Inv_F` | `Code` | ↔️ | `Materials_D` | `Material Code` |

> *Note: ⬅️ indicates a Many-to-One relationship, while ↔️ indicates a One-to-One or Bi-directional filtering setup optimized for specific DAX measures.*
</details>

---

## 💻 Advanced DAX Highlights

### 1. Dynamic BOM Explosion (Preventing Multiplication Explosion)
To calculate the exact Raw Material equivalent of the produced Finished Goods without duplicating numbers, we iterate over the BOM table directly:

```dax
Actual Material Production = 
SUMX(
    'BOM_F', 
    VAR _ItemCode = 'BOM_F'[Item] 
    VAR _ActualFG = 
        CALCULATE(
            SUM('Actual FG'[Quantity]), 
            'Actual FG'[Code] = _ItemCode 
        )
    RETURN _ActualFG * 'BOM_F'[Dosage incl. waste %]
)
