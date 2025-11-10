FROM quay.io/jupyter/datascience-notebook:python-3.11.9
#FROM jupyter/datascience-notebook:d53a302fbcd0

USER root
ENV DEBCONF_NOWARNINGS yes

# ベースOSが Ubuntu Noble (24.04) であるため、PostgreSQLリポジトリのコードネームを 'noble' に設定
ARG DISTRO_CODENAME="noble"

# PostgreSQLクライアントライブラリのインストール
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN apt-get update \
    # 鍵取得とリポジトリ設定に必要なツールをインストール
    && apt-get install -y --no-install-recommends gnupg wget \
    \
    # PostgreSQL GPGキーのダウンロードと配置
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /usr/share/keyrings/postgres-archive-keyring.gpg \
    \
    # APTソースリストにリポジトリと署名情報を追加。コードネームに ${DISTRO_CODENAME} (noble) を使用
    && echo "deb [signed-by=/usr/share/keyrings/postgres-archive-keyring.gpg] http://apt.postgresql.org/pub/repos/apt ${DISTRO_CODENAME}-pgdg main" > /etc/apt/sources.list.d/pgdg.list \
    \
    # リポジトリ追加後の更新
    && apt-get update \
    \
    # libpq-dev (PostgreSQLクライアントライブラリ) のインストール
    && apt-get install -y --no-install-recommends libpq-dev \
    \
    # 不要なパッケージとキャッシュのクリーンアップ
    && apt-get remove -y gnupg wget \
    && apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* \
               /opt/julia-*/share/julia/stdlib/v*/SuiteSparse/.devcontainer/Dockerfile \
               /opt/julia/packages/Pluto/*/.vscode/ \
    \
    # 権限の変更 (元のコードを維持)
    && find / -type f -perm /u+s -ignore_readdir_race -exec chmod u-s {} \; \
    && find / -type f -perm /g+s -ignore_readdir_race -exec chmod g-s {} \;

---

## 💻 ユーザー環境設定と依存関係のインストール

```dockerfile
USER jovyan
WORKDIR /home/jovyan
COPY requirements.txt .
COPY Pipfile .
COPY Pipfile.lock .
COPY install.R .

# Python, Rの依存関係をインストールし、ファイルをクリーンアップ
RUN pip install --no-cache-dir -r requirements.txt \
    && pipenv install --system \
    && Rscript install.R \
    && rm -rf requirements.txt Pipfile* install.R .cache/R/pkgcache/sysreqs/docker/*/Dockerfile /tmp/* /var/tmp/* .npm/

# ヘルスチェックの設定
HEALTHCHECK --interval=5s --retries=20 CMD ["curl", "-s", "-S", "-o", "/dev/null", "http://localhost:8888"]
