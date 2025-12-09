# SIMRS_PLSQL_Capstone
PL/SQL Capstone Project - Smart Inventory Monitoring system

# **Phase III: Logical Database Design**
 **Overview**
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


# SIMS_PLSQL_Capstone
PL/SQL Capstone Project - Smart Inventory Monitoring &amp; Reordering System.

#  PHASE_IV
# Phase IV: Database Creation - SIMS Project

## Overview
This phase involves the creation of the Oracle pluggable database (PDB) and its core configuration for the Smart Inventory Monitoring & Reordering System (SIMRS).

## Database Details
- **PDB Name:** `
GRPA_27129_DOROTHY_SIMS_DB`
- **Admin User:** `SIMS_admin` (Password: Dorothy) - Has DBA role within the PDB.
- **Application User:** `SIMS_user` (Password: inventory123) - Used for all table creation and PL/SQL development.

## Configuration Summary
1.  **Tablespaces Created:**
    - `SIMS_DATA`: For table data (200MB, auto-extensible)
CREATE TABLESPACE SIMS_data 
DATAFILE 'SIMS_data01.dbf' SIZE 100M AUTOEXTEND ON NEXT 50M MAXSIZE 1G;

    - `SIMRS_IDX`: For indexes (100MB, auto-extensible).
    - CREATE TABLESPACE SIMS_index
 DATAFILE 'SIMS_index01.dbf' SIZE 50M AUTOEXTEND ON NEXT 25M MAXSIZE 500M;

    - `SIMRS_TEMP`: Temporary tablespace for sorting (100MB, auto-extensible).
    CREATE TEMPORARY TABLESPACE SIMS_temp
 TEMPFILE 'SIMS_temp01.dbf' SIZE 50M AUTOEXTEND ON;
  
3.  **Privileges:** The application user (`SIMS_user`) has all necessary object creation privileges.
CREATE USER SIMS_user IDENTIFIED BY Dorothy
DEFAULT TABLESPACE SIMS_data
TEMPORARY TABLESPACE SIMS_temp
QUOTA UNLIMITED ON SIMS_data
QUOTA UNLIMITED ON SIMS_index;

'GRANT CONNECT, RESOURCE TO SIMS_user;
GRANT CREATE SESSION, CREATE VIEW, CREATE SEQUENCE TO SIMS_user;
GRANT UNLIMITED TABLESPACE TO SIMS_user;


## Execution Instructions
1.  **Prerequisites:** Oracle Database 12c/19c/21c with multitenant architecture. Access as a privileged user (SYS, SYSTEM, or a user with `CREATE PLUGGABLE DATABASE` privilege).
<img width="734" height="189" alt="GRANTING PREVILEGE TO THE USER" src="https://github.com/user-attachments/assets/87e01855-e4c8-4e1e-928a-39daa093c030" />

2.  **Step 1:** Run the `mon_27129_Dorothy_SIMRS_Database_Creation.sql` script in SQL*Plus or SQL Developer connected to the root container (CDB).
<img width="832" height="97" alt="shifting to the new pdb" src="https://github.com/user-attachments/assets/3ba2cd08-039c-43a4-8f2a-3b3a3b93bc3a" />

3.  **Step 2:** Verify the output. The script auto-switches to the new PDB and creates the tablespaces and users.
<img width="832" height="97" alt="shifting to the new pdb" src="https://github.com/user-attachments/assets/3ba2cd08-039c-43a4-8f2a-3b3a3b93bc3a" />
<img width="782" height="177" alt="pdb creation" src="https://github.com/user-attachments/assets/035b47a6-f592-49c5-b51b-de16de1dd1ae" />
<img width="686" height="319" alt="SHOWING PDBS AND CURRENT CONTAINER" src="https://github.com/user-attachments/assets/efb0bcaa-e320-4922-bdb1-62cdd0d20ccc" />
<img width="1017" height="205" alt="INDEX TABLESPACE CREATION" src="https://github.com/user-attachments/assets/cf72ee97-b64a-4b02-86c6-2042f99c3314" />
<img width="523" height="178" alt="USER CREATION" src="https://github.com/user-attachments/assets/2f485f2c-9ce2-47d3-b3b2-821e851c8b0e" />

# Phase V: Table Implementation & Data Insertion

## Overview
This phase implements the physical database structure with realistic test data for the Smart Inventory Monitoring System (SIMS).

## Files Created:
1. **GRPA_27129_Dorothy_SIMS_CREATE_TABLES.sql** - Creates all 5 tables with constraints, indexes, and sequences 
-- 1. SUPPLIERS TABLE
CREATE TABLE suppliers (
    supplier_id   NUMBER DEFAULT suppliers_seq.NEXTVAL PRIMARY KEY,
    supplier_name VARCHAR2(100) NOT NULL,
    contact_phone VARCHAR2(20),
    email         VARCHAR2(100),
    address       VARCHAR2(200)
);

-- Index for suppliers
CREATE INDEX idx_suppliers_name ON suppliers(supplier_name);

-- 2. PRODUCTS TABLE
CREATE TABLE products (
    product_id        NUMBER DEFAULT products_seq.NEXTVAL PRIMARY KEY,
    product_name      VARCHAR2(100) NOT NULL,
    category          VARCHAR2(50),
    supplier_id       NUMBER NOT NULL,
    min_quantity      NUMBER NOT NULL CHECK (min_quantity > 0),
    current_quantity  NUMBER DEFAULT 0 CHECK (current_quantity >= 0),
    unit_price        NUMBER(10,2) DEFAULT 0 CHECK (unit_price >= 0),
    CONSTRAINT fk_product_supplier 
        FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);

-- Indexes for products
CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_supplier ON products(supplier_id);

-- 3. STOCK_ENTRIES TABLE
CREATE TABLE stock_entries (
    stock_id           NUMBER DEFAULT stock_entries_seq.NEXTVAL PRIMARY KEY,
    product_id         NUMBER NOT NULL,
    transaction_type   VARCHAR2(10) NOT NULL CHECK (transaction_type IN ('IN', 'OUT')),
    quantity           NUMBER NOT NULL CHECK (quantity > 0),
    transaction_date   DATE DEFAULT SYSDATE,
    performed_by_user  VARCHAR2(50) DEFAULT USER NOT NULL,
    CONSTRAINT fk_stock_product 
        FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Indexes for stock_entries
CREATE INDEX idx_stock_product ON stock_entries(product_id);
CREATE INDEX idx_stock_date ON stock_entries(transaction_date);

-- 4. REORDER_LOG TABLE
CREATE TABLE reorder_log (
    reorder_id                 NUMBER DEFAULT reorder_log_seq.NEXTVAL PRIMARY KEY,
    product_id                 NUMBER NOT NULL,
    current_quantity           NUMBER NOT NULL,
    min_quantity               NUMBER NOT NULL,
    suggested_reorder_quantity NUMBER NOT NULL CHECK (suggested_reorder_quantity > 0),
    alert_date                 DATE DEFAULT SYSDATE,
    status                     VARCHAR2(20) DEFAULT 'PENDING' CHECK (status IN ('PENDING', 'ORDERED', 'IGNORED')),
    CONSTRAINT fk_reorder_product 
        FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Indexes for reorder_log
CREATE INDEX idx_reorder_product ON reorder_log(product_id);
CREATE INDEX idx_reorder_status ON reorder_log(status);
CREATE INDEX idx_reorder_date ON reorder_log(alert_date);

-- 5. AUDIT_LOG TABLE (For Phase VII business rules)
CREATE TABLE audit_log (
    audit_id        NUMBER DEFAULT audit_log_seq.NEXTVAL PRIMARY KEY,
    table_name      VARCHAR2(50) NOT NULL,
    operation       VARCHAR2(10) NOT NULL CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    user_name       VARCHAR2(50) NOT NULL,
    attempt_date    DATE DEFAULT SYSDATE,
    attempt_day     VARCHAR2(10),
    is_holiday      VARCHAR2(1) DEFAULT 'N' CHECK (is_holiday IN ('Y', 'N')),
    status          VARCHAR2(10) NOT NULL CHECK (status IN ('ALLOWED', 'DENIED')),
    error_message   VARCHAR2(200)
);

-- Indexes for audit_log
CREATE INDEX idx_audit_table ON audit_log(table_name);
CREATE INDEX idx_audit_status ON audit_log(status);
CREATE INDEX idx_audit_date ON audit_log(attempt_date);

-- ==================== VERIFICATION ====================
PROMPT ========== TABLES CREATED SUCCESSFULLY ==========
SELECT 'Suppliers' AS table_name, COUNT(*) AS row_count FROM user_tables WHERE table_name = 'SUPPLIERS'
UNION ALL
SELECT 'Products', COUNT(*) FROM user_tables WHERE table_name = 'PRODUCTS'
UNION ALL
SELECT 'Stock_Entries', COUNT(*) FROM user_tables WHERE table_name = 'STOCK_ENTRIES'
UNION ALL
SELECT 'Reorder_Log', COUNT(*) FROM user_tables WHERE table_name = 'REORDER_LOG'
UNION ALL
SELECT 'Audit_Log', COUNT(*) FROM user_tables WHERE table_name = 'AUDIT_LOG';


2. **GRPA_27129_Dorothy_SIMS_INSERT_DATA.sql** - Inserts realistic sample data (100+ rows)
BEGIN
    EXECUTE IMMEDIATE 'DELETE FROM reorder_log';
    EXECUTE IMMEDIATE 'DELETE FROM stock_entries';
    EXECUTE IMMEDIATE 'DELETE FROM products';
    EXECUTE IMMEDIATE 'DELETE FROM suppliers';
    EXECUTE IMMEDIATE 'DELETE FROM audit_log';
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        NULL; -- Tables might not exist yet
END;
/

-- 1. INSERT SUPPLIERS
INSERT INTO suppliers (supplier_id, supplier_name, contact_phone, email, address) 
VALUES (suppliers_seq.NEXTVAL, 'Fresh Foods Ltd', '+250788111001', 'orders@freshfoods.rw', 'Kigali, Rwanda');

INSERT INTO suppliers (supplier_id, supplier_name, contact_phone, email, address) 
VALUES (suppliers_seq.NEXTVAL, 'PharmaCare Suppliers', '+250788222002', 'contact@pharmacare.rw', 'Gisenyi, Rwanda');

INSERT INTO suppliers (supplier_id, supplier_name, contact_phone, email, address) 
VALUES (suppliers_seq.NEXTVAL, 'TechGadgets Inc.', '+250788333003', 'sales@techgadgets.com', 'Kampala, Uganda');

INSERT INTO suppliers (supplier_id, supplier_name, contact_phone, email, address) 
VALUES (suppliers_seq.NEXTVAL, 'Office Supplies Co.', '+250788444004', 'info@officesupplies.co', 'Nairobi, Kenya');

INSERT INTO suppliers (supplier_id, supplier_name, contact_phone, email, address) 
VALUES (suppliers_seq.NEXTVAL, 'Beverage Distributors', '+250788555005', 'delivery@beverages.rw', 'Kigali, Rwanda');

COMMIT;

-- 2. INSERT PRODUCTS (15 products with realistic data)
INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Rice 10kg', 'Food', 100, 50, 12000, 200);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Sugar 5kg', 'Food', 100, 40, 8000, 150);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Cooking Oil 3L', 'Food', 100, 30, 6000, 80);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Paracetamol 500mg', 'Pharmacy', 101, 100, 500, 300);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Vitamin C 100 tabs', 'Pharmacy', 101, 60, 3000, 120);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'USB-C Cable', 'Electronics', 102, 75, 2500, 200);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Wireless Mouse', 'Electronics', 102, 40, 8500, 60);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'A4 Paper Ream', 'Office', 103, 80, 4500, 150);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Blue Pens (Box of 20)', 'Office', 103, 120, 2000, 250);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Cola 500ml', 'Beverage', 104, 200, 800, 400);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Bottled Water 1L', 'Beverage', 104, 150, 500, 300);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Milk Powder 1kg', 'Food', 100, 60, 4500, 90);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Soap Bar', 'Personal Care', 101, 200, 800, 350);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Notebooks (Pack of 5)', 'Office', 103, 100, 3500, 180);

INSERT INTO products (product_id, product_name, category, supplier_id, min_quantity, unit_price, current_quantity) 
VALUES (products_seq.NEXTVAL, 'Tea Bags 100pc', 'Beverage', 104, 80, 2500, 120);

COMMIT;

-- 3. INSERT STOCK ENTRIES (Sample transactions)
-- Add some initial stock
INSERT INTO stock_entries (stock_id, product_id, transaction_type, quantity, transaction_date, performed_by_user)
SELECT stock_entries_seq.NEXTVAL, product_id, 'IN', current_quantity, SYSDATE - 30, 'admin'
FROM products;

-- Add some sales/out transactions (past month)
DECLARE
    v_counter NUMBER := 0;
BEGIN
    FOR prod IN (SELECT product_id, current_quantity FROM products WHERE current_quantity > 20) LOOP
        FOR i IN 1..5 LOOP
            INSERT INTO stock_entries (stock_id, product_id, transaction_type, quantity, transaction_date, performed_by_user)
            VALUES (
                stock_entries_seq.NEXTVAL,
                prod.product_id,
                'OUT',
                ROUND(DBMS_RANDOM.VALUE(1, 10)),
                SYSDATE - ROUND(DBMS_RANDOM.VALUE(1, 28)),
                'clerk_' || ROUND(DBMS_RANDOM.VALUE(1, 3))
            );
            v_counter := v_counter + 1;
        END LOOP;
    END LOOP;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Inserted ' || v_counter || ' stock out transactions');
END;
/

-- 4. UPDATE PRODUCT QUANTITIES based on stock entries
MERGE INTO products p
USING (
    SELECT 
        product_id,
        SUM(CASE WHEN transaction_type = 'IN' THEN quantity ELSE -quantity END) AS net_quantity
    FROM stock_entries
    GROUP BY product_id
) s
ON (p.product_id = s.product_id)
WHEN MATCHED THEN
    UPDATE SET p.current_quantity = GREATEST(s.net_quantity, 0);

COMMIT;

-- 5. INSERT REORDER LOG ENTRIES (for low stock items)
-- First, manually set some items to low quantity for testing
UPDATE products SET current_quantity = 5 WHERE product_id = 1000; -- Rice
UPDATE products SET current_quantity = 8 WHERE product_id = 1003; -- Paracetamol
UPDATE products SET current_quantity = 15 WHERE product_id = 1006; -- USB Cable
COMMIT;

-- Insert reorder alerts
INSERT INTO reorder_log (reorder_id, product_id, current_quantity, min_quantity, suggested_reorder_quantity, alert_date, status)
SELECT 
    reorder_log_seq.NEXTVAL,
    product_id,
    current_quantity,
    min_quantity,
    (min_quantity * 2) - current_quantity,
    SYSDATE - ROUND(DBMS_RANDOM.VALUE(0, 7)),
    CASE WHEN ROUND(DBMS_RANDOM.VALUE(1, 3)) = 1 THEN 'ORDERED' ELSE 'PENDING' END
FROM products 
WHERE current_quantity < min_quantity;

COMMIT;
3. **GRPA_27129_Dorothy_SIMS_VALIDATION_QUERIES.sql** - Validates data integrity and constraints
4. **GRPA_27129_Dorothy_SIMS_PHASE_V_COMPLETE.sql** - Complete Phase V execution script
-- ============================================
-- COMPLETE PHASE V DATA POPULATION
-- ============================================

PROMPT === STEP 1: Check current stock levels ===
SELECT 
    product_id,
    product_name,
    current_quantity,
    min_quantity,
    CASE 
        WHEN current_quantity < min_quantity THEN 'BELOW MINIMUM - NEEDS REORDER'
        ELSE 'OK'
    END AS status
FROM products
ORDER BY product_id;

PROMPT === STEP 2: Set some products below minimum ===
DECLARE
    v_count NUMBER;
BEGIN
    -- Update 5 random products to be below minimum (30% of min_quantity)
    UPDATE products 
    SET current_quantity = ROUND(min_quantity * 0.3)
    WHERE product_id IN (
        SELECT product_id FROM (
            SELECT product_id FROM products 
            ORDER BY DBMS_RANDOM.VALUE
        ) WHERE ROWNUM <= 5
    );
    
    v_count := SQL%ROWCOUNT;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Updated ' || v_count || ' products to be below minimum quantity');
END;
/

PROMPT === STEP 3: Insert reorder alerts ===
DECLARE
    v_count NUMBER;
BEGIN
    -- Delete any existing reorder logs
    DELETE FROM reorder_log;
    
    -- Insert new reorder alerts for low stock items
    INSERT INTO reorder_log (product_id, current_quantity, min_quantity, suggested_reorder_quantity, alert_date, status)
    SELECT 
        product_id,
        current_quantity,
        min_quantity,
        (min_quantity * 2) - current_quantity,
        SYSDATE - ROUND(DBMS_RANDOM.VALUE(0, 7)),
        CASE 
            WHEN DBMS_RANDOM.VALUE < 0.3 THEN 'ORDERED'
            WHEN DBMS_RANDOM.VALUE < 0.6 THEN 'PENDING'
            ELSE 'IGNORED'
        END
    FROM products 
    WHERE current_quantity < min_quantity;
    
    v_count := SQL%ROWCOUNT;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Created ' || v_count || ' reorder alerts');
END;
/

PROMPT === STEP 4: Insert audit log sample data ===
DECLARE
    v_count NUMBER;
BEGIN
    -- Delete any existing audit logs
    DELETE FROM audit_log;
    
    -- Insert sample audit logs
    INSERT INTO audit_log (table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message) VALUES
    ('STOCK_ENTRIES', 'INSERT', 'SIMS_USER', SYSDATE - 2, 'MONDAY', 'N', 'ALLOWED', NULL);
    
    INSERT INTO audit_log (table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message) VALUES
    ('STOCK_ENTRIES', 'INSERT', 'SIMS_USER', SYSDATE - 1, 'TUESDAY', 'N', 'DENIED', 'Operation not allowed on weekdays');
    
    INSERT INTO audit_log (table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message) VALUES
    ('PRODUCTS', 'UPDATE', 'MANAGER', SYSDATE - 3, 'SUNDAY', 'N', 'ALLOWED', NULL);
    
    INSERT INTO audit_log (table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message) VALUES
    ('SUPPLIERS', 'DELETE', 'ADMIN', SYSDATE - 4, 'SATURDAY', 'Y', 'DENIED', 'Operation not allowed on holidays');
    
    INSERT INTO audit_log (table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message) VALUES
    ('REORDER_LOG', 'INSERT', 'SYSTEM', SYSDATE, TO_CHAR(SYSDATE, 'DAY'), 'N', 'ALLOWED', NULL);
    
    v_count := SQL%ROWCOUNT;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Inserted ' || v_count || ' audit log records');
END;
/

PROMPT === STEP 5: Final verification ===
SELECT '=== TABLE COUNTS ===' AS verification FROM dual
UNION ALL
SELECT 'Suppliers: ' || COUNT(*) || ' rows' FROM suppliers
UNION ALL
SELECT 'Products: ' || COUNT(*) || ' rows' FROM products
UNION ALL
SELECT 'Stock Entries: ' || COUNT(*) || ' rows' FROM stock_entries
UNION ALL
SELECT 'Reorder Logs: ' || COUNT(*) || ' rows' FROM reorder_log
UNION ALL
SELECT 'Audit Logs: ' || COUNT(*) || ' rows' FROM audit_log
UNION ALL
SELECT ' ' FROM dual
UNION ALL
SELECT '=== INVENTORY STATUS ===' FROM dual
UNION ALL
SELECT 'Products below minimum: ' || COUNT(*) || ' items' 
FROM products WHERE current_quantity < min_quantity
UNION ALL
SELECT 'Pending reorders: ' || COUNT(*) || ' alerts' 
FROM reorder_log WHERE status = 'PENDING';

PROMPT === STEP 6: Sample data display ===
SELECT '=== SAMPLE PRODUCTS (first 5) ===' AS sample_data FROM dual;
SELECT product_id, product_name, current_quantity, min_quantity FROM products WHERE ROWNUM <= 5;

SELECT '=== REORDER LOG ===' AS sample_data FROM dual;
SELECT * FROM reorder_log;

SELECT '=== AUDIT LOG ===' AS sample_data FROM dual;
SELECT audit_id, table_name, operation, user_name, TO_CHAR(attempt_date, 'MM/DD') AS date, status 
FROM audit_log ORDER BY attempt_date DESC;

PROMPT ============================================
PROMPT PHASE V DATA POPULATION COMPLETE!
PROMPT ============================================

## Database Structure:
| Table | Rows | Description |
|-------|------|-------------|
| SUPPLIERS | 5 | Product suppliers with contact information |
| PRODUCTS | 39 | Inventory items with min/max quantities |
| STOCK_ENTRIES | 234 | Stock movement transactions (IN/OUT) |
| REORDER_LOG | 12 | Automatic reorder alerts for low stock |
| AUDIT_LOG | 5 | Audit trail for business rule enforcement |

## Key Features Implemented:
- **Primary/Foreign Keys**: All relationships properly enforced
- **CHECK Constraints**: min_quantity > 0, current_quantity >= 0, etc.
- **Indexes**: Performance optimization on frequently queried columns
- **Sequences**: Auto-increment for all primary keys
- **Data Validation**: Comprehensive validation queries

## Data Highlights:
- 5 suppliers with realistic contact information
- 39 products across categories (Food, Pharmacy, Electronics, Office, Beverage)
- 234 stock transactions simulating 2 months of inventory activity
- 12 automatic reorder alerts triggered by low stock levels
- 5 sample audit records for Phase VII business rules

## Verification Results:
- ✅ All tables created successfully
- ✅ All constraints enforced
- ✅ Foreign key relationships validated
- ✅ Sample data represents real business scenarios
- ✅ Data integrity maintained across all tables

# **PHASE VI: Database Interaction & Transactions (PL/SQL)**
**Smart Inventory Monitoring System (SIMRS)**
**Author: Nshuti DOROTHY**
 **Date: 2025-12-09**


1️⃣## **Purpose & Scope**

/*
**Purpose**:
Automate stock transactions, compute reorder alerts, expose utility functions for reporting,
and provide efficient bulk operations. All PL/SQL objects are intended to be executed
by the schema owner (the same user that owns products, suppliers, stock_entries, reorder_log, audit_log).

**Scope**:

5 main procedures (add stock, update price, delete supplier safe, generate reorder alerts, update min)

5 utility functions (current stock, reorder-needed flag, supplier rating placeholder, stock value, count by category)

Cursor example script (list low-stock items)

Bulk insert procedure using BULK COLLECT / FORALL

Window-function analytics example

Package: inventory_pkg (spec + body) with audit helper (autonomous transaction)

Test scripts for each object
*/
 ##  **PROCEDURES**
 
 ## **1.proc_add_stock_entry**

CREATE OR REPLACE PROCEDURE proc_add_stock_entry(
    p_product_id    IN NUMBER,
    p_txn_type      IN VARCHAR2, -- 'IN' or 'OUT'
    p_quantity      IN NUMBER,
    p_performed_by  IN VARCHAR2 DEFAULT NULL
)
IS
    v_new_qty NUMBER;
BEGIN
    IF p_txn_type NOT IN ('IN','OUT') THEN
        RAISE_APPLICATION_ERROR(-20001, 'Invalid transaction type. Use IN or OUT.');
    END IF;

    INSERT INTO stock_entries(stock_id, product_id, transaction_type, quantity, transaction_date, performed_by_user)
    VALUES (stock_entries_seq.NEXTVAL, p_product_id, p_txn_type, p_quantity, SYSDATE, NVL(p_performed_by, SYS_CONTEXT('USERENV','SESSION_USER')));

    -- compute net quantity and update products.current_quantity
    SELECT NVL(SUM(CASE WHEN transaction_type='IN' THEN quantity ELSE -quantity END),0)
      INTO v_new_qty
      FROM stock_entries
     WHERE product_id = p_product_id;

    UPDATE products SET current_quantity = GREATEST(v_new_qty, 0) WHERE product_id = p_product_id;

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Stock entry added and product quantity updated. New qty: ' || v_new_qty);
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END proc_add_stock_entry;
/

<img width="1593" height="683" alt="-- 1  proc_add_stock_entry" src="https://github.com/user-attachments/assets/ccefadcb-a1e5-4463-89af-ecf7193b35c2" />

-- ## **2.proc_update_product_price**
CREATE OR REPLACE PROCEDURE proc_update_product_price(
    p_product_id IN NUMBER,
    p_new_price  IN NUMBER
)
IS
BEGIN
    IF p_new_price < 0 THEN
        RAISE_APPLICATION_ERROR(-20002, 'Unit price cannot be negative.');
    END IF;

    UPDATE products
       SET unit_price = p_new_price
     WHERE product_id = p_product_id;

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Updated price for product ' || p_product_id || ' to ' || TO_CHAR(p_new_price));
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Product not found: ' || p_product_id);
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END proc_update_product_price;
/


<img width="1532" height="730" alt="-- 2  proc_update_product_price" src="https://github.com/user-attachments/assets/e6c6ffbe-706b-4445-b827-650ece9012a5" />


-- ## **3.proc_delete_supplier_safe**

--### **Attempts soft-delete: only allowed if no products reference supplier.**

CREATE OR REPLACE PROCEDURE proc_delete_supplier_safe(p_supplier_id IN NUMBER)
IS
    v_cnt NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_cnt FROM products WHERE supplier_id = p_supplier_id;

    IF v_cnt > 0 THEN
        RAISE_APPLICATION_ERROR(-20003, 'Cannot delete supplier: referenced by products (' || v_cnt || ').');
    ELSE
        DELETE FROM suppliers WHERE supplier_id = p_supplier_id;
        COMMIT;
        DBMS_OUTPUT.PUT_LINE('Supplier ' || p_supplier_id || ' deleted.');
    END IF;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END proc_delete_supplier_safe;
/

<img width="1572" height="689" alt="-- 3  proc_delete_supplier_safe" src="https://github.com/user-attachments/assets/48237f32-28f4-44a7-971b-04362d734087" />


 ## **4.proc_reorder_generate**

### **Scans products and inserts into reorder_log when current < min.**

CREATE OR REPLACE PROCEDURE proc_reorder_generate(p_threshold_mult IN NUMBER DEFAULT 2)
IS
    CURSOR c_low IS
        SELECT product_id, current_quantity, min_quantity
          FROM products
         WHERE current_quantity < min_quantity
         FOR UPDATE;

    v_suggest NUMBER;
BEGIN
    FOR r IN c_low LOOP
        v_suggest := (r.min_quantity * p_threshold_mult) - r.current_quantity;
        IF v_suggest <= 0 THEN
            v_suggest := r.min_quantity; -- fallback
        END IF;

        INSERT INTO reorder_log(reorder_id, product_id, current_quantity, min_quantity, suggested_reorder_quantity, alert_date, status)
        VALUES (reorder_log_seq.NEXTVAL, r.product_id, r.current_quantity, r.min_quantity, v_suggest, SYSDATE, 'PENDING');
    END LOOP;

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Reorder alerts generated.');
EXCEPTION
    WHEN DUP_VAL_ON_INDEX THEN
        -- ignore duplicates if any (rare)
        NULL;
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END proc_reorder_generate;
/

<img width="1427" height="817" alt="-- 4  proc_reorder_generate" src="https://github.com/user-attachments/assets/45d16191-ac75-4603-939e-933e23391b04" />


 ## **5.proc_update_min_quantity**

CREATE OR REPLACE PROCEDURE proc_update_min_quantity(
    p_product_id IN NUMBER,
    p_new_min    IN NUMBER
)
IS
BEGIN
    IF p_new_min <= 0 THEN
        RAISE_APPLICATION_ERROR(-20004, 'min_quantity must be > 0');
    END IF;

    UPDATE products SET min_quantity = p_new_min WHERE product_id = p_product_id;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('min_quantity updated for product ' || p_product_id || ' to ' || p_new_min);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Product not found: ' || p_product_id);
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END proc_update_min_quantity;
/

<img width="1481" height="743" alt="-- 5  proc_update_min_quantity" src="https://github.com/user-attachments/assets/9aeb8556-1f06-4a43-bb23-814e11145616" />

  ## **FUNCTIONS** 
-- fn_get_current_stock(product_id) returns current quantity
CREATE OR REPLACE FUNCTION fn_get_current_stock(p_product_id IN NUMBER) RETURN NUMBER IS
    v_qty NUMBER;
BEGIN
    SELECT current_quantity INTO v_qty FROM products WHERE product_id = p_product_id;
    RETURN NVL(v_qty, 0);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;
END fn_get_current_stock;
/
<img width="1163" height="610" alt="-- fn_get_current_stock(product_id) returns current quantity" src="https://github.com/user-attachments/assets/fea060b3-2288-41d5-a8df-1634a69a9784" />

-- fn_is_reorder_needed(product_id) returns 'Y' or 'N'
CREATE OR REPLACE FUNCTION fn_is_reorder_needed(p_product_id IN NUMBER) RETURN VARCHAR2 IS
    v_cur NUMBER;
    v_min NUMBER;
BEGIN
    SELECT current_quantity, min_quantity INTO v_cur, v_min FROM products WHERE product_id = p_product_id;
    RETURN CASE WHEN v_cur < v_min THEN 'Y' ELSE 'N' END;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'N';
END fn_is_reorder_needed;
/ 
<img width="1217" height="643" alt="-- fn_is_reorder_needed(product_id) returns &#39;Y&#39; or &#39;N&#39;" src="https://github.com/user-attachments/assets/5bfe226e-223d-411f-b72a-af4286f90a7f" />


-- fn_get_supplier_rating(supplier_id) simple lookup (placeholder, returns 0..5)
-- For now returns an average simulated value based on supplier_id (could be replaced by real ratings table)
CREATE OR REPLACE FUNCTION fn_get_supplier_rating(p_supplier_id IN NUMBER) RETURN NUMBER IS
BEGIN
    RETURN MOD(p_supplier_id, 5) + 1; -- not real; placeholder for demo
END fn_get_supplier_rating;
/


-- fn_calculate_stock_value(product_id) returns current_quantity * unit_price
CREATE OR REPLACE FUNCTION fn_calculate_stock_value(p_product_id IN NUMBER) RETURN NUMBER IS
    v_qty NUMBER;
    v_price NUMBER;
BEGIN
    SELECT NVL(current_quantity,0), NVL(unit_price,0) INTO v_qty, v_price FROM products WHERE product_id = p_product_id;
    RETURN v_qty * v_price;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;
END fn_calculate_stock_value;
/
-- fn_total_products_in_category(category_name) returns number of products in category
CREATE OR REPLACE FUNCTION fn_total_products_in_category(p_category IN VARCHAR2) RETURN NUMBER IS
    v_cnt NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_cnt FROM products WHERE category = p_category;
    RETURN v_cnt;
END fn_total_products_in_category;
/
<img width="1400" height="769" alt="-- fn_ 111" src="https://github.com/user-attachments/assets/2f722f3c-2829-4676-8f35-8cb11d0be9ef" />


--             ## **Cursor & Sample Loop** 


-- Example: cursor to list products below minimum and print details
SET SERVEROUTPUT ON
DECLARE
    CURSOR c_below IS
        SELECT product_id, product_name, current_quantity, min_quantity
          FROM products
         WHERE current_quantity < min_quantity
         ORDER BY product_id;

    r_prod c_below%ROWTYPE;
BEGIN
    OPEN c_below;
    LOOP
        FETCH c_below INTO r_prod;
        EXIT WHEN c_below%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE('Product ' || r_prod.product_id || ' - ' || r_prod.product_name ||
                             ' cur:' || r_prod.current_quantity || ' min:' || r_prod.min_quantity);
    END LOOP;
    CLOSE c_below;
END;
/

<img width="1375" height="687" alt="procedures creation 1" src="https://github.com/user-attachments/assets/23fdfe55-19d3-44d7-bf81-be6fa0d426d5" />


 ## **BULK Operation (BULK COLLECT + FORALL)**
CREATE OR REPLACE PROCEDURE proc_bulk_insert_reorders(p_product_ids IN SYS.ODCINUMBERLIST)
IS
    TYPE t_prod_rec IS RECORD (product_id NUMBER, current_quantity NUMBER, min_quantity NUMBER);
    TYPE t_prod_tab IS TABLE OF t_prod_rec;
    l_prods t_prod_tab;
BEGIN
    SELECT product_id, current_quantity, min_quantity
      BULK COLLECT INTO l_prods
      FROM products
     WHERE product_id MEMBER OF p_product_ids;

    FORALL i IN 1..l_prods.COUNT
        INSERT INTO reorder_log(reorder_id, product_id, current_quantity, min_quantity, suggested_reorder_quantity, alert_date, status)
        VALUES (reorder_log_seq.NEXTVAL,
                l_prods(i).product_id,
                l_prods(i).current_quantity,
                l_prods(i).min_quantity,
                GREATEST((l_prods(i).min_quantity * 2) - l_prods(i).current_quantity, l_prods(i).min_quantity),
                SYSDATE,
                'PENDING');

    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Bulk inserted ' || l_prods.COUNT || ' reorder rows.');
END proc_bulk_insert_reorders;
/



<img width="1567" height="740" alt="4) BULK Operation" src="https://github.com/user-attachments/assets/c11d2448-a22c-4a9e-853b-fe58739b99f8" />

5) Window-function analytics example (for Phase VI testing)

<img width="1120" height="720" alt="5) Window-function analytics example" src="https://github.com/user-attachments/assets/63207a8f-6248-403c-a854-f9361d3723e5" />

-- ==================================================================
-- 4️⃣## **Package: inventory_pkg (spec + body)**
-- =====================================================================
## **Package: inventory_pkg**
-- PACKAGE SPEC
CREATE OR REPLACE PACKAGE inventory_pkg IS
    PROCEDURE log_audit(p_table_name VARCHAR2, p_operation VARCHAR2, p_status VARCHAR2, p_error_msg VARCHAR2);
    PROCEDURE generate_reorders(p_mult IN NUMBER DEFAULT 2);
    FUNCTION is_reorder_needed(p_product_id IN NUMBER) RETURN VARCHAR2;
    FUNCTION get_stock_value(p_product_id IN NUMBER) RETURN NUMBER;
END inventory_pkg;
/
<img width="1528" height="776" alt="6) Package inventory" src="https://github.com/user-attachments/assets/2c9ab4a0-cb16-459f-9923-70acc1b6ac51" />


## **PACKAGE BODY**

 ### **CREATE OR REPLACE PACKAGE BODY inventory_pkg IS**

    -- autonomous audit writer so that audit entries persist even if caller rolls back
    PROCEDURE log_audit(p_table_name VARCHAR2, p_operation VARCHAR2, p_status VARCHAR2, p_error_msg VARCHAR2) IS
        PRAGMA AUTONOMOUS_TRANSACTION;
    BEGIN
        INSERT INTO audit_log(audit_id, table_name, operation, user_name, attempt_date, attempt_day, is_holiday, status, error_message)
        VALUES (audit_log_seq.NEXTVAL, p_table_name, p_operation, SYS_CONTEXT('USERENV','SESSION_USER'), SYSDATE, TO_CHAR(SYSDATE,'DAY'), 'N', p_status, SUBSTR(p_error_msg,1,200));
        COMMIT;
    EXCEPTION
        WHEN OTHERS THEN
            NULL; -- avoid cascading failures from audit
    END log_audit;

    -- wrapper that calls proc_reorder_generate
    PROCEDURE generate_reorders(p_mult IN NUMBER DEFAULT 2) IS
    BEGIN
        proc_reorder_generate(p_mult);
        log_audit('REORDER_LOG','INSERT','ALLOWED','Reorders generated by inventory_pkg.generate_reorders');
    EXCEPTION
        WHEN OTHERS THEN
            log_audit('REORDER_LOG','INSERT','DENIED',SQLERRM);
            RAISE;
    END generate_reorders;

    FUNCTION is_reorder_needed(p_product_id IN NUMBER) RETURN VARCHAR2 IS
    BEGIN
        RETURN fn_is_reorder_needed(p_product_id);
    END is_reorder_needed;

    FUNCTION get_stock_value(p_product_id IN NUMBER) RETURN NUMBER IS
    BEGIN
        RETURN fn_calculate_stock_value(p_product_id);
    END get_stock_value;

END inventory_pkg;
/


<img width="1460" height="644" alt="-- PACKAGE BODY" src="https://github.com/user-attachments/assets/46c4b829-94ca-4d9e-8321-eee4146122af" />

 ## **Test procedure proc_add_stock_entry**

DECLARE
  v_product_id products.product_id%TYPE;
BEGIN
  -- Get the product_id into a variable
  SELECT product_id 
  INTO v_product_id 
  FROM products 
  WHERE ROWNUM = 1;

  -- Call the procedure using the variable
  proc_add_stock_entry(v_product_id, 'IN', 10, 'tester');
END;
/


<img width="1237" height="655" alt="Test procedure proc_add_stock_entry" src="https://github.com/user-attachments/assets/af047ee6-506d-4d86-a600-d1b77808c04f" />

## **Test proc_update_product_price**


DECLARE
  v_product_id products.product_id%TYPE;
BEGIN
  -- Fetch a product_id
  SELECT product_id INTO v_product_id
  FROM products
  WHERE ROWNUM = 1;

  -- Call procedure
  proc_update_product_price(v_product_id, 9999.50);
END;
/


<img width="1223" height="679" alt="1️⃣ Test proc_update_product_price" src="https://github.com/user-attachments/assets/a5a13920-b068-48d2-ae90-d91d0d65424d" />

DECLARE
  v_supplier_id suppliers.supplier_id%TYPE;
BEGIN
  -- Fetch a supplier_id
  SELECT supplier_id INTO v_supplier_id
  FROM suppliers
  WHERE ROWNUM = 1;

  ## **Call procedure**
  proc_delete_supplier_safe(v_supplier_id);

EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/

<img width="1064" height="664" alt="2️⃣ Test proc_delete_supplier_safe" src="https://github.com/user-attachments/assets/ef99f2d2-69f5-4d90-8fda-2330ad522185" />
BEGIN
  proc_reorder_generate(2);
END;
/

<img width="1055" height="663" alt="3️⃣ Test proc_reorder_generate" src="https://github.com/user-attachments/assets/f3ac92d8-dfd3-488f-8c76-09029aca69be" />
SELECT * FROM reorder_log WHERE ROWNUM <= 10;
<img width="1401" height="761" alt="Check results" src="https://github.com/user-attachments/assets/caaec665-4cb9-41f6-b489-9415930166be" />

4️⃣## **Test proc_update_min_quantity**

DECLARE
  v_product_id products.product_id%TYPE;
BEGIN
  -- Fetch a product_id
  SELECT product_id INTO v_product_id
  FROM products
  WHERE ROWNUM = 1;

  -- Call procedure
  proc_update_min_quantity(v_product_id, 25);
END;
/
 ## **Test Package inventory_pkg.generate_reorders and check audit log**
BEGIN
  -- Generate reorders for a given supplier/product (example 2)
  inventory_pkg.generate_reorders(2);
END;
/

-- ## **Check the audit log**
SELECT * 
FROM audit_log
WHERE ROWNUM <= 10
ORDER BY attempt_date DESC;
<img width="1423" height="786" alt="2️⃣ Test Package inventory_pkg generate_reorders and check audit log" src="https://github.com/user-attachments/assets/291ea370-077f-46e4-9bf9-8f8f3a316c61" />


# **Phase VII: Advanced Programming & Auditing**

-- **PHASE VII: Advanced Programming & Auditing**
-- **Smart Inventory Monitoring System**
-- **Author: Nshuti DOrothy**
-- Date: 2025-12-09


SET SERVEROUTPUT ON;

-- ===========================
-- 1️⃣## **Holiday Management**
-- ===========================

CREATE TABLE public_holidays (
    holiday_date DATE PRIMARY KEY,
    description VARCHAR2(100)
);
INSERT INTO public_holidays (holiday_date, description)
VALUES (TO_DATE('2025-12-25','YYYY-MM-DD'), 'Christmas');

INSERT INTO public_holidays (holiday_date, description)
VALUES (TO_DATE('2026-01-01','YYYY-MM-DD'), 'New Year'); 
<img width="1172" height="715" alt="HOLIDAY MANAGEMENT" src="https://github.com/user-attachments/assets/9fff871e-2617-49a2-bc3c-8750cb757f22" /> 

-- 2️⃣## **Audit Logging Procedure**

CREATE OR REPLACE PROCEDURE log_audit(
    p_action_type IN VARCHAR2,
    p_table_name  IN VARCHAR2,
    p_status      IN VARCHAR2,
    p_error_msg   IN VARCHAR2
)
IS
BEGIN
    INSERT INTO audit_log (
        username, action_type, table_name, action_date, status, error_msg
    )
    VALUES (
        USER, p_action_type, p_table_name, SYSTIMESTAMP, p_status, p_error_msg
    );
END;
/


<img width="1168" height="638" alt="image" src="https://github.com/user-attachments/assets/ba727586-8ca9-4c35-a485-be581192d19d" />


 ## **Restriction Check Function**

CREATE OR REPLACE FUNCTION check_dml_allowed
RETURN BOOLEAN
IS
    v_today DATE := TRUNC(SYSDATE);
    v_day_of_week VARCHAR2(10);
    v_count NUMBER;
BEGIN
    v_day_of_week := TO_CHAR(v_today, 'DY', 'NLS_DATE_LANGUAGE=ENGLISH');

    IF v_day_of_week IN ('MON', 'TUE', 'WED', 'THU', 'FRI') THEN
        RETURN FALSE;
    END IF;

    -- Check public holiday
    SELECT COUNT(*) INTO v_count 
    FROM public_holidays
    WHERE holiday_date = v_today;

    IF v_count > 0 THEN
        RETURN FALSE;
    END IF;

    RETURN TRUE;
END;
/
<img width="1537" height="754" alt="4️⃣ Restriction Check Function" src="https://github.com/user-attachments/assets/12aa46c1-9e8e-4917-a707-a2c05494f96e" />

## **Compound Trigger**
CREATE OR REPLACE TRIGGER trg_products_compound
FOR INSERT OR UPDATE OR DELETE ON products
COMPOUND TRIGGER

  -- Before Statement
  BEFORE STATEMENT IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Compound Trigger: Statement start');
  END BEFORE STATEMENT;

  -- After Each Row
  AFTER EACH ROW IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Compound Trigger: Row processed');
  END AFTER EACH ROW;

  -- After Statement
  AFTER STATEMENT IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Compound Trigger: Statement end');
  END AFTER STATEMENT;

END trg_products_compound;
/
<img width="1585" height="782" alt="Compound Trigger" src="https://github.com/user-attachments/assets/71bf4491-5f00-4615-a68a-2eb8f097bf47" />


-- 4️⃣## **Trigger for PRODUCTS Table**

CREATE OR REPLACE TRIGGER trg_products_dml
BEFORE INSERT OR UPDATE OR DELETE ON products
FOR EACH ROW
DECLARE
    v_allowed BOOLEAN;
BEGIN
    v_allowed := check_dml_allowed;

    IF NOT v_allowed THEN
        log_audit(
            p_action_type  => CASE
                                WHEN INSERTING THEN 'INSERT'
                                WHEN UPDATING THEN 'UPDATE'
                                WHEN DELETING THEN 'DELETE'
                              END,
            p_table_name   => 'PRODUCTS',
            p_status       => 'DENIED',
            p_error_msg    => 'DML not allowed on weekdays or public holidays'
        );
        RAISE_APPLICATION_ERROR(-20001, 'DML not allowed on weekdays or public holidays');
    ELSE
        log_audit(
            p_action_type  => CASE
                                WHEN INSERTING THEN 'INSERT'
                                WHEN UPDATING THEN 'UPDATE'
                                WHEN DELETING THEN 'DELETE'
                              END,
            p_table_name   => 'PRODUCTS',
            p_status       => 'ALLOWED',
            p_error_msg    => NULL
        );
    END IF;
END;
/
<img width="1168" height="638" alt="image" src="https://github.com/user-attachments/assets/ba727586-8ca9-4c35-a485-be581192d19d" />


 5️⃣## **Trigger for SUPPLIERS Table**

CREATE OR REPLACE TRIGGER trg_suppliers_dml
BEFORE INSERT OR UPDATE OR DELETE ON suppliers
FOR EACH ROW
DECLARE
    v_allowed BOOLEAN;
BEGIN
    v_allowed := check_dml_allowed;

    IF NOT v_allowed THEN
        log_audit(
            p_action_type  => CASE
                                WHEN INSERTING THEN 'INSERT'
                                WHEN UPDATING THEN 'UPDATE'
                                WHEN DELETING THEN 'DELETE'
                              END,
            p_table_name   => 'SUPPLIERS',
            p_status       => 'DENIED',
            p_error_msg    => 'DML not allowed on weekdays or public holidays'
        );
        RAISE_APPLICATION_ERROR(-20002, 'DML not allowed on weekdays or public holidays');
    ELSE
        log_audit(
            p_action_type  => CASE
                                WHEN INSERTING THEN 'INSERT'
                                WHEN UPDATING THEN 'UPDATE'
                                WHEN DELETING THEN 'DELETE'
                              END,
            p_table_name   => 'SUPPLIERS',
            p_status       => 'ALLOWED',
            p_error_msg    => NULL
        );
    END IF;
END;
/



## **Test 1: Attempt INSERT on weekday (should fail)**
BEGIN
    INSERT INTO products(product_name, current_quantity, min_quantity)
    VALUES('Test Product Weekday', 10, 5);
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Expected failure: ' || SQLERRM);
END;
/

<img width="1413" height="680" alt="Test 1 Attempt INSERT on weekday (should fail)" src="https://github.com/user-attachments/assets/8777fbb0-5eed-479f-8d89-5271111bcc76" />

## **Test 3: Attempt INSERT on public holiday**

-- Simulate by inserting into the holiday table today
INSERT INTO public_holidays(holiday_date, description)
VALUES(TRUNC(SYSDATE), 'Test Holiday');
COMMIT;

BEGIN
    INSERT INTO products(product_name, current_quantity, min_quantity)
    VALUES('Test Product Holiday', 15, 5);
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Expected failure: ' || SQLERRM);
END;
/
<img width="1262" height="759" alt="Test 3 Attempt INSERT on public holiday" src="https://github.com/user-attachments/assets/8db40acd-232f-4e03-8bdf-178747448631" />
 ## **Test 2: Attempt INSERT on weekend**
BEGIN
    INSERT INTO products(product_name, current_quantity, min_quantity)
    VALUES('Test Product Weekend', 20, 5);
END;
/




## Files in This Phase
- `GRPA_27129_Dorothy_SIMS_Database_Creation.sql`: Main creation script.
- `PHASE_IV_README.md`: This file.
