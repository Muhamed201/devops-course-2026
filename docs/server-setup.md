# Эксплуатация учебной VM

## 10. Ansible

**Управляющий узел:** Ubuntu в WSL. **Управляемый узел:** devops-vm, Ubuntu Server 24.04 amd64, 192.168.56.101 в Host-only сети. SSH слушает 2222.

Проект лежит в ansible/. Inventory bootstrap.ini использует student для первичной настройки нового стенда. Inventory prod.ini использует devops и локальный ключ. Проверка ключа хоста включена. На существующей VM SSH-ключ был настроен в предыдущей работе.

| Роль | Назначение |
| --- | --- |
| base | Проверка ОС, системные зависимости, Docker Engine |
| ssh | Открытый ключ, sudoers, параметры SSH, правила UFW |
| nginx | Виртуальный хост по IP и проверка конфигурации перед reload |
| app | Получение проверенного образа, контейнер и health-check |

Образ — ghcr.io/muhamed201/devops-ci-app с тегом 9ea8df6b995744f53713dd3b05061873cb691394. Приложение слушает порт 5000 в контейнере. Для нового приложения Docker публикует 127.0.0.1:5003 на VM. Внешняя проверка — http://192.168.56.101/health через Nginx.

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml --syntax-check

# Для нового стенда с паролем student:
ansible-playbook -i inventory/bootstrap.ini site.yml --ask-pass --ask-become-pass

# Для существующего стенда и повторных запусков:
ansible-playbook site.yml
ansible-playbook verify.yml
curl --fail http://192.168.56.101/health
```

Корректный результат повторного применения — changed=0. Аудит проверяет SSH, состояние контейнера, привязку порта и health-ответ. В автоматическом выпуске значение app_tag приходит через -e.
