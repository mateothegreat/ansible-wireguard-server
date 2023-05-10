setup:
	if [ ! -d "venv" ]; then python3 -m venv venv; fi
	venv/bin/pip install -r requirements.txt

reset: nuke wireguard/server

nuke:
	venv/bin/ansible-playbook -i inventory.yaml nuke.yaml

wireguard/server:
	venv/bin/ansible-playbook -i inventory.yaml deploy.yaml
