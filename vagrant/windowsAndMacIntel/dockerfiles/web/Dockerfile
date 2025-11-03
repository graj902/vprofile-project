FROM nginx:latest

LABEL project="vprofile"

# Remove default Nginx configuration
RUN rm -rf /etc/nginx/conf.d/default.conf

# Copy our custom configuration
COPY nginvproapp.conf /etc/nginx/conf.d/vproapp.conf