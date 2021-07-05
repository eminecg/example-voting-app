FROM maven:3.6.1-jdk-8-slim
WORKDIR /app

COPY . .

RUN mvn package -DskipTests && / mvn target/worker-jar-with-dependencies.jar /run/worker.jar  && / mvn -rf *


CMD java -jar /run/worker.jar
