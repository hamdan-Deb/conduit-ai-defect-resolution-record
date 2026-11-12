# Root Causes - Conduit Defect Resolution

## Problem 1

**Problem**: The tags are broken up into individual characters on the post view page.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**:
The underlying issue originates from how tag inputs are captured and passed from the frontend to the backend without proper type transformation, coupled with how the backend handles iterable string values:

1. **Frontend Origin (`libs/articles/feature-article-edit/src/lib/article-edit.component.ts` and `libs/articles/data-access/src/lib/+state/article-edit/article-edit.effects.ts`)**:
   The article edit form defines the `tagList` field as a text input. When entering a comma-separated list of tags (e.g., `"coding, testing"`), the form stores it as a raw `string`. In `publishArticle$`, data was transmitted without splitting the string into an array of strings (`string[]`).

2. **Backend Consumption (`apps/backend/src/article/article.service.ts`)**:
   In `ArticleService.create()`, `article.tagList.push(...dto.tagList)` was used. Because strings in JavaScript are iterable, the spread syntax (`...`) iterated over each character of the string rather than elements of an array, pushing individual characters (`c, o, d, i, n, g...`) into `article.tagList`.

## Problem 2

**Problem**: New tags are not shown on the home page under "Popular Tags", even after a page refresh.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**:
The underlying issue originates from `apps/backend/src/article/article.service.ts` due to a decoupling between article creation and global tag persistence:

1. **Architecture & Tag Retrieval**:
   The home page sidebar retrieves tags via `GET /api/tags` (`TagService.findAll()`), which queries the `Tag` entity table in MySQL.

2. **Backend Issue in `ArticleService.create()`**:
   When a new article was created, tags were saved only into the article's local `tagList` property on the `Article` entity. The service never checked or persisted new tags into the dedicated `Tag` table.

3. **Consequence**:
   Because new tags (such as `"testing"`) were never written into the database `tag` table, `TagService.findAll()` only returned pre-existing seed tags. The fix requires querying existing tags, identifying missing ones, instantiating new `Tag` entities, and persisting them before transaction flush.
