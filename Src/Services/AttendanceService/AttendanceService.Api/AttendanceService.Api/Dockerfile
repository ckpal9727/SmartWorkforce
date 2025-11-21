# ---------- STAGE: build ----------
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Copy only csproj first to improve docker layer caching
COPY ["AttendanceService.Api/AttendanceService.Api.csproj", "AttendanceService.Api/"]
# If AttendanceService depends on Shared or Domain/Application layers, add them also:
# COPY ["Shared/Shared.csproj", "Shared/"]
# COPY ["AttendanceService.Domain/AttendanceService.Domain.csproj", "AttendanceService.Domain/"]
# COPY ["AttendanceService.Application/AttendanceService.Application.csproj", "AttendanceService.Application/"]

RUN dotnet restore "AttendanceService.Api/AttendanceService.Api.csproj"

# Copy full source
COPY . .

WORKDIR /src/AttendanceService.Api
RUN dotnet build "AttendanceService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/build --no-restore

# ---------- STAGE: publish ----------
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "AttendanceService.Api.csproj" -c ${BUILD_CONFIGURATION} -o /app/publish /p:UseAppHost=false --no-build

# ---------- STAGE: final ----------
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Create non-root user (security best practice)
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

# Environment variables
ENV ASPNETCORE_URLS=http://+:8080
ENV DOTNET_RUNNING_IN_CONTAINER=true
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

# Copy published output
COPY --from=publish /app/publish .

# Expose ports
EXPOSE 8080
EXPOSE 8081

# Switch to non-root user
USER appuser

# Healthcheck to improve container orchestration
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "AttendanceService.Api.dll"]
