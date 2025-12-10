# PHASE 1: Build the code
# We use lowercase 'build_image' to avoid plugin errors
FROM openjdk:8 AS build_image
RUN apt update && apt install maven -y
# Clone your specific branch
RUN git clone -b jenkins-cicd https://github.com/graj902/vprofile-project.git
RUN cd vprofile-project && mvn install

# PHASE 2: Deploy to Tomcat
FROM tomcat:9-jre11
RUN rm -rf /usr/local/tomcat/webapps/*
# Copy the artifact from the build_image phase
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]