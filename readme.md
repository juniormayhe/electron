# Electron

## Requirements

Node installed
Electron installed

to install go to root project folder and run:
```
npm i -D electron@latest
```
This will add devDependencies { "electron": ... } in project packages.json file.
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
```
