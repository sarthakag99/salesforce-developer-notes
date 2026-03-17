Many Salesforce developers assume `Database.update(records, false)` always guarantees partial success  

If some records fail → others will still save ✅  

Sounds correct, right? Not always  

---

I came across this line in Salesforce documentation:

> “If the allOrNone parameter of a Database DML method is set to false and a before-trigger assigns an invalid value to a field, the partial set of valid records isn’t inserted.”

This felt counterintuitive  

So I tested it in Salesforce 👇  

---

## ⚡ Scenario 1 – Execute Anonymous (System Validation)

**Field:** NumberOfEmployees (max 8 digits)  

I updated multiple Accounts, but for one record I set:  
→ NumberOfEmployees = 100000000 (9 digits)  

### ➡️ Result:
- ❌ Only that record failed  
- ✅ All other records updated  
- ✔ Partial success worked  

---

## ⚡ Scenario 2 – Before Trigger (Same System Validation)

Same logic, but moved to a before update trigger  

### ➡️ Result:
- ❌ ALL records failed  
- ❌ No partial success  
- ❌ No records updated at all  

Even though only one record was invalid  

---

## ⚡ Scenario 3 – Execute Anonymous (Validation Rule)

**Validation Rule:**  
NumberOfEmployees = 100 → throw error  

Some records set to 100 (invalid)  
Others set to valid values  

### ➡️ Result:
- ❌ Only invalid records failed  
- ✅ Other records updated  
- ✔ Partial success worked  

---

## ⚡ Scenario 4 – Before Trigger (Validation Rule)

Trigger sets some records to:  
→ NumberOfEmployees = 100 (violates validation rule)  

### ➡️ Result:
- ❌ Only those records failed  
- ✅ Other records updated  
- ✔ Partial success STILL works  

---

## 🤯 So what’s really going on?

“Invalid value” here refers to **system validations**, not validation rules  

👉 Field length overflow  
👉 Data type mismatch  
👉 Required field violations  

---

## 🧠 Core Insight

**Order of execution:**
1. Before Trigger  
2. System Validations  
3. Validation Rules  
4. DML Save  

---

## 🔥 What changes behavior?

If invalid data (like 9-digit value in 8-digit field) is set in a before trigger  

👉 It fails during system validation  
👉 Entire batch is treated as invalid  

➡️ ❌ No partial success  
➡️ ❌ No records saved  

---

## ✅ But for validation rules:

👉 Run after system validation  
👉 Evaluated per record  

➡️ ✔ Partial success works  

👉 Because:  
- Validation rule = handled error  
- System validation failure (in trigger) = unhandled DML exception  

---

## 📩 Apex Exception Email Behavior

- Validation rule failure → ❌ No email  
- Partial success (SaveResult) → ❌ No email  
- System validation via anonymous → ❌ No email  
- System validation via trigger → 📩 Email sent  

---

## ⚠️ Important Observation

In Scenario 2:

- ❌ No exception in logs  
- ❌ Transaction does NOT halt mid-way  
- ❌ All logic executes  

Yet:

👉 Nothing gets committed  

---

## 💡 Final Takeaway

`allOrNone = false` does NOT guarantee partial success  

It only works when:  
👉 errors are record-level and occur during DML processing  

But if:  
👉 a before-trigger introduces a system-level invalid value  

➡️ the entire transaction can fail silently (no records saved)  

---

This was a surprising behavior for me. Curious how many people have actually tested this 👇  

---

#Salesforce #SalesforceDeveloper #SalesforceArchitecture #Apex #Trailblazer
