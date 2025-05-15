
# A Week in the Life of an SAP Consultant 


## **What is SAP?**
SAP is a **business software system** used by big companies to manage:
- Money (Finance)
- Buying/Selling (Procurement/Sales)
- Manufacturing (Production Planning)
- Payroll (HR)


---

## **📅 Monday: Preparing a System Update**
**Goal:** Get a new purchasing system ready to launch.

### **7:30 AM - Team Meeting**
**What Happens:**
- The SAP team meets to discuss the upcoming changes.
- They check which parts of the system will be updated (called **"transport requests"**).
- They agree on when to do the update (late at night to avoid disrupting work).

**Example Command:**
```bash
STMS_IMPORT_CHECK -transport CHG123456
```
*This checks if all the updates will work together.*

### **10:00 AM - Testing Vendor Data**
**Problem:**  
The company is adding new suppliers (vendors) to the system, but some are missing tax codes.

**Solution:**  
The consultant writes a small program to load the data correctly.

```abap
DATA: lt_vendors TYPE TABLE OF lfa1.
APPEND VALUE #( lifnr = 'V10022' name1 = 'BrexitParts Ltd' ) TO lt_vendors.
CALL FUNCTION 'BAPI_VENDOR_CREATE' 
  EXPORTING vendor_data = lt_vendors.
```
*This code adds a new vendor named "BrexitParts Ltd" into SAP.*

**Issue Found:**  
Some vendors failed because they didn’t have tax codes.  
**Fix:** The consultant updates tax settings in **Transaction Code (TCODE) OBYZ**.

---

## **📅 Tuesday: Fixing Problems After the Update**
**Goal:** Solve issues from last night’s update.

### **8:00 AM - Emergency Fix**
**Problem:**  
Purchase orders (POs) are stuck waiting for approval.

**How It’s Fixed:**
1. The consultant checks the **workflow logs** (using **TCODE: SWU3**).
2. Finds out the system doesn’t know how to handle UK suppliers after Brexit.
3. Updates the approval rule:

```abap
IF iv_country = 'GB' AND iv_value > 50000.
  cv_approver = 'APROVER_UK'.
ENDIF.
```
*This means: "If the supplier is from the UK and the order is over €50,000, send it to a special approver."*

### **2:00 PM - Training Warehouse Staff**
**What Happens:**  
The consultant teaches warehouse workers how to use a new **mobile app** to scan barcodes and check stock.

**Feedback:**  
- The app works, but they need **offline mode** (for when Wi-Fi is bad).  
- The consultant adds this to the improvement list.

---

## **📅 Wednesday: Improving the System**
**Goal:** Make the system smarter for Brexit.

### **9:00 AM - Updating the MRP System**
**What is MRP?**  
**Material Requirements Planning (MRP)** helps companies decide:
- What materials to buy
- When to order them
- How much to store

**Problem:**  
After Brexit, UK suppliers now have extra **taxes (tariffs)**.  
The system needs to account for this.

**Solution:**  
The consultant adds a new rule to the MRP system:

```abap
METHOD calculate_mrp.
  DATA(ld_tariff) = zcl_brexit_tariff=>get_duty( 
    iv_matnr = ms_material-matnr 
    iv_werks = ms_material-werks ).
  
  IF ld_tariff > 100. " If tariff is high
    REDUCE order_qty BY 10%. " Buy 10% less from UK
  ENDIF.
ENDMETHOD.
```
*This means: "If importing from the UK is too expensive, order less from them."*

**Testing:**  
The consultant runs a test with 500 materials to make sure it works.

---

## **📅 Thursday: Connecting SAP to the Cloud**
**Goal:** Automate sending purchase orders (POs) to AWS.

### **10:00 AM - Setting Up Cloud Integration**
**Problem:**  
The company wants to store PDF copies of POs in **AWS S3** (cloud storage).

**Solution:**  
The consultant sets up a **SAP CPI (Cloud Platform Integration)** flow:

```xml
<!-- CPI iflow.xml snippet -->
<AmazonS3Connection>
  <Bucket>autoparts-po</Bucket>
  <Region>eu-central-1</Region>
  <AccessKey>${property.AWS_KEY}</AccessKey>
</AmazonS3Connection>
```
*This tells SAP where to send the files.*

**Issue:**  
Large POs (over 5MB) were failing.  
**Fix:**  
- Split big files into smaller chunks.  
- Increase timeout from 25s → 120s.

---

## **📅 Friday: Checking Results & Planning**
**Goal:** See how well the changes worked.

### **9:00 AM - Updating Reports**
**What Happens:**  
The consultant checks **SAP Analytics Cloud (SAC)** dashboards to see improvements:

| Metric | Before | After |
|--------|--------|-------|
| MRP Run Time | 4.2 hours | 55 min |
| PO Approvals | 3 days | 4 hours |

**New Alert System:**  
```sql
CREATE CALCULATION VIEW "ZSTOCK_ALERT" AS 
SELECT matnr, werks, labst 
FROM mard 
WHERE labst < (SELECT min_stock FROM zstock_rules WHERE matnr = mard.matnr);
```
*This warns when stock is too low.*

### **1:00 PM - Team Review**
**Lessons Learned:**  
✔ **Test Brexit rules earlier next time**  
✔ **Monitor cloud integration memory usage**  

---

## **🛠️ Tools Used This Week**
| Task | Tools |
|------|-------|
| **Development** | ABAP (SAP’s programming language), Eclipse |
| **Cloud Integration** | SAP CPI, AWS S3 |
| **Testing** | SAP Solution Manager, ECATT |

---
