# SIMRS_PLSQL_Capstone
PL/SQL Capstone Project - Smart Inventory Monitoring &amp; Reordering System.

# File: PHASE_IV_README.md
# Phase IV: Database Creation - SIMRS Project

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

PHASE 6 CODES  ON PROCECEDURES

-- 1. proc_add_stock_entry
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

-- 2. proc_update_product_price
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


-- 3. proc_delete_supplier_safe
-- Attempts soft-delete: only allowed if no products reference supplier.
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


-- 4. proc_reorder_generate
-- Scans products and inserts into reorder_log when current < min.
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


-- 5. proc_update_min_quantity
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

###     FUNCTIONS ###
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

3) Explicit Cursor example & loop

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

4) BULK Operation (BULK COLLECT + FORALL)
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

6) Package: inventory_pkg
-- PACKAGE SPEC
CREATE OR REPLACE PACKAGE inventory_pkg IS
    PROCEDURE log_audit(p_table_name VARCHAR2, p_operation VARCHAR2, p_status VARCHAR2, p_error_msg VARCHAR2);
    PROCEDURE generate_reorders(p_mult IN NUMBER DEFAULT 2);
    FUNCTION is_reorder_needed(p_product_id IN NUMBER) RETURN VARCHAR2;
    FUNCTION get_stock_value(p_product_id IN NUMBER) RETURN NUMBER;
END inventory_pkg;
/
<img width="1528" height="776" alt="6) Package inventory" src="https://github.com/user-attachments/assets/2c9ab4a0-cb16-459f-9923-70acc1b6ac51" />

-- PACKAGE BODY
CREATE OR REPLACE PACKAGE BODY inventory_pkg IS

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

Test procedure proc_add_stock_entry:
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
1️⃣ Test proc_update_product_price

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

  -- Call procedure
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
4️⃣ Test proc_update_min_quantity
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






## Files in This Phase
- `GRPA_27129_Dorothy_SIMS_Database_Creation.sql`: Main creation script.
- `PHASE_IV_README.md`: This file.
