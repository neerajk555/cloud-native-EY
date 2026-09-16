# Lab 3.2: Write a Dockerfile

**Module:** 3 — Containerization with Docker & Amazon ECR
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (all local)

## Objective
Write your first Dockerfile for a tiny Node.js web app, applying basic Dockerfile best practices (small base image, layer caching, non-root user).

## Prerequisites
- Completed Lab 3.1

## Steps

1. **Create a project folder:**
   ```bash
   mkdir hello-cloudnative-app && cd hello-cloudnative-app
   ```
2. **Create `app.js`:**
   ```javascript
   const http = require('http');
   const server = http.createServer((req, res) => {
     res.writeHead(200, { 'Content-Type': 'text/plain' });
     res.end('Hello from a cloud native container!\n');
   });
   server.listen(3000, () => console.log('Server running on port 3000'));
   ```
3. **Create `package.json`:**
   ```json
   {
     "name": "hello-cloudnative-app",
     "version": "1.0.0",
     "main": "app.js",
     "scripts": { "start": "node app.js" }
   }
   ```
4. **Create a `Dockerfile`** (best practices: small base image, copy dependency files first for layer caching, run as non-root):
   ```dockerfile
   FROM node:20-alpine

   WORKDIR /usr/src/app

   COPY package*.json ./
   RUN npm install --production

   COPY . .

   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser

   EXPOSE 3000
   CMD ["node", "app.js"]
   ```
5. **Create a `.dockerignore`** (avoid copying unnecessary files into the image):
   ```
   node_modules
   npm-debug.log
   .git
   ```

## Expected Result / Validation
Your folder should now contain: `app.js`, `package.json`, `Dockerfile`, `.dockerignore`. Run:
```bash
cat Dockerfile
```
and confirm it matches step 4 exactly — this file will be built in the next lab.

## Cleanup
Nothing to clean up — no containers or images were built yet (that happens in Lab 3.3). Keep this folder; you'll reuse it in Labs 3.3 and 3.4.

## Troubleshooting
- **`npm install` will fail later if package.json has a syntax error** → Validate with `cat package.json | python3 -m json.tool` (or any JSON validator) before moving on.
