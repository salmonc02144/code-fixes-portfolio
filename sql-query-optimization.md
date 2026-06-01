# Case Study: SQL Query Optimization — E-Commerce Reporting

**Client:** E-commerce platform (2M+ orders)  
**Issue:** Daily revenue reports timing out, running 45+ seconds, failing overnight  
**Language:** SQL (PostgreSQL)  
**Impact:** Client losing automated reporting, manual workarounds costing 4 hours/day  

## Problems Identified

1. **Full Table Scans:** SELECT * FROM orders WHERE DATE(created_at) = '2026-05-01' — function on column prevents index use
2. **N+1 Query Pattern:** Looping order IDs to fetch customer data in separate queries
3. **Missing Indexes:** No composite index on (created_at, status) for filtered aggregations
4. **Redundant Joins:** Joining full customers table when only name and email needed

## Fixes Applied

| Issue | Before | After |
|-------|--------|-------|
| Date filtering | WHERE DATE(created_at) = '2026-05-01' | WHERE created_at >= '2026-05-01' AND created_at < '2026-05-02' |
| Customer lookup | 2000 individual SELECT * FROM customers WHERE id = ? | Single JOIN with customers(id, name, email) subquery |
| Aggregation | GROUP BY on unindexed calculated field | Pre-aggregated CTE with indexed range scan |
| Index strategy | None on (created_at, status, total) | Composite index idx_orders_created_status_total |

## Result

- **Execution time:** 45.2s → 0.28s (161x improvement)
- **CPU load:** 78% → 12% during report window
- **Report reliability:** 23% failure rate → 0% over 30 days
- **Client savings:** 4 hours/day manual work eliminated

**Turnaround:** Single session  
**Status:** Resolved, monitoring for 30 days
