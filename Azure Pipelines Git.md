# Azure Pipeline Git Commands

You might come across this error:
```
remote: 0000000000aaTF401027: You need the Git 'GenericContribute' permission to perform this action. Details: identity 'Build\d9a1f269-95a3-41d6-9728-c7fa1446a7d6', scope 'repository'.
remote: TF401027: You need the Git 'GenericContribute' permission to perform this action. Details: identity 'Build\d9a1f269-95a3-41d6-9728-c7fa1446a7d6', scope 'repository'.
fatal: unable to access 'https://dev.azure.com/cbre/Advisory%20Services%20Continental%20Europe/_git/GDP-Replatform/': The requested URL returned error: 403
##[error]Bash exited with code '128'.

```

To solve:

- Copy the guid part of the identity name from the error message
- Go to the repository settings
- Go to the "**Security**" tab
- Paste the guid into the permissions search box for user or group.
- This will then display a the user name “**Advisory Services Continental Europe Build Service (cbre)**”. Select that user.
- Set the permissions (read, contribute)


```yaml
# See https://aka.ms/yaml
pool: Gdp-Replatform-pool
pr:
  paths:
    include:
      - table_list.txt

variables:
  - name: ENV
    ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/dev') }}:
      value: "dev"
    ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/main') }}:
      value: "qa"
    ${{ if startsWith( variables['Build.SourceBranch'], 'refs/tags/') }}:
      value: "prod"
jobs:
  - job: CheckConditions
    displayName: 'Check if Table List Has Changes'
    steps:
    - checkout: self
    - bash: |
        sourceBranch="$(Build.SourceBranch)"
        targetBranch="$(System.PullRequest.TargetBranch)"
        if [[ "$sourceBranch" != "refs/heads/dev" && "$targetBranch" != "refs/heads/main" ]]; then
          echo "##vso[task.setvariable variable=runJob;isOutput=true]true"
        else
          echo "##vso[task.setvariable variable=runJob;isOutput=true]false"
        fi
      name: conditionCheck
  - job: generate_tables
    displayName: Generate Tables
    dependsOn: CheckConditions
    condition: and(succeeded(), eq(dependencies.CheckConditions.outputs['conditionCheck.runJob'], 'true'))
    variables:
      changesDetected: 'false'
    steps:
    # - script: |
    #     if ! command -v az &> /dev/null
    #     then
    #         curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    #     else
    #         echo "Azure CLI is already installed."
    #     fi
    #     az --version
    #   displayName: Install Azure CLI if not present
    - checkout: self
      persistCredentials: true
      clean: true
    # - bash: |
    #     echo "Install Task"
    #     sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
    #     echo "Installing Python Deps..."
    #     sudo apt-get update
    #     sudo apt-get install freetds-dev build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev -y
    #     curl -sSL https://install.python-poetry.org | python3 -
    #     curl -sSL https://install.python-poetry.org | POETRY_VERSION=1.4.2 python3 -
    #     export PATH=$PATH:$HOME/.local/bin
    #     poetry --version
    #     echo "Setup env with Task"
    #     task setup
      # displayName: Install Task and Python Deps
    # This step makes sure that the PATH variable includes poetry bin directory (Azure docs: https://github.com/microsoft/azure-pipelines-tasks/blob/master/docs/authoring/commands.md)
    - script: |
        echo "##vso[task.prependpath]$HOME/.local/bin"
      displayName: "Update PATH"
    # - bash: |
    #     poetry run python -m pipeline.validate_connection
    #   env:
    #     MSSQL_USER: $(MSSQL_USER)
    #     MSSQL_PASSWORD: $(MSSQL_PASS)
    #     MSSQL_DATABASE: "TST_ArgoWeb_DE_Crash"
    #     MSSQL_HOST: "SQLTSTDNS_ArgoWeb_DE.emea.cbre.net"
    #     MSSQL_PORT: "1433"
    #   displayName: 'Validate MSSQL Access'

    - bash: |
        echo "pseudo table generation" > mytable.sql
      displayName: 'Generate Tables'
    - bash: |
        git fetch origin main
        if git diff --quiet main -- ${pathToTableList}; then
          echo "No changes detected"
        else
          echo "##vso[task.setvariable variable=changesDetected]true"
          echo "Changes detected"
        fi
      displayName: 'Diff Table List'

    - bash: |
        echo "Configure git"
        git config user.email "$(Build.RequestedForEmail)"
        git config user.name "$(Build.RequestedFor)"
        echo "Adding changes"
        git add .
        git commit -m "chore: Adding generated mytable-$(Build.BuildId)"
        git push
      displayName: 'Commit Generated Tables'
      condition: eq(variables.changesDetected, 'true')

    # - bash: az devops configure --defaults organization="$(System.TeamFoundationCollectionUri)" project="$(System.TeamProject)" --use-git-aliases true
    #   displayName: 'Set default Azure DevOps organization and project'
    # - bash: |
    #     az repos pr create \
    #       --title "chore: Adding generated mytable-$(Build.BuildId)" \
    #       --description "Automatically generated" \
    #       --source-branch "mytable-$(Build.BuildId)" \
    #       --target-branch "main" \
    #       --repository "$(Build.Repository.Name)" \
    #       --detect false \
    #       --org "$(System.TeamFoundationCollectionUri)" \
    #       --project "$(System.TeamProject)"
    #   displayName: 'Create PR'
    #   condition: eq(variables.changesDetected, 'true')
    #   env:
    #     AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)


# - job: Terraform
#   displayName: Terraform Apply Infra
#   variables:
#   - ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/dev') }}:
#     - group: "Secrets-dev"
#   - ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/main') }}:
#     - group: "Secrets-qa"
#   - ${{ if startsWith( variables['Build.SourceBranch'], 'refs/tags/') }}:
#     - group: "Secrets-prod"

#   steps:
#   - checkout: self
#     displayName: 'Checkout $(Build.SourceBranch)'

#   - bash: |
#       # Check if AWS CLI is already installed
#       if ! command -v aws &> /dev/null; then
#         # Clean up any existing awscliv2.zip file
#         rm -f awscliv2.zip

#         # Download and unzip the AWS CLI installer
#         curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
#         unzip -u awscliv2.zip

#         # Install the AWS CLI
#         sudo ./aws/install --update

#         # Clean up the downloaded files
#         rm -f awscliv2.zip
#         rm -rf ./aws
#       else
#         echo "AWS CLI is already installed."
#       fi
#     displayName: Install AWS CLI (if not already installed)
#   - bash: |
#       echo "Install Task"
#       sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
#       echo "Installing Python Deps..."
#       sudo apt-get update
#       sudo apt-get install freetds-dev build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev -y
#       curl -sSL https://install.python-poetry.org | python3 -
#       curl -sSL https://install.python-poetry.org | POETRY_VERSION=1.4.2 python3 -
#       export PATH=$PATH:$HOME/.local/bin
#       poetry --version
#       echo "Setup env with Task"
#       task setup
#     displayName: Install Task and Python Deps
#   # This step makes sure that the PATH variable includes poetry bin directory (Azure docs: https://github.com/microsoft/azure-pipelines-tasks/blob/master/docs/authoring/commands.md)
#   - script: |
#       echo "##vso[task.prependpath]$HOME/.local/bin"
#     displayName: "Update PATH"

#   - script: |
#       echo "ENV: $(ENV)"
#       echo "MSSQL_PASS (between _): _$(MSSQL_PASS)_"
#       echo "MSSQL_USER: (between _): _$(MSSQL_USER)_"
#     displayName: 'Debug ENV Secrets'

#   - bash: |
#       aws sts get-caller-identity
#       aws iam list-attached-role-policies --role-name $(aws sts get-caller-identity --query 'Arn' --output text | awk -F/ '{print $2}')
#       output=$(aws iam list-attached-role-policies --role-name $(aws sts get-caller-identity --query 'Arn' --output text | awk -F/ '{print $2}') --query 'AttachedPolicies[].PolicyName' --output text) && [ "$output" == "gdp-replatforming-terraform" ] || { echo "Failure: PolicyName is not gdp-replatforming-terraform"; exit 1; }
#     displayName: 'Validate AWS Access'

#   - bash: |
#       poetry run python -m pipeline.validate_connection
#     env:
#       MSSQL_USER: $(MSSQL_USER)
#       MSSQL_PASSWORD: $(MSSQL_PASS)
#       MSSQL_DATABASE: "TST_ArgoWeb_DE_Crash"
#       MSSQL_HOST: "SQLTSTDNS_ArgoWeb_DE.emea.cbre.net"
#       MSSQL_PORT: "1433"
#     displayName: 'Validate MSSQL Access'

#   - bash: |
#       wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
#       echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
#       sudo apt update && sudo apt install terraform -y
#     displayName: Install Terraform
  # - bash: |
  #     ./scripts/ensure_prerequisites.sh
  #   displayName: "Ensure Prerequisites Env: $(ENV)"

  # - bash: |
  #     echo "terraform init -backend-config=backend-$(ENV).conf"
  #     # terraform init -backend-config=backend-$(ENV).conf
  #   workingDirectory: terraform/infra
  #   displayName: "Terraform Infra Init. Env: $(ENV)"

  # - bash: |
  #     echo "terraform plan -var-file=vars-$(ENV).tfvars -out=project.tfplan"
  #     # terraform plan -var-file=vars-$(ENV).tfvars -out=project.tfplan
  #   workingDirectory: terraform/infra
  #   displayName: "Terraform Infra Plan. Env: $(ENV)"
  #   env:
  #     TF_VAR_snowflake_username: 'FAKE_USERNAME'
  #     TF_VAR_snowflake_password: 'FAKE_PASSWORD'
  #     TF_VAR_snowflake_account_identifier: 'FAKE_ACCOUNT'
  #     TF_VAR_dms_mssql_password: $(MSSQL_PASS)
  #     TF_VAR_dms_mssql_username: 'FAKE_MSSQL_USERNAME'

```


```yaml
# See https://aka.ms/yaml
pool: Gdp-Replatform-pool
pr:
  paths:
    include:
      - table_list_prod.txt
      - table_list_dev.txt
# variables:
#   - name: ENV
#     ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/dev') }}:
#       value: "dev"
#     ${{ if eq( variables['Build.SourceBranch'], 'refs/heads/main') }}:
#       value: "qa"
#     ${{ if startsWith( variables['Build.SourceBranch'], 'refs/tags/') }}:
#       value: "prod"
jobs:
  - job: CheckConditions
    displayName: 'Check if Table List Has Changes'
    steps:
    - checkout: self
      persistCredentials: true
      clean: true
    - bash: |
        sourceBranch="$(Build.SourceBranch)"
        echo "Source Branch: $sourceBranch"
        if [[ "$sourceBranch" != "refs/heads/dev" ]]; then
          echo "Source branch is NOT DEV. Running the job"
          echo "##vso[task.setvariable variable=runJob;isOutput=true]true"
        else
          echo "Source branch is DEV. Skipping the job"
          echo "##vso[task.setvariable variable=runJob;isOutput=true]false"
        fi
      name: conditionCheck
    - bash: |
        set -x
        echo The target branch is $TARGET_BRANCH
        echo $(Build.Reason)
        echo "-------------------ALL ENV. VARS-----------------------"
        printenv
        echo "-------------------------------------------------------"
      displayName: 'DEBUG'
      env:
        TARGET_BRANCH: $(System.PullRequest.TargetBranch)
  - job: generate_tables
    displayName: Generate Tables
    dependsOn: CheckConditions
    condition: and(succeeded(), eq(dependencies.CheckConditions.outputs['conditionCheck.runJob'], 'true'))
    variables:
    - group: Secrets-dev
    - name: changesDetected
      value: 'false'
    - name: pathToTableFiles
      value: 'table_files'
    steps:
    - checkout: self
      persistCredentials: true
      clean: true
      fetchDepth: 0
    - bash: |
        set -x
        echo "Configure git"
        git config user.email "$(Build.RequestedForEmail)"
        git config user.name "$(Build.RequestedFor)"
        thisBranch=$(echo "$(Build.SourceBranch)" | awk -F/ '{print $NF}')
        git checkout "${thisBranch}"
        git pull --rebase
      displayName: 'Prepare Git'
    - bash: |
        echo "Install Task"
        sudo sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin
        echo "Installing Python Deps..."
        sudo apt-get update
        sudo apt-get install freetds-dev build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev -y
        curl -sSL https://install.python-poetry.org | python3 -
        curl -sSL https://install.python-poetry.org | POETRY_VERSION=1.4.2 python3 -
        export PATH=$PATH:$HOME/.local/bin
        poetry --version
        echo "Setup env with Task"
        task setup
      displayName: Install Task and Python Deps
    # This step makes sure that the PATH variable includes poetry bin directory (Azure docs: https://github.com/microsoft/azure-pipelines-tasks/blob/master/docs/authoring/commands.md)
    - script: |
        echo "##vso[task.prependpath]$HOME/.local/bin"
      displayName: "Update PATH"
    - bash: |
        poetry run python -m pipeline.validate_connection
      env:
        MSSQL_USER: $(MSSQL_USER)
        MSSQL_PASSWORD: $(MSSQL_PASS)
        MSSQL_DATABASE: "TST_ArgoWeb_DE_Crash"
        MSSQL_HOST: "SQLTSTDNS_ArgoWeb_DE.emea.cbre.net"
        MSSQL_PORT: "1433"
      displayName: 'Validate MSSQL Access'

    - bash: |
        echo "pseudo table generation" >> $(pathToTableFiles)/mytable.sql
      displayName: 'Generate Tables'
    - bash: |
        echo git fetch origin main
        git fetch origin main
        if git diff --quiet origin/main -- $(pathToTableFiles); then
          echo "No changes detected"
        else
          echo "##vso[task.setvariable variable=changesDetected]true"
          echo "Changes detected"
        fi
      displayName: 'Diff Table List'
    - bash: |
        git add .
        git commit -m "chore: Adding generated mytable-$(Build.BuildId) [skip ci]"
        git push origin $(Build.SourceBranch)
      displayName: 'Commit Generated Tables'
      condition: eq(variables.changesDetected, 'true')


```