# Conduit - AI-Augmented Defect Resolution (Assessment Showcase)

This repository showcases my completed full-stack solution and documentation for the **Conduit AI-Augmented Defect Resolution** assessment, featuring a **NestJS backend** and an **Angular frontend**.

## 🛠️ Technology Stack
* **Backend:** NestJS, TypeScript, MikroORM, MySQL
* **Frontend:** Angular 16, NgRx (State Management), TypeScript, Reactive Forms
* **AI Tooling:** Guided using **Cline** (AI coding agent) for Root Cause Analysis and precision refactoring.

---

## 🐛 Defects Resolved

### 1. Tag Splitting Bug (Problem 1)
* **The Defect:** When creating a new article with tags (e.g., `coding, testing`), the backend received `tagList` as a raw string and spread it into individual characters (`c-o-d-i-n-g`), causing single-letter badges to render on the UI.
* **The Solution:** 
  - Updated the frontend NgRx effect (`article-edit.effects.ts`) to parse, split by commas, trim whitespace, and transmit `tagList` as a clean string array (`string[]`).
  - Updated `ArticleService` in the NestJS backend to normalize tag input structures defensively on both `create` and `update`.

### 2. Popular Tags Persistence Bug (Problem 2)
* **The Defect:** Newly created tags (e.g., `testing`) did not appear under "Popular Tags" on the home page sidebar, even after a page refresh. This happened because tags were saved solely on the individual article record and never inserted into the global `Tag` database table queried by `GET /api/tags`.
* **The Solution:**
  - Enhanced `ArticleService.create()` and `update()` to cross-reference submitted tags against the global `Tag` repository.
  - Dynamically instantiate and persist new `Tag` entities for any unrecognized tags during the database transaction flush.

<img width="1535" height="819" alt="image" src="https://github.com/user-attachments/assets/da60a3b9-3960-428c-a54b-a40f09bf243d" />


---

## 📁 Repository Structure
* [`apps/backend/`](./apps/backend/) — NestJS backend API, entities, migrations, and services.
* [`apps/frontend/`](./apps/frontend/) & [`libs/`](./libs/) — Angular frontend application and feature libraries.
* [`root-causes.md`](./root-causes.md) — Detailed Root Cause Analysis for both defects.
* [`submission/`](./submission/) — Acceptance test screenshots and generated code patch.
