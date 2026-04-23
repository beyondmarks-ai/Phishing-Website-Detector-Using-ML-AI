# Contributing to SafeSurf

Thank you for your interest in contributing! 
We welcome bug reports, feature requests, code, documentation, and testing help.


## Project Structure

- `server/` – Backend (Golang)
- `web/` – Frontend (HTML/CSS/JS or framework)
- `docker/` – Dockerfiles & Compose configs
- `docs/` – Project documentation


##  How to Contribute

1. Create a branch from the main line of development:
   ```bash
   git checkout -b your-feature-name
   ```

2. Make your changes
3. Run tests (from the repo root, with `make` available):

   ```bash
   make local-test
   ```

4. Submit your changes through your team’s usual review process (for example, a pull request or internal code review).


##  Development Setup

### Backend (Go)

```bash
cd server
go mod tidy
go run cmd/safesurf/safesurf.go
```

### Frontend

Serve `web/` using your preferred method (e.g., `live-server`, Python HTTP server, or static hosting).

### Docker (Full Stack)

```bash
docker-compose -f docker/docker-compose.dev.yml up --build
```


##  Code Style & Tools

* Format Go code:

  ```bash
  gofmt -w .
  golint ./...
  ```
* Write clear commit messages:

  ```
  feat(auth): add token expiration check
  fix(api): correct 404 response logic
  ```


## Reporting Bugs

1. Search existing bug reports or task tickets first
2. If not found, open a new report with your team’s process
3. Include:

   * Steps to reproduce
   * Logs or screenshots
   * Your environment (OS, browser, etc.)


<!-- ## Feature Requests

Use the [Feature Request template](../../issues/new?template=feature_request.yml)
Describe the use case and expected behavior. -->


## Code of Conduct

Please follow our [Code of Conduct](CODE_OF_CONDUCT.md).


##  Thanks

Your contributions make this project better — whether it’s code, feedback, or documentation.

