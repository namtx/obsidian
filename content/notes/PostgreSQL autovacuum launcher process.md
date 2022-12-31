---
title: "PostgreSQL autovacuum launcher process"
---

> “Assume that we delete a few records from a table. PostgreSQL does not immediately remove the deleted tuples from the data files. These are marked as deleted. Similarly, when a record is updated, it's roughly equivalent to one delete and one insert. The previous version of the record continues to be in the data file. Each update of a database row generates a new version of the row. The reason is simple: there can be active transactions, which want to see the data as it was before. As a result of this activity, there will be a lot of unusable space in the data files. After some time, these dead records become irrelevant as there are no transactions still around to see the old data. However, as the space is not marked as reusable, inserts and updates (which result in inserts) happen in other pages in the data file.”

>“The vacuum process marks space used by previously deleted (or updated) records as being available for reuse within the table. There is a vacuum command to do this manually. Vacuum does not lock the table.”

Excerpt From
PostgreSQL for Data Architects
This material may be protected by copyright.