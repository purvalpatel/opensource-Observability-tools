setup:
-------

docker-compose.yaml
```
version: "3.9"

services:
  phoenix:
    image: arizephoenix/phoenix:latest
    container_name: phoenix
    ports:
      - "6006:6006"
      - "4317:4317"
    networks:
     - observability
    volumes:
      - phoenix_data:/data
    tty: true
    stdin_open: true
    restart: unless-stopped

volumes:
  phoenix_data:
networks:
  observability:
    external: true

```
