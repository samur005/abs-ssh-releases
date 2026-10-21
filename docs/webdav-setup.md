# Синхронизация через WebDAV / WebDAV sync setup

Файл синхронизации (`abs-ssh.abssync`) шифруется на устройстве (AES-GCM, ключ из пароля синхронизации через Argon2id) — провайдер видит только зашифрованный файл. Пароль синхронизации (≥ 8 символов) никуда не отправляется: одинаковый пароль нужен на всех устройствах.

The sync file is encrypted on the device; the provider only stores ciphertext. Use the same sync password (≥ 8 chars) on every device.

## Как настроить / How to set up
1. **Настройки → Синхронизация → WEBDAV** (Settings → Sync → WEBDAV).
2. Выберите шаблон сервиса — он заполнит URL; замените `HOST`/`USER`/`NAS`/`SHARE`, если есть. / Pick a preset; replace placeholders.
3. Логин и пароль — см. таблицу. **Если включена двухфакторная аутентификация (2FA), нужен пароль приложения**, обычный пароль не подойдёт. / With 2FA enabled you need an app-specific password.
4. Придумайте пароль синхронизации → **Отправить** на первом устройстве, **Получить** на остальных. / Push on the first device, Pull on the others.

URL — это полный путь к файлу; папки по пути должны уже существовать (кладите файл в корень или в созданную заранее папку). / The URL is the full file path; parent folders must exist.

## Сервисы / Services (проверено по официальной документации, октябрь 2026)

| Сервис | URL в приложении | Логин | Пароль |
|---|---|---|---|
| Яндекс Диск | `https://webdav.yandex.ru/abs-ssh.abssync` | логин Яндекса | **пароль приложения** типа «WebDAV» (Яндекс ID → Безопасность → Пароли приложений; активируется до 2–3 ч). ⚠ С 22.06.2026 WebDAV доступен только с подпиской **Яндекс 360**. |
| Облако Mail | `https://webdav.cloud.mail.ru/abs-ssh.abssync` | полный адрес ящика | **пароль для внешнего приложения** с доступом к Облаку (Настройки → Безопасность → Пароли для внешних приложений). Обычный пароль не работает с 01.01.2022. ⚠ По сторонним данным WebDAV доступен только на платных тарифах — проверьте свой тариф. |
| Nextcloud | `https://HOST/remote.php/dav/files/USER/abs-ssh.abssync` | имя пользователя | пароль или **пароль приложения** (Настройки → Безопасность → Устройства и сессии; обязателен при 2FA) |
| ownCloud 10 / oCIS | `https://HOST/remote.php/dav/files/USER/abs-ssh.abssync` | имя пользователя | пароль / токен приложения (при 2FA) |
| Koofr | `https://app.koofr.net/dav/Koofr/abs-ssh.abssync` | e-mail | **только пароль приложения** (Preferences → Password → App passwords → Generate) |
| pCloud | США: `https://webdav.pcloud.com/abs-ssh.abssync` · ЕС: `https://ewebdav.pcloud.com/abs-ssh.abssync` | e-mail | пароль pCloud (выберите сервер своего региона; при 2FA подтвердите вход из письма) |
| 4shared | `https://webdav.4shared.com/abs-ssh.abssync` | логин 4shared | пароль аккаунта |
| Synology NAS | `https://NAS:5006/SHARE/abs-ssh.abssync` | учётка DSM | пароль DSM (установите пакет **WebDAV Server**, включите HTTPS 5006) |
| Свой сервер (Apache mod_dav, nginx-dav, rclone serve webdav, SFTPGo…) | `https://host/path/abs-ssh.abssync` | — | — |

**Не поддерживается / Not supported:** **Box** — WebDAV (`dav.box.com`) отключён Box с 28.04.2023 (официальное объявление EOL). Используйте другой сервис.

Используйте только `https://`. Самоподписанные сертификаты не принимаются (проверка TLS не отключается).

### Sources
- Yandex: https://yandex.com/support/yandex-360/customers/disk/web/en/webdav , https://yandex.com/support/id/en/authorization/app-passwords.html
- Mail.ru: https://help.mail.ru/cloud/desktop/webdav/
- Koofr: https://koofr.eu/help/koofr_with_webdav/how-do-i-connect-a-service-to-koofr-through-webdav/
- pCloud: https://help.pcloud.com/article/connect-to-pcloud-using-webdav-and-rsync
- 4shared: https://search.4shared.com/faq.jsp
- Box EOL: https://support.box.com/hc/en-us/articles/360052806073
- Nextcloud: https://docs.nextcloud.com/server/latest/user_manual/en/files/access_webdav.html
