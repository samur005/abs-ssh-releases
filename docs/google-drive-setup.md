# Google Диск — пошаговая настройка (RU)

ABS SSH хранит **зашифрованный** контейнер синхронизации (тот же формат AES-256-GCM + Argon2id, что для файла и WebDAV) в скрытой папке приложения на вашем Google Диске (`appDataFolder`, scope `drive.appdata`). Приложение не видит остальные файлы Диска, а Google не видит содержимое — ключ получается только из вашей парольной фразы.

Google требует, чтобы у приложения был зарегистрированный OAuth-клиент. ABS SSH не встраивает чужие ключи: вы создаёте **свой** проект (бесплатно, ~10 минут).

## 1. Проект в Google Cloud Console
1. Откройте https://console.cloud.google.com/ и войдите тем аккаунтом, чей Диск будете использовать.
2. Вверху: выбор проекта → **Новый проект** → имя, например `abs-ssh-sync` → **Создать**. Убедитесь, что проект выбран.

## 2. Включите Google Drive API
**API и сервисы → Библиотека** → найдите **Google Drive API** → **Включить**.

## 3. Экран согласия OAuth (OAuth consent screen / Google Auth Platform)
1. **API и сервисы → Экран согласия OAuth** (в новом интерфейсе: **Google Auth Platform → Брендинг / Аудитория / Доступ к данным**).
2. Тип пользователей: **Внешний** (External). Имя приложения — любое (например «ABS SSH (личное)»), e-mail поддержки и разработчика — ваш.
3. **Доступ к данным → Добавить или удалить области**: добавьте `https://www.googleapis.com/auth/drive.appdata` (не «drive» целиком).
4. **Аудитория → Тестовые пользователи**: добавьте свой Google-аккаунт. Публиковать приложение не нужно; в режиме «Тестирование» токен обновления живёт ~7 дней — потом просто нажмите «Войти через Google» ещё раз (или переведите приложение в «В производстве» — для области `drive.appdata` проверка Google не требуется).

## 4. OAuth-клиент для Android
**API и сервисы → Учётные данные → Создать учётные данные → Идентификатор клиента OAuth**:
- Тип приложения: **Android**.
- Название пакета: `com.abs.ssh`
- Отпечаток сертификата SHA-1 — того ключа, которым подписан установленный у вас APK:
  - официальные релизы (`abs-ssh-…-android-release.apk` из https://github.com/samur005/abs-ssh-releases): 
    `95:40:75:DE:9B:9B:E5:CE:41:6F:C0:06:E0:E2:1A:EA:03:C4:D4:8A`
  - собственная debug-сборка: `keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android` → строка `SHA1:` (Windows: `%USERPROFILE%\.android\debug.keystore`).
- **Создать**. Вводить этот client ID в приложение на Android **не нужно**: Google сопоставляет приложение по пакету и SHA-1 (Google Identity / AuthorizationClient).

## 5. OAuth-клиент для Windows (ПК) — в ТОМ ЖЕ проекте
> **Важно:** папка `appDataFolder` своя у каждого проекта Google Cloud. Android и Windows видят одну и ту же копию, только если их OAuth-клиенты созданы **в одном проекте** (в том, где Android-клиент из шага 4). Клиент из другого проекта увидит пустую папку — PULL ответит «на сервере ничего нет».

В том же проекте снова **Создать учётные данные → Идентификатор клиента OAuth**:
- Тип приложения: **Приложение для ПК** (Desktop app), название любое → **Создать**.
- Скопируйте **Client ID** (вида `1234-abc.apps.googleusercontent.com`) и **Client secret** (для ПК-клиентов Google не считает его секретом, но не публикуйте его).
- Куда вписать:
  - **Сборка со встроенным клиентом** (если в Windows-версии под полем написано «Используется встроенный клиент (общий с Android)») — ничего вводить не нужно, просто **Войти через Google**.
  - Иначе — в ABS SSH (Windows): Настройки → Зашифрованная синхронизация → **GOOGLE DRIVE** → поля **Client ID** и **Client secret** → **Войти через Google**. Введённый ID всегда важнее встроенного.
  - Для сборщика: встроить ID при сборке — `ABS_GDRIVE_DESKTOP_CLIENT_ID=… ABS_GDRIVE_DESKTOP_CLIENT_SECRET=… ./gradlew :windowsApp:jar` (или `-PabsGdriveDesktopClientId=…`). В репозиторий значения не попадают. Откроется браузер; после согласия вкладка покажет «connected». Приложение использует поток PKCE с адресом возврата `http://127.0.0.1:<порт>` — настраивать redirect URI в консоли не нужно.

## 6. Использование
1. На первом устройстве: Google Drive → **Войти** → парольная фраза → **PUSH**.
2. На втором: войти тем же аккаунтом, та же фраза → **PULL**.
3. Конфликты обрабатываются как в WebDAV: ничего не перезаписывается молча.

## Частые ошибки
- `403 access denied` — не включён Drive API (шаг 2) или не добавлена область `drive.appdata` / аккаунт не в тестовых пользователях.
- Android: «Developer error» / код 10 — SHA-1 или имя пакета в Android-клиенте не совпадают с установленным APK.
- `401 token expired` / `invalid_grant` — войдите заново (в режиме «Тестирование» раз в ~7 дней).
- Выйти: кнопка **ВЫЙТИ** удаляет токен из защищённого хранилища; отозвать доступ полностью — https://myaccount.google.com/permissions.

Ограничение: у Drive API нет условной записи, поэтому версия файла проверяется непосредственно перед записью; одновременный PUSH с двух устройств в одну и ту же секунду теоретически может пройти без конфликта.

---

# Google Drive — step-by-step setup (EN)

ABS SSH stores an **encrypted** sync container (AES-256-GCM + Argon2id, same format as file/WebDAV) in your Drive's hidden app folder (`appDataFolder`, scope `drive.appdata`). The app cannot see your other files; Google cannot read the content.

1. **Project:** https://console.cloud.google.com/ → **New project** (e.g. `abs-ssh-sync`) → select it.
2. **Enable API:** APIs & Services → Library → **Google Drive API** → Enable.
3. **Consent screen** (Google Auth Platform): External; add scope `https://www.googleapis.com/auth/drive.appdata`; add your account as a **test user** (in Testing mode re-sign-in is needed about every 7 days).
4. **Android client:** Credentials → Create credentials → OAuth client ID → **Android**, package `com.abs.ssh`, SHA-1 of the APK signing certificate (official releases: `95:40:75:DE:9B:9B:E5:CE:41:6F:C0:06:E0:E2:1A:EA:03:C4:D4:8A`). Nothing to type into the Android app — Google matches package + SHA-1.
5. **Windows client — in the SAME project:** `appDataFolder` is per Google Cloud project, so Android and Windows share the backup only if both OAuth clients live in the project from step 4. Create credentials → OAuth client ID → **Desktop app** → copy Client ID and Client secret (Google does not treat a desktop secret as confidential, but don't publish it).
   - Build with a built-in client (the Windows app shows "Built-in client (shared with Android) is used"): nothing to enter, just **Sign in with Google**.
   - Otherwise: Windows app → Settings → Encrypted sync → **GOOGLE DRIVE** → paste **Client ID** / **Client secret** → **Sign in with Google**. A user-entered ID overrides the built-in one.
   - Builders: inject at build time with `ABS_GDRIVE_DESKTOP_CLIENT_ID` / `ABS_GDRIVE_DESKTOP_CLIENT_SECRET` env vars (or `-PabsGdriveDesktopClientId=…`); the values are never committed.
6. **Use:** device 1 → Sign in → passphrase → **PUSH**; device 2 → same Google account, same passphrase → **PULL**. Conflicts are never overwritten silently.

Common errors: `403` — Drive API not enabled / scope or test user missing; Android "Developer error" (code 10) — package or SHA-1 mismatch; `invalid_grant` — sign in again; PULL says nothing remote although the other device pushed — the two clients are in **different projects** (see step 5).
