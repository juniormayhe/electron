# Electron

## Requirements

Node installed
Electron installed

to install go to root project folder and run:
```
npm i -D electron@latest
```
This will add devDependencies { "electron": ... } in project package.json file.
Then run npm install to download the packages set in devDependencies
```
npm install
```
To list dependencies installed:
```
npm list --depth=0

master-electron@1.0.0 /home/junior/master-electron-master
├── electron@9.0.0
└── nodemon@2.0.3
```
to update dependencies
```
npm uninstall electron
npm uninstall nodemon
npm install electron@latest --save-dev
npm install nodemon@latest --save-dev
```

## Running app

app entry point is set on packages.json { "main": "start file.js" } which will be used to start the electron app.

to run the app defined in entry point:
```
npm start
```
npm start should call the script defined in package.json > scripts > start: `electron .`

to observe changes and restart electron (hot reload):
```
npm run watch
```

## Packages that make electron quit unexpectedly
The message `dyld: lazy symbol binding failed: Symbol not found` may show up when you run `npm start`.

The electron app may crash because we are trying to use an  external package that was compiled with a different version of node compared to our local node's internal v8 engine.

To fix this we could use electron-rebuild command line tool to compile the node module to our electron node version.

To rebuild the package to use our installed electron node module version:
```
npm install -g electron-rebuild
./electron-rebuild <package name>
```

bcrypt module is an example of node module that can be rebuilt using our node installation.

## Debugging
Create a .vscode\launch.json file in vscode
```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Debug Main Process",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron",
      "windows": {
        "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron.cmd"
      },
      "args" : ["."],
      "outputCapture": "std"
    }
  ]
}
```
## Frameless Window
https://stackoverflow.com/questions/35876939/frameless-window-with-controls-in-electron-windows

## Remember window positions and sizes
```
npm install --save electron-window-state
```

```javascript
const {app, BrowserWindow} = require('electron')
const windowState = require('electron-window-state')

//prevent garbage collecting
let mainWindow
function createWindow()
{
//set defaults to remember when app restart. x and y would be centered
let windowStateManager = windowState({
    defaultWidth: 1000,
    defaultHeight: 800,
  });
  
 mainWindow = new BrowserWindow({
    width: windowStateManager.width, 
    height: windowStateManager.height,
    x: windowStateManager.x,
    y: windowStateManager.y,
    webPreferences: { nodeIntegration: true },
    titleBarStyle: 'hidden', /* for mac only*/
    show: false, //avoid blank window when starting app
    backgroundColor: '#2B2E3B', // avoid blank window when starting app
    frame: false,
    minWidth: 400,
    minHeight: 400
  })
  
  mainWindow.loadFile('index.html')
  mainWindow.once('ready-to-show', mainWindow.show)
  
  windowStateManager.manage(mainWindow)
  mainWindow.on('closed',  () => {
    mainWindow = null
  })  
}

app.on('ready', ()=>{
  console.log('app is running');
  createWindow();
})
```

## Sessions

Global session for all renderers / browser windows. Default session is persisted on disk and kept when app is restarted.
A default session is a persistent partition.

```typescript
const {app, BrowserWindow, session} = require('electron')
let globalSession = session.defaultSession
```
Custom session only for current window. Partition sessions are stored in memory and gone when app is restarted.
```typescript
let customSession = session.fromPartition('my-partition')
mainWindow = new BrowserWindow({ webPreferences: {session: customSession} })
```
to make custom session be persisted in disk and available after app restart, add `persist:` to the name of the partition
```typescript
let customSession = session.fromPartition('persist:my-partition')
```
you can also set custom session directly in BrowserWindow
```typescript
mainWindow = new BrowserWindow({ webPreferences: {session: 'persist:my-partiton'} })
```

to clear all data from global sessions (cookies, local storage, index db or service workers):
```typescript
session.defaultSession.clearStorageData()
```

Session cookies without expiration set will keep the cookie until the app is finished, so the data will not be available during app restart.
```typescript
let cookie = { url: 'https://mydomain', name: 'cookie1': value: 'electron' }
session.defaultSession.cookies.set(cookie).then(()=>{ console.log('temporary cookie was set')})
```
Session cookies with expiration will keep the cookie until the time defined, and data will be available during app restart.
```typescript
let cookie = { url: 'https://mydomain', name: 'cookie1': value: 'electron', expirationDate: 1622810000 }
session.defaultSession.cookies.set(cookie).then(()=>{ console.log('cookie was set with expiration')})
```

Get all cookies
```typescript
let getCookies = () => {
  session.defaultSession.cookies.get({})
    .then(cookies => console.log(cookies))
    .error(errors=> console.log(errors))
}
```

Get cookie by name
```typescript
let getCookies = () => {
  session.defaultSession.cookies.get({name: 'cookie1'})
    .then(cookies => console.log(cookies))
    .error(errors=> console.log(errors))
}
```

Remove cookie
```typescript
session.defaultSession.cookies.remove({url: 'https://mydomain', name: 'cookie1'}).then(cookies => console.log('cookie removed'))
```

## Messaging between Main and Renderer

Inter-process communication can be done between main process (ipcMain) and renderer process (ipcRenderer). All send is async by default.

Listen to a message channel from renderer in main process
```typescript
const {ipcMain} = require('electron')

// listen to channel-name sent by renderer process
ipcMain.on('channel-name', (ipcMainEvent, dataReceived) => {
  // do something with dataReceived, a string or object. this arg is optional
  console.log(dataReceived)
  
  // reply to renderer, if needed
  ipcMainEvent.reply('reply-channel-name', stringOrObjectHere)
  // or
  ipcMainEvent.sender.send('reply-channel-name', stringOrObjectHere)
})

// send to the any render process listening channel 2
ipcMainEvent.send('channel2-name', dataToSend)

// send message to renderer from main window webcontents is also possible 
// but we must wait for page to load
mainWindow.webContents.on('did-finish-load', e=>{
  mainWindow.webContents.send('channel3-name', dataToSend)
})
```

Listen to a message channel from main in renderer process (a browser instance)
```typescript
const {ipcRenderer} = require('electron')

// send to the main process
ipcRenderer.send('channel-name', dataToSend)

// listen to channel-name sent by main process
ipcRenderer.on('reply-channel-name', (ipcRenderer, dataReceived) => {
  // do something with dataReceived, a string or object. this arg is optional
  console.log(dataReceived)
})
```

By default send messages are async, all following methods are executed with no blocking call.
But if we need to make synchronous calls, we must change the send method.

Send sync message to main from renderer
```typescript
const {ipcRenderer} = require('electron')

// send synchronous message to the main process, this will be blocked until main responds
let returnValueFromMain = ipcRenderer.sendSync('channel-name', dataToSend)
```

Listen sync message in main
```typescript
const {ipcMain} = require('electron')

// listen to channel-name sent by renderer process
ipcMain.on('channel-name', ipcMainEvent => {
  
  // do stuff here before responding to renderer
  
  ipcMainEvent.returnValue = 'A response sent from main to be waited by renderer'  
})
```

Having hundreds of functions and using channels this way can become difficult to manage, remote module could simplify but exposes main to the client which may not be secure because using messaging from remote module exposes main process to renderer (client browser). Besides this security threat, remote modules causes performance penalty in long running loop. As long as the remote module is kept in renderer process (not be garbage collected), the corresponding object in main will be kept. Remote module is disabled by default. To use it we must enable in main:

```typescript
const {ipcMain} = require('electron')

mainWindow = new BrowserWindow({
    webPreferences: { nodeIntegration: true, enableRemoteModule: true },
```
Remote module helps to access UI elements available only in main process, such as menus and dialog.
To open a dialog in main from renderer using remote module:

```typescript
const {remote} = require('electron')
const {dialog} = remote
dialog.showMessageBox({message: 'hello from renderer', buttons: ['One','Two']}).then(res=>{console.log(res})
```

An alternative to keep avoid hundreds of listeners is to use invoke and handle methods for messaging:

Send message to main from renderer
```typescript
const {ipcRenderer} = require('electron')
// send message to invoke the dialog in main
ipcRenderer.invoke('ask-fruit-channel')
  .then(answer => console.log(answer)) // when option is chosen in main, prints the choice
```

Listen to renderer message in main
```typescript
async function askFruit(){
  let fruits = ['Apple', 'Orange', 'Grape']
  let choice = await dialog.showMessageBox({message: 'Picke the fruit',buttons: fruits})
  return fruits[choice.response];
}
// listen renderer request to show a dialog and return a promise
ipcMain.handle('ask-fruit-channel', e=>{
  return askFruit()
})
```


## WSL problems
Wsl 1 does not work well with electron. The tweaks below could not make it work: 

If start does not work, review logs and try to install missing libs in Linux.
```
sudo apt-get update
sudo apt-get install libnss3-dev
sudo apt-get install -y libatk-adaptor
sudo apt-get install -y libgdk-pixbuf2.0-dev
sudo apt-get install -y libgtk-3-0
sudo apt-get install -y libgbm-dev
sudo apt-get install -y libxss1
sudo apt install -y ffmpeg
```

try to give permissions to node_modules
```
chmod -R a+rwx ./node_modules
```

Review proper permissions to chrome-sandbox folder to avoid the error:

[10038:0912/125247.035184:FATAL:setuid_sandbox_host.cc(158)] The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /home/junior/master-electron-master/node_modules/electron/dist/chrome-sandbox is owned by root and has mode 4755.

```
sudo chown root /home/junior/master-electron-master/node_modules/electron/dist/chrome-sandbox
sudo chown 4755 /home/junior/master-electron-master/node_modules/electron/dist/chrome-sandbox
```
If that does not work, maybe for versions incompatibility between node latest and older electron, we could try to uninstall electron and get a new one.
```
npm uninstall electron
npm uninstall nodemon
npm install electron@latest --save-dev
npm install nodemon@latest --save-dev
```

If that does not work try to update enable metadata if you are using WSL Ubuntu on Windows
```
vi /etc/wsl.conf
[automount]
options = "metadata"
```
Then rerun permissions
```
sudo chown root:$USER ./node_modules/electron/dist/chrome-sandbox
sudo chmod 4755 ./node_modules/electron/dist/chrome-sandbox
```
The error on libffmpeg.so in WSL ubuntu is a restriction so maybe we cannot run electron in wsl yet, since WSL needs chormium-chromium-browser, chromium snap which is not supported.

