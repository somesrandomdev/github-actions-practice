

--------------------------------------------------
THE 4-PIECE MAP YOU CAN RE-USE FOREVER
--------------------------------------------------
1. **Trigger**   – when should it run?  
2. **Job**       – one unit of work (runs on a clean VM).  
3. **Step**      – a single command inside a job.  
4. **Artifact**  – pass files between jobs (optional).

Every workflow file is just:

```yaml
name: whatever
on: { trigger }
jobs:
  job-id:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4   # always start with this
      - run:  any-shell-command
      - uses: any-pre-made-action@v1
```

--------------------------------------------------
WHAT WE DID, STEP-BY-STEP
--------------------------------------------------
Exercise 1 – “Hello World”  
Trigger: every push  
Job: 1 step that prints text.  
Key point: the VM already has Ubuntu + bash; we only echoed.

Exercise 2 – Run Jest tests  
Trigger: push / pull-request to main  
Job: 4 steps  
1. checkout (download your repo)  
2. setup-node (install Node 22 + cache node_modules)  
3. npm ci (clean install)  
4. npm test (run Jest)  
Key point: `setup-node` with `cache: npm` saves ~30 s next run.

Exercise 3-A – Matrix  
Same as Exercise 2, but we added:

```yaml
strategy:
  matrix:
    node-version: [21, 22]
```

GitHub creates **two identical jobs**, one per value, and runs them **in parallel**.  
You can matrix anything: OS, Python version, browser, etc.

Exercise 3-B – Pipeline (sequential + parallel)  
We split work into **four jobs** and used `needs:` to enforce order:

lint  →  test-matrix  →  build  →  deploy  
            ↑  (parallel)          ↑
Only lint runs first; the two test jobs run side-by-side; build waits for **both**; deploy waits for build.  
We also used `actions/upload-artifact` and `download-artifact` to pass the `dist/` folder between jobs.

--------------------------------------------------
TINY TEMPLATES FOR YOUR NEXT PROJECT
--------------------------------------------------
A. Run tests on every PR (Node)

```yaml
name: CI
on:
  pull_request:
    branches: [ main ]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm test
```

B. Matrix across Node versions

```yaml
strategy:
  matrix:
    node: [18, 20, 22]
```

C. Pass files between jobs

```yaml
# job 1
- uses: actions/upload-artifact@v4
  with:
    name: myfiles
    path: dist/

# job 2
- uses: actions/download-artifact@v4
  with:
    name: myfiles
    path: dist/
```

--------------------------------------------------
COMMON TRAPS YOU ALREADY MET
--------------------------------------------------
- **Missing script** – npm run lint → add the script in package.json.  
- **Linter picky** – either fix the code or disable the rule; for labs, disabling is fine.  
- **YAML indentation** – 2 spaces, never tabs.  
- **Forgot checkout** – every job starts in an empty VM; always `uses: actions/checkout@v4` first.

--------------------------------------------------
CHEAT-SHEET YOU CAN KEEP
--------------------------------------------------
```
need a Linux command?        →  run: <bash>
need Node?                   →  uses: actions/setup-node@v4
need Python?                 →  uses: actions/setup-python@v5
need to save files?          →  uses: actions/upload-artifact@v4
need to read them later?     →  uses: actions/download-artifact@v4
need secrets (API keys)?     →  ${{ secrets.YOUR_SECRET }}  (add in repo Settings → Secrets)
```

That’s it.  
