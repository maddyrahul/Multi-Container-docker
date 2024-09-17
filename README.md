# Multi-Container .NET Web Application

This project demonstrates how to create, containerize, and manage a multi-container .NET web application using Docker and Docker Compose. The application includes two services:

- **api-service**
- **worker-service**

This README file provides a complete guide for setting up, running, and managing the application, as well as instructions for pushing and pulling Docker images.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Setup and Installation](#setup-and-installation)
3. [Creating Dockerfiles](#creating-dockerfiles)
   - [Dockerfile for api-service](#dockerfile-for-api-service)
   - [Dockerfile for worker-service](#dockerfile-for-worker-service)
4. [Building Docker Images](#building-docker-images)
5. [Running Containers Locally](#running-containers-locally)
6. [Using Docker Compose](#using-docker-compose)
7. [Pushing Images to Docker Hub](#pushing-images-to-docker-hub)
8. [Pulling and Running Images from Docker Hub](#pulling-and-running-images-from-docker-hub)

---

## Prerequisites

Ensure the following tools are installed before proceeding:

- **Docker**: Version 20.10 or later
- **Docker Compose**: Version 1.29 or later
- **.NET SDK**: Version 6.0 or later
- **Git**: For version control
- **GitHub Account**: For repository and project management
- **Docker Hub Account**: For image storage and management



## Setup and Installation

# Clone the Repository

- git clone https://github.com/maddyrahul/Multi-Container-docker.git
- cd Docker

# Creating Dockerfiles
  Dockerfile for api-service
  
# In the api-service/ directory:

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["api-service.csproj", "."]
RUN dotnet restore "./api-service.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "./api-service.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./api-service.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "api-service.dll"]


## Dockerfile for worker-service
In the worker-service/ directory:

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["worker-service.csproj", "."]
RUN dotnet restore "./worker-service.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "./worker-service.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./worker-service.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "worker-service.dll"]


## Building Docker Images
# Build Docker Image for api-service

cd api-service
docker build -t api-service:latest .
Build Docker Image for worker-service

cd ../worker-service
docker build -t worker-service:latest .

---
# Running Containers Locally

# Run Container for api-service
docker run -d -p 7000:80 api-service:latest

# Run Container for worker-service
docker run -d -p 8000:80 worker-service:latest

---
## Access the services at:
http://localhost:7000 for api-service
http://localhost:8000 for worker-service

---
## Using Docker Compose
# Create docker-compose.yml

In the project root directory:
version: '3.8'

services:
  api-service:
    image: ${DOCKER_REGISTRY-}api-service
    build:
      context: .
      dockerfile: api-service/Dockerfile
    ports:
      - "7000:80"

  worker-service:
    image: ${DOCKER_REGISTRY-}worker-service
    build:
      context: .
      dockerfile: worker-service/Dockerfile
    ports:
      - "8000:80"


## Build and Bring Up the Application
docker-compose up --build

# The services will be accessible at:

http://localhost:7000 for api-service
http://localhost:8000 for worker-service

# Stopping the Services
docker-compose down

## Pushing Images to Docker Hub
# Log in to Docker Hub

docker login
Tag and Push Images


## For api-service:

docker tag api-service:latest rahulmaddy123/api-service:latest
docker push rahulmaddy123/api-service:latest

# For worker-service:
docker tag worker-service:latest rahulmaddy123/worker-service:latest
docker push rahulmaddy123/worker-service:latest


## Pulling and Running Images from Docker Hub
# Pull Images
For api-service:
docker pull rahulmaddy123/api-service:latest

For worker-service:
docker pull rahulmaddy123/worker-service:latest

# Run Containers
For api-service:
docker run -d -p 7000:80 rahulmaddy123/api-service:latest

For worker-service:
docker run -d -p 8000:80 rahulmaddy123/worker-service:latest
