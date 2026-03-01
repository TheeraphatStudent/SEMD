# SEMD

an malicious url checker tools integrated with ML

this project was usage python `3.12.3` to development

## Project structure overview

```
SEMD
├── semd-backend/          # Backend service (Python/FastAPI)
│   ├── config/             # Configuration files
│   ├── database/           # Database scripts and migrations
│   ├── models/             # Data models and schemas
│   ├── routers/            # API endpoints and routes
│   ├── services/           # Business logic and utilities
│   ├── workers/            # Background workers
│   ├── main.py             # Application entry point
│   ├── compose.yaml        # Docker compose configuration
│   └── requirements/       # Dependency files
│
├── semd-ml/                # Machine Learning service
│   ├── src/                # Source code and datasets
│   ├── models/             # Trained ML models
│   ├── reports/            # Model evaluation reports
│   ├── requirements.txt    # Python dependencies
│   ├── docker-compose.yml  # ML service orchestration
│   └── Dockerfile          # ML service container
│
├── semd-frontend/          # Frontend application (Next.js/React)
│   ├── src/                # Application source code
│   ├── public/             # Static assets and images
│   ├── package.json        # Node.js dependencies
│   ├── tailwind.config.ts  # Styling configuration
│   └── next.config.js      # Next.js configuration
│
├── semd-extension/         # Browser extension (Chrome/Firefox)
│   ├── chrome_extension/   # Chrome-specific extension files
│   ├── firefox_addons/     # Firefox-specific addon files
│   ├── scripts/            # Build and deployment scripts
│   ├── package.json        # Extension dependencies
│   └── extension.conf.yaml # Extension configuration
│
└── README.md               # Project documentation
```

## Service port

- Backend: 3000
- Frontend: 3001
- MLflow: 5000
- Redis: 6379
- PostgreSQL: 5432

## To usage pyenv

1. List version of python

```bash
pyenv install -l
```

2. Install specification version
```bash
pyenv install 3.12
```

3. Enjoy (:

## Defind network

```bash
podman network create semd-shared-network
```

## For python project

```bash
pyformat --in-place **/*.py
```

## Remark

Kill running process on linux
```bash
ps aux | grep semd | grep python | awk '{print $2}' | xargs kill -9
```