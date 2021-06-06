FROM node:current-slim

# Copy source code to /src in container
COPY . /src

# Install app and dependencies into /src in container
RUN cd /src; npm install

# Listening port
EXPOSE 8080

# Start the app
CMD cd /src && node ./app.js
