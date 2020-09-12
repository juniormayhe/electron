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

app entry point is set on packages.json { "main": "start file.js" } which will be used to start the electron app.

to run the app defined in entry point:
```
npm start
```

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
