# Перед созданием ветки
git pull
git checkout -b <branch-type>/<ticket-number>-<desc>

# Перед открытием пул реквеста
git fetch origin
git rebase origin/main
git push --force-with-lease origin <your-branch>

# После подтверждения пул реквеста
git checkout main
git pull

# branch-type
hotfix
bugfix
refactor
task
features