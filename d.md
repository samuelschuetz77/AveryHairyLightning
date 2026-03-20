### make new repo

go to github and make a new repo. Clone into the empty repo and run
```
dotnet new web
mkdir tests
cd tests
dotnet new xunit
```

add this dockerfile to the root directory:
```
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:80
EXPOSE 80
ENTRYPOINT ["dotnet", "harry--azure-ci-cd.dll"]
```

the last line is named what my project is named, so rename that to match yours

push!

### portal.azure.com

Log in with your student email. We're going to be using a free web app to demonstrate how to set up a simple CI/CD pipeline with azure.

### set up docker hub

hub.docker.com
create a repository
public

build in this directory: 
docker build -t < repoName > .
docker push < repoName >

This is just so you can create and push an image, verifying that it'll work

### Create a resource --> Web App

Subscription: Azure for Students
Resource group: Make a new one

publish: Container!
OS: Linux
Region: leave whatever was the default
Linux plan: Create new
Pricing plan: Free F1

To the container tab!

Image source: Other container registries
Access type: public

Registry server URL: docker.io
Image and tag: < repoName >

### set up deployment!!!

settings --> configuration --> check the box that says SCM Basic Auth Publishing Credentials

search for deploy once your web app resource is up

deployment center

source: GitHub Actions

organization: your github account
repository: the one you just made!
branch: main
Workflow option: add a workflow

Authentication type: basic authentication

Registry user name: your docker username
#### Registry password

back to hub.docker.com --> account settings --> personal access tokens --> make new one
it's got to be read, write, delete
copy new token
put that in for password
save!

You might have to re-enter it, there's some funny problems
registry docker.io (get rid of trailing /)

```
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - name: testing
        run: |
          docker run \
            --rm \
            -u $(id -u):$(id -g) \
            -e DOTNET_CLI_HOME=/tmp/dotnet-home \
            -e HOME=/tmp \
            -e NUGET_PACKAGES=/tmp/nuget-packages \
            -v .:/app \
            -w /app \
            mcr.microsoft.com/dotnet/sdk:10.0 \
              dotnet test --no-restore tests/tests.csproj
```