FROM openfaas/of-watchdog:0.7.3 as watchdog
FROM python:3-slim
# REFERENCE: 
# 1. https://github.com/LucasRoesler/pydatascience-template/blob/master/template/pydatascience/Dockerfile
# 2. https://pythonspeed.com/articles/base-image-python-docker-images/

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

ARG ADDITIONAL_PACKAGE
# Alternatively use ADD https:// (which will not be cached by Docker builder)
RUN apt-get update \
    && apt-get -y install ${ADDITIONAL_PACKAGE}

# Add the testing https://stackoverflow.com/a/50443531/8853476
# Chose to move away from alpine due to issues in installing numpy pandas etc and in future Gurobi.
# With alpine and without numpy pandas etc the size was 140 MB| Now it is 413MB
# TODO: Decrease image size somehow

# Add non root user
RUN groupadd -r app && useradd app -r -m -g app
RUN chown app /home/app

USER app

ENV PATH=$PATH:/home/app/.local/bin

WORKDIR /home/app/

USER app
COPY index.py           .
COPY requirements.txt   .
USER root
RUN pip install -r requirements.txt
USER app

RUN mkdir -p function
RUN touch ./function/__init__.py
WORKDIR /home/app/function/
COPY function/requirements.txt	.
RUN pip install --user -r requirements.txt

WORKDIR /home/app/

USER root
COPY function   function
RUN chown -R app:app ./
USER app

# Set up of-watchdog for HTTP mode
ENV fprocess="python index.py"
ENV cgi_headers="true"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:5000"

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]