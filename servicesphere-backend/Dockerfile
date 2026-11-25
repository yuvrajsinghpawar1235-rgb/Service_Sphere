# Stage 1: Build the Spring Boot JAR with Java 21 & Maven
FROM maven:3.9.6-eclipse-temurin-21 AS build
WORKDIR /app

# Route central repository requests through Google's Maven Central mirror
RUN mkdir -p /root/.m2 && \
    printf '<settings><mirrors><mirror><id>google-maven-central</id><name>Google Maven Central Mirror</name><url>https://maven-central.storage-download.googleapis.com/maven2/</url><mirrorOf>central</mirrorOf></mirror></mirrors></settings>' > /root/.m2/settings.xml

# Copy pom.xml first so dependency downloads are cached across code-only changes
COPY pom.xml .
RUN for i in 1 2 3 4 5; do \
      mvn dependency:go-offline -B && break; \
      echo "Dependency download attempt $i failed, retrying in 10s..."; \
      sleep 10; \
    done

# Copy source code and package the JAR (skipping tests for faster deployment)
COPY src ./src
RUN for i in 1 2 3 4 5; do \
      mvn clean package -DskipTests -B && break; \
      echo "Maven package attempt $i failed, retrying in 10s..."; \
      sleep 10; \
    done

# Stage 2: Run the application on a lightweight Java 21 JRE image
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]