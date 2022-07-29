# HTTPS on Containers

Why did I create this? Well, this is a place where I can borrow stuff that are common in a variety of projects I worked and work on - both as an employee and my side projects -. I do hope this may help you as well.

It is divided in two separated ways of doing it:
1. Using localhost self-signed certificate from Microsoft;
2. Specific certificates (CA and XXX) to be used in each of your applications under the same solution (or domain/"umbrella").

## Notes
> 1. All commands below were written for PowerShell on Windows. You may have to adapt them for other OSs and CLIs.
> 1. ?!

## Using an HTTPS Development Certificate

1. You need to have a development certificate installed. You verify its existence by running the following:
   ```shell
   dotnet dev-certs https --check
   ```

   If you got:

   > A valid certificate was found: SOME_HEXADECIMAL_FINGERPRINT - CN=localhost - Valid from YYYY-MM-DD HH:MM:SSZ to YYYY-MM-DD HH:MM:SSZ - IsHttpsDevelopmentCertificate: true - IsExportable: true
   > Run the command with both --check and --trust options to ensure that the certificate is not only valid but also trusted.

   :white_check_mark: Then you're good to go. Move to next step.

   :x: Otherwise, you should get something like this:

   > No valid certificate found.

   So, you need to create a new certificate and trust it. Do it by running:

   ```shell
   dotnet dev-certs https --trust
   ```

1. Add Docker support for each project by creating a `Dockerfile`. See an example: [here](./src/HttpsOnContainers.WebApiFirst/Dockerfile).

1. Build your image<a name="step-build-image"></a>:
   > :info: Depending on how your `Dockerfile` was created, you may have to adapt the directory to run this command.

   If you created using `Docker Support` option in Visual Studio, you should run it from your solution folder:

   ```shell
   docker build -f ./src/HttpsOnContainers.WebApiFirst/Dockerfile -t httpsoncontainers-webapifirst .
   ```

   In other cases, usually you don't need to specify the `Dockerfile` and should run this command from the same folder where your `Dockerfile` is:

   ```shell
   docker build -t httpsoncontainers-webapifirst .
   ```

1. THIS IS TEMP!!! Run your image as a container:
   ```shell
   docker run -p 6220:80 httpsoncontainers-webapifirst
   ```

   `-p` indicates the port mapping `EXTERNAL:INTERNAL` where `EXTERNAL` is the port exposed to external access and `INTERNAL` is the internal port in the container.

1. Export a new HTTPS certificate for your project:
   > The PFX file name should match exactly your assembly name, or DLL file name.

   ```shell
   dotnet dev-certs https -ep $env:USERPROFILE/.aspnet/https/HttpsOnContainers.WebApiFirst.pfx -p pa55w0rd!
   dotnet dev-certs https -ep $env:APPDATA/ASP.NET/Https/HttpsOnContainers.WebApiFirst.pfx -p pa55w0rd!
   ```

1. Go to your project directory and run the following command:
   > User-secrets are meant for **DEVELOPMENT** environment.

   ```shell
   dotnet user-secrets set "Kestrel:Certificates:Development:Password" "pa55w0rd!"
   ```

   > The `value` (last argument) in the command is the password you defined in the previous step (flag `-p`) when exporting the certificate

1. Now, rebuild your image (see [build your image](#step-build-image) step).

1. Run your image, now including some additional arguments:
   ```shell
   docker run -p 6220:80 -p 8220:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8220 -e ASPNETCORE_ENVIRONMENT=Development -v $env:APPDATA/Microsoft/UserSecrets/:/root/.microsoft/usersecrets -v $env:APPDATA/ASP.NET/Https:/root/.aspnet/https httpsoncontainers-webapifirst
   ```

1. To make your life easier and type a shorter command, you can use Docker Compose. You have to, basically, transform the arguments from the command immediately above into Docker Compose structure and then run:
   ```shell
   docker compose up -d
   ```
