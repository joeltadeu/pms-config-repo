# PMS Configuration Repository

This repository is the **single source of truth** for all externalised configuration
in the Patient Management System. It is read by `infra-config-server` at startup and
served to every microservice via the Spring Cloud Config protocol.

---

## Folder structure

```
pms-config-repo/
├── appointment-service/
│   ├── application.yml            ← default (all profiles)
│   ├── application-local.yml      ← local dev overrides
│   └── application-docker.yml     ← Docker / production overrides
├── auth-service/
│   ├── application.yml
│   └── application-docker.yml
├── doctor-service/
│   ├── application.yml
│   └── application-docker.yml
└── patient-service/
    ├── application.yml
    └── application-docker.yml
```

### Naming conventions

| File name | When is it loaded? |
|---|---|
| `{service}/application.yml` | Always — base configuration for every profile |
| `{service}/application-{profile}.yml` | Only when `SPRING_PROFILES_ACTIVE` contains `{profile}` |

The `{service}` folder name **must match** the value of `spring.application.name` in
that service (e.g. `appointment-service`, `auth-service`).

---

## How the Config Server resolves files

Given a service named `appointment-service` running with profile `docker`, the server
checks (in priority order, highest first):

1. `appointment-service/application-docker.yml`
2. `appointment-service/application.yml`

Values in the higher-priority file override those in the lower-priority one.

---

## Making a configuration change

1. Edit the relevant YAML file in this repository.
2. Commit and push the change to the `main` branch (or the branch set in `CONFIG_GIT_BRANCH`).
3. Trigger a live refresh **without restarting** the service:

   ```bash
   curl -X POST http://localhost:9094/config/actuator/refresh
   # or target a specific service directly:
   curl -X POST http://localhost:8083/actuator/refresh
   ```

   > **Note:** Only beans annotated with `@RefreshScope` (or `@ConfigurationProperties`)
   > pick up the new values. The `AppointmentProperties` bean uses `@ConfigurationProperties`
   > and is therefore refreshable.

---

## Secrets and sensitive values

**Do not store secrets in this repository.** JWT secrets, database passwords, and API
keys must be supplied as environment variables (via `.env` or your secrets manager) and
referenced with `${ENV_VAR}` placeholders in the YAML files.
