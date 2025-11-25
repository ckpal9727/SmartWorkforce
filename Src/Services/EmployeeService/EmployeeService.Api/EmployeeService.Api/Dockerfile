# ---------- STAGE: build ----------
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# copy csproj(s) and restore to leverage cache
COPY ["EmployeeService.Api/EmployeeService.Api.csproj", "EmployeeService.Api/"]
# if you have shared libs, copy them too before restore (e.g. Shared/*.csproj)
# COPY ["Shared/Shared.csproj", "Shared/"]
RUN dotnet restore "EmployeeService.Api/EmployeeService.Api.csproj"

# Copy full source
COPY . .

# Build
WORKDIR /src/EmployeeService.Api
RUN dotnet build "EmployeeService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/build

# ---------- STAGE 2: Publish ----------
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
WORKDIR /src/EmployeeService.Api
RUN dotnet publish "EmployeeService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/publish /p:UseAppHost=false

# ---------- STAGE 3: Final runtime ----------
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Create non-root user
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

ENV ASPNETCORE_URLS=http://+:8080
ENV DOTNET_RUNNING_IN_CONTAINER=true
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false


# Use .NET SDK image for build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy only the .csproj from the correct nested path
COPY ["Src/Services/EmployeeService/EmployeeService.Api/EmployeeService.Api/EmployeeService.Api.csproj", "EmployeeService.Api/"]

# Restore dependencies
RUN dotnet restore "EmployeeService.Api/EmployeeService.Api.csproj"

# Copy everything in the project
COPY . .

# Build the project
WORKDIR "/src/Src/Services/EmployeeService/EmployeeService.Api/EmployeeService.Api"
RUN dotnet build "EmployeeService.Api.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "EmployeeService.Api.csproj" -c Release -o /app/publish

# Final stage - runtime image
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# 👇 Important: Make API listen on port 8080
ENV ASPNETCORE_URLS=http://+:8080


COPY --from=publish /app/publish .

EXPOSE 8080

USER appuser

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "EmployeeService.Api.dll"]