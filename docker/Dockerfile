FROM ubuntu:20.04 as BUILDER

RUN apt-get update && apt-get upgrade -y
RUN apt-get -y install apt-transport-https ca-certificates wget gnupg
RUN mkdir /usr/local/share/keyrings/ && \
    wget -q -O - "https://repos.ripple.com/repos/api/gpg/key/public" | gpg --dearmor > ripple-key.gpg && \
    mv ripple-key.gpg /usr/local/share/keyrings
RUN echo "deb [signed-by=/usr/local/share/keyrings/ripple-key.gpg] https://repos.ripple.com/repos/rippled-deb focal stable" | \
    tee -a /etc/apt/sources.list.d/ripple.list
RUN apt-get update
RUN apt-get install rippled

FROM ubuntu:20.04 AS final

ENV PATH=$PATH:/root/.local/bin:/root/.pyenv/bin
  # POETRY_VERSION=1.1.13 \

RUN apt-get update && apt-get install -y git curl htop vim python3-venv
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev


RUN curl https://pyenv.run | bash
RUN pyenv install 3.10.5
RUN pyenv global 3.10.5
RUN apt-get install -y tmux
RUN curl -sSL https://install.python-poetry.org | python3 -
# RUN git clone --depth=1 https://github.com/xrplf/xrpl-py.git
COPY . /xrpl-py
RUN echo 'export PYENV_ROOT="/root/.pyenv"' >> root/.bashrc
RUN echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> root/.bashrc
RUN echo 'eval "$(pyenv init -)"' >> root/.bashrc

WORKDIR /xrpl-py
RUN poetry install
# try multistage build
COPY --from=BUILDER /opt/ripple /opt/ripple
RUN apt-get install -y ncdu
ENV PATH=$PATH:/root/.local/bin:/root/.pyenv/bin

# CMD /opt/ripple/bin/rippled -a --start --conf /opt/ripple/etc/rippled.cfg & poetry run python3 -m unittest discover tests
CMD /opt/ripple/bin/rippled -a --start --conf /opt/ripple/etc/rippled.cfg & poetry run poe test_unit
