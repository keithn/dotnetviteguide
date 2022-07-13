# A guide to getting started with ASP.NET Core 6, Vite, and Vue 3

This guide shows how to setup your environment for development.  While there are some templates for [ASP.NET Core 6](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-6.0) and [Vue](https://vuejs.org/), it's often not really what you want.  This guide will show you how to hack your project to get it to run your frontend how you want.  You can use the information here and make variations for whatever tooling you like to use.

# Goal for this guide

- Setup a ASP.NET Core 6 project with a Vue 3 SPA front end
- Use [Yarn](https://yarnpkg.com/) as my package manager
- Use [Vite](https://vitejs.dev/) to generate the Vue project and run the development server
- Customize the .NET publish to publish your Vite project

# Tools I'm using

While you can all kinds of different editors and tools and operating systems, for reference here is my setup

- JetBrains Rider
- Windows 11

# Let's go!

## Create a new ASP.NET Core project

There are a number of templates that you could use, but for this guide use the React template.
```
dotnet new react
```
in Rider

![image](https://user-images.githubusercontent.com/86080/144359457-c1485bff-7d36-4a57-ad55-b346ebca480f.png)

This project will give you a good starting point and will setup the SPA proxy.

## Small overview of the ASP.NET Core SPA proxy

In previous .NET versions (before .NET 6) dotnet projects setup a development proxy from the .NET [Kestrel](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0) web server to your frontened project (whether it be Vue, React, Angular or other).  I never used this as it was silliness, I always manually setup the proxy the other way round, so proxy from my Vue app to the .NET API backend.  Thankfully, in .NET 6 this is the approach they now also use.

So now, the SPA development proxy isn't so much of a proxy, it is more of a SPA launcher, the main guts you can find here:

https://github.com/dotnet/aspnetcore/blob/main/src/Middleware/Spa/SpaProxy/src/SpaProxyLaunchManager.cs

It is pretty simple and will help you understand what it is trying to do.

The general process is that it checks to see if your front is already running, and if it isn't, it attempts to launch it.

More annoyingly, it force kills it when your program ends.  It provides no option not to do this, and most often this is not what you want.  If you want to change your .NET project, such that you need to stop it and restart it, quite often you don't want your frontend to stop and start, so if you prefer, rather than letting your dev tools launch your front end, just manually launch it yourself in a terminal, i.e.

If you use yarn:
```
yarn dev
```
If you use npm:
```
npm run dev
```

However, we will setup the project to correctly run the SPA development proxy (i.e., launcher) properly.

## Remove the ClientApp folder

Under your newly created .NET project, it will have a ClientApp folder. This has all the React code in it. Delete it.

## Create your Vue 3 project with Vite

In the main project folder (where the csproj file is), run the following command to create a Vite project (or whatever tooling you want to use, as long as it makes a subfolder off the main project folder)

If you use yarn:
```
yarn create vite
```

Or if you use npm:
```
npm create vite@latest
```

It will ask you for the project name, this will be the name of the folder it creates, choose something you like, or you can call it ClientApp, which means you need to configure a bit less later.

Select the options you want. 

In my case `vue -> vue-ts`.

change into the directory, and type `yarn` to install dependencies or `npm install` if you use npm (or let your dev tools do it if they know how)

You can treat this project like a normal Vue 3 / Vite project and set it up however you like.  We will have to configure some options for Vite which we will discuss a little later in the guide.


## Configure the .NET project

Now that we have the Vite project we need to make some changes to the .NET project.

Choose a port you want to run your Vite Vue project on (I tend to choose a unique port per project), in this case port 3399. The default port used by Vite is port 3000.

```xml
<SpaProxyServerUrl>https://localhost:3399</SpaProxyServerUrl>
```
 
set the launch command for dev mode, in this case `yarn dev` (or `npm run dev` if you use npm)
```xml
<SpaProxyLaunchCommand>yarn dev</SpaProxyLaunchCommand>
```
 
If you have a different name for your project folder rather than "ClientApp" specify it in the SpaRoot
```xml
<SpaRoot>ClientApp\</SpaRoot>
```
 
Which should result in a project file something like below

```xml
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

## Certificate for your Vue project

By default we generally want to use https for everything, so for your Vite project we can use "makecert" https://github.com/liuweiGL/vite-plugin-mkcert

If you use yarn:
```
yarn add vite-plugin-mkcert -D
```
Or if you use npm:
```
npm install --save-dev vite-plugin-mkcert
```

and configure it like so

```ts
import mkcert from 'vite-plugin-mkcert'

export default defineConfig({
  plugins: [vue(), mkcert()],
  server: {
    https: true
    //  the rest of the config...
  }
});
```

## Proxy from Vite to ASP.NET Core API

Now lets setup the Vite proxy. Use the same port you chose back when editing the project, in this case port 3399.

`strictPort` means it won't try another port if it finds that the port is already in use.  

I suggest nesting all API calls under the route `/api` which makes it easy to work out what things to proxy. However, if you want to, you can rewrite the routing to the backend.  You mostly don't want to do this, otherwise in production you will end up with different routes and the two projects won't marry together without some other kind of config to set the correct routes.  However, I include the rewrite in the example below for reference (currently it doesn't change the route at all), you can omit the rewrite rule altogether if you aren't doing anything fancy.

In the proxy specify the `target` as whatever the .NET project has chosen for running the API on.

`secure: false` just means the proxy won't check certificates, as we don't really care for our local setup (though if you've installed the dotnet developer certificate, it should be ok).


vite.config.ts (or .js if you go with JavaScript)

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import mkcert from 'vite-plugin-mkcert'

export default defineConfig({
  plugins: [vue(), mkcert()],
  server: {
    port: 3399,
    https: true,
    strictPort : true,
    proxy: {
      '/api' : {
        target: 'https://localhost:7153',
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/api/, '/api')
      }
    } 
  }
})
```

Read more on [configuring Vite](https://vitejs.dev/config/).

### ASP.NET Core minimal API and fetching from Vue

In your `Program.cs` file add a route that returns some test data

```csharp
app.MapGet("api/test", () => new { Test = "hello" });
```

and in Vue, in your `App.vue` file, you can use the following template to test whether you can fetch the data :-

```vue
<script setup lang="ts">
import { onMounted, ref } from "vue";

interface TestData {
  test: string;
}

const result = ref<TestData>({test: ""})
onMounted(async () => {
  result.value = await (await fetch("/api/test")).json();
});

</script>

<template>
  <div>Result: {{ result.test }}</div>
</template>
```


### Windows 11 / Windows Terminal 

By default (at time of writing), the Windows Terminal does not automatically close.  However in the terminal settings you can change it so it closes on exit (no matter the exit reason)

![image](https://user-images.githubusercontent.com/86080/144360255-2ce34b66-773f-4396-aa08-ff347f5e42a7.png)

Otherwise it will leave a lot of stray terminals around.


## Good to Develop!

At this stage everything should work in your development environment and you should be able to write code and have it proxy to your ASP.NET Core backend.

If you have any issues, or something is unclear or notice any problems with getting to this point, raise an issue on this repo and I will try to improve the guide!

## Publish

In the `.csproj` file change:
```xml
<DistFiles Include="$(SpaRoot)build\**" />
```
to:
```xml
<DistFiles Include="$(SpaRoot)dist\**" />
```

TO BE CONTINUED....
