# Stage 1: Build the artifact
FROM maven:3.9.9-eclipse-temurin-21-jammy AS build_image

# Clone the source code
RUN git  clone https://github.com/graj902/vprofile-project.git

# CD into project, checkout branch, and build artifact
# Using && to combine commands (creates single layer - smaller image)
RUN cd vprofile-project && \
    git checkout containers && \
    mvn install

# Stage 2: Create the Tomcat runtime image
FROM tomcat:10-jdk21

# Remove default Tomcat applications
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy artifact from build stage to Tomcat webapps as ROOT.war
COPY --from=build_image vprofile-project/target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war

# Expose port 8080 (documentation purpose)
EXPOSE 8080

# Start Tomcat (this is default in tomcat image, but shown for clarity)
CMD ["catalina.sh", "run"]