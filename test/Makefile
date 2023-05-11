.DEFAULT_GOAL := all

all: install server clients

clean:
	@echo "Cleaning up.."
	@rm -rf venv .galaxy

setup:
	@echo "Configuring virtual environment & installing requirements (if needed).."
	@if [ ! -d "venv" ]; then python3 -m venv venv && venv/bin/pip install --upgrade -r requirements.txt; else venv/bin/pip install --upgrade -r requirements.txt --report - --quiet 2> /dev/null | jq -r '.install[] | "\(.metadata.name)  \(.metadata.version)"'; fi

reset: setup
	@echo "Resetting.."
	@venv/bin/ansible-playbook -i inventory.yaml playbook.yaml --tags reset

install: setup
	venv/bin/ansible-playbook -i inventory.yaml playbook.yaml --tags install

server: setup
	venv/bin/ansible-playbook -i inventory.yaml playbook.yaml --tags server

clients: setup
	venv/bin/ansible-playbook -i inventory.yaml playbook.yaml --tags clients
