FROM python:3.12.11-alpine3.22

ARG G2P_REF=v3.1.0

RUN apk add --no-cache --virtual .build-deps gcc libc-dev linux-headers make \
 && apk add --no-cache bash git gettext libpq-dev libffi-dev openssl-dev

WORKDIR /app

# Dynamic adapters from workflow (optional; e.g., bank/geo/notification connectors)
COPY adapters.requirements.txt /tmp/adapters.requirements.txt
RUN if [ -s /tmp/adapters.requirements.txt ]; then pip install --no-cache-dir -r /tmp/adapters.requirements.txt --verbose; fi

# Install all Python packages in one layer for better caching and speed
# Use --parallel-builds for faster git cloning and building
RUN pip install --upgrade pip setuptools && \
    pip install jinja2 && \
    pip install --disable-pip-version-check \
        git+https://github.com/openg2p/openg2p-fastapi-common@v1.1.5\#subdirectory=openg2p-fastapi-common \
        git+https://github.com/openg2p/openg2p-fastapi-common@v1.1.5\#subdirectory=openg2p-fastapi-auth \
        git+https://github.com/openg2p/openg2p-fastapi-common@v1.1.5\#subdirectory=openg2p-fastapi-auth-models \
        git+https://github.com/openg2p/openg2p-g2p-bridge@2.0#subdirectory=openg2p-g2p-bridge-models \
        "git+https://github.com/openg2p/openg2p-pbms-bg-tasks@${G2P_REF}#subdirectory=openg2p-pbms-models" \
        "git+https://github.com/openg2p/openg2p-pbms-bg-tasks@${G2P_REF}#subdirectory=openg2p-bg-task-models" \
        "git+https://github.com/openg2p/openg2p-pbms-gen2-bg-tasks@${G2P_REF}#subdirectory=openg2p-bg-task-celery-workers"

RUN apk del --no-network .build-deps

ENV PYTHONUNBUFFERED=1

# Use module path for Celery app (no local main.py)
CMD ["celery", "-A", "openg2p_bg_task_celery_workers.main.celery_app", "worker", "-Q", "bg_task_worker_queue", "--loglevel=debug"]
