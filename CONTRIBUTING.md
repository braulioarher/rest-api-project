# CONTRIBUTING

## How to create a Docker image locally

`docker build -t rest-apis-flask-python .`

## How to run the Dockerfile locally

`docker run -dp 5000:5000 -w /app -v "$(pwd):/app" rest-apis-flask-python sh -c "flask run --host 0.0.0.0"`

Para poder correr localmente nuestro proyecto usando una base de datos remota dentro de nuestro Dockerfile usaremos el comando `CMD ["gunicorn", "--bind", "0.0.0.0:80", "app:create_app()"]`
