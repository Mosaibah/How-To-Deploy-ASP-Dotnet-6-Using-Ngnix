# How to Deploy ASP.NET 6 Using Nginx

In this brief article, I will explain how to deploy your ASP.NET web application on an Ubuntu server using Nginx.

### Setup
We will:
- Use an Ubuntu server on DigitalOcean
- Deploy an ASP.NET web application
---

> When apps are deployed in a reverse proxy configuration, the proxy handles connection security (HTTPS). If the proxy also manages HTTPS redirection, there's no need to use HTTPS Redirection Middleware.
for more information see [Require HTTPS](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#require-https)

---
### Code
In the Program.cs file, remove the Force Redirect command. We want to run our application without HTTPS because we'll configure the SSL certificate in our Nginx server. Therefore, we don't need to force HTTPS redirection in the application code.
```diff
- app.UseHttpsRedirection();
+ //app.UseHttpsRedirection();
```

After making this change, push your code to GitHub.
Next, access your Ubuntu server via SSH and follow these steps:
1. Navigate to the web directory `cd /var/www`
2. Clone your repository `sudo git clone {repo_url}`
3. Before building, test your code: `sudo dotnet run`, If the project runs without errors, exit the app using `cntl + c`
4. Build the app `sudo dotnet build`
5. Publish the app `sudo dotnet publish`
You should see output similar to this:
```
Microsoft (R) Build Engine version 17.0.0 for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  All projects are up-to-date for restore.
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/{Application_Name}.dll
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/publish/
```

Take note of the last two lines, as you'll need this information later:
```
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/{Application_Name}.dll
  {Application_Name} -> /var/www/{Repository_Name}/bin/Debug/net6.0/publish/
```


After publishing your project, create a `service` to run, stop, and monitor your application. Follow these steps:



1. Create a new service file `sudo nano /etc/systemd/system/{Name_Your_Service}.service`
 
2- Add this configuration block
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
To change the port, add this line: <br/>
If want to run another application in the same server, you need to create a new server, but note that in the configuration of app you also need to change the port that applicatio will run on it <br/>

`Environment=ASPNETCORE_URLS=http://127.0.0.1:5001` <br/>
Remember to also change the port in the Nginx configuration if you do this.
> If you're confused, please contact me to schedule a free session. [free]

Now, start your new service: <br/>
`sudo systemctl start {Your_Service_Name}` <br/>  
To check if it's working: `sudo systemctl status {Your_Service_Name}`  <br/>

#### Nginx Configuration
Navigate to `/etc/nginx/site-avalive`, add a new file, and insert the following code block:

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

#### Adding SSL with Certbot
Follow these two steps:
1. Install Certbot: `sudo apt install python3-certbot-nginx`
2. Obtain and install a certificate:  `sudo certbot --nginx -d {domain}.com`
This will automatically update your Nginx configuration file to use HTTPS.


#### How Does a Reverse Proxy Work?

In simple terms, a reverse proxy acts as an intermediary between clients and servers. The client doesn't know which server it's connecting to. For a visual explanation, watch this video: [Proxy vs. Reverse Proxy](https://www.youtube.com/watch?v=ozhe__GdWC8&t=2s&ab_channel=HusseinNasser)
![Screenshot 2022-09-03 222221](https://user-images.githubusercontent.com/76538765/188285148-d15112e0-7ca6-4bd3-ad1d-06650110a8dd.jpg)




