FROM ubuntu:20.04
RUN [ -z "$(apt-get indextargets)" ]
RUN set -xe && echo '#!/bin/sh' > /usr/sbin/policy-rc.d && echo 'exit 101' >> /usr/sbin/policy-rc.d && chmod +x /usr/sbin/policy-rc.d && dpkg-divert --local --rename --add /sbin/initctl && cp -a /usr/sbin/policy-rc.d /sbin/initctl && sed -i 's/^exit.*/exit 0/' /sbin/initctl && echo 'force-unsafe-io' > /etc/dpkg/dpkg.cfg.d/docker-apt-speedup && echo 'DPkg::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' > /etc/apt/apt.conf.d/docker-clean && echo 'APT::Update::Post-Invoke { "rm -f /var/cache/apt/archives/*.deb /var/cache/apt/archives/partial/*.deb /var/cache/apt/*.bin || true"; };' >> /etc/apt/apt.conf.d/docker-clean && echo 'Dir::Cache::pkgcache ""; Dir::Cache::srcpkgcache "";' >> /etc/apt/apt.conf.d/docker-clean && echo 'Acquire::Languages "none";' > /etc/apt/apt.conf.d/docker-no-languages && echo 'Acquire::GzipIndexes "true"; Acquire::CompressionTypes::Order:: "gz";' > /etc/apt/apt.conf.d/docker-gzip-indexes && echo 'Apt::AutoRemove::SuggestsImportant "false";' > /etc/apt/apt.conf.d/docker-autoremove-suggests
RUN mkdir -p /run/systemd && echo 'docker' > /run/systemd/container
CMD ["/bin/bash"]
ENV LANG=en_US.UTF-8 LANGUAGE=en_US:en LANGUAGE=en_US:en
RUN apt-get update && apt-get install -y software-properties-common apt-transport-https locales libaio1 libaio-dev wget unzip ca-certificates && locale-gen en_US.UTF-8 && update-locale LANG=en_US.UTF-8 && add-apt-repository ppa:deadsnakes/ppa && add-apt-repository ppa:neovim-ppa/unstable # buildkit
RUN mkdir -p /opt/oracle && cd /opt/oracle && wget https://download.oracle.com/otn_software/linux/instantclient/195000/instantclient-basic-linux.x64-19.5.0.0.0dbru.zip && unzip instantclient-basic-linux.x64-19.5.0.0.0dbru.zip && echo /opt/oracle/instantclient_19_5 > /etc/ld.so.conf.d/oracle-instantclient.conf && ldconfig && rm -rf instantclient-basic-linux.x64-19.5.0.0.0dbru.zip # buildkit
RUN apt-get update && apt-get install -y build-essential python3.7 python3.7-dev python3-pip python3.7-venv python3-psycopg2 freetds-dev unixodbc unixodbc-dev libssl-dev libffi-dev libxml2-dev libxslt1-dev zlib1g-dev libz-dev vim nano tmux screen curl sshfs fuse nfs-common make htop cmake ffmpeg build-essential libboost-all-dev git silversearcher-ag neovim exuberant-ctags python3-neovim && python3.7 -m pip install pip --upgrade # buildkit
RUN sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - && curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list > /etc/apt/sources.list.d/mssql-release.list && apt-get update && ACCEPT_EULA=Y apt-get install msodbcsql17 # buildkit
ENV TINI_VERSION=v0.18.0
RUN FILE=/usr/local/bin/supercronic && curl "https://github.com/aptible/supercronic/releases/download/v0.1.6/supercronic-linux-amd64" -Lo "$FILE" && echo "c3b78d342e5413ad39092fd3cfc083a85f5e2b75 $FILE" | sha1sum -c - && chmod +x "$FILE" && FILE=/tini && curl "https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini" -Lo "$FILE" && chmod +x /tini # buildkit
#COPY requirements.txt /app/ # buildkit
#RUN pip3 install --no-cache-dir -r /app/requirements.txt # buildkit
RUN pip3 install pandas matplotlib plotly flake8 jupyter-lsp pydocstyle pyflakes # buildkit
RUN ln -sf /usr/bin/python3.7 /usr/bin/python # buildkit
WORKDIR /app
#COPY pip.conf /root/.config/pip/ # buildkit
RUN echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list && curl https://bazel.build/bazel-release.pub.gpg | apt-key add - # buildkit
RUN apt-get update && apt-get install -y bazel && rm -rf /var/lib/apt/lists/* # buildkit
ENV WORKSPACE=/app

RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - && apt-get update && apt-get install nodejs && pip install jupyterlab "ipywidgets>=7.5" jupyterlab-git && jupyter labextension install jupyterlab-plotly && jupyter labextension install @jupyter-widgets/jupyterlab-manager plotlywidget && pip install psutil requests && npm install -g electron@6.1.4 orca --unsafe-perm=true && jupyter labextension install jupyterlab-execute-time && jupyter labextension install jupyterlab-spreadsheet && jupyter labextension install @jupyterlab/git && jupyter labextension install @krassowski/jupyterlab-lsp && pip install python-language-server[all] && jupyter labextension install jupyterlab_theme_onedark && jupyter labextension install DavidZhang73/jupyterlab-atom-one-dark-theme && jupyter labextension install @telamonian/theme-darcula && jupyter lab build
RUN python -c 'from pathlib import Path; Path("~/.jupyter/lab/user-settings/@krassowski/jupyterlab-lsp").mkdir(parents=True, exist_ok=True)' && echo '{"language_servers": {"pyls": {"serverSettings": {"pyls.plugins.pydocstyle.enabled": false,"pyls.plugins.pyflakes.enabled": false,"pyls.plugins.flake8.enabled": true}},"r-languageserver": {"serverSettings": {"r.lsp.debug": false,"r.lsp.diagnostics": false}}}}' > '~/.jupyter/lab/user-settings/@krassowski/jupyterlab-lsp/plugin.jupyterlab-settings' && python -c 'from pathlib import Path; Path("~/.jupyter/lab/user-settings/@jupyterlab/notebook-extension").mkdir(parents=True, exist_ok=True)' && echo '{"recordTiming": true}' > '~/.jupyter/lab/user-settings/@jupyterlab/notebook-extension/tracker.jupyterlab-settings' && jupyter lab build  # buildkit 


RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9 && add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/' && apt update && apt install r-base -y
RUN R -e "install.packages(c('IRkernel', 'languageserver', 'devtools', 'arrow', 'reticulate', 'xlsx', 'tidyverse', 'caret', 'data.table'), dependencies=TRUE, repos='http://cran.rstudio.com/'); IRkernel::installspec(user = FALSE)"
#RUN mkdir $HOME/.config
#ADD ../flake8 $HOME/.config/ # buildkit
