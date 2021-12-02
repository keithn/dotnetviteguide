# A Guide to getting started with .NET 6, Vite, and Vue 3

This shows how to setup your environment for development.  While there are some templates for .NET 6 and Vue, it's often not really what you want.   This guide will show you how to hack your Project to get it to run your frontend how you want.   You can use the information here and make variations for whatever tooling you like to use

# Goal For this Guide

- Setup .NET 6 Project with a Vue 3 Spa front end
- Use Yarn as my package manager
- Use Vite to generate the Vue project and run the development server
- Customize the .NET publish to publish your Vite project

# Tools I'm using

While you can all kinds of different editors and tools and operating systems, for reference here is my setup

- Jetbrains Rider
- Windows 11




## Let's go!

### Create .NET Project

There are a number of templates that you could use, but for this guide use the react template

```dotnet new react```

in Rider

![image](https://user-images.githubusercontent.com/86080/144359457-c1485bff-7d36-4a57-ad55-b346ebca480f.png)

This project will give you a good starting point and will setup the SPA Proxy

### Small overview of SPA Proxy

In previous .NET versions ( before .NET 6 ) dotnet projects setup a proxy from the .NET webserver to your frontened project (whether it be Vue, React, Angular or other ).  I never used this as it was silliness, I always manually setup the proxy the other way round, so proxy from my Vue app to the .NET API backend.  Thankfully, in .NET 6 this is the approach they now also use.

So now, Spa Proxy isn't so much of a proxy, it is a SpaLauncher, the main guts you can find here:

[https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/Spa/SpaProxy/src/SpaProxyLaunchManager.cs]

It is pretty simple and will help you understnad what it is trying to do.

The general process is that it checks to see if your front is already running, and if it isn't, it trys to launch it.

More annoyingly, it force kills it when your program ends.  It provides no option not to do this, and most often this is not what you want.  If you want to change your .NET 6 project, such that you need to stop it and restart it, quite often you don't want your frontend to stop and start, so if you prefer, rather than letting your dev tools launch your front end, just manually launch it yourself in a terminal, ie

```yarn dev```

However, we will setup the project to correctly run the SpaProxy (ie, launcher) properly

### Configure

Open your project file, and it will have a section something like 

```XML
    <PropertyGroup>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>
        <TypeScriptToolsVersion>Latest</TypeScriptToolsVersion>
        <IsPackable>false</IsPackable>
        <SpaRoot>ClientApp\</SpaRoot>
        <DefaultItemExcludes>$(DefaultItemExcludes);$(SpaRoot)node_modules\**</DefaultItemExcludes>
        <SpaProxyServerUrl>https://localhost:3399</SpaProxyServerUrl>
        <SpaProxyLaunchCommand>yarn dev</SpaProxyLaunchCommand>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>
```

----- TO BE CONTINUED :)



### Certificate for your Vue Project

### Windows 11 / Windows Terminal 

By default ( at time of writing ), terminal does not automatically close.  However in the terminal settings you can change it so it closes on exit (no matter the exit reason)

![image](https://user-images.githubusercontent.com/86080/144360255-2ce34b66-773f-4396-aa08-ff347f5e42a7.png)

Otherwise it will leave a lot of stray terminals around.



### Proxy from Vue to .NET Api

### Publish
