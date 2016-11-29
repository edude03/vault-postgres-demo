# Postgres Vault Integration Demo

Ensure vault(-cli) and docker-compose are installed
```
$ brew install vault docker-compose
```

Start the servers
```
$ docker-compose up
```

Export the path to the vault server
```
$ export VAULT_ADDR='http://0.0.0.0:8200'
```

Authenticate with vault (with root token)
```
$ vault auth 06f6e86b-0440-748f-3007-863c5702de32
```

Add the postgresql backend
```
$ vault mount postgresql
```

Configure the backend

```
$ vault write postgresql/config/connection connection_url="postgresql://postgres:postgres@postgres/postgres?sslmode=disable"
```

Configure the lease time for the postgres backend
```
$ vault write postgresql/config/lease lease=1h lease_max=24h
```

Create a readonly role
```
$ vault write postgresql/roles/readonly \
    sql="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";"
```

Check if you can generate credentials (as root still)
```
$ vault read postgresql/creds/readonly
```

Create a policy for support
For example in policies.json:

```json
{
  "path": {
    "postgresql/creds/readonly": {
      "policy": "read"
    }
  }
}
```

Create a load the policy for support
```
$ vault policy-write support policy.json
```

Add the username & password auth method
```
$ vault mount userpass
$ vault auth-enable userpass
```

Add a username/password combo for support
```
$ vault write auth/userpass/users/support password=testing policies=support
```

Login as support
```
$ vault auth -method=userpass username=support password=testing
```

Get your credentials
```
$ vault read postgresql/creds/readonly
Key             Value
---             -----
lease_id        postgresql/creds/readonly/0a223986-5ea4-197c-90e4-9c13e1705559
lease_duration  1h0m0s
lease_renewable true
password        95d4160a-fd24-51e4-29bd-b50c1f881d10
username        userpass-support-946d3ed1-d12e-111a-7f5f-ddeb31d612cb
```
