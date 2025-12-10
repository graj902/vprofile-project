# PHASE 1: Build the code
# UPDATED: Using Java 17 to match pom.xml requirements
FROM eclipse-temurin:17-jdk AS build_image

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