# Base arguments to leverage across build stages
ARG APP_DIR=/backend

#############
# Build (Prepare Environment) image
#############
FROM python:3.10-alpine AS build
ARG APP_DIR

# Install curl so we can get poetry
# install build-base and libffi-dev so we can install poetry and dependencies (compiles some code)
RUN apk --no-cache add curl build-base libffi-dev

# install poetry and add to path
ENV POETRY_VERSION=1.5.1
RUN curl -sSL https://install.python-poetry.org | python -
ENV PATH /root/.local/bin:$PATH

# change work directory for where our app will live
WORKDIR ${APP_DIR}
COPY pyproject.toml poetry.lock ./

# Copy over the venv including any symbolic links, do not install development/testing libraries when install poetry dependencies
RUN python -m venv --copies ${APP_DIR}/venv
RUN . ${APP_DIR}/venv/bin/activate && poetry install --no-dev


#############
# Deployment image
#############
FROM python:3.10-alpine as run
ARG APP_DIR

COPY --from=build ${APP_DIR}/venv ${APP_DIR}/venv/
ENV PATH ${APP_DIR}/venv/bin:$PATH

WORKDIR ${APP_DIR}/
COPY  . ./

HEALTHCHECK --start-period=30s CMD python -c "import requests; requests.get('http://localhost:8080/health', timeout=2)"

WORKDIR ${APP_DIR}/src
CMD ["gunicorn", "wsgi:app", "-w 2", "-b 0.0.0.0:8080", "-t 30"]