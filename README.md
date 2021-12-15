# Vault GitHub JWT Login

OpenID Connect (OIDC) allows your GitHub Actions workflows to authenticate with a HashiCorp Vault to retrieve secrets.

This guide gives an overview of how to configure HashiCorp Vault to trust GitHub's OIDC as a federated identity, and demonstrates how to use this configuration in hashicorp/vault-action to retrieve secrets from HashiCorp Vault.


1. Configure the JWT Auth using GitHub OIDC settings. Assign the default role, if user logged in via JWT.
    ```bash
    vault write auth/jwt/config \
        oidc_discovery_url="https://token.actions.githubusercontent.com" \
        bound_issuer="https://token.actions.githubusercontent.com" \
        default_role="ceidemo-staging"
    ```

2. Create roles to check the audiences and the claims to verify.  
    ```bash
    vault write auth/jwt/role/ceidemo-staging - <<EOF
    {
        "role_type": "jwt",
        "policies": ["ceidemo-staging"],
        "token_explicit_max_ttl": 60,
        "user_claim": "actor",
        "bound_audiences": "ceidemo",
        "bound_claims": {
            "repository": "ksivamuthu-cei/github-vault-oidc"
        }
    }
    EOF
    ```

    > The sample JWT claims are here. You can add this in bound_claims to verify.

    ```json
    {
        "jti": "7864abd6-d6a6-4142-b2de-e429872c3f42",
        "sub": "repo:ksivamuthu-cei/github-vault-oidc:ref:refs/heads/main",
        "aud": "ceidemo",
        "ref": "refs/heads/main",
        "sha": "7170bd0ba604ed9414da11e36eef66690b7526ff",
        "repository": "ksivamuthu-cei/github-vault-oidc",
        "repository_owner": "ksivamuthu-cei",
        "run_id": "1576943580",
        "run_number": "6",
        "run_attempt": "2",
        "actor": "ksivamuthu-cei",
        "workflow": "CI",
        "head_ref": "",
        "base_ref": "",
        "event_name": "push",
        "ref_type": "branch",
        "job_workflow_ref": "ksivamuthu-cei/github-vault-oidc/.github/workflows/build.yaml@refs/heads/main",
        "iss": "https://token.actions.githubusercontent.com",
        "nbf": 1639471952,
        "exp": 1639472852,
        "iat": 1639472552
    }
    ```


1. Create policies to give permissions to read the secret

    ```bash
    vault policy write ceidemo-staging - <<EOF
    path "secret/data/ceidemo/staging/*" {
    capabilities = [ "read" ]
    }
    EOF
    ```

2. Retrieve the secret using hashicorp vault action

```yaml
  - name: Retrieve secret from Vault
    uses: hashicorp/vault-action@v2.4.0
    id: secrets
    with:
        url: ${{ secrets.VAULT_URL }}
        role: <ROLE>
        method: jwt
        namespace: <Namespace>
        jwtGithubAudience: <Audience>
        secrets: secret/data/ceidemo/staging/db password | DB_PASSWORD    
```

5. Use the retrieved secret in the sensitive operation

```yaml
  - name: Sensitive Operation
    run: "db-cli --password '${{ steps.secrets.outputs.DB_PASSWORD }}'"
```