# OS Bootstrap

## How to use

Install Ansible Galaxy packages:

    ansible-galaxy install -r requirements.yml

Replace variable placeholders:

    grep placeholder group_vars/vars.yml

Run the playbook:

    ansible-playbook playbook.yml -l localhost -K