# Heapsteep Config Repo

External configuration for Spring Cloud Config Server. This folder is a **separate Git repository** from the microservices codebase.

## What belongs here

Only configuration files — no Java code:

| File | Served to |
|---|---|
| `application.properties` | All clients (shared) |
| `microservice-1.properties` | `microservice-1` |
| `microservice-2.properties` | `microservice-2` |

Spring Cloud Config resolves: `{application-name}.properties` + shared `application.properties`.

## Option A — Local Git (quick start)

Already set up if you ran `git init` in this folder. Config Server uses:

```properties
spring.cloud.config.server.git.uri=file:///C:/Heapsteep/Microservices/codebase-2/config-repo
```

After editing files here:

```bash
cd config-repo
git add .
git commit -m "Update config"
```

Restart **config-server**, then `POST /actuator/refresh` on clients (or restart clients).

## Option B — Public GitHub (production-style)

### When to create the repo

Create the GitHub repo **before** switching config-server from `file://` to `https://`.

### What to create on GitHub

1. Go to [https://github.com/new](https://github.com/new)
2. Repository name: `heapsteep-config-repo` (or any name)
3. Visibility: **Public** (simplest — no token needed for read-only clone)
4. Do **not** add README, .gitignore, or license (keep the repo empty)

### Push this folder to GitHub

Replace `YOUR_GITHUB_USERNAME` with your GitHub username:

```bash
cd c:\Heapsteep\Microservices\codebase-2\config-repo
git init
git add application.properties microservice-1.properties microservice-2.properties
git commit -m "Initial microservices configuration"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/heapsteep-config-repo.git
git push -u origin main
```

### Update config-server

In `config-server/src/main/resources/application.properties`, comment the `file://` URI and set:

```properties
spring.cloud.config.server.git.uri=https://github.com/YOUR_GITHUB_USERNAME/heapsteep-config-repo.git
spring.cloud.config.server.git.default-label=main
spring.cloud.config.server.git.clone-on-start=true
```

Restart config-server.

## Verify

```text
http://localhost:8888/microservice-1/default
http://localhost:8000/api/config
```

`greeting.message-suffix` should mention **Git**.

## Production tips

- Use **private** repo + deploy key or token on config-server for secrets
- Use branches or folders per environment (`main` = prod, `develop` = dev)
- Never commit raw passwords; use Vault or encrypted properties for secrets
