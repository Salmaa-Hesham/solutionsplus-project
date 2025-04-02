# Stage 1: Build the application
FROM node:20-alpine AS build

# Set the working directory
WORKDIR /app

# Create a non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the working directory
COPY . .

# Build the application
RUN npm run build-all

# Stage 2: Run the application
FROM node:20-alpine

# Set the working directory
WORKDIR /app

# Create the non-root user in the runtime image
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Ensure necessary directories have the correct permissions
RUN mkdir -p /app/logs && chown -R appuser:appgroup /app

# Copy the build output from the previous stage
COPY --from=build /app/build ./build
COPY --from=build /app/views ./views
COPY --from=build /app/public ./public
COPY --from=build /app/ecosystem.config.js ./ecosystem.config.js

# Copy node_modules from the build stage (no need to reinstall)
COPY --from=build /app/node_modules ./node_modules

# Use the non-root user
USER appuser

# Expose the application port
EXPOSE 3000

# Use local PM2 instead of global install
CMD ["npx", "pm2-runtime", "start", "ecosystem.config.js", "--env", "prod"]
