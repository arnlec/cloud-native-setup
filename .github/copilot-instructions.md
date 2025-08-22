# Cloud Native Setup - Ansible Role

Cloud Native Setup is an Ansible role that deploys a cloud-native environment based on K3S (lightweight Kubernetes) with ArgoCD for GitOps workflows.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Dependencies
- Install Ansible and required tools:
  - `pip install ansible ansible-lint` -- takes 45-60 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
  - `ansible-galaxy install -r requirements.yaml` -- requires network access to galaxy.ansible.com
  - **WARNING**: Network restrictions may prevent dependency installation. If ansible-galaxy fails with "No address associated with hostname", you cannot install dependencies in restricted environments.

### Linting and Validation
- **CRITICAL**: Always run linting before making changes to understand existing issues:
  - `yamllint .` -- takes <1 second. Checks YAML syntax and formatting.
  - `ansible-lint --skip-list role-name .` -- takes 3-4 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
  - **Note**: The role has known linting issues that are not your responsibility to fix unless they relate to your specific changes.

### Testing
- **Local Testing with Vagrant** (requires network access and vagrant):
  - `cd tests/`
  - `vagrant up` -- takes 5-15 minutes. NEVER CANCEL. Set timeout to 20+ minutes.
  - `ansible-playbook test.yml` -- takes 2-5 minutes. NEVER CANCEL. Set timeout to 10+ minutes.
  - **WARNING**: Cannot test in network-restricted environments due to dependency download requirements.

- **Syntax Checking**:
  - `cd tests/ && ansible-playbook --syntax-check test.yml` -- takes <1 second.
  - **Note**: Will fail without dependencies installed, but validates basic YAML syntax.

### Manual Validation Scenarios
- **After making changes to the role, you MUST validate**:
  - Run `yamllint .` to ensure YAML formatting is correct.
  - Run `ansible-lint --skip-list role-name .` to check Ansible best practices.
  - If you modify templates: Test template rendering with: 
    ```python
    python3 -c "
    from jinja2 import Template
    with open('templates/argocd-ingress.yaml.j2', 'r') as f:
        template = Template(f.read())
    result = template.render(argocd_domain='argocd.k3s.dev', argocd_cert='LS0tLS1CRUdJTi==', argocd_key='LS0tLS1CRUdJTi==')
    print('Template renders successfully!')
    "
    ```
  - If you modify tasks: Verify task syntax with `ansible-playbook --syntax-check tests/test.yml` (will fail on missing dependencies but shows syntax errors).

## Repository Structure

### Key Components
- **`tasks/main.yml`**: Entry point that includes other task files
- **`tasks/prerequisites.yml`**: Installs Python3 and Kubernetes libraries required by ansible.core.k8s module
- **`tasks/kubectl.yml`**: Configures kubectl access for the ansible user
- **`tasks/argocd.yml`**: Installs ArgoCD from upstream manifests and configures Traefik ingress
- **`templates/argocd-ingress.yaml.j2`**: Jinja2 template for ArgoCD Traefik IngressRoute with TLS
- **`meta/main.yml`**: Role metadata and dependencies (depends on xanmanning.k3s)
- **`requirements.yaml`**: External role dependencies
- **`tests/`**: Vagrant-based testing environment with Debian VM

### Important Variables
- **`argocd_domain`**: Domain for ArgoCD access (e.g., "argocd.k3s.dev")
- **`argocd_cert`**: Base64-encoded TLS certificate for ArgoCD
- **`argocd_key`**: Base64-encoded TLS private key for ArgoCD
- **`k3s_version`**: Version of K3S to install (e.g., "v1.31.1+k3s1")

## Common Tasks

### Repo Root Structure
```
.
├── .devcontainer/          # VS Code development container config
├── LICENSE                 # BSD 3-Clause license
├── README.md              # Role documentation
├── defaults/main.yml      # Default variables (currently empty)
├── handlers/main.yml      # Event handlers (currently empty)
├── meta/main.yml          # Role metadata and dependencies
├── requirements.yaml      # External role dependencies
├── tasks/                 # Ansible tasks
├── templates/            # Jinja2 templates
├── tests/                # Testing infrastructure
└── vars/main.yml         # Role variables (currently empty)
```

### Key Files Content Summary
- **README.md**: Shows example playbook usage with required variables
- **meta/main.yml**: Declares dependency on xanmanning.k3s role, targets Debian bullseye
- **requirements.yaml**: Specifies xanmanning.k3s v3.4.4 dependency
- **tests/Vagrantfile**: Creates Debian bookworm64 VM with 2GB RAM for testing
- **tests/inventory**: Points to VM at 192.168.124.23 with ansible user and SSH key

### What This Role Does
1. **Prerequisites**: Installs Python3, pip, and kubernetes/yaml libraries needed for k8s module
2. **Kubectl Setup**: Copies k3s kubeconfig to ansible user's ~/.kube/config and sets up shell completions
3. **ArgoCD Installation**: Downloads ArgoCD manifests from upstream, installs to argocd namespace, disables TLS (uses --insecure flag), creates Traefik IngressRoute for HTTPS access

### Build and Test Timing
- **yamllint**: <1 second - always complete quickly
- **ansible-lint**: 3-4 seconds - usually quick but set 30+ second timeout
- **pip install ansible-lint**: 45-60 seconds - NEVER CANCEL, set 120+ second timeout
- **ansible-galaxy install**: 1-2 minutes per role (network dependent) - NEVER CANCEL
- **vagrant up**: 5-15 minutes - NEVER CANCEL, set 20+ minute timeout
- **ansible-playbook test**: 2-5 minutes - NEVER CANCEL, set 10+ minute timeout

### Known Limitations
- **Network Restrictions**: Cannot install dependencies from galaxy.ansible.com in restricted environments
- **Testing Limitations**: Full integration testing requires Vagrant and network access
- **Linting Issues**: Role has existing linting violations (meta formatting, FQCN usage, etc.)
- **No Unit Tests**: Role relies on integration testing via Vagrant

### Development Container
- Uses `quay.io/ansible/ansible-runner:latest` image
- Automatically installs ansible and ansible-lint via postCreateCommand
- Includes VS Code extensions for Docker, Ansible, and Python
- Mounts workspace to /workspace in container

### When Working on This Role
- **ALWAYS** run `yamllint .` first to understand YAML formatting issues
- **ALWAYS** run `ansible-lint --skip-list role-name .` to check for best practices violations
- Focus changes on your specific requirements - do not fix unrelated linting issues
- Test template rendering manually if modifying templates/argocd-ingress.yaml.j2
- Remember this role depends on xanmanning.k3s being run first to set up the K3S cluster