# Use the SDK image for building
# Specifying the build platform using --platform for cross-compilation
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build

# Defines the target architecture for cross-platform builds 
ARG TARGETARCH

# Set the working directory for the build process
WORKDIR /source

# Copy only necessary files to restore dependencies, this helps in caching layers
# Use --link to improve performance of caching, recommended in most cases
# For more information of --link, see https://docs.docker.com/reference/dockerfile/#benefits-of-using---link
COPY --link *.csproj .
RUN dotnet restore -a $TARGETARCH

# Copy the remaining source code
COPY --link . .

# Publish the app (with --no-restore flag as dependencies were restored earlier)
RUN dotnet publish -a $TARGETARCH --no-restore -o /app

# We do not need the SDK in the final image, only the runtime
# This helps reduce the image size
FROM mcr.microsoft.com/dotnet/aspnet:8.0 as final

# Indicate that the app runs on port 8080 (Optional)
EXPOSE 8080

# Set the working directory for the runtime environment
WORKDIR /app

# Copy the published app from the build stage
COPY --link --from=build /app .

# Ensure the container runs as a non-root user, improving security
USER $APP_UID

# Run the application
ENTRYPOINT ["./dotnet"]