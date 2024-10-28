---
layout: page
title: Commit Messages
parent: Best Coding Practices
description: Best practices for writing a clear commit message
nav_order: 1
tags: [commit, devops, git, version-control, pull-request]
---

# Commit Message Guidelines: Keeping Your Code History Clear
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Writing clear and informative commit messages is essential for any developer working on a project. Effective messages act as a roadmap, helping everyone understand the "why" behind code changes. This not only improves collaboration but also makes maintaining and revisiting code in the future much easier.

### Structure of a Good Commit Message:

A well-written commit message consists of two parts:

1. **A concise title (under 72 characters):**
    
    - Written in the **imperative mood** (e.g., "Implemented user authentication" instead of "Implemented user authentication functionality").
    - Briefly summarize the change and its purpose.
    - **Avoid** vague titles like "Changes to X" or "Fixed stuff".

2. **Detailed message body (optional):**
    
    - Provide context and additional details about the changes made.
    - Explain the motivation behind the change (e.g., fixing a bug, implementing a new feature).
    - Mention relevant issue references (e.g., "Fixes #123").
    - Describe potential limitations or trade-offs of the implemented solution.
    - Use clear and concise language, avoiding unnecessary jargon.
    - Maintain readability with proper formatting and line breaks.

**Example:**

```
Refactored login flow for improved security (Fixes #145)

- Implemented secure password hashing using bcrypt.
- Added validation checks for username and password length.
- Removed unnecessary session handling logic.

This commit addresses security concerns raised in issue #145 by improving the login flow. Sensitive data is now protected with strong hashing, and user input is validated to prevent potential vulnerabilities.
```

### Benefits of Clear Commit Messages:

- **Improved Collaboration:** Clear messages allow team members to understand code changes quickly, facilitating effective collaboration.
- **Enhanced Code Maintainability:** Informative messages make it easier to understand the codebase's evolution and make future modifications.
- **Efficient Code Review:** Detailed descriptions help reviewers grasp the changes and provide constructive feedback.
- **Future Reference:** Well-written messages act as valuable documentation for revisiting code changes even years later.

### Additional Tips:

- **Use consistent formatting:** Consider adopting a team-wide style guide for commit messages to ensure uniformity.
- **Separate commits for logical changes:** Break down large changes into smaller, more focused commits with clearer messages.
- **Focus on the impact:** Explain how the changes affect system behavior or user experience.
- **Be specific, not generic:** Use descriptive language that pinpoints the exact changes made.

### Checklist for Writing Effective Commit Messages

**Content and Clarity:**

- [ ] Clearly explain the **motivation** behind the change.
- [ ] Describe the **expected outcome** of the change.
- [ ] Explain the **reasoning** for choosing this approach over others.
- [ ] Acknowledge any **limitations** or potential downsides.
- [ ] Discuss the **impact** on other parts of the codebase or system.
- [ ] Provide **relevant background information** or references.
- [ ] Use **clear and concise language** that is easy to understand.

**Structure and Formatting:**

- [ ] Organize the message into **well-structured paragraphs**.
- [ ] Use **line breaks** to separate different points or sections.
- [ ] Consider using **bullet points** to list key takeaways or changes.

**Additional Considerations:**

- [ ] **Split large commits** into smaller, focused ones if necessary.
- [ ] Use **code comments** for additional context.
- [ ] Incorporate **review feedback** to improve your message.

By following this checklist, we can write commit messages that are informative, well-structured, and easy to understand.