FROM mvn:3.8.1-jdk-11
WORKDIR /app

COPY . .

RUN mvn install  && \
    mvn audit fix

EXPOSE 4000

CMD worker.jar start
