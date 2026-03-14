# Mixed DML in Salesforce – Real Scenarios

Many Salesforce developers assume Mixed DML is simply about “Setup vs Non-Setup objects”.

If both appear in the same transaction → ERROR.  
If they don’t → SAFE.

Sounds simple, right?

Not exactly.  
The platform behavior is far more nuanced, and some real scenarios actually break this common assumption.

Here are a few experiments I ran 👇

---

## ⚡ Scenario 1 – Execute Anonymous

Updating a Setup object (User) and a Non-Setup object (Account) in the same anonymous transaction.

➡️ **Result:** ❌ Mixed DML Error  

This is the classic example everyone knows.

---

## ⚡ Scenario 2 – Trigger on Setup Object

A User trigger updates an Account record.  
Many developers assume this will fail because it mixes Setup and Non-Setup objects.

➡️ **Result:** ✅ No Error. Both records update successfully.

This alone breaks the common belief that any Setup → Non-Setup combination fails.

---

## ⚡ Scenario 3 – Multiple operations inside the User trigger

A User trigger performs two operations:

- Updates Account (Non-Setup)
- Creates GroupMember (Setup)

(Both happening from the same User trigger, either through helper methods or a record-triggered Flow on User)

➡️ **Result:** ❌ Mixed DML Error  

Because the transaction now tries to modify Setup(GroupMember) + Non-Setup(Account) objects together within the same execution path.

---

## ⚡ Scenario 4 – Anonymous + Trigger Chain

From Execute Anonymous:

Update User + create GroupMember, while the User trigger updates Account.

➡️ **Result:** ❌ Mixed DML Error again  

Because the transaction now tries to modify Setup(GroupMember) + Non-Setup(Account) objects together within the same execution path.

---

## 🔥 Key Insight

Mixed DML isn’t just about touching Setup and Non-Setup objects together.

It’s about how Salesforce evaluates the transaction boundary and execution chain. Triggers, flows, and direct operations can change the outcome completely.

Understanding this subtle behavior is crucial when designing User provisioning logic, automation chains, and complex trigger flows.

Because sometimes what should fail… actually works.  
And sometimes what looks safe… breaks production.

---

## 💡 Bonus Insight (often overlooked)

Many people assume Group and GroupMember behave the same, but they don’t.

- **Group → Non-Setup Object**
- **GroupMember → Setup Object**

This difference alone can create unexpected Mixed DML errors when automation interacts with public groups or queues.

Understanding these subtle platform behaviors is critical when designing user provisioning flows, automation chains, and trigger frameworks.

Because in Salesforce…

what looks simple on the surface often hides deeper platform rules. 🚀

---

Curious if others have encountered surprising Mixed DML behaviors in real projects. 👀

#Salesforce  
#SalesforceDeveloper  
#Apex  
#MixedDML  
#SalesforceArchitecture  
#SalesforceTips  
#SalesforceCommunity  
#DeveloperLife  
#SalesforceLearning

