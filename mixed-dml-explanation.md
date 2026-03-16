# Mixed DML in Salesforce – Real Experiments

Many Salesforce developers assume Mixed DML is just **"Setup vs Non-Setup objects."**

If both appear in the same transaction → ❌ ERROR  
If not → ✅ SAFE

Not exactly. Real scenarios break this assumption completely.

Here are experiments I ran 👇

---

## ⚡ Scenario 1 – Execute Anonymous

Update **User (Setup)** + **Account (Non-Setup)** in one anonymous transaction.

➡️ **Result:** ❌ Mixed DML Error. Classic. Everyone knows this one.

---

## ⚡ Scenario 2 – Trigger on User, updates Account

Many expect failure. Setup → Non-Setup mix.

➡️ **Result:** ✅ No Error.

⚠️ Is this because triggers are exempt? Or because **User is special**? Keep reading.

---

## ⚡ Scenario 3 – User trigger: update Account + insert GroupMember

Same trigger, two explicit DML statements.

➡️ **Result:** ❌ Mixed DML Error.

Transaction now has Setup + Non-Setup in same execution path.

---

## ⚡ Scenario 4 – Execute Anonymous

Update **User** + insert **GroupMember**, while **User trigger updates Account**.

➡️ **Result:** ❌ Mixed DML Error.

Same reason: setup + non-setup accumulate in one transaction.

---

# 🔬 Deeper – Trigger Direction Tests

## ⚡ Scenario 5 – User trigger → insert GroupMember  
Setup → Setup

➡️ **Result:** ✅ No Error.

---

## ⚡ Scenario 6 – Account trigger → insert/update User  
Non-Setup trigger → Setup DML

➡️ **Result:** ✅ No Error.

---

## ⚡ Scenario 7 – Account trigger → insert GroupMember  
Non-Setup trigger → Setup DML

➡️ **Result:** ❌ Mixed DML Error.

---

Scenarios **6 and 7** look identical in structure.

Different objects.  
Completely different results.

This is where it gets interesting.

---

# 🚨 The Real Finding – It Is Not About the User Object  
## It Is About Specific Fields

Salesforce documentation states:

> "This restriction exists because some sObjects affect the user's access to records in the org."

The official **setup-sensitive fields on User** are:

- ProfileId  
- UserRoleId  
- IsActive  

---

## 🔬 Field-Level Experiment

I tested this directly:

• Update **User.Title** in trigger → Account updates → ✅ No Error  

• Update **User.ProfileId** in trigger → Account updates → ❌ Mixed DML Error  

---

Same trigger.  
Same Account DML.  

One field change = **completely different outcome**.

---

### Why?

Because **ProfileId directly controls what a user can see and do in the org.**

Salesforce cannot safely run **Account DML under uncertain permission state**.

**Title** is just data.  
**ProfileId rewrites access.**

That is the difference.

---

# 🔥 Key Insight

Mixed DML is **not Setup vs Non-Setup.**

It is about whether your transaction touches **fields or objects that affect Salesforce’s permission evaluation pipeline.**

Triggers, fields, execution chains — all of it changes the outcome.

Because sometimes what should fail… **actually works.**

And sometimes what looks safe… **breaks production.**

---

# 💡 Bonus: Group vs GroupMember

- **Group → Non-Setup**
- **GroupMember → Setup**

They look related.  
They behave **completely differently.**

---

None of this is officially documented as behavior.

All of it is based on **real testing in Dev Console.**

---

Curious if others have hit surprising **Mixed DML behaviors in production.** 👀

#Salesforce  
#SalesforceDeveloper  
#Apex  
#MixedDML  
#SalesforceArchitecture
