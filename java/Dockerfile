# Use Maven to build the project
FROM maven:3.6.3-openjdk-11 as build

# Copy source code and pom files
COPY digitalcrafting-base/pom.xml /build/digitalcrafting-base/pom.xml
COPY basic-web-app-java/pom.xml /build/basic-web-app-java/pom.xml

# Copy the source code of digitalcrafting-base and install it
COPY digitalcrafting-base/src /build/digitalcrafting-base/src
WORKDIR /build/digitalcrafting-base
RUN mvn install

# Copy the source code of basic-web-app-java and package it
COPY basic-web-app-java/src /build/basic-web-app-java/src
WORKDIR /build/basic-web-app-java
RUN mvn package

# Final stage: Create the Docker container with Java runtime
FROM openjdk:11-jre-slim
WORKDIR /app

# Copy the JAR file from the build stage
COPY --from=build /build/basic-web-app-java/target/*.jar /app/app.jar

# Expose the port on which the web app will run
EXPOSE 8080

# Run the jar file
CMD ["java", "-jar", "app.jar"]
