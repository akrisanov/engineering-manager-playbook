# Ansible

### Базовые знания

{% embed url="https://youtube.com/playlist?list=PLT98CRl2KxKEUHie1m24-wkyHpEsa4Y70&si=F6XdoLrMgvp7_Jyb" %}

### Инструменты

#### Линтер `ansible-lint`

```
# установка
pip install --upgrade ansible-lint
# использование
ansible-lint .
```

Может не работать на Windows (тред в [GitHub](https://github.com/ansible/ansible-lint/issues/328)) → [устанавливайте в WSL2](https://blog.jora.dev/en/posts/ansible-lint-in-vscode-on-wsl/).

#### Плагины для Visual Studio Code

* [Ansible](https://marketplace.visualstudio.com/items/?itemName=redhat.ansible) – официальный плагин от RedHat
* [Error Lens](https://marketplace.visualstudio.com/items/?itemName=usernamehw.errorlens) – показывает ошибки in-line, а не в статусной строке редактора
* [Better Jinja](https://marketplace.visualstudio.com/items/?itemName=samuelcolvin.jinjahtml) – для работы с шаблонами
* [markdownlint](https://marketplace.visualstudio.com/items/?itemName=DavidAnson.vscode-markdownlint) – линтер для Markdown
* [Markdown Preview](https://marketplace.visualstudio.com/items/?itemName=shd101wyy.markdown-preview-enhanced) – просмотр рендеринга разметки
* [Better Toml](https://marketplace.visualstudio.com/items/?itemName=tamasfe.even-better-toml) – поддержка языка TOML
* [Nginx Configuration Support](https://marketplace.visualstudio.com/items/?itemName=ahmadalli.vscode-nginx-conf) – поддержка языка конфигурирования
* [nginx-formatter](https://marketplace.visualstudio.com/items/?itemName=raynigon.nginx-formatter) – форматироавние конфига
* [Path Intellisense](https://marketplace.visualstudio.com/items/?itemName=christian-kohler.path-intellisense) – автокомплит для путей к файлам
* [Todo Tree](https://marketplace.visualstudio.com/items/?itemName=Gruntfuggly.todo-tree) – выводит списком комментарии `TODO`, `FIXME` и пр. в отдельной панели
* [String Manipulation](https://marketplace.visualstudio.com/items/?itemName=marclipovsky.string-manipulation) – перевод строки в uppercase, downcase, titlecase
* [Trailing Spaces](https://marketplace.visualstudio.com/items/?itemName=shardulm94.trailing-spaces) – показывает лишние пробелы в конце строк и удаляет их

<figure><img src="../.gitbook/assets/image.png" alt="Пример работы линтера и плагина Ansible"><figcaption></figcaption></figure>
