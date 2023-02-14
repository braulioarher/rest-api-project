# CONTRIBUTING

## How to create a Docker image locally

`docker build -t rest-apis-flask-python .`

## How to run the Dockerfile locally

`docker run -p 5000:5005 -w /app -v "$(pwd):/app" rest-apis-flask-python sh -c "flask run"`
