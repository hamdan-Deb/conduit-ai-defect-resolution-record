# Root Causes

Please copy-paste the final answer that you obtained from the AI for each question. The chat interface has a copy button that you can use to copy each message in Markdown format. Please do NOT include images or screenshots.

## Problem 1

**Problem**: The tags are broken up into individual characters on the post view page.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**: The underlying issue is a mismatch between how tags are collected on the frontend and how they are consumed on the backend. In `libs/articles/feature-article-edit/src/lib/article-edit.component.ts`, the article edit form uses a single text input for tags, so the tag data is handled as one comma-separated string in the UI. That raw value is then forwarded by `libs/articles/data-access/src/lib/+state/article-edit/article-edit.effects.ts` when publishing the article, instead of being normalized into a string array first.

On the backend, `apps/backend/src/article/article.service.ts` assumes `dto.tagList` is already an array and executes `article.tagList.push(...dto.tagList)`. When `dto.tagList` is actually a string such as `"angular, ngrx"`, the JavaScript spread operator expands the string into individual characters rather than logical tags. As a result, the persisted `tagList` becomes a character array, which is why the post view page displays broken tags instead of complete tag names.

The defect originates from the article creation/edit flow across `libs/articles/feature-article-edit/src/lib/article-edit.component.ts`, `libs/articles/data-access/src/lib/+state/article-edit/article-edit.effects.ts`, and the backend write logic in `apps/backend/src/article/article.service.ts`, with the immediate data corruption occurring in `ArticleService.create()` because of the spread operation on a raw string.


## Problem 2

**Problem**: New tags  are not shown on the home page under "Popular Tags", even after a page refresh.

**Question**: What is the underlying issue that causes this problem to occur and from which component (file) of this project does this issue originate?

**Answer**: The underlying issue is that newly entered tags are only stored inside the article record itself and are never persisted as standalone `Tag` entities. In `apps/backend/src/article/article.service.ts`, article creation and updates write values into the article's `tagList`, but the service does not create corresponding rows in the `Tag` table for any new tag names.

The home page's "Popular Tags" data is loaded through `TagService.findAll()` in `apps/backend/src/tag/tag.service.ts`. That service reads exclusively from the `Tag` repository, which means it can only return tags that already exist in the `Tag` table. Because `ArticleService` never creates or saves new `Tag` entities when users publish articles with new tags, those tags are absent from the `Tag` table and therefore never appear on the home page, even after a refresh.

The defect originates in `apps/backend/src/article/article.service.ts`, where tag persistence is incomplete. The visible symptom appears in the tags feed served by `apps/backend/src/tag/tag.service.ts`, but the root cause is that the article workflow never inserts newly created tag values into the `Tag` entity store.
