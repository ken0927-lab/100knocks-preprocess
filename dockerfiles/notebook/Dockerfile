FROM quay.io/jupyter/datascience-notebook:python-3.11.9
#FROM jupyter/datascience-notebook:d53a302fbcd0

USER root
ENV DEBCONF_NOWARNINGS yes

# ベースOSが Ubuntu Noble (24.04) であるため、コードネームを 'noble' に固定
ARG DISTRO_CODENAME="noble"

# PostgreSQLクライアントライブラリ、MariaDBライブラリ、Kerberosライブラリのインストール
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
# STEP 6: 依存関係のインストールとセキュリティ処理 (findエラーは無視)
RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg wget libmariadb-dev libkrb5-dev \
    \
    # PostgreSQL GPGキーのダウンロードと配置
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /usr/share/keyrings/postgres-archive-keyring.gpg \
    \
    # APTソースリストにリポジトリと署名情報を追加
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
    # findコマンドがパーミッションエラーを出すのを回避するため、システムディレクトリを除外
    # u+s (Setuidビット) の解除
    && find / -path /proc -prune -o -path /sys -prune -o -path /dev -prune -o -path /run -prune -o \
        -type f -perm /u+s -ignore_readdir_race -exec chmod u-s {} \; \
    \
    # g+s (Setgidビット) の解除
    && find / -path /proc -prune -o -path /sys -prune -o -path /dev -prune -o -path /run -prune -o \
        -type f -perm /g+s -ignore_readdir_race -exec chmod g-s {} \; || true

#---

# ユーザー環境設定と依存関係のインストール (STEP 7 to 12)
USER jovyan
WORKDIR /home/jovyan
COPY requirements.txt .
COPY Pipfile .
COPY Pipfile.lock .
COPY install.R .

# STEP 13: Python, Rの依存関係をインストールし、ファイルをクリーンアップ
# 💡 修正済み: LD_PRELOAD の行が正しく継続/クォートされている
RUN pip install --no-cache-dir -r requirements.txt \
    && pipenv install --system \
    && LD_PRELOAD="/usr/lib/x86_64-linux-gnu/libssl.so:/usr/lib/x86_64-linux-gnu/libcrypto.so:/usr/lib/x86_64-linux-gnu/libgssapi_krb5.so" Rscript install.R || true \
    && rm -rf requirements.txt Pipfile* install.R .cache/R/pkgcache/sysreqs/docker/*/Dockerfile /tmp/* /var/tmp/* .npm/

# ヘルスチェックの設定
HEALTHCHECK --interval=5s --retries=20 CMD ["curl", "-s", "-S", "-o", "/dev/null", "http://localhost:8888"]
