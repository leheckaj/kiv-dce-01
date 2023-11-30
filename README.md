# Semestrální práce z KIV/DCE

Na základě projektu z [KIV/DS exercise 3](https://github.com/maxotta/kiv-ds-vagrant/tree/master/demo-3) jsem vytvořil script v Ansible a Terraformu na deployment aplikace na infrastrukturu OpenNebula na ZČU (nuada.zcu.cz).
Pro deploment aplikace je využíváno technologie Terraform (viz soubory terraform.tf), která vychází ze cvičení s Kubernetes. Terraform vygeneruje příslušné soubory pro Ansible a seznam nodů pro Frontend. Na vytvořených nodech jsou následně deployovány soubory pro backend nebo frontend dle typu nodu.
Cílem je vytvořit infrastrukturu s jedním frontendem a několika parametrem upravitelnými počty nodů.

## Terraform
Terraform se stará o vytváření infrasturktury dle JSON zápisu uvedeného v souborech .tfvars, variables.tf a terraform.tf. Projekt vychází z projektu KIV/DCE - kubernetes. Skript vytvoří DCE_Master a DCE_Nodes dle specifikace. Po vytvoření virtuálů jsou nakopírovány soubory z init-scripts a vykonány na cílových nodech. Skripty se starají o zabezpečení, update, vytvoření sudo práv pro účet pro Ansible a restart serveru. U terraformu je využita vlastnost OUTPUT, která umožňuje proměnné vykopírovat do výstupních souborů dle specifikované šablony. Tato vlastnost je využita pro vytvoření inventory a pro seznam NODES  pro frontend.  Na takto připravené servery je následně "puštěn" Ansible.
### Parametry
Pro správný běh je potřeba v terraform.tfvars specifikovat hodnoty one_username a one_password. One password je token, který se nechá získat jako API Token v OpenNebule.

  nodes_count - je parametr, který ovlivňuje počet backendů pro běh aplikace.

## Ansible
Ansible je technologie pro konfigurační správu například serverů. Ansible je definován promocí souborů ve formátu YAML. 
V rámci našeho projektu se soubory pro Ansible nachází ve složce Ansible. Pro běh ještě využívá seznam nodů uložených v dynamic_inventories/inventory.

Při instalaci backendu a frontendu se u obou na začátku nakopírují soubory z localhostu na cílový server. Následě je na frontendu nainstalován Nginx a nakopírovány konfigurační soubory pro nginx do příslušných složek. Po nastavení je zkontrolována validita konfigurace a Nginx je pomocí Handleru v Ansible restartován. Na backendech je nainstalován docker. Po překopírování souborů je zbuilděn Docker IMAGE a ten je spuštěn.

Při startu Nginx je nutné, aby všechny nodes běžely, jinak to nenaběhne. Pak již můžeme jakékoliv nodes mazat a nahrazovat.

Ansible nemá ode mě žádné parametry pro kofiguraci zvenčí. Očekává se jen, že existuje uživatel nodeadm s přávy SUDO a bude mít přístup odkudkoliv s klíčem vygenerovaným v KIV/DCE.

## Příkazy pro rozchození projektu
Předpokládá se, že uživatel vyvíjí v docker image z KIV/DCE se všemi vygenerovanými klíči a s veškerým nainstalovaným toolingem.
```
terraform init
terraform plan
terraform apply

export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook -i dynamic_inventories/inventory ansible/main-play.yml


ssh nodeadm@147.228.173.95
http://147.228.173.95/service-api/find/echo
```

### Pro případ znovu deployování images:
```
docker rm -f backend_container
docker rmi backend:v1
```
