# Automated Precheck Source Code  

![Project Initiation Phase](../../img/ProjectInitiationPhase.png)  

## 1. Overall Architecture (Recommended)

    Developer (VS Code)
     ├─ Pre-commit hooks
     │   ├─ clang-format
     │   ├─ clang-tidy / cppcheck
     │   └─ unit test (optional)
     │
     └─ git commit / push
            ↓
    GitLab (LAN)
     ├─ Server-side hooks (optional hard gate)
     └─ GitLab CI Pipeline
         ├─ build
         ├─ static analysis
         └─ test

This gives you:

*   **Automatic review BEFORE commit**
*   **Enforced quality BEFORE merge**
*   **Zero internet dependency**

***

## 2. Pre-commit Review (Fast Feedback, Local)

### 2.1 Install Required Tools (Ubuntu)

```bash
sudo apt install -y \
  clang-format \
  clang-tidy \
  cppcheck \
  git \
  cmake
```

***

### 2.2 Use `pre-commit` Framework (Recommended)

It manages hooks cleanly and consistently.

```bash
pip install pre-commit
```

In your repo root:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/mirrors-clang-format
    rev: v17.0.6
    hooks:
      - id: clang-format
        files: \.(c|cpp|h|hpp)$

  - repo: local
    hooks:
      - id: clang-tidy
        name: clang-tidy
        entry: clang-tidy
        language: system
        files: \.(c|cpp)$
        args: ["-p=build"]

      - id: cppcheck
        name: cppcheck
        entry: cppcheck
        language: system
        files: \.(c|cpp|h|hpp)$
        args:
          - --enable=warning,performance,portability
          - --inline-suppr
          - --error-exitcode=1
```

Install hooks:

```bash
pre-commit install
```

✅ Result:

*   Auto-format
*   Static analysis
*   Commit blocked on violations

***

## 3. VS Code Integration (Optional but Powerful)

### 3.1 Tasks.json for Manual “Review”

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Static Analysis",
      "type": "shell",
      "command": "cppcheck --enable=warning,performance,portability src",
      "problemMatcher": ["$gcc"]
    }
  ]
}
```

Now you can:

*   Run code review with **Ctrl+Shift+B**
*   See issues inline

***

## 4. Server-side Enforcement (GitLab – Offline)

### 4.1 GitLab CI (Highly Recommended)

Even without internet, GitLab CI works perfectly inside LAN.

`.gitlab-ci.yml`

```yaml
stages:
  - build
  - analyze
  - test

build:
  stage: build
  script:
    - cmake -S . -B build
    - cmake --build build

clang_tidy:
  stage: analyze
  script:
    - clang-tidy src/*.cpp -p build
  allow_failure: false

cppcheck:
  stage: analyze
  script:
    - cppcheck --enable=warning,performance,portability \
      --error-exitcode=1 src

unit_test:
  stage: test
  script:
    - ctest --test-dir build
```

✅ Result:

*   No merge allowed if analysis fails
*   Automatic “review comments” in MR (GitLab UI)

***

## 5. GitLab Merge Request as Review Gate

Use **Protected Branches**:

1.  Settings → Repository → Protected branches
2.  Protect `main` / `develop`
3.  Require:
    *   Pipeline success
    *   (Optional) 1 human approval

This turns CI results into a **mandatory automated reviewer**.

***

## 6. (Optional) Server-side Git Hooks (Hard Stop)

If you want **absolute enforcement**, add a server hook:

```bash
# /var/opt/gitlab/git-data/repositories/<group>/<repo>.git/hooks/pre-receive
#!/bin/bash
echo "Server-side check: CI must be used"
exit 1
```

⚠️ Usually **CI + protected branches** is safer and more flexible.

***

## 7. About GitHub Copilot (Business)

Important clarification:

*   Copilot **does not review code automatically**
*   It is:
    *   ✅ Editor-time suggestion
    *   ❌ Not enforceable
    *   ❌ Not auditable
*   In an **air‑gapped GitLab**, Copilot cannot:
    *   Access repo context beyond open files
    *   Perform policy checks

✅ Best practice:  
**Use Copilot only for authoring. Use static analyzers for review.**

***

## 8. Recommended Tool Stack Summary

| Purpose            | Tool         |
| ------------------ | ------------ |
| Formatting         | clang-format |
| Static analysis    | clang-tidy   |
| Lightweight checks | cppcheck     |
| Build correctness  | CMake        |
| Automation         | pre-commit   |
| Enforcement        | GitLab CI    |
| Human review       | GitLab MR    |

***

## 9. Typical Developer Flow (Final)

    Edit code (VS Code + Copilot)
     ↓
    git commit
     → pre-commit auto review (local)
     ↓
    git push
     → GitLab CI analysis
     ↓
    Merge Request blocked until CI passes


---

# License

![BY NC SA](../../../img/Cc-by-nc-sa.png)  
Automated Precheck Source Code © 2026 by Jen Yuan Pan is licensed under the `Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International`.  

For the complete license terms, rights and obligations, restrictions on use, and other applicable conditions, please refer to the [Full License Terms](https://github.com/retamotome/AgileSoftwareProjectGuide/blob/main/LICENSE.md). By using this work, you acknowledge that you have read, understood, and agree to comply with such terms.  