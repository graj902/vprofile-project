# PHASE 1: Build the code
# "openjdk:8" is deprecated. We use "eclipse-temurin:8-jdk" instead.
FROM eclipse-temurin:8-jdk AS build_image

# Install Maven and Git
RUN apt-get update && apt-get install -y maven git

# Clone your specific branch
RUN git clone -b jenkins-cicd https://github.com/graj902/vprofile-project.git

# Build the artifact
RUN cd vprofile-project && mvn install

# PHASE 2: Deploy to Tomcat
FROM tomcat:9-jre11
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy the artifact from the build_image phase
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]