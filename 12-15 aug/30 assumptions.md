### Ekho — Critical Assumptions to Validate

**P0 — могут убить весь продукт**

- [ ]  **Problem severity:** организация поступления настолько болезненна, что applicant реально хочет отдельный инструмент, а не просто терпит Google Sheets + tabs.
- [ ]  **Switching:** пользователь готов перенести реальные applications в Ekho, а не просто сказать на интервью «прикольно».
- [ ]  **Core Job:** главный job действительно близок к **«держать всё поступление под контролем и понимать, что делать дальше»**, а не только university research / essays / chances / financial aid.
- [ ]  **Target User:** international applicant, подающийся в несколько университетов, — лучший первый сегмент.
- [ ]  **Wedge:** application management + requirements является достаточно сильной причиной впервые открыть Ekho.
- [ ]  **Retention:** после первого использования существует естественная причина возвращаться каждую неделю/несколько дней.
- [ ]  **Data Trust:** пользователь доверяет структурированным данным Ekho настолько, чтобы использовать их для реальных дедлайнов и требований.
- [ ]  **Data Accuracy:** мы способны поддерживать critical admissions data достаточно актуальными, чтобы доверие не разрушалось.
- [ ]  **Data Economics:** поддержание актуальности десятков/сотен университетов не требует неподъёмного количества ручной работы.
- [ ]  **Narrow MVP:** Ekho полезен даже при ограниченной базе университетов; нам не нужны тысячи вузов до запуска.

Последний пункт особенно важен. Массовое управление несколькими applications — не выдумка: к 1 января 2026 через Common App 1.28 млн applicants отправили 7.61 млн applications — около **5.9 application на applicant**; к 1 марта — около **6.6**. Но это подтверждает наличие multi-application workflow, **не подтверждает**, что людям нужен именно Ekho.

**P1 — критичны для правильного MVP**

- [ ]  **Requirements personalization:** персональный статус `Satisfied / Missing / Optional / Action Required / Unknown` значительно полезнее обычного списка требований.
- [ ]  **Progressive profiling:** можно персонализировать Ekho, не заставляя пользователя проходить длинный onboarding.
- [ ]  **30-second value:** новый пользователь способен получить заметную ценность примерно за 30 секунд.
- [ ]  **Next Action:** один понятный «что делать дальше» полезнее большого dashboard.
- [ ]  **Source visibility:** наличие official source + last verified повышает доверие и является заметным differentiator.
- [ ]  **Global-first:** пользователю действительно важно видеть applications разных стран в одном месте.
- [ ]  **Cross-system value:** Ekho полезен даже если submission всё равно происходит через Common App, UCAS и university portals.
- [ ]  **Application count:** первые пользователи действительно управляют достаточным числом applications, чтобы отдельный workspace имел смысл.
- [ ]  **Replacement behavior:** активные пользователи начинают меньше использовать spreadsheet/Notes/собственные trackers.
- [ ]  **Simplicity differentiation:** пользователи реально предпочитают более простой продукт функционально более насыщенным альтернативам.

Есть объективная причина проверять cross-system гипотезу: например, UCAS имеет собственные этапы, reference и отдельные дедлайны, тогда как Common App содержит свои college-specific requirements; даже внутри одного университета могут существовать разные application rounds и отдельные aid deadlines.

**P2 — проверять после подтверждения core product**

- [ ]  **Live Updates:** изменения admissions pages происходят достаточно часто и достаточно важны, чтобы люди возвращались/платили за monitoring.
- [ ]  **Financial Aid Intelligence:** aid является достаточно сильной болью, чтобы стать одним из core anchors.
- [ ]  **Free Tools Acquisition:** GPA converter / deadline checker / compare и другие utilities способны приводить качественных applicants в основной продукт.
- [ ]  **Organic Sharing:** пользователи естественно рассказывают Ekho другим applicants.
- [ ]  **Community Distribution:** Reddit/Discord/student communities позволяют получать пользователей с приемлемой стоимостью.
- [ ]  **Creator Distribution:** небольшие admissions/student creators могут стать работающим acquisition channel.
- [ ]  **Paid Conversion:** часть активных пользователей готова платить за premium-функции.
- [ ]  **Premium Value:** monitoring/personalization/data/financial aid — именно те вещи, за которые будут платить.
- [ ]  **Pricing:** диапазон примерно $10–30/month приемлем для целевого пользователя.
- [ ]  **Seasonality:** продукт способен сохранять достаточно ценности на протяжении всего admissions cycle, а не использоваться один раз.
- [ ]  **Defensibility:** структурированные данные + freshness + personalization со временем создают реальное преимущество, которое трудно быстро скопировать.
- [ ]  **Category potential:** Ekho способен стать не просто полезным tracker, а стандартным workspace для поступающих.

### Что **не надо больше проверять как гипотезу**

Можно уже считать достаточно подтверждённым, что:

- applications имеют разные requirements/deadlines;
- поступающие часто подают несколько applications;
- admissions workflow включает несколько типов информации и действий;
- правила отличаются между университетами и системами;
- официальные источники существуют, но информация распределена;
- понятность admissions-информации объективно может быть проблемой.