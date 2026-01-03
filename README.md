Specialist Journal — README (укр.)

Коротко: репозиторій містить кастомний модуль specialist_journal для Odoo 19 і docker-скелет для локальної розробки (dev) та розгортання (prod). Нижче — логіка структури проекту, навіщо кожен файл/папка і що перевіряти при тестуванні. Команди для підняття контейнерів — в кінці.

Структура проєкту (пояснення)
```
.
├── addons/
│   └── specialist_journal/         # основний модуль Odoo
│       ├── __init__.py             # імпорт моделей/модулів
│       ├── __manifest__.py         # манифест (залежності, data-послідовність)
│       ├── models/                 # python-моделі (ORM)
│       │   ├── journal_entry.py    # модель запису журналу (specialist.journal)
│       │   ├── res_partner.py      # розширення res.partner (one2many на записи)
│       │   └── template.py         # модель шаблонів записів (specialist.template)
│       ├── reports/                # QWeb шаблон(и) та action для PDF
│       │   ├── report_journal_action.xml
│       │   └── report_journal_template.xml
│       ├── security/               # права доступу та правила
│       │   ├── groups.xml
│       │   ├── ir.model.access.csv
│       │   └── record_rules.xml
│       ├── views/                  # xml-view'и та спадкування форм
│       │   ├── journal_views.xml
│       │   ├── res_partner_inherit.xml
│       │   ├── template_views.xml
│       │   └── wizard_report_views.xml
│       └── wizards/                # transient wizard для друку (report wizard)
│           └── print_journal_wizard.py
├── config/
│   └── odoo.conf                   # конфіг Odoo (addons_path, db_host і т.п.)
├── data/
│   ├── filestore/                  # filestore Odoo (файлові вкладення)
│   └── pgdata/                     # (не рекомендується мапити на Windows NTFS)
├── docker/
│   └── nginx/nginx.conf            # проксі-конфіг для Odoo (reverse proxy)
├── docker-compose.dev.yml          # compose для розробки (nginx + odoo; БД — зовнішня)
└── docker-compose.prod.yml         # compose для проду (db + pgadmin + nginx + odoo)
```

Чому саме так

addons/specialist_journal — стандартна структура Odoo-модуля: models, views, security, reports, wizards. Це дозволяє модуль встановлювати як звичайний Odoo addon.

Моделі (models) — вся бізнес-логіка та структура даних має бути в Python (ORM). XML — тільки для опису форм/видів/репортів.

security — ACL і record rules визначають, що спеціаліст може створювати/редагувати лише свої записи, а менеджер — бачити й правити всі.

reports / wizard — логіка формування даних вираховується в wizard (python), QWeb — лише рендерить HTML/PDF. Це правильна архітектура для продуктивності й безпеки (менше логіки в QWeb).

docker-compose.prod.yml — повний стек для перевірки того, що модуль працює в реалістичному середовищі (Postgres + pgAdmin + nginx + Odoo).
dev — мінімальний стек: nginx + odoo (БД — локальна або віддалена), щоб швидко тестувати модуль, не піднімаючи повний postgres-кластер.

Що перевіряти (короткий чек-лист для тестування)

Встановити модуль через Apps (Activate developer mode → Update Apps List → знайти Specialist Journal → Install).

Відкрити картку клієнта (Contacts) → вкладка Journal повинна з'явитися.

Створити шаблон(и) запису → створити запис, застосувати шаблон (можна декілька) → відредагувати вміст → зберегти.

Перевірити права:

створити двох користувачів у групі Journal Specialist (вони мають створювати і редагувати лише власні записи);

адміністратор / Journal Manager — бачить і редагує всі записи та шаблони.

Тест PDF: у вкладці Journal натиснути Print Journal → ввішеться wizard → вибрати All entries або вказати період → натиснути Print → згенерується PDF.

Якщо при спробі згенерувати PDF з'являється повідомлення про missed filleds, значить в профілі користувача/карточці партнера відсутні обов'язкові дані — Odoo підсвічує, яке поле потрібно заповнити.

Примітки по розгортанню (логіка)

Prod: docker-compose.prod.yml піднімає всю екосистему — Postgres, pgAdmin, nginx, Odoo. Odoo у Prod в цьому проєкті іниціалізує/підключається до своєї бази (може бути налаштований так, щоби створювати базу під час старту або робити -i base). Це зручно для перевірки повного життя модуля.

Dev: docker-compose.dev.yml піднімає nginx + odoo, а базу використовує локальну/удалену (щоб не мучитися з Postgres volume на Windows). Модуль у Dev ставиться вручну через Apps — так зручніше для інтерактивного тестування і швидких правок.

Команди для збірки / запуску (тільки вони потрібні)

Prod (повна екосистема):

docker compose -f docker-compose.prod.yml --env-file .env.prod up --build


Dev (nginx + odoo, БД зовнішня або локальна):

docker compose -f docker-compose.dev.yml --env-file .env.dev up --build
