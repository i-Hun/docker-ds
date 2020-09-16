# RUN curl -fsSL https://code-server.dev/install.sh | sh
# RUN code-server --install-extension ms-python.python

docker build -t nagornyy/ds -f ./Dockerfile .

docker run -d \
              -v ${PWD}:/app \
              -v /srv/gpdb:/srv/gpdb \
              -v /etc/passwd:/etc/passwd \
              -v /mnt/s3:/mnt/s3 \
              -v /etc/localtime:/etc/localtime:ro \
              -e HOME=/tmp \
              -p 12101:12101 \
              -p 12102:12102 \
              -p 12103:12103 \
              -p 12104:12104 \
              -p 12105:12105 \
              -p 12106:12106 \
              -p 12107:12107 \
              -p 12108:12108 \
              -p 12109:12109 \
              -p 12110:8080 \
              -u ${UID} \
              --env-file ../docker/environments.list \
              --name nagornyy_container_12101_12110 \
              nagornyy/ds:latest \
              bash -c "[ -f /app/requirements.txt ] && \
                       pip install -r /app/requirements.txt || true && \
                       echo 'ulimit -c 0' > .project-rc && \
                       touch /app/jupyter_local.log && \
                       jupyter lab --port=12101 \
                       --notebook-dir=/app \
                       --ip=0.0.0.0 \
                       --no-browser \
                       --NotebookApp.password='' \
                       --allow-root >> /app/jupyter_local.log 2>&1 " > /dev/null


# запуск code-server --bind-addr 0.0.0.0:8080
