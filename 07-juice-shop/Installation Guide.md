Install Docker Desktop -> Open windows powershell

run:
docker pull bkimminich/juice-shop
docker run --rm -p 3000:3000 bkimminich/juice-shop
http://localhost:3000

p.s: keep the powershell open or else the juice shop will close as well


