Bugzilla web server
===================
Docker image for bugzilla web server.

Installation
------------
Create user and group to own bugzilla files and to run docker container
```
sudo useradd -r bugzilla
```

Make **build** executable
```
sudo chmod u+x ./build
```

Execute **build**
```
sudo ./build bugzilla bugzilla
```

Create folder for bugzilla files
```
sudo mkdir /bugzilla
```
```
sudo mkdir /bugzilla/database
```
