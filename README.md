# Azure Function `PUT` issue with 3.0.15417

There is an issue with the Azure Function Runtime Version 3.0.15417. This repo reproduces the defect. It was created using the Azure tools in Visual Studio Code.

There is a regression preventing empty http route prefix for `HTTP PUT` methods. This worked previously in the previous version used on our production Azure Function App Service instances (Linux, 3.0.15405).

This repo reproduces the defect.

## Reproduction Steps

There are two `Dockerfile`s:

* `Dockerfile.15405` targetting `mcr.microsoft.com/azure-functions/dotnet:3.0.15405-appservice`
* `Dockerfile.15417` targetting `mcr.microsoft.com/azure-functions/dotnet:3.0.15417-appservice`

### Working version

```bash
docker build -t azure-function-3-0-15405 -f Dockerfile.15405 .
container=$(docker run -d -p 8080:80 azure-function-3-0-15405)
curl --location --request GET 'http://localhost:8080/HttpTriggerCSharp1' -i
curl --location --request PUT 'http://localhost:8080/HttpTriggerCSharp1' -i --data-raw ''
docker container stop $container
docker container rm $container
```

Both `HTTP GET` and `HTTP PUT` work as expected with empty `routePrefix`, returning `HTTP 200`.

### Failing version

```bash
docker build -t azure-function-3-0-15417 -f Dockerfile.15417 .
container=$(docker run -d -p 8080:80 azure-function-3-0-15417)
curl --location --request GET 'http://localhost:8080/HttpTriggerCSharp1' -i
curl --location --request PUT 'http://localhost:8080/HttpTriggerCSharp1' -i --data-raw ''
docker container stop $container
docker container rm $container
```

Only `HTTP GET` works with empty `routePrefix` in `15417`. `HTTP PUT` always returns `HTTP 401`.

### Workaround

If you edit `host.json` and set a non-empty `routePrefix` then `15417` will work. You will need to change the request URL to match the prefix you used `http://localhost:8080/test/HttpTriggerCSharp1`
