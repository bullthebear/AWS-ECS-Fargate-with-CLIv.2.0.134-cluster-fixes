FROM nginx:alpine
WORKDIR /usr/share/nginx/html

# Copy everything manually — no builder, no Vite involved
COPY public/appconfig.json ./appconfig.json
COPY public ./public
COPY index.html .

RUN ls -l /usr/share/nginx/html
RUN cat /usr/share/nginx/html/appconfig.json

CMD ["nginx", "-g", "daemon off;"]
