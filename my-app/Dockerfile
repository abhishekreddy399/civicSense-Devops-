# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
# Set the API URL for the build
ARG REACT_APP_API_URL=http://localhost:5001/api
ENV REACT_APP_API_URL=$REACT_APP_API_URL
RUN npm run build

# Serve stage
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
# Copy nginx config if needed (e.g. for React Router)
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
