# How-To-Deploy-ASP-Dotnet-6-Using-Ngnix
In this small article I will explain how can you deploy yor ASP .NET we applicaiton in Ubontu server using Nginx

### How The Reverise Proxy Work?

In very simple word, Reverise Proxy means the clint doesn't know which server are connecting to, see video [Proxy vs. Reverse Proxy](https://www.youtube.com/watch?v=ozhe__GdWC8&t=2s&ab_channel=HusseinNasser)
![Screenshot 2022-09-03 222221](https://user-images.githubusercontent.com/76538765/188285148-d15112e0-7ca6-4bd3-ad1d-06650110a8dd.jpg)

### Setup
What we will do?
- we had Ubontu server in Digitalocean
- we had a ASP .NET wep app
---

> Apps deployed in a reverse proxy configuration allow the proxy to handle connection security (HTTPS). If the proxy also handles HTTPS redirection, there's no need to use HTTPS Redirection Middleware.
see [Require HTTPS](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#require-https)

---
### Code
In Program.cs file, we should remove Force Redirect command, because we want to run our applicaiton without https, In our Nginx server we will configure the ssl cert. from there, so we don't need to force the appliction from code.
```diff
- app.UseHttpsRedirection();
+ //app.UseHttpsRedirection();
```

After that push it to Github
And the go throw your Ubontu server using ssh
1. `cd /var/www`
2. clone your repository `sudo git clone {repo_url}`
3. before be build, we need to test our code `sudo dotnet run`, If the project work without error we can know exit from the app by `cntl + c`
4. build the app `sudo dotnet build`
5. publish the app `sudo dotnet publish`
you will see something like this
```
Microsoft (R) Build Engine version 17.0.0 for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  All projects are up-to-date for restore.
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/{Application_Name}.dll
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/publish/
```

Be careful when you do that, you need to memorize the last two lines
```
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/{Application_Name}.dll
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/publish/
```


After we publish our project, we need to create a `service` to run,stop and monitor that appliction
We can do that in few Not simple steps

1. To create a new service `sudo nano /etc/systemd/system/{Name_Your_Service}.service`
 
2- Add this block
```
[Unit]
Description=Server - Side
[Service]
WorkingDirectory= /var/www/{Repository_Name}/bin/Debug/net6.0/publish
ExecStart=/usr/bin/dotnet /var/www/{Repository_Name}/bin/Debug/net6.0/publish/{Application_Name}.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier= {Any_Name}
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target

```
If you want to change the port you just need to add this line <br/>
`Environment=ASPNETCORE_URLS=http://127.0.0.1:5001` <br/>
But make sure that you also change it in the Nginx configuration

> If that confuses you, Please contact me to set a seesion with you [free]

And now we can start our new service <br/>
`sudo systemctl start {Your_Service_Name}` <br/>  
and to check if its work or not `sudo systemctl status {Your_Service_Name}`  <br/>

#### Nginx
After navigate to `/etc/nginx/site-avalive`, add a new one and put the following block of code

```
server {
 listen 80;
        listen [::]:80;

        server_name test.mosaibah.com *.test.mosaibah.com;

        location /{
                proxy_pass http://localhost:5001;
        }
}
```

#### Adding the certbot
Just in two steps:
1. `sudo apt install python3-certbot-nginx`
2. `sudo certbot --nginx -d {domain}.com`,  and that will change the configuation file in Nginx





