+++
title = "Cherry-picking commits in Git"
date = 2024-09-09
updated = 2024-09-09
type = "post"
in_search_index = true
[taxonomies]
TIL-Tags = ["git"]
+++


I did cherry-pick commits many times before. But I kept forgetting how simple it is.

Cherry picking is fairly simple. it works just like git merge but with individual commits. Let's say staging is ahead of main by 5 commits. Now if you want to apply only those changes from latest two commits, you can checkout to main and run 

```bash
git cherry-pick <first-commit-hash-on-staging> <second-commit-hash-on-staging>
```

That's it, you now have the changes only from these 2 commits applied to your main with the same commit metadata. Instead of directly pushing to main, You can create a new branch from this by gco -b "cherry-picked-brach"  and then git push origin "cherry-picked-brach"  and then create a PR on GH.