Using Git we can push the source code of the aplication to Azure repo

1. **Repositories**:

- Service: Azure Repos (Git)
- Purpose: Store code and manage version control
- Configuration: Set up branching strategy (e.g., GitFlow) and pull request policies

2. **Managing Source Code in Azure Repo**:

- Adding Source Code

```bash

git add .
git commit -m "Initial commit"
git push origin main

```

3. **Managing Source Code in Azure Repo**

- For each new feature or bug fix, we can create a new branch:

```bash

git checkout -b feature/your-feature-name

```

4. **Modify on our Branch**

- We make changes and commit them

```bash
git add .
git commit -m "Description of changes"
git push origin feature/your-feature-name

```
