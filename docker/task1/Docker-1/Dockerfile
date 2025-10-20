FROM maven:3.9.4-sapmachine-17 
WORKDIR /app
COPY . .
RUN mvn package
CMD ["java","-jar","target/demo-0.0.1-SNAPSHOT.jar"]
EXPOSE 8080
