
Old update project infra script
```bash
#!/bin/bash
set -e
if [[ -z ${ENV} ]]; then
  echo "ENV variable not set"
  echo "Set to 'dev', 'qa', 'prod' or 'sandbox'"
  exit 1
fi
# Verify the value of the ENV variable
case "${ENV}" in
  qa|dev|prod|sandbox)
    echo "The ENV is set to ${ENV}"
    ;;
  *)
    echo "The ENV variable is set to an invalid value: ${ENV}. Please set it to one of the following values: qa, dev, prod, sandbox."
    exit 1
    ;;
esac

if [ "$1" == "import" ]; then
  echo -e "\033[1;33mRunning terraform import -var-file=vars.tfvars ${@:2}\033[0m"
  terraform import -var-file=vars.tfvars "${@:2}"
  exit 0
fi
# Check if 1password CLI is installed:
if ! command -v op &> /dev/null
then
    echo "op not found"
    TF_CMD="terraform"
else
    TF_CMD="op run -- terraform"
    echo -e "\033[0;36m------ 1password account details ------ \033[0m"
    op account get
    echo -e "\033[0;36m---------------------------------------\033[0m"
    # Export project-specific variables fo that we are not bothered by Terraform all the time:
    if [[ ${ENV} == "sandbox" ]]; then
      export TF_VAR_snowflake_username='op://Shared/Contiamo Snowflake/username'
      export TF_VAR_snowflake_password='op://Shared/Contiamo Snowflake/password'
      export TF_VAR_snowflake_account_identifier='op://Shared/Contiamo Snowflake/account-identifier'
    fi
fi

INIT_CMD="$TF_CMD init -backend-config=backend-${ENV}.conf"
# Run init if --skip-innit not passed:
if [ "$1" != "--skip-init" ]; then
  echo -e "\033[1;33mRunning $INIT_CMD\033[0m"
  $INIT_CMD
fi


PLAN_CMD="${TF_CMD} plan -var-file=vars-${ENV}.tfvars -out=project.tfplan"
echo "running \"${PLAN_CMD}\""
${PLAN_CMD}
read -p "Do you want to apply the changes? (y/N) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
  ${TF_CMD} apply "project.tfplan"
else
  echo "Changes not applied"
fi
```