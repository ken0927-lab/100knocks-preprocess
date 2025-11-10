FROM postgres:15.1-bullseye

# 修正点: findコマンドによるパーミッションエラーを無視するため '|| true' を追加
RUN sed -i -E 's/# (ja_JP.UTF-8)/\1/' /etc/locale.gen \
    && locale-gen \
    && find / -type f -perm /u+s -ignore_readdir_race -exec chmod u-s {} \; \
    && find / -type f -perm /g+s -ignore_readdir_race -exec chmod g-s {} \; || true

ENV LANG ja_JP.utf8

USER postgres

HEALTHCHECK --interval=5s --retries=20 CMD ["pg_isready"]
