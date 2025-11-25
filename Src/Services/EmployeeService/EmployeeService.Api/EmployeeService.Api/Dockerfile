# ---------- STAGE 1: Build ----------
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy only the csproj and restore (cache friendly)
COPY ["Src/Services/EmployeeService/EmployeeService.Api/EmployeeService.Api/EmployeeService.Api.csproj", "EmployeeService.Api/"]
RUN dotnet restore "EmployeeService.Api/EmployeeService.Api.csproj"

# Copy full source
COPY . .
WORKDIR "/src/Src/Services/EmployeeService/EmployeeService.Api/EmployeeService.Api"
RUN dotnet build "EmployeeService.Api.csproj" -c Release -o /app/build

# ---------- STAGE 2: Publish ----------
FROM build AS publish
RUN dotnet publish "EmployeeService.Api.csproj" -c Release -o /app/publish /p:UseAppHost=false

# ---------- STAGE 3: Final Runtime ----------
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# 👇 Make ASP.NET listen correctly on Azure Container Apps
ENV ASPNETCORE_URLS=http://+:8080
ENV DOTNET_RUNNING_IN_CONTAINER=true
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

# Create non-root user (optional security best practice)
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

COPY --from=publish /app/publish .

EXPOSE 8080

USER appuser

ENTRYPOINT ["dotnet", "EmployeeService.Api.dll"]
