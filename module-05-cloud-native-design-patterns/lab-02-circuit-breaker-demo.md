# Lab 5.2: Circuit Breaker Pattern Demo

**Module:** 5 — Cloud Native Design Patterns
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (runs entirely locally with Node.js — no AWS resources)

## Objective
Understand the Circuit Breaker pattern by running a small local Node.js script that simulates a flaky downstream service and observes a circuit breaker "open" to stop hammering it, then "close" again once it recovers.

## Prerequisites
- Node.js installed locally (v18+): https://nodejs.org/

## Steps

1. **Create a project folder:**
   ```bash
   mkdir circuit-breaker-demo && cd circuit-breaker-demo
   npm init -y
   npm install opossum
   ```
   *(`opossum` is a popular, lightweight open-source circuit breaker library for Node.js.)*

2. **Create `flaky-service.js`** — simulates a downstream service that fails 70% of the time:
   ```javascript
   function callFlakyService() {
     return new Promise((resolve, reject) => {
       setTimeout(() => {
         if (Math.random() < 0.7) {
           reject(new Error("Downstream service failed"));
         } else {
           resolve("Downstream service responded OK");
         }
       }, 100);
     });
   }
   module.exports = callFlakyService;
   ```
3. **Create `index.js`** — wraps the flaky service with a circuit breaker:
   ```javascript
   const CircuitBreaker = require('opossum');
   const callFlakyService = require('./flaky-service');

   const options = {
     timeout: 500,
     errorThresholdPercentage: 50,
     resetTimeout: 3000
   };

   const breaker = new CircuitBreaker(callFlakyService, options);

   breaker.on('open', () => console.log('🔴 Circuit OPEN — blocking calls to protect the system'));
   breaker.on('halfOpen', () => console.log('🟡 Circuit HALF-OPEN — testing if service recovered'));
   breaker.on('close', () => console.log('🟢 Circuit CLOSED — calls flowing normally'));

   let i = 0;
   const interval = setInterval(() => {
     i++;
     breaker.fire()
       .then(result => console.log(`Call ${i}: SUCCESS - ${result}`))
       .catch(err => console.log(`Call ${i}: FAILED - ${err.message}`));

     if (i >= 25) clearInterval(interval);
   }, 300);
   ```
4. **Run it:**
   ```bash
   node index.js
   ```

## Expected Result / Validation
Within the first few seconds you should see several `FAILED` calls, then a `🔴 Circuit OPEN` message — at which point calls start failing **instantly** with a "Breaker is open" message instead of waiting on the (still failing) downstream service. After the `resetTimeout` (3 seconds), you should see `🟡 Circuit HALF-OPEN`, followed by either `🟢 Circuit CLOSED` (if a test call succeeds) or another `🔴 Circuit OPEN` (if it still fails) — demonstrating the full state machine.

## Cleanup
No AWS resources were created. Clean up your local files if desired:
```bash
cd ..
rm -rf circuit-breaker-demo
```

## Troubleshooting
- **Circuit never opens** → Random failure is probabilistic; if you get lucky with successes, just re-run `node index.js` — over 25 calls at 70% failure rate, it will open in nearly all runs.
- **`opossum` install fails** → Confirm Node.js and npm are correctly installed with `node -v` and `npm -v`.
