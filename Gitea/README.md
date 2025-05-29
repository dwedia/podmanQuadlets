# Gitea Podman Quadlet

#### Files:
- gitea.container
    - Gitea Container is defined here
- gitea.pod
    - Gitea Podman Pod is defined here, exposed ports is defined here, not in the .container files
- gitea-postgres.container
    - Postgres Container is defined here
- gitea-postgres.volume
    - volume for postgres is defined here. It will be auto created in the graphroot folder
- gitea.volume
    - volume for postgres is defined here. It will be auto created in the graphroot folder
- gitea-secrets.env
    - environment variables are defined in here. This file should be kept out of version control (git). Its ignored via the .gitignore file

#### Environment variables:

```bash
## Gitea Env Vars
GITEA__database__DB_TYPE=postgres
GITEA__database__HOST=localhost:5432
GITEA__database__NAME=database-name-goes-here
GITEA__database__USER=database-user-name-goes-here
GITEA__database__PASSWD=long-and-complicated-password-here-i-recommend-atleast-24-characters

## Postgres Env Vars
POSTGRES_USER=database-user-name-goes-here
POSTGRES_PASSWORD=long-and-complicated-password-here-i-recommend-atleast-24-characters
POSTGRES_DB=database-name-goes-here
```
