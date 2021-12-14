```
vault write auth/jwt/role/ceidemo-staging - <<EOF
{
  "role_type": "jwt",
  "policies": ["ceidemo-staging"],
  "token_explicit_max_ttl": 60,
  "user_claim": "actor",
  "bound_claims": {
    "repository": "ksivamuthu-cei/github-vault-oidc"
  }
}
EOF
```

```
vault policy write ceidemo-staging - <<EOF
path "secret/data/ceidemo/staging/*" {
  capabilities = [ "read" ]
}
EOF
```