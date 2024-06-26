# TestMelow
Secuenciador campaña virtual
# CampaignFlow - Creador de Campañas de Marketing

CampaignFlow es una herramienta interactiva diseñada para facilitar la creación y gestión de campañas de marketing digitales. Con una interfaz dinámica basada en nodos, permite a los usuarios configurar flujos complejos que simulan el recorrido de un cliente dentro de una campaña promocional. 

## Características Principales

- **Creador de Flujos de Nodos**: Define los pasos de una campaña a través de nodos interconectados que representan diferentes acciones.
- **Acciones Diversificadas**: Cada nodo puede realizar acciones específicas, como enviar correos electrónicos (Mailer) o establecer periodos de espera (Wait), permitiendo un seguimiento personalizado de cada cliente.
- **Condiciones Personalizables**: Establece reglas para cada campaña basadas en `last_login`, `created_at`, y `page_visit`, que determinan la participación de los usuarios en la campaña.
- **Simulación de Actividad**: Genera actividades de usuario ficticias para probar y visualizar el flujo de la campaña.
- **Worker Automatizado**: Un proceso en segundo plano (worker) verifica periódicamente si los usuarios cumplen con las condiciones establecidas para avanzar en el flujo de la campaña.

## Uso Ideal

CampaignFlow es ideal para empresas y equipos de marketing que buscan optimizar la interacción con sus clientes a través de campañas automatizadas y altamente personalizables. Su capacidad para simular y evaluar condiciones específicas hace que sea una herramienta valiosa para estrategias de marketing dirigidas y efectivas.

## Cómo Funciona

1. **Crear una Campaña**: Define las características de la campaña y configura los nodos con sus respectivas acciones.
2. **Establecer Condiciones**: Personaliza las condiciones bajo las cuales los usuarios entrarán en el flujo de la campaña.
3. **Simular Actividades**: Crea actividades de usuario simuladas para entender cómo interactúan con la campaña.
4. **Evaluación Automatizada**: El worker ejecuta la lógica de procesamiento para mover a los usuarios a través del flujo basándose en sus actividades y condiciones cumplidas.
   
## Paso a Paso
### 0. Instalacion y Configuracion

Antes de comenzar con la creación y simulación de campañas, necesitas instalar y configurar el proyecto en tu entorno local. Aquí te explicamos cómo:

El proyecto esta configurado para poder ejecutarse mediante docker, para esto hay que realizar los siguientes comandos en la terminal.

```bash
git clone https://github.com/JoaquinBurgos/CampaignService.git
docker-compose build
docker-compose up
```
Una vez que esten los containers arriba, ejecutar comandos para setup de la database del backend en otra terminal
```
docker exec -it campaignservice-backend-1 rails db:create
docker exec -it campaignservice-backend-1 rails db:migrate
```
### Tareas asincronicas de Sidekiq
Para la realización de validación y avance de usuarios por los nodos de las campañas se utiliza Sidekiq, el cual realiza una acción cada cierto tiempo. Este intervalo se puede ajustar en el archivo config/sidekiq_scheduler.yml, modificando el atributo every, que por defecto está configurado para ejecutarse cada 5 segundos.
```bash
docker exec -it campaignservice-backend-1 bundle exec sidekiq -C config/sidekiq_scheduler.yml
## IMPORTANTE QUE EL SERVIDOR REDIS ESTE FUNCIONANDO
```
Una vez ejecutado el worker de Sidekiq, este funciona en tiempo real percibe los cambios realizados a la base de datos.
### 1. Crear una Campaña y Establecer Condiciones
El primer y segundo paso se realizan a través de la interfaz gráfica de CampaignFlow, donde podrás:

- Definir las características generales de la campaña.
- Crear nodos para construir el flujo de la campaña y definir acciones de Mailer o Wait.
- Personalizar las condiciones bajo las cuales los usuarios interactúan con la campaña (basadas en `last_login`, `created_at`, y `page_visit`).

### 2. Simular Actividades de Usuario

#### Crear Usuarios
Para simular las actividades de los usuarios, primero debes crear usuarios en la base de datos:

- Editar el archivo `seeds.rb` en el backend con los correos electrónicos y nombres de los usuarios que desees simular.
- Ejecutar el comando `docker exec -it campaignservice-backend-1 rails db:seed` en la consola, en el directorio CampaignService/campaign_service.
- La consola arrojara los id de los usuarios creados, los cuales usaremos para crear actividades de usuarios.
- La aplicacion cuenta con un servicio de mailer, por lo que es importante definir los emails bien para recibir estos.

#### Generar Actividades Ficticias
Con los usuarios ya creados, procederás a generar actividades que simulan la interacción con un e-commerce o negocio:

- Realizar peticiones POST a la ruta `/user_activities` utilizando Postman, tu cliente HTTP favorito o directamente en consola.
- Enviar datos estructurados como se muestra a continuación:

```bash
## Para la ultima vez que se logeo
curl -X POST http://localhost:3001/user_activities \
     -H 'Content-Type: application/json' \
     -d '{
           "user_activity": {
             "user_id": "ID_del_usuario_creado",
             "event_type": "last_login",
             "data": {
               "logged_at": "Fecha en formato ISO8601 o YYYY-MM-DD"
             }
           }
         }'
## Para el tiempo en el que se creo el user
  curl -X POST http://localhost:3001/user_activities \
     -H 'Content-Type: application/json' \
     -d '{
           "user_activity": {
             "user_id": "ID_del_usuario_creado",
             "event_type": "account_creation",
             "data": {
               "created_at": "Fecha en formato ISO8601 o YYYY-MM-DD"
             }
           }
         }'
## Para las rutas visitadas
  curl -X POST http://localhost:3001/user_activities \
     -H 'Content-Type: application/json' \
     -d '{
           "user_activity": {
             "user_id": "ID_del_usuario_creado",
             "event_type": "page_visit",
             "data": {
               "path": "/ruta_de_la_pagina"
             }
           }
         }'
```
