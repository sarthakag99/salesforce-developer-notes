# 🚀 Salesforce SOQL Governor Limits Deep Dive

🚨 Not every SOQL query counts toward the 100 query limit.

Some don't count at all.

Some use a completely different limit.

And some still hit limits in unexpected ways.

---

If you treat every query the same, you'll miss how Salesforce actually enforces governor limits.

📘 What does Salesforce say?

> "In a SOQL query with parent-child relationship subqueries, each parent-child relationship counts as an extra query. These types of queries have a limit of three times the number for top-level queries. This limit doesn't apply to custom metadata types. In a single Apex transaction, custom metadata records can have unlimited SOQL queries."

Here are a few insights I explored 👇

---

⚡ 1. Parent-Child Subqueries behave differently

▶️ `SELECT Id, (SELECT Id FROM Contacts) FROM Account`

- Main query → counts toward 100 SOQL limit
- Each subquery → counts as an Aggregate Query

👉 Governed by a separate limit → 300 (using `Limits.getLimitAggregateQueries()`)

Many people assume it's "free" — it's not.
It just falls under a different limit bucket.

💡 How does this actually work?

▶️ `SELECT Id, (SELECT Id FROM Contacts) FROM Account`

Counts as:
- 1 query → (out of 100)
- 1 aggregate query → (out of 300)

⚠️ Important clarification (very commonly misunderstood)

"Aggregate Query" here does NOT mean:
- `COUNT()`
- `SUM()`
- `AggregateResult`

👉 Instead, Salesforce uses this term for:
- Parent-child subqueries
- Relationship queries

So even without aggregation functions,
it still counts as an Aggregate Query.

---

⚡ 2. Row limit still applies (and can surprise you)

Even though subqueries don't hit the 100 limit:
- All records fetched (parent + child) → Count toward 50,000 row limit

👉 Example:
1 Account
50,000 Contacts
➡️ You still hit the limit ❌

---

⚡ 3. Custom Metadata – special behavior

You can query Custom Metadata in multiple ways:
- SOQL SELECT
- `getAll()`
- `getInstance()`

👉 Key differences:

- SOQL:
  - ❌ Does NOT count toward 100 SOQL limit
  - ✔ Counts toward 50k row limit

- `getAll()` / `getInstance()`:
  - ❌ No SOQL limit
  - ❌ No row limit

👉 Effectively allows unlimited queries (within transaction scope)

---

⚡ 4. Custom Settings – not the same as Metadata

This is where many developers get confused.

- Using SOQL:
  - ✔ Counts toward 100 SOQL limit
  - ✔ Counts toward 50k row limit

- Using `getAll()` / `getInstance()`:
  - ❌ Does NOT count toward 100 limit
  - ❌ Does NOT count toward 50k row limit

👉 Same methods, different behavior than SOQL.

---

💡 Key Insight

Not all "queries" are treated equally in Salesforce.
- Subqueries → Separate limit (Aggregate Queries)
- Metadata → Relaxed SOQL limits
- Settings → Standard SOQL limits
- `getAll()`/`getInstance()` → Cache-based, no limits

👉 Salesforce gives you:
Performance path (cache) vs Flexible path (SOQL)

---

🚀 Takeaway

If you treat all queries the same, you'll hit limits unexpectedly.
If you understand how each behaves,
you can design far more efficient Apex code.

---

#Salesforce #Apex #SOQL #GovernorLimits #SalesforceDeveloper
