
name: Auto Restart and Keep Codespace Active

on:
  schedule:
    - cron: '* * * * *'  # Runs every minute
  workflow_dispatch:

jobs:
  restart_codespace:
    runs-on: ubuntu-latest

    steps:
      - name: Install GitHub CLI if not present
        run: |
          if ! command -v gh &> /dev/null; then
            sudo apt-get install gh jq
          fi

      - name: Authenticate GitHub CLI with Personal Access Token
        run: |
          echo "your github classic token gen from development settings" | gh auth login --with-token

      - name: Check Codespace Status and Keep Alive
        run: |
          CODESPACE_NAME="your codespace app name"
          STATUS=$(gh codespace list --json name,state | jq -r --arg name "$CODESPACE_NAME" '.[] | select(.name == $name) | .state')

          if [ -z "$STATUS" ]; then
            echo "Error fetching codespace status or codespace does not exist."
            exit 1
          fi

          echo "Codespace status: $STATUS"

          if [ "$STATUS" == "Shutdown" ]; then
            echo "Codespace is offline. Attempting to start it by SSH..."
            gh codespace ssh --codespace $CODESPACE_NAME -- echo 'Codespace started!'
          else
            echo "Codespace is active. Sending keep-alive signal..."
            gh codespace ssh --codespace $CODESPACE_NAME -- echo 'Keep-alive signal'
          fi
