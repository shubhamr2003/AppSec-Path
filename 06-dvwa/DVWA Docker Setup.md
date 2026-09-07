Steps to set up DVWA on your Docker Desktop

Step-1. Open your docker desktop app and make sure it is running

Step-2. Open terminal and run: 

```
git clone https://github.com/digininja/DVWA.git
```

Step-3. 

```
cd DVWA
```

Step-4.

```
docker compose up -d
```
Docker will download the required images and start the containers

Step-5. Open your Windows browser:

```
http://localhost:4280
```
You should see the DVWA interface.