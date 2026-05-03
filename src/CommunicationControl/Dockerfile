FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy project files first to leverage Docker layer caching for restore.
COPY DevOpsProject/DevOpsProject.CommunicationControl.API.csproj DevOpsProject/
COPY DevOpsProject.CommunicationControl.Logic/DevOpsProject.CommunicationControl.Logic.csproj DevOpsProject.CommunicationControl.Logic/
COPY DevOpsProject.Shared/DevOpsProject.Shared.csproj DevOpsProject.Shared/

RUN dotnet restore DevOpsProject/DevOpsProject.CommunicationControl.API.csproj

COPY . .
RUN dotnet publish DevOpsProject/DevOpsProject.CommunicationControl.API.csproj -c Release --no-restore -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app

COPY --from=build /app/publish .

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

ENTRYPOINT ["dotnet", "DevOpsProject.CommunicationControl.API.dll"]