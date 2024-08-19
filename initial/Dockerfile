FROM maven:3.8.6-amazoncorretto-17 AS build

# Set the working directory in the container
WORKDIR /app

# Copy the pom.xml and download the dependencie
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy the source code into the container
COPY src ./src

# Package the application
RUN mvn package -DskipTests

# Use Amazon Corretto 17 JDK for the runtime image
FROM amazoncorretto:17-alpine

# Set the working directory in the container
WORKDIR /app

# Copy the WAR file from the build image
COPY --from=build /app/target/spring-boot-initial-0.0.1-SNAPSHOT.war app.war

# Expose the port the application will run on
EXPOSE 8080

# Command to run the application
ENTRYPOINT ["java", "-jar", "app.war"]
