ARG PYTHON_VERSION=3.8-alpine

# define an alias for the specfic python version used in this file.
FROM arm32v6/python:${PYTHON_VERSION} as python


# Python build stage
FROM python as python-build-stage

ENV PYTHONDONTWRITEBYTECODE 1

# Install apk packages
RUN apk -U add \
  # dependencies for building Python packages
  build-base \
  # psycopg2 dependencies
  libpq-dev \
  # libffi
  libffi libffi-dev \
  # The older versions of packages can be removed with the clean command.
  && apk cache clean

# Requirements are installed here to ensure they will be cached.
COPY ./requirements /requirements

# create python dependency wheels
RUN pip wheel --no-cache-dir --wheel-dir /usr/src/app/wheels  \
  -r /requirements/local.txt -r /requirements/production.txt \
  && rm -rf /requirements


# Python 'run' stage
FROM python as python-run-stage

ARG BUILD_ENVIRONMENT
ENV PYTHONUNBUFFERED 1
ENV PYTHONDONTWRITEBYTECODE 1

RUN apk -U add \
  # To run the Makefile
  make \
  # psycopg2 dependencies
  libpq-dev \
  # Translations dependencies
  gettext \
  # Uncomment below lines to enable Sphinx output to latex and pdf
  # texlive-latex-recommended \
  # texlive-fonts-recommended \
  # texlive-latex-extra \
  # latexmk \
  # The older versions of packages can be removed with the clean command.
  && apk cache clean

# copy python dependency wheels from python-build-stage
COPY --from=python-build-stage /usr/src/app/wheels /wheels

# use wheels to install python dependencies
RUN pip install --no-cache /wheels/* \
  && rm -rf /wheels

COPY ./compose/local/docs/start /start-docs
RUN sed -i 's/\r$//g' /start-docs
RUN chmod +x /start-docs

WORKDIR /docs
