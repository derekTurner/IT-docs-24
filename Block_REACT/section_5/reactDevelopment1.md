## Docker/Vite/ Client Side App

[React](https://reactjs.org) is a javascript library for building user interfaces.

It is isomorphic, which means it may run on the server or on the client.  In this section a client app examples are developed in a container on a local machine using a development server in a node environment.

Since React is a javascript library it can be run in a simple way as we have done so far using script files.


```javascript
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
      <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
      <script src="https://unpkg.com/babel-standalone@6.26.0/babel.js"></script>
```

However, this approach is inefficient.  

The normal approach is to develop client sideReact Apps within a node environment and then build them to produce a final codebase which stands alone and can be run from a simple HTML server with no reference to node.


## Node environment on PC

Javascript is a scripting language, which means that it runs inside an application and does not compile down to stand-alone executable code.  Normally that environment is a browser, but browsers are sandboxed and constrain javascript is allowed to do.

If you are working on a development machine you can use a javascript run time environment to develop code.

If you wish javascript to run on a server, this can be done if a javascript run time is available.

The most popular run time is nodejs.  This is built upon Chrome's V8 javaScript engine.  There is usually a current version (18) and a Long Term Support version (16), the latter is recommended for most users.

![node versions](images/nodeVersions.png)


So I will follow through with node version 18 in these notes.

As well as node, there are now two alternative javascript run times, [deno](https://deno.land/) and [bun](https://bun.sh/).

Deno has been developed to overcome some of the drawbacks of node, but it is not fully backward compatible with node.

Bun uses the webkit javascript core engine, which is claimed to run faster than V8, it also claims to be backwards compatible with node.  However this is still only at version 0.1.8.

Both of these alternative run times are worthy of investigation, but for the present purpose I will work with the stable node version 18.

Node ia also associated with the package manager [npm](https://www.npmjs.com/) this is a repository which holds thousands of node modules which can be drawn into javascript applications.

Node is good, but it relies on installing these small node modules.  If you are careful you should make a distinction between modules which are installed globally, and therefore become part of your base node environment and those which are installed within the node-modules folder of your application.

Applications may be dependant on different versions of these node modules, so it is essential to be careful with version installation. Node dependencies are an important consideration in getting an application to run. This takes care to manage on a personal machine, but on a multi-user machine could become impracticable.

If you are using gitHub, there is no need to store node modules as part of your repository code, so a .gitnore file is often used to prevent this.

Because of dependancies you may run into problems if you try to develop a number of applications in the same node environment.

You could use a number of virtual linux machines to hold separate node environments for different applications.  This would work, but the codebase which is being duplicated for each machine is large.

I prefer to us lightweight linux based Docker containers to hold isolated node environments.

>If after that warning you still want to run node locally on windows you can install node locally but I generally prefer not to.  I suggest you prefer the docker approach as below.



## VSC/Docker development

Since you have loaded the remote containers plug-in to visual studio code, you can easily open up code which sits on your local machine into a container.

The code can run on a server in the container  while you develop it.  When you are ready the container can be closed and the developed code is once again available in the local folder.

If this local folder is a gitHub repository, this can be kept up to date by committing and synchronising changes as you work.

Docker has a number of modes of use, this is the simplest.  Docker can also set up a development environment which links to github, but does not use the local file structure.

Finally Docker can be used to set up containers with fuly developed code ready for deployment.


## Using React/Vite in a Development environment

This is an adaptation of the tutorial [Creating a react app with Vite]](https://blog.openreplay.com/how-to-build-your-react-app-using-vite/) to a docker development system and and the typescript language.  Firstly an environment will be set up and then vite will be used in this environment to run a simple typescript application.


Before starting you will need docker desktop installed and running.  

The about tab shows version details.

![about docker](images/about.png)

Then you will need VScode with the devContainers extension installed.

![devContainers](images/devContainers.png)

This will allow any local folder to be run in a docker development container and is an easy way to run with a development environment.

Using github desktop, create a new repository named reactTS23.  This will be your initial working space for react code development.

> File|New Reppository or CTRL + N

![now repository](images/newrepo.png)

Publish the repository.

![publish](images/Publish.png)

Open this in visual studio code and create a blank file in the folder called dev.md.  You can use this file subsequently to keep notes on your  code development as you go.

![notes](images/notes.png)

In VScode 

>CRTL + SHIFT + P

to show a list of commands and select open folder in container.

![dev menu](images/openFolder.png)

Click on Open Folder in Container. This will then open a browser dialog to choose the devContainer folder and open.

![choose folder](images/selectFolder.png)

Make sure you select the folder itself and not a parent gitHub folder or a file within the folder.


On the first time of opening a prompt appears asking what type of container is needed.  Show all definitions and then choose 'Node & Typescript'

![node and typescript](images/nodeTypescript.png)

Then you are asked to choose a node version, I have accepted the default version 20-bullseye.

![node 20](images/bullseye.png)


Then you are asked what additional features you need from a large checklist.  I selected none and pressed ok.

![additional features](images/addFeatures.png)

The system then takes time to create the container image.  

![reading container](images/showlog.png)

Click on show log to view progress, be patient.



![container running](images/showlog.png)

The details list in the terminal.

![terminal](images/terminal.png)


When this is complete docker desktop shows that the container is running.

![container](images/containerRunning.png)

The file structure which has been created in the container contains .devcontainer which contains devcontainer.json showing that this is a node and typescript environment.


``` json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/typescript-node
{
	"name": "Node.js & TypeScript",
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	"image": "mcr.microsoft.com/devcontainers/typescript-node:1-20-bullseye"

	// Features to add to the dev container. More info: https://containers.dev/features.
	// "features": {},

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Use 'postCreateCommand' to run commands after the container is created.
	// "postCreateCommand": "yarn install",

	// Configure tool-specific properties.
	// "customizations": {},

	// Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
	// "remoteUser": "root"
}
```
Open a terminal from the VSC menu.  The prompt should appear as

```code
node ➜ /workspaces/reactTS23 (main) $
```

The node version can be checked by 

>node -v

```code
v20.3.1
```

The typescript version is checked by:

>tsc -v

```code
Version 5.1.6
```

Install vite with

>npm install vite

This led to a comment inviting an update to npm.

```
added 8 packages in 12s

3 packages are looking for funding
  run `npm fund` for details
npm notice 
npm notice New major version of npm available! 9.6.7 -> 10.1.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v10.1.0
npm notice Run npm install -g npm@10.1.0 to update!
npm notice 
```

Don't update or you will get an unsupported engine message.



Once the loading process has completed a package.json file is crated in the reactTS23 folder, this displays the dependancy for Vite.

```json
{
  "dependencies": {
    "vite": "^4.4.4"
  }
}
```

Now initialise vite

> npm init vite

```code
Need to install the following packages:
  create-vite@4.4.1
Ok to proceed? (y) 
```
> y

```code
? Project name: › react23
```

Select a framework

```code
Select a framework: › - Use arrow-keys. Return to submit.
    Vanilla
    Vue
❯   React
    Preact
    Lit
    Svelte
    Solid
    Qwik
    Others

```

Choose typescript

```code
? Select a variant: › - Use arrow-keys. Return to submit.
❯   TypeScript
    TypeScript + SWC
    JavaScript
    JavaScript + SWC
```
Project is scaffolded

```code
Scaffolding project in /workspaces/reactTS23/react23...

Done. Now run:

  cd react23
  npm install
  npm run dev
```

The folder react23 has been set up with the basic template code in the src folder.  Any necessary assets are in the public folder.

![starter structure](images/starterStructure.png)

> cd react23



Install the dependancies listed in src/package.json

**src/package.json**
```json
{
  "name": "react23",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "@vitejs/plugin-react": "^4.0.3",
    "eslint": "^8.45.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.3",
    "typescript": "^5.0.2",
    "vite": "^4.4.5"
  }
}
```

> npm install

Run the src code on the Vite development server.



> npm run dev

```code
  VITE v4.4.9  ready in 5088 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h to show help

```

The output is available to the browser on port 5173.

![vite and react](images/viteReact.png)

> CTRL + C to stop the server

While the server is running any changes will update immediately.

The index.html file has a div id = root which will be written into by react.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

The starting react app code is in main.tsx

```javascript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

This imports React and ReactDom and then imports from App.tsx.

The <App /> component which is imported contains the operational code.

**App.tsx**
```javascript
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((count) => count + 1)}>
          count is {count}
        </button>
        <p>
          Edit <code>src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="read-the-docs">
        Click on the Vite and React logos to learn more
      </p>
    </>
  )
}

export default App
```

App.tsx includes a function based component App(){}  which uses the useState hook to maintain the state of a count which is incremented when a button is clicked.

Once code has been completed it can be built.  This polls all the javascript into a small number of files which can be run on a server which does not have a node environment.

> CTRL + C

> npm run build


```code
> react23@0.0.0 build
> tsc && vite build

vite v4.4.9 building for production...
✓ 34 modules transformed.
dist/index.html                   0.46 kB │ gzip:  0.30 kB
dist/assets/react-35ef61ed.svg    4.13 kB │ gzip:  2.14 kB
dist/assets/index-d526a0c5.css    1.42 kB │ gzip:  0.74 kB
dist/assets/index-c7e05d32.js   143.41 kB │ gzip: 46.10 kB
✓ built in 1.17s
```

Refresh the Explorer view to see that a new dist folder has been created.

![dist](images/dist.png)

This folder can be copied to a local folder and run on a server such as VSC live server of uploaded to a flat file server online.

Commit code via VScode

![commit](images/commit.png)

Sync changes to github

![sync](images/sync.png)

## Challenge 1

Edit the code to implement the three containers which display 5 different messages each.

## Challenge 2

Implement a game of tic tac toe referring to the online tutorial.

[Tic Tac Toe](https://react.dev/learn/tutorial-tic-tac-toe)

Watch out for minor differences between typescript and javascript.

The instructions also note that you could use the codesandbox development environment.  If you follow the fork in the tutorial this will take you to a framework vased on create-react-app and not vite.  This works but is not as up to date as an approach.