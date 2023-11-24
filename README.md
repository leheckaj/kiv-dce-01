# Senestrální práce z KIV/DCE

```
terraform init
terraform plan
terraform apply

export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i dynamic_inventories/inventory ansible/main-play.yml

docker rm -f frontend_container
docker rmi frontend:v1

ssh nodeadm@147.228.173.95
http://147.228.173.95/service-api/find/echo

ssh nodeadm@1.2.3.4

Při startu Nginx je nutné, aby všechny nodes běžely, jinak to nenaběhne. Pak již můžeme jakékoliv nodes mazat a nahrazovat.
```
