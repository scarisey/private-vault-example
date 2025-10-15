# Private Vault Example

This repository is a template for managing encrypted secrets with SOPS and age encryption, designed to work with [nixos-dotfiles](https://github.com/scarisey/nixos-dotfiles).

## Fork and Setup

### 1. Fork this repository

Create your own private repository from this template to store your encrypted secrets.

### 2. Generate your age keys

Generate your personal age key:

```bash
mkdir -p ~/.config/sops/age
age-keygen -o ~/.config/sops/age/keys.txt
```

The public key will be displayed - copy it for the next step.

### 3. Update .sops.yaml

Replace the age keys in `.sops.yaml` with your own:

```yaml
keys:
  - &ci age1vlcfusx5f235x84rx5gftfshc7twhz3ydj3mmwev4a44m0j2gdvsd9vj3n  # Replace with CI key
  - &yourusername age1your...publickey  # Replace with your personal key
creation_rules:
  - path_regex: secrets.yaml
    key_groups:
    - age:
      - *ci
      - *yourusername
```

### 4. Re-encrypt secrets.yaml

```bash
# Edit the secrets file with SOPS (it will re-encrypt with your keys)
sops secrets.yaml
```

### 5. Update your nixos-dotfiles

In your fork of `nixos-dotfiles`, update the `flake.nix` to point to your private vault:

```nix
inputs = {
  # ... other inputs ...
  
  private-vault = {
    url = "github:yourusername/your-vault-repo";
  };
}
```


## Adding Age Keys via CI

This repository includes a GitHub Action workflow that allows you to add new age public keys and automatically re-encrypt all secrets without having access to the original age private key locally.

### Prerequisites

Store your CI age private key in GitHub Actions secrets as `SOPS_AGE_KEY`. This key must be one of the existing age recipients in `.sops.yaml`.

### Usage

To add a new age public key from the GitHub Actions UI:

1. Navigate to the **Actions** tab in your repository
2. Select the **Update SOPS Secrets** workflow
3. Click **Run workflow**
4. Enter the new age public key when prompted
5. Click **Run workflow** to start the process

The workflow will automatically add the new public key to `.sops.yaml`, re-encrypt `secrets.yaml` with the updated recipient list, and commit the changes back to the repository.

## GitHub Actions Integration

### HTTPS with Personal Access Token

This approach doesn't require SSH key management.

**Step 1: Create a Personal Access Token (PAT)**

* Create a **Personal Access Token** with `repo` scope
* Store it in GitHub Actions secrets as `GH_TOKEN`

**Step 2: Use github: URLs in flake.nix**

```nix
inputs.private-vault = {
  url = "github:yourusername/your-vault-repo";
};
```

**Step 3: Configure authentication in workflow**

```yaml
env:
  NIX_CONFIG: |
    experimental-features = nix-command flakes
    access-tokens = github.com=${{ secrets.GH_TOKEN }}
```

This method is simpler, doesn't require SSH key management, and works seamlessly with Nix flakes.
