FROM maven:3.9.8-eclipse-temurin-21-alpine
WORKDIR /app
COPY . .
RUN mvn package && \
    mv target/worker-jar-with-dependencies.jar /run/worker.jar &&\
    rm -rf /app/* /root/.m2/repository
CMD ["java", "-jar", "/run/worker.jar"]
