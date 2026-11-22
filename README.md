# Proyecto Final - Sistemas Operativos

Infraestructura en contenedores Docker: Nginx como proxy inverso y balanceador de carga (Round Robin), tres backends en Python (Flask + Gunicorn), y monitoreo con cAdvisor, Nginx Prometheus Exporter, Prometheus y Grafana.

**Integrantes:** Samuel Canon, Santiago Bermudez, Samuel Ramirez

Requisitos previos

- Docker y Docker Compose v2 
- curl
- Puertos libres: 80, 3000, 8080, 9090

Levantar la infraestructura

    docker compose up -d --build
    docker compose ps

Para detenerla:

    docker compose down

URLS

| Servicio | URL |

| Aplicación (vía Nginx) | http://localhost 
| cAdvisor | http://localhost:8080 
| Prometheus | http://localhost:9090 (Status > Target health) 
| Grafana | http://localhost:3000 (acceso sin usuario, solo uso local) 

El puerto 8088 (stub_status de Nginx) y el 9113 (exporter) quedan dentro de la red de Docker y no se publican.


Verificar el balanceo de carga

Cada solicitud la atiende un backend distinto (Backend-1, Backend-2, Backend-3):

    for i in $(seq 1 6); do curl -s localhost | grep -o 'Backend-[0-9]'; done

Conteo de 30 solicitudes (debe quedar repartido en partes iguales):

    for i in $(seq 1 30); do curl -s localhost | grep -o 'Backend-[0-9]'; done | sort | uniq -c

    
Verificar el monitoreo

    docker compose exec nginx wget -qO- http://app1:5000/metrics
    docker compose exec nginx wget -qO- http://127.0.0.1:8088/stub_status

En http://localhost:9090, Status > Target health, deben aparecer 6 targets en UP: prometheus, app1, app2, app3, nginx-exporter y cadvisor.



Ejecutar la prueba de carga

Con la infraestructura arriba y el dashboard de Grafana abierto:

    docker compose run --rm artillery | tee resultados/resumen.txt

El escenario (`performance-test.yml`) tiene un calentamiento de 30 s (de 5 a 50 usuarios nuevos por segundo) y 60 s de carga sostenida (50 por segundo). El resumen queda en `resultados/resumen.txt` y los datos completos en `resultados/reporte.json`.
