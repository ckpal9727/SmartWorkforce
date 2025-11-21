# ---------- STAGE: build ----------
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# copy csproj(s) and restore to leverage cache
COPY ["EmployeeService.Api/EmployeeService.Api.csproj", "EmployeeService.Api/"]
# if you have shared libs, copy them too before restore (e.g. Shared/*.csproj)
# COPY ["Shared/Shared.csproj", "Shared/"]
RUN dotnet restore "EmployeeService.Api/EmployeeService.Api.csproj"

# copy everything and build
COPY . .
WORKDIR /src/EmployeeService.Api
RUN dotnet build "EmployeeService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/build --no-restore

# ---------- STAGE: publish ----------
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "EmployeeService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/publish /p:UseAppHost=false --no-build

# ---------- STAGE: final ----------
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Create a non-root user (optional but recommended)
# The official image does not create "app" user by default; create one:
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

# set app URLs & env
ENV ASPNETCORE_URLS=http://+:8080
ENV DOTNET_RUNNING_IN_CONTAINER=true
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

# copy published output
COPY --from=publish /app/publish .

# expose the port your app listens to
EXPOSE 8080
EXPOSE 8081
# Set non-root user
USER appuser

# Optional healthcheck - used by Docker / orchestrator to know container readiness
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "EmployeeService.Api.dll"]
