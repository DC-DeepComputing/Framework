# Applications apt-get install notes
Developers can add a new entry pointing to OS source files and then use `apt-get` to install some common applications or libraries.

## Command Example

```bash
vim /etc/apt/sources.list.d/local.list
//Add the following line to the file:
deb [trusted=yes] http://47.89.252.251:8000/ubuntu24.04 ./

```
## Supported Applications and Tools
- Firefox
```bash
sudo apt-get update
sudo apt-get install firefox
```
- Chromium
```bash
sudo apt-get update
sudo apt-get install libdouble-conversion3 libjpeg62-turbo libsnappy1v5 libwoff1 libxml2 x11-utils xdg-utils libjpeg62-turbo-dev -y
sudo apt-get install chromium 
```
- LibreOffice
```bash
sudo apt update
sudo apt-get install libreoffice
```
