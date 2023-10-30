# Contributing

Thank you for considering contributing to our projects! We welcome
your contributions to help make our projects even better.

## Conventional Commits

Conventional Commits are a standardized format for commit messages that 
help improve the clarity and consistency of version control history. 

They follow a strict structure:

- `<type>`: describes the purpose of the commit. It can be one of the following:
  - feat: represents a new feature or enhancement. 
  - fix: indicates a bug fix.
  - docs: refers to documentation changes.
  - style: relates to code style and formatting improvements.
  - refactor: relates code refactoring with no functional changes.
  - perf: indicates performance improvements.
  - test: covers the addition or modification of tests.
- `<scope>`: is optional, describes the scope of the commit, often specifying the 
component or area of the code that's affected.
- `<description>`: a concise and clear description of the change, written in the 
imperative mood (e.g., add, fix, update).

### Why Conventional Commits?

We follow these standards mainly because of consistency, as it 
establishes a consistent and standardized way to communicate the 
nature of updates and new features. This consistency makes it easier 
for us and other users to understand the purpose of a commit by 
just looking to the commit message.

### Making good commits

To write conventional commits, follow these best practices:

- Use a clear and descriptive <description> in the imperative mood.
  - Example: `feat(users): add registration feature`

- Consider specifying a `<scope>` to provide context for larger 
codebases or projects with multiple components.
  - Example: `fix(api): resolve authentication issue`

- Keep commit messages concise and focused on a single change. If a 
change spans multiple aspects, consider breaking it into multiple commits, 
each addressing a specific concern.

Please make sure to format your commit messages according to the
Conventional Commits standard to ensure consistency over our
projects.

## How to contribute

To contribute to our projects, please follow these steps:

1. Fork the repository.
2. Create a new branch for your feature:

    ```Bash
    git checkout -b scope/your-feature
    ```

3. Make changes and write your code.
4. Test the changes and ensure they are working correctly.
5. Commit your changes using the [Conventional Commits message standard](#conventional-commits).
6. Push your branch to your fork.
7. Create a pull request from your branch to the main repository.
8. Describe your changes and provide more context about the feature.

Then, just wait for any Flyte member to review your code and address
any feedback. Once approved, your changes will be merged into
the main repository.

## Observations

- Provide detailed information about the problem your changes are solving or about 
the new feature you're adding.
- If your PR addresses any existing issues, please reference that issue in the PR
description using the syntax `Closes #123` or `Fixes #456` to automatically close
the issue when the PR is merged.
- Be prepared for feedback and potential requests for changes. Our goal is to
maintain the quality and integrity of our projects.