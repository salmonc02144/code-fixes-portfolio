# Case Study: React Memory Leak — E-Commerce Cart Crash

**Client:** Fashion e-commerce, 50K daily active users  
**Issue:** Cart page crashes after 3-5 minutes, browser tab freezes, 4GB RAM spike  
**Language:** JavaScript (React 18)  
**Impact:** 12% cart abandonment, $8K daily revenue loss  

## Problems Identified

1. **Missing useEffect Cleanup:** setInterval for countdown timer not cleared on unmount
2. **Event Listener Accumulation:** window.addEventListener('resize', handler) added every render, never removed
3. **State Updates on Unmounted Component:** Async fetch resolves after user navigates away, triggers warning cascade
4. **Closure Stale References:** useCallback dependency array missing cartItems, causing re-renders with old data

## Fixes Applied

| Issue | Before | After |
|-------|--------|-------|
| Timer leak | setInterval(() => setTimeLeft(t => t - 1), 1000) | useEffect(() => { const id = setInterval(...); return () => clearInterval(id); }, []) |
| Resize leak | window.addEventListener('resize', handleResize) in render | useEffect(() => { window.addEventListener(...); return () => window.removeEventListener(...) }, []) |
| Unmounted state | fetch('/api/cart').then(data => setCart(data)) | AbortController signal, check isMounted ref before setCart |
| Stale closure | useCallback(() => calculateTotal(cartItems), []) | useCallback(..., [cartItems]) |

## Result

- **RAM usage:** 4.2GB → 180MB (23x reduction)
- **Tab uptime:** 3 minutes → indefinite
- **Cart completion rate:** 88% → 96% (+8 percentage points)
- **Revenue recovery:** $8K/day → $0 loss

**Turnaround:** 2 hours diagnosis, 1 hour fix  
**Status:** Resolved, monitoring for 14 days
