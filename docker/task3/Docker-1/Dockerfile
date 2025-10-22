# -------- Stage 1: Build --------
FROM maven:3.9.4-sapmachine-17 AS Build
WORKDIR /app
COPY . .
RUN mvn package 
# -------- Stage 2: Run --------
FROM sapmachine:17-jdk AS Run
WORKDIR /app
COPY --from=Build /app/target/demo-0.0.1-SNAPSHOT.jar app.jar
CMD ["java","-jar","app.jar"]
EXPOSE 8080

