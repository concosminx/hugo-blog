1. Use `hugo list drafts` to identy all drafts
2. Extract drafts that have `section: posts` and `draft: true` 
3. Sort the results by date (oldest first)
4. Display the list of draft posts with:
   - Date
   - Title
   - Path

Background:
Drafts are pages that need more work before publication. Drafts are identified by 'draft: true' in the front matter. 

The command `hugo list drafts` will list all drafts. Its output is in CSV format. The first line of output identifies the name of each field. Every subsequent line represents a draft.

Example Output: 

```
path,slug,title,date,expiryDate,publishDate,draft,permalink,kind,section
```
