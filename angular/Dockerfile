# Stage 1: Build the Angular application
FROM node:14 as build-step

WORKDIR /app

# Copy the package.json and package-lock.json files
COPY package.json  ./
COPY package-lock.json* ./

# Install dependencies
RUN npm install

# Copy the rest of the Angular application files
COPY . .

# Build the project
RUN npm run build

# Stage 2: Serve the application with Nginx
FROM nginx:alpine

# Copy the build output to replace the default Nginx contents
COPY --from=build-step /app/dist/angular /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start Nginx and serve the application
CMD ["nginx", "-g", "daemon off;"]

