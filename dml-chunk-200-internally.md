# "My trigger only has 1 SOQL and 1 DML. I'm well within governor limits."

That’s what I believed too.

Until production proved me wrong.

After 4 years of writing Apex, I've learned this is one of the most dangerous assumptions a developer can make. Here is why your "safe" code might be secretly failing at scale. 👇

---

When 10,000 records hit your trigger, the platform splits them into 200-record chunks.
👉 Your trigger fires 50 separate times.
👉 All 50 run inside the **SAME transaction**.

That single line of code, "1 SOQL and 1 DML," just became **50 SOQLs and 50 DMLs**. You didn't write bad code; you just didn't design it for heavy bulkification.

---

## The Static Boolean Trap

To prevent trigger recursion under these loads, devs often use a common shortcut, a Static Boolean:

```apex
if(hasRun) return;
hasRun = true;
```

**This is a dangerous trap.** Static variables persist for the entire transaction, crossing all chunks.

| Chunk | Records | hasRun | Result |
|---|---|---|---|
| Chunk 1 | 1–200 | false | ✅ Processes perfectly. `hasRun` becomes true. |
| Chunk 2 | 201–400 | true | 💥 Hits `return` immediately. Logic never runs. |
| Chunk 3 | 401–600 | true | 💥 Hits `return` immediately. Logic never runs. |
| ... | ... | true | 💥 |
| Chunk 50 | 9,801–10,000 | true | 💥 Hits `return` immediately. Logic never runs. |

> ❌ No errors are thrown. No logs are flagged. Just **9,800 records silently failing to process.**

---

## The Fix

### Level 1 Fix — Static Set of IDs

Use a `Static Set<Id>` to track the exact records processed, rather than a blanket true/false flag. This ensures every record is evaluated.

```apex
public static Set<Id> processedIds = new Set<Id>();
```

Every chunk processes its own new IDs → all 10,000 records handled.
Same ID appearing again (actual recursion) → already in Set → skipped. ✅

### Level 2 Fix — Enterprise Scale (Trigger Framework)

Mature orgs should move away from standalone triggers entirely. All logic should route through a centralized **Trigger Framework** to manage bypasses, execution order, and context cleanly.

A Set of IDs is a great band-aid, but the real architectural question is:

> **Why is your architecture forcing the trigger to recurse in the first place?**

---

## Key Takeaways

- The platform processes triggers in **200-record chunks**
- Static variables persist for the **entire transaction** — across all chunks
- `Static Boolean` = silently drops records at bulk scale ❌
- `Static Set<Id>` = processes every record, blocks true recursion ✅
- Always test with **200+ records**, not just 1 or 2

---

Fellow devs: Have you ever caught a "silent data loss" bug caused by a Static Boolean in your org? 👇

---

`#Salesforce` `#Apex` `#SalesforceDeveloper` `#SalesforceArchitecture` `#SFDC`
