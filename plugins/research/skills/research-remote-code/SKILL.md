---
name: research-remote-code
description: Techniques for retrieving and investigating remote codebases and their documentation. TRIGGER when: user provides a GitHub/GitLab URL to look at, asks to follow conventions from a remote repo, asks "how does package/library X implement/handle Y", or references any external repository's code, structure, or patterns. Examples: "follow the conventions in https://github.com/...", "look at how repo X does Y", "check this GitHub link".
---

## Accessing code from remote repositories

When you need to look at code in a remote repository (GitHub, GitLab, etc.):

1. Shallow clone it: `git clone --depth 1 <url> /tmp/<repo-name>`
2. Use Read and Grep to examine the code
3. Clean up when done: `rm -rf /tmp/<repo-name>`

Do NOT use WebFetch or gh api to read files from remote repos; clone locally instead.

When asked to follow conventions or patterns from a remote repo, clone it, read the relevant files, then apply those patterns to the local project.