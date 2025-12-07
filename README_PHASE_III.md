# Phase III: Logical Database Design

## Student: Dorothy
## ID: 27129
## Date: November 2025

## Overview
This phase involves designing the logical data model for the Smart Inventory Monitoring System (SIMS) with a focus on normalization, data integrity, and Business Intelligence readiness.

## Files:
1. **mon_27129_Dorothy_SIMRS_ER_Diagram.png** - Entity-Relationship Diagram (visual data model)
<img width="1540" height="1732" alt="Database ER diagram (crow&#39;s foot)" src="https://github.com/user-attachments/assets/3aa1c460-b812-446d-a846-718846d7b6ad" />

2. **mon_27129_Dorothy_SIMRS_Data_Dictionary.md** - Complete data dictionary with specifications

## Database Schema Design:

### Core Entities:
1. **SUPPLIERS** - Master list of product suppliers
2. **PRODUCTS** - Inventory items with stock tracking
3. **STOCK_ENTRIES** - Transaction log for stock movements
4. **REORDER_LOG** - Automated reorder alert system
5. **AUDIT_LOG** - Security and compliance audit trail

### Normalization Achieved:
- **1NF**: Eliminated repeating groups (all atomic values)
- **2NF**: Eliminated partial dependencies (all non-key attributes depend on full PK)
- **3NF**: Eliminated transitive dependencies (no attributes depend on other non-key attributes)

## Key Design Decisions:

### 1. Primary Keys:
- All tables use `NUMBER` data type for primary keys
- Sequences provide auto-increment functionality
- Naming convention: `[table_name]_seq`

### 2. Foreign Key Relationships:
- `PRODUCTS.supplier_id` → `SUPPLIERS.supplier_id`
- `STOCK_ENTRIES.product_id` → `PRODUCTS.product_id`
- `REORDER_LOG.product_id` → `PRODUCTS.product_id`

### 3. Business Rules Enforced:
- `min_quantity > 0` (Products must have minimum stock level)
- `current_quantity >= 0` (Stock cannot be negative)
- `unit_price >= 0` (Prices cannot be negative)
- `transaction_type IN ('IN', 'OUT')` (Valid stock movements only)
- `status IN ('PENDING', 'ORDERED', 'IGNORED')` (Valid reorder statuses)

## BI Considerations:

### Fact Tables:
- **STOCK_ENTRIES**: Grain = per transaction (for inventory turnover analysis)
- **REORDER_LOG**: Grain = per alert (for reorder pattern analysis)

### Dimension Tables:
- **PRODUCTS**: Product master data
- **SUPPLIERS**: Supplier master data
- **Time Dimension**: Derived from transaction_date and alert_date

### Slowly Changing Dimensions (SCD):
- `PRODUCTS.unit_price` and `PRODUCTS.min_quantity` treated as Type 1 (overwrite current)
- Historical tracking possible through `STOCK_ENTRIES` transaction history

### Aggregation Levels:
- Daily stock levels by product/category
- Monthly supplier performance metrics
- Quarterly inventory turnover rates

## Audit & Compliance:
- **AUDIT_LOG** table designed for Phase VII business rule enforcement
- Captures: table_name, operation, user_name, timestamp, status, error messages
- Supports regulatory compliance and security monitoring

## Assumptions:
1. Single business entity (no store_id or branch_id required)
2. `current_quantity` is a derived attribute maintained by triggers/procedures
3. Date fields include timestamp for precise auditing
4. System designed for internal business use (not customer-facing)

## Tools Used:
- **Draw.io**: For ER diagram creation
- **Markdown**: For documentation
- **Oracle SQL Developer**: For design validation

## Validation:
- All entities map to business requirements
- Cardinalities correctly represent business relationships
- Data types appropriate for African business context (prices in local currency, realistic quantities)
- Constraints prevent invalid data entry

---

## ER Diagram Legend:
- **Rectangle**: Entity
- **Diamond**: Relationship
- **Arrow**: Direction of relationship
- **1:M**: One-to-Many relationship
- **PK**: Primary Key
- **FK**: Foreign Key

## Next Steps:
This logical design serves as blueprint for:
1. Physical database implementation (Phase IV)
2. Table creation scripts (Phase V)
3. PL/SQL development (Phase VI)
4. Business rule implementation (Phase VII)
