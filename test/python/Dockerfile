ARG IMAGE=containerbase/base
ARG BUILDPACK_DEBUG

FROM ${IMAGE} as base

RUN touch /.dummy

COPY --chown=1000:0 test test

WORKDIR /test

FROM base as build

ARG APT_HTTP_PROXY
ARG BUILDPACK_DEBUG

# Python
# renovate: datasource=github-releases packageName=containerbase/python-prebuild
RUN install-tool python 3.11.1


FROM base as build-rootless

ARG APT_HTTP_PROXY
ARG BUILDPACK_DEBUG

RUN prepare-tool python

USER 1000

# renovate: datasource=github-releases packageName=containerbase/python-prebuild
RUN install-tool python 3.11.1

#--------------------------------------
# build: pipenv
#--------------------------------------
FROM build as pipenv

ARG BUILDPACK_DEBUG

# renovate: datasource=pypi
RUN install-tool pipenv 2022.12.19


USER 1000

#--------------------------------------
# build: poetry
#--------------------------------------
FROM build as poetry
ARG BUILDPACK_DEBUG


# renovate: datasource=pypi
RUN install-tool poetry 1.3.2

USER 1000

#--------------------------------------
# test poetry-a: build
#--------------------------------------
FROM build as test-poetry-a

ARG BUILDPACK_DEBUG

# try install again, sould skip
# renovate: datasource=github-releases packageName=containerbase/python-prebuild
RUN install-tool python 3.11.1

# renovate: datasource=pypi
RUN install-tool poetry 1.3.2

USER 1000

SHELL [ "/bin/sh", "-c" ]
RUN python --version
RUN poetry --version

#--------------------------------------
# test pipenv-a: pipenv
#--------------------------------------
FROM pipenv as test-pipenv-a

RUN set -ex; \
  cd a; \
  pipenv lock;


#--------------------------------------
# test poetry-b: c-poetry
#--------------------------------------
FROM poetry as test-poetry-b

RUN set -ex; cd c-poetry && poetry update --lock --no-interaction


RUN set -ex; cd c-poetry && poetry add h3py


#--------------------------------------
# test poetry-c: d-poetry
#--------------------------------------
FROM poetry as test-poetry-c

RUN set -ex; cd d-poetry && poetry update --lock --no-interaction pytest


#--------------------------------------
# test poetry-d: poetry (old versions)
#--------------------------------------
FROM build as test-poetry-d

ARG BUILDPACK_DEBUG

RUN install-tool poetry 0.12.17
RUN install-tool poetry 1.1.15

#--------------------------------------
# test a: pip_requirements
#--------------------------------------
FROM build as testa

RUN pip install hashin

RUN set -ex; \
  cd f; \
  hashin distribute==0.6.27; \
  cat requirements.txt

#--------------------------------------
# test b: non-root
#--------------------------------------
FROM build-rootless as testb

ARG BUILDPACK_DEBUG

# renovate: datasource=pypi
RUN install-pip pipenv 2022.12.19

RUN set -ex; \
  cd a; \
  pipenv lock;

# renovate: datasource=pypi
RUN install-tool poetry 1.3.2

RUN set -ex \
  && cd c-poetry \
  && poetry update --lock --no-interaction \
  && poetry add h3py \
  ;

# renovate: datasource=pypi
RUN install-tool hashin 0.17.0

RUN set -ex \
  && cd f \
  && hashin distribute==0.6.27 \
  && cat requirements.txt \
  ;

#--------------------------------------
# test pipenv-b: pipenv (multiple python)
#--------------------------------------
FROM build as test-pipenv-b

ARG BUILDPACK_DEBUG

# Do not update minor
RUN install-tool python 3.8.13

# make as latest
# renovate: datasource=github-releases packageName=containerbase/python-prebuild
RUN install-tool python 3.11.1

# renovate: datasource=pypi
RUN install-tool pipenv 2022.12.19

RUN set -ex; \
  cd h-pipenv; \
  pipenv lock;

#--------------------------------------
# test c: python2.7
#--------------------------------------
FROM build-rootless as testc

ARG BUILDPACK_DEBUG

# needs to be v2
RUN install-tool python 2.7.18

# don't change, never versions don't work on python 2.7
RUN install-tool pipenv 2020.11.15
RUN install-tool poetry 0.12.17
RUN install-tool poetry 1.1.15
RUN install-tool hashin 0.17.0
RUN install-tool pip-tools 5.5.0

# https://github.com/python-poetry/poetry/issues/3010
RUN poetry config experimental.new-installer false

RUN set -ex; \
  poetry new poetry-demo; \
  cd poetry-demo; \
  poetry update --lock --no-interaction pytest; \
  true

#--------------------------------------
# final
#--------------------------------------
FROM base

COPY --from=test-poetry-a /.dummy /.dummy
COPY --from=test-poetry-b /.dummy /.dummy
COPY --from=test-poetry-c /.dummy /.dummy
COPY --from=test-poetry-d /.dummy /.dummy
COPY --from=test-pipenv-a /.dummy /.dummy
COPY --from=test-pipenv-b /.dummy /.dummy
COPY --from=testa /.dummy /.dummy
COPY --from=testb /.dummy /.dummy
COPY --from=testc /.dummy /.dummy
