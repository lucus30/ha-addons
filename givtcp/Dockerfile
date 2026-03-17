FROM python:3.11-alpine

RUN apk add --update --no-cache mosquitto npm git tzdata musl-utils redis nginx \
    && mkdir -p /run/nginx

RUN npm install -g serve

WORKDIR /app

# Pull all source files from giv_tcp repo, stripping any HA addon config files
RUN git clone https://github.com/lucus30/giv_tcp.git /tmp/giv_tcp \
    && find /tmp/giv_tcp -name "config.yaml" -delete \
    && find /tmp/giv_tcp -name "repository.yaml" -delete

# Copy everything into place
RUN cp /tmp/giv_tcp/requirements.txt . \
    && cp -r /tmp/giv_tcp/GivTCP ./GivTCP \
    && cp -r /tmp/giv_tcp/WebDashboard ./WebDashboard \
    && cp -r /tmp/giv_tcp/givtcp-vuejs ./givtcp-vuejs \
    && cp -r /tmp/giv_tcp/ingress ./ingress \
    && cp /tmp/giv_tcp/ingress.conf /etc/nginx/http.d/ \
    && cp /tmp/giv_tcp/ingress_no_ssl.conf . \
    && cp /tmp/giv_tcp/api.json ./GivTCP/api.json \
    && cp /tmp/giv_tcp/redis.conf . \
    && cp /tmp/giv_tcp/settings.json . \
    && cp /tmp/giv_tcp/startup.py startup.py \
    && rm -rf /tmp/giv_tcp

RUN pip install -r requirements.txt

# Build the Vue.js frontend
RUN cd /app/givtcp-vuejs && npm install
RUN cd /app/givtcp-vuejs && npm run build \
    && mv dist/index.html dist/config.html \
    && cp -a dist/. /app/ingress/

RUN cp -r /app/GivTCP/givenergy_modbus_async/ /usr/local/lib/python3.11/site-packages/givenergy_modbus_async

RUN rm /etc/nginx/http.d/default.conf

EXPOSE 1883 6379 8099
CMD ["python3", "/app/startup.py"]
