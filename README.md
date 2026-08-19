# test project

dummy project for developing, training, testing and validating human + agent (ORCA + remora) project cowork github workflow

## test process
tests should be carried in a loop; the following describes one iteration (start to finish) of the loop:

### step-1
human-side: human refreshes local git repository of project 

#### step-1.1
runs:
```bash
git fetch origin
```
to fetch current remote state

#### step-1.2
runs:
```bash
git reset origin/master
```
to reset local head to same as remote

### step-2
human-side: human makes changes to project; commits changes; pushes changes to github

### step-3
agent-side: generates token; uses token to refresh local git repository

#### step-3.1
if not already loaded, loads the `remora_github.env` file in the shell for the token generation script to be able to access necessary environment variables
```bash
set -a
source ~/.openclaw/secrets/remora_github.env
set +a
```

#### step-3.2
runs token generation script; stores outputted token in a variable
```bash
TOKEN=$(~/.openclaw/scripts/mint_github_app_token.sh)
```

#### step-3.3
uses token to fetch current remote state
```bash
git fetch "https://x-access-token:${TOKEN}@github.com/orcinus-sapien/test-project.git" master:refs/remotes/origin/master
```

#### step-3.4
runs:
```bash
git reset origin/master
```
to reset local head to same as remote

### step-4
agent-side: agent makes changes to project; opens a new branch and commits changes to it; pushes changes to github

#### step-4.1
after making changes to project, runs:
```bash
git checkout -b remora/<new_branch_name>
```
to create a new branch and switch to it

#### step-4.2
after commiting changes, pushes to github using token
```bash
git push "https://x-access-token:${TOKEN}@github.com/orcinus-sapien/test-project.git" remora/<new_branch_name>
```

### step-5
agent-side: agent creates a pull request using token, for human to review and merge
```bash
TOKEN=$(~/.openclaw/scripts/mint_github_app_token.sh)
GH_TOKEN="${TOKEN}" gh pr create \
  --repo orcinus-sapien/test-project \
  --head remora/<new_branch_name> \
  --base master \
  --title "<PR_title>" \
  --body "PR made using remora-gh app for test<number>"
```

### step-6
human-side: human reviews pull request; merges branch  or denys request and provides feedback

loop ends if branch merged

if feedback provided, agent commits and pushes changes to the branch accordingly, then repeats step-5

## test tool

### `.txt` file
every iteration is named consecutively (test1, test2...), and correspond to the `.txt` files (test1.txt, test-2.txt)

files depict the current condition of the iteration, used to track the status of the iteration; see test template[./test_template.txt]
