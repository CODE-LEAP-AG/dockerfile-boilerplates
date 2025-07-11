ARG NODE_VERSION=22.16.0
ARG ANGULAR_VERSION=19

# Build stage
FROM --platform=$BUILDPLATFORM node:${NODE_VERSION}-alpine AS build

# Set the working directory for the build process
WORKDIR /app

# Copy only necessary files to install dependencies, this helps in caching layers
# Use --link to improve performance of caching, recommended in most cases
# For more information of --link, see https://docs.docker.com/reference/dockerfile/#benefits-of-using---link
COPY --link package.json yarn.lock* package-lock.json* pnpm-lock.yaml* .npmrc* ./

# Install dependencies
# Use the appropriate package manager based on the lock file present
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm i --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Install Angular CLI globally
RUN npm install -g @angular/cli@${ANGULAR_VERSION}

# Copy the remaining source code
COPY --link . .

# Build the Angular application for production
# This will create a build in the dist/angular/browser directory
RUN \
  if [ -f yarn.lock ]; then yarn run build; \
  elif [ -f package-lock.json ]; then npm run build; \
  elif [ -f pnpm-lock.yaml ]; then corepack enable pnpm && pnpm run build; \
  else echo "Lockfile not found." && exit 1; \
  fi

# Production stage
# Use Nginx to serve the built Angular application
# Use the unprivileged Nginx image to enhance security
FROM nginxinc/nginx-unprivileged:alpine-perl

# Copy the build output from the build stage.
COPY --link --from=build /app/dist/angular/browser /usr/share/nginx/html

# Indicate that Nginx serves the app on port 8080
EXPOSE 8080

# Run Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]