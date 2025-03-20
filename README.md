# Ansible Role: Elasticsearch Curator

Роль Ansible, которая устанавливает [Elasticsearch Curator](https://github.com/elasticsearch/curator) на RedHat/CentOS или Debian/Ubuntu.

## Requirements

Нет, но будет намного полезнее, если у вас где-нибудь будет работать Elasticsearch!

В RedHat/CentOS убедитесь, что у вас настроен репозиторий EPEL, чтобы можно было установить пакет `python-pip`. Вы можете установить репозиторий EPEL, просто добавив `geerlingguy.repo-epel` в список ролей вашего сценария.

## Role Variables

Доступные переменные перечислены ниже вместе со значениями по умолчанию (see `defaults/main.yml`):

    elasticsearch_curator_version: ''

Версия `elasticsearch-curator` для установки. Доступные версии перечислены в [Python Package Index](https://pypi.org/project/elasticsearch-curator/). По умолчанию версия не указана, поэтому будет установлена последняя версия.

    elasticsearch_curator_cron_jobs:
      - name: "Run elasticsearch curator actions."
        job: "/usr/local/bin/curator ~/.curator/action.yml"
        minute: 0
        hour: 1

Список заданий cron. Обычно вы запускаете одно задание cron, используя действия, определённые в `action.yml`, но вы можете разделить задания cron или использовать `curator_cli` для запуска действий по отдельности, а не через файл действий. Вы можете добавить в задания cron любые из `minute`, `hour`, `day`, `weekday` и `month` — значения, которые не заданы явно, будут иметь значение `*` по умолчанию. Вы также можете использовать `state` для определения того, должно ли задание быть `present` или `absent`.

    elasticsearch_curator_config_directory: ~/.curator

Каталог, в котором будет храниться конфигурация (и файл действий) Curator.

    elasticsearch_curator_hosts:
      - 'localhost:9200'
    elasticsearch_curator_http_auth: ''

Эти переменные управляют параметрами, используемыми по умолчанию в `elasticsearch_curator_yaml`. Если вы определяете собственные `elasticsearch_curator_yaml` параметры, вам может не потребоваться определять или переопределять эти переменные.  `_hosts` — это список хостов с портами, а `_http_auth` — базовая `http_auth` `user:pass` строка, если для вашего экземпляра `Elasticsearch` требуется базовая HTTP-авторизация.

    elasticsearch_curator_yaml: |
      ---
      client:
        hosts: {{ elasticsearch_curator_hosts | to_yaml }}
        url_prefix:
        use_ssl: False
        certificate:
        client_cert:
        client_key:
        ssl_no_validate: False
        http_auth: {{ elasticsearch_curator_http_auth }}
        timeout: 30
        master_only: False
      logging:
        loglevel: INFO
        logfile:
        logformat: default
        blacklist: ['elasticsearch', 'urllib3']

Этот файл YAML помещается в файл `~/.curator/curator.yml` и содержит сведения о подключении, которые `Elasticsearch Curator` использует при подключении к `Elasticsearch`, а также конфигурацию ведения журнала `Curator`.

    elasticsearch_curator_action_yaml: |
      ---
      actions:
        1:
          action: delete_indices
          options:
            ignore_empty_list: True
            disable_action: False
          filters:
          - filtertype: pattern
            kind: prefix
            value: logstash-
            exclude:
          - filtertype: age
            source: name
            direction: older
            timestring: '%Y.%m.%d'
            unit: days
            unit_count: 45
            exclude:

Этот YAML-файл помещается в файл `~/.curator/action.yml` и определяет действия, которые выполняет `Curator` при запуске задания `cron` по умолчанию. См. [Curator actions file](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/actionfile.html).

    elasticsearch_curator_pip_package: 'python3-pip'
    elasticsearch_curator_pip_executable: "{{ 'pip3' if elasticsearch_curator_pip_package.startswith('python3') else 'pip' }}"

Системный пакет `pip`, который необходимо установить, и исполняемый файл `pip` для запуска. Для более старых операционных систем или при использовании `Python 2` вам может потребоваться изменить эти значения на `python-pip` и `pip` соответственно.

## Dependencies

  - geerlingguy.repo-epel (RedHat/CentOS only)

## Example Playbook

    - hosts: search
      roles:
        - { role: geerlingguy.elasticsearch-curator }

## License

MIT / BSD

## Author Information

This role was created in 2014 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
