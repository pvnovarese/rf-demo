FROM reg.mini.dev/python:3.14.7-dev@sha256:de70299cb1d08c333f3148f792fb1e4eebce2630987a0568723a62dd1e7c2af7 AS builder 

USER 0
RUN apk add --no-cache postgresql-dev jq

WORKDIR /app
COPY requirements.txt .
#RUN pip install --no-cache-dir -r requirements.txt --prefix=/install
# Use Chainguard Libraries index with credentials injected at build time
RUN --mount=type=secret,id=cg_username \
    --mount=type=secret,id=cg_password \
    CG_USER=$(jq -sRr @uri < /run/secrets/cg_username) && \
    CG_PASS=$(jq -sRr @uri < /run/secrets/cg_password) && \
    pip install --no-cache-dir --require-hashes \
    --index-url https://${CG_USER}:${CG_PASS}@libraries.cgr.dev/python-remediated/simple/ \
    --extra-index-url https://${CG_USER}:${CG_PASS}@libraries.cgr.dev/python/simple/ \
    --prefix=/install \
    -r requirements.txt

#    CG_USER=$(python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.stdin.read().strip(), safe=''))" < /run/secrets/cg_username) && \
#    CG_PASS=$(python3 -c "import urllib.parse,sys; print(urllib.parse.quote(sys.stdin.read().strip(), safe=''))" < /run/secrets/cg_password) && \

#    --index-url https://$(cat /run/secrets/cg_username):$(cat /run/secrets/cg_password)@libraries.cgr.dev/python-remediated/simple/ \
#    --extra-index-url https://$(cat /run/secrets/cg_username):$(cat /run/secrets/cg_password)@libraries.cgr.dev/python/simple/ \

FROM reg.mini.dev/python:3.14.7@sha256:44be0c48e9479a9cda05b4308503b2ca72c77f485b9f1626aadbdba92fe14b6e
COPY --from=builder /install/lib/python3.14/site-packages /usr/lib/python3.14/site-packages
WORKDIR /app
COPY --chown=nonroot:nonroot . /app

ENV FLASK_APP=app.py
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
#ENV FLASK_ENV=development
ENV FLASK_DEBUG=0

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"
ENTRYPOINT ["python", "/app/app.py"]

