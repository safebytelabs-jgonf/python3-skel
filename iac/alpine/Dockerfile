# ##################################### #
#                                       #
#   How-To build this container image   #
#                                       #
# ##################################### #
#
#
# Local build
# -----------
# docker build \
#   -f iac/Dockerfile-alpine \
#   --build-arg VERSION=`cat src/VERSION` \
#   --build-arg GIT_BRANCH=`git branch --show-current` \
#   -t ea1het/my_test_container:`cat src/VERSION` .
#
#
# Platform specific images (when you build for your own platform)
# ---------------------------------------------------------------
# docker build \
#   --build-arg BUILD_DATE=`date -u +"%Y-%m-%d"` \
#   --build-arg VERSION=`cat src/VERSION` \
#   --build-arg GIT_BRANCH=`git branch --show-current` \
#   -t registry.gitlab.com/ea1het/my_test_container:`cat src/VERSION` .
#
#
# Cross-compile platform independent images (when you use a build facility
# in your platform to cross-compile for another platform)
# ------------------------------------------------------------------------
# docker buildx ls
# docker buildx create --name testbuilder
# docker buildx use testbuilder
# docker buildx inspect --bootstrap
# docker buildx build \
#   --platform linux/arm/v6 \
#   --build-arg BUILD_DATE=`date -u +"%Y-%m-%d"` \
#   --build-arg VERSION=`cat src/VERSION` \
#   --build-arg GIT_BRANCH=`git branch --show-current` \
#   -t registry.gitlab.com/ea1het/my_test_container:`cat src/VERSION` --push .
#
# #############################################################################

FROM python:3.10.4-alpine3.15

LABEL maintainer="Jonathan Gonzalez <j@0x30.io>"
ENV RUN_DEPENDENCIES="python3 python3-dev py3-pip gcc build-base ensurepip"
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONFAULTHANDLER 1
ARG GIT_BRANCH

# Let service stop gracefully
STOPSIGNAL SIGQUIT

RUN apk add --no-cache $RUN_DEPENDENCIES \
    && python3 -m ensurepip              \
    && pip3 install -U pip               \
    && rm -rf /usr/lib/python*/ensurepip \
    && rm -rf /root/.cache               \
    && rm -rf /var/cache/apk/*           \
    && find /                            \
        \( -type d -a -name test -o -name tests \) \
        -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
        -exec rm -rf '{}' + \
    && mkdir /app

# - Install virtualenv wrapper and execute it
RUN pip3 install virtualenv     \
    && virtualenv venv          \
    && source venv/bin/activate

# - Copy project files into working directory
WORKDIR /app/
COPY ./src/ ./src/
COPY ./events/ ./events/
COPY ./requirements/ ./requirements/
COPY ./requirements/$GIT_BRANCH.txt ./requirements.txt

# - Install project requirements
RUN pip install --no-cache-dir -r requirements.txt

# - Run the program
WORKDIR /app/src
CMD [ "python3", "app.py" ]
