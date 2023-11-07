#contiamo


# Tech We Prefer

- [Taskfiles](https://taskfile.dev/)
- K8S
	- We always try deploying everything into K8S clusters
	- We run several clusters ourselves
	- We use [K3S](https://k3s.io/). As the last resort 😉
- [Helm](https://helm.sh/)
- Terraform
- Github Actions
- We tolerate Cloudformation

# Dev. Practices at Contiamo
1. **Semantic Commits**: We follow the convention of semantic commits for all commit messages.
	- `fix`
	- `feat`
	- `chore`
	- `docs`
	- `ci`
1. **Default GitHub Action Workflows**:
	- Semantic Commits
	- Release Please

2. **Taskfile**
	- Created at the start of any project
3. **Branch Protection**
	- `main` branch
4. **Dependabot**
5. **Slack Channels**:
	- Every new project or customer gets a dedicated Slack channel.
# Example

- Stadtwerke Bonn Chatbot https://github.com/contiamo/swb-chatbot
	- Taskfile
	- Dependabot
	- Workflows
		- Deploy: parallel; releases
		- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
		- Release Please


## Datascience Project Template
- https://github.com/contiamo/datascience-project-repo-template
# Project Ops
https://github.com/contiamo/project-ops
- Project Ops repo contains the Terraform code for most projects.
- Some projects require the TF code to live in their own repo.

# Customer Repos
Some customers require that we use their own Git providers. In these cases we generally prefer developing in our Github and then sync upstream.

- CBRE:
	- https://dev.azure.com/cbre/Advisory%20Services%20Continental%20Europe/_git/GDP-Replatform-infrastructure

- Telekom
	**Github**: https://github.com/contiamo/telekom-mpathic-dwh
	synced to
	**Gitlab**: https://gitlab.devops.telekom.de/eliza/mpathic_data_analytics_platform/mpathic-dwh

## 1Password
We use 1Password to store all our secrets.
We usually create a vault per customer and grant access to the vault on a need-to-know  basis.