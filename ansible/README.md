# Управление учебной VM

Управляющий узел WSL подключается по SSH к devops-vm в сети Host-only. Используются существующий порт 2222, локальный ключ ~/.ssh/devops_vm и known_hosts. Открытый ключ берётся из ~/.ssh/devops_vm.pub.

Из ansible/ установите коллекции командой ansible-galaxy collection install -r requirements.yml. Затем проверьте ansible-inventory --graph и ansible-playbook site.yml --syntax-check. Для проверки конфигурации после деплоя используйте ansible-playbook verify.yml.

Плейбук site.yml применяет роли base, ssh, nginx и app. Nginx обслуживает IP VM на порту 80 и передаёт запрос в контейнер на 127.0.0.1:5003. Именованные сайты предыдущих работ используют собственные server_name. Для деплоя фиксируется SHA образа, а при автоматическом выпуске app_tag передаётся через -e.
