---
title: "Make Service in Linux"
series: ["Make Service in Linux"]
date: 2022-08-07T00:00:00+00:00
draft: false
author: "Masum Osman Khan"
categories:
  - blog
tags:
  - Linux
  - systemd
  - service
thumbnail: linux
---
### Intro:
Generally service files stores in 

```go
/etc/systemd/system
```

location. So to create new service, we need to make a new file by `vim or nano`in this directory. The filename is the service name you call after. The file extension should be .service

For example, we are creating “***prod-image-server.service***” service. 

We made a file with service extension. Now the bellow codes should be in the file.

```go
[Service]                                                                       
WorkingDirectory=/home/Go/image-service                                         
ExecStart=/home/Go/image-service                                         
                                                                                 
ExecReload=/bin/kill -HUP $MAINPID                                              
LimitNOFILE=65536                                                               
Restart=always                                                                  
RestartSec=5                                                                    
                                                                                
                                                                                 
[Install]                                                                       
WantedBy=multi-user.target
```

the WorkingDirectory is the directory locating your file. ExecStart is the command to run. I am using go lang here. So I don’t need any other command to run except the ./ shell command. 

Then enable and start your service by like those commands:

```shell
sudo systemctl enable prod-image-server.service 
sudo systemctl start prod-image-server.service
```