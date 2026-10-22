# Base image
FROM python:3.8-slim-buster

# WOrking directory
WORKDIR /app

# Copy
COPY requirements.txt requirements.txt

# Command
RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m", "flask", "run", "--host:0.0.0.0" ]