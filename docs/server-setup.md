# Настройка сервера


## 10. Ansible

**Расположение проекта.** Каталог `ansible/` в репозитории `devops-ci-app`
(ветка `feature/ansible`). Запуск выполняется из каталога `ansible/`,
так как `ansible.cfg` рассчитан на этот каталог.

**Узлы.** Управляющий узел: Ubuntu в WSL на хосте студента (Ansible 2.16.3).
Управляемый узел: VM `devops-vm`, Ubuntu Server 26.04, адрес Host-only
`192.168.56.102`, SSH на порту 22.

**Inventory и учётные записи.**
- `inventory/bootstrap.ini`: первичный запуск от `student` по паролю
  (`--ask-pass --ask-become-pass`).
- `inventory/prod.ini`: последующие запуски от `devops` по ключу
  `~/.ssh/devops_vm`, без интерактивного ввода. Это inventory по умолчанию
  (`ansible.cfg`).
- Пароли и закрытый ключ в репозиторий не входят.

**Роли и ответственность.**
- `base`: проверка ОС, базовые пакеты, `python3-requests`, официальный
  репозиторий Docker, служба Docker, отключение IPv6 для доступа к реестру.
- `ssh`: пользователь `devops`, SSH-ключ, sudo, hardening SSH
  (пароли отключены), UFW (22/tcp и 80/tcp).
- `nginx`: пакет Nginx, reverse proxy на `127.0.0.1:5000`, отключение
  default-сайта, проверка `nginx -t` перед reload.
- `app`: загрузка образа, контейнер `devops-ci-app`, проверка `/health`.

**Образ приложения.** `ghcr.io/damir-ilyasov/devops-ci-app`, тег равен полному
SHA коммита `4bf987f4240f9d3e97646e9a2e98407362a28139` (переменная `app_tag`
в `group_vars/all.yml`, тег `latest` не используется). Пакет публичный,
для pull достаточно анонимного чтения.

**Порты и health.** Контейнер публикуется на `127.0.0.1:5000` (внутренний
порт 5000), наружу доступен только Nginx на порту 80. Health endpoint:
`/health`, ожидаемый ответ `{"status":"ok"}`.

**Команды.**
- Первичный запуск: `ansible-playbook -i inventory/bootstrap.ini site.yml --ask-pass --ask-become-pass`
- Повторный запуск: `ansible-playbook site.yml` (ожидается `changed=0`)
- Read-only аудит: `ansible-playbook verify.yml`
