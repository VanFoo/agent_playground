FROM ubuntu:24.04

# squid-openssl (not the plain `squid` package) is required for ssl_bump /
# TLS-handshake inspection support (peek/stare/splice). On Debian/Ubuntu the
# two packages are built differently for licensing reasons; only the
# -openssl variant has the TLS inspection support squid.conf relies on.
RUN apt-get update && apt-get install -y --no-install-recommends \
        squid-openssl \
        openssl \
        netcat-openbsd \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Squid's ssl-bump engine requires a cert/key on the listening port even
# though this proxy never actually intercepts/decrypts application traffic
# (see squid.conf: only peek/stare/splice/terminate actions are used, never
# bump). This self-signed cert is never presented to any real client or
# server — it only satisfies Squid's ssl-bump engine setup requirements.
RUN openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 \
        -subj "/CN=devisolation-firewall-unused" \
        -keyout /etc/squid/dummy.pem -out /etc/squid/dummy.pem

RUN mkdir -p /var/log/squid /var/spool/squid \
    && chown -R proxy:proxy /var/log/squid /var/spool/squid /etc/squid/dummy.pem

EXPOSE 3128

CMD ["squid", "-N", "-f", "/etc/squid/squid.conf"]
