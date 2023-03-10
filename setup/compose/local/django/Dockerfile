ARG PYTHON_VERSION=3.8-alpine

# define an alias for the specfic python version used in this file.
FROM arm32v6/python:${PYTHON_VERSION} as python


# Python build stage
FROM python as python-build-stage

ARG BUILD_ENVIRONMENT=local

# Install apk packages
RUN apk -U add \
  # dependencies for building Python packages
  build-base \
  # psycopg2 dependencies
  libpq-dev \
  # libffi
  libffi libffi-dev


# Installing Rust is mandatory for cryptography
RUN ["apk", "--no-cache", "add", "curl"]
RUN curl https://sh.rustup.rs -sSf | sh

# Requirements are installed here to ensure they will be cached.
COPY ./requirements .

# Create Python Dependency and Sub-Dependency Wheels.
RUN pip wheel --wheel-dir /usr/src/app/wheels  \
  -r ${BUILD_ENVIRONMENT}.txt


# Python 'run' stage
FROM python as python-run-stage

ARG BUILD_ENVIRONMENT=local
ARG APP_HOME=/app

ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1
ENV BUILD_ENV ${BUILD_ENVIRONMENT}

WORKDIR ${APP_HOME}

# Install required system dependencies
RUN apk -U add \
  # psycopg2 dependencies
  libpq-dev \
  # Translations dependencies
  gettext \
  # The older versions of packages can be removed with the clean command.
  && apk cache clean

# All absolute dir copies ignore workdir instruction. All relative dir copies are wrt to the workdir instruction
# copy python dependency wheels from python-build-stage
COPY --from=python-build-stage /usr/src/app/wheels  /wheels/

# use wheels to install python dependencies
RUN pip install --no-cache-dir --no-index --find-links=/wheels/ /wheels/* \
	&& rm -rf /wheels/

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint

COPY ./compose/local/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start



# copy application code to WORKDIR
COPY . ${APP_HOME}

ENTRYPOINT ["/entrypoint"]
